binaryen = binary + emscripten

### C, C++ —> Emscripten —> Asm.js —> binaryen —> webAssembly

1.binaryen  webassembly的编译器

binaryen shell 可以夹在运行代码，类似解释器

asm2wasm asm.js 编译到WebAssembly

Wasm2asm  webAssembly 编译到 asm.js

s2wasm 汇编文件编译到webAssembly

### 支持编译到WebAssembly的语言

C/C++, Kotlin, Rust, Golang, AssemblyScript

WebAssembly是针对Web设计的一种低级语言，这种可移植的二进制格式旨在提高Web应用的运行速度。

### WebAssembly如何工作

加载wasm字节码 -> 将wasm字节码编译成模块 -> 将模块实例化 -> 运行函数

### WebAssembly使用场景

webassembly 是Javascript的一个受限的子集，里面不包含Javascript的任何对象，也不能直接访问DOM，本质上来讲WebAssembly只允许对TypedArray做算术运算和操作

目前，WebAssembly只是在简单模仿JS的功能，但人们计划扩展WebAssembly的使用场景，以处理JS中难以处理的事情，同时不增加语言的复杂度。比如，人们计划使WebAssembly默认支持SIMD（Single Instruction，Multiple Data，单指令流多数据流）、线程、共享内存等等功能。

许多流行视频游戏编辑器已经准备就绪，开始将WebAssembly技术与WebGL 2.0相结合，将部分3D功能引擎移植到这个全新平台上。

### 并不是JavaScript的替代品

WebAssembly 不单单给Javascript带来了性能提升，同时也造福了浏览器

### webassembly的字节码

字节码是一种可运行于其它应用中的低阶二进制代码表示

### webassembly兼容性

