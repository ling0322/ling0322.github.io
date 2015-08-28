--- 
layout: post
title: Go语言奇怪的迭代器模式
---

Go语言1.1版本前天终于发布了，据说有性能上30%的提升，其实相对于性能，我更想去吐槽这个的语法。因为咱发现了Go语言1.x中一个严重的**语法缺陷**

Go语言崇尚的哲学是简洁和快速，因此这门语言学习起来非常快，花差不多30min的时间看看[A Tour of Go](http://tour.golang.org/)就可以开始写了，相对于C++各种臃肿和冗余的语法来说，Go语言做的确实不错。

但是为了简洁，Go语言看上去各种奇怪。我最想吐槽的两个东西就是make只能去生成list, map和chan，还有for...range也只能迭代list, map以及chan。

也就是说用户自己没有办法让自己定义的一个迭代器能够用for...range迭代。因此，有洁癖的coder(比如我)就倾向于使用Go Language Patterns中介绍的[Iterators模式](https://sites.google.com/site/gopatterns/object-oriented/iterators)去使用一个chan外加一个goroutine去假装一个迭代器使得它能够用在for...range中。

比如以下代码(暂时叫做程序A吧)，用chan+goroutine迭代1~K然后求和

    package main
    import "fmt"

    var K int = 1000000000
    func iter() <- chan int {
    	ch := make(chan int)
    	go func () {
    		for i := 0; i < K; i++ {
    			ch <- i
    		}
    		close(ch)
    	}()
    	return ch	
    }

    func main() {
        sum := 0
        for i := range iter() {
        	sum += i
        }
        fmt.Println(sum)
    }

但是怎么想为了一个For循环好看一点还要特地去开一个协程开一个管道，这个怎么想怎么不爽啊！因此，通常在Go语言里面也使用Java里面的next/hasNext模式来实现迭代器，比如

    package main

    import "fmt"

    var K int = 1000000000
    var i int = 0

    func hasNext() bool {
        if i < K {
            return true
        } else {
            return false
        }
    }

    func next() int {
        t := i
        i++
        return t 
    }

    func main() {
        sum := 0
        for hasNext() {
            sum += next()
        }
        fmt.Println(sum)
    }

这个就暂时叫做程序B吧。

这边为了代码短一点，使用了一个全局变量。虽然，这种方式在一定程度上可以解决性能的问题，但是！！！！！！！！

设想下面一种情况，一个struct里面有四个map: A, B, C, D，每个map里面大约1000w条数据。需要设计一个迭代器迭代这四个map里面的key-value对，这个时候O(1)的空间复杂度使用next/HasNext怎么解决？！！！！肿么解决！！！！map有next方法么！！！！好吧这个时候还只能使用chan+goroutine。但是你知道这货的性能有多坑么！

咱OSX系统，双核i5 1.8GHz，使用64为go语言1.1正式版

程序A的运行时间是

>real 2m54.522s  
>user 2m54.310s  
>sys 0m0.272s

程序B的运行时间是

>real 0m0.970s  
>user 0m0.908s  
>sys 0m0.049s

对！你没有看错这个goroutine+chan需要2分54秒，比next/hasNext慢了足足180倍！

不过还好，其实还有一种迭代器的解决方案，不过不怎么常用，就是forEach+闭包实现，暂时叫做程序C吧

    package main

    import "fmt"

    var i int = 0
    func forEach(callback func (int))  {
        for i := 0; i < 1000000000; i++ {
            callback(i)
        }
    }

    func main() {
        sum := 0
        forEach(func (i int) {
            sum += i
        })
        fmt.Println(sum)
    }

程序C的运行时间是

>real 0m3.133s  
>user 0m3.074s  
>sys 0m0.047s

虽然使用forEach+闭包可以解决N个map的迭代问题，但是它的速度还是比next/hasNext的模式慢了3倍左右，这个对于数据量大且对性能要求高的程序确实很伤啊（


