# 【golang】传递任意类型的切片

### 前言

Golang 是一门类型严格的静态语言。看看下面的代码

```
type MyInt = int

var bar MyInt = 2
var foo int = 1
```

虽然上面 foo 和 bar 的底层类型都是一样的，是 int， 但是 bar 和 foo 的类型在语言层面并不一样，因此它们不能互相赋值，这样会引起 panic。

所以这样就出现了一个问题，有时候我们并不想关心传递进来的类型，比如我想实现一个函数叫`traverse`，我想遍历传入进来的`slice`，但是我并不关心传入进来的`slice`是什么类型。哦克，初始版本我可能会这样写

```
func traverse(slice []interface{}) {
	for _, v := range s {
		fmt.Println(v)
	}
}

fucn main(){
	// call
	traverse([]int{1,2,3})
}
```

你会发现在编译期就报错了！这是为什么呢？这是因为 `[]interface{}` 和`[]int` 根本是两个类型，而且并不兼容，但是 `interface{}`不是能兼容所有类型吗？没错！它确实能够兼容所有类型，于是所有类型都相当于实现了 `interface{}`，并且能够赋值给它，但是`[]interface{}`呢？它描述的是一个装载`任何类型的元素`的切片，所以它与`[]int`并不兼容！

好了接下来该进入正题了，如何来解决这个问题

### 传递任意类型的切片

要解决这个问题，我们首先得搞定传递参数这个地方，为了获得一个任意类型切片的参数，我们先把`[]interface{}`改为`interface{}`，但是这样并不安全，因为`interface{}`海纳百川，但是我们想要的东西其实一个任意类型的切片，而不想要切片之外的其他类型。

嚯！现在怎么办呢？好好想想，我们现在拿到的是一个`interface{}`，我们又想知道它的动态类型是啥，于是我们就需要反射来帮助我们啦！

尝试写一个`isSlice`的函数来帮助我们检查该参数是否是一个`Slice`

```
func isSlice(arg interface{}) (val reflect.Value, ok bool) {
	val = reflect.ValueOf(arg)

	if val.Kind() == reflect.Slice {
		ok = true
	}

	return
}
```

当然这样还不够，就算我们知道它是一个切片了，但是我们得给他类型啊，于是采用`interface()`把切片里所有的元素都转换为`interface{}`，目的在于构造一个`[]interface{}`的类型。下面看看代码

```
func CreateAnyTypeSlice(slice interface{}) ([]interface{}, bool){
	val, ok := isSlice(slice)

	if !ok {
		return nil, false
	}

	sliceLen := val.Len()

	out := make([]interface{}, sliceLen)

	for i := 0;i < sliceLen;i++ {
		out[i] = val.Index(i).Interface()
	}

	return out, true
}

func isSlice(arg interface{}) (val reflect.Value, ok bool) {
	val = reflect.ValueOf(arg)

	if val.Kind() == reflect.Slice {
		ok = true
	}

	return
}
```

当我们调用`CreateAnyTypeSlice`的时候，我们就能拿到一个`[]interface{}`的类型的切片了。

万事俱备，在任何需要`[]interface{}`的地方，我们通过调用`CreateAnyTypeSlice`就能传入了！比如上面的`traverse`