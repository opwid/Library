# Introduction to Data Abstraction
The analogous notion for compound data is called _data abstraction_. Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects.  
The basic idea of data abstraction is to structure the programs that are to use compound data objects so that they operate on "abstract data".That is, our programs should use data in such a way as to make no assumptions about the data that are not strictly necessary for performing the task at hand. At the same time, a "concrete" data representation is defined independent of the programs that use the data. The interface between these two parts of our system will be a set of procedures, called _selectors_ and _constructors_, that implement the abstract data in terms of the concrete representation. To illustrate this technique, we will consider how to design a set of procedures for manipulating rational numbers.
## Example: Arithmetic Operations for Rational Numbers
Suppose we want to do arithmetic with rational numbers. Let us begin by assuming that we already have a way of constructing a rational number from a numerator and a denominator. We also assume that, given a rational number, we have a way of extracting (or selecting) its numerator and its denominator. Let us further assume that the constructor and selectors are available as procedures:

* (make-rat n d) returns the rational number whose numerator is the integer n and whose denominator is the integer d.
* (numer x) returns the numerator of the rational number x.
* (denom x) returns the denominator of the rational number x.

We are using here a powerful strategy of synthesis: _wishful thinking_. We haven't yet said how a rational number is represented, or how the procedures _numer_, _denom_, and _make-rat_ should be implemented.  
We can express arithmetic rules of rational numbers as procedures:
```Scheme
(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (sub-rat x y)
  (make-rat (- (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))
            
(define (div-rat x y)
  (make-rat (* (numer x) (denom y))
            (* (denom x) (numer y))))

(define (equal-rat? x y)
  (= (* (numer x) (denom y))
     (* (numer y) (denom x))))
```
Now we have the operations on rational numbers defined in terms of the _selector_ and _constructor_ procedures _numer_, _denom_, and _make-rat_. But we haven't yet defined these. What we need is some way to glue together a numerator and a denominator to form a rational number.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Pairs</h3>
To enable us to implement the concrete level of our data abstraction, our language provides a compound structure called a _pair_, which can be constructed with the primitive procedure _cons_. This procedure takes two arguments and returns a compound data object that contains the two arguments as parts. Given a pair, we can extract the parts using the primitive procedures _car_ and _cdr_.
```Scheme
(define x (cons 1 2))

(car x)
>>> 1
(cdr x)
>>> 2
```
```Scheme
(define x (cons 1 2))
(define y (cons 3 4))
(define z (cons x y))

(cdr (car z))
>>> 2
(car (cdr z))
>>> 3
```
The single compound-data primitive pair, implemented by the procedures cons, car, and cdr, is the only glue we need. Data objects constructed from pairs are called _list-structured_ data.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Representing rational numbers</h3>
make-rat, numer, and denom are readily implemented as follows:
```Scheme
(define (make-rat n d) (cons n d))
(define (number x) (car x))
(define (denom y) (cdr y))
```
Also, in order to display the results of our computations, we can print rational numbers by printing the numerator, a slash, and the denominator:
```Scheme
(define (print-rat x)
  (newline)
  (display (numer x))
  (display "/")
  (display (denom x)))
```
Now we can try our rational-number procedures:
```Scheme
(define one-half (make-rat 1 2))
(print-rat one-half)
>>> 1/2
(define one-third (make-rat 1 3))
(print-rat (add-rat one-half one-third))
>>> 5/6
```

## Abstraction Barriers
add-rat, sub-rat, mul-rat, div-rat, and equal-rat? are implemented solely in terms of the constructor and selectors make-rat, numer, and denom, which themselves are implemented in terms of pairs. In effect, procedures at each level are the interfaces that define the abstraction barriers and connect the different levels.  
Constraining the dependence on the representation to a few interface procedures helps us design programs as well as modify them, because it allows us to maintain the flexibility to consider alternate implementations.
## What is Meant by Data?
But exactly what is meant by data? It is not enough to say "whatever is implemented by the given selectors and constructors". Clearly, not every arbitrary set of three procedures can serve as an appropriate basis for the rational-number implementation. We need to guarantee that, if we construct a rational number x from a pair of integers n and d, then extracting the numer and the denom of x and dividing them should yield the same result as dividing n by d.  

In fact, this is the only condition make-rat, numer, and denom must fulfill in order to form a suitable basis for a rational number representation. In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation.  

Consider the notion of a pair, which we used in order to define our rational numbers. We never actually said what a pair was, only that the language supplied procedures cons, car, and cdr for operating on pairs. But the only thing we need to know about these three operations is that if we glue two objects together using cons we can retrieve the objects using car and cdr. That is, the operations satisfy the condition that, for any objects x and y, if z is (cons x y) then (car z) is x and (cdr z) is y.  

