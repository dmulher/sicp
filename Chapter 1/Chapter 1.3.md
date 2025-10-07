This chapter is going to cover procedures that either take procedures as an argument, or return procedures, known as **higher-order procedures**.
### 1.3.1 - Procedures as Arguments
We look at procedures as arguments here, mostly in terms of summation.
##### Exercise 1.29
> Simpson’s Rule is a more accurate method of numerical integration than the method illustrated above. Using Simpson’s Rule, the integral of a function $f$ between $a$ and $b$ is approximated as $\frac{h}{3}(y_0+4y_1+2y_2+4y_3+2y_4+2y_{n-2}+4y_{n-1}+y_n)$ where $h=(b-a)/n$, for some even integer $n$, and $y_k=f(a+kh)$. (Increasing $n$ increases the accuracy of the approximation.) Define a procedure that takes as arguments $f$, $a$, $b$, and $n$ and returns the value of the integral, computed using Simpson’s Rule. Use your procedure to integrate $cube$ between $0$ and $1$ (with $n = 100$ and $n = 1000$), and compare the results to those of the integral procedure shown above.

```scheme
(define (simpson f a b n)
  (define h (/ (- b a) n))
  (define (next-a x) (+ x 2))
  (define (yk k) (f (+ a (* k h))))
  (* (/ h 3)
     (+ (yk 0)
        (* 4 (sum yk (+ a 1) next-a (- n 1)))
        (* 2 (sum yk (+ a 2) next-a (- n 1)))
        (yk n))))
```

Both $n=100$ and $n=1000$ result in $\frac{1}{4}$.
##### Exercise 1.30
> The sum procedure above generates a linear recursion. The procedure can be rewritten so that the sum is performed iteratively. Show how to do this by filling in the missing expressions in the following definition:
```scheme
(define (sum term a next b)
  (define (iter a result)
    (if <??>
        <??>
        (iter <??> <??>)))
  (iter <??> <??>))
```

```scheme
(define (sum term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (+ (term a) result))))
  (iter a 0))
```
##### Exercise 1.31
> a. The `sum` procedure is only the simplest form of a vast number of similar abstractions that can be captured as higher-order procedures. Write an analogous procedure called product that returns the product of the values of a function at points over a given range. Show how to define `factorial` in terms of product. Also use `product` to compute approximations to $\pi$ using the formula $\frac{\pi}{4}=\frac{2 \cdot 4 \cdot 4 \cdot 6 \cdot 6 \cdot 8 \cdot \cdot \cdot}{3 \cdot 3 \cdot 5 \cdot 5 \cdot 7 \cdot 7 \cdot \cdot \cdot}$.
> b. If your `product` procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

```scheme
(define (product term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (* (term a) result))))
  (iter a 1))

(define (product-rec term a next b)
  (if (> a b)
      1
      (* (term a)
         (product-rec term (next a) next b))))

(define (factorial a b)
  (product identity a inc b))
```
##### Exercise 1.32
> a.
>   Show that `sum` and `product` (Exercise 1.31) are both special cases of a still more general notion called accumulate that combines a collection of terms, using some general accumulation function:
```scheme
(accumulate combiner null-value term a next b)
```
>    `accumulate` takes as arguments the same term and range specifications as `sum` and `product`, together with a `combiner` procedure (of two arguments) that specifies how the current term is to be combined with the accumulation of the preceding terms and a `null`-value that specifies what base value to use when the terms run out. Write `accumulate` and show how `sum` and `product` can both be defined as simple calls to `accumulate`.
> b.
>   If your `accumulate` procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

```scheme
(define (accumulate combiner null-value term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (combiner (term a) result))))
  (iter a null-value))

(define (accumulate-rec combiner null-value term a next b)
  (if (> a b)
    null-value
    (combiner
     (term a)
     (accumulate-rec combiner null-value term (next a) next b))))

(define (sum term a next b)
  (accumulate + 0 term a next b))

(define (product term a next b)
  (accumulate * 1 term a next b))
```
##### Exercise 1.33
> You can obtain an even more general version of `accumulate` (Exercise 1.32) by introducing the notion of a *filter* on the terms to be combined. That is, combine only those terms derived from values in the range that satisfy a specified condition. The resulting `filtered-accumulate` abstraction takes the same arguments as `accumulate`, together with an additional predicate of one argument that specifies the filter. Write `filtered-accumulate` as a procedure. Show how to express the following using `filtered-accumulate`:
> a.
>    the sum of the squares of the prime numbers in the interval $a$ to $b$ (assuming that you have a `prime?` predicate already written)
> b.
>    the product of all positive integers less than $n$ that are relatively prime to $n$ (i.e., all positive integers $i<n$ such that $GCD(i,n)=1$).

