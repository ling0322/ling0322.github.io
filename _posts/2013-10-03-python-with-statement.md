--- 
layout: post
title: Python高端、大气、上档次的with语句
---

在说with语句之前，先看看一段简单的代码吧

    lock = threading.Lock()
    ...
    lock.acquire()
    elem = heapq.heappop(heap)
    lock.release()

很简单直观，多个线程共用一个优先级队列的时候，首先先用互斥锁lock.acquire()把优先级队列锁上，然后取元素，再然后lock.release()释放这个锁。非常符合逻辑的一个过程，但是里面隐藏着一个巨大的bug：当heap里面没有元素的时候，会抛出一个IndexError异常，再然后堆栈回滚，再然后lock.release()根本不会执行，这个锁就永远得不到释放，因此就发生了喜闻乐见的死锁问题。这个也是很多大神们讨厌异常的原因。经典Java风格的解决方案就是

    lock = threading.Lock()
    ...
    lock.acquire()
    try:
        elem = heapq.heappop(heap)
    finally:
        lock.release()

这个虽然可以，但是怎么看怎么dirty，和Python优雅、简单的风格出入很大。其实，自从Python2.5开始引入了with语句，一切就变得非常简单：

    lock = threading.Lock()
    ...
    with lock:
        elem = heapq.heappop(heap)

在此无论以何种方式离开with语句的代码块，锁都会被释放。

with语句的设计目的就是为了使得之前需要通过try...finally解决的清理资源问题变得简单、清晰，它的的用法是

    with expression [as variable]:
        with-block

其中expression返回一个叫做「context manager」的对象，然后这个对象被赋给variable（如果有的话）。「context manager」对象有两个方法，分别是__enter__()和__exit__()，很明显一个在进入with-block时调用，一个离开with-block的时候调用。这样的对象不需要自己去实现，在Python标准库里面很多API都是已经实现了「context manager」的两个方法的，最常见的一个例子就是读写文件的open语句。

    with open('1.txt', encoding = 'utf-8') as fp:
        lines = fp.readlines()

无论是正常离开还是因为异常原因离开with语句块，打开的文件资源总是会释放。

接下去讨论一下with语句配合contextlib库的一些比较实用的方法，比如需要同时打开两个文件，一个读一个写，这个时候就可以这样写：

    from contextlib import nested
    ...
    with nested(open('in.txt'), open('out.txt', 'w')) as (fp_in, fp_out):
        ...

这样就可以省掉两个with的语句的嵌套了，另外如果遇到一些还没有支持「context manager」的API呢？比如urllib.request.urlopen()，这个返回的对象因为不是「context manager」，结束的时候还需要自己去调用close方法。类似这种API，contextlib提供了一个叫做closing方法，它会在离开with语句的时候，自动调用对象的close方法，因此urlopen也可以这样写：
    
    from contextlib import nested
    ...
    with closing(urllib.request.urlopen('http://www.yahoo.com')) as f:
        for line in f:
            sys.stdout.write(line)

References:

[PEP 343: The 'with' statement](http://docs.python.org/release/2.5/whatsnew/pep-343.html)