# 【golang】Go实现set类型

### 如何实现set

Go中是不提供Set类型的，Set是一个集合，其本质就是一个List，只是List里的元素不能重复。

Go提供了map类型，但是我们知道，map类型的key是不能重复的，因此，我们可以利用这一点，来实现一个set。那value呢？value我们可以用一个常量来代替，比如一个空结构体，实际上空结构体不占任何内存，使用空结构体，能够帮我们节省内存空间，提高性能

下面看看两种结构体的声明方法

```
type Empty struct { }

func main(){
	empty := new(Empty)
	fmt.Println(unsafe.Sizeof(empty)) //8
}
```

这种形式的声明会返回一个指向该结构体的指针

而下面这种结构体的声明就是一个随处可用的空缓存

```
var empty Empty
fmt.Println(unsafe.Sizeof(empty)) //0
```

### 构造一个Set

构造一个set，首先定义set的类型

```
//set类型
type Set struct {
	m map[int]Empty
}
```

为一个结构体类型，内部一个成员为一个map，这也是主要我们存储值的容器

产生set的工厂

```
//返回一个set
func SetFactory() *Set{
	return &Set{
		m:map[int]Empty{},
	}
}
```

这里初始化一个set，内部的map置空

现在给该set类型添加几个方法，分别为

- Add  添加元素
- Remove 删除元素
- Len 获取set长度
- Clear 清空set
- Traverse 遍历set
- SortTraverse 有顺序的遍历Set

由于map自身的特性，在golang中它是由一个hash表做支持的，每个hash函数都会导致不同的遍历顺序，因此，golang要求程序不依赖于具体的hash函数实现，因此，每次遍历map都会有不一样的顺序，然而，对于set来说，可能会要求提供一种有顺序的遍历。因此，这里提供一个有顺序的遍历方法

下面是具体的实现

```
//添加元素
func (s *Set) Add(val int) {
	s.m[val] = empty
}

//删除元素
func (s *Set) Remove(val int) {
	delete(s.m, val)
}

//获取长度
func (s *Set) Len() int {
	return len(s.m)
}

//清空set
func (s *Set) Clear() {
	s.m = make(map[int]Empty)
}

//遍历set
func (s *Set) Traverse(){
	for v := range s.m {
		fmt.Println(v)
	}
}

//排序输出 
func (s *Set) SortTraverse(){
	vals := make([]int, 0, s.Len())

	for v := range s.m {
		vals = append(vals, v)
	}

	//排序
	sort.Ints(vals)

	for _, v := range vals {
		fmt.Println(v)
	}
}
```

大家可以挨着挨着的打印测试，我这里就不再赘述了