```scheme
(define (filter-accumulate predicate combiner null-value term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a)
              (if (predicate a) (combiner (term a) result) result))))
  (iter a null-value))

(define (sum-of-square-primes a b)
  (filter-accumulate prime? + 0 square a inc b))

(define (product-of-relative-primes n)
  (define (relative-prime? i) (= (gcd i n) 1))
  (filter-accumulate relative-prime? * 1 identity 1 inc n))
```
### 1.3.2 - Constructing Procedures Using `lambda`
This subchapter introduces the concept of `lambda` being used to make anonymous procedures, similar to how it is used in modern Python. An example is defining
$f(x,y)=x(1+xy)^2+y(1-y)+(1+xy)(1-y)$
```scheme
(define (f x y)
  (define (f-helper a b)
    (+ (* x (square a))
       (* y b)
       (* a b)))
  (f-helper (+ 1 (* x y))
            (- 1 y)))
```
which we can use `lambda` to remove the need for a named procedure. Note the format is `((lambda (<var>) (<lambda-body>)) (<args>)))`, so `lambda` is effectively a procedure returning the function, which is being called with the following arguments.
```scheme
(define (f x y)
  ((lambda (a b)
    (+ (* x (square a))
       (* y b)
       (* a b)))
  (+ 1 (* x y))
  (- 1 y)))
```
We can also use a special form `let` if we are just defining local variables. Note the change in structure here, where we are instead defining the arguments, and then applying them to the procedure. `(let ((<v1> (<d1>)) ...) (<body>))`, such that the body is the last argument to the `let` special form.
```scheme
(define (f x y)
  (let ((a (+ 1 (* x y)))
        (b (- 1 y)))
    (+ (* x (square a))
       (* y b)
       (* a b))))
```
##### Exercise 1.34
> Supposed we define the procedure
```scheme
(define (f g) (g 2))
```
> Then we have
```scheme
> (f square)
4
> (f (lambda (z) (* z (+ z 1))))
6
```
> What happens if we (perversely) ask the interpreter to evaluate the combination `(f f)`? Explain.

``` scheme
(f f)
(f 2)  ; (f g) => (g 2) thus (f f) => (f 2)
(2 2)  ; (f g) => (g 2) thus (f 2) => (2 2)
```
Substituting the variables, we end up with an error where we try to call 2 as a function.
### 1.3.3 Procedures as General Methods
Looks like we are going to do a binary search through partial integrals.
```scheme
(define (search f neg-point pos-point)
  (let ((midpoint (average neg-point pos-point)))
    (if (close-enough? neg-point pos-point)
        midpoint
        (let ((test-value (f midpoint)))
          (cond ((positive? test-value)
                 (search f neg-point midpoint))
                ((negative? test-value)
                 (search f midpoint pos-point))
                (else midpoint))))))

(define (close-enough? x y) (< (abs (- x y)) 0.001))

(define (half-interval-method f a b)
  (let ((a-value (f a))
       (b-value (f b)))
    (cond ((and (negative? a-value) (positive? b-value))
           (search f a b))
          ((and (negative? b-value) (positive? a-value))
           (search f b a))
          (else
           (error "Values are not of opposite sign" a b)))))
```

```scheme
(define tolerance 0.00001)
(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2))
       tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))
```
##### Exercise 1.35
> Show that the golden ratio $ϕ$ (Section 1.2.2) is a fixed point of the transformation $x \mapsto 1 + \frac{1}{x}$, and use this fact to compute $ϕ$ by means of the fixed-point procedure.

As we might recall, the golden ratio $ϕ=\frac{1+\sqrt{5}}{2}$ which satisfies $ϕ^2=ϕ+1$. Converting $ϕ$ to $x$ and dividing both sides by $x$ gives us $x=1+\frac{1}{x}$.
```scheme
(define gold-jerry (fixed-point (lambda (x) (+ 1 (/ 1 x))) 1.0))
```
##### Exercise 1.36
> Modify `fixed-point` so that it prints the sequence of approximations it generates, using the `newline` and `display` primitives shown in Exercise 1.22. Then find a solution to $x^x = 1000$ by finding a fixed point of $x \mapsto \frac{log(1000)}{log(x)}$. (Use Scheme’s primitive $log$ procedure, which computes natural logarithms.) Compare the number of steps this takes with and without average damping. (Note that you cannot start fixed-point with a guess of 1, as this would cause division by $log(1) = 0$.)

