# Elements of Programming
## Expressions
Expressions representing numbers may be combined with an expression representing a primitive procedure (such as __+__ or __*__) to form a compound expression that represents the application of the procedure to those numbers.

```Scheme
(+ 2.7 10)
>>> 12.7
```
```Scheme
(+ 3 7 12 30)
>>> 52
```
```Scheme
(* 25 4 12)
>>> 1200
```
```Scheme
(+ (* 3 5) (- 10 6))
>>> 19
```
Even with complex expressions, the interpreter always operate in the some basic cycle: It reads an expression from terminal, evaluates the expression, and prints the result. This mode of operation is often expressed by saying that the interpreter runs in __read-eval-print loop__.
## Naming and the Environment
A critical aspect of a programming language is the means it provides for using names to refer to computational objects. We say that the name identifies a __variable__ whose __value__ is the object.
In Scheme, we name things with __define__.

```Scheme
(define size 2)
(* 5 size)
>>> 10
```
It should be clear that the possibility of associating values with symbols and later retrieving them means that the interpreter must maintain some sort of memory that keeps track of the name-object pairs. This memory is called the __environment__.
## Evaluating Combinations
In evaluating combinations, the interpreter is itself following a procedure. To evaluate a combination, do the following:

1. Evaluate the subexpression of the combination
2.  Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the value of the other subexpressions (the operands).

```Scheme
(* (+ 2 (* 4 6))
   (+ 3 5 7))
>>> 390
```
Symbols such as __+__ and __*__ are also in the global environment, and are associated with the sequences of machine instructions that are their "values". The key point to notice is the role of the environment in determining the meaning of the symbols in expressions. 

## Compound Procedures

```Scheme
(define (square x) (* x x))
```
We have here a compound procedure, which has been given the name square. 

```Scheme
(define (<name> <formal parameters>)
   (<body>))
```
```Scheme
(square 12)
(square (square 3))
(+ (square x) (square y))
```
## The Substitution Model for Procedure Application
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Normal-order evaluation vs Applicative-order evaluation</h3>
```Scheme
Applicative-order evaluation (evaluate the arguments and then apply)

(sum-of-squares (+ 5 1) (* 5 2))
(+ (square 6) (square 10))
(+ 36 100)
>>> 136
```
```Scheme
Normal-order evaluation (full expand and then reduce)

(sum-of-squares (+ 5 1) (* 5 2))
(+ (square (+ 5 1)) (square (* 5 2)))
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2)))
(+ (* 6 6) (* 10 10))
(+ 36 100)
>>> 136
```
Interpreter actually uses applicative order evaluation because of the additional efficiency obtained from avoiding multiple evaluations of expressions such as those illustrated with `(+ 5 1)` and `(* 5 2)` above.

## Conditional Expressions and Predicates

```Scheme
(define (abs x)
   (cond ((> x 0) x)
         ((= x 0) 0)
         ((> x 0) (- x))
         ))
```
```Scheme
(cond (<p1> <e1>)
      (<p2> <e2>)
         ...
      (<pN> <eN>)
      )
```
`(<p> <e>)` called __clauses__. The first expression in each pair is a __predicate__ -that is an expression whose value is interpreted as either true or false.

Conditional expression evaluated as follows: The predicate `<p1>` is evaluated first. If its value is false, then `<p2>` is evaluated. This process continues until a predicate is found whose value is true, in which case the interpreter returns the value of the corresponding expression `<e>` of the clause as the value of conditional expression. If none of the `<p>` is found to be true, the value of the `cond` is undefined. 

```Scheme
(define (sub x y) (- x y))
(define (add x y) (+ x y))
(define (arith t x y) (cond ((= t 1) (add x y))
                            ((= t 0) (sub x y))
                            ))

(arith 1 3 4)
>>> 7
(arith 0 3 4)
>>> -1
```
Another way to write absolute-value procedure is:
```Scheme
(define (abs x)
        (cond ((< x 0) (- x))
              (else x)
              ))
```
```Scheme
(define (abs x)
        (cond ((< x 0) (- x))
              ((> x 0) x)
              (else 0)
              ))
```
```Scheme
(define (abs x)
        (if (< x 0)
            (- x)
            x ))
```
This uses special form __if__, a restricted type of conditional that can be used when there are precisely two cases in the case analysis.
`(if <predicate> <consequent> <alternative>)`

