---
layout: post
title: Notes on Church Numerals
date: 2016-06-12 09:25:31
disqus: y
---
![Numbers](/images/Babylonian_numerals.png)
对于最早期的人类来说，虽然没有现代庞大的数学体系，但还是能从感官上直接判断出一只羊和两只狼的差别。虽然没有数的概念，但想必也知道自己一个人猿面对八九十只狼也是无能为力的。并且随着群居以及打猎生活的精进，人猿们越来越有了数的观念。知道用堆积的方式来表示数量，ψ代表了一个工具叉，ψψ代表两个，ψψψψψψψψψψ代表十个，不用说，这种计数的方式随着量越来越多，缺点越来越明显。于是就开始使用不同的符号来表示不同的数。这样我们依旧可以使用很少的符号来表示较大的数，每当数字过大，不便阅读书写，遂立即发明一个新符号。由此可见，对于表达数字，我们可以使用任何符号。我们可以用ψ来表达数字1，自然，我们也可以用()来表达数字1，那么ψψψψψ等于()()()()()， 也就是5。今天我们要来介绍的[Church numerals](https://en.wikipedia.org/wiki/Church_encoding)，则是一种用函数来表示数的方式。
那么我们就一起来看看，如何使用最基本的元素来构造我们的自然数，以及能否构造出操作自然数的运算法则。

根据[Peano axioms](https://en.wikipedia.org/wiki/Peano_axioms)，对于自然数而言，我们只需要定义两个符号便可描述整个自然数集：
**0**, **next**

这两个符号定义如下：
{% highlight python %}
0 = 0
(next x) = x + 1
{% endhighlight %}
有了这两个基本工具，我们便可以描述出我们能想到的任何数
{% highlight python %}
1   = (next 0)
2   = (next 1)
    = (next (next 0))
3   = (next 2)
    = (next (next (next 0)))
100 = (next 99)
    = (next (next (...... 0)))
{% endhighlight %}

那么我们接下来使用上文所说的函数，来尝试构造一下自然数。根据刚才的构造法则，我们需要一个**0**，以及一个代表下一个数的**next**。那么，我们假使**x**为**0**，函数**f**代表下一个数**next**，于是，我们可以得到：`x = 0, f(x) = 1, f(f(x)) = 2`

Church Numerals的表示略有不同：
{% highlight python %}
zero  = λf.λx.x
one   = λf.λx.(f x)
two   = λf.λx.(f (f x))
three = λf.λx.(f (f (f x)))
{% endhighlight %}
{% highlight python %}
zero  = λf.λx.x
one   = (next zero)
      = (λn.λf.λx.(f ((n f) x)) zero)
      = λf.λx.(f ((zero f) x))
      = λf.λx.(f ((λf.λx.x f) x))
      = λf.λx.(f (λx.x x))
      = λf.λx.(f x)

two   = (next one)
      = (λn.λf.λx.(f ((n f) x)) one)
      = λf.λx.(f ((one f) x))
      = λf.λx.(f ((λf.λx.(f x) f) x))
      = λf.λx.(f (λx.(f x) x))
      = λf.λx.(f (f x))

three = (next two)
      = (λn.λf.λx.(f ((n f) x)) two)
      = λf.λx.(f ((two f) x))
      = λf.λx.(f ((λf.λx.(f (f x)) f) x))
      = λf.λx.(f (λx.(f (f x)) x))
      = λf.λx.(f (f (f x)))
{% endhighlight %}

{% highlight Scheme %}
scheme:

(define zero
  (lambda (f)
    (lambda (x) x)))

(define (next n)
  (lambda (f)
    (lambda (x)
      (f ((n f) x)))))
{% endhighlight %}

{% highlight python %}
python:

zero = lambda f: lambda x: x
next = lambda n: lambda f: lambda x: (f)((n)(f)(x))
{% endhighlight %}

到现在为止，我们已经可以表达出我们想要表达的任何一个数字。那么，有了数字，自然需要一些运算法则。我们在**0，递增，递减**的基础上，可以表达自然数的所有数字，并且我们还能构造出加减乘除法则，其可以由**递增，递减**来构造**+，-**，然后由**+，-**构造**\*，/**，等等等等。如果我们再进一步，我们便会发现，其实我们不仅仅可以用这简单的语句来构造数学表达式，如果我们再增加一两条基本的编程指令，如**while**，我们便能使用**递增，递减，循环**来模拟出与现代语言相当的编程语言，只不过会复杂一些。

下面顺便也给出几个Church Numerals的运算法则。

{% highlight Scheme %}
add = λm.λn.λf.λx.((((m next) n) f) x)

two = λf.λx.(f (f x))
four = ((add two) two)
     = ((λm.λn.λf.λx.((((m next) n) f) x) two) two)
     = (λn.λf.λx.((((two next) n) f) x) two)
     = λf.λx.((((two next) two) f) x)
     = λf.λx.((((λf.λx.(f (f x)) next) two) f) x)
     = λf.λx.(((λx.(next (next x)) two) f) x)
     = λf.λx.(((next (next two)) f) x)
     = λf.λx.(((next (λn.λf.λx.(f ((n f) x)) two)) f) x)
     = λf.λx.(((next (λf.λx.(f ((two f) x)))) f) x)
     = λf.λx.(((next (λf.λx.(f (λx.(f (f x)) x)))) f) x)
     = λf.λx.(((next (λf.λx.(f (f (f x))))) f) x)
     = λf.λx.((λf.λx.(f ((λf.λx.(f (f (f x))) f) x)) f) x)
     = λf.λx.((λf.λx.(f (λx.(f (f (f x))) x)) f) x)
     = λf.λx.((λf.λx.(f (f (f (f x)))) f) x)
     = λf.λx.(λx.(f (f (f (f x)))) x)
     = λf.λx.(f (f (f (f x))))
{% endhighlight %}

{% highlight Scheme %}
mult = λm.λn.λx.(m (n x))

four = ((mult two) two)
     = ((λm.λn.λx(m (n x)) two) two)
     = (λn.λx(two (n x)) two)
     = λx(two (two x))
     = λx(two (λf.λy.(f (f y)) x))
     = λx(two (λy.(x (x y))))
     = λx(λf.λz.(f (f z)) (λy.(x (x y))))
     = λx.λz(λy.(x (x y)) (λy.(x (x y)) z))
     = λx.λz(λy.(x (x y)) (x (x z)))
     = λx.λz(x (x (x (x z))))
{% endhighlight %}
