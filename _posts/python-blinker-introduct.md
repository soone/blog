title: python之blinker模块简单介绍(翻)
date: 2013-02-20 22:05:22
tags:
- python
- blinker
categories:
- python
- programming
---
本文纯粹翻译自[blinker](http://discorporate.us/projects/Blinker)官网的1.1版本的[documentation](http://discorporate.us/projects/Blinker/docs/1.1/signals.html)  
首先介绍下blinker模块的作用： blinker为python对象提供快速、简单的object-to-object和信号广播功能。  

blinker的核心虽然很小但却提供了强大的功能：  

* 全局命名的信号注册  
* 匿名信号  
* 自定义名称注册  
* 永久或暂时的连接接收器  
* 通过弱引用自动断开连接接收器  
* sending arbitrary data payloads(这句没看明白)  
* 从信号接收器搜集返回值  

###**信号**  
####**通过命名信号解耦**  

通过**singal()**进行信号命名:
{% codeblock %}
>>> from blinker import signal
>>> initialized = signal('initialized')
>>> initialized is signal('initialized')
True
{% endcodeblock %}
每次调用**signal('name')**都将返回想通过的信号对象，它允许那些没有连接部分的代码(不同的模块、插件、任何东西)都能够在没有要求任何其他代码共享或制定导入的情况下使用这个相同的信号。  

####**订阅信号**  

**signal.connect()**注册一个函数，该函数在每次这个信号被发送的时候被调用。注册函数总是会被传递到引起信号发送的改对象中。  
{% codeblock %}
>>> def subscriber(sender):
...     print("Got a signal sent by %r" % sender)
...
>>> ready = signal('ready')
>>> ready.connect(subscriber)
<function subscriber at 0x...>
{% endcodeblock %}

####**信号发送**  

代码产生的事件会通过**Signal.sender()**通知到所有已连接的接收器上。  
{% codeblock %}
>>> class Processor:
...     def __init__(self, name):
...         self.name = name
... 
...     def go(self):
...         ready = signal("ready")
...         ready.send(self)
...         print("Processing")
...         complete = signal("complete")
...         complete.send(self)
... 
...     def __repr__(self):
...         return "<Processor %s>" % self.name
... 
>>> processor_a = Processor('a')
>>> processor_a.go()
Got a signal sent by <Processor a>
Processing
{% endcodeblock %}
注意看complete这个信号虽然也在go()函数中，但并还没有连接上这个信号的接收器，在没有接收器的情况下调用**send()**不会发送任何通知并且这些没有发送操作将尽可能的被低廉的优化掉。（译者注：不是很明白）  

####**订阅制定的发送者**

当任何的发送者发送信号，信号连接时默认的那个接收器将被调用。**Signal.connect()**函数接受一个可选的参数表明订阅者对应的特定发送对象。  
{% codeblock %}
>>> def b_subscriber(sender):
...     print("Caught signal from processor_b.")
...     assert sender.name == 'b'
... 
>>> processor_b = Processor('b')
>>> ready.connect(b_subscriber, sender=processor_b)
{% endcodeblock %}
这个函数只有当**processor_b**函数在发送的时候才会对ready订阅生效。  
{% codeblock %}
>>> processor_a.go()
Got a signal sent by <Processor a>
Processing.
>>> processor_b.go()
Got a signal sent by <Processor b>
Caught signal from processor_b
Processing.
{% endcodeblock %}

####**通过信号发送和接收数据**  

**send()**函数添加额外的关键词参数，这些参数将被传递到已连接的函数:  
{% codeblock %}
>>> send_data = signal("send-data")
>>> @send_data.connect
... def receive_data(sender, **kw):
...     print("Caught signal from %r, data %r" % (sender, kw))
...     return "received!"
... 
>>> result = send_data.send("anonymous", abc=123)
Caught signal from "anonymous", data {'abc': 123}
{% endcodeblock %}
**send()**函数返回每个连接函数的(receiver function, return value)形式的列表。  
{% codeblock %}
>>> result
[(<function receive_data at 0x...>, 'received!')]
{% endcodeblock %}

####**匿名信号**  

信号并不一定要命名。**Signal**在每次调用构造的时候会生成一个唯一的信号。比如：上面Processor的另一种实现提供了processing信号最为类的属性。  
{% codeblock %}
>>> from blinker import Signal
>>> class AltProcessor:
...     on_ready = Signal()
...     on_complete = Signal()
... 
...     def __init__(self, name):
...         self.name = name
... 
...     def go(self):
...         self.on_ready.send(self)
...         print("Alternate processing.")
...         self.on_complete.send(self)
... 
...     def __repr__(self):
...         return "<AltProcessor %s>" % self.name
... 
{% endcodeblock %}

####**连接作为装饰器**  

将上面的代码在终端执行输出后，你可能已经注意到了**connect()**的返回值了。**connect()**被允许用来做位函数的装饰器。
{% codeblock %}
>>> apc = AltProcessor('c')
>>> @apc.on_complete.connect
... def completed(sender):
...     print "AltProcessor %s completed!" % sender.name
... 
>>> apc.go()
Alternate processing.
AltProcessor c completed!
{% endcodeblock %}
不行的是，这种形式虽然方便，但它不允许发送者或者弱参数为连接函数进行自定义。针对这点，可以用**connect_via()**函数。  
{% codeblock %}
>>> dice_roll = signal('dice_roll')
>>> @dice_roll.connect_via(1)
... @dice_roll.connect_via(3)
... @dice_roll.connect_via(5)
... def odd_subscriber(sender):
...     print("Observed dice roll %r." % sender)
... 
>>> result = dice_roll.send(3)
Observed dice roll 3
{% endcodeblock %}

####**优化信号发送**  

不管接受者有没有连接，信号总是被优化得发送非常快。如果要发送的信号非常昂贵，它会通过接收器的属性来更有效的检验是否有接收器连接。  
{% codeblock %}
>>> bool(signal('ready').receivers)
True
>>> bool(signal('complete').receivers)
False
>>> bool(AltProcessor.on_complete.receivers)
True
{% endcodeblock %}
检查一个接收器监听的特定的发件人也是可能的。  
{% codeblock %}
>>> signal('ready').has_receivers_for(processor_a)
True
{% endcodeblock %}

####**信号的文档块**  

不管是不是匿名信号，都可以传递一个doc参数去为信号设置pydoc帮助文档。这个帮助文档将会被大多数的文档生成起所检出，对将被发送的信号的参数进行文档注释是很好的方式。  
具体例子可以查看**[receiver_connected](http://discorporate.us/projects/Blinker/docs/1.1/api.html#blinker.base.receiver_connected)**函数的文档。  
