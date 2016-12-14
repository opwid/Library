# Specifying Data via Interfaces
Every time we decide to represent a certain set of quantities in a particular way, we are defining a new data type: the data type whose values are those representations and whose operations are the procedures that manipulate those entities. If we decide to change the representation of some data for any reason, we must be able to locate all parts of a program that are dependent on the representation. This is accomplished using the technique of _data abstraction_. Data abstraction divides a data type into two pieces: an _interface_ and an _implementation_. The interface tells us what the data of the type represents, what the operations on the data are, and what properties these operations may be relied on to have. The implementation provides a specific representation of the data and code for the operations that make use of that data representation. A data type that is abstract in this way is said to be an abstract data type. The rest of the program, the client of the data type, manipulates the new data only through the operations specified in the interface. Thus if we wish to change the representation of the data, all we must do is change the implementation of the operations in the interface.  
When the client manipulates the values of the data type only through the procedures in the interface, we say that the client code is _representation-independent_, because then the code does not rely on the representation of the values in the data type. All the knowledge about how the data is represented must therefore reside in the code of the implementation. The most important part of an implementation is the specification of how the data is represented. We use the notation ⌈v⌉ for "the representation of data v."  

To make this clearer, let us consider a simple example: the data type of natural numbers. The data to be represented are the natural numbers. The interface is to consist of four procedures: _zero_, _is-zero?_, _successor_, and _predecessor_. Of course, not just any set of procedures will be acceptable as an implementation of this interface. A set of procedures will be acceptable
as implementations of zero, is-zero?, successor, and predecessor only if they satisfy the four equations  

