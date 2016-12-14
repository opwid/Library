# Recursively Specified Data
## Inductive Specification
Inductive specification is a powerful method of specifying a set of values. To illustrate this method, we use it to describe a certain subset S of the natural numbers N = {0,1,2,...}.
```
Definition 1.1.1 
A natural number n is in S if and only if
1. n = 0, or
2. n - 3 ∈ S
```
From this definition, we conclude that S is the set of natural numers that are multiples of 3. We can use this definition to write a procedure to decide whether a natural number n is in S.
```Scheme
in-S? : N -> Bool
(define in-S?
  (lambda (n)
    (if (zero? n) #t
        (if (>= (- n 3) 0)
            (in-S? (- n 3))
            #f))))
```
We have written a recursive procedure in Scheme that follows the definition. The notation in-S?: N-> Bool is a comment, called the contract for this procedure. It means that in-S? is intended to be a procedure that takes a natural number and produces a boolean.
```
Definition 1.1.3 (list of integers, top-down)
A Scheme list is a list of integers if and only if either
1. it is the empty list, or
2. it is a pair whose car is an integer and whose cdr is a list of integers
```
We use Int to denote the set of all integers, and List-of-Int to denote the set of lists of integers.
```
Definition 1.1.4 (list of integers, bottom-up)
The set List-of-Int is the smallest set of Scheme lists satisfying the following two properties
1. () ∈ List-of-Int, and
2. if n ∈ Int and l ∈ List-of-Int, then (n.l) ∈ List-of-Int
```
Here we use the infix "." to denote the result of the `cons` operation in Scheme. The phrase (n.l) denotes a Scheme pair whose car is n and whose cdr is l.

```
Definition 1.1.5 (list of integers, rules of inference)
() ∈ List-of-Int

n ∈ Int l ∈ List-of-Int
------------------------
(n.l) ∈ List-of-Int
```
These three definitions are equivalent. We can show how to use them to generate some elements of List-of-Int.  

1. () is a list of integers, because of property 1 of definition 1.1.4 or the first rule of definition 1.1.5  
2. (14 . ()) is a list of integers, because of property 2 of definition 1.1.4, since 14 is an integer and () is a list of integers  
3. (3 . (14 . ())) is a list of integers, because of property 2, since 3 is an integer and (14 . ()) is a list of integers  
4. (-7 . (3 . (14 . ()))) is a list of integers, because of property 2, since -7 is a integer and (3 . (14 . ())) is a list of integers  
5. Nothing is a list of integers unless it is built in this fashion  

## Defining Sets Using Grammars
Grammars are typically used to specify set of strings, but we can use them to define sets of values as well. For example, we can define the set List-of-Int by grammar
```
List-of-Int ::= ()
List-of-Int ::= (Int . List-of-Int)
```
Here we have two rules corresponding to the two properties in definition 1.1.4 above. The first rule says that the empty list is in List-of-Int, and the second says that if n is in Int and l is in List-of-Int, then (n . l) is in List-of-Int. This set of rules is called a grammar.   
Let us look at the pieces of this definition. In this definition we have  

* __Nonterminal Symbols__: There are the names of the sets being defined. In this case there is only one such set, but in general, there might be several sets being defined. We will use the convention that nonterminals and sets have names that are capitalized, but we will use lower-case names when referring to their elements in prose. For example, Expression is a nonterminal, but we will write e ∈ Expression or "e is an expression". Another common convention, called Backus-Naur Form or BNF, is to surround the word with angle brackets, e.g. \<expression\>. List-of-Int and Int are nonterminal symbols.  
* __Terminal Symbols__: These are the characters in the external representation. ., (, and ) are terminal symbols.
* __Productions__: The rules are called productions. Each production has a left-hand side, which is a nonterminal symbol, and a right-hand side, which consists of terminal and nonterminal symbols. The left- and right- hand sides are usually seperated by the symbol ::=, read is or can be. The right-hand side specifies a method for constructing members of the syntactic category in terms of other syntactic categories and terminal symbols, such as the left parenthesis, right parenthesis and the period. 

It is common to omit the left-hand side of a production when it is same as the left-hand side of the preceding production.
```
List-of-Int ::= ()
            ::= (Int . List-of-Int)
```
It is also common to write a set of rules for a single syntactic category by writing the left-hand side and ::= just once, followed by all the right-hand sides seperated by the special symbol "|". 
```
List-of-Int ::= () | (Int . List-of-Int)
```
Another shortcut is the Kleene star, expressed by the notation {...}*. When this appears in a right-hand side, it indicates a sequence of any number of instances of whatever appears between braces. This includes the possibility of no instances at all. If there are zero instances, we get the empty string.
```
List-of-Int ::= ({Int}*)
```
A variant of the star notation is Kleene plus {...}+, which indicates a sequence of one or more instances. Substituting + for * in the example above would define the syntatic category of non-empty lists of integers.   
Still another variant of the star notation is the seperated list notation. For example, we write {Int}*(c) to denote a sequence of any number of instances of the nonterminal Int, seperated by the non-empty character sequence c.
```
{Int}*(,) 
8  
14, 12  
7, 3, 14, 16
```
The previous demonstration that (14 . ()) is a list of integers may be formalized with the syntactic derivation
```
List-of-Int
=> (Int . List-of-Int)
=> (14 . List-of-Int)
=> (14 . ())
```
Let us consider the definitions of some other useful sets.  

