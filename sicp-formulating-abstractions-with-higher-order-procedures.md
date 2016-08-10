---
title: SICP 用高阶函数做抽象
date: 2016-07-17 19:08:42
tags:
  - SICP
  - Scheme
---

能够以过程作为参数，或者以过程作为返回值，这样能操作过程的过程称为**高阶过程**

## 过程作为参数

计算 a 到 b 的各个整数之和：

``` scheme
(define (sum-integers)
  (if (> a b)
      0
      (+ a (sum-integers (+ a 1) b))))
```
<!-- more -->

计算 a 到 b 的各个整数的立方之和：

``` scheme
(define (cube x) (* x x x))

(define (sum-cubes a b)
  (if (> a b)
      0
      (+ (cube a) (sum-cubes (+ a 1) b))))
```

通过上述的两个例子，可以发现它们有着相同的过程，不同的只是过程的名称。由此我们可以得到下面这个抽象过程：

``` scheme
(define (<name> a b)
  (if (> a b)
      0
      (+ (<term> a)
         (<name> (<next> a) b)))
```

按照这样的模式，讲其中的空位换成形式参数即可得到：

``` scheme
(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) (next b)))))
```

对比之前计算 a 到 b 的各个整数之和的过程，依然有作为上下界的 a、b 两个参数，另外增加了一个过程参数 term 和 next。 sum 可以接受过程为参数并调用。即可以通过 sum 作为基本构件，去形式化其他概念。

由此我们可以使用这个抽象的过程算出之前的两个例子.

计算 a 到 b 之间的各个整数之和：

``` scheme
(define (identity x) x)

(define (inc n) (+ n 1))

(define (sum-integers a b)
  (sum identity a inc b))
```

计算 a 到 b 之间的各个整数的立方之和：

``` scheme
(define (cube x) (* x x x))

(define (inc n) (+ n 1))

(define (sum-cubes a b)
  (sum cube a inc b))
```

## 用 lambda 构造过程
有一种方式可以直接描述过程而不需要显式的定义。通过 lambda 这种特殊形式去完成这类描述，这种特殊形式能够创建出所需要的过程。

借助 `lambda`，可以将 `sum-cubes` 写成这种形式：

``` scheme
(define (sum-cubes a b)
  (sum (lambda (x) (* x x x))
       a
       (lambda (n) (+ n 1))
       b))
```

这样创建的过程与使用 `define` 定义的过程的区别是这种过程没有与环境中的任何名字关联。其他编程语言中匿名函数也是如此。

`(define (inc n) (+ n 1))` 等价于 `(define inc (lambda (n) (+ n 1)))`

### 用 let 创建局部变量
lambda 的另一个应用是创建局部变量。例如计算 `f(x,y) = x(1+x)^2+y(1-y)+(1+x)(1-y)`，我们可以将其分解为：

```
a = 1+x
b = 1-y
f(x,y) = xa^2 + yb + ab
```

这一计算过程，使用 define 定义为：

``` scheme
(define (f x y)
  (define (f-helper a b)
    (+ (* x (square a))
       (* y b)
       (* a b)))

  (f-helper (+ 1 x)
            (- 1 y)))
```

这一过程可以通过 lambda 表达式：

``` scheme
(define (f x y)
  ((lambda (a b)
      (+ (* x (square a))
         (* y b)
         (* a b))))
  (+ 1 x)
  (- 1 y))
```

这样的结构使用的概率不小，因此有一个专门的特殊形式称为 let，使得定义这一过程更加方便。

使用 let 创建的过程：

``` scheme
(define (f x y)
  (let ((a (+ 1 x)
        (b (- 1 y))
    (+ (* x (square a))
       (* y b)
       (* a b))))))
```

let 表达式的第一个部分是一个名字-表达式的对偶表，第二部分是含有对偶表中名字的一些过程。当 let 被求值时，每个名字将会被关联到对应的表达式。

let 表达式描述的变量的作用域即为该 let 的体。

当 x 的值为 5 时，下述过程的值为 38：

``` scheme
(+ (let (x 3)
      (+ x (* x 10)))
    x)
```

也就是说在 let 的作用域范围内的变量与外部变量相同时，它的值为 let 定义时的值。即依赖变量与外部相同时，使用外部变量；不依赖时使用局部变量值。

## 过程作为一般性的方法

