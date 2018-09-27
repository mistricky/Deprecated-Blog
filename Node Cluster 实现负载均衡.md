# Node child_process的fork，spawn，exec我有话要说

![这里写图片描述](http://img.blog.csdn.net/20171129205714417?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSGFvRGFXYW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

node的异步io处理方式是众所周知的，io越密集，node 的优势就越明显，但是node是单线程的，不能像java那种多线程语言一样开启一些worker thread，充分利用cpu资源，提高计算效率和操作系统的吞吐量，但是node依然给我们提供了child_process让我们可以创建子进程来充分的利用cpu资源，让node也能处理cpu密集的应用。

## exec

child_process给我们提供了三个方法用于创建子进程，fork，spawn，exec，首先来看一下exec。

```
child_process.exec(command[, options][, callback])
```

exec开始一个子进程执行shell命令，并缓存输出传入callback的第二个参数，这个缓存区默认只有200kb的大小，可以通过options.maxBuffer进行设置。

**options**

```
command <string> 要运行的命令，用空格分隔参数。
options <Object>
cwd <string> 子进程的当前工作目录。
env <Object> 环境变量键值对。
encoding <string> 默认为 'utf8'。
shell <string> 用于执行命令的 shell。 在 UNIX 上默认为 '/bin/sh'，在 Windows 上默认为 process.env.ComSpec。 详见 Shell Requirements 与 Default Windows Shell。
timeout <number> 默认为 0。
maxBuffer <number> stdout 或 stderr 允许的最大字节数。 默认为 200*1024。 如果超过限制，则子进程会被终止。 查看警告： maxBuffer and Unicode。
killSignal <string> | <integer> 默认为 'SIGTERM'。
uid <number> 设置该进程的用户标识。（详见 setuid(2)）
gid <number> 设置该进程的组标识。（详见 setgid(2)）
windowsHide <boolean> Hide the subprocess console window that would normally be created on Windows systems. Default: false.
callback <Function> 当进程终止时调用，并带上输出。
error <Error>
stdout <string> | <Buffer>
stderr <string> | <Buffer>
```

废话不多说看代码吧
（我这里使用typescript来写的）

```
import { ChildProcess, exec } from 'child_process';

let child:ChildProcess = exec("ls -a",(err:any,stdout:string,stderr:string) => {
    if(err){
        console.log((err as Error).stack);
        return;
    }
    console.log(`data is \n ${stdout}`);
});
```

这里执行一个简单的ls命令来查看当前目录下的所有文件。
我的操作系统是mac os，unix默认操作系统启动的时候会开启一个0号进程来创建一个1号进程，0号进程被称作交换进程，用来把磁盘交换区的进程换入主存，1号进程又被称作初始化进程，它会为用户创建一个login进程，同时，假如我们在shell 里输入命令的时候，比如刚刚我们输入的ls -a ，那么shell会为我们创建一个子进程用来执行ls，这里的node的exec也是一样的，node通过child_process创建一个子进程用来执行ls -a，从作用来说，exec就是用来开启一个子进程来执行shell命令的。

这里的callback接受三个参数，第一个是一个错误对象，第二个是一个标准输出流的数据，第三个是标准错误流的数据，err这个参数有两种状态（在用ts写的时候需要注意），err可以为null也可以为error，因此我这里把它的类型声明为any，当执行发生错误的时候，err的类型就为Error，如果没有发生错误则为null，在if(err)里把err进行类型转换，把any转换为Error，然后再打印错误栈。

下面是一句打印stdout的语句，这里把这个参数命名为stdout有点不太严谨，stdout是node里的一个标准输出流，但是这里的stdout仅仅代表标准输出流的data，因此把它声明为string（它可以被声明为string | Buffer）

（由于我开始之前已经tsc -w了，这里我直接执行）

```
node main.js
```

输出如下：

```
➜  dist node main.js 
data is 
 .
..
main.js
```

同时为了证明maxBuffer的作用，我们特地可以把maxBuffer设置为1，来让它抛出异常

```
let child:ChildProcess = exec("ls -a",{maxBuffer:1},(err:any,stdout:string,stderr:string) => {
    if(err){
        console.log((err as Error).stack);
        return;
    }
    console.log(`data is \n ${stdout}`);
});
```

```
node main.js
```

```
Error: stdout maxBuffer exceeded
    at Socket.onChildStdout (child_process.js:328:14)
    at emitOne (events.js:116:13)
    at Socket.emit (events.js:211:7)
    at addChunk (_stream_readable.js:263:12)
    at readableAddChunk (_stream_readable.js:246:13)
    at Socket.Readable.push (_stream_readable.js:208:10)
    at Pipe.onread (net.js:594:20)
```

## spawn

spawn与exec有点相似之处，这是因为spawn也是作为开启一个子进程执行shell的存在，不同之处是spawn不会把输出流中的数据做一个缓存，所以没有一个大小的限制，这通常用spawn来运行返回大量数据的子进程，如图像处理，文件读取等。而`exec`则应用来运行只返回少量返回值的子进程，如只返回一个状态码。

下面我们还是执行ls -a，与exec不同的是spawn把shell命令的参数专门放在一个数组里。

```
child_process.spawn(command[, args][, options])
```

下面还是来看一下代码

```
import { spawn, ChildProcess } from 'child_process';
import * as iconvLite from 'iconv-lite';

let spawnObj:ChildProcess = spawn('ls',['-a']);

spawnObj.stdout.on("data",(data:Buffer) => {
    console.log(iconvLite.decode(data,'utf-8'));
});

spawnObj.stderr.on('err',(err:Error) => {
    console.log(err);
});

spawnObj.on('exit',(code:number) => {
    console.log(`exit code is ${code}`);
});
```

这里我引入了iconv用来把buffer转换为string，其实也直接可以用object.toString()方法来转换。

运行main.js

```
node main.js

➜  dist node main.js 
.
..
main.js

exit code is 0
```

可以看出运行并无大碍

## fork

接下来介绍最后一个方法，fork方法。
node还有一个模块叫作cluster，可以用它来创建集群实现负载均衡，cluster的创建进程的方法就是使用了child_process的fork。

fork与前两个方法不太一样，它不再是用来执行shell的一个方法，而是接受一个node文件来创建子进程，子进程立即执行该文件。使用fork会开启子进程和父进程之间的ipc通道，用fork开启的子进程还会拥有额外的send方法用于发送消息到付进程。

下面来编写一个child.ts用于给子进程执行

```
console.log(`child pid is ${process.pid}`);

process.on("message",(msg:string) => {
    console.log(`[child] get a data from parent is ${msg}\n`);
    process.send(`\nhello parent\n`);
});
```

整个代码非常简单，开始打印一下该进程的pid，然后监听message事件，打印父进程发送来的消息，同时发送消息给父进程。值得注意的是，process对象永远指代当前进程。

**main.ts**

```
let child:ChildProcess = fork("./child.js");

child.on("message",(msg:string) => {
    console.log(`[parent] get a data from child is ${msg}\n`);
});

child.send("\nhello child\n");
```

这是父进程的代码，和子进程差不多这里就不再赘述了。

```
➜  dist node main.js
child pid is 1287
[child] get a data from parent is 
hello child


[parent] get a data from child is 
hello parent

```

我们可以用ps命令查看这个子进程

```
ps -ef 1287

UID   PID  PPID   C STIME   TTY           TIME CMD
501  1287  1286   0  9:37下午 ttys000    0:00.07 /usr/local/bin/node ./child.js
```



还有一点需要说明，send方法是同步的因此不建议发送大量数据, 发送大量的数据可以使用 pipe 来代替