However, any triple of procedures that satisfies the above condition can be used as the basis for implementing pairs.
```Scheme
(define (cons x y)
  (define (dispatch m)
    (cond ((= m 0) x)
          ((= m 1) y)
          (else (error "Argument not 0 or 1: CONS" m))))
  dispatch)
(define (car z) (z 0))
(define (cdr z) (z 1))

(define one-third (cons 1 3))
(car one-third)
>>> 1
```
The subtle point to notice is that the value returned by (cons x y) is a procedure —namely the internally defined procedure _dispatch_, which takes one argument and returns either x or y depending on whether the argument is 0 or 1. Correspondingly, (car z) is defined to apply z to 0. Hence, if z is the procedure formed by (cons x y), then z applied to 0 will yield x. Thus, we have shown that (car (cons x y)) yields x, as desired. Similarly, (cdr (cons x y)) applies the procedure returned by (cons x y) to 1, which returns y. Therefore, this procedural implementation of pairs is a valid implementation, and if we access pairs using only cons, car, and cdr we cannot distinguish this implementation from one that uses "real" data structures.  

This example also demonstrates that the ability to manipulate procedures as objects automatically provides the ability to represent compound data. This may seem a curiosity now, but procedural representations of data will play a central role in our programming repertoire. This style of programming is often called _message passing_.

```Scheme
Exercise 2.4:

(define (cons x y)
  (lambda (m) (m x y)))
(define (car z)
  (z (lambda (p q) p)))
(define (cdr z)
  (z (lambda (p q) q)))

(define one-third (cons 1 3))
(cdr one-third)
>>> 3
```

# Hierarchical Data and the Closure Property
The ability to create pairs whose elements are pairs is the essence of list structure's importance as a representational tool. We refer to this ability as the _closure property_ of cons. In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation. Closure is the key to power in any means of combination because it permits us to create _hierarchical_ structures —structures made up of parts, which themselves are made up of parts, and so on.
```Scheme
(cons (cons 1 2)
      (cons 3 4))
```
## Representing Sequences
One of the useful structures we can build with pairs is a _sequence_—an ordered collection of data objects. There are many ways to represent sequences in terms of pairs. One particular sequence is constructed by nested cons operations. The car of each pait is the corresponding item in the chain, and the cdr of the pair is the next paint in the chain. The value of nil, used to terminate the chain of pairs, can be thought of as a sequence of no elements, the empty list.
![SEQUENCE](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/sequence.png) 
```Scheme
(cons 1
      (cons 2
            (cons 3
                  (cons 4 nil))))
```

Such a sequence of pairs, formed by nested conses, is called a _list_, and Scheme provides a primitive called list to help in constructing lists. The above sequence could be produced by (list 1 2 3 4).
```Scheme
(list <a1> <a2> <a3> ... <aN>)
```
Lisp systems conventionally print lists by printing the sequence of elements, enclosed in parentheses.
```Scheme
(define one-through-four (list 1 2 3 4))

one-through-four
>>> (1 2 3 4)
```
We can think of car as selecting the first item in the list, and of cdr as selecting the sublist consisting of all but the first item. Nested applications of car and cdr can be used to extract the second, third, and subsequent items in the list. The constructor cons makes a list like the original one, but with an additional item at the beginning.
```Scheme
(car one-through-four)
>>> 1
```
```Scheme
(cdr one-through-four)
>>> (2 3 4)
```
```Scheme
(car (cdr one-through-four))
>>> 2
```
```Scheme
(cons 10 one-through-four)
>>> (10 1 2 3 4)
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;List operations</h3>
The procedure _list-ref_ takes as arguments a list and a number n and returns the n<sup>th</sup> item of the list.
```Scheme
(define (list-ref items n)
  (if (= n 0)
      (car items)
      (list-ref (cdr items) (- n 1))))

(define squares (list 1 4 9 16 25))
(list-ref squares 3)
>>> 16
```
Often we cdr down the whole list. To aid in this, Scheme includes a primitive predicate _null?_, which tests whether its argument is the empty list. The procedure _length_, which returns the number of items in a list, illustrates this typical pattern of use:
```Scheme
(define (length items)
  (if (null? items)
      0
      (+ 1 (length (cdr items)))))

(define odds (list 1 3 5 7))
(length odds)
>>> 4
```
Another conventional programming technique is to "cons up" an answer list while cdring down a list, as in the procedure append, which takes two lists as arguments and combines their elements to make a new list:
```Scheme
(append squares odds)
>>> (1 4 9 16 25 1 3 5 7)

(append odds squares)
>>> (1 3 5 7 1 4 9 16 25)
```
```Scheme
(define (append list1 list2)
  (if (null? list1)
      list2
      (cons (car list1) (append (cdr list1) list2))))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Mapping over lists</h3>
One extremely useful operation is to apply some transformation to each element in a list and generate the list of results. _Map_ takes as arguments a procedure of one argument and a list, and returns a list of the results produced by applying the procedure to each element in the list:
```Scheme
(map (lambda (x) (+ x 1)) (list 1 2 3 4))
>>> (2 3 4 5)
```

## Hierarchical Structures
The representation of sequences in terms of lists generalizes naturally to represent sequences whose elements may themselves be sequences. For example, we can regard the object ((1 2) 3 4) constructed by
```Scheme
(cons (list 1 2) (list 3 4))
```
as a list of three items, the first of which is itself a list, (1 2).  

Another way to think of sequences whose elements are sequences is as _trees_. The elements of the sequence are the branches of the tree, and elements that are themselves sequences are subtrees. Recursion is a natural tool for dealing with tree structures, since we can often reduce operations on trees to operations on their branches, which reduce in turn to operations on the branches of the branches, and so on, until we reach the leaves of the tree.  