```scheme
(define tolerance 0.00001)
(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2))
       tolerance))
  (define (try guess i)
    (display i) (display ": ") (display guess)
    (newline)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next (+ i 1)))))
  (try first-guess 1))

(define (find-x start-guess)
  (fixed-point (lambda (x) (/ (log 1000) (log x))) start-guess))

(define (damp-x start-guess)
  (fixed-point (lambda (x) (average x (/ (log 1000) (log x)))) start-guess))
```
With a small extra update to `fixed-point` to also display how many guesses are being made, we can see that `find-x` takes 35 attempts (Starting at 2), where `damp-x` takes 10. Pretty big improvement.
##### Exercise 1.37
> a.
>    An infinite *continued* fraction is an expression of the form
> $f=\frac{N_1}{D_1+\frac{N_2}{D_2+\frac{N_3}{D_3+...}}}$
>    As an example, one can show that the infinite continued fraction expansion with the $N_i$ and the $D_i$ all equal to $1$ produces $\frac{1}{ϕ}$, where $ϕ$ is the golden ratio (described in Section 1.2.2). One way to approximate an infinite continued fraction is to truncate the expansion after a given number of terms. Such a truncation—a so-called *$k$-term finite continued fraction*—has the form
> $\frac{N_1}{D_1+\frac{N_2}{...+\frac{N_k}{D_k}}}$
>    Suppose that $n$ and $d$ are procedures of one argument (the term index $i$) that return the $N_i$ and $D_i$ of the terms of the continued fraction. Define a procedure `cont-frac` such that evaluating `(cont-frac n d k)` computes the value of the $k$-term finite continued fraction. Check your procedure by approximating $\frac{1}{ϕ}$ using
```scheme
(cont-frac (lambda (i) 1.0)
           (lambda (i) 1.0)
           k)
```
>    for successive values of $k$. How large must you make $k$ in order to get an approximation that is accurate to 4 decimal places?
> b.
>    If your `cont-frac` procedure generates a recursive process, write one that generates an iterative process. If it generates an iterative process, write one that generates a recursive process.

Remembering the golden ratio: $ϕ=\frac{1+\sqrt{5}}{2} \approx 1.6180$, $1/ϕ=\frac{2}{1+\sqrt{5}} \approx 0.6180$
```scheme
(define (cont-frac n d k)
  (define (cont-frac-iter n d k den)
    (if (= k 0)
        den
        (cont-frac-iter n d (- k 1) (/ (n k) (+ (d k) den)))))
  (cont-frac-iter n d k 0))

(define (cont-frac-rec n d k)
  (define (cont-frac-rec-inner n d k i)
    (if (> i k)
        0
        (/ (n i) (+ (d i) (cont-frac-rec-inner n d k (+ i 1))))))
  (cont-frac-rec-inner n d k 1))
```
We start to hit 4 decimal precision at $k=11$.
##### Exercise 1.38
> In 1737, the Swiss mathematician Leonhard Euler published a memoir *De Fractionibus Continuis*, which included a continued fraction expansion for $e-2$, where $e$ is the base of the natural logarithms. In this fraction, the $N_i$ are all $1$, and the $D_i$ are successively $1, 2, 1, 1, 4, 1, 1, 6, 1, 1, 8,\ldots$. Write a program that uses your `cont-frac` procedure from Exercise 1.37 to approximate $e$, based on Euler’s expansion.

```scheme
(define (approx-e k)
  (let ((d (lambda (i) (if (= (remainder i 3) 2)
                           (* (remainder i 3)
                              (+ (quotient i 3) 1))
                           1)))
        (n (lambda (i) 1.0)))
    (+ 2 (cont-frac n d k))))
```
Checking other answers online, you could rewrite the first lambda:
```scheme
(define (approx-e k)
  (let ((d (lambda (i) (if (= (remainder i 3) 2)
                           (* (/ (+ i 1) 3) 2)
                           1)))
        (n (lambda (i) 1.0)))
    (+ 2 (cont-frac n d k))))
```
Which I think I prefer.
##### Exercise 1.39
> A continued fraction representation of the tangent function was published in 1770 by the German mathematician J.H. Lambert:
> $tan(x)=\frac{x}{1-\frac{x^2}{3-\frac{x^2}{5-\ldots}}}$
> where $x$ is in radians. Define a procedure `(tan-cf x k)` that computes an approximation to the tangent function based on Lambert’s formula. `k` specifies the number of terms to compute, as in Exercise 1.37.

