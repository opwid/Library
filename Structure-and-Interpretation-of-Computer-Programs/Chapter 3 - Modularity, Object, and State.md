
# Assignment and Local State
An object is said to "have state" if its behavior is influenced by its history. We can characterize an object's state by one or more state variables , which among them maintain enough information about history to determine the object's current behavior.  

In a system composed of many objects, the objects are rarely completely independent. Each may influence the states of others through interactions, which serve to couple the state variables of one object to those of other objects. Indeed, the view that a system is composed of separate objects is most useful when the state variables of the system can be grouped into closely coupled subsystems that are only loosely coupled to other subsystems. This view of a system can be a powerful framework for organizing computational models of the system. For such a model to be modular, it should be decomposed into computational objects that model the actual objects in the system. Each computational object must have its own local state variables describing the actual object's state. Since the states of objects in the system being modeled change over time, the state variables of the corresponding computational objects must also change. If we choose to model the flow of time in the system by the elapsed time in the computer, then we must have a way to construct computational objects whose behaviors change as our programs run. In particular, if we wish to model state variables by ordinary symbolic names in the programming language, then the language must provide an assignment operator to enable us to change the value associated with a name.



## Local State Variables
If we begin with $100 in the account, we should obtain the following sequence of responses using withdraw:
```
(withdraw 25)
>>> 75
(withdraw 25)
>>> 50
(withdraw 15)
>>> 35
```
Observe that the expression (withdraw 25), evaluated twice, yields different values. This is a new kind of behavior for a procedure.  

To implement withdraw, we can use a variable balance to indicate the balance of money in the account and define withdraw as a procedure that accesses balance. The withdraw procedure checks to see if balance is at least as large as the requested amount. If so, withdraw decrements balance by amount and returns the new value of balance. Otherwise, withdraw returns the Insufficient funds message. Here are the definitions of balance and withdraw:
```Scheme
(define balance 100)
(define (withdraw amount)
  (if (>= balance amount)
      (begin (set! balance (- balance amount))
             balance)
      "Insufficient funds"))
```

Decrementing balance uses the set! special form, whose syntax is

```
(set! <name> <new-value>)
```
Set! changes \<name> so that its value is the result obtained by evaluating \<new-value>.  

Withdraw also uses the begin special form to cause two expressions to be evaluated in the case where the if test is true: first decrementing balance and then returning the value of balance. In general, evaluating the expression
```
(begin <exp 1> <exp 2> ... <exp k> )
```
causes the expressions \<exp 1> through \<exp k> to be evaluated in sequence and the value of the final expression \<exp k> to be returned as the value of the entire begin form.  

As specified above, balance is a name defined in the global environment and is freely accessible to be examined or modified by any procedure. It would be much better if we could somehow make balance internal to withdraw, so that withdraw would be the only procedure that could access balance directly and any other procedure could access balance only indirectly (through calls to withdraw). We can make balance internal to withdraw by rewriting the definition as follows:
```Scheme
(define new-withdraw
  (let ((balance 100))
    (lambda (amount)
      (if (>= balance amount)
          (begin (set! balance (- balance amount))
                 balance)
          "Insufficient funds"))))
```
What we have done here is use let to establish an environment with a local variable balance, bound to the initial value 100. Within this local environment, we use lambda to create a procedure that takes amount as an argument and behaves like our previous withdraw procedure. This procedure -returned as the result of evaluating the let expression- is new-withdraw, which behaves in precisely the same way as withdraw but whose variable balance is not accessible by any other procedure. Combining set! with local variables is the general programming technique we will use for constructing computational objects with local state. Unfortunately, as soon as we introduce assignment into our language, substitution is no longer an adequate model of procedure application. In order to really understand a procedure such as new-withdraw, we will need to develop a new model of procedure application. In The Environment Model of Evaluation section we will introduce such a model, together with an explanation of set! and local variables.
***
Closure https://www.youtube.com/watch?v=swU3c34d2NQ
```Scheme
(define count 0)
(define (inc x)
    (lambda ()
      (set! count (+ count x))
      count))

(define increment-by-3 (inc 3))
(define increment-by-2 (inc 2))

(increment-by-3)
>>> 3
(increment-by-2)
>>> 5
(increment-by-3)
>>> 8
```
```Scheme
(define (inc x)
  (define count 0)
    (lambda ()
      (set! count (+ count x))
      count))

(define increment-by-3 (inc 3))
(define increment-by-2 (inc 2))

(increment-by-3)
>>> 3
(increment-by-2)
>>> 2
(increment-by-3)
>>> 6
```
```Scheme
(define (inc x)
  (let ((count 0))
    (lambda ()
      (set! count (+ count x))
      count)))

(define increment-by-3 (inc 3))
(define increment-by-2 (inc 2))

(increment-by-3)
>>> 3
(increment-by-2)
>>> 2
(increment-by-3)
>>> 6
```
***
The following procedure, make-withdraw, creates "withdrawal processors." The formal parameter balance in make-withdraw specifies the initial amount of money in the account.
```Scheme
(define (make-withdraw balance)
  (lambda (amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds")))
```
Make-withdraw can be used as follows to create two objects W1 and W2:
```Scheme
(define W1 (make-withdraw 100))
(define W2 (make-withdraw 100))
```
```Scheme
(W1 50)
>>> 50
(W2 70)
>>> 30
(W1 40)
>>> 10
```
Observe that W1 and W2 are completely independent objects, each with its own local state variable balance. Withdrawals from one do not affect the other.  