In addition to primitive predicates such as __<__, __=__ and __>__, there are logical composition operations, which enable us to construct compound predicates.
```Scheme
(and <e1> ... <eN>)
```
```Scheme
(or <e1> ... <eN>)
```
```Scheme
(not <e1> ... <eN>)
```
The condition that a number x be in the range 5<x<10 may be expressed as:
```Scheme
(and (> x 5) (< x 10))
```
We can define predicate to test whether one number is greater than or equal to another as:
```Scheme
(define (>= x y) (= x y))
```
## Example: Square Roots by Newton's Method
Procedures, as introduced above, are much like ordinary mathematical functions. But there is an important difference between mathematical functions and computer procedures. Procedures must be effective.  
We can define the square-root function as √x = the y such that y ≥ 0 and y<sup>2</sup> = x  
This describes a perfectly legitimate mathematical function. On the other hand, the definition does not describe a procedure. Indeed, it tells us almost nothing about how to actually find the square root of a given number.  

The contrast between function and procedure is a reflection of the general distinction between describing properties of things and describing how to do things, or, as it is sometimes referred to, the distinction between declarative knowledge and imperative knowledge. In mathematics we are usually concerned with declarative (what is) descriptions, whereas in computer science we are usually concerned with imperative (how to) descriptions.  

Now let's formalize the process in terms of procedures.
```Scheme
(define (sqrt-iter guess x)
   (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x) x)))
```
```Scheme
(define (improve guess x)
   (average guess (/ x guess)))
```
```Scheme
(define (average x y)
   (/ (+ x y) 2))
```
```Scheme
(define (good-enough? guess x)
   (< (abs (- (square guess) x)) 0.001))
```
Finally, we need a way to get started.
```Scheme
(define (sqrt x)
   (sqrt-iter 1.0 x))
```
```Scheme
(sqrt 9)
>>>3.00009155413138

(sqrt (+ 100 37))
>>>11.704699917758145
```
```Scheme
Exercise 1.4:

(define (a-plus-abs-b a b)
        ((if (> b 0) + -) a b))

(a-plus-abs-b 2 -1)
>>> 3
(a-plus-abs-b 2 7)
>>> 9
```
## Procedures as Black-Box Abstractions
_Sqrt_ is our first example of a process defined by a set of mutually defined procedures. Notice that the definition of sqrt-iter is __recursive__; that is, the procedure is defined in terms of itself.  

Observe that the problem of computing square roots breaks up naturally into a number of subproblems: how to tell whether a guess is good enough, how to improve a guess, and so on. Each of these tasks is accomplished by a separate procedure. The entire _sqrt_ program can be viewed as a cluster of procedures that mirrors the decomposition of the problem into subproblems.  

It is crucial that each procedure accomplishes an identifiable task that can be used as a module in defining other procedures.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Local names</h3>
One detail of a procedure's implementation that should not matter to the user of the procedure is the implementer's choice of names for the procedure's formal parameters. Thus, the following procedures should not be distinguishable:
```Scheme
(define (square x) (* x x))
```
```Scheme
(define (square y) (* y y))
```
This principle—that the meaning of a procedure should be independent of the parameter names used by its author—seems on the surface to be self-evident, but its consequences are profound. The simplest consequence is that the parameter names of a procedure must be local to the body of the procedure.  

A formal parameter of a procedure has a very special role in the procedure definition, in that it doesn't matter what name the formal parameter has. Such a name is called a _bound variable_, and we say that the procedure definition binds its formal parameters. If a variable is not bound, we say that it is _free_. The set of expressions for which a binding defines a name is called the scope of that name. In a procedure definition, the bound variables declared as the formal parameters of the procedure have the body of the procedure as their scope.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Internal definitions and block structure</h3>

