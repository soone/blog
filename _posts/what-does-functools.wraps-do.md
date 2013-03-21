title: functools.wraps用来做什么(翻)
date: 2013-02-18 00:39:12
tags:
- python
- functools
categories:
- python
- tech
---
以前对functools.wraps总是没有好好的深入去了解，刚好看到stackoverflow上有这样一个[帖子](http://stackoverflow.com/questions/308999/what-does-functools-wraps-do "what does functools.wraps do?")在解释，就翻译过来，当作笔记记录。  
## 问题：  
在本站的另一个问题的评论里，有一些网友在说他们不确定functools.wraps是用来做什么的，于是我就单独在网站上问一下这个问题，这样在stackoverflower上就有对functools.wraps特性的一个记录了。  
## 第一个答案  
当我们在用装饰器的时候，其实就是用一个函数替换另一个函数。换句话说，就是比如你有如下一个装饰器
{% codeblock %}
def logged(func):  
    def with_logging(*args, **kwargs):
        print func.__name__ + " are called"
        return func(*args, **kwargs)
    return with_logging
{% endcodeblock %}
接下去，把这个装饰器用在一个函数上
{% codeblock %}
@logged
def f(x):
    """does some math"""
    return x + x * x
{% endcodeblock %}
上面的代码实际和下面的代码是一个效果
{% codeblock %}
def f(x):
    """does some math"""
    return x + x * x
f = logged(x)
{% endcodeblock %}
f这个函数被with\_logging函数替换掉了。很不幸的，这样就意味着如果你执行下面的代码
{% codeblock %}
print f.__name__
{% endcodeblock %}
将打印**with_logging**，因为这个是新函数的名字。事实上如果你查看f函数的\_\_doc\_\_属性，那将返回空，因为with\_logging并没有设置\_\_doc\_\_描述，所以在f函数上设置的那个\_\_doc\_\_描述内容将不再会出现了。如果你查看pydoc的结果，那你将看到的是\*args和\*\*kwargs而不再是x，因为现在整个函数就是with\_logging而不是f了。  
按照上面的使用方法，如果你使用一个装饰器，那就意味着你将丢失被装饰的函数的这些信息，这是个很严重的问题。而这就是为什么我们要使用functools.wraps的原因了。它将被装饰函数的函数名称、docstring、形参等拷贝到新的这个装饰器里。\*\*wraps\*\*本身就是个装饰器，下面的代码可以正确显示我们想要的。
{% codeblock %}
from functools import wraps
def logged(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print func.__name__ + " are called"
        return func(*args, **kwargs)
    return with_logging

@logged
def f(x):
    """does same math"""
    return x + x * x

print f.__name__ # 打印"f"
print f.__doc__ # 打印"does same math"
{% endcodeblock %}