1. Many symbol manipulation procedures are designed to operate on lists that contain only symbols and other similarly restricted lists. We can these lists s-lists, defined as follows:
  ```
  Definition 1.1.6
  (s-list, s-exp)
  S-list ::= ({S-exp}*)
  S-exp ::= Symbol | S-list
  ```   
  ```
  (a b c)
  (an (((s-list)) (with () lots) ((of) nesting)))
  ```  

2. A binary tree with numeric leaves and interior nodes labeled with symbols may be represented using three-element lists for the interior nodes by the grammar:
  ```
  Definition 1.1.7
  (binary tree)
  Bintree ::= Int | (Symbol Bintree Bintree)
  ```
  ```
  1
  2
  (foo 1 2)
  (bar 1 (foo 1 2))
  (baz
    (bar 1 (foo 1 2))
    (biz 4 5))
  ```
3. The lambda calculus is a simple language that is often used to study the theory of programming languages. This language consists only of variable references, procedures that take a single argument, and procedure calls. We can define it with the grammar:
  ```
  Definition 1.1.8
  (lambda expression)
  LcExp ::= Identifier
        ::= (lamda (Identifier) LcExp)
        ::= (LcExp LcExp)
        
  where an identifier is any symbol other than lambda.
  ```
  The identifier in the second production is the name of a variable in the body of the lambda expression. This variable is called the bound variable of the expression, because it binds or captures any occurrences of the variable in the body. Any occurrence of that variable in the body refers to this one.  
  To see how this works, consider lambda calculus extended with arithmetic operators. In that language,
  ```Scheme
  (lambda (x) (+ x 5))
  ```
  is an expression in which x is the bound variable. Therefore, in
  ```Scheme
  ((lambda (x) (+ x 5)) (- x 7))
  ```
  the last occurrence of x does not refer to the x that is bound in the lambda expression.
  This grammar defines the elements of LcExp as Scheme values, so it becomes easy to write programs that manipulate them.
  

## Induction
We can use inductive definitions in two ways: to prove theorems about members of the set and to write programs that manipulate them. 
>Theorem 1.1.1 Let t be a binary tree, as defined in definition 1.1.7. Then t contains an odd number of nodes.   

_Proof_: The proof is by induction on the size of t, where we take the size of t to be the number of nodes in t. The induction hypothesis, IH(k), is that any tree of size <= k has an odd number of nodes. We follow the usual prescription for an inductive proof: we first proof that IH(0) is true, and we then prove that whenever k is an integer such that IH is true for k, then IH is true for k+1 also.

1. There are no trees with 0 nodes, so IH(0) holds trivially
2. Let k be an integer such that IH(k) holds, that is, any tree with <= k nodes actually has an odd number of nodes. We need to show that IH(k+1) holds as well: that any tree with <= k+1 nodes has an odd number of nodes. If t has <= k+1 nodes, there are exactly two possibilities according to the definition of a binary tree:
  1. t could be of the form n, where n is an integer. In this case, t has exactly one node, and one is odd
  2. t could be of the form (sym t<sub>1</sub> t<sub>2</sub>), where sym is a symbold and t<sub>1</sub> and t<sub>2</sub> are trees. Now t<sub>1</sub> and t<sub>2</sub> must have fewer nodes than t. Since t has <= k+1 nodes, t<sub>1</sub> and t<sub>2</sub> must have <= k nodes. Therefore they are covered by IH(k), and they must each have an odd number of nodes, say 2n<sub>1</sub>+1 and 2n<sub>2</sub>+1, respectively. Hence the total number of nodes in the tree, counting the two subtrees and the root, is (2n<sub>1</sub> + 1) + (2n<sub>2</sub> + 1) + 1 = 2(n<sub>1</sub> + n<sub>2</sub> + 1) + 1 which is once again odd.
  
  
This completes the proof of the claim that IH(k+1) holds and therefore completes the induction.
# Deriving Recursive Programs
We have used the method of inductive definition to characterize complicated sets. We have seen that we can analyze an element of an inductively defined set to see how it is built from smaller elements of the set. We now use the same idea to define more general procedures that compute on inductively defined sets.  
Recursive procedures rely on an important principle:
>If we can reduce a problem to a smaller subproblem, we can call the procedure that solves the problem to solve the subproblem.  