As an example, we can implement count-leaves procedure. To aid in writing recursive procedure on trees, Scheme provides the primitive predicate _pair?_, which tests whethet its argument is a pair. Here is the complete procedure:
```Scheme
(define (count-leaves x)
  (cond ((null? x) 0)
        ((not (pair? x)) 1)
        (else (+ (count-leaves (car x))
                 (count-leaves (cdr x))))))
```
```Scheme
Exercise 2.26:

(define x (list 1 2 3))
(define y (list 4 5 6))

(append x y)
>>> (1 2 3 4 5 6)
(cons x y)
>>> ((1 2 3) 4 5 6)
(list x y)
>>> ((1 2 3) (4 5 6))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Mapping over trees</h3>
_map_ together with recursion is a powerful abstraction for dealing with trees. For instance, the scale-tree procedure takes as arguments a numeric factor and a tree whose leaves are numbers. It returns a tree of the same shape, where each number is multiplied by the factor.
```Scheme
(define (scale-tree tree factor)
  (cond ((null? tree) nil)
        ((not (pair? tree)) (* tree factor))
        (else (cons (scale-tree (car tree) factor)
                    (scale-tree (cdr tree) factor)))))

(scale-tree (list 1 (list 2 (list 3 4) 5) (list 6 7)) 10)
>>> (10 (20 (30 40) 50) (60 70))
```
Another way to implement scale-tree is to regard the tree as a sequence of sub-trees and use map.

```Scheme
(define (scale-tree tree factor)
  (map (lambda (sub-tree)
         (if (pair? sub-tree)
             (scale-tree sub-tree factor)
             (* sub-tree factor)))
       tree))
```

## Sequences as Conventional Interfaces
In this section, we introduce another powerful design principle for working with data structures—the use of _conventional interfaces_.  

Our ability to formulate analogous operations for working with compound data depends crucially on the style in which we manipulate our data structures. Consider, for example, the following procedure which takes a tree as argument and computes the sum of the squares of the leaves that are odd:
```Scheme
(define (sum-odd-squares tree)
  (cond ((null? tree) 0)
        ((not (pair? tree))
         (if (odd? tree) (square tree) 0))
        (else (+ (sum-odd-squares (car tree))
                 (sum-odd-squares (cdr tree))))))
```
In sum-odd-squares, we begin with an _enumerator_, which generates a "signal" consisting of the leaves of a given tree. This signal is passed through a _filter_, which eliminates all but the odd elements. The resulting signal is in turn passed through a _map_, which is a "transducer" that applies the square procedure to each element. The output of the map is then fed to an _accumulator_, which combines the elements using +, starting from an initial 0.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Sequence Operations</h3>
The key to organizing programs so as to more clearly reflect the signal-flow structure is to concentrate on the "signals" that flow from one stage in the process to the next. If we represent these signals as lists, then we can use list operations to implement the processing at each of the stages. For instance, we can implement the mapping stages using the map procedure.

```Scheme
(map square (list 1 2 3 4 5))
>>> (1 4 9 16 25)
```
Filtering a sequence to select only those elements that satisfy a given predicate is accomplished by
```Scheme
(define (filter predicate sequence)
  (cond ((null? sequence) nil)
        ((predicate (car sequence))
         (cons (car sequence)
               (filter predicate (cdr sequence))))
        (else (filter predicate (cdr sequence)))))
```
```Scheme
(filter odd? (list 1 2 3 4 5))
>>> (1 3 5)
```
Accumulations can be implemented by
```Scheme
(define (accumulate op initial sequence)
  (if (null? sequence)
      initial
      (op (car sequence)
          (accumulate op initial (cdr sequence)))))
```
```Scheme
(accumulate + 0 (list 1 2 3 4 5))
```
To enumerate the leaves of a tree, we can use
```Scheme
(define (enumerate-tree tree)
  (cond ((null? tree) nil)
        ((not (pair? tree)) (list tree))
        (else (append (enumerate-tree (car tree))
                      (enumerate-tree (cdr tree))))))

(enumerate-tree (list 1 (list 2 (list 3 4)) 5))
>>> (1 2 3 4 5)
```
Now we can reformulate sum-odd-squares
```Scheme
(define (sum-odd-squares tree)
  (accumulate
   + 0 (map square (filter odd? (enumerate-tree tree)))))
```
The value of expressing programs as sequence operations is that this helps us make program designs that are modular, that is, designs that are constructed by combining relatively independent pieces.  

Sequence operations provide a library of standard program elements that we can mix and match. For instance, we can reuse pieces from the sum-odd-squares procedure in a program that constructs a list of the squares of the first n + 1 Fibonacci numbers:
```Scheme
(define (list-fib-squares n)
  (accumulate
   cons
   nil
   (map square (map fib (enumerate-interval 0 n)))))

