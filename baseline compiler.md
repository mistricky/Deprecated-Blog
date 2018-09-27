# baseline compiler

## Baseline Compiler 是什么？

可以将 Baseline 编译器看成是 IonMonkey 的一个预备编译器（warm-up）。它的诞生使得我们在未来丢弃 JaegerMonkey 成为可能，也因此使得我们有机会大幅度的减少内存占用（在移除了JaegerMonkey之后）。这还使得我们实现新的语言特性、实现新的优化变得更加容易和直观。

引入了 Baseline 编译器之后我们的 SpiderMonkey 在各个 benchmark 上的性能提高了 5%～10%。你们可以在我们的[自动性能评测网站](http://arewefastyet.com/)上看到结果

## 为什么又要做一个JIT？

目前 Firefox 有 JaegerMonkey 和 IonMonkey 两个 JIT（这还不算已经移除的 TraceMonkey）。JaegerMonkey能够生成“很快的”代码，而IonMonkey能够生成“真正快的”代码。如果一段代码运行次数频繁，就使用JaegerMonkey编译它；如果之后运行还是很频繁，那就用IonMonkey再去编译一次。这样能够让重量级的优化用在真正需要的地方。但是这种策略的成功，需要仔细的权衡不同优化界别中编译优化的选择，因为编译本身也是需要耗费大量时间的，重量级的优化尤其如此。

简单来说，JaegerMonkey目前被我们当作简单快速的JIT来使用，但是它的设计初衷并不是这个。而 IonMonkey 跟 JaegerMonkey 之间的协作也存在一些问题。最好的解决方式是设计一个跟IonMonkey设计很接近的快速JIT，而baseline就是为了这个目的设计的。

## SpiderMonkey的现状

JavaScript代码在SpiderMonkey中的解析过程大致如下：

1. JavaScript代码首先被解释器解释执行。解释器很慢，但是它可以保证较快的启动速度，并且可以为JIT收集类型信息。
2. 当一个函数变得“hot”（执行频率达到一定阈值），则调用JaegerMonkey进行JIT，JaegerMonkey使用解释器收集的类型信息来进行优化。
3. 当函数已经被JaegerMonkey编译，并且执行次数达到IonMonkey的编译阈值（“really hot”），IonMonkey启动，花费比JaegerMonkey多得多的时间生成比JaegerMonkey好得多的代码。
4. 如果函数中有变量（或其它东西）的类型发生了变化，那么之前JaegerMonkey和IonMonkey生成的jit代码都会被丢弃；以上的三个过程从头开始。

SpiderMonkey 这么设计是有原因的：解释器虽然很慢，但是能够保证执行的结果总是正确的；而IonMonkey虽然能够让编译之后的代码执行速度变快，但是需要花费很多的时间来编译；如果IonMonkey编译的代码执行次数很少，那么反而会导致总体的运行时间变长；而JaegerMonkey处于两者之间，能够用比IonMonkey少的时间生成比解释器速度快的代码，因此它被作为一个折中。

这样的设计很完美，不是么？

不，有几个重要的问题。

## 问题

- JaegerMonkey和IonMonkey都没有能力收集类型信息，但是它们依赖类型信息来生成代码。
- JaegerMonkey和IonMonkey的调用规范是不一样的，这导致两个JIT生成的代码之间的相互调用成本很高。
- 解释器收集类型信息的功能具有局限性，并不能满足IonMonkey的需求。
- 类型推断机制需要大量的内存，而类型推断的作者 Brian Hackett 表示把 JaegerMonkey 抛开之后事情会简单很多。
- 很多 JavaScript 代码执行的次数很少，连 JaegerMonkey 不会被调用。SpiderMonkey 在 SunSpider 测试集得分上落后很大程度上就是因为这个原因。
- JaegerMonkey 写得太复杂了，我们真的不想继续维护了 ![🙂](https://s.w.org/images/core/emoji/2.4/svg/1f642.svg)

## 解决方案

我们使用 Baseline JIT 解决这些问题。它的优点：

- 代码比JaegerMonkey和IonMonkey都简单；
- 不会失效（invalidate）；
- 可以收集类型信息，inline cache机制可以帮助收集更多类型信息；
- 比解释器快10～100倍

## 致谢

Baseline was developed by Jan De Mooij and myself（注：Kannan Vijayan）, with significant contributions by Tom Schuster and Brian Hackett. Development was greatly helped by our awesome fuzz testers Christian Holler and Gary Kwong.

And of course it must be noted Baseline by itself would not serve a purpose. The fantastic work done by the IonMonkey team, and the rest of the JS team provides a reason for Baseline’s existence.