```scheme
(define (tan-cf x k)
  (let ((n (lambda (i) (if (= i 1) x (- (square x)))))
        (d (lambda (i) (- (* 2 i) 1))))
    (cont-frac n d k)))
```
Keeping in mind that we are using subtraction instead of addition for the continuous fraction, we want our numerator to be negative.
Also, be wary of testing values around $\frac{\pi}{2}$, or you'll end up with something very large or small and, having forgotten what a tan graph looks like, get very confused. Think before you test.
### 1.3.3 - Procedures as Returned Values
This subchapter teaches us about the other half of higher-order ~~functions~~ procedures, ones that return procedures. This is something used in python, with decorators.
```scheme
(define (average-damp f)
  (lambda (x) (average x (f x))))

(define (sqrt x)
  (fixed-point (average-damp (lambda (y) (/ x y)))
               1.0))

(define (cube-root x)
  (fixed-point (average-damp (lambda (y) (/ x (square y))))
               1.0))
```

The chapter goes on to do a lot of maths around Newton's method of approximating roots and gets a little bit much. I might have to return to this chapter later, to fully appreciate the maths and therefore the power of these procedures.

```scheme
(define dx 0.00001)
(define (deriv g)
  (lambda (x) (/ (- (g (+ x dx)) (g x)) dx)))

(define (newton-transform g)
  (lambda (x) (- x (/ (g x) ((deriv g) x)))))

(define (newtons-method g guess)
  (fixed-point (newton-transform g) guess))

(define (sqrt x)
  (newtons-method
   (lambda (y) (- (square y) x)) 1.0))
```
##### Exercise 1.40
> Define a procedure `cubic` that can be used together with the `newtons-method` procedure in expressions of the form `(newtons-method (cubic a b c) 1)` to approximate zeros of the cubic $x^3 + ax^2 + bx + c$.

```scheme
(define (square x) (* x x))
(define (cube x) (* x x x))
(define (cubic a b c)
  (lambda (x) (+ (cube x) (* a (square x)) (* b x) c)))
```
With a = b = c = 1, we would expect x to be equal to -1.
```
1: 1
2: 0.33333777776275186
3: -0.40739341574970156
4: -1.4188731238603447
5: -1.1184919351394478
6: -1.0124818785025846
7: -1.000153742427375
8: -1.000000022096024
-0.9999999999997796
```
##### Exercise 1.41
> Define a procedure `double` that takes a procedure of one argument as argument and returns a procedure that applies the original procedure twice. For example, if `inc` is a procedure that adds 1 to its argument, then `(double inc)` should be a procedure that adds 2. What value is returned by `(((double (double double)) inc) 5)`

```scheme
(define (double f) (lambda (x) (f (f x))))
```
`(double (double inc))` should add 4, as should `((double double) inc)`. However, `(double (double double))` is harder to track. `(double (double double)) inc` should apply double 4 times, thus leaving us with 16 `inc`s, so the returned value should be 21, which it is.
##### Exercise 1.42
> Let $f$ and $g$ be two one-argument functions. The composition $f$ after $g$ is defined to be the function $x \mapsto f(g(x))$. Define a procedure `compose` that implements composition. For example, if `inc` is a procedure that adds 1 to its argument,
```scheme
((compose square inc) 6)
49
```

```scheme
(define (compose f g)
  (lambda (x) (f (g x))))
```
##### Exercise 1.43
> If $f$ is a numerical function and $n$ is a positive integer, then we can form the $n^{th}$ repeated application of $f$, which is defined to be the function whose value at $x$ is $f(f(\ldots(f(x))\ldots))$. For example, if $f$ is the function $x \mapsto x + 1$, then the $n^{th}$ repeated application of $f$ is the function $x \mapsto x+n$. If $f$ is the operation of squaring a number, then the $n^{th}$ repeated application of $f$ is the function that raises its argument to the $2^n$-th power. Write a procedure that takes as inputs a procedure that computes $f$ and a positive integer $n$ and returns the procedure that computes the $n^{th}$ repeated application of $f$ . Your procedure should be able to be used as follows:
```scheme
((repeated square 2) 5)
625
```
> Hint: You may find it convenient to use `compose` from Exercise 1.42

```scheme
(define (repeated f n)
  (define (identity g) g)
  (cond ((= n 0) identity)  ; I know the question says positive, but whatever
        ((= n 1) f)
        (else (compose f (repeated f (- n 1))))))
```
##### Exercise 1.44
> The idea of *smoothing* a function is an important concept in signal processing. If $f$ is a function and $dx$ is some small number, then the smoothed version of $f$ is the function whose value at a point $x$ is the average of $f(x-dx)$, $f(x)$, and $f(x+dx)$. Write a procedure `smooth` that takes as input a procedure that computes $f$ and returns a procedure that computes the smoothed $f$. It is sometimes valuable to repeatedly smooth a function (that is, smooth the smoothed function, and so on) to obtain the *$n$-fold smoothed function*. Show how to generate the $n$-fold smoothed function of any given function using $smooth$ and $repeated$ from Exercise 1.43.

