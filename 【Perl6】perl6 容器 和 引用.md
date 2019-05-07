# 【Perl6】perl6 中的容器和引用	

perl6 中没有引用，或者说 perl6 中到处都能见到引用

## 常量

perl6 中声明一个常量是通过 `:=` 二元运算符来进行操作的，这个操作在 perl6 中又被称作为 `bind`

比如

```perl
my $b := 1;

$b = 2;
# Cannot assign to an immutable value
# in block <unit> at container.p6 line 3
```

perl6 会马上抛出一个异常，来中断程序，因为赋值给了一个 immutable value，至于为什么它会是 immutable 的，这个后面再说了

## Bind

```perl
my $a = 1;
my $b := $a;

$b = 2;

say $a; # 2
```

这段代码实际上是 `$b` 这里是一个 标量容器，然后将 `$a` 赋值给了 `$b` 的容器里的值，赋值操作符在运算的时候，实际是直接改变了容器内部的值，引用本身就是内存的别名，这里可以看作是 `$b` 是 `$a` 的别名，所以 `$a` 也跟着一起改变了。

再回过头来看前面的例子，为什么刚刚会报 `immutable`

```perl
my $b := 1;

$b = 2;
```

从 `容器`的角度来看，这里实际上是把常量 `1` 赋值给了标量容器 `$b` 里的值，这里赋值的话，实际上是改变的容器内的值，但是 `1` 本身是个常量是不能被更改的，所以就会抛出异常了。

## List

perl 中的线性容器有 `Array`, `List`, `Seq` 

`Array` 是可变的，`List` 是不可变的，`Seq` 一般是指 的序列，通过惰性列表出来的数据结构就是 `Seq` 。

先来看一个 `List` 的例子

```perl
my $list = (1, 2, 3);

$list[1] = 1;
# Cannot modify an immutable List ((1 2 3))
```

List 中的每个元素都是 `Immutable` 的，但是并不影响整体的赋值。

```perl
my $list = (1, 2, 3);

$list = ();
```

但是如果你用 `Bind` 将它和现有的 List 字面量进行绑定的话，它就不能被赋值给其他的值了。

这是因为 List 字面量被装入了 `$list` 容器，后面更改值的时候，被认定为正在修改一个 List 字面量，因此会报错。

但是我们怎么让 `$list` 的元素变成可变的呢？用一个标量容器去封装它。如何声明一个标量容器，`Scalar` 并没有提供一个 new 方法来为我们创建 `Scalar` 的实例，但是我们可以通过匿名的 `$` 来创建，像下面这样。

```perl
my $list = (my $, my $, my $);
```

这里实际上 `list` 中的每个元素都是一个标量元素，用赋值来修改元素的值的时候，实际上是修改的元素容器内部的值。

```perl
$list[0] = 1;
$list[1] = 2;
$list[2] = 3;
```

也可以在声明的时候就初始化它，然后再进行改变

```perl
my $list = (my $ = 1, my $ = 2, my $ = 3);

$list[0] = 4;
.say for $list.List;
# 4
# 2
# 3
```

为了防止一些意外的情况发生，可以用 `Positional` 进行修饰

```perl
my Positional $list = (my $ = 1, my $ = 2, my $ = 3);
```

## Array

如果每次要声明一个可变的 list，这样做确实很麻烦，有一个 `sigil` 可以从一开始就帮你做好这一切，它就是 `@`，对，就是数组

```perl
my @names = ['Jon', 'Henry', 'Jerry'];

say @names.WHAT; # Array
```

数组会在创建的时候就把每个元素进行标量容器的装箱，从而使得它是可变的。但是却不会影响元素的本身的类型

```perl
@names[2] = 'hah';

say @names[0].WHAT # Str
```

## 奇怪的 List

来看一个奇怪的例子

```perl
my $list = (1, 2, 3);

.say for $list; # (1, 2, 3)
say $list.WHAT; # List
```

这里只打印出了一个元素，是 `(1, 2, 3)`，但是我们 WHAT 它的类型，确实是一个 `List`，这是为什么呢？ 

因为 `$` sigil 就代表一个标量，而标量的意思就是一个值，因此，会出现这样的情况，但是它确实是个 List，可以通过 List 来让它迭代起来!

```perl
my $list = (1, 2, 3);

.say for $list.List;
# 1
# 2
# 3
```

如果你认为这样是正常的，那么看看下面的例子

```perl
my $list := (1, 2, 3);

.say for $list;
# 1
# 2
# 3
```

这里没有使用 `List` 还是能正常的迭代，这是为什么呢？

`:=` 用于将元素的值装入标量的容器内，所以是可以迭代的

## 容器的默认值

容器的值是可以有默认值的

```perl
my $name is default('qqqq') = 'Henry';

$name = Nil;

say $name; # qqq
```

包括数组容器也可以有默认值的

```perl
my @arr is default<-1> = [1, 2];

say @arr[2]; # -1
```

容器的值的默认值也是可以有默认值的，Perl 在开始执行之前就已经分配了默认值了，比如

```perl
my $hello;

say $hello # Any
```

Any 就是这个 `$hello` 标量容器的一个默认值

