# 【golang】实现一个 html 解析器

今天发现 `golang` 中提供的 `xml` 包可以方便的帮助我们解析标记语言，所以，我们可以很方便的就实现一个 `html` 的解析器。

先来看一下数据结构

```go
type Node interface {}

type Element struct {
	tagName string
	attrs []xml.Attr
	children []Node
}
```

之所以声明 `Node` 是因为 `children` 不止是 `Element` 还可能是 `string`

讲实现之前先看一下用法，先来一个需要解析的 `html` 文件

**index.html**

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0"/>
    <meta http-equiv="X-UA-Compatible" content="ie=edge"/>
    <title>Document</title>
</head>
<body>
    <h1 name="haodawang">Hello World</h1>
</body>
</html>
```

**html-parser.go**

```go
func main(){
  ele, err := h("./index.html")

	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	fmt.Println(ele)
}
```

**output:**

```go
&{html [{{ lang} en}] [
 {head [] [
     {meta [{{ charset} UTF-8}] []} 
     {meta [{{ name} viewport} {{ content} width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0}] []} 
     {meta [{{ http-equiv} X-UA-Compatible} {{ content} ie=edge}] []} 
     {title [] [Document]} 
]} 
 {body [] [
     {h1 [{{ name} haodawang}] [Hello World]} 
]}]}
```

返回的是一个 `Element` 的指针是方便我们可以随时的去改变它

可以看到 `index.html` 已经解析成了 `Element` 的树状结构，大概类似这样

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190503164726561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==,size_16,color_FFFFFF,t_70)

我的解析策略是这样的，声明一个 `Element`指针类型的栈（这里存指针，有两个方面的作用，第一是防止栈的空间随数据的膨胀成正相关，第二是后面涉及到修改元素的 `children`），当遇到 `StartElement`类型的时候就 `push` 进栈，当遇到 `EndElement` 的时候就把它 `pop` 出来，这样就能组成一个完成的标签元素。现在是第二个关键点，我们需要设置 `Element` 的 `children` 让所有 `Element` 都关联成一颗树（如上图），因此，需要声明一个 `currentNode` 的 *Element 类型变量，来保存当前的 `Element`，具体是怎样的呢？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190503165830650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==,size_16,color_FFFFFF,t_70)

第一次解析 `index.html` 的时候，遇到 `html` 标签，于是 `push` 进栈，然后解析到 `meta` 标签，于是又 `push` 进栈，现在栈里面有两个元素，再往下，遇到了 `meta` 的闭合标签，这时候，我们需要将 `meta` 弹出，然后把 `html` 的 children 里 `push` 进 `meta` 元素，并且把 `current` Node 指向 `html`。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190503170350126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==,size_16,color_FFFFFF,t_70)

再往下解析就是一直重复这个步骤

至于标签中的字符串内容，解析到之后直接 `push` 进栈中顶层元素的 `children` 中

下面放上 parser 的全部实现代码

```go
func h(filename string) (*Element, error) {
	file, err := os.Open(filename)

	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	decoder := xml.NewDecoder(file)
	var stack []*Element
	var currentElement *Element

	for {
		token, err := decoder.Token()

		if err == io.EOF {
			break
		} else if err != nil {
			fmt.Fprintln(os.Stderr, err)
			return nil, err
		}

		switch token := token.(type) {
		case xml.StartElement:
			stack = append(stack, &Element{
				token.Name.Local,
				token.Attr,
				[]Node{},
			})


			break
		case xml.EndElement:
			currentNode := stack[len(stack) - 1]
			stack = stack[:len(stack) - 1]

			if len(stack) == 0 {
				break
			}

			preNode := stack[len(stack) - 1]
			preNode.children = append(preNode.children, *currentNode)
			currentElement = preNode

			break
		case xml.CharData:
			if len(stack) == 0 {
				break
			}

			lastNode := stack[len(stack) - 1]
			lastNode.children = append(lastNode.children, string(token[:]))
			break
		}
	}

	return currentElement, nil
}case xml.CharData:
		if len(stack) == 0 {
			break
		}

		lastNode := stack[len(stack) - 1]
		lastNode.children = append(lastNode.children, string(token[:]))
		break
}
```

这里实际上是用到了 `discriminated union`，也就是可辨识联合，通过 `switch` 去断言当前的 `token` 是哪个类型，然后 `dispatch` 相应的处理

其实细追 `Token` 的源码，你会发现它其实就是 `interface{}` 的别名

```go
// A Token is an interface holding one of the token types:
// StartElement, EndElement, CharData, Comment, ProcInst, or Directive.
type Token interface{}
```

这就是实现可辨识联合的基础呀