![INT](https://github.com/opwid/Library/blob/master/Essentials-of-Programming-Languages/Images/int.png)  

This specification does not dictate how these natural numbers are to be represented. It requires only that these procedures conspire to produce the specified behavior. The specification says nothing about (predecessor (zero)), so under this specification any behavior would be acceptable.  

We can now write client programs that manipulate natural numbers, and we are guaranteed that they will get correct answers, no matter what representation is in use. For example,
```Scheme
(define plus
  (lambda (x y)
    (if (is-zero? x) y
        (successor (plus (predecessor x) y)))))
```
will satisfy (plus ⌈x⌉ ⌈y⌉) = ⌈x + y⌉, no matter what implementation of the natural numbers we use.  

Most interfaces will contain some __constructors__ that build elements of the data type, and some __observers__ that extract information from values of the data type. Here we have three constructors, zero, successor, and predecessor, and one observer, is-zero?.  

There are many possible representations of this interface. Let us consider three of them.  

* _Unary representation_: In the unary representation, the natural number n is represented by a list of n #t's. Thus, 0 is represented by (), 1 is represented by (#t), 2 is represented by (#t #t), etc. We can define this representation inductively by:  
⌈0⌉ = ()  
⌈n + 1⌉ = (#t . ⌈n⌉)  
In this representation, we can satisfy the specification by writing  

 ```Scheme  
(define zero (lambda () '()))  
(define is-zero? (lambda (n) (null? n)))  
(define successor (lambda (n) (cons #t n)))  
(define predecessor (lambda (n) (cdr n)))  
```

* _Scheme number representation_: In this representation, we simply use Scheme's internal representation of numbers (which might itself be quite complicated!). We let ⌈n⌉ be the Scheme integer n, and define the four required entities by  

 ```Scheme
(define zero (lambda () 0))
(define is-zero? (lambda (n) (zero? n)))
(define successor (lambda (n) (+ n 1)))
(define predecessor (lambda (n) (- n 1)))
```

* _Bignum representation_: In the bignum representation, numbers are represented in base N, for some large integer N. The representation becomes a list consisting of numbers between 0 and N - 1 (sometimes called bigits rather than digits). This representation makes it easy to represent integers that are much larger than can be represented in a machine word. For our purposes, it is convenient to keep the list with least-significant bigit first. We can define the representation inductively by  
![INT2](https://github.com/opwid/Library/blob/master/Essentials-of-Programming-Languages/Images/int2.png)  
So if N = 16, then ⌈33⌉ = (1 2) and ⌈258⌉ = (2 0 1)

None of these implementations enforces data abstraction. There is nothing to prevent a client program from looking at the representation and determining whether it is a list or a Scheme integer. On the other hand, some languages provide direct support for data abstractions: they allow the programmer to create new interfaces and check that the new data is manipulated only through the procedures in the interface. If the representation of a type is hidden, so it cannot be exposed by any operation (including printing), the type is said to be _opaque_. Otherwise, it is said to be _transparent_. Scheme does not provide a standard mechanism for creating new opaque types. Thus we settle for an intermediate level of abstraction: we define interfaces and rely on the writer of the client program to be discreet and use only the procedures in the interfaces.


# Representation Strategies for Data Types

In this section we introduce some strategies for representing data types. We illustrate these choices using a data type of __environments__. An environment associates a value with each element of a finite set of variables. An environment may be used to associate variables with their values in a programming language implementation. A compiler may also use an environment to associate each variable name with information about that variable. Variables may be represented in any way we please, so long as we can check two variables for equality. We choose to represent variables using Scheme symbols, but in a language without a symbol data type, variables could be represented by strings, by references into a hash table, or even by numbers.

## The Environment Interface
An environment is a function whose domain is a finite set of variables, and whose range is the set of all Scheme values. Since we adopt the usual mathematical convention that a finite function is a finite set of ordered pairs, then we need to represent all sets of the form {(var<sub>1</sub>, val<sub>1</sub>), ... , (var<sub>n</sub>, val<sub>n</sub>)} where the var<sub>i</sub> are distinct variables and the val<sub>i</sub> are any Scheme values. We sometimes call the value of the variable var in an environment env its binding in env.  

The interface for this data type has three procedures, specified as follows:  
![ENV](https://github.com/opwid/Library/blob/master/Essentials-of-Programming-Languages/Images/env.png)  

The procedure empty-env, applied to no arguments, must produce a representation of the empty environment; apply-env applies a representation of an environment to a variable and (extend-env var val env) produces a new environment that behaves like env, except that its value at variable var is val. For example, the expression
```Scheme
(define e
  (extend-env 'd 6
              (extend-env 'y 8
                          (extend-env 'x 7
                                      (extend-env 'y 14
                                                  (empty-env))))))
```
defines an environment e such that e(d) = 6, e(x) = 7, e(y) = 8 and e is undefined on any other variables. This is, of course, only one of many different ways of building this environment. For instance, in the example above the binding of y to 14 is overridden by its later binding to 8. In this example, empty-env and extend-env are the constructors, and apply-env is the only observer.

## Data Structure Representation

We can obtain a representation of environments by observing that every environment can be built by starting with the empty environment and applying extend-env n times, for some n ≥ 0.  

So every environment can be built by an expression in the following grammar:
```
Env-exp ::= (empty-env)
        ::= (extend-env Identifier Scheme-value Env-exp)
```
The procedure apply-env looks at the data structure env representing an environment, determines what kind of environment it represents, and does the right thing. If it represents the empty environment, then an error is reported. If it represents an environment built by extend-env, then it checks to see if the variable it is looking for is the same as the one bound in the environment. If it is, then the saved value is returned. Otherwise, the variable is looked up in the saved environment.
```Scheme
Env = (empty-env) | (extend-env Var SchemeVal Env)
Var = Sym

empty-env : () -> Env
(define empty-env
  (lambda () (list 'empty-env)))

extend-env : Var X SchemeVal X Env -> Env
(define extend-env
  (lambda (var val env)
    (list 'extend-env var val env)))

apply-env : Env X Var -> SchemeVal
(define apply-env
  (lambda (env search-var)
    (cond
      ((eqv? (car env) 'empty-env)
       (report-no-binding-found search-var))
      ((eqv? (car env) 'extend-env)
       (let ((saved-var (cadr env))
             (saved-val (caddr env))
             (saved-env (cadddr env)))
         (if (eqv? search-var saved-var)
             saved-val
             (apply-env saved-env search-var))))
      (else
       (report-invalid-env env)))))

(define report-no-binding-found
  (lambda (search-var)
    (eopl:error 'apply-env "No binding for ~s" search-var)))

(define report-invalid-env
  (lambda (env)
    (eopl:error 'apply-env "Bad environment: ~s" env)))
```
## Procedural Representation
The environment interface has an important property: it has exactly one observer, apply-env. This allows us to represent an environment as a Scheme procedure that takes a variable and returns its associated value. To do this, we define empty-env and extend-env to return procedures that, when applied, do the same thing that apply-env did in the preceding section. This gives us the following implementation.
```Scheme
Env = Var -> SchemeVal

empty-env : () -> Env
(define empty-env
  (lambda ()
    (lambda (search-var)
      (report-no-binding-found search-var))))

extend-env : Var X SchemeVal X Env -> Env
(define extend-env
  (lambda (saved-var saved-val saved-env)
    (lambda (search-var)
      (if (eqv? search-var saved-var)
          saved-val
          (apply-env saved-env search-var)))))

apply-env : Env X Var -> SchemeVal
(define apply-env
  (lambda (env search-var)
    (env search-var)))
```
If the empty environment, created by invoking empty-env, is passed any variable whatsoever, it indicates with an error message that the given variable is not in its domain. The procedure extend-env returns a new procedure that represents the extended environment. This procedure, when passed a variable search-var, checks to see if the variable it is looking for is the same as the one bound in the environment. If it is, then the saved value is returned. Otherwise, the variable is looked up in the saved environment. We call this a _procedural representation_, in which the data is represented by its action under apply-env.  

The case of a data type with a single observer is less rare than one might think. For example, if the data being represented is a set of functions, then it can be represented by its action under application. In this case, we can extract the interface and the procedural representation by the following recipe:  

* Identify the lambda expressions in the client code whose evaluation yields values of the type. Create a constructor procedure for each such lambda expression. The parameters of the constructor procedure will be the free variables of the lambda expression. Replace each such lambda expression in the client code by an invocation of the corresponding constructor.
* Define an apply- procedure like apply-env above. Identify all the places in the client code, including the bodies of the constructor procedures, where a value of the type is applied. Replace each such application by an invocation of the apply- procedure.

If these steps are carried out, the interface will consist of all the constructor procedures and the apply- procedure, and the client code will be representation-independent: it will not rely on the representation, and we will be free to substitute another implementation of the interface.  

If the implementation language does not allow higher-order procedures, then one can perform the additional step of implementing the resulting interface using a data structure representation and the interpreter recipe. This process is called _defunctionalization_. The derivation of the data structure representation of environments is a simple example of defunctionalization.


# Interfaces for Recursive Data Types
Our interface for lambda expressions will have constructors and two kinds of observers: predicates and extractors.  
The constructors are:  
```
var-exp : Var -> Lc-exp
lambda-exp : Var X Lc-exp -> Lc-exp
app-exp : Lc-exp X Lc-exp -> Lc-exp
```
The predicates are:
```
var-exp? : Lc-exp -> Bool
lambda-exp? : Lc-exp -> Bool
app-exp? : Lc-exp -> Bool

```
The extractors are:
```
var-exp -> var : Lc-exp -> Var
lambda-exp -> bound-var : Lc-exp -> Var
lambda-exp -> body : Lc-exp -> Lc-exp
app-exp -> rator : Lc-exp -> Lc-exp
app-exp -> rand : Lc-exp -> Lc-exp
```
Each of these extracts the corresponding portion of the lambda-calculus expression. We can now write a version of occurs-free? that depends only on the interface.
```Scheme
occurs-free? : Sym X LcExp -> Bool
(define occurs-free?
  (lambda (search-var exp)
    (cond
      ((var-exp? exp) (eqv? search-var (var-exp->var exp)))
      ((lambda-exp? exp)
       (and
        (not (eqv? search-var (lambda-exp->bound-var exp)))
        (occurs-free? search-var (lambda-exp->body exp))))
      (else
       (or
        (occurs-free? search-var (app-exp->rator exp))
        (occurs-free? search-var (app-exp->rand exp)))))))
```
This works on any representation of lambda-calculus expressions, so long as they are built using these constructors.  

We can write down a general recipe for designing an interface for a recursive data type:
> Designing an interface for a recursive data type

>1- Include one constructor for each kind of data in the data type.  
>2- Include one predicate for each kind of data in the data type.  
>3- Include one extractor for each piece of data passed to a constructor of the data type.

# A Tool for Defining Recursive Data Types
For complicated data types, applying the recipe for constructing an interface can quickly become tedious. In this section, we introduce a tool for automatically constructing and implementing such interfaces in Scheme. The interfaces constructed by this tool will be similar, but not identical, to the interface constructed in the preceding section.  

Consider again the data type of lambda-calculus expressions, as discussed in the preceding section. We can implement an interface for lambda-calculus expressions by writing
```Scheme
(define-datatype lc-exp lc-exp?
  (var-exp
   (var identifier?))
  (lambda-exp
   (bound-var identifier?)
   (body lc-exp?))
  (app-exp
   (rator lc-exp?)
   (rand lc-exp?)))
```
Here the names var-exp, var, bound-var, app-exp, rator, and rand abbreviate variable expression, variable, bound variable, application expression, operator, and operand, respectively.  
This expression declares three constructors, var-exp, lambda-exp, and app-exp, and a single predicate lc-exp?. The three constructors check their arguments with the predicates identifier? and lc-exp? to make sure that the arguments are valid, so if an lc-exp is constructed using only these constructors, we can be certain that it and all its subexpressions are legal lc-exps. This allows us to ignore many checks while processing lambda expressions.  

In place of the various predicates and extractors, we use the form `cases` to determine the variant to which an object of a data type belongs, and to extract its components. To illustrate this form, we can rewrite occurs-free? using the data type lc-exp:
```Scheme
occurs-free? : Sym X LcExp -> Bool
(define occurs-free?
  (lambda (search-var exp)
    (cases lc-exp exp
      (var-exp (var) (eqv? var search-var))
      (lambda-exp (bound-var body)
                  (and
                   (not (eqv? search-var bound-var))
                   (occurs-free? search-var body)))
      (app-exp (rator rand)
               (or
                (occurs-free? search-var rator)
                (occurs-free? search-var rand))))))
```
To see how this works, assume that exp is a lambda-calculus expression that was built by app-exp. For this value of exp, the app-exp case would be selected, rator and rand would be bound to the two subexpressions, and the expression
```Scheme
(or
  (occurs-free? search-var rator)
  (occurs-free? search-var rand))
```
would be evaluated, just as if we had written
```Scheme
(if (app-exp? exp)
  (let ((rator (app-exp->rator exp))
        (rand (app-exp->rand exp)))
    (or
      (occurs-free? search-var rator)
      (occurs-free? search-var rand)))
...)
```
The recursive calls to occurs-free? work similarly to finish the calculation.  

In general, a define-datatype declaration has the form  
![DDT](https://github.com/opwid/Library/blob/master/Essentials-of-Programming-Languages/Images/ddt.png)  

This creates a data type, named type-name, with some variants. Each variant has a variant-name and zero or more fields, each with its own field-name and associated predicate. No two types may have the same name and no two variants, even those belonging to different types, may have the same name. Also, type names cannot be used as variant names. Each field predicate must be a Scheme predicate.  

For each variant, a new constructor procedure is created that is used to create data values belonging to that variant. These procedures are named after their variants. If there are n fields in a variant, its constructor takes n arguments, tests each of them with its associated predicate, and returns a new value of the given variant with the i-th field containing the i-th argument value.  

The _type-predicate-name_ is bound to a predicate. This predicate determines if its argument is a value belonging to the named type.  

A record can be defined as a data type with a single variant. To distinguish data types with only one variant, we use a naming convention. When there is a single variant, we name the constructor _a-type-name_ or _an-type-name_;otherwise, the constructors have names like _variant-name-type-name_.  

Data types built by define-datatype may be mutually recursive. For example, consider the grammar for s-lists from section 1.1:
```
S-list ::= ({S-exp}*)
S-exp ::= Symbol | S-list
```
The data in an s-list could be represented by the data type s-list defined by
```Scheme
(define-datatype s-list s-list?
  (empty-s-list)
  (non-empty-s-list
   (first s-exp?)
   (rest s-list?)))

(define-datatype s-exp s-exp?
  (symbol-s-exp
   (sym symbol?))
  (s-list-s-exp
   (slst s-list?)))
```
The data type s-list gives its own representation of lists by using (empty-s-list) and non-empty-s-list in place of () and cons; if we wanted to specify that Scheme lists be used instead, we could have written
```Scheme
(define-datatype s-list s-list?
  (an-s-list
   (sexps (list-of s-exp?))))

(define list-of
  (lambda (pred)
    (lambda (val)
      (or (null? val)
          (and (pair? val)
               (pred (car val))
               ((list-of pred) (cdr val)))))))
```
Here (list-of pred) builds a predicate that tests to see if its argument is a list, and that each of its elements satisfies pred.  

The general syntax for cases is 
```Scheme
(cases type-name expression
  {(variant-name ({field-name}*) consequent)}*
  (else default))
```
The form specifies the type, the expression yielding the value to be examined, and a sequence of clauses. Each clause is labeled with the name of a variant of the given type and the names of its fields. The else clause is optional. First, expression is evaluated, resulting in some value v of type-name. If v is a variant of variant-name, then the corresponding clause is selected. Each of the field-names is bound to the value of the corresponding field of v. Then the consequent is evaluated within the scope of these bindings and its value returned. If v is not one of the variants, and an else clause has been specified, default is evaluated and its value returned. If there is no else clause, then there must be a clause for every variant of that data type.  

The form cases binds its variables positionally: the i-th variable is bound to the value in the i-th field. So we could just as well have written
```Scheme
(app-exp (exp1 exp2)
         (or
          (occurs-free? search-var exp1)
          (occurs-free? search-var exp2)))
```
instead of 
```Scheme
(app-exp (rator rand)
         (or
          (occurs-free? search-var rator)
          (occurs-free? search-var rand)))
```
The forms define-datatype and cases provide a convenient way of defining an inductive data type, but it is not the only way. Depending on the application, it may be valuable to use a special-purpose representation that is more compact or efficient, taking advantage of special properties of the data. These advantages are gained at the expense of having to write the procedures in the interface by hand.  

The form define-datatype is an example of a _domain-specific language_. A domain-specific language is a small language for describing a single task among a small, well-defined set of tasks. In this case, the task was defining a recursive data type. Such a language may lie inside a general-purpose language, as define-datatype does, or it may be a standalone language with its own set of tools. In general, one constructs such a language by identifying the possible variations in the set of tasks, and then designing a language that describes those variations. This is often a very useful strategy.

# Abstract Syntax and Its Representation

A grammar usually specifies a particular representation of an inductive data type: one that uses the strings or values generated by the grammar. Such a representation is called _concrete syntax_, or _external_ representation.  

Consider, for example, the set of lambda-calculus expressions defined in definition 1.1.8. This gives a concrete syntax for lambda-calculus expressions. We might have used some other concrete syntax for lambda-calculus expressions. For example, we could have written
```
LcExp ::= Identifier
      ::= proc Identifier => Lc-exp
      ::= Lc-exp (Lc-exp)

```
to define lambda-calculus expressions as a different set of strings.  

In order to process such data, we need to convert it to an _internal_ representation. The define-datatype form provides a convenient way of defining such an internal representation. We call this _abstract syntax_. In the abstract syntax, terminals such as parentheses need not be stored, because they convey no information. On the other hand, we want to make sure that the data structure allows us to determine what kind of lambda-calculus expression it represents, and to extract its components.  

It is convenient to visualize the internal representation as an _abstract syntax tree_. The following figure shows the abstract syntax tree of the lambda-calculus expression `(lambda (x) (f (f x)))`, using the data type lc-exp. Each internal node of the tree is labeled with the associated production name. Edges are labeled with the name of the corresponding nonterminal occurrence. Leaves correspond to terminal strings.  

![AST](https://github.com/opwid/Library/blob/master/Essentials-of-Programming-Languages/Images/ast.png) 

To create an abstract syntax for a given concrete syntax, we must name each production of the concrete syntax and each occurrence of a nonterminal in each production. It is straightforward to generate define-datatype declarations for the abstract syntax. We create one define-datatype for each nonterminal, with one variant for each production.  

We can summarize the choices we have made in figure using the following concise notation:
```Scheme
Lc-exp ::= Identifier
           var-exp (var)
       ::= (lambda (Identifier) Lc-exp)
           lambda-exp (bound-var body)
       ::= (Lc-exp Lc-exp)
           app-exp (rator rand)
```
Such notation, which specifies both concrete and abstract syntax, is used throughout this book.  

Having made the distinction between concrete syntax, which is primarily useful for humans, and abstract syntax, which is primarily useful for computers, we now consider how to convert from one syntax to the other.  

If the concrete syntax is a set of strings of characters, it may be a complex undertaking to derive the corresponding abstract syntax tree. This task is called _parsing_ and is performed by a _parser_. Because writing a parser is difficult in general, it is best performed by a tool called a _parser generator_. A parser generator takes as input a grammar and produces a parser. Since the grammars are processed by a tool, they must be written in some machine-readable language: a domain-specific language for writing grammars. There are many parser generators available.  

If the concrete syntax is given as a set of lists, the parsing process is considerably simplified. For example, the grammar for lambda-calculus expressions at the beginning of this section specified a set of lists, as did the grammar for define-datatype. In this case, the Scheme read routine automatically parses strings into lists and symbols. It is then easier to parse these list structures into abstract syntax trees as in parse-expression.
```Scheme
parse-expression : SchemeVal -> LcExp
(define parse-expression
  (lambda (datum)
    (cond
      ((symbol? datum) (var-exp datum))
      ((pair? datum)
       (if (eqv? (car datum) 'lambda)
           (lambda-exp
            (car (cadr datum))
            (parse-expression (caddr datum)))
           (app-exp
            (parse-expression (car datum))
            (parse-expression (cadr datum)))))
      (else (report-invalid-concrete-syntax datum)))))
```
It is usually straightforward to convert an abstract syntax tree back to a list-and-symbol representation. If we do this, the Scheme print routines will then display it in a list-based concrete syntax. This is performed by unparse-lc-exp:
```Scheme
unparse-lc-exp : LcExp -> SchemeVal
(define unparse-lc-exp
  (lambda (exp)
    (cases lc-exp exp
      (var-exp (var) var)
      (lambda-exp (bound-var body)
                  (list 'lambda (list bound-var)
                        (unparse-lc-exp body)))
      (app-exp (rator rand)
               (list
                (unparse-lc-exp rator) (unparse-lc-exp rand))))))
```










