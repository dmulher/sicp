# Introduction to Data Abstraction
We are introducing the concept of compound data types to represent more complex data structures.
### 2.1.1 - Example: Arithmetic Operations for Rational Numbers
To start, we are looking at rational numbers (fractions), a personal favorite.
We are using `cons`, which creates a pair, and then accessing the values of the pair using `car` and `cdr`.
```scheme
(define x (cons 1 2))
(car x)
> 1
(cdr x)
> 2
```
There is a specific call-out for making our fractions. A direct definition of the `car` function for `numer` is faster, but means it is harder to debug. The second implementation has two functions calls instead of one, but allows us to just debug `numer` calls, instead of all `car` calls.
```scheme
(define numer car)
; vs
(define numer x (car x))
```
##### Exercise 2.1
> Define a better version of `make-rat` that handles both positive and negative arguments. `make-rat` should normalize the sign so that if the rational number is positive, both the numerator and denominator are positive, and if the rational number is negative, only the numerator is negative.

```scheme
(define (make-rat n d)
  (let ((g (abs (gcd n d)))
        (n (if (not (equal? (< n 0) (< d 0))) (- 0 (abs n)) (abs n)))
        (d (abs d)))
    (cons (/ n g) (/ d g))))
```
### 2.1.2 - Abstraction Barriers
This section talks about the layers of abstraction of a system. Specifically pointing out that having those abstraction layers allows us to better update a system. If we have correct abstraction, we can mess with the inner workings of pieces of the system without having to update the rest of the system.
##### Exercise 2.2
> Consider the problem of representing line segments in a plane. Each segment is represented as a pair of points: a starting point and an ending point. Define a constructor `make-segment` and selectors `start-segment` and `end-segment` that define the representation of segments in terms of points. Furthermore, a point can be represented as a pair of numbers: the $x$ coordinate and the $y$ coordinate. Accordingly, specify a constructor `make-point` and selectors `x-point` and `y-point` that define this representation. Finally, using your selectors and constructors, define a procedure `midpoint-segment` that takes a line segment as argument and returns its midpoint (the point whose coordinates are the average of the coordinates of the endpoints). To try your procedures, you’ll need a way to print points:
```scheme
(define (print-point p)
  (newline)
  (display "(")
  (display (x-point p))
  (display ",")
  (display (y-point p))
  (display ")"))
```

```scheme
(define (make-segment a b) (cons a b))
(define (start-segment x) (car x))
(define (end-segment x) (cdr x))

(define (make-point x y) (cons x y))
(define (x-point a) (car a))
(define (y-point a) (cdr a))

(define (midpoint-segment s)
  (let ((x (/ (+ (x-point (start-segment s)) (x-point (end-segment s))) 2))
        (y (/ (+ (y-point (start-segment s)) (y-point (end-segment s))) 2)))
    (make-point x y)))
```
##### Exercise 2.3
> Implement a representation for rectangles in a plane. (Hint: You may want to make use of Exercise 2.2.) In terms of your constructors and selectors, create procedures that compute the perimeter and the area of a given rectangle. Now implement a different representation for rectangles. Can you design your system with suitable abstraction barriers, so that the same perimeter and area procedures will work using either representation?

```scheme
(define (make-point x y) (cons x y))
(define (x-point a) (car a))
(define (y-point a) (cdr a))

; Rect as top-left point and bottom-right point
(define (make-rect x y) (cons x y))
(define (top-left-point a) (car a))
(define (bottom-right-point a) (cdr a))
(define (width a) (abs (- (x-point (bottom-right-point a)) (x-point (top-left-point a)))))
(define (height a) (abs (- (y-point (bottom-right-point a)) (y-point (top-left-point a)))))

; Rect as start point and dimension
(define (make-rect x y) (cons x y))
(define (start-point a) (car a))
(define (dimensions a) (cdr a))
(define (width a) (x-point (dimensions a)))
(define (height a) (y-point (dimensions a)))

(define (rect-area a) (* (width a) (height a)))
(define (rect-perimeter a) (* 2 (+ (width a) (height a))))
```
Making an abstraction barrier of width and height that stands between the constructors and selectors, and the area and perimeter procedures
### 2.1.3 - What is Meant By Data?
Here we start to show the different ways to represent data, and the contracts by which this data hangs together. The example provides a different representation of `cons`, `car`, and `cdr` that is equally valid
```scheme
(define (cons x y)
  (define (dispatch m)
    (cond ((= m 0) x)
          ((= m 1) y)
          (else (error "Argument not 0 or 1: CONS" m))))
  dispatch)
(define (car z) (z 0))
(define (cdr z) (z 1))
```
##### Exercise 2.4
> Here is an alternative procedural representation of pairs. For this representation, verify that `(car (cons x y))` yields `x` for any objects `x` and `y`.
```scheme
(define (cons x y)
  (lambda (m) (m x y)))
(define (car z)
  (z (lambda (p q) p)))
```
> What is the corresponding definition of `cdr`? (Hint: To verify that this works, make use of the substitution model of Section 1.1.5.)

