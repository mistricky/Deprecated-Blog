# 【golang】unsafe.SizeOf浅析

博主也是正在学习golang，在学习过程中遇到了SizeOf的问题。我原先以为，golang中的sizeof和c的sizeof差不多，但是当我开始使用的时候，才发现了许多奇怪的问题

```
slice := []int{1,2,3}
fmt.Println(unsafe.Sizeof(slice)) //24
```

上面声明了一个切片，然后打印出sizeof的值为24，但是不管slice里的元素为多少，sizeof返回的数据都是24。

然后在官方文档上找到一句话

>Sizeof takes an expression x of any type and returns the size in bytes of a hypothetical variable v as if v was declared via var v = x. The size does not include any memory possibly referenced by x. For instance, if x is a slice, Sizeof returns the size of the slice descriptor, not the size of the memory referenced by the slice.

最后一句

`if x is a slice, Sizeof returns the size of the slice descriptor, not the size of the memory referenced by the slice.`

如果x为一个切片，sizeof返回的大小是切片的描述符，而不是切片所指向的内存的大小。

那么这里如果换成一个数组呢？而不是一个切片

```
arr := [...]int{1,2,3,4,5}
fmt.Println(unsafe.Sizeof(arr)) //40
arr2 := [...]int{1,2,3,4,5,6}
fmt.Println(unsafe.Sizeof(arr)) //48
```

可以看到sizeof(arr)的值是在随着arr的元素的个数的增加而增加

这是为啥？

**sizeof总是在编译期就进行求值，而不是在运行时，这意味着，sizeof的返回值可以赋值给常量**

在编译期求值，还意味着可以获得数组所占的内存大小，因为数组总是在编译期就指明自己的容量，并且在以后都是不可变的

再来看一个字符串的例子

```
str := "hello"
fmt.Println(unsafe.Sizeof(str)) //16
```

不论字符串的len有多大，sizeof始终返回16，这是为啥，字符串不是也是不可变的吗？实际上字符串类型对应一个结构体，该结构体有两个域，**第一个域是指向该字符串的指针，第二个域是字符串的长度**，每个域占8个字节，但是并不包含指针指向的字符串的内容，这也就是为什么sizeof始终返回的是16

### 总结

unsafe包的官方文档这样说道

`导入unsafe的软件包可能不可移植，并且不受Go 1兼容性指南的保护。`

而Go圣经里这样说

`虽然这几个函数在不安全的unsafe包，但是这几个函数调用并不是真的不安全，特别在需要优化内存空间时它们返回的结果对于理解原生的内存布局很有帮助。`

虽然unsafe这个名字听起来很危险，但是既然go官方提供了这个包，那我们就可以在任何必要的时候，去根据情况取舍