The solution returned for the subproblem may then be used to solve the original problem.   
We illustrate this idea with a sequence of examples.
## list-length
The standard Scheme procedure `length` determines the number of elements in a list.
```Scheme
(length '(a b c))
>>> 3
(length '((x) ()))
>>> 2
```
Let us write our own procedure, called `list-length`, that does the same thing.
```
list-length : List -> Int
usage : (list-length l) = the length of l
```
We define the set of lists by
```
List ::= () | (Scheme value . List)
```
If the list is empty, then its length is 0. If a list is non-empty, then its length is one more than the length of its cdr.
```Scheme
(define list-length
  (lambda (lst)
    (if (null? lst)
      0
      (+ 1 (list-length (cdr lst))))))
```
## nth-element
Let us write a procedure, called nth-element, that takes a list lst and a zero-based index n and returns element number n of lst.
```
nth-element : List X Int -> SchemeVal
usage : (nth-element lst n) = the n-th element of lst
```
```Scheme
(define nth-element
  (lambda (lst n)
    (if (zero? n)
      (car lst)
      (nth-element (cdr lst) (- n 1)))))
```
```Scheme
(nth-element '(a b c d e) 3)
(nth-element '(b c d e) 2)
(nth-element '(c d e) 1)
(nth-element '(d e) 0)
>>> d
```
## occurs-free?
The procedure occurs-free? should take a variable var, represented as a Scheme symbol, and a lambda-calculus expression exp as defined in definition 1.1.8, and determine whether or not var occurs free in exp. We say that a variable occurs free in an expression exp if it has some occurence in exp that is not inside some lambda binding of the same variable. For example,
```Scheme
(occurs-free? 'x 'x)
>>> #t
(occurs-free? 'x '(lambda (x) (x y)))
>>> #f
(occurs-free? 'x '(lambda (y) (x y)))
>>> #t
(occurs-free? 'x '((lambda (x) x) (x y)))
>>> #t
(occurs-free? 'x '(lambda (y) (lambda (z) (x (y z)))))
>>> #t
```
We can solve this problem by following the grammar for lambda-calculus expressions
```
LcExp ::= Identifier
      ::= (lamda (Identifier) LcExp)
      ::= (LcExp LcExp)
```
We can summarize these cases in the rules:

* If the expression e is a variable, then the variable x occurs free in e if and only if x is the same as e. 
* If the expression e is of the form (lambda (y) e'), then the variable x occurs free in e if and only if y is different from x and x occurs free in e'.
* If the expression e is of the form (e<sub>1</sub> e<sub>2</sub>), then x occurs free in e if and only if it occurs free in e<sub>1</sub> or e<sub>2</sub>. Here, we use "or" to mean inclusive or, meaning that this includes the possibility that x occurs free in both e<sub>1</sub> and e<sub>2</sub>. 

Then it is easy to define occurs-free?. Since there are three alternatives to be checked, we use a Scheme `cond` rather than an `if`. 
```
occurs-free? : Sym X LcExp -> Bool
usage : returns #t if the symbol var occurs free in exp, otherwise return #f
```
```Scheme
(define occurs-free?
  (lambda (var exp)
    (cond
      ((symbol? exp) (eqv? var exp))
      ((eqv? (car exp) 'lambda)
       (and
        (not (eqv? var (car (cadr exp))))
        (occurs-free? var (caddr exp))))
      (else
       (or
        (occurs-free? var (car exp))
        (occurs-free? var (cadr exp)))))))
```
>Follow the Grammar!

>When defining a procedure that operates on inductively defined data, the structure of the program should be patterned after the structure of the data.
                                                                                                                       
# Auxiliary Procedures and Context Arguments
The _Follow-the-Grammar_ recipe is powerful, but sometimes it is not sufficient. Consider the procedure `number-elements` which takes any list (v<sub>0</sub> v<sub>1</sub> v<sub>2</sub> ...) and returns the list ((0 v<sub>0</sub>) (1 v<sub>1</sub>) (2 v<sub>2</sub>) ...). A straightforward decomposition of the kind we've used so far does not solve this problem, because there is no obvious way to build the value of (number-elements lst) from the value of (number-elements (cdr lst)).  

To solve this problem, we need to generalize the problem.
```
number-elements-from : Listof(SchemeVal) X Int -> Listof(List(Int,SchemeVal))
usage : (number-elements-from '(v0 v1 v2 ...) n) = ((n v0) (n+1 v1) (n+2 v2) ...)
```
```Scheme
(define number-elements-from
  (lambda (lst n)
    (if (null? lst) '()
        (cons
         (list n (car lst))
         (number-elements-from (cdr lst) (+ n 1))))))
    
    
    
(number-elements-from '(1 2 3 4) 9)
>>> '((9 1) (10 2) (11 3) (12 4))
```
Once we have defined `number-elements-from`, it is easy to write the desired procedure.
```
number-elements : List -> Listof(List(Int,SchemeVal))
```
```Scheme
(define number-elements
  (lambda (lst)
    (number-elements-from lst 0)))
```
It is very common for a programmer to write a procedure that simply calls some auxiliary procedure with additional constant arguments.

>No Mysterious Auxiliaries!

>When defining an auxiliary procedure, always specify what it does on all arguments, not just the initial values.

# Exercises

```Scheme
Exercise 1.16
(define invert
  (lambda (lst)
    (if (null? lst) '()
        (cons
         (list (car (cdr (car lst))) (car (car lst)))
         (invert (cdr lst))))))
```
