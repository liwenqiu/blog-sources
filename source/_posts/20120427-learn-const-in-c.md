title: Learn Clang's Const
date: 2012-04-27
tags: clang
---

## Preface

c语言里面的const，它是用来定义一个常量的，它复杂之处在于当它用来修饰指针类型，放在不同的位置起到的效果完全不一样。对于我这种日常完全用不到c的来说十分有必要做好笔记。


__对于非指针类型的数据来说，const置于类型声明之前还是之后只没有区别的。__

```c
int const i = 5;
const int i = 5;
```

<!--more-->

以上两中定义效果完全一样，i是常量，赋初值5之后不允许重新赋值，否则编译器会提示错误。

__当const用来修饰指针类型的时候，它放置的位置不同会有很大的差别。__

## int const * pi

```c
int i = 5;
int j = 8;

int const * pi = &i;   // 
*pi = 7;               // error: assignment of read-only location '*pi'
i = 6;                 // i可以重复赋值
pi = &j;               // 可以让pi指向其它的位置
```

这种定义方式不允许通过指针来修改指向位置的值，但是可以修改指针指向其它位置。

## const int * pi

```c
int i = 5;
int j = 8;

i = 6;
const int * pi = &i;
*pi = 7;               // error: assignment of read-only location '*pi'
i = 6;                 // i可以重复赋值
pi = &j;               // 可以让pi指向其它的位置
```

这种方式同样不允许通过指针来修改指向位置的值，但是可以修改指向的其它位置。

## int * const pi

```c
int i = 5;
int j = 8;

int * const pi = &i;
*pi = 7;               // 允许通过指针修改指向位置的值
pi = &j;               // error: assignment of read-only variable 'pi'
```

这种方式指针变量是只读的，不允许修改指针使其指向其他位置，但是允许通过指针修改它指向位置的值。


## int const * const pi

```c
int i = 5;
int j = 8;

int const * const pi = &i;
*pi = 7;               // error: assignment of read-only location '*pi'
pi = &j;               // error: assignment of read-only variable 'pi'
```

这种定义方式即不允许修改指针变量，也不允许通过指针修改它指向的值。


## Summary

__1.const放在类型定义符的之前或之后是没有区别的。__

__2.const放在指针修饰符之前表示指向的位置是只读的，不允许通过指针修改目标位置的数值。__

__3.const放在指针修饰符之后表示指针本身是只读的，不允许修改指向其它地方。__

__4.const分别放在指针修饰符之前和之后，表示指针自身和指向的目标位置的数值都不允许修改。__


const在修饰函数参数的时候，与修饰普通变量没有区别。