WebAssembly向后兼容。团队会针对老版本浏览器提供polyfill支持，目前已经构建出[初始原型](https://github.com/WebAssembly/polyfill-prototype-1)

### webassembly的表现形式

webassembly所描述的结构是一种AST的结构，它同时也可以描述一些高阶的结构，例如循环和分支。

```
;; Iterative factorial named
(func $fac-iter (param $n i64) (result i64)
    (local $i i64)
        (local $res i64)
    (set_local $i (get_local $n))
    (set_local $res (i64.const 1))
        (label $done
        (loop
            (if
                (i64.eq (get_local $i) (i64.const 0))
                    (break $done)
                (block
                    (set_local $res (i64.mul (get_local $i) (get_local $res)))
                        (set_local $i (i64.sub (get_local $i) (i64.const 1)))
                    )
                )
            )
        )
        (return (get_local $res))
)
```

### MVP(Minimum Viable Product) 最小可行性产品

类似软件开发周期模型的原型模型，先用最快最简明的方式完成一个原型，然后再逐步的迭代去完成一些细节
每次迭代都需要交付一个最小功能集合，这个产品虽然不完善，但是至少可以用

### Asm.js

给我的一个感觉就是始终是服务于C/C++的，最初的意图就是为了让C/C++程序员不用再去学习其他的东西，而是直接可以把c/c++转换为可以在浏览器中跑的代码， 这个代码就是asm.js，当然c/c++的性能和js相差很大，于是asm.js就在类型上下了很大的功夫，包括模拟溢出

asm仅仅是js的一个子集，所以能运行js的地方必然能够运行asm.js

c/c++ -> emscripten -> asm.js

### 各大浏览器的javascript引擎

运行于JavaScript引擎中，我们熟悉的有Mozilla的SpiderMonkey，Safari的JavaScriptCore，Edge的Chakra还有大名鼎鼎的V8

### js 的运行性能瓶颈

v8的出现让js的性能翻了一番，但是v8再快也难以越过语言本身的瓶颈

### webAssembly的特点

- 它是一个新的语言，它定义了一种AST，并可以用字节码的格式表示。
- 它是对浏览器的加强，浏览器能够直接理解WebAssembly并将其转化为机器码。
- 它是一种目标语言，任何其他语言都可以编译成WebAssembly在浏览器上运行。

 

### 适应的领域（详细分析以下四种领域，为什么需要提高性能，换一种说法，性能瓶颈在哪里）

计算机视觉 //TODO

游戏动画 //TODO

视频解编码 //TODO

数据加密 //TODO

### webAssembly不是一种为了让其他语言编写的模块运行到浏览器中的技术

如果只是想让C, C++ , Java等语言运行在浏览器上，这种技术早就有了。

C/C++ —> Emscripten —> asm.js

Java —> Google Web Toolkit（GWT）—> javascript

Python —> pyjamas —> js

但是这并没有解决JavaScript执行慢的问题，这跟直接用JavaScript来重写代码库是一样的作用。这就是为什么Electron能直接运行Node.js但对比传统桌面应用依然弱鸡的原因。

### Electron相对于传统桌面应用弱在哪里

安装包比较大

第一次启动比较慢（需要一些启动动画来提高下用户体验）

还有一些全平台不兼容的api，比如水果的一些api，Windows的通知，Linux的托盘，当然不是很多。

### Js在引擎中的处理过程

js —> AST —>  机器语言 （整个过程由js引擎来执行， 最后一步转换为机器语言交给底层执行）

### baseline compiler

解释器 -> Baseline JIT / Simple JIT -> Opt JIT / Full JIT (Optional Off Threading)

### JIT

现主流都采用解释器和编译器并存的架构

#### Firefox spiderMonkey

JavaScript代码在SpiderMonkey中的解析过程大致如下：

1. JavaScript代码首先被解释器解释执行。解释器很慢，**但是它可以保证较快的启动速度**，并且可以为JIT收集类型信息。
2. 当一个函数变得“hot”（执行频率达到一定阈值），则调用JaegerMonkey进行JIT，JaegerMonkey使用解释器收集的类型信息来进行优化。
3. 当函数已经被JaegerMonkey编译，并且执行次数达到IonMonkey的编译阈值（“really hot”），IonMonkey启动，花费比JaegerMonkey多得多的时间生成比JaegerMonkey好得多的代码。
4. 如果函数中有变量（或其它东西）的类型发生了变化，那么之前JaegerMonkey和IonMonkey生成的jit代码都会被丢弃；以上的三个过程从头开始。

After：鉴于解释器很慢，导致收集类型信息也很慢，于是使用baseline compiler 来进行类型收集的工作，baseline相对于jeagerMonkey和IonMonkey更简单，并且更快

#### chrome V8

V8 最初是被设计用来提高网页浏览器内部 JavaScript 执行的性能。为了获得更快的速度，V8 将 JavaScript 代码翻译成更高效的机器代码，而不是使用解释器来翻译代码。它通过使用 JIT（Just-In-Time）编译器（如 SpiderMonkey 或 Rhino（Mozilla）等许多现代 JavaScript 引擎）来将 JavaScript 代码编译为机器代码。 这里的主要区别在于 V8 不生成字节码或任何中间代码。

在 V8 的 5.9 版本出来之前（今年早些时候发布），引擎使用了两个编译器：

- full-codegen - 一个简单而且速度非常快的编译器，可以生成简单且相对较慢的机器代码。
  Full-codegen = v8中的baseline compiler
- Crankshaft  - 一种更复杂（Just-In-Time）的优化编译器，生成高度优化的代码。

V8 引擎也在内部使用多个线程：

- 主线程完成您期望的任务：获取代码，编译并执行它
- 还有一个单独的线程用于编译，以便主线程可以继续执行，而前者正在优化代码
- 一个 Profiler 线程，它会告诉运行时我们花了很多时间，让 Crankshaft 可以优化它们
- 一些线程处理垃圾收集器

当第一次执行 JavaScript 代码时，V8 利用 full-codegen 编译器，直接将解析的 JavaScript 翻译成机器代码而不进行任何转换。这使得它可以非常快速地开始执行机器代码。请注意，V8 不使用中间字节码，从而不需要解释器。

v8的两种JIT compiler

```
To recall, V8 has two compilers: one that does a quick-and-dirty job (full-codegen), and one that focuses on hot spots (Crankshaft).
```



#### V8中的隐藏类

----

V8在第一次执行JavaScript代码的时候，会直接将它翻译为机器相关的本地机器码，而不是使用中间字节码

在第一次执行到访问某个对象的属性的代码时，V8 会找出对象当前的隐藏类。同时，V8 会假设在相同代码段里的其他所有对象的属性访问都由这个隐藏类进行描述，并修改相应的内联代码让他们直接使用这个隐藏类。当 V8 预测正确的时候，属性值的存取仅需一条指令即可完成。如果预测失败了，V8 会再次修改内联代码并移除刚才加入的内联优化。

----

JavaScript 是一种基于原型的语言：没有类和对象而是使用克隆创建的。 JavaScript 也是一种动态编程语言，这意味着属性可以在实例化后方便地添加或从对象中移除。

大多数JavaScript解释器用了类似字典的结构（基于hash），来储存js的属性值在内存中的位置，这种结构使得js查找一个属性要做的计算量要大得多。

由于使用字典查找内存中对象属性的位置效率非常低，因此 V8 使用了不同的方法：隐藏类。隐藏类与 Java 等语言中使用的固定对象（类）的工作方式类似，除了隐藏类是在运行时创建的这点区别。现在，让我们看看他们实际的例子

在JavaScript对象被创建的时候，v8会创建一个隐藏类，每次往这个对象上添加属性的时候，v8都会创建一个隐藏类，咋一看似乎每次添加一个属性都创建一个新的隐藏类非常低效。实际上，利用类转移信息，隐藏类可以被重用。下次创建一个 `Point` 对象的时候，就可以直接共享由最初那个 `Point`对象所创建出来的隐藏类。例如，如果又一个 `Point` 对象被创建出来了：

#### 性能瓶颈

随着最近 AJAX 技术的兴起，JavaScript 现在已经变成了实现基于 web 的应用程序（例如我们自己的 Gmail）的核心技术。JavaScript 程序从聊聊几行变成数百 KB 的代码。JavaScript 被设计于完成一些特定的任务，虽然 JavaScript 在做这些事情的时候通常都很高效，但是性能已经逐渐成为进一步用 JavaScript 开发复杂的基于 web 的应用程序的瓶颈。