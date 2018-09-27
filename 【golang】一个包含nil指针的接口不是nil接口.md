# 【golang】一个包含nil指针的接口不是nil接口

今天拜读go圣经的时候，看到这么一个标题，这是interface的一个坑，作为自己的理解，我直接把它拿过来当作我的这篇博文的标题了

当接口作为类型时，可以帮助我们完成多态，对外隐藏实现等。一个接口的零值就是它的运行时类型和运行时的值都为nil。可是尽管这样也会有一些坑存在。

先看看`go圣经`上的一个例子

```
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        // ...use buf...
    }
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
    // ...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

使用debug来作为是否收集输出的开关，f函数接受一个实现了Writer接口的对象out（在go中不需要显示的实现相应的接口，如果实现了该接口的相应的方法，那么可以认为实现了该接口），通过防御性的判断out是否为nil来避免对空指针取地址。但是你发现，当debug为false时，会发生panic。这是为什么呢？

标题中隐含了两个对象，一个是`包含nil指针的接口`，第二个是`nil接口`。
像go这样的静态类型的语言，类型仅仅是一个编译期的概念，我们需要用类型描述符来提供每个类型的具体信息。

我们可以利用反射来获取运行时类型的相关信息

```
func printDynamicType(any interface{}){
	fmt.Printf("%T\n",any)
}
```

这个函数接受一个任意类型的参数，然后在函数内部打印出该参数的类型

下面进行一个简单的赋值

```
var w io.Writer
var f *os.File
w = f
fmt.Println(w == nil) //false
```

这里为什么是false呢？原因是当File类型的指针赋值给Writer接口时，接口的类型被赋值为File类型的指针，而类型描述符被赋值为nil，而一个接口为nil的充要条件为接口的运行时类型为nil，并且接口的运行时值为nil。这里将运行时的类型赋值为了File类型的指针，因此是false。这显然不符合我们的预期，换种说法，这显然不是我们想要的结果，那我们如何进行更改呢？只需要将f的编译时类型改为io.Writer即可。因为两种类型是相同的，因此`w = f`进行赋值的时候，并不会改变运行时类型，又因为两个的值都是nil，所以运行时的值为nil，这样的话w == nil 就显然为true了