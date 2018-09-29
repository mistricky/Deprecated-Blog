# 【golang】不要滥用deferred函数

golang中的defer能够很好的做“善后”工作，defer的执行时机是在赋值给返回值之后，ret之前，使用defer会将函数都push到一个函数调用栈里，然后等待执行时机，以FILO的顺序依次执行。在许多场景中都是非常有用的。

既然defer这么有用，为什么我们不能滥用defer呢？答案是defer也是有开销的。看一段代码：

```
func main(){
	const COUNT = 10000000
	i := 0
	
	start := time.Now()
	for n := 0;n < COUNT;n++ {
		updateI(&i)
	}
	end := time.Since(start)
	fmt.Println("花费的时间为 : " + end.String())

	i = 0
	start = time.Now()
	for n := 0;n < COUNT;n++ {
		updateIDefer(&i)
	}
	end = time.Since(start)
	fmt.Println("defer花费的时间为 : " + end.String())
}
```

输出结果 : 

```
花费的时间为 : 22.175311ms
defer花费的时间为 : 429.882262ms
```

差距是相当明显的，两个函数的主要目的都是为了修改i的值

这篇文章不是让你不要使用defer了，而是不要去滥用它，它固然有用，也是golang不可缺少的，在提升代码的整洁性方面也是非常有用的，但是还是尽量不要在热代码里使用它，避免成为难以发现的性能瓶颈