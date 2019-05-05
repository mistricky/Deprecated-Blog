# Perl6 中的异常处理

perl6 的异常处理机制，个人认为大体上分为两个部分，一个是 `exception`， 另一个是 `failure`，有时候又叫 `soft failure`，意思为遇到错误的时候不及时的抛出，在使用它的时候才会被抛出来。

## Failure

`failure` 是 Perl6 作为弱类型的标志，这种错误被 Perl6 发现， 但是并没有立即抛出

Golang 是一种典型的强类型和静态类型的语言，看一小段 Golang 的代码

```go
func main() {
	result := 1 / 0
	fmt.Println("hello")
	fmt.Println(result)
}
```

这段代码在编译时期就会被抛出异常，因为 Golang 是强类型语言，它不容忍代码里有错误没有被 catch 也没有被做其他的处理而被忽略的，换句话说，强类型的语言不容许 `soft failure`

而 Perl6 却不太一样

```perl
my $result = 1 / 0;

say "hello";
say $result;
```

output: 

```shell
hello
Attempt to divide by zero when coercing Rational to Str
  in block <unit> at main.p6 line 4
```

可以看到这里的 `hello` 被打印出来了

为了做对比，看一下同为脚本语言的 `Python`

```python
foo = 1 / 0
print(foo)
```

**output**

```shell
Traceback (most recent call last):
  File "strong.py", line 1, in <module>
    foo = 1 / 0
ZeroDivisionError: integer division or modulo by zero
```

因为 `Python` 是强类型的，它也不容许异常被忽略

说白了，`Failure` 只是 `soft failure` 的一个代表类型，它会允许错误遗留在代码中（只要你不去显式的使用它）

下面的代码不会发生任何异常

```perl
my $result = 1 / 0;

say "hello";
```

**output**

```perl
hello
```

## Exception

异常是任何时候可以中断程序的一个东西，当然这个有例外， 如果你捕获了它，不然它就会一直往外层抛出，最终造成程序中断。`Perl6` 中的异常对象大多存在于 `X` 模块中

我们可以简单的创建一个自己的异常，并且抛出

```perl
sub foo() {
	X::AdHoc.new(payload => "This is my exception").throw
}

foo();
```

**output**

```bash
This is my exception
  in sub foo at main.p6 line 1
  in block <unit> at main.p6 line 5
```

通过创建一个 `AdHoc` 对象然后调用它的 throw 方法就能将它抛出了

当然你可以通过调用 die subroutine 来实现相同的效果，与上面的代码等价

```perl
sub foo() {
	die "This is my exception"
}

foo();
```

除此之外，还有很多的 `Exception` 类型，这个留到后面再说

## 错误的捕获

在 Perl6 中，几乎跟其他语言的惯有方法一致，都是通过 `try catch` 代码块来将其捕获，在 `catch` 中进行处理，接着上面的例子

```perl
my $result;

try {
	$result = 1 / 0;
	say $result;
	
	CATCH {
		say $_.^name;
	}
}
```

异常的对象会保存在 CATCH 块中的 `$_` 变量中，这是 Perl 一贯的做法， 这里不再赘述了。

这里会有一个坑，这段代码的执行结果是

```perl
X::Numeric::DivideByZero
Attempt to divide by zero when coercing Rational to Str
  in block <unit> at main.p6 line 4
```

很显然打印出了异常的类型，但是异常依旧被抛出了！

这是因为 CATCH 是类似于一个 given 的结构，可以用 `when` 来分支各种类型的异常，进行处理，单纯的直接在里面写处理逻辑是不会被捕获的， 它还是会向外面抛，这样的设计，得益我们可以轻松实现记录异常信息，但是不用 `rethrow`。

下面让我们尝试捕获它

```perl
my $result;

try {
	$result = 1 / 0;
	say $result;

	CATCH {
		default {say $_.^name}
	}
}
```

在 default 分支里写上处理逻辑，default 分支是 "万能"的，你可以在这里捕获任意类型的异常，但是当我们知道该异常的类型时，我们可以针对性的捕获

```perl
my $result;

try {
	$result = 1 / 0;
	say $result;

	CATCH {
		when X::Numeric::DivideByZero {say $_.^name}
	}
}
```

现在我们已经可以成功的捕获异常了！

但是尝试把上面的 `say $result` 语句去掉呢？

```perl
my $result;

try {
	$result = 1 / 0;

	CATCH {
		when X::Numeric::DivideByZero {say $_.^name}
	}
}
```

这时运行发现，不会打印异常的名字了！

上面说了，这是一个 `failure` ，是 `Exception` 的一层封装，它并不会立即抛出，而是等到使用它的时候它才会被抛出，因此，这段代码是不会抛出异常的，当然也不会被捕获！

上面 try 的代码只有一行，这样写是不是有点太重了？所以我们可以使用单行 try

```perl
my $result = 1 / 0;

try say $result;

say $!.^name; #X::Numeric::DivideByZero
```

在外部，错误对象被存进了 `$!`，我们可以在外面进行异常的处理，值得注意的是，单行 try 并不会重新抛出。

## Warning

除了异常，我们还可以自己定义 `Warning`，Warning 也是一种类型的异常，不同的是，它并不会导致程序直接中断。

```perl
sub generateWarn(){
	warn "this is warn";
}

generateWarn();

(0 .. 10)>>.say;
```

**output**

```shell
this is warn
  in sub generateWarn at main.p6 line 1
0
1
2
3
4
5
6
7
8
9
10
```

发现尽管 warn 被抛出，但是程序还是能正常的运行下去

因为 warn 是一种特殊的异常，所以我们不能再用 `CATCH` 去捕获了，现在，我们要用 `CONTROL`来进行捕获

```perl
sub generateWarn(){
	warn "this is warn";
}

try {
	generateWarn();

	CONTROL {
		default {say $_.^name}
	}
}

(0 .. 10)>>.say;
```

**output**

```shell
CX::Warn
0
1
2
3
4
5
6
7
8
9
10
```

警告⚠️不会给你的程序带来致命的危害，它有的时候就想叽叽喳喳的小鸟，你可以让它们安静下来，用 `quietly`！

```perl
sub generateWarn(){
	warn "this is warn";
}

quietly {
	generateWarn();
}

(0 .. 10)>>.say;
```

任何被 `quietly` 包住的代码块，都会屏蔽掉 warn，不让它跑到标准输出流中

**output**

```shell
0
1
2
3
4
5
6
7
8
9
10
```

