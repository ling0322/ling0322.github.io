--- 
layout: post
title: Python3的静态类型检查
---

Python和大多数的解释型语言一样，属于弱类型的语言。它的优点就是语言灵活，不需要考虑各种类型以及模板。但是缺少类型检查也会引入很多问题。比如，

{% highlight python %}
def join(string_list):
    return ', '.join(string_list)
{% endhighlight %}

对于这个函数，期望的结果就是会把['hello', 'world']变成'hello, world'。 但是如果不小心没有传list只是传了一个字符串'hello'，这段代码也不会报错，只是会返回'h, e, l, l, o'这个并不期望的结果。简单的程序没有什么大问题，如果这段代码在大程序里面隐藏的非常深，那debug成本会非常的高。这也就是为什么现在对动态语言进行静态类型检查这么流行，比如著名的TypeScript。

其实Python3从很早开始起，就逐步支持静态类型检查了，比如说，上面的代码可以写成

{% highlight python %}
from typing import List

def join(string_list: List[str]) -> str:
    return ', '.join(string_list)
{% endhighlight %}

这样声明函数有一个好处，就是再也不需要在注释里面说明变量类型了，更加直观。但是这个时候下面的代码依旧可以运行

    >>> join('hello')
    'h, e, l, l, o'

这个是因为Python只是把这种类型的声明看成是一种对函数的注解(annotation)，而注解本身并不具有任何的意义，也不影响运行的过程。与没有注解的版本差别就是多了一个__annotations__的字段

    >>> join.__annotations__
    {'string_list': typing.List[str], 'return': <class 'str'>}

那么静态类型检查怎么实现呢？Python对于注解语法的设计，就是把静态的类型检查与代码的解释运行分开，静态类型检查可以使用第三方库去做， CPython解释器无需做任何事情。就好比TypeScript，静态类型检查以及转换成js是它的工作，与js的解释运行没有关系。

目前，基于注解(annotation)的Python静态类型检查库有很多，比如Guido van Rossum的mypy以及google的pytype。在这里简单介绍一下mypy。mypy安装非常容易，使用pip即可，但是注意这个包的名字叫做mypa-lang而不是mypy，在Ubuntu 16.10下

    $ pip3 install mypy-lang

在安装成功后，对于下面代码test_mypy.py

{% highlight python %}
from typing import List

def join(string_list: List[str]) -> str:
    return ', '.join(string_list)

print(join('hello'))
{% endhighlight %}

运行以及使用mypy检查

    $ python3 test_mypy.py
    h, e, l, l, o
    $ python3 -m mypy test_mypy.py
    test_mypy.py:6: error: Argument 1 to "join" has incompatible type "str"; expected List[str]
    $

就可以找到第六行`print(join('hello'))`的类型错误, 而把test_mypy.py改成

{% highlight python %}
from typing import List

def join(string_list: List[str]) -> str:
    return ', '.join(string_list)

slist = ['hello', 'world']
print(join(slist))
{% endhighlight %}

运行mypy检查

    $ python3 test_mypy.py
    hello, world
    $ python3 -m mypy test_mypy.py
    $

就可以成功的通过。

除了上面对函数的注解，其实Python3的annotation还有非常多的功能，比如对于类的注解，或者泛型，或者在Python 3.6中还有对变量的注解。如果发现Python代码越开越多，调试越来越麻烦的时候，不妨可以试试使用这个。