```scheme
(define dx 0.00001)
(define (smooth f)
  (lambda (x) (/ (+ (f (- x dx))
                    (f x)
                    (f (+ x dx)))
                 3)))

(define (nth-smoothed f n)
  ((repeated smooth n) f))
```
##### Exercise 1.45
> We saw in Section 1.3.3 that attempting to compute square roots by naively finding a fixed point of $y \mapsto \frac{x}{y}$ does not converge, and that this can be fixed by average damping. The same method works for finding cube roots as fixed points of the average-damped $y \mapsto \frac{x}{y^2}$. Unfortunately, the process does not work for fourth roots—a single average damp is not enough to make a fixed-point search for $y \mapsto \frac{x}{y^3}$ converge. On the other hand, if we average damp twice (i.e., use the average damp of the average damp of $y \mapsto \frac{x}{y^3}$) the fixed-point search does converge. Do some experiments to determine how many average damps are required to compute $n^{th}$ roots as a fixed-point search based upon repeated average damping of $y \mapsto \frac{x}{y^{n-1}}$. Use this to implement a simple procedure for computing $n^{th}$ roots using `fixed-point`, `average-damp`, and the `repeated` procedure of Exercise 1.43. Assume that any arithmetic operations you need are available as primitives.

Not necessary, but we can see a piece of abstraction between `nth-smoothed` and `nth-damped`
```scheme
(define (nth-applied f n g)
  (compose (repeated f n) g))  ; compose could be removed with no fuss
```
Using our previous best logic for exponents:
```scheme
(define (exp b n)
  (define (exp-iter a b n)
    (cond ((= n 0) a)
          ((even? n) (exp-iter a (* b b) (/ n 2)))
          (else (exp-iter (* a b) b (- n 1)))))
  (exp-iter 1 b n))
```
Using the following procedure to test different damp values:
```scheme
(define (nth-root-test n damps)
  (fixed-point (nth-applied average-damp
                            damps
                            (lambda (y) (/ (exp 2 n) (exp y (- n 1)))))
               1.0))
```
We can see that the minimum damps requires for each value of n are as follows:

| n     | damps |
| ----- | ----- |
| 1-3   | 1     |
| 4-7   | 2     |
| 8-15  | 3     |
| 16-31 | 4     |
So our value is `ln(n)`, rounded down.
```scheme
(define (nth-root x n)
  (let ((damps (floor (log n 2))))
    (fixed-point (nth-applied average-damp
                              damps
                              (lambda (y) (/ x (exp y (- n 1)))))
                1.0)))
```
##### Exercise 1.46
> Several of the numerical methods described in this chapter are instances of an extremely general computational strategy known as *iterative improvement*. Iterative improvement says that, to compute something, we start with an initial guess for the answer, test if the guess is good enough, and otherwise improve the guess and continue the process using the improved guess as the new guess. Write a procedure `iterative-improve` that takes two procedures as arguments: a method for telling whether a guess is good enough and a method for improving a guess. `iterative-improve` should return as its value a procedure that takes a guess as argument and keeps improving the guess until it is good enough. Rewrite the `sqrt` procedure of Section 1.1.7 and the `fixed-point` procedure of Section 1.3.3 in terms of `iterative-improve`.

For `sqrt`, I've elected to use the `good-enough?` from Exercise 1.7 and assume the initial guess is always 1.

```scheme
(define (iterative-improve eval-f impr-f)
  (define (iter-improve eval-f impr-f guess)
    (if (eval-f guess)
        guess
        (iter-improve eval-f impr-f (impr-f guess))))
  (lambda (guess) (iter-improve eval-f impr-f guess)))

(define tolerance 0.00001)
(define (sqrt x)
  (define (impr-f guess) (average guess (/ x guess)))
  ; (define (eval-f guess) (< (abs (- (square guess) x)) 0.001))
  (define (eval-f guess) (< (abs (/ (- (impr-f guess) guess) guess)) tolerance))
  ((iterative-improve eval-f impr-f) 1))

(define (fixed-point f start-guess)
  (define (impr-f guess) (f guess))
  (define (eval-f guess) (< (abs (- guess (impr-f guess))) tolerance))
  ((iterative-improve eval-f impr-f) start-guess))
```
