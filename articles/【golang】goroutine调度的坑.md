# 【golang】goroutine调度的坑

今天说说我遇到的一个小坑， 关于goroutine 调度的问题。

关于goroutine的调度，网上资料已经一大堆了，这里就不再赘述了。还是简单的说一下我理解的goroutine的调度。goroutine是语言层面的，它和内核线程是M:N的关系，并且用了分段栈，是相当轻量了。如果设置runtime.GOMAXPROCS为1，那么会有一个上下文G，在G上会有一个对应的内核线程M，内核线程M上可以对应很多个goroutine记作G，每个上下文都会有一个队列称作runqueue，在用go关键字开启一个goroutine的时候，该goroutine就会被装入runqueue中，然后被M用来执行，如果刚好有两个goroutine在队列里，先执行的goroutine因为执行一些耗时操作（系统调用，读写 channel，gosched 主动放弃，网络IO）会被挂起（扔到全局runqueue），然后调度后面的goroutine。好，重点在这里，看一下下面的一段代码

```
func main(){
	runtime.GOMAXPROCS(1)

	waitGroup.Add(1)
	go func(){
		defer waitGroup.Done()
		for i := 0;i < 20;i++ {
			fmt.Println("hello")
			f, _ := os.Open("./data")
			f.Write([]byte("hello"))
		}
	}()

	waitGroup.Add(1)
	go func(){
		defer waitGroup.Done()
		for {
		}
	}()

	waitGroup.Wait()
}
```

这段代码你运行，你会发现，永远都会被阻塞住，hello永远都打印不出来

好，这里出现了两个问题。
1.为什么死循环的goroutine总是先运行？按理说不应该是随机的吗？
2.为什么死循环的goroutine会阻塞而没有被挂起？

先看第二个问题。这里的话，我当时也很苦恼，于是在网上发了问题，得到的回复是，死循环不属于上述任何一种需要被挂起的状态，于是死循环的goroutine会一直运行，想象一个高并发的场景，如果其中一个goroutine因为某种原因陷入死循环了，当前执行这个goroutine的OS thread基本就会被一直执行这个goroutine，直到程序结束，这简直是一场灾难。但是，1.12 会修正这个小问题。我们还是默默的等待新版本发布吧。

再看第一个问题。为什么死循环的goroutine总是先运行？按理说不应该是随机的吗？测试过很多次，都是第二个goroutine先运行。嗯，其实就算是第二个goroutine先运行也是具有随机性的，这关于golang的编译器如何去实现随机。看一下大佬的回答 : 

<不是说测试很多遍它就会一直这样，语言规范没有说必须是这个顺序，那编译器怎么实现都可以，因为都不违反规范。所以你要把它看作是随机的，不能依赖这种未确定的行为，不然很可能新版的编译器就会破坏你依赖的事实。有些项目不敢升级编译器版本，就是因为依赖了特定版本的编译器的行为，一升级就坏了。不是你自己测试很多遍你就能依赖它，编译器、操作系统、硬件等等不同，都有可能出现不同的结果。可以依赖的只有语言规范（ <https://golang.org/ref/spec> ），编译器实现者是一定会遵守的。

到这里也算是解决了上述的两个问题了。来看一下另外一个版本

```
func main(){
	
	runtime.GOMAXPROCS(1)

	waitGroup.Add(1)
	go func(){
		defer waitGroup.Done()
		for {
		}
	}()
	
	waitGroup.Add(1)
	go func(){
		defer waitGroup.Done()
		for i := 0;i < 20;i++ {
			fmt.Println("hello")
			f, _ := os.Open("./data")
			f.Write([]byte("hello"))
			http.Get("http://www.baidu.com")
			fmt.Println("request successful")
		}
	}()

	waitGroup.Wait()
}
```

执行结果是，会先打印一个hello，然后陷入死循环，这也是说明了goroutine在遇到耗时操作或者系统调用的时候，后面的代码都不会执行了（request successful 没有被打印），会被抛到全局runqueue里去，然后执行runqueue中等待的goroutine

希望能够帮助和我一样正在学习golang的友军们更好的理解goroutine的调度问题