We can also create objects that handle deposits as well as withdrawals, and thus we can represent simple bank accounts. Here is a procedure that returns a "bank-account object" with a specified initial balance:
```Scheme
(define (make-account balance)
  (define (withdraw amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds"))
  (define (deposit amount)
    (set! balance (+ balance amount))
    balance)
  (define (dispatch m)
    (cond ((eq? m 'withdraw) withdraw)
          ((eq? m 'deposit) deposit)
          (else (error "Unknown request: MAKE-ACCOUNT"
                       m))))
  dispatch)
```
Each call to make-account sets up an environment with a local state variable balance. Within this environment, make-account defines procedures deposit and withdraw that access balance and an additional procedure dispatch that takes a "message" as input and returns one of the two local procedures. The dispatch procedure itself is returned as the value that represents the bank-account object. This is precisely the message-passing style of programming, although here we are using it in conjunction with the ability to modify local variables.  
Make-account can be used as follows:
```Scheme
(define acc (make-account 100))
((acc 'withdraw) 50)
>>> 50
((acc 'withdraw) 60)
>>> "Insufficient funds"
((acc 'deposit) 40)
>>> 90
((acc 'withdraw) 60)
>>> 30
```
Each call to acc returns the locally defined deposit or withdraw procedure, which is then applied to the specified amount. As was the case with make-withdraw, another call to make-account
```Scheme
(define acc2 (make-account 100))
```
will produce a completely separate account object, which maintains its own local balance.


## The Benefits of Introducing Assignment
By introducing assignment and the technique of hiding state in local variables, we are able to structure systems in a more modular fashion than if all state had to be manipulated explicitly, by passing additional parameters. Unfortunately, as we shall see, the story is not so simple.



## The Costs of Introducing Assignment
As we have seen, the set! operation enables us to model objects that have local state. However, this advantage comes at a price. Our programming language can no longer be interpreted in terms of the substitution model of procedure application. Moreover, no simple model with "nice" mathematical properties can be an adequate framework for dealing with objects and assignment in programming languages.  
So long as we do not use assignments, two evaluations of the same procedure with the same arguments will produce the same result, so that procedures can be viewed as computing mathematical functions. Programming without any use of assignments, as we did throughout the first two chapters of this book, is accordingly known as _functional programming_.  
The trouble here is that substitution is based ultimately on the notion that the symbols in our language are essentially names for values. But as soon as we introduce set! and the idea that the value of a variable can change, a variable can no longer be simply a name. Now a variable somehow refers to a place where a value can be stored, and the value stored at this place can change.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Sameness and change</h3>
A language that supports the concept that "equals can be substituted for equals" in an expression without changing the value of the expression is said to be referentially transparent. Referential transparency is violated when we include set! in our computer language. This makes it tricky to determine when we can simplify expressions by substituting equivalent expressions. Consequently, reasoning about programs that use assignment becomes drastically more difficult.  

Once we forgo referential transparency, the notion of what it means for computational objects to be "the same" becomes difficult to capture in a formal way. Indeed, the meaning of "same" in the real world that our programs model is hardly clear in itself. In general, we can determine that two apparently identical objects are indeed "the same one" only by modifying one object and then observing whether the other object has changed in the same way. But how can we tell if an object has "changed" other than by observing the "same" object twice and seeing whether some property of the object differs from one observation to the next? Thus, we cannot determine "change" without some a priori notion of "sameness," and we cannot determine sameness without observing the effects of change.  

As an example of how this issue arises in programming, consider the situation where Peter and Paul have a bank account with $100 in it. There is a substantial difference between modeling this as
```Scheme
(define peter-acc (make-account 100))
(define paul-acc (make-account 100))
```
and modelling it as
```Scheme
(define peter-acc (make-account 100))
(define paul-acc peter-acc)
```
In the first situation, the two bank accounts are distinct. Transactions made by Peter will not affect Paul's account, and vice versa. In the second situation, however, we have defined paul-acc to be the same thing as peter-acc. In effect, Peter and Paul now have a joint bank account, and if Peter makes a withdrawal from peter-acc Paul will observe less money in paul-acc. These two similar but distinct situations can cause confusion in building computational models. With the shared account, in particular, it can be especially confusing that there is one object (the bank account) that has two different names (peter-acc and paul-acc); if we are searching for all the places in our program where paul-acc can be changed, we must remember to look also at things that change peter-acc.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Pitfalls of imperative programming</h3>
In contrast to functional programming, programming that makes extensive use of assignment is known as imperative programming. In addition to raising complications about computational models, programs written in imperative style are susceptible to bugs that cannot occur in functional programs. The complexity of imperative programs becomes even worse if we consider applications in which several processes execute concurrently. We will return to this in the next. First, however, we will address the issue of providing a computational model for expressions that involve assignment, and explore the uses of objects with local state in designing simulations.




# The Environment Model of Evaluation
In the presence of assignment, a variable can no longer be considered to be merely a name for a value. Rather, a variable must somehow designate a "place" in which values can be stored. In our new model of evaluation, these places will be maintained in structures called environments.  

An environment is a sequence of frames. Each frame is a table (possibly empty) of bindings, which associate variable names with their corresponding values. (A single frame may contain at most one binding for any variable.) Each frame also has a pointer to its enclosing environment, unless, for the purposes of discussion, the frame is considered to be global. The value of a variable with respect to an environment is the value given by the binding of the variable in the first frame in the environment that contains a binding for that variable. If no frame in the sequence specifies a binding for the variable, then the variable is said to be unbound in the environment.  

Figure 3.1 shows a simple environment structure consisting of three frames, labeled I, II, and III. In the diagram, A, B, C, and D are pointers to environments. C and D point to the same environment. The variables z and x are bound in frame II, while y and x are bound in frame I. The value of x in environment D is 3. The value of x with respect to environment B is also 3. This is determined as follows: We examine the first frame in the sequence (frame III) and do not find a binding for x, so we proceed to the enclosing environment D and find the binding in frame I. On the other hand, the value of x in environment A is 7, because the first frame in the sequence (frame II) contains a binding of x to 7. With respect to environment A, the binding of x to 7 in frame II is said to shadow the binding of x to 3 in frame I.  

