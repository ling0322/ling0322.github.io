--- 
layout: post
title: rust vs go, 强大的代码优化能力
---

半年前我吐槽过go语言坑爹的迭代器模式和根本没有的闭包优化。这一次我想继续去吐槽一下rust语言，结果发现rust有着llvm的支持，竟然效果惊人的好。

先拿go语言做一个对照组吧，很简单，从1到1000000000求和，一个for循环解决的代码

{% highlight go %}
func main() {
  sum := 0
  for i := 0; i < 1000000000; i++ {
    sum += i
  }
  
  fmt.Println(sum)
}
{% endhighlight %}

使用go build编译，执行的时间是

    real 0m0.691s
    user 0m0.687s
    sys 0m0.004s

把它写成闭包的形式

{% highlight go %}
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
{% endhighlight %}

使用go build编译一下，运行的时间是

    real 0m2.756s
    user 0m2.747s
    sys 0m0.010s

说明go语言根本对闭包的优化不感兴趣，这个时候看看rust语言的效果吧

{% highlight rust %}
fn for_each(f: |int| -> ()) {
  let mut i = 0;
  while i < 1000000000 {
    f(i);
    i += 1;
  }
}

fn main() {
  let mut sum = 0;
  for_each(|x| sum += x);
  println!("{}", sum);
}
{% endhighlight %}

使用rust -O编译，运行的时间竟然是

    real 0m0.008s
    user 0m0.002s
    sys 0m0.005s

这个是要逆天么！！！然后我用rustc -O -S查看汇编代码竟然发现了这一行

    movabsq $499999999500000000, %rax
    
竟然在编译时期就计算好了！！！！！！！！！！于是就打算改一下代码从stdin读入1000000000，而不是在代码里面hard code这样应该不会优化了吧，但是...

{% highlight rust %}
use std::io::BufferedReader;
use std::io;

fn for_each(f: |int| -> (), end: int) {
  let mut i = 0;
  while i < end {
    f(i);
    i += 1;
  }
}

fn main() {
  let mut reader = BufferedReader::new(io::stdin());
  let input = reader.read_line().unwrap();
  let num_str = input.trim();
  let num = from_str::<int>(num_str).unwrap();

  let mut sum = 0;
  for_each(|x| sum += x, num);
  println!("{}", sum);
}
{% endhighlight %}

这个使用rustc -O编译号之后，因为是从stdin读入一个数，所以使用‘time echo 1000000000 | ./3’运行，结果还是

    real 0m0.005s
    user 0m0.002s
    sys 0m0.004s
    
这个是什么，看了半天汇编代码竟然发现它已经直接优化成了end * (end - 1) / 2！！！！！后来把'i += 1;'换成'i += 2;'之后总算正常了，毕竟编译器不是AI，不会这么聪明。因为每次都是加2，所以运行的时候为了确保循环次数一样要用‘time echo 2000000000 | ./3’运行，

    real 0m0.714s
    user 0m0.708s
    sys 0m0.007s

这个rust使用闭包的运行时间和go语言没有使用闭包的运行时间差不过，后来我用rust写了一个没有闭包版本的，发现运行时间和有闭包版本的一样，证明rust确实做了闭包的优化，而且效果不错。并且rust语言依赖于llvm后端，因此代码优化能力不会差到哪里，但是像刚才情况的优化，我还确实有点小惊讶。

期待rust 1.0早日到来:)