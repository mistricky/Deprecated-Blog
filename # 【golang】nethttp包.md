# 【golang】net/http包

在golang中，使用net/http包可以轻松的创建一个web服务器。

其中，http.listenAndServe是一个很重要的方法，这里说说我踩的一个坑吧。这跟自己的开发背景有关。
listenAndServe接受两个参数，第一个是需要一个字符串形式的服务器地址，比如"localhost:8080"，第二个参数是一个用于分派所有请求的http.Handler接口的实例。

我当时看到这个方法，想着跟node的httpServer.listen很像，第一个是接受一个服务器地址，第二个是一个回调函数，所以这里理所当然的认为第二个参数应该是一个回调函数了。先不管三七二十一试一下再说。

```
type ListenFunc func(res http.ResponseWriter, req *http.Request) 

func (l ListenFunc) ServeHTTP(res http.ResponseWriter, req *http.Request){
	l(res, req)
}

func main(){
	http.HandleFunc("/", func(res http.ResponseWriter, req *http.Request){
        fmt.Fprintln(res,"welcome to my house!!")
	})
	
	listenFunc := ListenFunc(func(res http.ResponseWriter, req *http.Request){
		fmt.Println("用户请求")
	})

	http.ListenAndServe(":8080",listenFunc)
}
```

看上去很没问题。。可是不知道为什么handleFunc打死不触发。为此纠结了很久（看来还是要多看文档）

原因就是把listenAndServe方法的第二个参数当作了一个回调函数。其实它是一个派发所有请求的Handler。知道了这些过后，改一下上面的代码

```
func main(){
	mux := http.NewServeMux()

	mux.HandleFunc("/", func(res http.ResponseWriter, req *http.Request){
        fmt.Fprintln(res,"welcome to my house!!")
	})

	http.ListenAndServe(":8080",mux)
}
```

访问`http://localhost:8080`

welcome to my house!!

mux是Multiplexer的缩写，是一个多路复用器，用于派发所有接收到的请求。我们还能在上面绑定多个路由，以及它的控制器。

http包除了ServeMux对象还有一个默认的DefaultServeMux对象，它的功能和ServeMux一样，后者是一种更加便利的方式，让我们能够使用多路复用器。当ListenAndServe方法的第二个参数传递为nil时，会默认使用DefaultServeMux对象。
比如要使用HandleFunc，直接使用`http.HandleFunc`即可。那么上述的代码可以改为

```
func main(){
	http.HandleFunc("/", func(res http.ResponseWriter, req *http.Request){
        fmt.Fprintln(res,"welcome to my house!!")
	})

	http.ListenAndServe(":8080",nil)
}
```

是不是很简单？