We have one kind of name isolation available to us so far: The formal parameters of a procedure are local to the body of the procedure. The existing square-root program consists of separate procedures:
```Scheme
(define (sqrt x)
   (sqrt-iter 1.0 x))
(define (sqrt-iter guess x)
   (if (good-enough? guess x)
        guess
        (sqrt-iter (improve guess x) x)))
(define (good-enough? guess x)
   (< (abs (- (square guess) x)) 0.001))
(define (improve guess x)
   (average guess (/ x guess)))
```
The problem with this program is that the only procedure that is important to users of sqrt is _sqrt_. The other procedures (_sqrt-iter_, _good-enough?_, and _improve_) only cluster up their minds. We would like to localize the subprocedures, hiding them inside _sqrt_ so that _sqrt_ could coexist with other successive approximations, each having its own private _good-enough?_ procedure. To make this possible, we allow a procedure to have internal definitions that are local to that procedure. For example, in square-root problem we can write:
```Scheme
(define (sqrt x)
   (define (good-enough? guess x)
      (< (abs (- (square guess) x)) 0.001))
   (define (improve guess x) (average guess (/ x guess)))
   (define (sqrt-iter guess x)
      (if (good-enough? guess x)
           guess
           (sqrt-iter (improve guess x) x)))
(sqrt-iter 1.0 x))

```
Such nesting of definitions, called _block structure_ is basically the right solution to the simplest name-packaging problem. In addition to internalizing the definitions of the auxiliary procedures, we can simplify them. Since _x_ is bound in the definition of _sqrt_, the procedures _good-enough?_, _improve_, and _sqrt-iter_, which are defined internally to _sqrt_, are in the scope of _x_. Thus it is not necessary to pass _x_ explicitly to each of these procedures. Instead, we allow _x_ to be a free variable in the internal definitions, as shown below. Then _x_ gets its value from the argument with which the enclosing procedure _sqrt_ is called. This is discipline is called _lexical scoping_.
```Scheme
(define (sqrt x)
   (define (good-enough? guess)
      (< (abs (- (square guess) x)) 0.001))
   (define (improve guess)
      (average guess (/ x guess)))
   (define (sqrt-iter guess)
      (if (good-enough? guess)
           guess
          (sqrt-iter (improve guess))))
  (sqrt-iter 1.00))
```
# Procedures and the Processes They Generate
A procedure is a pattern for local evolution of a computational process. In this section we will examine some common "shapes" for processes generated by simple procedures. We will also investigate the rates at which these processes consume the important computational resources of time and space.
##Linear Recursion and Iteration
There are many ways to compute factorials. One way is to make use of the observaton that n! is equal to n times (n-1)! for any positive integer n. Thus, we can compute n! by computing (n-1)! and multiplying the result by n. If we add the stipulation that 1! is equal to 1, this observation translates directly into a procedure:
```Scheme
(define (factorial n)
  (if (= n 1) 1
      (* n (factorial(- n 1)))
      )
  )
```
```Scheme
(factorial 6)
(* 6 (factorial 5))
(* 6 (* 5 (factorial 4)))
(* 6 (* 5 (* 4 (factorial 3))))
(* 6 (* 5 (* 4 (* 3 (factorial 2)))))
(* 6 (* 5 (* 4 (* 3 (* 2 (factorial 1))))))
(* 6 (* 5 (* 4 (* 3 (* 2 1)))))
(* 6 (* 5 (* 4 (* 3 2))))
(* 6 (* 5 (* 4 6)))
(* 6 (* 5 24))
(* 6 120)
720
```
We used substitution model to watch this procedure in action computing 6!.  
Now let's take a different perspective on computing factorials. We could describe a rule for computing n! by specifying that we first multiply 1 by 2, then multiply the result by 3, then by 4, and so on until we reach n. More formally, we maintaing a running product, together with a counter that counts from 1 up to n. We can describe the computation by saying that the counter and the product simultaneously change from one step to the next according to the rule
```Scheme
product <- counter * product
counter <- counter + 1
```
and stipulating that n! is the value of the product when the counter exceeds n.