(list-fib-squares 10)
>>> (0 1 1 4 9 25 64 169 441 1156 3025)
```
Sequences, implemented here as lists, serve as a conventional interface that permits us to combine processing modules. Additionally, when we uniformly represent structures as sequences, we have localized the data-structure dependencies in our programs to a small number of sequence operations. By changing these, we can experiment with alternative representations of sequences, while leaving the overall design of our programs intact.

## Example: A Picture Language(Incomplete)
# Symbolic Data
All the compound data objects we have used so far were constructed ultimately from numbers. In this section we extend the representational capability of our language by introducing the ability to work with arbitrary symbols as data.
## Quotation
If we can form compound data using symbols, we can have lists such as
```Scheme
(a b c d)
((Norah 12) (Molly 9) (Anna 7) (Lauren 6) (Charlotte 4))
```
In order to manipulate symbols we need a new element in our language: the ability to _quote_ a data object. Suppose we want to construct the list (a b). We can't accomplish this with (list a b), because this expression constructs a list of the values of a and b rather than the symbols themselves.  

To identify lists and symbols that are to be treated as data objects rather than as expressions to be evaluated, we place a quotation mark (traditionally, the single quote symbol ') only at the beginning of the object to be quoted. 
```Scheme
(define a 1)
(define b 2)

(list a b)
>>> (1 2)

(list 'a 'b)
>>> (a b)

