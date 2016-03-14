---
layout: post
title: SICP selected exercises
image: sicp.png
date:  2016-02-12 12:00:00
---
<p class="intro"> <span class="dropcap">S</span>olutions to selected problems in sicp </p>

**Exercise 1.5** Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:

```
(define (p) (p))

(define (test x y)
  (if (= x 0)
      0
      y))
```      
Then he evaluates the expression

```
(test 0 (p))
```


**Answer**  For applicative order, the expression is going to be an infinite loop while for normal order it is going to return 0

Order of execution for normal order

```
(test 0 (p))
;;Reduces to 

(if (= 0 0)
    0
   (p)))
```   
   

Since predicate is true, the expression ```(p)``` is never evaluated, 

For applicative order, the expression reduces to 

```
(test 0 (p))
```
Now the interpreter tries to find the value of both 0 and (p) before substituting the expressions in ```(if (= x 0) 0 y)```

0 is a primitive and resolves to 0, while (p) is an infinite loop and never ends.

**Exercise 1.6.**  Alyssa P. Hacker doesn't see why if needs to be provided as a special form. ``Why can't I just define it as an ordinary procedure in terms of cond?'' she asks. Alyssa's friend Eva Lu Ator claims this can indeed be done, and she defines a new version of if:

```(define (new-if predicate then-clause else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```        
Eva demonstrates the program for Alyssa:

```
(new-if (= 2 3) 0 5)
5

(new-if (= 1 1) 0 5)
0
```

Delighted, Alyssa uses new-if to rewrite the square-root program:

```
(define (sqrt-iter guess x)
  (new-if (good-enough? guess x)
          guess
          (sqrt-iter (improve guess x)
                     x)))
```

What happens when Alyssa attempts to use this to compute square roots? 

Alyssa’s solution enters an infinite loop. This is because lisp reduces the function in applicative order. Since new-if is not a special form , 
before applying new-if, lisp first evaluates (good-enough? Guess x) 
Guess 
And
 
```(sqrt-iter (improve guess x)
                     x)```
the above is a recursive function with no terminating condition and hence the entire function enters an infinite loop

**Exercise 1.10.**  The following procedure computes a mathematical function called Ackermann's function.

```
(define (A x y)
  (cond ((= y 0) 0)
        ((= x 0) (* 2 y))
        ((= y 1) 2)
        (else (A (- x 1)
                 (A x (- y 1))))))
```                 
What are the values of the following expressions?

```
(A 1 10)
(A 2 4)
(A 3 3)
```
Consider the following procedures, where A is the procedure defined above:

```
(define (f n) (A 0 n))

(define (g n) (A 1 n))

(define (h n) (A 2 n))

(define (k n) (* 5 n n)) ;; where (k n) = 5n^2
```




**Solution:**

```
     (A 1 10) = (A 0 (A 1 9))
              = 2* (A 1 9)
              = 2 * 2 * 2…. Ten times
              = 2 ^ 10
              = 1024
    
     (A 2 4)  = (A 1 (A 2 3))
              =  2 ^ (A 2 3)
              =  2 ^ 2 ^ (A 2 2)
              =  2 ^ 2 ^ 2 ^ (A 2 1)
              = 2 ^ 2 ^ 2 ^ 2
              = 2 ^ 16
              = 65536
              
              
     (A 3 3)  = (A 2 (A 3 2))
              =  2 ^ 2 ^ 2 ... (A 3 2) times
              =  2 ^ 2 ^ 2 .. (A 2 (A 3 1)) times
              =  2 ^ 2 ^ 2 .. 4 times
              =  2 ^ 16
              = 65536
                       
     (f n)    = (A 0 n)
              =  2n
              
              
     (g n)    = (A 1 n)
              = (A 0 (A 1 (- n 1)))
              = (f (A 1 (- n 1)))
              = (f (g (- n 1) ))
              = 2 x (g (- n 1))
              = 2 x 2 x (g (- n 2))
              = 2 x 2 x 2 ... n times
              = 2 ^ n
              
              
     (h n)    = (A 2 n)
              = (A 1 (A 2 (- n 1)))
              = (g (A 2 (- n 1))))
              = (g (h (- n 1)))
              = 2 ^ h (- n 1)
              = 2 ^ 2 ^ h (- n 2)
              = 2 ^ 2 ^ 2 .....n times
    
               

```


**Exercise 1.34**  Suppose we define the procedure


```
(define (f g)
  (g 2))
```
Then we have

```
(f square)
4

(f (lambda (z) (* z (+ z 1))))
6
```

What happens if we (perversely) ask the interpreter to evaluate the combination ```(f f)```?

Solution:

We will get an exception

```(f f)``` resolves to ```(2 2)``` . For lisp interpreter any expression of the form ```(a b)```, will try 
to execute the procedure *a* with argument *b*. In this case, *a* is 2 which is a primitive and not a procedure with one argument.




**Exercise 1.37.**  

a. An infinite continued fraction is an expression of the form


As an example, one can show that the infinite continued fraction expansion with the Ni and the Di all equal to 1 produces 1/, where  is the golden ratio (described in section 1.2.2). One way to approximate an infinite continued fraction is to truncate the expansion after a given number of terms. Such a truncation -- a so-called k-term finite continued fraction -- has the form


Suppose that n and d are procedures of one argument (the term index i) that return the Ni and Di of the terms of the continued fraction. Define a procedure cont-frac such that evaluating (cont-frac n d k) computes the value of the k-term finite continued fraction. Check your procedure by approximating 1/ using

