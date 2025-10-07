### Summary
Generally, we are looking at the structure of a computer program in Lisp.
Lisp uses prefix notation, like Haskell.
We discuss a lot about applicative-order, where a program is evaluated as SOMETHING and normal-order, where it is done a different way.
### Exercises
##### Exercise 1.1
Below is a sequence of expressions. What is the result printed by the interpreter in response to each expression? Assume the sequence is to be evaluated in the order in which it is presented.

```scheme
10
> 10
(+ 5 3 4)
> 12
(- 9 1)
> 8
(/ 6 2)
> 3
(+ (* 2 4) (- 4 6))
> 6
(define a 3)
> 
(define b (+ a 1))
> 
(+ a b (* a b))
> 19
(= a b)
> #f
(if (and (> b a) (< b (* a b)))
	b
	a)
> 4
cond ((= a 4) 6)
	 ((= b 4) (+ 6 7 a))
	 (else 25))
> 16
(+ 2 (if (> b a) b a))
> 6
(* (cond ((> a b) a)
         ((< a b) b)
         (else -1))
   (+ a 1))
> 16
```
##### Exercise 1.2
Translate the following expression into prefix form
> $\frac{5+4+(2-(3-(6+\frac{4}{5})))}{3(6-2)(2-7)}$

```scheme
(/ (+ 5 4 (- 2 (- 3 (+ 6 (/ 4 5))))) (* 3 (- 6 2) (- 2 7)))
```
##### Exercise 1.3
> Define a procedure that takes three numbers as arguments and returns the sum of the squares of the two larger numbers.

```scheme
(define (square x) (* x x))
(define (sumofsquares a b) (+ (square a) (square b))
(define (sumoflargesttwosquares a b c)
        (cond ((and (> a c) (> b c)) (sumofsquares a b))
              ((and (> a b) (> c b)) (sumofsquares a c))
              (else (sumofsquares b c))))
```
##### Exercise 1.4
> Observe that our model of evaluation allows for combinations whose operators are compound expressions. Use this observation to describe the behavior of the following procedure

```scheme
(define (a-plus-abs-b a b)
  ((if (> b 0) + -) a b))
```
The initial step of the procedure, the conditional `(> b 0)`, is returning an operator, so if b is greater than 0, we are performing `(+ a b)`, else we are performing `(- a b)`, which effectively adds the absolute of b to a. Nifty stuff.
##### Exercise 1.5
> Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:
```scheme
> (define (p) (p))
> (define (test x y)
>   (if (= x 0) 0 y))
```
> Then he evaluates the expression:
```scheme
> (test 0 (p))
```
> What behavior will Ben observe with an interpreter that uses applicative-order evaluation? What behavior will he observe with an interpreter that uses normal-order evaluation? Explain your answer. (Assume that the evaluation rule for the special form `if` is the same whether the interpreter is using normal or applicative order: The predicate expression is evaluated first, and the result determines whether to evaluate the consequent or the alternative expression.)

I feels like it would error regardless of order, but whatever. In applicative-order, because p is undefined when we try to define it to p, it would error out. In normal-order, it would infinitely recurse, because it is constantly trying to evaluate p, which is equal to p, etc etc.
EDIT: Seems I was wrong, it is the other way around maybe.
##### Exercise 1.6
> Alyssa P. Hacker doesn't see why `if` needs to be provided as a special form. "Why can't I just define it as an ordinary procedure in terms of `cond`?" she asks. Alyssa's friend Eva Lu Ator claims that this can indeed be done, and she defines a new version of if:
```scheme
(define (new-if predicate then-clause else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```
> Eva demonstrates the program for Alyssa:
```scheme
> (new-if (= 2 3) 0 5)
5
> (new-if (= 1 1) 0 5)
0
```
> Delighted, Alyssa uses `new-if` to rewrite the square-root program:
```scheme
(define (sqrt-iter guess x)
  (new-if (good-enough? guess x)
          guess
          (sqrt-iter (improve guess x) x)))
```
> What happens when Alyssa attempts to use this to compute square roots? Explain.

Seems like it will just work to me, I also think a `case` statement is just a neater `if`.
EDIT: Seems it will be an infinite loop. Basically because the order of operations is that a function will evaluate it's arguments before applying it's own logic. The special form `if` doesn't do that, it evaluates the condition first and then only evaluates the relevant branch.
##### Exercise 1.7
> The `good-enough?` test used in computing square roots will not be very effective for finding the square roots of very small numbers. Also, in real computers, arithmetic operations are almost always performed with limited precision. This makes our test inadequate for very large numbers. Explain these statements, with examples showing how the test fails for small and large numbers. An alternative strategy for implementing good-enough? is to watch how `guess` changes from one iteration to the next and to stop when the change is a very small fraction of the guess. Design a square-root procedure that uses this kind of end test. Does this work better for small and large numbers?

To the first point, it is trivial to find an example where `good-enough?` will not be just that for small numbers. Let's take `0.000001`. The square root for that is `0.001`. If we use `sqrt-iter 0.002 0.000001`, our result is `0.002`, which isn't very close at all.

For large numbers, we are generally not looking at decimal precision. I am not sure when this is appropriate with square numbers, example, but if you are always rounding to the nearest integer, our result requires more work at the end, and also is doing a lot of unnecessary computation. Due to the amount of computation, it will go on for a long, long time (if at all). Due to floating point numbers having specific representations in bytes, a larger number will just not be as precise as a floating point number and therefore never hit that magic threshold value.

```scheme
#lang sicp
(define (average a b)
  (/ (+ a b) 2))

(define (improve guess x)
  (average guess (/ x guess)))

(define (good-enough? guess next-guess)
  (< (abs (/ (- next-guess guess) guess)) 0.00000000001))

(define (sqrt-iter guess x)
  (define next-guess (improve guess x))
  (if (good-enough? guess next-guess)
      next-guess
      (sqrt-iter next-guess x)))

(define (sqrt x)
  (sqrt-iter 1.0 x))
```

This works better, because the precision is proportional to the size of the number.
##### Exercise 1.8
> Newton's method for cube roots is based on the fact that if $y$ is an approximation to the cube root of $x$, then a better approximation is given by the value $\frac{\frac{x}{y^{2}}+2y}{3}$.
> Use this formula to implement a cube-root procedure analogous to the square root procedure. (In section 1.3.4 we will see how to implement Newton's method in general as an abstraction of these square-root and cube-root procedures.)

```scheme
#lang sicp
(define (average a b)
  (/ (+ a b) 2))

(define (improve guess x)
  (/ (+ (/ x (* guess guess)) (* 2 guess)) 3))

(define (good-enough? guess next-guess)
  (< (abs (/ (- next-guess guess) guess)) 0.00000000001))

(define (cube-iter guess x)
  (define next-guess (improve guess x))
  (if (good-enough? guess next-guess)
      next-guess
      (cube-iter next-guess x)))

(define (cube x)
  (cube-iter 1.0 x))
```