```Scheme
(define (factorial n)
  (fact-iter 1 1 n)
  )
(define (fact-iter product counter max-count)
  (if (> counter max-count) product
      (fact-iter (* product counter) (+ counter 1) max-count)
      )
  )
```

```Scheme
(factorial 6)
(fact-iter 1 1 6)
(fact-iter 1 2 6)
(fact-iter 2 3 6)
(fact-iter 6 4 6)
(fact-iter 24 5 6)
(fact-iter 120 6 6)
(fact-iter 720 7 6)
```

Compare the two processes. From one point of view, they seem hardly different at all. Both compute the same mathematical function on the same domain, and each requires a number of steps proportional to n to compute n!. Indeed, both processes even carry out the same sequence of multiplications, obtaining the same sequence of partial products. On the other hand, when we consider the "shapes" of the two processes, we find that they evolve quite differently.  

Consider the first process. The substitution model reveals a shape of expansion followed by contraction. The expansion occurs as the process builds up a chain of deferred operations (in this case, a chain of multiplications). The contraction occurs as the operations are actually performed. This type of process, characterized by a chain of deferred operations, is called a _recursive_ process. Carrying out this process requires that the interpreter keep track of the operations to be performed later on. In the computation of n!, the length of the chain of deferred multiplications, and hence the amount of information needed to keep track of it, grows linearly with n (is proportional to n), just like the number of steps. Such a process is called a _linear recursive process_.  

By contrast, the second process does not grow and shrink. At each step, all we need to keep track of, for any n, are the current values of the variables _product_, _counter_, and _max-count_. We call this an _iterative process_. In computing n!, the number of steps required grows linearly with n. Such a process is called a _linear iterative process_.  

The contrast between the two processes can be seen in another way. In the iterative case, the program variables provide a complete description of the state of the process at any point. If we stopped the computation between steps, all we would need to do to resume the computation is to supply the interpreter with the values of the three program variables. Not so with the recursive process. In this case there is some additional "hidden" information, maintained by the interpreter and not contained in the program variables, which indicates "where the process is" in negotiating the chain of deferred operations. The longer the chain, the more information must be maintained.  

In contrasting iteration and recursion, we must be careful not to confuse the notion of a recursive process with the notion of a recursive procedure. When we describe a procedure as recursive, we are referring to the syntactic fact that the procedure definition refers (either directly or indirectly) to the procedure itself. But when we describe a process as following a pattern that is, say, linearly recursive, we are speaking about how the process evolves, not about the syntax of how a procedure is written. It may seem disturbing that we refer to a recursive procedure such as _fact-iter_ as generating an iterative process. However, the process really is iterative: Its state is captured completely by its three state variables, and an interpreter need keep track of only three variables in order to execute the process.  

One reason that the distinction between process and procedure may be confusing is that most implementations of common languages (including Ada, Pascal, and C) are designed in such a way that the interpretation of any recursive procedure consumes an amount of memory that grows with the number of procedure calls, even when the process described is, in principle, iterative. As a consequence, these languages can describe iterative processes only by resorting to special-purpose "loop-ing constructs" such as do, repeat, until, for, and while. The implementation of Scheme we shall consider in Chapter 5 does not share this defect. It will execute an iterative process in constant space, even if the iterative process is described by a recursive procedure. An implementation with this property is called _tail-recursive_. With a tail-recursive implementation, iteration can be expressed using the ordinary procedure call mechanism, so that special iteration constructs are useful only as syntactic sugar.

