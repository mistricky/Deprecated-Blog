# Golang 可变参数

很多语言都提供了这个特性，这里不再赘述，但是众所周知的是 `Golang` 没有可选参数！是的，因为 `rest` 和 可选参数有的时候是可以替代的，但是 rest 还是有很多的坑。

Golang 中的 `rest` 参数传递后会隐式的在内部创建一个新的切片，所以你可以在内部无所顾及的使用 `range` 或者直接改变它的元素的值

```go
func rest(args ...int){
  for index, val := range args {
		    args[index] = index;
  }
}
```

然鹅 `Golang` 也会允许你传递一个切片进来，这就导致了问题，问题是，`Golang` 不会再帮助你在内部去创建一个新的切片！

```go
func foo(args ...string){
	args[0] = "a"
}

func main(){
	names := []string{"Jon", "Henry", "Tom"}
	
  foo(names...)

	fmt.Println(names) // [a, Henry, Tom]
}
```

有的时候我们必须尽量的避免这种副作用，解决的方式有很多种，可以在函数内部去创建一个副本，也可以在传递参数的时候就去做处理

```go
func foo(args ...string){
	args[0] = "a"
}

func main(){
	names := []string{"Jon", "Henry", "Tom"}
	
  foo(append([]string{}, names...)...)

	fmt.Println(names) // [Jon, Henry, Tom]
}
```

这样实际上是创建了一个新的切片并且把原先切片里的元素都装入了新的切片，但其实这里切片是开辟了一段新的内存，然后存放东西。所以还是创建了一个副本。

## 关于 append

有的时候我们可能过于相信 `append` 是没有副作用的，因为经常会看到这样的写法

```go
slice = append(slice, "a")
```

它会赋值给原来的对象，而不是副作用的添加元素，但实际上它有的时候是没有副作用的，在 Golang 的 `append` 实现中，append 会去判断当前增加的元素是否超出 slice 的容量，如果超出了会去申请一块比当前容量大一倍的内存，然后将现有的元素拷贝进新的内存。这里涉及到 slice 的底层数组是不唯一的，也是不可预料的，所以才会每次都赋值给原有的切片，但是并不是说 `append` 就一定是没有副作用的或者说 `append` 是一定有副作用的。

还是刚刚那个例子

```go
func foo(args ...string){
	args = append(args, "b")
}

func main(){
	names := []string{"Jon", "Henry", "Tom"}

	foo(names...)

	fmt.Println(names[3]) // ["Jon", "Henry", "Tom"]
}
```

由此看出 `names` 并没有得到改变，但是 `append` 确实改变了 `names` 的底层数组，并且在函数内赋值给了 `names` ，没有改变的原因是因为 `Golang` 中没有引用传递，传递进来的切片也是拷贝一份过后才传递进来的，至于上面为什么通过下标能直接改变，你可以把传递进来的切片就看做一个只包含底层数组的指针，cap，len 的一个结构体，所以这里自然是不能被访问到的