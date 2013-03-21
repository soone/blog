title: 关于python的装饰器
date: 2013-03-17 17:54:54
tags:
- python
- decorator
categioes:
- python
- tech
---
python提供的装饰器是个很优雅的语法，不过这里面的理解也让我花了比较多的时间，python的装饰器其实有多种方式，google出来的文章比较多的是介绍函数装饰器，以及python内置的classmethod，staticmethod和property这些秀时期。  
关于python的函数装饰器和内置装饰器可以参考一下几个我google出来的文章：  
1. [Python装饰器学习](http://blog.csdn.net/thy38/article/details/4471421 "Python装饰器学习")  
2. [Python装饰器与面向切面编程](http://www.cnblogs.com/huxi/archive/2011/03/01/1967600.html "Python装饰器与面向切面编程")  
3. [理解Python中的装饰器](http://www.codecho.com/understanding-python-decorators/ "理解Python中的装饰器")  
4. [python装饰器的一个妙用](http://www.vimer.cn/2011/04/python装饰器的一个妙用.html "python装饰器的一个妙用")  
下面重点描述下我对类装饰器的理解，先看看下面的例子：
{% codeblock %}
class DecoClass:

    def __init__(self, func):
        print "deco init"
        self.func = func

    def __call__(self):
        print "deco be called"
        return self.func() + 10

@DecoClass
def testA():
    return 10

print testA()
{% endcodeblock %}
上面的代码会得到下面的结果
{% codeblock %}
deco init
deco be called
20
{% endcodeblock %}
类修饰器绑定的函数在调用时，会生成一个类的实例，这时候就调用\_\_init\_\_打印出了"deco init"，接着这个类实例会被调用，即调用了\_\_call\_\_方法，这时候打印出了"deco be called"，同时testA()返回的10加上10得到20的结果。它等同于:
{% codeblock %}
print DecoClass(testA)()
{% endcodeblock %}
