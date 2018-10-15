# 【dart】被省略的函数返回值和 void

在 dart 里，被省略的函数返回值不是 void，dart 可以允许你不写函数的返回值，编译器会自动帮助你返回 null

```
void main(){
    print(hello() == null); // true
}

hello() { }
```

但是如果你显示的声明了 void ,那就另当别论了。

```
void main(){
    print(hello() == null); // error
}

void hello() { }
```

这和有些语言的不声明返回值默认是 void 的有点不一样