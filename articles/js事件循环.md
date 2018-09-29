# 深入理解Javascript-JS异步和事件循环

![这里写图片描述](//img-blog.csdn.net/20180318154800603?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 前言

想想接触js已经很久了，也没有写过一篇关于js异步的文章。于是今天准备写一写js的异步，希望对想了解的同行们有所帮助

首先需要说明的是这里讨论的是浏览器里的js，而不是服务端上的js，比如node。
node和浏览器中的js的异步还是多少有些不同的，如果想了解node中的异步或者事件循环，可以参考我的另一篇文章[Node 事件循环]()

### 正文

### Sync

众所周知的js是一种单线程的语言，单线程，如果说它只能一次做一件事情，那也是再适合不过了。比如我们去请求一个远程的资源，这里我们是同步请求，假设有一个find函数作为拉取远程资源的函数，它接收一个url作为参数，假设从建立连接到拉取成功需要10s

```
let src = find('http://xxx/xxx/x/x/x');
//wait 10s
console.log(src)
```

上面已经说了js是单线程的（虽然现在有技术能让js运用多线程技术来提高计算能力，比如web worker，但是需要注意的是，web worker还是浏览器创建的线程，根本来说，js是单线程是不会变的），js运行在各大浏览器的解释器里，在chrome里，js运行在v8解释器里（得益于v8，js才能从前端领域脱离，在后端领域也颇具影响力）。

来看看上述代码的执行过程。

![这里写图片描述](//img-blog.csdn.net/20180318154737894?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

当前栈里只有一个js程序的入口。

接下来是find函数执行

![这里写图片描述](//img-blog.csdn.net/20180318155232178?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这里会进行阻塞，它不会立即弹出栈，等待10s后才会弹出栈。
最后是打印src

![这里写图片描述](//img-blog.csdn.net/20180318155542946?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

打印完毕后弹出
最后弹出main（）

整个程序执行结束。

这里会发现一个致命的问题，由于js是单线程的，拉取网络资源会在这里阻塞，阻塞的这段时间，js什么事情也不能做，只能在这里等待，等待资源拉取完毕，如果是我，上一个网站，我需要获取一些资源，按钮点击后10s内什么事情都干不了，我会马上离开。。谁不是呢？对吧。这是一种极差的体验。为了弥补这个缺陷，就要介绍一下js里的异步了。

### ASync

异步的js充斥着我们的代码，比如经常使用的setTimeout，setInterval等，还有经常写的AJAX，这些都是异步的。通俗的讲，比如我需要吃午饭，但是又不想出门，于是我打电话给商家订外卖，等外卖到的时候，外卖小哥会通知我去取外卖，但是等待的这段时间内，我可以做任何我想做的事情，比如我可以睡一觉，看会儿电视，玩会儿手机都行，这就是异步！我只需发出我订外卖的这条消息，而不必一直阻塞等待外卖小哥打电话给我，如果是同步的话，在现实生活中看来就很傻了，如果是同步，那我打电话订完外卖之后，那我什么事情都做不了，只能在那里像个呆子一样的傻等，等待外卖小哥打电话给我（终于不用傻啦吧唧的等待了）于是下楼取外卖。

回到js，如果是异步，那些异步操作不会阻塞，而是会马上返回，然后等待执行完毕，通过你传递的callback的引用，调用callback，进行数据处理。

比如上面的find函数如果是异步的，可以写成下面这样:

```
asyncFind('http://xxx/xxx/x/x/x',(err, data) => {
    if(err) throw err;
    console.log(data)
})
```

回调函数一般会接受两个参数，第二个参数是执行完毕后返回的资源，第一个参数是错误对象。假如此次拉取过程中有错误发生，会创建一个错误对象传入回调的第一个参数。

如果是同步的话，捕获错误只能用try catch来进行捕获了

```
try{
    let src = find('http://xxx/xxx/x/x/x');
    console.log(src)
}
catch(e){
    throw e;
}
```

现在回到异步的状态。我们看一下异步是如何战胜同步的（虽然有些时候，我们需要同步的状态），在asyncFind执行后，会立即返回，那么这样，当然不会阻塞。比如下面的代码

```
asyncFind('http://xxx/xxx/x/x/x',(err, data) => {
    if(err) throw err;
    console.log(data)
})

console.log("hello man");
```

这样会先打印"hello man"，再打印data，因为asyncFind立即返回了，所以console.log('hello man')才有机会执行，如果是同步，那你只能在那傻等个10s才会打印hello man了。

现在我们来探讨一下js内部是如何支撑异步机制的。

### JS的异步机制

js内部维护一个回调队列，回调队列用于装入回调函数，js通过调用异步的API进行异步操作，比如setTimeout等等等，通过调用的异步操作不会阻塞js线程，而是在异步操作完成后把callback加入回调队列，事件循环会检查执行栈是否为空，如果为空，则检查回调队列中是否有回调函数，如果有，就把它压入执行栈中进行执行。这里需要明白的是，浏览器不会专门开个线程去维护事件循环，而是一个js线程只能有一个事件循环，事件循环被挂载在js的线程上。

假如现在有这么一段代码

```
setTimeout(() => console.log('hello man'),5000);
```

都知道这段代码会在5s后打印一句hello man，下面来看一张梗概图

![这里写图片描述](//img-blog.csdn.net/20180318203036551?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图可以看到，首先是setTimeout会被压入栈，setTimeout会去调用web API，由于它是一个异步操作，因此压入执行栈后，马上就会被弹出，并返回一个timer id，随之，5秒过后，setTimeout的回调函数会进入callback queue也就是回调队列，这时，执行栈空闲，通过事件循环把callback queue的首个callback压入执行栈进行执行，因此我们可以看到5秒后打印出了hello man，最后把main弹出，执行完毕。

如果有多个异步操作呢?

```
function asyncSomething(arr, callback){
	arr.forEach((item, index, arr) => setTimeout(() => callback(item, index, arr),0))    
}

asyncSomething([1,2,3,4,5,6], item => console.log(item));
console.log('async begin');
```

打印出来的结果应该是，先打印async begin，再依次打印数组里的每个元素

我们再结合上面的图看一下内部的是怎样执行的。

（画图太累\_(:з」∠)\_，我就光说吧，有什么问题，在评论区留下就行）

1. 首先压入执行栈的是main
2. 随后压入asyncSomething执行
3. 然后压入arr.forEach…执行
4. arr.forEach遍历所有数组元素，6个回调函数被依次进入回调队列
5. 弹出arr.forEach
6. 弹出asyncSomething
7. 压入console.log('async begin')执行
8. 弹出console.log('async begin')
9. 此时执行栈为空闲状态，事件循环依次取出回调队列的回调函数，压入执行栈进行执行。
10. 弹出main，执行完毕

打印结果:

```
async begin
1
2
3
4
5
6
```

### 对浏览器渲染性能的影响

我们通常会做的操作是用AJAX从后端拉取资源，后端再捕获相关的路由，调用相关的控制器，封装数据模型返回给我们，然后我们再把它更新到当前页面上去，这时，浏览器会令渲染线程去根据相关数据重绘页面

这会给我们的应用带来什么样的性能影响呢？首先，AJAX是异步的，必然会有回调函数进入回调队列，如果当时有很多个回调函数在前面，那AJAX的回调函数就会很慢才会执行。来看一个比较极端的例子:

定义一个函数:Render，作用为每500毫秒更新一次页面

```
function render(){
    let index = 0;
    setInterval(() => {
        flag.innerHTML = index++;
    },500);
}
```

然后还是刚刚那个做异步操作的函数

```
//异步操作
function asyncSomeThing(arr, cb){
     arr.forEach((item, index, arr) => {
         setTimeout(cb(item, index, arr),0);
     },0)
}
```

定义一个耗时的操作，这个操作需要1s才能完成

```
//耗时的操作
function delay(){
    let nowTime = Date.now();
    while(Date.now() - nowTime < 1000){}
}
```

Ok!执行！

```
render();
setTimeout(() => asyncSomeThing([1,2,4,5,6,7,8,9,2,3],item => {delay() || console.log(item)}),5000);
```

加上html整体的代码形如下面这样:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1 id="flag"></h1>
    <script>
        let flag = document.querySelector("#flag");
        
        //耗时的操作
        function delay(){
            let nowTime = Date.now();
            while(Date.now() - nowTime < 1000){}
        }
        
        //渲染函数
        function render(){
            let index = 0;
            setInterval(() => {
                flag.innerHTML = index++;
            },500);
        }
        
        //异步操作
        function asyncSomeThing(arr, cb){
            arr.forEach((item, index, arr) => {
                setTimeout(cb(item, index, arr),0);
            },0)
        }
        
        render();
        setTimeout(() => asyncSomeThing([1,2,4,5,6,7,8,9,2,3],item => {delay() || console.log(item)}),5000);
    </script>
</body>
</html>
```

可以看到当index=8的时候，会有明显的阻塞。

![这里写图片描述](//img-blog.csdn.net/20180318210311533?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这是因为当第五秒执行asyncSomething的时候，后面的渲染操作已经被排到了这些回调的后面，必须等待每个回调函数一个一个，压入执行栈后执行完毕后才能执行，为什么它们会进入同一个队列我们后面会讨论。

现在我们知道，太多耗时的回调函数确实会对性能产生严重的影响。

### 事件循环

上面大致的谈论了一下js的异步机制，实际上js还有一个重要的成员没有介绍，那就是事件循环。

首先需要涉及到两个概念，一个是macrotask，一个是microtask。别急，这两个概念我们待会儿就会讲到。

macrotask：包括timeout和interval等都属于macrotask

microtask:  promise等都属于microtask

当事件循环从回调队列中拉取回调函数的时候，会对拉取的回调函数进行判断，把他们归为macrotask，和miscrotask两类。一般是先执行完macrotask中的回调函数后，再执行microtask中的回调函数

来看一个例子

```
console.log('script1')

        const interval = setInterval(() => {  
            console.log('setInterval')
        }, 0)

        setTimeout(() => {  
            console.log('setTimeout 1')
            Promise.resolve()
                .then(() => {
                    console.log('promise 3')
                })
                .then(() => {
                    console.log('promise 4')
                })
                .then(() => {
                    setTimeout(() => {
                        console.log('setTimeout 2')
                        Promise.resolve()
                            .then(() => {
                                console.log('promise 5')
                            })
                            .then(() => {
                                console.log('promise 6')
                            })
                            .then(() => {
                                clearInterval(interval)
                            })
                    }, 0)
                })
        }, 0)

        Promise.resolve()
            .then(() => {  
                console.log('promise 1')
            })
            .then(() => {
                console.log('promise 2')
            })

        console.log('script2')
```

这段代码如何来判断打印的顺序呢？

（整个代码段也属于一个macrotask）第一次执行下来

```
script1
script2
```

此时的macrotask和microtask中的情况如下:

| Macrotask                  |                            |      |      |
| :------------------------- | -------------------------- | ---- | ---- |
| console.log("setInterval") | console.log("setTimeout1") |      |      |

| Microtask               |                         |      |      |
| ----------------------- | ----------------------- | ---- | ---- |
| console.log('promise1') | console.log('promise2') |      |      |

接着取出microtask中的所有的回调函数执行

```
promise1
promise2
```

再取出macrotask中的回调函数执行

```
setInterval
setTimeout1
```

执行timeout的时候，里面还有promise，因此执行完毕后，macrotask，和microtask中的情况如下

| Macrotask                  |      |      |      |
| -------------------------- | ---- | ---- | ---- |
| console.log("setInterval") |      |      |      |

| Microtask               |                         |                    |      |
| ----------------------- | ----------------------- | ------------------ | ---- |
| console.log("promise3") | console.log("promise4") | () => {setTimeout} |      |

执行Microtask中的回调函数

```
promise3
promise4
```

Promise的第三个then中的setTimeout进入macrotask，此时macrotask，和microtask中的情况如下

| Macrotask                  |            |      |      |
| -------------------------- | ---------- | ---- | ---- |
| console.log("setInterval") | setTimeout |      |      |

执行Macrotask中的回调函数

```
setInterval
setTimeout2
```

timeout中还包含一个promise，其中promise进入microtask中

| Microtask               |                         |               |      |
| ----------------------- | ----------------------- | ------------- | ---- |
| console.log("promise5") | console.log("promise6") | clearInterval |      |

再执行Microtask中的回调函数

```
promise5
promise6
```

综上，执行完后打印出:

![这里写图片描述](//img-blog.csdn.net/20180318223847149?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 尾

以上就是我对js的异步和事件循环的理解。

