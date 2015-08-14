title: 玩转Python Lambda
date: 2015-8-3 16:28:01
tags:
- Python
- Functional Programming
categories: Memo
---
最近在学习函数式编程，又想顺便学下大名鼎鼎的**Python** ，遂把**The little schemer**里的**scheme**代码全部用**Python**实现了一遍。
函数式编程的精髓当然是传说中的**lambda**表达式，本来打算用**Python lambda**实现**scheme**代码，没想到网上一查**API**，却发现**Python lambda**实在是做的弱到不行……不能缩进（一个**lambda**函数只能写在一行），只能用**if…else…**，只能写一条语句等等。
>“Why can’t lambda expressions contain statements?

>Python lambda expressions cannot contain statements because Python’s syntactic framework can’t handle statements nested inside expressions. However, in Python, this is not a serious problem. Unlike lambda forms in other languages, where they add functionality, Python lambdas are only a shorthand notation if you’re too lazy to define a function.

>Functions are already first class objects in Python, and can be declared in a local scope. Therefore the only advantage of using a lambda instead of a locally-defined function is that you don’t need to invent a name for the function – but that’s just a local variable to which the function object (which is exactly the same type of object that a lambda expression yields) is assigned!”

为啥**lambda**表达式不能包含语句？

以上是**Python**官方**FAQ**给出的答复：**lambda**表达式就是给你偷懒少打几个字用的，不支持在语句中嵌套表达式。

基本上**Python Lambda**就应用在以下场景：
```
f = lambda x, y: x + y
g = lambda x: x + 1

print(f(1, 2))
print(g(1))
```
这匿名函数还tm有个名字。
或者你可以节约名字:
```
print((lambda x, y: x + y)(1, 2))
print((lambda x: x + 1)(1))
```
是不是觉得还不如这样：
```
print(1 + 2)
print(1 + 1)
```
不得不说**Python lambda**简直是鸡肋。

但是，你注意到**Python lambda**支持**if…else…**语句了吗，**scheme**也只有**cond…else**这一种控制语句，却是非常优美的函数式**PL**。
所以我试着用**if…else…**来搞点名堂。
```
lambda x: True if x % 2 == 0 else False
```
上面的函数判断一个数是不是偶数，其中**if…else…**是**Python lambda**的标准用法。
冒号后面接返回值**return-expression**，接着是该返回值的条件**cond-expression**，接着**else**，然后是**else**的返回值。
**Python lambda**总共以下2种写法，一种是不含**if…else…**的，一种是包含**if…else…**的：
```
lambda <args>: <return-expression>
lambda <args>: <return-expression> if <cond-expression> else <return-expression>
```
第二条语句的**"< return-expression > if < cond-expression > else < return-expression >"** 整体也是一个**return-expression**。
所以我们可以在**else**后的**return-expression**中嵌套**return-expression**来完成复杂的条件判断语句：
```
lambda <args>: <return-expression> if <cond-expression> else (<return-expression> if <cond-expression> else <return-expression>)
```
提问！如果想要在匿名函数中做一些除了**return-expression**，其他事情（比如说**print()**）,应该怎么写？
确实不太好写，不过我还是想出了一个非常**投机取巧**的办法：
```
lambda x: True if x % 2 == 0 and print(x) == None else x")
```
上面这个函数不仅返回了**x**的值，并且还打印了**x**，这里我把需要执行的语句(**print()**)转换成了逻辑表达式，而且是一个永远为真的表达式，它一定会执行，并且对业务的逻辑判断没有影响。
前提是你必须准确的知道需要执行的语句的返回值，才能写出永远为真的表达式。再写一个例子：
```
lambda x, l: l if l.insert(0,x) == None else l
```
上面这个函数把元素**x**插入到**l**的头部，并返回**l**。
通过这种**投机取巧**的办法我们可以大大扩充**Python lambda**的功能。

**Python lambda**还可以返回异常：
```
lambda x: TypeError("x is zero.") if x == 0 else 5/x
```

提问！迭代（循环）怎么写？
问得好！迭代确实是一个难题，**scheme**这样没有迭代的语句是怎么实现迭代的？递归。
首先我们先用常规函数的递归实现迭代：
```
#求和递归版本
def sum_recur(l):
  if len(l) == 0:
    return 0
  else:
    return l[0] + sum_recur(l[1:])

#求和迭代版本
def sum_iter(l):
  sum = 0
  for i in l:
    sum += i
  return sum
```
以上2个函数均实现了**list l**中元素求和，功能上是等价的。

但是匿名函数要递归调用自身难度很高。
以下可能有点高能，初学者慎入。
首先要知道，**lambda**表达式也是一个**return-expression**。所以我们还可以在**lambda**中返回**lambda**：
```
lambda <args>: lambda <args>: <return-expression>
```
先看看下面这个函数
```
def sum(f):
  return lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:])

print(sum(sum)([1,2]))
```
这个函数接受一个自身作为参数，返回值是一个求和函数，也就是实际的递归本体，即**sum(sum) = lambda l:0 if len(l) == 0 else l[0] + sum(sum)(l[1:])**,如此实现递归调用自身。
我们先把**sum(f)**转换成**lambda**函数：
```
def sum(f):
  return lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:])

lambda f: lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:])
```
上述2个函数式功能是等价的,返回值都相同，只是一个有名字而已，匿名函数没有名字如何调用自身？也是通过函数的参数，并且直接使用自身的定义，所以我们可以令**sum(sum)**为：
```
sum(lambda f: lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:]))
#第二个sum，也就是作为参数的sum已被等价的lambda替换，
#现在我们再来替换第一个sum
(lambda f: lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:])) (lambda f: lambda l:0 if len(l) == 0 else l[0] + f(f)(l[1:]))
```
这就完成了匿名函数递归调用自身。测试：

```
g = (lambda f: lambda l: 0 if len(l) == 0 else l[0] + f(f)(l[1:])) (lambda f: lambda l: 0 if len(l) == 0 else l[0] + f(f)(l[1:]))

print(g([1,2,3]))
```
接下来优化一下~

注意到
```
#f(f)(x)
#等价于
#lambda x: f(f)(x)

#那么我们可以把上面出现的f(f)都用参数替换，然后传一个lambda x: f(f)(x)作为参数即可
g = (lambda f: lambda l: 0 if len(l) == 0 else l[0] + (lambda x: f(f)(x))(l[1:])) (lambda f: lambda l: 0 if len(l) == 0 else l[0] + (lambda x: f(f)(x))(l[1:]))


g = (lambda y: (lambda f:lambda l: 0 if len(l) == 0 else l[0] + f(l[1:]))(lambda x: y(y)(x)))(lambda y: (lambda f:lambda l: 0 if len(l) == 0 else l[0] + f(l[1:]))(lambda x: y(y)(x)))
#注意：
lambda f:lambda l: 0 if len(l) == 0 else l[0] + f(l[1:])
#此函数就是真正的业务递归过程，其他结构均是辅助，我们可以提取出来
job = lambda f:lambda l: 0 if len(l) == 0 else l[0] + f(l[1:])

g = (lambda y: job(lambda x: y(y)(x)))(lambda y: job(lambda x: y(y)(x)))

print(g([1,2,3]))
```
这个**g**就是传说中的上古神器**Y-Combinator**！它就是一个通用的匿名函数递归公式。

至此，我们已经把**Python lambda**的功能扩充的基本和普通函数差不多了（额，可能还差一点）。
而且所有函数全部一行完成，然而并没有什么卵用。