![ENV](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/env.png)  

In our model of evaluation we will always speak of evaluating an expression with respect to some environment. To describe interactions with the interpreter, we will suppose that there is a global environment, consisting of a single frame (with no enclosing environment) that includes values for the symbols associated with the primitive procedures. For example, the idea that + is the symbol for addition is captured by saying that the symbol + is bound in the global environment to the primitive addition procedure.


## The Rules of Evaluation
In the environment model of evaluation, a procedure is always a pair consisting of some code and a pointer to an environment. Procedures are created in one way only: by evaluating a λ-expression. This produces a procedure whose code is obtained from the text of the λ-expression and whose environment is the environment in which the λ-expression was evaluated to produce the procedure. For example, consider the procedure definition
```Scheme
(define (square x)
  (* x x))
```
evaluated in the global environment. The procedure definition syntax is just syntactic sugar for an underlying implicit λ-expression. It would have been equivalent to have used
```Scheme
(define square
  (lambda (x) (* x x)))
```
which evaluates `(lambda (x) (* x x))` and binds square to the resulting value, all in the global environment.  

Figure 3.2 shows the result of evaluating this define expression. The procedure object is a pair whose code specifies that the procedure has one formal parameter, namely x, and a procedure body (\* x x). The environment part of the procedure is a pointer to the global environment, since that is the environment in which the λ-expression was evaluated to produce the procedure. A new binding, which associates the procedure object with the symbol square, has been added to the global frame. In general, define creates definitions by adding bindings to frames.  

![SQUARE_ENV](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/square_env.png)  

Now that we have seen how procedures are created, we can describe how procedures are applied. The environment model specifies: To apply a procedure to arguments, create a new environment containing a frame that binds the parameters to the values of the arguments. The enclosing environment of this frame is the environment specified by the procedure. Now, within this new environment, evaluate the procedure body.  

![APPLY_ENV](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/apply_env.png)  

To show how this rule is followed, Figure 3.3 illustrates the environment structure created by evaluating the expression (square 5) in the global environment, where square is the procedure generated in Figure 3.2. Applying the procedure results in the creation of a new environment, labeled E1 in the figure, that begins with a frame in which x, the formal parameter for the procedure, is bound to the argument 5. The pointer leading upward from this frame shows that the frame's enclosing environment is the global environment. The global environment is chosen here, because this is the environment that is indicated as part of the square procedure object. Within E1, we evaluate the body of the procedure, (\* x x). Since the value of x in E1 is 5, the result is (\* 5 5), or 25.  

The environment model of procedure application can be summarized by two rules:  

* A procedure object is applied to a set of arguments by constructing a frame, binding the formal parameters of the procedure to the arguments of the call, and then evaluating the body of the procedure in the context of the new environment constructed. The new frame has as its enclosing environment the environment part of the procedure object being applied.
* A procedure is created by evaluating a λ-expression relative to a given environment. The resulting procedure object is a pair consisting of the text of the λ-expression and a pointer to the environment in which the procedure was created.

We also specify that defining a symbol using define creates a binding in the current environment frame and assigns to the symbol the indicated value. Finally, we specify the behavior of set!, the operation that forced us to introduce the environment model in the first place. Evaluating the expression (set! \<variable> \<value>) in some environment locates the binding of the variable in the environment and changes that binding to indicate the new value. That is, one finds the first frame in the environment that contains a binding for the variable and modifies that frame. If the variable is unbound in the environment, then set! signals an error.

## Applying Simple Procedures

We can analyze the following example using the environment model.
```Scheme
(define (square x)
  (* x x))
(define (sum-of-squares x y)
  (+ (square x) (square y)))
(define (f a)
  (sum-of-squares (+ a 1) (* a 2)))
```
Figure 3.4 shows the three procedure objects created by evaluating the definitions of f, square, and sum-of-squares in the global environment. Each procedure object consists of some code, together with a pointer to the global environment.  
In Figure 3.5 we see the environment structure created by evaluating the expression (f 5). The call to f creates a new environment E1 beginning with a frame in which a, the formal parameter of f, is bound to the argument 5. In E1, we evaluate the body of f:
```Scheme
(sum-of-squares (+ a 1) (* a 2))
```
To evaluate this combination, we first evaluate the subexpressions. The first subexpression, sum-of-squares, has a value that is a procedure object. (Notice how this value is found: We first look in the first frame of E1, which contains no binding for sum-of-squares. Then we proceed to the enclosing environment, i.e. the global environment, and find the binding shown in Figure 3.4.) The other two subexpressions are evaluated by applying the primitive operations + and * to evaluate the two combinations (+ a 1) and (\* a 2) to obtain 6 and 10, respectively. Now we apply the procedure object sum-of-squares to the arguments 6 and 10. This results in a new environment E2 in which the formal parameters x and y are bound to the arguments. Within E2 we evaluate the combination (+ (square x) (square y)). This leads us to evaluate (square x), where square is found in the global frame and x is 6. Once again, we set up a new environment, E3, in which x is bound to 6, and within this we evaluate the body of square, which is (\* x x). Also as part of applying sum-of-squares, we must evaluate the subexpression (square y), where y is 10. This second call to square creates another environment, E4, in which x, the formal parameter of square, is bound to 10. And within E4 we must evaluate (\* x x).  

![3_4](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_4.png)  


![3_5](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_5.png)  

The important point to observe is that each call to square creates a new environment containing a binding for x. We can see here how the different frames serve to keep separate the different local variables all named x. Notice that each frame created by square points to the global environment, since this is the environment indicated by the square procedure object.