(list 'a b)
>>> (a 2)
```
Quotation also allows us to type in compound objects, using the conventional printed prepresentation for lists:
```Scheme
(car '(a b c))
>>> a

(cdr '(a b c))
>>> (b c)
```
In keeping with this, we can obtain the empty list by evaluating '(), and thus dispense with the variable nil.  

One additional primitive used in manipulating symbols is _eq?_, which takes two symbols as arguments and tests whether they are the same. We can consider two symbols to be "the same" if they consist of the same characters in the same order. Using eq?, we can implement a useful procedure called memq. This takes two arguments, a symbol and a list. If the symbol is not contained in the list (i.e., is not eq? to any item in the list), then memq returns false. Otherwise, it returns the sublist of the list beginning with the first occurrence of the symbol:
```Scheme
(define (memq item x)
  (cond ((null? x) false)
        ((eq? item (car x)) x)
        (else (memq item (cdr x)))))
        
(memq 'apple '(x (apple sauce) y apple pear))
>>> (apple pear)
```
```Scheme
(eq? 2 2)
>>> #t

(define a 2)
(define b 2)
(eq? a b)
>>> #t

(define a '2)
(define b 2)
(eq? a b)
>>> #t

(define a '2)
(define b '2)
(eq? a b)
>>> #t

(define a '(2 3))
(define b '(2 3))
(eq? a b)
>>> #f

(define x '())
(define y '())
(eq? x y)
>>> #t

(define x 'x)
(define y x)
(eq? x y)
>>> #t

(eq? (+ 2 100) (+ 2 100))
>>> #t

(eq? (expt 2 100) (expt 2 100))
>>> #f
```
In practice, programmers use _equal?_ to compare lists that contain numbers as well as symbols. Numbers are not considered to be symbols. The question of whether two numberically equal numbers (as tested by =) are also eq? is highly implementation dependent. A better definition of equal? (such as the one that comes as a primitive in Scheme) would also stipulate that if a and b are both numbers, then a and b are equal? if they are numerically equal.
```Scheme
(equal? (+ 2 100) (+ 2 100))
>>> #t

(equal? (expt 2 100) (expt 2 100))
>>> #f
```
## Example: Symbolic Differentiation(Incomplete)
## Example: Representing Sets(Incomplete)
## Example: Huffman Encoding Trees(Incomplete)
# Multiple Representation for Abstract Data
We have introduced data abstraction, a methodology for structuring systems in such a way that much of a program can be specified independent of the choices involved in implementing the data objects that the program manipulates. In this section, we will learn how to cope with data that may be represented in different ways by different parts of a program. This requires constructing _generic procedures_ -procedures that can operate on data that may be represented in more than one way. Our main technique for building generic procedures will be to work in terms of data objects that have type _tags_, that is, data objects that include explicit information about how they are to be processed. We will also discuss _data-directed_ programming, a powerful and convenient implementation strategy for additively assembling systems with generic operations.  

We begin with the simple complex-number example. We will see how type tags and data-directed style enable us to design separate rectangular and polar representations for complex numbers while maintaining the notion of an abstract "complex-number" data object. We will accomplish this by defining arithmetic procedures for complex numbers (add-complex, sub-complex, mul-complex, and div-complex) in terms of generic selectors that access parts of a complex number independent of how the number is represented.  




## Representation for Complex Numbers
We begin by discussing two plausible representations for complex numbers as ordered pairs: rectangular form (real part and imaginary part) and polar form (magnitude and angle). Tagged data section will show how both representations can be made to coexist in a single system through the use of type tags and generic operations.  

Assume that the operations on complex numbers are implemented in terms of four selectors: _real-part_, _imag-part_, _magnitude_ and _angle_. Also assume that we have two procedures for constructing complex numbers: _make-from-real-imag_ returns a complex number with specified real and imaginary parts, and _make-from-mag-ang_ returns a complex number with specified magnitude and angle. These procedures have the property that, for any complex number z, both
```Scheme
(make-from-real-imag (real-part z) (imag-part z))
```
and
```Scheme
(make-from-mag-ang (magnitude z) (angle z))
```
produce complex numbers that are equal to z.  

We can add and subtract complex numbers in terms of real and imaginary parts while multiplying and dividing complex numbers in terms of magnitudes and angles:
```Scheme
(define (add-complex z1 z2)
  (make-from-real-imag (+ (real-part z1) (real-part z2))
                       (+ (imag-part z1) (imag-part z2))))

(define (sub-complex z1 z2)
  (make-from-real-imag (- (real-part z1) (real-part z2))
                       (- (imag-part z1) (imag-part z2))))

(define (mul-complex z1 z2)
  (make-from-mag-ang (* (magnitude z1) (magnitude z2))
                     (+ (angle z1) (angle z2))))

(define (div-complex z1 z2)
  (make-from-mag-ang (/ (magnitude z1) (magnitude z2))
                     (- (angle z1) (angle z2))))
```
To complete the complex-number package, we must choose a representation and we must implement the constructors and selectors in terms of primitive numbers and primitive list structure. There are two obvious ways to do this: We can represent a complex number in "rectangular form" as a pair (real part, imaginary part) or in "polar form" as a pair (magnitude, angle).
```Scheme
Ben's reporesentation
Rectangular Form

(define (real-part z) (car z))
(define (imag-part z) (cdr z))
(define (magnitude z)
  (sqrt (+ (square (real-part z))
           (square (imag-part z)))))
(define (angle z)
  (atan (imag-part z) (real-part z)))
(define (make-from-real-imag x y) (cons x y))
(define (make-from-mag-ang r a)
  (cons (* r (cos a)) (* r (sin a))))
```

```Scheme
Alyssa's representation
Polar Form

(define (real-part z) (* (magnitude z) (cos (angle z))))
(define (imag-part z) (* (magnitude z) (sin (angle z))))
(define (magnitude z) (car z))
(define (angle z) (cdr z))
(define (make-from-real-imag x y)
  (cons (sqrt (+ (square x) (square y)))
        (atan y x)))
(define (make-from-mag-ang r a) (cons r a))
```
The discipline of data abstraction ensures that the same implementation of add-complex, sub-complex, mul-complex, and div-complex will work with either representation.

## Tagged Data 
If we desire, we can maintain the ambiguity of representation even after we have designed the selectors and constructors, and elect to use both Ben's representation and Alyssa's representation. If both representations are included in a single system, however, we will need some way to distinguish data in polar form from data in rectangular form. Otherwise, if we were asked, for instance, to find the magnitude of the pair (3, 4), we wouldn't know whether to answer 5 (interpreting the number in rectangular form) or 3 (interpreting the number in polar form). A straightforward way to accomplish this distinction is to include a type tag -the symbol rectangular or polar- as part of each complex number. Then when we need to manipulate a complex number we can use the tag to decide which selector to apply.  

In order to manipulate tagged data, we will assume that we have procedures type-tag and contents that extract from a data object the tag and the actual contents (the polar or rectangular coordinates, in the case of a complex number). We will also postulate a procedure attach-tag that takes a tag and contents and produces a tagged data object. A straightforward way to implement this is to use ordinary list structure:
```Scheme
(define (attach-tag type-tag contents)
  (cons type-tag contents))

(define (type-tag datum)
  (if (pair? datum)
      (car datum)
      (error "Bad tagged datum: TYPE-TAG" datum)))

(define (contents datum)
  (if (pair? datum)
      (cdr datum)
      (error "Bad tagged datum: CONTENTS" datum)))
```
Using these procedures, we can define predicates rectangular? and polar?, which recognize rectangular and polar numbers, respectively:
```Scheme
(define (rectangular? z) (eq? (type-tag z) 'rectangular))
(define (polar? z) (eq? (type-tag z) 'polar))
```
With type tags, Ben and Alyssa can now modify their code so that their two different representations can coexist in the same system. Whenever Ben constructs a complex number, he tags it as rectangular. Whenever Alyssa constructs a complex number, she tags it as polar. In addition, Ben and Alyssa must make sure that the names of their procedures do not conflict. One way to do this is for Ben to append the suffix rectangular to the name of each of his representation procedures and for Alyssa to append polar to the names of hers. Here is Ben's revised rectangular representation:
```Scheme
(define (real-part-rectangular z) (car z))
(define (imag-part-rectangular z) (cdr z))
(define (magnitude-rectangular z)
  (sqrt (+ (square (real-part-rectangular z))
           (square (imag-part-rectangular z)))))
(define (angle-rectangular z)
  (atan (imag-part-rectangular z)
        (real-part-rectangular z)))
(define (make-from-real-imag-rectangular x y)
  (attach-tag 'rectangular (cons x y)))
(define (make-from-mag-ang-rectangular r a)
  (attach-tag 'rectangular
              (cons (* r (cos a)) (* r (sin a)))))
```
and here is Alyssa's revised polar representation: 
```Scheme
(define (real-part-polar z)
  (* (magnitude-polar z) (cos (angle-polar z))))
(define (imag-part-polar z)
  (* (magnitude-polar z) (sin (angle-polar z))))
(define (magnitude-polar z) (car z))
(define (angle-polar z) (cdr z))
(define (make-from-real-imag-polar x y)
  (attach-tag 'polar
              (cons (sqrt (+ (square x) (square y)))
                    (atan y x))))
(define (make-from-mag-ang-polar r a)
  (attach-tag 'polar (cons r a)))
```
Each generic selector is implemented as a procedure that checks the tag of its argument and calls the appropriate procedure for handling data of that type.
```Scheme
(define (real-part z)
  (cond ((rectangular? z)
         (real-part-rectangular (contents z)))
        ((polar? z)
         (real-part-polar (contents z)))
        (else (error "Unknown type: REAL-PART" z))))

(define (imag-part z)
  (cond ((rectangular? z)
         (imag-part-rectangular (contents z)))
        ((polar? z)
         (imag-part-polar (contents z)))
        (else (error "Unknown type: IMAG-PART" z))))

(define (magnitude z)
  (cond ((rectangular? z)
         (magnitude-rectangular (contents z)))
        ((polar? z)
         (magnitude-polar (contents z)))
        (else (error "Unknown type: MAGNITUDE" z))))

(define (angle z)
  (cond ((rectangular? z)
         (angle-rectangular (contents z)))
        ((polar? z)
         (angle-polar (contents z)))
        (else (error "Unknown type: ANGLE" z))))
```
To implement the complex-number arithmetic operations, we can use the same procedures add-complex, sub-complex, mul-complex, and div-complex from previous section, because the selectors they call are generic, and so will work with either representation.  

Finally, we must choose whether to construct complex numbers using Ben's representation or Alyssa's representation. One reasonable choice is to construct rectangular numbers whenever we have real and imaginary parts and to construct polar numbers whenever we have magnitudes and angles:
```Scheme
(define (make-from-real-imag x y)
  (make-from-real-imag-rectangular x y))
(define (make-from-mag-ang r a)
  (make-from-mag-ang-polar r a))
```
Notice the general mechanism for interfacing the separate representations: Within a given representation implementation (say, Alyssa's polar package) a complex number is an untyped pair (magnitude, angle). When a generic selector operates on a number of polar type, it strips off the tag and passes the contents on to Alyssa's code. Conversely, when Alyssa constructs a number for general use, she tags it with a type so that it can be appropriately recognized by the higher-level procedures. This discipline of stripping off and attaching tags as data objects are passed from level to level can be an important organizational strategy,
as we shall see in Systems and Generic Operations section.
## Data-Directed Programming and Additivity 
The general strategy of checking the type of a datum and calling an appropriate procedure is called _dispatching on type_. This is a powerful strategy for obtaining modularity in system design. On the other hand, implementing the dispatch as in the previous section has two significant weaknesses. One weakness is that the generic interface procedures (real-part, imag-part, magnitude, and angle) must know about all the different representations. For instance, suppose we wanted to incorporate a new representation for complex numbers into our complex-number system. We would need to identify this new representation with a type, and then add a clause to each of the generic interface procedures to check for the new type and apply the appropriate selector for that representation.  
Another weakness of the technique is that even though the individual representations can be designed separately, we must guarantee that no two procedures in the entire system have the same name.  

The issue underlying both of these weaknesses is that the technique for implementing generic interfaces is not _additive_. What we need is a means for modularizing the system design even further. This is provided by the programming technique known as _data-directed programming_. To understand how data-directed programming works, begin with the observation that whenever we deal with a set of generic operations that are common to a set of different types we are, in effect, dealing with a two-dimensional table that contains the possible operations on one axis and the possible types on the other axis. The entries in the table are the procedures that implement each operation for each type of argument presented.  
![operations](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/operations.png)  

Data-directed programming is the technique of designing programs to work with such a table directly. Previously, we implemented the mechanism that interfaces the complex-arithmetic code with the two representation packages as a set of procedures that each perform an explicit dispatch on type. Here we will implement the interface as a single procedure that looks up the combination of the operation name and argument type in the table to find the correct procedure to apply, and then applies it to the contents of the argument. If we do this, then to add a new representation package to the system we need not change any existing procedures; we need only add new entries to the table.  
To implement this plan, assume that we have two procedures, put and get, for manipulating the operation-and-type table:
```
(put <op> <type> <item>) installs the <item> in the table, indexed by the <op> and the <type>
```
```
(get <op> <type>) looks up the <op>, <type> enrty in the table and returns the item found there. 
If no item is found, get returns false.

```
For now, we can assume that put and get are included in our language. Here is how data-directed programming can be used in the complex-number system. Ben, who developed the rectangular representation, implements his code just as he did originally. He defines a collection of procedures, or a package, and interfaces these to the rest of the system by adding entries to the table that tell the system how to operate on rectangular numbers. This is accomplished by calling the following procedure:
```Scheme
(define (install-rectangular-package)
;; internal procedures
  (define (real-part z) (car z))
  (define (imag-part z) (cdr z))
  (define (make-from-real-imag x y) (cons x y))
  (define (magnitude z)
    (sqrt (+ (square (real-part z))
             (square (imag-part z)))))
  (define (angle z)
    (atan (imag-part z) (real-part z)))
  (define (make-from-mag-ang r a)
    (cons (* r (cos a)) (* r (sin a))))
;; interface to the rest of the system
  (define (tag x) (attach-tag 'rectangular x))
  (put 'real-part '(rectangular) real-part)
  (put 'imag-part '(rectangular) imag-part)
  (put 'magnitude '(rectangular) magnitude)
  (put 'angle '(rectangular) angle)
  (put 'make-from-real-imag 'rectangular
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'rectangular
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
```
Since these procedure definitions are internal to the installation procedure, Ben needn't worry about name conflicts with other procedures outside the rectangular package. To interface these to the rest of the system, Ben installs his real-part procedure under the operation name real-part and the type (rectangular), and similarly for the other selectors. The interface also defines the constructors to be used by the external system. These are identical to Ben's internally defined constructors, except that they attach the tag.  

Alyssa's polar package is analogous:
```Scheme
(define (install-polar-package)
;; internal procedures
  (define (magnitude z) (car z))
  (define (angle z) (cdr z))
  (define (make-from-mag-ang r a) (cons r a))
  (define (real-part z) (* (magnitude z) (cos (angle z))))
  (define (imag-part z) (* (magnitude z) (sin (angle z))))
  (define (make-from-real-imag x y)
    (cons (sqrt (+ (square x) (square y)))
          (atan y x)))
;; interface to the rest of the system
  (define (tag x) (attach-tag 'polar x))
  (put 'real-part '(polar) real-part)
  (put 'imag-part '(polar) imag-part)
  (put 'magnitude '(polar) magnitude)
  (put 'angle '(polar) angle)
  (put 'make-from-real-imag 'polar
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'polar
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
```
Even though Ben and Alyssa both still use their original procedures defined with the same names as each other's (e.g., real-part), these definitions are now internal to different procedures, so there is no name conflict.  

The complex-arithmetic selectors access the table by means of a general "operation" procedure called apply-generic, which applies a generic operation to some arguments. Apply-generic looks in the table under the name of the operation and the types of the arguments and applies the resulting procedure if one is present:

```Scheme
(define (apply-generic op . args)
  (let ((type-tags (map type-tag args)))
    (let ((proc (get op type-tags)))
      (if proc
          (apply proc (map contents args))
          (error
           "No method for these types: APPLY-GENERIC"
           (list op type-tags))))))
```
Using apply-generic, we can define our generic selectors as follows:
```Scheme
(define (real-part z) (apply-generic 'real-part z))
(define (imag-part z) (apply-generic 'imag-part z))
(define (magnitude z) (apply-generic 'magnitude z))
(define (angle z) (apply-generic 'angle z))
```
Observe that these do not change at all if a new representation is added to the system.  

We can also extract from the table the constructors to be used by the programs external to the packages in making complex numbers from real and imaginary parts and from magnitudes and angles. We construct rectangular numbers whenever we have real and imaginary parts, and polar numbers whenever we have magnitudes and angles:
```Scheme
(define (make-from-real-imag x y)
  ((get 'make-from-real-imag 'rectangular) x y))
(define (make-from-mag-ang r a)
  ((get 'make-from-mag-ang 'polar) r a))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Message passing</h3>
An alternative implementation strategy is to decompose the table into columns and, instead of using "intelligent operations" that dispatch on data types, to work with "intelligent data objects" that dispatch on operation names. We can do this by arranging things so that a data object, such as a rectangular number, is represented as a procedure that takes as input the required operation name and performs the operation indicated. In such a discipline, make-from-real-imag could be written as
```Scheme
(define (make-from-real-imag x y)
  (define (dispatch op)
    (cond ((eq? op 'real-part) x)
          ((eq? op 'imag-part) y)
          ((eq? op 'magnitude) (sqrt (+ (square x) (square y))))
          ((eq? op 'angle) (atan y x))
          (else (error "Unknown op: MAKE-FROM-REAL-IMAG" op))))
  dispatch)
```
The corresponding apply-generic procedure, which applies a generic operation to an argument, now simply feeds the operation's name to the data object and lets the object do the work:
```Scheme
(define (apply-generic op arg) (arg op))
```
Note that the value returned by make-from-real-imag is a procedure -the internal dispatch procedure. This is the procedure that is invoked when apply-generic requests an operation to be performed.  

This style of programming is called message passing. The name comes from the image that a data object is an entity that receives the requested operation name as a "message."



# Systems and Generic Operations
In the previous section, we saw how to design systems in which data objects can be represented in more than one way. The key idea is to link the code that specifies the data operations to the several representations by means of generic interface procedures. Now we will see how to use this same idea not only to define operations that are generic over different representations but also to define operations that are generic over different kinds of arguments.

## Generic Arithmetic Operations
We would like, for instance, to have a generic addition procedure add that acts like ordinary primitive addition + on ordinary numbers, like add-rat on rational numbers, and like add-complex on complex numbers. We will attach a type tag to each kind of number and cause the generic procedure to dispatch to an appropriate package according to the data type of its arguments.  

The generic arithmetic procedures are defined as follows:
```Scheme
(define (add x y) (apply-generic 'add x y))
(define (sub x y) (apply-generic 'sub x y))
(define (mul x y) (apply-generic 'mul x y))
(define (div x y) (apply-generic 'div x y))
```
We begin by installing a package for handling ordinary numbers, that is, the primitive numbers of our language. We will tag these with the symbol scheme-number. The arithmetic operations in this package are the primitive arithmetic procedures (so there is no need to define extra procedures to handle the untagged numbers). Since these operations each take two arguments, they are installed in the table keyed by the list (scheme-number scheme-number):
```Scheme
(define (install-scheme-number-package)
  (define (tag x) (attach-tag 'scheme-number x))
  (put 'add '(scheme-number scheme-number)
       (lambda (x y) (tag (+ x y))))
  (put 'sub '(scheme-number scheme-number)
       (lambda (x y) (tag (- x y))))
  (put 'mul '(scheme-number scheme-number)
       (lambda (x y) (tag (* x y))))
  (put 'div '(scheme-number scheme-number)
       (lambda (x y) (tag (/ x y))))
  (put 'make 'scheme-number (lambda (x) (tag x)))
  'done)
```
Users of the Scheme-number package will create (tagged) ordinary numbers by means of the procedure:
```Scheme
(define (make-scheme-number n)
  ((get 'make 'scheme-number) n))
```
Now that the framework of the generic arithmetic system is in place, we can readily include new kinds of numbers. Here is a package that performs rational arithmetic.
```Scheme
(define (install-rational-package)
;; internal procedures
  (define (numer x) (car x))
  (define (denom x) (cdr x))
  (define (make-rat n d)
    (let ((g (gcd n d)))
      (cons (/ n g) (/ d g))))
  (define (add-rat x y)
    (make-rat (+ (* (numer x) (denom y))
                 (* (numer y) (denom x)))
              (* (denom x) (denom y))))
  (define (sub-rat x y)
    (make-rat (- (* (numer x) (denom y))
                 (* (numer y) (denom x)))
              (* (denom x) (denom y))))
  (define (mul-rat x y)
    (make-rat (* (numer x) (numer y))
              (* (denom x) (denom y))))
  (define (div-rat x y)
    (make-rat (* (numer x) (denom y))
              (* (denom x) (numer y))))
;; interface to rest of the system
  (define (tag x) (attach-tag 'rational x))
  (put 'add '(rational rational)
       (lambda (x y) (tag (add-rat x y))))
  (put 'sub '(rational rational)
       (lambda (x y) (tag (sub-rat x y))))
  (put 'mul '(rational rational)
       (lambda (x y) (tag (mul-rat x y))))
  (put 'div '(rational rational)
       (lambda (x y) (tag (div-rat x y))))
  (put 'make 'rational
       (lambda (n d) (tag (make-rat n d))))
  'done)
(define (make-rational n d)
  ((get 'make 'rational) n d))
```
We can install a similar package to handle complex numbers, using the tag complex . In creating the package, we extract from the table the operations make-from-real-imag and make-from-mag-ang that were defined by the rectangular and polar packages.
```Scheme
(define (install-complex-package)
;; imported procedures from rectangular and polar packages
  (define (make-from-real-imag x y)
    ((get 'make-from-real-imag 'rectangular) x y))
  (define (make-from-mag-ang r a)
    ((get 'make-from-mag-ang 'polar) r a))
;; internal procedures
  (define (add-complex z1 z2)
    (make-from-real-imag (+ (real-part z1) (real-part z2))
                         (+ (imag-part z1) (imag-part z2))))
  (define (sub-complex z1 z2)
    (make-from-real-imag (- (real-part z1) (real-part z2))
                         (- (imag-part z1) (imag-part z2))))
  (define (mul-complex z1 z2)
    (make-from-mag-ang (* (magnitude z1) (magnitude z2))
                       (+ (angle z1) (angle z2))))
  (define (div-complex z1 z2)
    (make-from-mag-ang (/ (magnitude z1) (magnitude z2))
                       (- (angle z1) (angle z2))))
;; interface to rest of the system
  (define (tag z) (attach-tag 'complex z))
  (put 'add '(complex complex)
       (lambda (z1 z2) (tag (add-complex z1 z2))))
  (put 'sub '(complex complex)
       (lambda (z1 z2) (tag (sub-complex z1 z2))))
  (put 'mul '(complex complex)
       (lambda (z1 z2) (tag (mul-complex z1 z2))))
  (put 'div '(complex complex)
       (lambda (z1 z2) (tag (div-complex z1 z2))))
  (put 'make-from-real-imag 'complex
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'complex
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
```
Programs outside the complex-number package can construct complex numbers either from real and imaginary parts or from magnitudes and angles. Notice how the underlying procedures, originally defined in the rectangular and polar packages, are exported to the complex package, and exported from there to the outside world.
```Scheme
(define (make-complex-from-real-imag x y)
  ((get 'make-from-real-imag 'complex) x y))
(define (make-complex-from-mag-ang r a)
  ((get 'make-from-mag-ang 'complex) r a))
```
In the above packages, we used add-rat, add-complex, and the otherarithmetic procedures exactly as originally written. Once these definitions are internal to different installation procedures, however, they no longer need names that are distinct from each other: we could simply name them add, sub, mul, and div in both packages.

## Combining Data of Different Types(Incomplete)
## Example: Symbolic Algebra(Incomplete)
