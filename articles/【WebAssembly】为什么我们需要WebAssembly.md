# 【WebAssembly】为什么我们需要WebAssembly

WebAssembly是一种运行在现代网络浏览器中的新型代码并且提供新的性能特性和效果。它设计的目的不是为了手写代码而是为诸如C、C++和Rust等低级源语言提供一个高效的编译目标。

对于网络平台而言，这具有巨大的意义——这为客户端app提供了一种在网络平台以接近本地速度的方式运行多种语言编写的代码的方式；在这之前，客户端app是不可能做到的。

而且，你可以在不知道如何编写WebAssembly代码的情况下就可以使用它。WebAssembly的模块可以被导入的到一个网络app（或Node.js）中，并且暴露出供JavaScript使用的WebAssembly函数。JavaScript框架不但可以使用WebAssembly获得巨大性能优势和新特性，而且还能使得各种功能保持对网络开发者的易用性。

### 为什么我们需要WebAssembly

现在的Js生态那么好， 好像所有事情都能用Js做到，换个角度，又好像所有的事情都不能用Js做到。
V8的出现导致Js的性能飙升，在不涉及一些需要性能的场景，做一些简单的动画，用户交互方面是绰绰有余的。尽管如此，WebApp与Native App的性能还是存在差距的。

WebAssembly是web界的汇编语言，能够以接近原生的速度运行。

![img](https://mdn.mozillademos.org/files/14647/emscripten-diagram.png)