## Frames as the Repository of Local State
We can turn to the environment model to see how procedures and assignment can be used to represent objects with local state. As an example, consider the "withdrawal processor" created by calling the procedure
```Scheme
(define (make-withdraw balance)
  (lambda (amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds")))
```
Let us describe the evaluation of
```Scheme
(define W1 (make-withdraw 100))
```
followed by
```Scheme
(W1 50)
>>> 50
```
Figure 3.6 shows the result of defining the make-withdraw procedure in the global environment. This produces a procedure object that contains a pointer to the global environment. So far, this is no different from the examples we have already seen, except that the body of the procedure is itself a λ-expression.  

![3_6](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_6.png)  

The interesting part of the computation happens when we apply the procedure make-withdraw to an argument:
```Scheme
(define W1 (make-withdraw 100))
```
We begin, as usual, by setting up an environment E1 in which the formal parameter balance is bound to the argument 100. Within this environment, we evaluate the body of make-withdraw, namely the λ-expression. This constructs a new procedure object, whose code is as specified by the lambda and whose environment is E1, the environment in which the lambda was evaluated to produce the procedure. The resulting procedure object is the value returned by the call to make-withdraw. This is bound to W1 in the global environment, since the define itself is being evaluated in the global environment. Figure 3.7 shows the resulting
environment structure.  

![3_7](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_7.png)  

Now we can analyze what happens when W1 is applied to an argument:
```Scheme
(W1 50)
>>> 50
```
We begin by constructing a frame in which amount, the formal parameter of W1, is bound to the argument 50. The crucial point to observe is that this frame has as its enclosing environment not the global environment, but rather the environment E1, because this is the environment that is specified by the W1 procedure object. Within this new environment, we evaluate the body of the procedure:
```Scheme
(if (>= balance amount)
    (begin (set! balance (- balance amount))
           balance)
    "Insufficient funds")
```
The resulting environment structure is shown in Figure 3.8. The expression being evaluated references both amount and balance. Amount will be found in the first frame in the environment, while balance will be found by following the enclosing-environment pointer to E1.  

![3_8](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_8.png)  

When the set! is executed, the binding of balance in E1 is changed. At the completion of the call to W1, balance is 50, and the frame that contains balance is still pointed to by the procedure object W1. The frame that binds amount (in which we executed the code that changed balance) is no longer relevant, since the procedure call that constructed it has terminated, and there are no pointers to that frame from other parts of the environment. The next time W1 is called, this will build a new frame that binds amount and whose enclosing environment is E1. We see that E1 serves as the "place" that holds the local state variable for the procedure object W1. Figure 3.9 shows the situation after the call to W1.  

![3_9](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_9.png)  

Observe what happens when we create a second "withdraw" object by making another call to make-withdraw:
```Scheme
(define W2 (make-withdraw 100))
```
This produces the environment structure of Figure 3.10, which shows that W2 is a procedure object, that is, a pair with some code and an environment. The environment E2 for W2 was created by the call to make-withdraw. It contains a frame with its own local binding for balance. On the other hand, W1 and W2 have the same code: the code specified by the λ-expression in the body of make-withdraw. We see here why W1 and W2 behave as independent objects. Calls to W1 reference the state variable balance stored in E1, whereas calls to W2 reference the balance stored in E2. Thus, changes to the local state of one object do not affect the other object.  

![3_10](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_10.png)  




## Internal Definition
Procedures as Black-Box Abstractions section introduced the idea that procedures can have internal definitions, thus leading to a block structure as in the following procedure to compute square roots:
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
  (sqrt-iter 1.0))
```
Now we can use the environment model to see why these internal definitions behave as desired. Figure 3.11 shows the point in the evaluation of the expression (sqrt 2) where the internal procedure good-enough? has been called for the first time with guess equal to 1.  

Observe the structure of the environment. Sqrt is a symbol in the global environment that is bound to a procedure object whose associated environment is the global environment. When sqrt was called, a new environment E1 was formed, subordinate to the global environment, in which the parameter x is bound to 2. The body of sqrt was then evaluated in E1. Since the first expression in the body of sqrt is
```Scheme
(define (good-enough? guess)
  (< (abs (- (square guess) x)) 0.001))
