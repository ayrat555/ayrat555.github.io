---
title:      Y combinator
date:       2017-11-12
summary:    also known as WTF combinator
categories: general

redirect_from:
  - /general/2017/11/12/ycombinator/
---

![mit-logo](/images/2017-11-12-mit-logo.svg)

In the [previous post](/general/2017/11/04/lisp-interpreter-in-lisp/) I wrote about Metacircular Evaluator - list interpeter that is written in lisp. Now I will give a short description of the Curry's Paradoxical Combinator. It will help to comprehend the idea that a thing can be described in terms of itself.

[Haskell Brooks Curry](https://en.wikipedia.org/wiki/Haskell_Curry) was an American mathematician. He was so cool that there are three programming languages named after him, Haskell, Brook and Curry. By the way, I can't find time to learn Haskell. I heard that when you know Haskell, your life is complete and you can die peacefully.

Another awesome thing that he created was Curry's Paradoxical Combinator, aka Y combinator, aka WTF combinator (by the people who try to understand it).

```lisp
Y = (lambda (f)
      ((lambda (x) (f (x x)))
       (lambda (x) (f (x x))))

(Y F) = ((lambda (x) (F (x x)))
         (lambda (x) (F (x x))))

      = (F ((lambda (x) (F (x x))) (lambda (x) (F x x))))

(Y F) = (F (Y F))
```

Y is higher order function which, when applied to some function, produces the object which is the fixed point of that function.

A fixed point of a function is an element of the function's domain that is mapped to itself by the function.

```
f(x0) = x0
```

> So Lisp is a fixed point of the of the Metacircular Evaluator.
![explosion](/images/2017-11-12-explosion.gif)
