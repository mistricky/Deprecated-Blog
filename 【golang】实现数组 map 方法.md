# 【golang】实现数组 map 方法

经常会在项目里用到数组的 `map` 方法，闲来无事打算在 `golang` 中实现一下 map 方法。上工具要先上用法

```go
numbers := []int{1, 2, 3}

// convert to []interface{}
interfaceNumbers, err := utils.ToInterface(numbers)

utils.Check(err)

dup := utils.Map(interfaceNumbers, func(item interface{}) interface{} {
		v, ok := item.(int)

		if !ok {
			utils.Check(fmt.Errorf("assert error"))
		}

		return v + 1
})

fmt.Println(dup) // [ 2, 3, 4 ]

```

其实跟其他支持函数式的语言无异，只是多了个类型断言，这是有点麻烦也迫不得已的地方，先别急，实现会放到下面。

首先要捋清思路的是，`map` 的第一个参数是传递目标切片，当然切片可以是任意类型的切片，但是有点头疼的是 `Golang` 中并没有泛型，所以这里只能用 `[]interface{}`，这样导致的结果是传递参数之前是有确切类型声明的，但是出来的时候就是 `[]interface{}` 还得在外面再做一层断言。

先来看一下 `map` 函数的具体实现

```go
func Map(slice  []interface{}, cb func(interface{}) interface{}) []interface{} {
	dup := make([]interface{}, len(slice))

	copy(dup, slice)

	for index, val := range slice  {
		dup[index] = cb(val)
	}

	return dup
}
```

实现比较简单，只是将传递进来的回调在内部执行传递切片的每个项，这里为了避免副作用，在内部重新创建了一个新的切片，通过 `copy` 方法把旧切片的值拷贝到新的切片当中。

重要的是 `map` 的函数签名，传递一个 `slice` 和 `cb` 返回一个 `[]interface{}`，值得关注的是 `slice` 的类型，`slice` 的类型为 `[]interface{}` ，看起来是没有问题的，但是如果你传递一个其他的类型的切片就会抛出 `panic`，这是因为 `golang` 会认为它们是两种不同的类型，尽管 `interface{}` 是可以兼容任何类型，但是 `[]interface{}` 是一种新的类型，它是一种能够容纳任何类型的切片，它并不兼容任何类型的切片，详情戳 

[【golang】传递任意类型的切片]: https://blog.csdn.net/HaoDaWang/article/details/83931629

这里就不再赘述了。

所以我们还要实现一个能够把任意切片转换为 `[]interface{}` 的一个转换函数，思路是利用反射判断类型，然后将它 copy 成一个新的切片并返回

```go
func ToInterface(slice interface{}) ([]interface{}, error) {
	v := reflect.ValueOf(slice)

	if v.Kind() != reflect.Slice {
			return nil, fmt.Errorf("argument must be a slice")
	}

	r := make([]interface{}, v.Len())

	for i := 0; i < v.Len(); i++ {
		r[i] = v.Index(i).Interface()
	}

	return r, nil
}
```

`ToInterface` 利用反射检测类型是否正确，如果不是切片，则抛出一个 error。

