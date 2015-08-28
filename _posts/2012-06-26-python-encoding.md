--- 
layout: post
title: 使Python 2.x字符串编码的小结
---

如果你 经常遇到这种错误提示的信息: UnicodeEncodeError: ‘ascii’ codec can’t encode characters in position 0-1: ordinal not in range(128), 或者杯具的发现明明在Eclipse中写的程序能够正常运行然后到了终端下面就跳出以上的一段话. 那么, 就证明你和我一样, 遇到了悲催的Python的编码问题了.

之前在用Python语言写我的毕业设计, 然后各种没有问题, 直到整个东西完成了, 突发奇想想去试一下对中文的支持. 然后你懂的, 就弹出了以上一串恶心的错误提示, 然后改半天, 各种改, 各种错误, 然后各种想砸键盘. 其实之前的一篇日志中也说到了, 解决这一类的问题最好的方法就是在程序开头加上以下几行代码:

    import sys
    reload(sys)
    sys.setdefaultencoding(“utf-8″)

那么就可助你解决几乎95%的这种问题, 但是如果想刨根问底的话, 就需要去了解很多东西了.

首先, 这个就是Python语言本身的问题. 因为在Python 2.x的语法中, 默认的str并不是真正意义上我们理解的字符串, 而是一个byte数组, 或者可以理解成一个纯ascii码字符组成的字符串, 与Python 3中的bytes类型的变量对应; 而真正意义上通用的字符串则是unicode类型的变量, 它则与Python 3中的str变量对应. 本来应该用作byte数组的类型, 却被用来做字符串用, 这种看似奇葩的设定是Python 2一直被人诟病的东西, 不过也没有办法, 为了与之前的程序保持兼容.

在Python 2中作为两种字符串类型, str与unicode之间就需要各种转换的方式. 首先是一种显式转换的方式, 就是encode和decode两种方法. 在这里这两货的意思很容易被搞反, 科学的调用方式是:

>str --- decode方法 ---> unicode  
>unicode --- encode方法 ---> str

比如:

    >>> type('x')
    <type 'str'>
    >>> type('x'.decode('utf-8'))
    <type 'unicode'>
    >>> type(u'x'.encode('utf-8'))
    <type 'str'>

这个逻辑是这样的, 对于unicode字符串使用utf-8编码进行编码, 即调用encode('utf-8')方法生成byte数组类型的结果. 相反对于byte数组进行解码, 生成unicode字符串. 这个新手表示理解不能, 不过熟悉了就见怪不怪了.

另外是隐式的转换, 和C语言中的int转double类似, 当一个unicode字符串和一个str字符串进行连接的时候会默认自动将str字符串转换成unicode类型然后再连接. 而这个时候使用的编码方式则是系统所默认的编码方式. 使用:

    import sys
    print sys.getdefaultencoding()

可以得到当前默认的编码方式, 是不是'ascii'? 是的话就恭喜你中彩了~!! 在这个时候如果有以下一行代码就保证会出错:

    >>> x = u'喵'
    >>> x
    u'\u55b5'
    >>> y = x.encode('utf-8')
    >>> x + y

>Traceback (most recent call last):  
>File "<stdin>", line 1, in <module>  
>UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 0: ordinal not in range(128)

x是unicode类型的变量, y是x经过encode后的结果是str类型的变量. x + y的时候, 首先要将y转换成unicode字符串, 那么使用什么编码格式转换呢, 用utf-8还是gb2312或者还是utf-16? 这个时候就要根据sys.getdefaultencoding()来确定, 而sys.getdefaultencoding()是'ascii'编码, 在ascii字符表中不存在0xe5这种大于128的字符存在, 所以当然报错啦! 通过加入

    import sys
    reload(sys)
    sys.setdefaultencoding(“utf-8″)

则可以将默认的编码转换格式变成utf-8, 且大多数情况下, 程序中的字符串是通过utf-8来编码的, 所以只要加上以上三行就可以了.

但是有没有觉得, 加上这些会使得代码有些dirty? 咳, 至少对于我来说确实很dirty. 所以我觉得平时写程序的过程中要养成尽量使用显示的转换的习惯, 并且要明确某个函数返回的到底是str还是unicode, 凡是str的主动decode成unicode, 不要将两者混淆掉, 这样写出来的代码才比较干净. 此外还可以在代码最上方加入'from __future__ import unicode_literals'可以默认将用户自定义字符串变成unicode类型.

最后我想大吼一声Python 2.x中str不是字符串, 而是B♂Y♂T♂E♂数♂组~!!