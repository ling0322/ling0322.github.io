--- 
layout: post
title: Rust有哪些令人印象深刻的特性？
---

Rust和Go语言类似，是最近兴起的编程语言之一，它继承了很多函数式编程的特性，被认为是将来C++语言的重要竞争对手。那么Rust到底有哪些令人印象深刻的特性呢？下面会有简单的介绍。

## 安全 ##

Rust的特性之一就是它的安全。在不影响性能的情况下，Rust语言实现在编译期间的安全检查。确保所有的变量及其引用的每一块内存，都是安全有效的。

比如以下这段会造成问题的C++代码：

{% highlight c++ %}
const char *foo = "foo";
{
  std::string bar = "bar";
  foo = bar.c_str();
}
puts(foo);
{% endhighlight %}

因为bar的作用域仅仅是在花括号中，当花括号这个作用域结束后bar也会被销毁。因此在puts调用的时候，foo指向的内存实际上是无效的。因为C++不会对指针lifetime做任何的检查，所以这也是C++常见的，但也是最难被发现的一类错误。

对于Rust来说，它所有的引用类型都会检查它的作用域以及生命周期(lifetime)。因此，这类错误可以在编译期间被检查出来，对于以下Rust语言代码

{% highlight rust %}
let mut foo: &str = "foo";
{
  let bar = String::from("bar");
  foo = bar.as_str();
}
println!("{}", foo);
{% endhighlight %}

Rust编译时就会报错

    error: `bar` does not live long enough

bar.as_str()返回值的生命周期与bar一致，bar生命周期在{}作用域结束的时候就结束了，它的返回值就不能赋值给生命周期更长的foo。以上这些，编译器可以检查和推断得到。

编译期间的静态引用变量生命周期检查，加上Box、Rc等智能指针可以有效的管理内存的分配以及销毁，从而避免复杂且开销大的GC.

## 迭代器以及链式的map/filter/fold ##

Rust语言提供了迭代器类型及其基础上的链式map/filter/fold连写。比如，1到100数组中所有偶数的数平方和，Go语言需要要这样写

{% highlight go %}
bar := 0
for i := 1; i < 100; i++ {
  if i % 2 == 0 {
    bar += i * i
  }
}
{% endhighlight %}

而对于Rust来说只需要一行即可解决

{% highlight rust %}
let bar = (1..100).filter(|&n| n % 2 == 0).map(|n| n * n).fold(0, |f, n| f + n);
{% endhighlight %}

首先(1..100)创建一个1到100的迭代器。注意，只是创建一个迭代器，并不创建数组。随后filter函数筛选出其中的素数，之后由map函数进行平方操作，最后由fold进行求和。所进行的每一步操作都有直观、清晰的定义。

其中的`|n| n * n`是lambda表达式，参数为n，返回`n * n`的值，其中n的类型由编译器自己推导，不需要在代码中显式定义。

在计算过程中，直到最终结果出来，没有任何作为中间结构的数组生成。fold中每个值仅当需要的时候才会去计算得到。加上内嵌lambda表达式的优化，上述代码的运行效率与for相差无几。这就是函数式编程中惰性求值(lazy evaluation)的概念。

## 泛型 ##

Rust的泛型的声明比较简单，比如泛型函数

{% highlight rust %}
fn takes_anything<T>(x: T) {
}
{% endhighlight %}

泛型的结构体

{% highlight rust %}
struct Point<T> {
    x: T,
    y: T,
}
let int_origin = Point { x: 0, y: 0 };
let float_origin = Point { x: 0.0, y: 0.0 };
{% endhighlight %}

其中泛型的实现，不同于Java的语法糖的方式，而更接近于C++模版的实现。并且在C++模版的基础上，加上了编译期间的检查，如

{% highlight rust %}
fn foo<T>(bar: &T) -> String {
  bar.to_string()
}
{% endhighlight %}

编译就会产生错误

    error: no method named `to_string` found for type `&T` in the current scope
    
因为此时Rust编译器无法知道&T是否能够调用to_string方法，需要在模版中定义中告诉编译器&T可以调用to_string方法

{% highlight rust %}
trait ToString {
  fn to_string(&self) -> String;
}
fn foo<T: ToString>(bar: &T) -> String {
  bar.to_string()
}
{% endhighlight %}

在上面的代码中首先声明的是Trait。Trait类似于Java中interface的概念，描述了实现此Trait的类型需要的方法。

然后在声明中加上`foo<T: ToString>`，告诉编译器T必须实现ToString这个Trait。只有这样，在后面`bar.to_string()`调用to_string的时候编译才会通过。

泛型也是Rust中多态的概念：Rust的多态分成两类，第一类是类似于上面的模版方式在编译期间静态分派(static dispatch)，第二类是类似于OOP语言中虚函数实现的动态分派(dynamic dispatch)。 

其中，foo方法也可以写成动态分派的形式

{% highlight rust %}
fn foo(bar: &ToString) -> String {
  bar.to_string()
}
{% endhighlight %}

以上就与Java中接口实现的多态基本相同。

P.S.对于模版的问题最近在C++17同样也中引入了Concepts-lite语法。

## 模式匹配 ##

与Scala语言一样Rust语言提供了非常强大的模式匹配功能，比如说

{% highlight rust %}
let x = 5;
match x {
    1 => println!("one"),
    2 => println!("two"),
    3 ... 5 => println!("three"),
    _ => println!("something else"),
}
{% endhighlight %}

用来处理x在不同取值或者范围中的情况，其中`_`代表默认的情况。不过，这个并不是模式匹配最主要的用途。match语句还可以与枚举类型相结合。

与Scala一样，Rust语言中的枚举类型(enum)除了可以定义常量外，还可以对其绑定变量。比如说Rust中作为返回值用的比较多的Result就是一个枚举类型

{% highlight rust %}
enum Result<T, E> {
   Ok(T),
   Err(E)
}
{% endhighlight %}

Ok和Err就是枚举类型Result的成员，T和E分别是Ok和Err绑定变量的类型。函数在成功的时候返回带类型T的Ok成员，错误的时候返回带类型E的Err成员。调用后可使用match语句处理各种情况，如

{% highlight rust %}
let mut buffer = String::new();
let result = io::stdin().read_line(&mut buffer);
match result {
    Ok(bytes) => println!("{} bytes read.", bytes),
    Err(_) => panic!("boom"),
}
{% endhighlight %}

read_line函数从stdin中读取一行字符串，返回一个Result<usize, Error>变量。然后在match语句中对于不同的情况进行处理，如果成功就会匹配Ok然后bytes为读取的字符数量；错误则匹配到Err，其中e为Error类型的错误描述。

错误处理有点复杂？没关系，rust语言早就考虑到这个问题，上述代码可写成

{% highlight rust %}
let bytes_read = io::stdin().read_line(&mut buffer).ok().unwrap();
{% endhighlight %}

`.ok()`将Result转变成Option类型，`.unwrap()`去取值。当错误发生，也就是返回Err(e)的时候unwrap就会抛panic，是不是相当简洁？

此外更佳复杂的错误处理还可以去参看`try!`宏，这边就不深入探讨了。

P.S. C++标准委员会正在商讨C++17中引入模式匹配的功能