```
evaluating this expression defined the procedure good-enough? in the environment E1. To be more precise, the symbol good-enough? was added to the first frame of E1, bound to a procedure object whose associated environment is E1. Similarly, improve and sqrt-iter were defined as procedures in E1. For conciseness, Figure 3.11 shows only the procedure object for good-enough?.  

![3_11](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_11.png) 


After the local procedures were defined, the expression (sqrt-iter 1.0) was evaluated, still in environment E1. So the procedure object bound to sqrt-iter in E1 was called with 1 as an argument. This created an environment E2 in which guess, the parameter of sqrt-iter, is bound to 1. Sqrt-iter in turn called good-enough? with the value of guess (from E2) as the argument for good-enough?. This set up another environment, E3, in which guess (the parameter of good-enough?) is bound to 1. Although sqrt-iter and good-enough? both have a parameter named guess, these are two distinct local variables located in different frames. Also, E2 and E3 both have E1 as their enclosing environment, because the sqrt-iter and good-enough? procedures both have E1 as their environment part. One consequence of this is that the symbol x that appears in the body of good-enough? will reference the binding of x that appears in E1, namely the value of x with which the original sqrt procedure was called.  

The environment model thus explains the two key properties that make local procedure definitions a useful technique for modularizing programs:

* The names of the local procedures do not interfere with names external to the enclosing procedure, because the local procedure names will be bound in the frame that the procedure creates when it is run, rather than being bound in the global environment.
* The local procedures can access the arguments of the enclosing procedure, simply by using parameter names as free variables. This is because the body of the local procedure is evaluated in an environment that is subordinate to the evaluation environment for the enclosing procedure.




# Modeling with Mutable Data
In Chapter 2 we introduced the discipline of data abstraction, according to which data structures are specified in terms of constructors, which create data objects, and selectors, which access the parts of compound data objects. But we now know that there is another aspect of data that Chapter 2 did not address. The desire to model systems composed of objects that have changing state leads us to the need to modify compound data objects, as well as to construct and select from them. In order to model compound objects with changing state, we will design data abstractions to include, in addition to selectors and constructors, operations called mutators, which modify data objects.  

Chapter 2 introduced pairs as a general-purpose "glue" for synthesizing compound data. We begin this section by defining basic mutators for pairs, so that pairs can serve as building blocks for constructing mutable data objects. These mutators greatly enhance the representational power of pairs, enabling us to build data structures other than the sequences and trees.


## Mutable List Structure
The basic operations on pairs -cons, car, and cdr- can be used to construct list structure and to select parts from list structure, but they are incapable of modifying list structure. The same is true of the list operations we have used so far, such as append and list, since these can be defined in terms of cons, car, and cdr. To modify list structures we need new operations.  

The primitive mutators for pairs are set-car! and set-cdr!. Set-car! takes two arguments, the first of which must be a pair. It modifies this pair, replacing the car pointer by a pointer to the second argument of set-car!.  

As an example, suppose that x is bound to the list ((a b) c d) and y to the list (e f). Evaluating the expression (set-car! x y) modifies the pair to which x is bound, replacing its car by the value of y. The structure x has been modified and would now be printed as ((e f) c d). The pairs representing the list (a b), identified by the pointer that was replaced, are now detached from the original structure.  

The set-cdr! operation is similar to set-car!. The only difference is that the cdr pointer of the pair, rather than the car pointer, is replaced.  

Cons builds new list structure by creating new pairs, while set-car! and set-cdr! modify existing pairs. Indeed, we could implement cons in terms of the two mutators, together with a procedure get-new-pair, which returns a new pair that is not part of any existing list structure. We obtain the new pair, set its car and cdr pointers to the designated objects, and return the new pair as the result of the cons.
```Scheme
(define (cons x y)
  (let ((new (get-new-pair)))
    (set-car! new x)
    (set-cdr! new y)
    new))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Sharing and identity</h3>
The theoretical issues of "sameness" and "change" raised by the introduction of assignment arise in practice when individual pairs are shared among different data objects. For example, consider the structure formed by
```Scheme
(define x (list 'a 'b))
(define z1 (cons x x))
```
As shown in Figure 3.16, z1 is a pair whose car and cdr both point to the same pair x. This sharing of x by the car and cdr of z1 is a consequence of the straightforward way in which cons is implemented. In general, using cons to construct lists will result in an interlinked structure of pairs in which many individual pairs are shared by many different structures.  

![3_16](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_16.png)  

In contrast to Figure 3.16, Figure 3.17 shows the structure created by
```Scheme
(define z2 (cons (list 'a 'b) (list 'a 'b)))
```
In this structure, the pairs in the two (a b) lists are distinct, although the actual symbols are shared.  

![3_17](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_17.png)  

When thought of as a list, z1 and z2 both represent "the same" list, ((a b) a b).
***
While printing cons, Scheme omits dot and surrounding parenthesis of cdr, if the cdr is a list.

```Scheme
(define x (cons (list 'a 'b) (list 'c 'd)))
x
>>> ((a b) c d)

(define y (cons (list 'a 'b) 'c))
y
>>> ((a b) . c)

(define z (cons (cons 'a (list 'b 'c)) (list 'a 'b)))
z
>>> ((a b c) a b)
```
***
In general, sharing is completely undetectable if we operate on lists using only cons, car, and cdr. However, if we allow mutators on list structure, sharing becomes significant. As an example of the difference that sharing can make, consider the following procedure, which modifies the car of the structure to which it is applied:
```Scheme
(define (set-to-wow! x) (set-car! (car x) 'wow) x)
```
Even though z1 and z2 are "the same" structure, applying set-to-wow! to them yields different results. With z1, altering the car also changes the cdr, because in z1 the car and the cdr are the same pair. With z2, the car and cdr are distinct, so set-to-wow! modifies only the car:
```Scheme
z1
>>> ((a b) a b)
(set-to-wow! z1)
>>> ((wow b) wow b)
z2
>>> ((a b) a b)
(set-to-wow! z2)
>>> ((wow b) a b)

```
One way to detect sharing in list structures is to use the predicate eq? as a way to test whether two symbols are equal. More generally, (eq? x y) tests whether x and y are the same object (that is, whether x and y are equal as pointers). Thus, with z1 and z2 as defined in Figure 3.16 and Figure 3.17, (eq? (car z1) (cdr z1)) is true and (eq? (car z2) (cdr z2)) is false.  

As will be seen in the following sections, we can exploit sharing to greatly extend the repertoire of data structures that can be represented by pairs. On the other hand, sharing can also be dangerous, since modifications made to structures will also affect other structures that happen to share the modified parts. The mutation operations set-car! and set-cdr! should be used with care; unless we have a good understanding of how our data objects are shared, mutation can have unanticipated results.

## Representing Queues (Incomplete)
## Representing Tables
Here we see how to build tables as mutable list structures.

We first consider a one-dimensional table, in which each value is stored under a single key. We implement the table as a list of records, each of which is implemented as a pair consisting of a key and the associated value. The records are glued together to form a list by pairs whose cars point to successive records. These gluing pairs are called the backbone of the table. In order to have a place that we can change when we add a new record to the table, we build the table as a headed list. A headed list has a special backbone pair at the beginning, which holds a dummy "record" -in this case the arbitrarily chosen symbol \*table\*. Figure 3.22 shows the box-and-pointer diagram for the table
```
a: 1
b: 2
c: 3
```

![3_22](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_22.png)  

