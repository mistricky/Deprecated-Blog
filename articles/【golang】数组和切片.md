# 【golang】数组和切片

在golang中，有一个“罕见”的复合类型，叫切片，切片是基于数组的，golang和其它语言不一样，在golang中，数组是不可变的，对数组进行类型转换等操作都会导致golang隐式的申请一块内存，然后将原数组的内容复制到这块内存。

数组是不可变的，这就决定了字符串也是不可变的，因为字符串底层就是一个byte数组实现的。

在实际的开发当中，我们经常使用的是切片，而不是数组。

### 切片

切片是基于数组的，它表示一个拥有相同类型元素的可变长度的序列

切片是一种轻量级的数据结构，可以用来访问数组的部分或者全部的元素，emmmmm，可以把它看作一个指向数组的某块区域的引用，这块区域可以是数组的部分，也可以是数组的全部。

切片包含长度，容量，指针三个属性，指针指向的是数组的某块区域，容量一般是数组的长度，而长度指的是切片开始的位置到切片结束的位置的长度，而容量是指切片开始的位置到数组结束的位置，看得出来，切片是依赖于数组的

假设现在有这样一个整型数组

```
arr := [...]int{1,2,3,4,5}
```

![这里写图片描述](https://img-blog.csdn.net/20180418091129776?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

现在创建一个基于该数组的切片

```
slice := arr[0:3]
```

![这里写图片描述](https://img-blog.csdn.net/20180418091435588?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

现在的slice的指针指向的就是arr的第一个元素，而它的长度是3，容量是整个数组的长度5

现在来看一下切片的实际应用

比如有时候我们想要频繁的修改一个数组的元素，那么我们从一开始就可以创建一个切片

比如创建一个1, 2, 3, 4, 5的整型的切片

```
slice := []int{1,2,3,4,5}
slice[1] = 0

fmt.Println(slice) //1 0 3 4 5
```

这段切片的声明只比上面的数组声明少了一个…，如果加上省略号，表示可以不写数组长度，而是根据后面的初始化的序列来推断出数组长度，而省略了省略号，就表示声明一个切片

上面说了，切片实际上是指向数组的一个引用，如何证明这一点呢？

```
slice := []int{1,2,3,4,5}
t := slice
t[0] = 5
fmt.Println(slice) // 5 2 3 4 5
```

t指向的实际上是slice指向的数组，t和slice共享一块内存

因此t[0]的更改也会影响到slice

### 数组

前面说了，数组是不可变的，那么对把数组赋值给另外一个变量实际上是对数据的深拷贝

比如

```
arr := [...]int{1,2,3,4,5}
arr2 := arr
arr2[0] = 2
fmt.Println(arr) // 1 2 3 4 5
```

对arr2的更改并不会影响到arr，这就证明了数组确实是不可变的

然而，对数组的类型转换，也会导致go去重新申请一块内存

比如

```
arr := "hello 世界"
str := []rune(arr)
fmt.Println(string(str)) //hello 世界
```

实际上string的底层就是byte数组，现在将它转换为rune数组（在实际开发中，我们经常用这个转换来处理中文字符），这些代码做了几件事情，首先是申请一块内存，存入rune类型的数组，然后将hello 世界的值拷贝到这个数组里面来，然后返回给这个数组的引用

也就是说，现在的str实际上是rune数组的引用

```
arr := "hello 世界"
	
str := []rune(arr)

str2 := str
str2[0] = 'w'
fmt.Println(string(str)) //wello 世界
```

包括函数的实参传递，实际上就是把实参拷贝给形参，当传入一个数组的时候，也是拷贝给了形参，而其它语言可能不一样，比如c，实际上是传递的数组的首地址，相当于传递了一个指向该数组的指针。当传递一个切片的时候，也是把该切片拷贝给形参，但是切片相当于是数组的引用，因此，引用的拷贝，是可以在函数内部修改的

```
func main(){
	arr := [...]int{1,2,3,4,5}
	foo(arr)
	fmt.Println(arr) //1 2 3 4 5
}

func foo(args [5]int){
	args[0] = 5
}
```

```
func main(){
	arr := [...]int{1,2,3,4,5}
	slice := arr[:]
	foo(slice)
	fmt.Println(arr) //5 2 3 4 5
}

func foo(args []int){
	args[0] = 5
}
```

那么总结一句话，函数参数传递的时候，如果传入的是数组，则对其进行深拷贝，如果传递的是切片，则对其进行浅拷贝

### append

切片可以理解为是可变的数组，但是切片的最大长度是不能超过容量的，有什么方法能够扩充容量呢？

golang提供了append方法，用于向切片后面追加元素，同样，数组的两倍容量 > 如果切片的容量 > 数组的容量 ，则原切片扩充到原数组的两倍，这样做是为了分摊线性复杂度。如果切片的容量 < 数组的容量，则只需要将切片长度+ 1即可

```
arr := [...]int{1,2,3,4,5}
slice := arr[:]
slice2 := append(slice, 1)
fmt.Println(cap(slice2)) //10 扩充至原来的两倍
```

下面我们来自己实现一个appendInt方法，用于将一个整型元素追加到int类型切片的后面

```
//往slice后面追加元素
func appendInt(slice []int, value int) []int{
	//需要返回的切片
	var z []int
	zlen := len(slice) + 1

	//判断是否超出slice的容量 
	if zlen <= cap(slice) {
		z = slice[:zlen]
	} else {
		//超出即申请创建一个新的数组，返回该数组的切片
		zcap := zlen
		//分摊线性时间复杂度
		//避免重复的申请空间
		if zcap < len(slice) * 2 {
			zcap = len(slice) * 2
		}
		z = make([]int, zlen, zcap)
	}
	z[len(slice)] = value
	copy(z, slice)
	return z
}
```

现在在测试一下这个appendInt

```
arr := [...]int{1,2,3,4,5}
slice := arr[:5]
slice = appendInt(slice, 1)
fmt.Println(slice, cap(slice)) //[1,2,3,4,5,1] 10
```

容量扩充至原来的两倍。

下次我们append元素的时候，也不用去重新申请空间了，而是和数组共用一块内存

```
arr := [...]int{1,2,3,4,5}
slice := arr[:5]
slice = appendInt(slice, 1)
slice = appendInt(slice, 2)
fmt.Println(slice, cap(slice)) //[1,2,3,4,5,1,2] 10
```

可以看到容量并没有改变

### 切片能不能作为map的键呢

答案是不能，slice是不能比较的，因为slice类似于引用，引用只能比较两个引用是否指向同一块内存，golang并没有提供slice比较方法，如果真的要比较slice，那么可以循环遍历slice，挨个挨个比较元素，但是这样做是不理想的，因为slice依赖于底层数组，数组元素改变，slice也会跟着改变。

