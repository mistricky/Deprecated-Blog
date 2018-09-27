嗯， 公司有在设计面试题，觉得一道题很有意思，于是记下来（涨知识）。这道题公司是要求15min之内写出来，本菜用了差不多30min，还好混进了实习生（滑稽）

话不多说，先看题

![这里写图片描述](https://img-blog.csdn.net/20180712232746676?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

问题和用时都已经写的很清楚，不知道狮子哥和万总是什么解题思路，先说说我的解题方法吧。

乍一看题，随机一个duration，duration为1 ～5 s，duration过后返回一个promise，resolve后打印一句话。
由于setTimeout是异步的，整个程序会因为异步而导致打印顺序不一。我们要做的就是让它顺序打印。挺简单的吧，我么可以这样。

```
(async () => {
    for(let i = 0;i < 5;i++){
        await startTask(`target_${i}`)
    }
})()
```

好像也能实现题目的要求。但是很恼火的就是必须在startTask函数里实现。

不知道破坏规矩没有，我还是修改了startTask函数的定义，emmmm，看看我的实现吧

```
function sleep(duration){
    return new Promise(resolve => setTimeout(resolve, duration))
}

function request(target){
    return sleep(Math.floor(Math.random() * 5 + 1) * 100).then(() => {
        console.log('response', target, 'data')
    })
}

let startTask = (() => {
    let sp = 0
    let waitQueue = []
    return async function Inner(target){
        if(sp < 0) {
            waitQueue.push(target)
            return
        }
        sp--
        await request(target)
        sp++
        if(waitQueue.length > 0) Inner(waitQueue.shift())
    }      
})()

(async () => {
    for(let i = 0;i < 5;i++){
        await startTask(`target_${i}`)
    }
})()
```

和semaphore有点像，但是又不是semaphore，sp < 0 表示callback queue中有callback等待被执行，为了保持有序，其他进来准备执行request的任务都会被压入一个waitqueue，等待第一个任务被执行完毕，就会拉取waitqueue中等待的任务执行。

打印结果

```
response target_0 data
response target_1 data
response target_2 data
response target_3 data
response target_4 data
```

----
华丽的分割线

---

晚上问了万总他们的解法，从一开始的解题思路大概想复杂了。看一下他的解题方法吧
```
function startTask(target){ startTask.p = (startTask.p || Promise.resolve()).then(() => request(target)).catch(console.error)}

```
在startTask上挂载了`p`，这个p是一个promise，这样做的目的是，让所有执行startTask的任务都使用同一个promise链，这样就理所当然的有了秩序

（流下了没有技术的泪水）