To extract information from a table we use the lookup procedure, which takes a key as argument and returns the associated value (or false if there is no value stored under that key). Lookup is defined in terms of the assoc operation, which expects a key and a list of records as arguments. Note that assoc never sees the dummy record. Assoc returns the record that has the given key as its car. Lookup then checks to see that the resulting record returned by assoc is not false, and returns the value (the cdr) of the record.
```Scheme
(define (lookup key table)
  (let ((record (assoc key (cdr table))))
    (if record
        (cdr record)
        false)))
(define (assoc key records)
  (cond ((null? records) false)
        ((equal? key (caar records)) (car records))
        (else (assoc key (cdr records)))))
```

To insert a value in a table under a specified key, we first use assoc to see if there is already a record in the table with this key. If not, we form a new record by cons ing the key with the value, and insert this at the head of the table's list of records, after the dummy record. If there already is a record with this key, we set the cdr of this record to the designated new value. The header of the table provides us with a fixed location to modify in order to insert the new record.
```Scheme
(define (insert! key value table)
  (let ((record (assoc key (cdr table))))
    (if record
        (set-cdr! record value)
        (set-cdr! table
                  (cons (cons key value)
                        (cdr table)))))
  'ok)
```
To construct a new table, we simply create a list containing the symbol \*table\*:
```Scheme
(define (make-table)
  (list '*table*))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Two-dimensional tables</h3>
In a two-dimensional table, each value is indexed by two keys. We can construct such a table as a one-dimensional table in which each key identifies a subtable. Figure 3.23 shows the box-and-pointer diagram for the table
```
math:
+: 43
-: 45
*: 42

letters:
a: 97
b: 98
```
which has two subtables. (The subtables don't need a special header symbol, since the key that identifies the subtable serves this purpose.)  

![3_23](https://github.com/opwid/Library/blob/master/Structure-and-Interpretation-of-Computer-Programs/Images/3_23.png)  

When we look up an item, we use the first key to identify the correct subtable. Then we use the second key to identify the record within the subtable.
```Scheme
(define (lookup key-1 key-2 table)
  (let ((subtable
         (assoc key-1 (cdr table))))
    (if subtable
        (let ((record
               (assoc key-2 (cdr subtable))))
          (if record
              (cdr record)
              false))
        false)))