```scheme
(define (cdr z)
  (z (lambda (p q) q)))
; (cdr (cons 1 2))
; ((cons 1 2) (lambda (p q) q))
; ((lambda (m) (m 1 2)) (lambda (p q) q))
; ((lambda (p q) q) 1 2)
; (lambda (1 2) 2)
; 2
```
##### Exercise 2.5
> Show that we can represent pairs of nonnegative integers using only numbers and arithmetic operations if we represent the pair $a$ and $b$ as the integer that is the product $2^a3^b$. Give the corresponding definitions of the procedures `cons`, `car`, and `cdr`.

```scheme
(define (cons x y)
  (* (expt 2 x) (expt 3 y)))
(define (count-primes x p)
  (define (count-iter x p count)
    (if (= (remainder x p) 0)
        (count-iter (/ x p) p (+ count 1))
        count))
  (count-iter x p 0))
(define (car z)
  (count-primes z 2))
(define (cdr z)
  (count-primes z 3))
```
##### Exercise 2.6
> In case representing pairs as procedures wasn’t mind-boggling enough, consider that, in a language that can manipulate procedures, we can get by without numbers (at least insofar as nonnegative integers are concerned) by implementing $0$ and the operation of adding $1$ as
```scheme
(define zero (lambda (f) (lambda (x) x)))
(define (add-1 n)
  (lambda (f) (lambda (x) (f ((n f) x)))))
```
> This representation is known as Church numerals, after its inventor, Alonzo Church, the logician who invented the $λ$-calculus.
> Define one and two directly (not in terms of `zero` and `add-1`). (Hint: Use substitution to evaluate `(add-1 zero))`. Give a direct definition of the addition procedure `+` (not in terms of repeated application of `add-1`).

```scheme
(define zero (lambda (f) (lambda (x) x)))
(define (add-1 n)
  (lambda (f) (lambda (x) (f ((n f) x)))))
; (add-1 zero)
; (lambda (f) (lambda (x) (f ((zero f) x))))
; (lambda (f) (lambda (x) (f (((lambda (g) (lambda (y) y)) f) x))))
; (lambda (f) (lambda (x) (f (((lambda (y) y)) x))))
; (lambda (f) (lambda (x) (f x)))
(define one (lambda (f) (lambda (x) (f x))))
; (add-1 one)
; (lambda (f) (lambda (x) (f ((one f) x))))
; (lambda (f) (lambda (x) (f (((lambda (g) (lambda (y) (g y))) f) x))))
; (lambda (f) (lambda (x) (f ((lambda (y) (f y)) x))))
; (lambda (f) (lambda (x) (f (f x))))
(define two (lambda (f) (lambda (x) (f (f x)))))

(define (+ a b)
  (lambda (f) (lambda (x) ((a f) ((b f) x)))))
```
### 2.1.4 - Extended Exercise: Interval Arithmetic
##### Exercise 2.7
> Alyssa’s program is incomplete because she has not specified the implementation of the interval abstraction. Here is a definition of the interval constructor:
```scheme
(define (make-interval a b) (cons a b))
```
> Define selectors upper-bound and lower-bound to complete the implementation.

```scheme
(define (lower-bound x) (car x))
(define (upper-bound x) (cdr x))
```
##### Exercise 2.8
> Using reasoning analogous to Alyssa’s, describe how the difference of two intervals may be computed. Define a corresponding subtraction procedure, called `sub-interval`.

```scheme
(define (sub-interval x y)
  (make-interval (- (lower-bound x) (upper-bound y))
                 (- (upper-bound x) (lower-bound y))))
```
##### Exercise 2.9
> The *width* of an interval is half of the difference between its upper and lower bounds. The width is a measure of the uncertainty of the number specified by the interval. For some arithmetic operations the width of the result of combining two intervals is a function only of the widths of the argument intervals, whereas for others the width of the combination is not a function of the widths of the argument intervals. Show that the width of the sum (or difference) of two intervals is a function only of the widths of the intervals being added (or subtracted). Give examples to show that this is not true for multiplication or division.

```scheme
(define (width x) (/ (+ (lower-bound x) (upper-bound x)) 2))
```
**Addition**
$width(x)=\frac{x_u-x_l}{2}$
$x+y=(x_l+y_l,x_u+y_u)$
$width(x+y)=width(x)+width(y)$
$\frac{(x_u+y_u)-(x_l+y_l)}{2}=\frac{x_u-x_l}{2}+\frac{y_u-y_l}{2}$
$x_u+y_u-x_l-y_l=x_u+y_u-x_l-y_l$

**Subtraction**
$width(x)=\frac{x_u-x_l}{2}$
$x-y=(x_l-y_u,x_u-y_l)$
$width(x-y)=width(x)+width(y)$
$\frac{(x_u-y_l)-(x_l-y_u)}{2}=\frac{x_u-x_l}{2}+\frac{y_u-y_l}{2}$
$(x_u-y_l)-(x_l-y_u)=(x_u-x_l)+(y_u-y_l)$
$x_u-x_l+y_u-y_l=x_u-x_l+y_u-y_l$

Multiplication and division are unreasonable. Multiplication uses max and min, where division is using multiplication with $y^{-1}$.
##### Exercise 2.10
> Ben Bitdiddle, an expert systems programmer, looks over Alyssa’s shoulder and comments that it is not clear what it means to divide by an interval that spans zero. Modify Alyssa’s code to check for this condition and to signal an error if it occurs.

```scheme
(define (div-interval x y)
  (if (or (and (negative? (lower-bound y))
               (positive? (upper-bound y)))
          (= (lower-bound y) 0)
          (= (upper-bound y) 0))
      (error "y interval spans 0")
      (mul-interval
        x
        (make-interval (/ 1.0 (upper-bound y))
                       (/ 1.0 (lower-bound y))))))
```
##### Exercise 2.11
> In passing, Ben also cryptically comments: “By testing the signs of the endpoints of the intervals, it is possible to break `mul-interval` into nine cases, only one of which requires more than two multiplications.” Rewrite this procedure using Ben’s suggestion.

```scheme
(define (mul-interval x y)
  ; | x_l | x_u | y_l | y_u |   z_l   |   z_u   |
  ; | --- | --- | --- | --- | ------- | ------- |
  ; |  -  |  N  |  -  |  N  | x_u*y_u | x_l*y_l |
  ; |  P  |  -  |  P  |  -  | x_l*y_l | x_u*y_u |
  ; |  -  |  N  |  P  |  -  | x_l*y_u | x_u*y_l |
  ; |  P  |  -  |  -  |  N  | x_u*y_l | x_l*y_u |
  ; |  P  |  -  |  N  |  P  | x_u*y_l | x_u*y_u |
  ; |  -  |  N  |  N  |  P  | x_l*y_u | x_l*y_l |
  ; |  N  |  P  |  P  |  -  | x_l*y_u | x_u*y_u |
  ; |  N  |  P  |  -  |  N  | x_u*y_l | x_l*y_l |
  ; |  N  |  P  |  N  |  P  | MIN(x_l*y_u, x_u*y_l) | MAX(x_l*y_l, x_u*y_u) |
  (cond ((and (negative? (upper-bound x))
              (negative? (upper-bound y)))
         (make-interval (* (upper-bound x) (upper-bound y))
                        (* (lower-bound x) (lower-bound y))))
        ((and (positive? (upper-bound x))
              (positive? (upper-bound y)))
         (make-interval (* (lower-bound x) (lower-bound y))
                        (* (upper-bound x) (upper-bound y))))
        ((and (negative? (upper-bound x))
              (positive? (lower-bound y)))
         (make-interval (* (lower-bound x) (upper-bound y))
                        (* (upper-bound x) (lower-bound y))))
        ((and (positive? (lower-bound x))
              (negative? (upper-bound y)))
         (make-interval (* (upper-bound x) (lower-bound y))
                        (* (lower-bound x) (upper-bound y))))
        ((and (positive? (lower-bound x))
              (negative? (lower-bound y)))
         (make-interval (* (upper-bound x) (lower-bound y))
                        (* (upper-bound x) (upper-bound y))))
        ((negative? (upper-bound x))
         (make-interval (* (lower-bound x) (upper-bound y))
                        (* (lower-bound x) (lower-bound y))))
        ((and (negative? (lower-bound x))
              (positive? (lower-bound y)))
         (make-interval (* (lower-bound x) (upper-bound y))
                        (* (upper-bound x) (upper-bound y))))
        ((negative? (upper-bound y))
         (make-interval (* (upper-bound x) (lower-bound y))
                        (* (lower-bound x) (lower-bound y))))
        ((else) (make-interval (min (* (lower-bound x) (upper-bound y))
                                    (* (upper-bound x) (lower-bound y)))
                               (max (* (lower-bound x) (lower-bound y))
                                    (* (upper-bound x) (upper-bound y)))))))
```
##### Interlude
> After debugging her program, Alyssa shows it to a potential user, who complains that her program solves the wrong problem. He wants a program that can deal with numbers represented as a center value and an additive tolerance; for example, he wants to work with intervals such as $3.5 \pm 0.15$ rather than $[3.35, 3.65]$. Alyssa returns to her desk and fixes this problem by supplying an alternate constructor and alternate selectors:
```scheme
(define (make-center-width c w)
  (make-interval (- c w) (+ c w)))
(define (center i)
  (/ (+ (lower-bound i) (upper-bound i)) 2))
(define (width i)
  (/ (- (upper-bound i) (lower-bound i)) 2))
```
> Unfortunately, most of Alyssa’s users are engineers. Real engineering situations usually involve measurements with only a small uncertainty, measured as the ratio of the width of the interval to the midpoint of the interval. Engineers usually specify percentage tolerances on the parameters of devices, as in the resistor specifications given earlier.
##### Exercise 2.12
> Define a constructor `make-center-percent` that takes a center and a percentage tolerance and produces the desired interval. You must also define a selector `percent` that produces the percentage tolerance for a given interval. The `center` selector is the same as the one shown above.

```scheme
(define (make-center-percent c p)
  (let ((t (* c p)))
    (make-interval (- c t) (+ c t))))

(define (percent x) (- (/ (upper-bound x) (center x)) 1))
```
##### Exercise 2.13
> Show that under the assumption of small percentage tolerances there is a simple formula for the approximate percentage tolerance of the product of two intervals in terms of the tolerances of the factors. You may simplify the problem by assuming that all numbers are positive.

TODO
##### Interlude
> After considerable work, Alyssa P. Hacker delivers her finished system. Several years later, after she has forgotten all about it, she gets a frenzied call from an irate user, Lem E. Tweakit. It seems that Lem has noticed that the formula for parallel resistors can be written in two algebraically equivalent ways: $\frac{R_1R_2}{R_1+R_2}$ and $\frac{1}{\frac{1}{R_1}+\frac{1}{R_2}}$.
> He has written the following two programs, each of which computes the parallel-resistors formula differently:
```scheme
(define (par1 r1 r2)
  (div-interval (mul-interval r1 r2)
                (add-interval r1 r2)))

(define (par2 r1 r2)
  (let ((one (make-interval 1 1)))
    (div-interval
     one (add-interval (div-interval one r1)
                       (div-interval one r2)))))
```
> Lem complains that Alyssa’s program gives different answers for the two ways of computing. This is a serious complaint.
##### Exercise 2.14
> Demonstrate that Lem is right. Investigate the behavior of the system on a variety of arithmetic expressions. Make some intervals $A$ and $B$, and use them in computing the expressions $\frac{A}{A}$ and $\frac{A}{B}$. You will get the most insight by using intervals whose width is a small percentage of the center value. Examine the results of the computation in center-percent form (see Exercise 2.12).
##### Exercise 2.15
> Eva Lu Ator, another user, has also noticed the different intervals computed by different but algebraically equivalent expressions. She says that a formula to compute with intervals using Alyssa’s system will produce tighter error bounds if it can be written in such a form that no variable that represents an uncertain number is repeated. Thus, she says, `par2` is a “better” program for parallel resistances than `par1`. Is she right? Why?
##### Exercise 2.16
> Explain, in general, why equivalent algebraic expressions may lead to different answers. Can you devise an interval-arithmetic package that does not have this shortcoming, or is this task impossible? (Warning: This problem is very difficult.)
