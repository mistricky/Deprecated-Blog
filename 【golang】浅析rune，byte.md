# 【golang】浅析rune，byte

golang内置类型有rune类型和byte类型。

需要知晓的是rune类型的底层类型是int32类型，而byte类型的底层类型是int8类型，这决定了rune能比byte表达更多的数。

在unicode中，一个中文占两个字节，utf-8中一个中文占三个字节，golang默认的编码是utf-8编码，因此默认一个中文占三个字节，但是golang中的字符串底层实际上是一个byte数组。因此可能会出现下面这种奇怪的情况

```
str := "hello 世界"
fmt.Println(len(str)) //12
```

我们期望得到的结果应该是8，原因是golang中的string底层是由一个byte数组实现的，而golang默认的编码是utf-8，因此在这里一个中文字符占3个字节，所以获得的长度是12，想要获得我们想要的结果也很简单，golang中的unicode/utf8包提供了用utf-8获取长度的方法

```
str := "hello 世界"
fmt.Println(utf8.RuneCountInString(str)) //8
```

上面说了byte类型实际上是一个int8类型，int8适合表达ascii编码的字符，而int32可以表达更多的数，可以更容易的处理unicode字符，因此，我们可以通过rune类型来处理unicode字符

```
str := "hello 世界"
str2 := []rune(str)
fmt.Println(len(str2)) //8
```

这里会将申请一块内存，然后将str的内容复制到这块内存，实际上这块内存是一个rune类型的切片，而str2拿到的是一个rune类型的切片的引用，我们可以很容易的证明这是一个引用

```
str := "hello 世界"
str2 := []rune(str)
t := str2
t[0] = 'w'
fmt.Println(string(str2)) //“wello 世界”
```

通过把str2赋值给t，t上改变的数据，实际上是改变的是t指向的rune切片，因此，str也会跟着改变

### 字符串的遍历

对于字符串，看一下如何遍历吧，也许你会觉得遍历轻而易举，然而刚接触golang的时候，如果这样遍历字符串，那么将是非常糟糕的

```
str := "hello 世界"
	
for i := 0;i < len(str);i++ {
	fmt.Println(string(str[i]))
}
```

输出:

```
h
e
l
l
o
 
ä
¸
ç
```

如何解决这个问题呢？

第一个解决方法是用range循环

```
str := "hello 世界"
	
for _,v := range str {
	fmt.Println(string(v))
}
```

输出	

```
h
e
l
l
o
 
界
```

原因是range会隐式的unicode解码

第二个方法是将str 转换为rune类型的切片，这个方法上面已经说过了，这里就不再赘述了

当然还有很多方法，其本质都是将byte向rune上靠

### rune和byte的区别

除开rune和byte底层的类型的区别，在使用上，rune能处理一切的字符，而byte仅仅局限在ascii