```
To insert a new item under a pair of keys, we use assoc to see if there is a subtable stored under the first key. If not, we build a new subtable containing the single record (key-2, value) and insert it into the table under the first key. If a subtable already exists for the first key, we insert the new record into this subtable, using the insertion method for one-dimensional tables described above:
```Scheme
(define (insert! key-1 key-2 value table)
  (let ((subtable (assoc key-1 (cdr table))))
    (if subtable
        (let ((record (assoc key-2 (cdr subtable))))
          (if record
              (set-cdr! record value)
              (set-cdr! subtable
                        (cons (cons key-2 value)
                              (cdr subtable)))))
        (set-cdr! table
                  (cons (list key-1
                              (cons key-2 value))
                        (cdr table)))))
  'ok)
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Creating local tables</h3>
The lookup and insert! operations defined above take the table as an argument. This enables us to use programs that access more than one table. Another way to deal with multiple tables is to have separate lookup and insert! procedures for each table. We can do this by representing a table procedurally, as an object that maintains an internal table as part of its local state. When sent an appropriate message, this "table object" supplies the procedure with which to operate on the internal table. Here is a generator for two-dimensional tables represented in this fashion:
```Scheme
(define (make-table)
  (let ((local-table (list '*table*)))
    (define (lookup key-1 key-2)
      (let ((subtable
             (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record
                   (assoc key-2 (cdr subtable))))
              (if record (cdr record) false))
            false)))
    (define (insert! key-1 key-2 value)
      (let ((subtable
             (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record
                   (assoc key-2 (cdr subtable))))
              (if record
                  (set-cdr! record value)
                  (set-cdr! subtable
                            (cons (cons key-2 value)
                                  (cdr subtable)))))
            (set-cdr! local-table
                      (cons (list key-1 (cons key-2 value))
                            (cdr local-table)))))
      'ok)
    (define (dispatch m)
      (cond ((eq? m 'lookup-proc) lookup)
            ((eq? m 'insert-proc!) insert!)
            (else (error "Unknown operation: TABLE" m))))
    dispatch))
```
Using make-table, we could implement the get and put operations used for data-directed programming, as follows:
```Scheme
(define operation-table (make-table))
(define get (operation-table 'lookup-proc))
(define put (operation-table 'insert-proc!))
```
Get takes as arguments two keys, and put takes as arguments two keys and a value. Both operations access the same local table, which is encapsulated within the object created by the call to make-table.


## A Simulator for Digital Circuits (Incomplete)
## Propagation of Constraints (Incomplete)
# Concurrency: Time Is of the Essence (Incomplete)
## The Nature of Time in Concurrent Systems (Incomplete)
## Mechanisms for Controlling Concurrency (Incomplete)
# Streams

In this section, we will see how to model change in terms of sequences that represent the time histories of the systems being modeled. To accomplish this, we introduce new data structures called streams. From an abstract point of view, a stream is simply a sequence. However, we will find that the straightforward implementation of streams as lists doesn't fully reveal the power of stream processing. As an alternative, we introduce the technique of delayed evaluation, which enables us to represent very large (even infinite) sequences as streams.  

## Streams Are Delayed Lists
As we saw in Sequences as Conventional Interfaces, sequences can serve as standard interfaces for combining program modules. We formulated powerful abstractions for manipulating sequences, such as map, filter, and accumulate, that capture a wide variety of operations in a manner that is both succinct and elegant.  

Unfortunately, if we represent sequences as lists, this elegance is bought at the price of severe inefficiency with respect to both the time and space required by our computations. When we represent manipulations on sequences as transformations of lists, our programs must construct and copy data structures (which may be huge) at every step of a process.  

To see why this is true, let us compare two programs for computing the sum of all the prime numbers in an interval. The first program is written in standard iterative style:
```Scheme
(define (sum-primes a b)
  (define (iter count accum)
    (cond ((> count b) accum)
          ((prime? count)
           (iter (+ count 1) (+ count accum)))
          (else (iter (+ count 1) accum))))
  (iter a 0))
```

The second program performs the same computation using the sequence operations of Sequences as Conventional Interfaces:
```Scheme
(define (sum-primes a b)
  (accumulate +
              0
              (filter prime?
                      (enumerate-interval a b))))
```
In carrying out the computation, the first program needs to store only the sum being accumulated. In contrast, the filter in the second program cannot do any testing until enumerate-interval has constructed a complete list of the numbers in the interval. The filter generates another list, which in turn is passed to accumulate before being collapsed to form a sum. Such large intermediate storage is not needed by the first program, which we can think of as enumerating the interval incrementally, adding each prime to the sum as it is generated.  

Streams are a clever idea that allows one to use sequence manipulations without incurring the costs of manipulating sequences as lists. With streams we can achieve the best of both worlds: We can formulate programs elegantly as sequence manipulations, while attaining the efficiency of incremental computation. The basic idea is to arrange to construct a stream only partially, and to pass the partial construction to the program that consumes the stream. If the consumer attempts to access a part of the stream that has not yet been constructed, the stream will automatically construct just enough more of itself to produce the required part, thus preserving the illusion that the entire stream exists. In other words, although we will write programs as if we were processing complete sequences, we design our stream implementation to automatically and transparently interleave the construction of the stream with its use.  

On the surface, streams are just lists with different names for the procedures that manipulate them. There is a constructor, cons-stream, and two selectors, stream-car and stream-cdr. There is a distinguishable object, the-empty-stream, which cannot be the result of any cons-stream operation, and which can be identified with the predicate stream-null?. Thus we can make and use streams, in just the same way as we can make and use lists, to represent aggregate data arranged in a sequence. In particular, we can build stream analogs of the list operations, such as list-ref, map, and for-each:
```Scheme
(define (stream-ref s n)
  (if (= n 0)
      (stream-car s)
      (stream-ref (stream-cdr s) (- n 1))))
(define (stream-map proc s)
  (if (stream-null? s)
      the-empty-stream
      (cons-stream (proc (stream-car s))
                   (stream-map proc (stream-cdr s)))))
(define (stream-for-each proc s)
  (if (stream-null? s)
      'done
      (begin (proc (stream-car s))
             (stream-for-each proc (stream-cdr s)))))
```
Stream-for-each is useful for viewing streams:
```Scheme
(define (display-stream s)
  (stream-for-each display-line s))
(define (display-line x) (newline) (display x))
```
To make the stream implementation automatically and transparently interleave the construction of a stream with its use, we will arrange for the cdr of a stream to be evaluated when it is accessed by the stream-cdr procedure rather than when the stream is constructed by cons-stream. As a data abstraction, streams are the same as lists. The difference is the time at which the elements are evaluated. With ordinary lists, both the car and the cdr are evaluated at construction time. With streams, the cdr is evaluated at selection time.  

Our implementation of streams will be based on a special form called _delay_. Evaluating (delay \<exp>) does not evaluate the expression \<exp>, but rather returns a so-called delayed object, which we can think of as a "promise" to evaluate \<exp> at some future time. As a companion to delay, there is a procedure called force that takes a delayed object as argument and performs the evaluation -in effect, forcing the delay to fulfill its promise. We will see below how delay and force can be implemented, but first let us use these to construct streams.  

Cons-stream is a special form defined so that
```Scheme
(cons-stream <a> <b>)
```
is equivalent to 
```Scheme
(cons <a> (delay <b>))
```
What this means is that we will construct streams using pairs. However, rather than placing the value of the rest of the stream into the cdr of the pair we will put there a promise to compute the rest if it is ever requested. Stream-car and stream-cdr can now be defined as procedures: 
```Scheme
(define (stream-car stream) (car stream))
(define (stream-cdr stream) (force (cdr stream)))
```
Stream-car selects the car of the pair; stream-cdr selects the cdr of the pair and evaluates the delayed expression found there to obtain the rest of the stream.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;The stream implementation in action</h3>

To see how this implementation behaves, let us analyze the "outrageous" prime computation we saw above, reformulated in terms of streams:
```Scheme
(stream-car
 (stream-cdr
  (stream-filter prime?
                 (stream-enumerate-interval
                  10000 1000000))))
```
We begin by calling stream-enumerate-interval with the arguments 10,000 and 1,000,000. Stream-enumerate-interval is the stream analog of enumerate-interval:
```Scheme
(define (stream-enumerate-interval low high)
  (if (> low high)
      the-empty-stream
      (cons-stream
       low
       (stream-enumerate-interval (+ low 1) high))))
```
and thus the result returned by stream-enumerate-interval, formed by the cons-stream, is:
```Scheme
(cons 10000
  (delay (stream-enumerate-interval 10001 1000000)))
```
That is, stream-enumerate-interval returns a stream represented as a pair whose car is 10,000 and whose cdr is a promise to enumerate more of the interval if so requested. This stream is now filtered for primes, using the stream analog of the filter procedure:
```Scheme
(define (stream-filter pred stream)
  (cond ((stream-null? stream) the-empty-stream)
        ((pred (stream-car stream))
         (cons-stream (stream-car stream)
                      (stream-filter
                       pred
                       (stream-cdr stream))))
        (else (stream-filter pred (stream-cdr stream)))))
```
Stream-filter tests the stream-car of the stream (the car of the pair, which is 10,000). Since this is not prime, stream-filter examines the stream-cdr of its input stream. The call to stream-cdr forces evaluation of the delayed stream-enumerate-interval, which now returns
```Scheme
(cons 10001
  (delay (stream-enumerate-interval 10002 1000000)))
```

Stream-filter now looks at the stream-car of this stream, 10,001, sees that this is not prime either, forces another stream-cdr, and so on, until stream-enumerate-interval yields the prime 10,007, whereupon stream-filter, according to its definition, returns
```Scheme
(cons-stream (stream-car stream)
  (stream-filter pred (stream-cdr stream)))
```
which in this case is
```Scheme
(cons 10007
      (delay (stream-filter
              prime?
              (cons 10008
                    (delay (stream-enumerate-interval
                            10009
                            1000000))))))
```
This result is now passed to stream-cdr in our original expression. This forces the delayed stream-filter, which in turn keeps forcing the delayed stream-enumerate-interval until it finds the next prime, which is 10,009. Finally, the result passed to stream-car in our original expression is
```Scheme
(cons 10009
      (delay (stream-filter
              prime?
              (cons 10010
                    (delay (stream-enumerate-interval
                            10011
                            1000000))))))
```
Stream-car returns 10,009, and the computation is complete. Only as many integers were tested for primality as were necessary to find the second prime, and the interval was enumerated only as far as was necessary to feed the prime filter.  

In general, we can think of delayed evaluation as "demand-driven" programming, whereby each stage in the stream process is activated only enough to satisfy the next stage. What we have done is to decouple the actual order of events in the computation from the apparent structure of our procedures. We write procedures as if the streams existed "all at once" when, in reality, the computation is performed incrementally, as in traditional programming styles.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Implementing delay and force</h3>
Although delay and force may seem like mysterious operations, their implementation is really quite straightforward. Delay must package an expression so that it can be evaluated later on demand, and we can accomplish this simply by treating the expression as the body of a procedure. Delay can be a special form such that
```Scheme
(delay <exp>)
```
is syntactic sugar for
```Scheme
(lambda () <exp>)
```
Force simply calls the procedure (of no arguments) produced by delay, so we can implement force as a procedure:
```Scheme
(define (force delayed-object) (delayed-object))
```
This implementation suffices for delay and force to work as advertised, but there is an important optimization that we can include. In many applications, we end up forcing the same delayed object many times. This can lead to serious inefficiency in recursive programs involving streams. The solution is to build delayed objects so that the first time they are forced, they store the value that is computed. Subsequent forcings will simply return the stored value without repeating the computation. In other words, we implement delay as a special-purpose memoized procedure. One way to accomplish this is to use the following procedure, which takes as argument a procedure (of no arguments) and returns a memoized version of the procedure. The first time the memoized procedure is run, it saves the computed result. On subsequent evaluations, it simply returns the result.
```Scheme
(define (memo-proc proc)
  (let ((already-run? false) (result false))
    (lambda ()
      (if (not already-run?)
          (begin (set! result (proc))
                 (set! already-run? true)
                 result)
          result))))
```
Delay is then defined so that (delay \<exp>) is equivalent to
```Scheme
(memo-proc (lambda () <exp>))
```
and force is as defined previously.

## Infinite Streams
We have seen how to support the illusion of manipulating streams as complete entities even though, in actuality, we compute only as much of the stream as we need to access. We can exploit this technique to represent sequences efficiently as streams, even if the sequences are very long. What is more striking, we can use streams to represent sequences that are infinitely long. For instance, consider the following definition of the stream of positive integers:
```Scheme
(define (integers-starting-from n)
  (cons-stream n (integers-starting-from (+ n 1))))
(define integers (integers-starting-from 1))
```
This makes sense because integers will be a pair whose car is 1 and whose cdr is a promise to produce the integers beginning with 2. This is an infinitely long stream, but in any given time we can examine only a finite portion of it. Thus, our programs will never know that the entire infinite stream is not there.  

Using integers we can define other infinite streams, such as the stream of integers that are not divisible by 7:
```Scheme
(define (divisible? x y) (= (remainder x y) 0))
(define no-sevens
  (stream-filter (lambda (x) (not (divisible? x 7)))
                 integers))
```
Then we can find integers not divisible by 7 simply by accessing elements of this stream:
```Scheme
(stream-ref no-sevens 100)
>>> 117
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Defining streams implicitly</h3>

The integers stream above was defined by specifying "generating" procedures that explicitly compute the stream elements one by one. An alternative way to specify streams is to take advantage of delayed evaluation to define streams implicitly. For example, the following expression defines the stream ones to be an infinite stream of ones:
```Scheme
(define ones (cons-stream 1 ones))
```
This works much like the definition of a recursive procedure: ones is a pair whose car is 1 and whose cdr is a promise to evaluate ones. Evaluating the cdr gives us again a 1 and a promise to evaluate ones, and so on.  

We can do more interesting things by manipulating streams with operations such as add-streams, which produces the elementwise sum of two given streams:
```Scheme
(define (add-streams s1 s2) (stream-map + s1 s2))
```
Now we can define the integers as follows:
```Scheme
(define integers
  (cons-stream 1 (add-streams ones integers)))
```
This defines integers to be a stream whose first element is 1 and the rest of which is the sum of ones and integers. Thus, the second element of integers is 1 plus the first element of integers, or 2; the third element of integers is 1 plus the second element of integers, or 3; and so on. This definition works because, at any point, enough of the integers stream has been generated so that we can feed it back into the definition to produce the next integer.  

We can define the Fibonacci numbers in the same style:
```Scheme
(define fibs
  (cons-stream
   0
   (cons-stream 1 (add-streams (stream-cdr fibs) fibs))))
```
This definition says that fibs is a stream beginning with 0 and 1, such that the rest of the stream can be generated by adding fibs to itself shifted by one place:  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1 1 2 3 5 8 13 21 ... = (stream-cdr fibs)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0 1 1 2 3 5 8  13 ... = fibs  
0 1 1 2 3


























## Exploiting the Stream Paradigm
## Streams and Delayed Evaluation
## Modularity of Functional Programs and Modularity of Objects