```Scheme
Exercise 1.9:

; Recursive process
(define (+ a b)
  (if (= a 0) b (inc (+ (dec a) b))))

(+ 4 5)
(inc (+ 3 5))
(inc (inc (+ 2 5)))
(inc (inc (inc (+ 1 5))))
(inc (inc (inc (inc (+ 0 5)))))
(inc (inc (inc (inc 5))))
(inc (inc (inc 6)))
(inc (inc 7))
(inc 8)
>>> 9 

; Iterative process
(define (+ a b)
  (if (= a 0) b (+ (dec a) (inc b))))

(+ 4 5)
(+ 3 6)
(+ 2 7)
(+ 1 8)
(+ 0 9)
>>> 9
```

## Tree Recursion
Another common pattern of computation is called _tree recursion_. As an example, consider computing the sequence of Fibonacci numbers, in which each number is the sum of the preceding two. 
```Scheme
(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1))
                 (fib (- n 2))))))
```
Consider the pattern of this computation. To compute (fib 5), we compute (fib 4) and (fib 3). To compute (fib 4) , we compute (fib 3) and (fib 2). In general, the evolved process looks like a tree. Notice that the branches split into two at each level (except at the bottom); this reflects the fact that the fib procedure calls itself twice each time it is invoked. This procedure is instructive as a prototypical tree recursion, but it is a terrible way to compute Fibonacci numbers because it does so much redundant computation.

## Orders of Growth
Processes can differ considerably in the rates at which they consume computational resources. One convenient way to describe this difference is to use the notion of _order of growth_ to obtain a gross measure of the resources required by a process as the inputs become larger.  

We say that R(n) has order of growth Θ(f(n)), written R(n) = Θ(f(n)) (pronounced "theta of f(n)"), if there are positive constants k<sub>1</sub> and k<sub>2</sub> independent of n such that k<sub>1</sub>f(n) ≤ R(n) ≤ k<sub>2</sub>f(n) for any sufficiently large value of n.  
For a Θ(n) (linear) process, doubling the size will roughly double the amount of resources used. For an exponential process, each increment in problem size will multiply the resource utilization by a constant factor.

## Exponentiation

Consider the problem of computing the exponential of a given number. One way to do this via the recursive definition  

b<sup>n</sup> = b . b<sup>n - 1</sup>  
b<sup>0</sup> = 1  

which translates readily into the procedure
```Scheme
(define (expt b n)
  (if (= n 0)
      1
      (* b (expt b (- n 1)))))
  
  
(expt 2 4)
(2 * (expt 2 3))
(2 * (2 * (expt 2 2)))
(2 * (2 * (2 * (expt 2 1))))
(2 * (2 * (2 * (2 * (expt 2 0)))))
(2 * (2 * (2 * (2 * 1))))
(2 * (2 * (2 * 2)))
(2 * (2 * 4))
(2 * 8)
>>> 16

```
This is a linear recursive process, which requires Θ(n) steps and Θ(n) space.  

We can readily formulate an equivalent linear iteration
```Scheme
(define (expt b n)
  (expt-iter b n 1))
(define (expt-iter b counter product)
  (if (= counter 0)
      product
      (expt-iter b
                 (- counter 1)
                 (* b product))))
                 
(expt 2 4)
(expt-iter 2 4 1)
(expt-iter 2 3 2)
(expt-iter 2 2 4)
(expt-iter 2 1 8)
(expt-iter 2 0 16)
>>> 16
```

This version requires Θ(n) steps and Θ(1) space.  

We can also take advantage of succesive squaring in computing exponentials in general if we use the rule  

b<sup>2</sup> = (b<sup>n / 2</sup>)<sup>2</sup>      if n is even,  
b<sup>2</sup> = b * (b<sup>n - 1</sup>)      if n is odd  

We can express this method as a procedure: 
```Scheme
(define (fast-expt b n)
  (cond ((= n 0) 1)
        ((even? n) (square (fast-expt b (/ n 2))))
        (else (* b (fast-expt b (- n 1))))))
```