```
(cont-frac (lambda (i) 1.0)
           (lambda (i) 1.0)
           k)
```
for successive values of k. How large must you make k in order to get an approximation that is accurate to 4 decimal places?

b. If your cont-frac procedure generates a recursive process, write one that generates an iterative process. 
If it generates an iterative process, write one that generates a recursive process.

**Exercise 1.38.** 
 In 1737, the Swiss mathematician Leonhard Euler published a memoir De Fractionibus Continuis, 
 which included a continued fraction expansion for e - 2, where e is the base of the natural logarithms. 
 In this fraction, the Ni are all 1, and the Di are successively 1, 2, 1, 1, 4, 1, 1, 6, 1, 1, 8, ....
 Write a program that uses your cont-frac procedure from exercise 1.37 to approximate e, based on Euler's expansion.
 
 **Solution** for exercice 1.38 depends on the solution for 1.37

 
```
 (define cont-frac 
   (lambda (nfn dfn k)
     (define cont-frac-iter 
 	     (lambda  (index nfn dfn k)
 	       (if (> index k) 
 		   0
 		   (/  (nfn index) 
 		       (+ (dfn index) 
 			  (cont-frac-iter (+ index 1) nfn dfn k ))))))
     (cont-frac-iter 1 nfn dfn k)))
 
 (define cont-frac-iterative
   (lambda (nfn dfn k)
     (define cfi-iter 
       (lambda (index sum nfn dfn)
 	(if (< index 1) 
 	    sum
 	    (cfi-iter (- index 1)  (/ (nfn index) (+ sum (dfn index)))  nfn dfn))))
       (cfi-iter k 0 nfn dfn)))

 (define defrac (lambda (k)
   (cont-frac-iterative 
    (lambda (i) 1.0)
    (lambda (i) 1.0)
    k)))
 
 (define find-reqd-iterations 
   (lambda (precision)
     (define iteration-iter 
       (lambda (iteration)
 	(let ((val1 (defrac (+ iteration 1)))
 	      (val2 (defrac iteration)))
 	  (if (or (> (abs (- 
 			   (ceiling (/ val1 precision)) 
 			   (ceiling (/ val2 precision))))
 		     0)
 		  (= (abs (* val1 (/ 1 precision)))
 		     (ceiling (abs (* val1 (/ 1 precision ))))))
 	      (iteration-iter (+ iteration 1))
 	      iteration))))
     (iteration-iter 1)))
```
 
 Now to solver for euler we will first find out e-2 and add e to the resulting sum. e -2 can be calculate from the above cont-frac function
 by substituting the value of denominator
 
```
(define find-e 
  (lambda (iterations)
    (define euler-denominator 
      (lambda (index)
	(if (= 0 (modulo (+ 1 index) 3)) ;; every third number is multiple of 2 if we start counting the the first index is 2, every index divisible by 3 is a multiple of 2
	    (* 2 (/ (+ index 1) 3))
	    1)))
    (+ 2
       (cont-frac-iterative 
	(lambda (i) 1.0);;this is the numerator function
	euler-denominator ;; this resolves to 1,2,1,1,4,1,1,6
	iterations))))
```

 **Exercise 1.41.**  Define a procedure double that takes a procedure of one argument as argument and returns a procedure that
  applies the original procedure twice. For example, if ```inc``` is a procedure that adds 1 to its argument, then ```(double inc)```
  should be a procedure that adds 2. What value is returned by ```(((double (double double)) inc) 5)```
  
  **Solution** 
  
```
  (define double 
    (lambda (proc)
      (lambda (val)
        (proc (proc val)))))
         
  (((double (double double)) inc) 5) = (((double (double double)) inc) 5)
                                       (((double (lambda (val) (double (double val))) ) inc) 5)
                                       (((double g) inc) 5) ;; g = (lambda (val) (double (double val)) )
                                       ((g (g inc)) 5)
                                       ((g (double (double inc))) 5)
                                       ((g (double (lambda (x) (inc (inc x))))) 5)
                                       ((g (double (lambda (x) (+ 1 (+ 1 x))))) 5)
                                       ((g (double (lambda (x) (+ 2 x) ))) 5)
                                       (( g (lambda (x) (+ 2 (+ 2 x)) )) 5)
                                       ((g (lambda (x) (+ 4 x))) 5)
                                       ((double (double (lambda (x) (+ 4 x)))) 5)
                                       ((double (lambda (x) (+ 4 (+ 4 x)))) 5)
                                       ((double (lambda (x) (+ 8 x))) 5)
                                       ((lambda (x) (+ 8 (+ 8 x)) 5)
                                       ((lambda (x) (+ 16 x)) 5)

```
  
  
  **Exercise 1.42.**  Let f and g be two one-argument functions. The composition f after g is defined to be the function x  f(g(x)).
  
  Solution
  
```
  (define compose 
    (lambda (procouter procinner)
      (lambda (x)
        (procouter 
         (procinner x)))))      
```


**Exercise 1.43.**  If f is a numerical function and n is a positive integer, then we can form the nth repeated application of f, which is defined to be the function whose value at x is f(f(...(f(x))...)). Exercise 1.43.  If f is a numerical function and n is a positive integer, then we can form the nth repeated application of f, which is defined to be the function whose value at x is f(f(...(f(x))...)). 

**Solution**


```
 (define repeated
   (lambda (proc repeats)
     (lambda (x)
       (define proc-iter
 	(lambda (index val-till-now)
 	  (if (= index repeats)
 	      val-till-now
 	      (proc-iter (+ index 1) (proc val-till-now) ))))
       (proc-iter 1 (proc x)))))
```
  
         