The process evolved by fast-expt grows logarithmically with n in both space and number of steps. To see this, observe that computing b<sup>2n</sup> using _fast-expt_ requires only one more multiplication than computing b<sup>n</sup>. The size of the exponent we can compute therefore doubles (approximately) with every new multiplication we are allowed. Thus, the number of multiplications required for an exponent of n grows about as fast as the logarithm of n to the base 2. The process has Θ(log n) growth.

# Formulating Abstractions with Higher-Order Procedures

We have seen that procedures are abstractions that describe compound operations on numbers independent of the particular numbers. When we `(define (cube x) (* x x x))` we are not talking about the cube of a particular number, but rather about a method for obtaining the cube of any number. One of the things we should demand from a powerful programming language is the ability to build abstractions by assigning names to common patterns and then to work in terms of the abstractions directly. Procedures provide this ability.  

Yet even in numerical processing we will be severely limited in our ability to create abstractions if we are restricted to procedures whose parameters must be numbers. Ofen the same programming pattern will be used with a number of different procedures. To express such patterns as concepts, we will need to construct procedures that can accept procedures as arguments or return procedures as values. Procedures that manipulate procedures are called _higher-order procedures_.

## Procedures as Arguments
The power of sigma notation is that it allows mathematicians to deal with the concept of summation itself rather than only with particular sums—for example, to formulate general results about sums that are independent of the particular series being summed. As program designers, we would like our language to be powerful enough so that we can write a procedure that express the concept of summation itself rather than only procedures that compute particular sum.  

We can do so readily in our procedural language
```Scheme
(define (sum term a next b)
(if (> a b)
    0
    (+ (term a)
       (sum term (next a) next b))))
```

Notice that _sum_ takes as its arguments the lower and upper bounds _a_ and _b_ together with the procedures _term_ and _next_. We can use _sum_ just as we would any procedure. For example, we can use it (along with a procedure _inc_ that increments its argument by 1) to define _sum-cubes_.
```Scheme
(define (cube x) (* x x x))
(define (inc n) (+ n 1))
(define (sum-cubes a b)
   (sum cube a inc b))
```
Using this, we can compute the sum of the cubes of the integers from 1 to 10.
```Scheme
(sum-cubes 1 10)
>>> 3025
```
With the aid of an _identity_ procedure to compute the term, we can define _sum-integers_ in terms of _sum_.
```Scheme
(define (identity x) x)
(define (inc n) (+ n 1))
(define (sum-integers a b)
   (sum identity a inc b))
```
```Scheme
(sum-integers 1 10)
>>> 55
```

## Constructing Procedures Using _Lambda_

Using _lambda_ we can describe what we want as 
```Scheme
(lambda (x) (+ x 4))
```
Then our _sum-cubes_ procedure can be expressed without defining any auxiliary procedures, such as _inc_ as
```Scheme
(define (sum term a b)
(if (> a b)
    0
    (+ (term a)
       (sum term ((lambda (x) (+ 1 x)) a) b))))
(define (cube a) (* a a a))
(define (sum-cubes a b)
   (sum cube a b))
```
In general, _lambda_ is used to create procedures in the same way as _define_, except that no name is specified for the procedure:
```Scheme
(lambda (<formal parameters>)
   <body>)
```
The resulting procedure is just as much a procedure as one that is created using _define_. The only difference is that has not been associated with any name in the environment.  

In fact, `(define (plus4 x) (+ x 4))` is equivalent to `(define plus4 (lambda (x) (+ x 4)))`. We can read lambda expression as "the procedure of an argument x that adds x and 4".  

We can use lambda expression in any contect where we would normally use a procedure name. 
```Scheme
((lambda (x y z) (+ x y z)) 1 2 3)
>>> 6
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Using let to create local variables</h3>
Another use of _lambda_ is in creating local variables. We often need local variables in our procedures other than those that have been bound as formal parameters. Suppose we wish to compute f(x,y) where  
a = 1 + xy  
b = 1 - y  
f(x,y) = xa<sup>2</sup> + yb + ab  

Using _let_ the f procedure can be written as
```Scheme
(define (f x y)
  (let ((a (+ 1 (* x y)))
        (b (- 1 y)))
    (+ (* x (square a))
       (* y b)
       (* a b))))
       
(f 1 2)
>>> 4
```
The general form of a _let_ expression is
```Scheme
(let ((<var1> <exp1>)
      (<var2> <exp2>)
      ...
      (<varN> <expN>))
  <body>)
```

When the _let_ is evaluated, each _name_ is associated with the value of the corresponding _expression_. The body of the _let_ is evaluated with these _names_ bound as local variables. The scope of a variable specified by a _let_ expression is the body of the _let_.  
So the result of the following expression will be 38.
```Scheme
(let ((x 5))
(+ (let ((x 3))
     (+ x (* x 10))
     )
   x
   ))
```
Sometimes we can use internal definitions to get the same effect as with _let_.
```Scheme
(define (f x y)
  (define a (+ 1 (* x y)))
  (define b (- 1 y))
  (+ (* x (square a))
     (* y b)
     (* a b)))
```

## Procedures as General Methods

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Finding roots of equations by the half-interval method</h3>
The half-interval method is a simple but powerful technique for finding roots of an equation f(x) = 0, where f is a continuous function. The idea is that, if we are given points a and b such that f(a) < 0 < f(b), then f must have at least one zero between a and b. To locate a zero, let x be the average of a and b, and compute f (x). If f(x) > 0, then f must have a zero between a and x. If f(x) < 0, then f must have a zero between x and b. Continuing in this way, we can identify smaller
and smaller intervals on which f must have a zero. When we reach a point where the interval is small enough, the process stops.
```Scheme
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
```
We assume that we are initially given the function f together with points at which its values are negative and positive. We first compute the midpoint of the two given points. Next we check to see if the given interval is small enough, and if so we simply return the midpoint as our answer. Otherwise, we compute as a test value the value of f at the midpoint. If the test value is positive, then we continue the process with a new interval running from the original negative point to the midpoint. If the test value is negative, we continue with the interval from the midpoint to the positive point. Finally, there is the possibility that the test value is 0, in which case the midpoint is itself the root we are searching for.  
To test whether the endpoints are "close enough" we can use the following procedure:
```Scheme
(define (close-enough? x y) (< (abs (- x y)) 0.001))
```
Search is awkward to use directly, because we can accidentally give it points at which f's values do not have the required sign, in which case we get a wrong answer. Instead we will use search via the following procedure, which checks to see which of the endpoints has a negative function value and which has a positive value, and calls the search procedure accordingly. If the function has the same sign on the two given points, the half-interval method cannot be used, in which case the procedure signals an error.
```Scheme
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
```Scheme
(half-interval-method (lambda (x) (- (* x x x) (* 2 x) 3))
                      1.0
                      2.0)
>>> 1.89306640625
```
## Procedures as Returned Values

The above examples demonstrate how the ability to pass procedures as arguments significantly enhances the expressive power of our programming language. We can achieve even more expressive power by creating procedures whose returned values are themselves procedures.

_average-damp_ is a procedure that takes as its argument a procedure _f_ and returns as its value a procedure (produced by the lambda) that, when applied to a number x,  produces the average of x and (f x). For example, applying _average-damp_ to the _square_ procedure produces a procedure whose value at some number x is the average of x and x<sup>2</sup>.
```Scheme
(define (square x) (* x x))
(define (average x y) (/ (+ x y) 2))
(define (average-damp f)
  (lambda (x) (average x (f x))))
  
((average-damp square) 10)
>>> 55
```
```Scheme
(define (cube x) (* x x x))
(define (average x y) (/ (+ x y) 2))
(define (average-damp f)
  (lambda (x) (average x (f x))))
  
((average-damp cube) 10)
>>> 505
```

```Scheme
Exercise 1.41:

(define (inc x) (+ x 1))
(define (double f)
  (lambda (x) (f (f x))))
  
(((double (double double)) inc) 5)
>>> 21

((double (double (double (double inc)))) 5)
>>> 21
```



























