Metalinguistic abstraction -establishing new languages- plays an important role in all branches of engineering design. It is particularly important to computer programming, because in programming not only can we formulate new languages but we can also implement these languages by constructing evaluators. An evaluator (or interpreter ) for a programming language is a procedure that, when applied to an expression of the language, performs the actions required to evaluate that expression. 

> The evaluator, which determines the meaning of expressions in a programming language, is just another program.


# The Metacircular Evaluator
Our evaluator for Lisp will be implemented as a Lisp program. It may seem circular to think about evaluating Lisp programs using an evaluator that is itself implemented in Lisp. However, evaluation is a process, so it is appropriate to describe the evaluation process using Lisp, which, after all, is our tool for describing processes. An evaluator that is written in the same language that it evaluates is said to be _metacircular_.  

The metacircular evaluator is essentially a Scheme formulation of the environment model of evaluation described in Section 3.2. Recall that the model has two basic parts:

* To evaluate a combination (a compound expression other than a special form), evaluate the subexpressions and then apply the value of the operator subexpression to the values of the operand subexpressions.
* To apply a compound procedure to a set of arguments, evaluate the body of the procedure in a new environment. To construct this environment, extend the environment part of the procedure object by a frame in which the formal parameters of the procedure are bound to the arguments to which the procedure is applied.

These two rules describe the essence of the evaluation process, a basic cycle in which expressions to be evaluated in environments are reduced to procedures to be applied to arguments, which in turn are reduced to new expressions to be evaluated in new environments, and so on, until we get down to symbols, whose values are looked up in the environment, and to primitive procedures, which are applied directly. This evaluation cycle will be embodied by the interplay between the two critical procedures in the evaluator, _eval_ and _apply_.  

The implementation of the evaluator will depend upon procedures that define the syntax of the expressions to be evaluated. We will use data abstraction to make the evaluator independent of the representation of the language. For example, rather than committing to a choice that an assignment is to be represented by a list beginning with the symbol set! we use an abstract predicate assignment? to test for an assignment, and we use abstract selectors assignment-variable and assignment-value to access the parts of an assignment.

## The Core of the Evaluator
The evaluation process can be described as the interplay between two procedures: eval and apply.
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Eval</h3>
Eval takes as arguments an expression and an environment. It classifies the expression and directs its evaluation. Eval is structured as a case analysis of the syntactic type of the expression to be evaluated. In order to keep the procedure general, we express the determination of the type of an expression abstractly, making no commitment to any particular representation for the various types of expressions. Each type of expression has a predicate that tests for it and an abstract means for selecting its parts. This abstract syntax makes it easy to see how we can change the syntax of the language by using the same evaluator, but with a different collection of syntax procedures.  

__Primitive expressions__  

* For self-evaluating expressions, such as numbers, eval returns the expression itself.
* Eval must look up variables in the environment to find their values.

__Special forms__  

* For quoted expressions, eval returns the expression that was quoted.
* An assignment to (or a definition of) a variable must recursively call eval to compute the new value to be associated with the variable. The environment must be modified to change (or create) the binding of the variable.
* An if expression requires special processing of its parts, so as to evaluate the consequent if the predicate is true, and otherwise to evaluate the alternative.
* A lambda expression must be transformed into an applicable procedure by packaging together the parameters and body specified by the lambda expression with the environment of the evaluation.
* A begin expression requires evaluating its sequence of expressions in the order in which they appear.
* A case analysis (cond) is transformed into a nest of if expressions and then evaluated.

__Combinations__  

* For a procedure application, eval must recursively evaluate the operator part and the operands of the combination. The resulting procedure and arguments are passed to apply, which handles the actual procedure application.

Here is the definition of eval:
```Scheme
(define (eval exp env)
  (cond ((self-evaluating? exp) exp)
        ((variable? exp) (lookup-variable-value exp env))
        ((quoted? exp) (text-of-quotation exp))
        ((assignment? exp) (eval-assignment exp env))
        ((definition? exp) (eval-definition exp env))
        ((if? exp) (eval-if exp env))
        ((lambda? exp) (make-procedure (lambda-parameters exp)
                                       (lambda-body exp)
                                       env))
        ((begin? exp)
         (eval-sequence (begin-actions exp) env))
        ((cond? exp) (eval (cond->if exp) env))
        ((application? exp)
         (apply (eval (operator exp) env)
                (list-of-values (operands exp) env)))
        (else
         (error "Unknown expression type: EVAL" exp))))
```
For clarity, eval has been implemented as a case analysis using cond. The disadvantage of this is that our procedure handles only a few distinguishable types of expressions, and no new ones can be defined without editing the definition of eval. In most Lisp implementations, dispatching on the type of an expression is done in a data-directed style. This allows a user to add new types of expressions that eval can distinguish, without modifying the definition of eval itself.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Apply</h3>
Apply takes two arguments, a procedure and a list of arguments to which the procedure should be applied. Apply classifies procedures into two kinds: It calls apply-primitive-procedure to apply primitives; it applies compound procedures by sequentially evaluating the expressions that make up the body of the procedure. The environment for the evaluation of the body of a compound procedure is constructed by extending the base environment carried by the procedure to include a frame that binds the parameters of the procedure to the arguments to which the procedure is to be applied. Here is the definition of apply:
```Scheme
(define (apply procedure arguments)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure procedure arguments))
        ((compound-procedure? procedure)
         (eval-sequence
          (procedure-body procedure)
          (extend-environment
           (procedure-parameters procedure)
           arguments
           (procedure-environment procedure))))
        (else
         (error
          "Unknown procedure type: APPLY" procedure)))) 
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Procedure arguments</h3>
When eval processes a procedure application, it uses list-of-values to produce the list of arguments to which the procedure is to be applied. List-of-values takes as an argument the operands of the combination. It evaluates each operand and returns a list of the corresponding values:
```Scheme
(define (list-of-values exps env)
  (if (no-operands? exps)
      '()
      (cons (eval (first-operand exps) env)
            (list-of-values (rest-operands exps) env))))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Conditionals</h3>
Eval-if evaluates the predicate part of an if expression in the given environment. If the result is true, eval-if evaluates the consequent, otherwise it evaluates the alternative:
```Scheme
(define (eval-if exp env)
  (if (true? (eval (if-predicate exp) env))
      (eval (if-consequent exp) env)
      (eval (if-alternative exp) env)))
```
The use of true? in eval-if highlights the issue of the connection between an implemented language and an implementation language. The if-predicate is evaluated in the language being implemented and thus yields a value in that language. The interpreter predicate true? translates that value into a value that can be tested by the if in the implementation language: The metacircular representation of truth might not be the same as that of the underlying Scheme.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Sequences</h3>

Eval-sequence is used by apply to evaluate the sequence of expressions in a procedure body and by eval to evaluate the sequence of expressions in a begin expression. It takes as arguments a sequence of expressions and an environment, and evaluates the expressions in the order in which they occur. The value returned is the value of the final expression.
```Scheme
(define (eval-sequence exps env)
  (cond ((last-exp? exps)
         (eval (first-exp exps) env))
        (else
         (eval (first-exp exps) env)
         (eval-sequence (rest-exps exps) env))))
```

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Assignments and definitions</h3>
The following procedure handles assignments to variables. It calls eval to find the value to be assigned and transmits the variable and the resulting value to set-variable-value! to be installed in the designated environment.
```Scheme
(define (eval-assignment exp env)
  (set-variable-value! (assignment-variable exp)
                       (eval (assignment-value exp) env)
                       env)
  'ok)
```
Definitions of variables are handled in a similar manner.
```Scheme
(define (eval-definition exp env)
  (define-variable! (definition-variable exp)
    (eval (definition-value exp) env)
    env)
  'ok)
```

## Representing Expressions

The evaluator program operate on symbolic expressions. The result of operating on a compound expression is determined by operating recursively on the pieces of the expression and combining the results in a way that depends on the type of the expression. We used data abstraction to decouple the general rules of operation from the details of how expressions are represented. For the evaluator, this means that the syntax of the language being evaluated is determined solely by the procedures that classify and extract pieces of expressions.  
Here is the specification of the syntax of our language:

* The only self-evaluating items are numbers and strings:
```Scheme
(define (self-evaluating? exp)
  (cond ((number? exp) true)
        ((string? exp) true)
        (else false)))
```

* Variables are represented by symbols:
```Scheme
(define (variable? exp) (symbol? exp))
```

* Qotations have the form (quote \<text-of-quotation>):
```Scheme
(define (quoted? exp) (tagged-list? exp 'quote))
(define (text-of-quotation exp) (cadr exp))
```
Quoted? is defined in terms of the procedure tagged-list?, which identifies lists beginning with a designated symbol:
```Scheme
(define (tagged-list? exp tag)
  (if (pair? exp)
      (eq? (car exp) tag)
      false))
```

* Assignments have the form (set! \<var> \<value>):
```Scheme
(define (assignment? exp) (tagged-list? exp 'set!))
(define (assignment-variable exp) (cadr exp))
(define (assignment-value exp) (caddr exp))
```

* Definitions have the form
```Scheme
(define <var> <value> )
```
  or the form
```Scheme
(define (<var> <parameter1> . . . <parameterN> )
<body> )

```
The corresponding syntax procedures are the following:
```Scheme
(define (definition? exp) (tagged-list? exp 'define))
(define (definition-variable exp)
  (if (symbol? (cadr exp))
      (cadr exp)
      (caadr exp)))
(define (definition-value exp)
  (if (symbol? (cadr exp))
      (caddr exp)
      (make-lambda (cdadr exp); formal parameters
                   (cddr exp)))); body
```

* Lambda expressions are lists that begin with the symbol lambda:
```Scheme
(define (lambda? exp) (tagged-list? exp 'lambda))
(define (lambda-parameters exp) (cadr exp))
(define (lambda-body exp) (cddr exp))
```
We also provide a constructor for lambda expressions, which is used by definition-value, above:
```Scheme
(define (make-lambda parameters body)
  (cons 'lambda (cons parameters body)))
```

* Conditionals begin with if and have a predicate, a consequent, and an (optional) alternative. If the expression has no alternative part, we provide false as the alternative.
```Scheme
(define (if? exp) (tagged-list? exp 'if))
(define (if-predicate exp) (cadr exp))
(define (if-consequent exp) (caddr exp))
(define (if-alternative exp)
  (if (not (null? (cdddr exp)))
  (cadddr exp)
  'false))
```
We also provide a constructor for if expressions, to be used by cond->if to transform cond expressions into if expressions:
```Scheme
(define (make-if predicate consequent alternative)
  (list 'if predicate consequent alternative))
```

* Begin packages a sequence of expressions into a single expression. We include syntax operations on begin expressions to extract the actual sequence from the begin expression, as well as selectors that return the first expression and the rest of the expressions in the sequence.
```Scheme
(define (begin? exp) (tagged-list? exp 'begin))
(define (begin-actions exp) (cdr exp))
(define (last-exp? seq) (null? (cdr seq)))
(define (first-exp seq) (car seq))
(define (rest-exps seq) (cdr seq))
```
We also include a constructor sequence->exp (for use by cond->if) that transforms a sequence into a single expression, using begin if necessary: 
```Scheme
(define (sequence->exp seq)
  (cond ((null? seq) seq)
        ((last-exp? seq) (first-exp seq))
        (else (make-begin seq))))
(define (make-begin seq) (cons 'begin seq))
```

* A procedure application is any compound expression that is not one of the above expression types. The car of the expression is the operator, and the cdr is the list of operands:
```Scheme
(define (application? exp) (pair? exp))
(define (operator exp) (car exp))
(define (operands exp) (cdr exp))
(define (no-operands? ops) (null? ops))
(define (first-operand ops) (car ops))
(define (rest-operands ops) (cdr ops))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Derived expressions</h3>

Some special forms in our language can be defined in terms of expressions involving other special forms, rather than being implemented directly. One example is cond, which can be implemented as a nest of if expressions. Implementing the evaluation of cond in this way simplifies the evaluator because it reduces the number of special forms for which the evaluation process must be explicitly specified.  

We include syntax procedures that extract the parts of a cond expression, and a procedure cond->if that transforms cond expressions into if expressions. A case analysis begins with cond and has a list of predicate-action clauses. A clause is an else clause if its predicate is the symbol else.

```Scheme
(define (cond? exp) (tagged-list? exp 'cond))
(define (cond-clauses exp) (cdr exp))
(define (cond-else-clause? clause)
  (eq? (cond-predicate clause) 'else))
(define (cond-predicate clause) (car clause))
(define (cond-actions clause) (cdr clause))
(define (cond->if exp) (expand-clauses (cond-clauses exp)))
(define (expand-clauses clauses)
  (if (null? clauses)
      'false; no else clause
      (let ((first (car clauses))
            (rest (cdr clauses)))
        (if (cond-else-clause? first)
            (if (null? rest)
                (sequence->exp (cond-actions first))
                (error "ELSE clause isn't last: COND->IF"
                       clauses))
            (make-if (cond-predicate first)
                     (sequence->exp (cond-actions first))
                     (expand-clauses rest))))))
```
Expressions (such as cond) that we choose to implement as syntactic transformations are called _derived expressions_. Let expressions are also derived expressions.

## Evaluator Data Structures

In addition to defining the external syntax of expressions, the evaluator implementation must also define the data structures that the evaluator manipulates internally, as part of the execution of a program, such as the representation of procedures and environments and the representation of true and false.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Testing of predicates</h3>

For conditionals, we accept anything to be true that is not the explicit false object.
```Scheme
(define (true? x) (not (eq? x false)))
(define (false? x) (eq? x false))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Representing procedures</h3>

To handle primitives, we assume that we have available the following procedures:

* (apply-primitive-procedure \<proc> \<args>) applies the given primitive procedure to the argument values in the list \<args> and returns the result of the application.
* (primitive-procedure? \<proc>) tests whether \<proc> is a primitive procedure.

These mechanisms for handling primitives are further described in Running the Evaluator as a Program.

Compound procedures are constructed from parameters, procedure bodies, and environments using the constructor make-procedure:
```Scheme
(define (make-procedure parameters body env)
  (list 'procedure parameters body env))
(define (compound-procedure? p)
  (tagged-list? p 'procedure))
(define (procedure-parameters p) (cadr p))
(define (procedure-body p) (caddr p))
(define (procedure-environment p) (cadddr p))
```

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Operations on Environments</h3>
The evaluator needs operations for manipulating environments. We use the following operations for manipulating environments:

* (lookup-variable-value \<var> \<env>) returns the value that is bound to the symbol \<var> in the environment \<env>, or signals an error if the variable is unbound.
* (extend-environment \<variables> \<values> \<base-env>) returns a new environment, consisting of a new frame in which the symbols in the list \<variables> are bound to the corresponding elements in the list \<values>, where the enclosing environment is the environment \<base-env>.
* (define-variable! \<var> \<value> \<env>) adds to the first frame in the environment \<env> a new binding that associates the variable \<var> with the value \<value>.
* (set-variable-value! \<var> \<value> \<env>) changes the binding of the variable \<var> in the environment \<env> so that the variable is now bound to the value \<value>, or signals an error if the variable is unbound.
To implement these operations we represent an environment as a list of frames. The enclosing environment of an environment is the cdr of the list. The empty environment is simply the empty list. 
```Scheme
(define (enclosing-environment env) (cdr env))
(define (first-frame env) (car env))
(define the-empty-environment '())
```
Each frame of an environment is represented as a pair of lists: a list of the variables bound in that frame and a list of the associated values. 
```Scheme
(define (make-frame variables values)
  (cons variables values))
(define (frame-variables frame) (car frame))
(define (frame-values frame) (cdr frame))
(define (add-binding-to-frame! var val frame)
  (set-car! frame (cons var (car frame)))
  (set-cdr! frame (cons val (cdr frame))))
```

To extend an environment by a new frame that associates variables with values, we make a frame consisting of the list of variables and the list of values, and we adjoin this to the environment. We signal an error if the number of variables does not match the number of values.
```Scheme
(define (extend-environment vars vals base-env)
  (if (= (length vars) (length vals))
      (cons (make-frame vars vals) base-env)
      (if (< (length vars) (length vals))
          (error "Too many arguments supplied" vars vals)
          (error "Too few arguments supplied" vars vals))))
```

To look up a variable in an environment, we scan the list of variables in the first frame. If we find the desired variable, we return the corresponding element in the list of values. If we do not find the variable in the current frame, we search the enclosing environment, and so on. If we reach the empty environment, we signal an "unbound variable" error.
```Scheme
(define (lookup-variable-value var env)
  (define (env-loop env)
  (define (scan vars vals)
    (cond ((null? vars)
           (env-loop (enclosing-environment env)))
          ((eq? var (car vars)) (car vals))
          (else (scan (cdr vars) (cdr vals)))))
    (if (eq? env the-empty-environment)
        (error "Unbound variable" var)
        (let ((frame (first-frame env)))
          (scan (frame-variables frame)
                (frame-values frame)))))
  (env-loop env))
```

To set a variable to a new value in a specified environment, we scan for the variable, just as in lookup-variable-value, and change the corresponding value when we find it.
```Scheme
(define (set-variable-value! var val env)
  (define (env-loop env)
    (define (scan vars vals)
      (cond ((null? vars)
             (env-loop (enclosing-environment env)))
            ((eq? var (car vars)) (set-car! vals val))
            (else (scan (cdr vars) (cdr vals)))))
    (if (eq? env the-empty-environment)
        (error "Unbound variable: SET!" var)
        (let ((frame (first-frame env)))
          (scan (frame-variables frame)
                (frame-values frame)))))
  (env-loop env))
```
To define a variable, we search the first frame for a binding for the variable, and change the binding if it exists (just as in set-variable-value!). If no such binding exists, we adjoin one to the first frame.
```Scheme
(define (define-variable! var val env)
  (let ((frame (first-frame env)))
    (define (scan vars vals)
      (cond ((null? vars)
             (add-binding-to-frame! var val frame))
            ((eq? var (car vars)) (set-car! vals val))
            (else (scan (cdr vars) (cdr vals)))))
    (scan (frame-variables frame) (frame-values frame))))
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Running the Evaluator as a Program</h3>
Given the evaluator, we have in our hands a description (expressed in Lisp) of the process by which Lisp expressions are evaluated. One advantage of expressing the evaluator as a program is that we can run the program. This gives us, running within Lisp, a working model of how Lisp itself evaluates expressions. This can serve as a framework for experimenting with evaluation rules, as we shall do later in this chapter.  

Our evaluator program reduces expressions ultimately to the application of primitive procedures. Therefore, all that we need to run the evaluator is to create a mechanism that calls on the underlying Lisp system to model the application of primitive procedures.  

There must be a binding for each primitive procedure name, so that when eval evaluates the operator of an application of a primitive, it will find an object to pass to apply. We thus set up a global environment that associates unique objects with the names of the primitive procedures that can appear in the expressions we will be evaluating. The global environment also includes bindings for the symbols true and false, so that they can be used as variables in expressions to be evaluated.
```Scheme
(define (setup-environment)
  (let ((initial-env
         (extend-environment (primitive-procedure-names)
                             (primitive-procedure-objects)
                             the-empty-environment)))
    (define-variable! 'true true initial-env)
    (define-variable! 'false false initial-env)
    initial-env))
(define the-global-environment (setup-environment))
```
It does not matter how we represent the primitive procedure objects, so long as apply can identify and apply them by using the procedures primitive-procedure? and apply-primitive-procedure. We have chosen to represent a primitive procedure as a list beginning with the symbol primitive and containing a procedure in the underlying Lisp that implements that primitive.
```Scheme
(define (primitive-procedure? proc)
  (tagged-list? proc 'primitive))
(define (primitive-implementation proc) (cadr proc))
```
Setup-environment will get the primitive names and implementation procedures from a list:
```Scheme
(define primitive-procedures
  (list (list 'car car)
        (list 'cdr cdr)
        (list 'cons cons)
        (list 'null? null?)
        <more primitives> ))
(define (primitive-procedure-names)
  (map car primitive-procedures))
(define (primitive-procedure-objects)
  (map (lambda (proc) (list 'primitive (cadr proc)))
       primitive-procedures))
```
To apply a primitive procedure, we simply apply the implementation procedure to the arguments, using the underlying Lisp system:
```Scheme
(define (apply-primitive-procedure proc args)
  (apply-in-underlying-scheme
   (primitive-implementation proc) args))
```
For convenience in running the metacircular evaluator, we provide a driver loop that models the read-eval-print loop of the underlying Lisp system. It prints a prompt, reads an input expression, evaluates this expression in the global environment, and prints the result. We precede each printed result by an output prompt so as to distinguish the value of the expression from other output that may be printed.
```Scheme
(define input-prompt "\;\;\; M-Eval input:")
(define output-prompt ";;; M-Eval value:")
(define (driver-loop)
  (prompt-for-input input-prompt)
  (let ((input (read)))
    (let ((output (eval input the-global-environment)))
      (announce-output output-prompt)
      (user-print output)))
  (driver-loop))
(define (prompt-for-input string)
  (newline) (newline) (display string) (newline))
(define (announce-output string)
  (newline) (display string) (newline))
```
We use a special printing procedure, user-print, to avoid printing the environment part of a compound procedure, which may be a very long list (or may even contain cycles).
```Scheme
(define (user-print object)
  (if (compound-procedure? object)
      (display (list 'compound-procedure
                     (procedure-parameters object)
                     (procedure-body object)
                     '<procedure-env>))
      (display object)))
```
Now all we need to do to run the evaluator is to initialize the global environment and start the driver loop. Here is a sample interaction:
```Scheme
(define the-global-environment (setup-environment))
(driver-loop)
;;; M-Eval input:
(define (append x y)
  (if (null? x)
      y
      (cons (car x) (append (cdr x) y))))
;;; M-Eval value:
ok
;;; M-Eval input:
(append '(a b c) '(d e f))
;;; M-Eval value:
(a b c d e f)
```
## Data as Programs

We can regard the evaluator as a very special machine that takes as input a description of a machine. Given this input, the evaluator configures itself to emulate the machine described. For example, if we feed our evaluator the definition of factorial the evaluator will be able to compute factorials. From this perspective, our evaluator is seen to be a universal machine. It mimics other machines when these are described as Lisp programs. This is striking.  

Another striking aspect of the evaluator is that it acts as a bridge between the data objects that are manipulated by our programming language and the programming language itself. Imagine that the evaluator program (implemented in Lisp) is running, and that a user is typing expressions to the evaluator and observing the results. From the perspective of the user, an input expression such as (\* x x) is an expression in the programming language, which the evaluator should execute. From the perspective of the evaluator, however, the expression is simply a list (in this case, a list of three symbols: * , x , and x ) that is to be manipulated according to a well-defined set of rules.  

That the user's programs are the evaluator's data need not be a source of confusion. In fact, it is sometimes convenient to ignore this distinction, and to give the user the ability to explicitly evaluate a data object as a Lisp expression, by making eval available for use in programs. Many Lisp dialects provide a primitive eval procedure that takes as arguments an expression and an environment and evaluates the expression relative to the environment. Thus,
```Scheme
(eval '(* 5 5) user-initial-environment)
```
and
```Scheme
(eval (cons '* (list 5 5)) user-initial-environment)
```
will both return 25.

## Internal Definitions
Our environment model of evaluation and our metacircular evaluator execute definitions in sequence, extending the environment frame one definition at a time. This is particularly convenient for interactive program development, in which the programmer needs to freely mix the application of procedures with the definition of new procedures. However, if we think carefully about the internal definitions used to implement block structure, we will find that name-by-name extension of the environment may not be the best way to define local variables.  

Consider a procedure with internal definitions, such as
```Scheme
(define (f x)
  (define (even? n) (if (= n 0) true (odd? (- n 1))))
  (define (odd? n) (if (= n 0) false (even? (- n 1))))
  <rest of body of f> )
```

Our intention here is that the name odd? in the body of the procedure even? should refer to the procedure odd? that is defined after even?. The scope of the name odd? is the entire body of f, not just the portion of the body of f starting at the point where the define for odd? occurs. Indeed, when we consider that odd? is itself defined in terms of even? -so that even? and odd? are mutually recursive procedures- we see that the only satisfactory interpretation of the two define s is to regard them as if the names even? and odd? were being added to the environment simultaneously. More generally, in block structure, the scope of a local name is the entire procedure body in which the define is evaluated.  

As it happens, our interpreter will evaluate calls to f correctly, but for an "accidental" reason: Since the definitions of the internal procedures come first, no calls to these procedures will be evaluated until all of them have been defined. Hence, odd? will have been defined by the time even? is executed. In fact, our sequential evaluation mechanism will give the same result as a mechanism that directly implements simultaneous definition for any procedure in which the internal definitions come first in a body and evaluation of the value expressions for the defined variables doesn't actually use any of the defined variables.  

There is, however, a simple way to treat definitions so that internally defined names have truly simultaneous scope -just create all local variables that will be in the current environment before evaluating any of the value expressions. One way to do this is by a syntax transformation on lambda expressions. Before evaluating the body of a lambda expression, we "scan out" and eliminate all the internal definitions in the body. The internally defined variables will be created with a let and then set to their values by assignment. For example, the procedure
```Scheme
(lamda <vars>
       (define u <e1>)
       (define v <e2>)
       <e3>)
```
would be transformed into
```Scheme
(lambda <vars>
  (let ((u '*unassigned*)
        (v '*unassigned*))
    (set! u <e1>)
    (set! v <e2>)
    <e3>))
```
where \*unassigned\* is a special symbol that causes looking up a variable to signal an error if an attempt is made to use the value of the not-yet-assigned variable.

## Separating Syntactic Analysis from Execution

The evaluator implemented above is simple, but it is very inefficient, because the syntactic analysis of expressions is interleaved with their execution. Thus if a program is executed many times, its syntax is analyzed many times. Consider, for example, evaluating (factorial 4) using the following definition of factorial:
```Scheme
(define (factorial n)
  (if (= n 1) 1 (* (factorial (- n 1)) n)))
```
Each time factorial is called, the evaluator must determine that the body is an if expression and extract the predicate. Only then can it evaluate the predicate and dispatch on its value. Each time it evaluates the expression `(* (factorial (- n 1)) n)`, or the subexpressions `(factorial (- n 1)) and (- n 1)`, the evaluator must perform the case analysis in eval to determine that the expression is an application, and must extract its operator and operands. This analysis is expensive. Performing it repeatedly is wasteful.  

We can transform the evaluator to be significantly more efficient by arranging things so that syntactic analysis is performed only once. We split eval, which takes an expression and an environment, into two parts. The procedure analyze takes only the expression. It performs the syntactic analysis and returns a new procedure, the execution procedure, that encapsulates the work to be done in executing the analyzed expression. The execution procedure takes an environment as its argument and completes the evaluation. This saves work because analyze will be called only once on an expression, while the execution procedure may be called many times.  

With the separation into analysis and execution, eval now becomes
```Scheme
(define (eval exp env) ((analyze exp) env))
```
The result of calling analyze is the execution procedure to be applied to the environment. The analyze procedure is the same case analysis as performed by the original eval of The Core of the Evaluator, except that the procedures to which we dispatch perform only analysis, not full evaluation:
```Scheme
(define (analyze exp)
  (cond ((self-evaluating? exp) (analyze-self-evaluating exp))
        ((quoted? exp) (analyze-quoted exp))
        ((variable? exp) (analyze-variable exp))
        ((assignment? exp) (analyze-assignment exp))
        ((definition? exp) (analyze-definition exp))
        ((if? exp) (analyze-if exp))
        ((lambda? exp) (analyze-lambda exp))
        ((begin? exp) (analyze-sequence (begin-actions exp)))
        ((cond? exp) (analyze (cond->if exp)))
        ((application? exp) (analyze-application exp))
        (else (error "Unknown expression type: ANALYZE" exp))))
```
Here is the simplest syntactic analysis procedure, which handles self-evaluating expressions. It returns an execution procedure that ignores its environment argument and just returns the expression:
```Scheme
(define (analyze-self-evaluating exp)
  (lambda (env) exp))
```
For a quoted expression, we can gain a little efficiency by extracting the text of the quotation only once, in the analysis phase, rather than in the execution phase.
```Scheme
(define (analyze-quoted exp)
  (let ((qval (text-of-quotation exp)))
    (lambda (env) qval)))
```

Looking up a variable value must still be done in the execution phase, since this depends upon knowing the environment. 
```Scheme
(define (analyze-variable exp)
  (lambda (env) (lookup-variable-value exp env)))
```
Analyze-assignment also must defer actually setting the variable until the execution, when the environment has been supplied. However, the fact that the assignment-value expression can be analyzed (recursively) during analysis is a major gain in efficiency, because the assignment-value expression will now be analyzed only once. The same holds true for definitions.
```Scheme
(define (analyze-assignment exp)
  (let ((var (assignment-variable exp))
        (vproc (analyze (assignment-value exp))))
    (lambda (env)
      (set-variable-value! var (vproc env) env)
      'ok)))
(define (analyze-definition exp)
  (let ((var (definition-variable exp))
        (vproc (analyze (definition-value exp))))
    (lambda (env)
      (define-variable! var (vproc env) env)
      'ok)))
```
For if expressions, we extract and analyze the predicate, consequent, and alternative at analysis time.
```Scheme
(define (analyze-if exp)
  (let ((pproc (analyze (if-predicate exp)))
        (cproc (analyze (if-consequent exp)))
        (aproc (analyze (if-alternative exp))))
    (lambda (env) (if (true? (pproc env))
                      (cproc env)
                      (aproc env)))))
```
Analyzing a lambda expression also achieves a major gain in efficiency: We analyze the lambda body only once, even though procedures resulting from evaluation of the lambda may be applied many times.
```Scheme
(define (analyze-lambda exp)
  (let ((vars (lambda-parameters exp))
        (bproc (analyze-sequence (lambda-body exp))))
    (lambda (env) (make-procedure vars bproc env))))
```

Analysis of a sequence of expressions (as in a begin or the body of a lambda expression) is more involved. Each expression in the sequence is analyzed, yielding an execution procedure. These execution procedures are combined to produce an execution procedure that takes an environment as argument and sequentially calls each individual execution procedure with the environment as argument.
```Scheme
(define (analyze-sequence exps)
  (define (sequentially proc1 proc2)
    (lambda (env) (proc1 env) (proc2 env)))
  (define (loop first-proc rest-procs)
    (if (null? rest-procs)
        first-proc
        (loop (sequentially first-proc (car rest-procs))
              (cdr rest-procs))))
  (let ((procs (map analyze exps)))
    (if (null? procs) (error "Empty sequence: ANALYZE"))
    (loop (car procs) (cdr procs))))
```
To analyze an application, we analyze the operator and operands and construct an execution procedure that calls the operator execution procedure (to obtain the actual procedure to be applied) and the operand execution procedures (to obtain the actual arguments). We then pass these to execute-application, which is the analog of apply in Section 4.1.1. Execute-application differs from apply in that the procedure body for a compound procedure has already been analyzed, so there is no need to do further analysis. Instead, we just call the execution procedure for the body on the extended environment.
```Scheme
(define (analyze-application exp)
  (let ((fproc (analyze (operator exp)))
        (aprocs (map analyze (operands exp))))
    (lambda (env)
      (execute-application
       (fproc env)
       (map (lambda (aproc) (aproc env))
            aprocs)))))
(define (execute-application proc args)
  (cond ((primitive-procedure? proc)
         (apply-primitive-procedure proc args))
        ((compound-procedure? proc)
         ((procedure-body proc)
          (extend-environment
           (procedure-parameters proc)
           args
           (procedure-environment proc))))
        (else
         (error "Unknown procedure type: EXECUTE-APPLICATION"
                proc))))
```
Our new evaluator uses the same data structures, syntax procedures, and run-time support procedures as in sections Section 4.1.2, Section 4.1.3, and Section 4.1.4.

# Variations on a Scheme — Lazy Evaluation

Now that we have an evaluator expressed as a Lisp program, we can experiment with alternative choices in language design simply by modifying the evaluator. Indeed, new languages are often invented by first writing an evaluator that embeds the new language within an existing high-level language.

## Normal Order and Applicative Order

We noted that Scheme is an applicative-order language, namely, that all the arguments to Scheme procedures are evaluated when the procedure is applied. In contrast, normal-order languages delay evaluation of procedure arguments until the actual argument values are needed. Delaying evaluation of procedure arguments until the last possible moment (e.g., until they are required by a primitive operation) is called _lazy evaluation_.  

Consider the procedure
```Scheme
(define (try a b) (if (= a 0) 1 b))
```
Evaluating (try 0 (/ 1 0)) generates an error in Scheme. With lazy evaluation, there would be no error. Evaluating the expression would return 1, because the argument (/ 1 0) would never be evaluated. An advantage of lazy evaluation is that some procedures can do useful computation even if evaluation of some of their arguments would produce errors or would not terminate.  

If the body of a procedure is entered before an argument has been evaluated we say that the procedure is _non-strict_ in that argument. If the argument is evaluated before the body of the procedure is entered we say that the procedure is _strict_ in that argument. In a purely applicative-order language, all procedures are strict in each argument. In a purely normal-order language, all compound procedures are non-strict in each argument, and primitive procedures may be either strict or non-strict.  

A striking example of a procedure that can usefully be made non-strict is cons (or, in general, almost any constructor for data structures). One can do useful computation, combining elements to form data structures and operating on the resulting data structures, even if the values of the elements are not known. It makes perfect sense, for instance, to compute the length of a list without knowing the values of the individual elements in the list.

## An Interpreter with Lazy Evaluation

In this section we will implement a normal-order language that is the same as Scheme except that compound procedures are non-strict in each argument. Primitive procedures will still be strict. It is not difficult to modify the evaluator of The Core of the Evaluator so that the language it interprets behaves this way. Almost all the required changes center around procedure application.  

The basic idea is that, when applying a procedure, the interpreter must determine which arguments are to be evaluated and which are to be delayed. The delayed arguments are not evaluated; instead, they are transformed into objects called _thunks_. The thunk must contain the information required to produce the value of the argument when it is needed, as if it had been evaluated at the time of the application. Thus, the thunk must contain the argument expression and the environment in which the procedure application is being evaluated.  

The process of evaluating the expression in a thunk is called _forcing_. In general, a thunk will be forced only when its value is needed: when it is passed to a primitive procedure that will use the value of the thunk; when it is the value of a predicate of a conditional; and when it is the value of an operator that is about to be applied as a procedure. One design choice we have available is whether or not to _memoize_ thunks, as we did with delayed objects in Streams Are Delayed Lists. With memoization, the first time a thunk is forced, it stores the value that is computed. Subsequent forcings simply return the stored value without repeating the computation. We'll make our interpreter memoize, because this is more efficient for many applications. There are tricky considerations here, however.

<h3>&nbsp;&nbsp;&nbsp;&nbsp;Modifying the evaluator</h3>
The main difference between the lazy evaluator and the one in The Metacircular Evaluator is in the handling of procedure applications in eval and apply.  
The application? clause of eval becomes
```Scheme
((application? exp)
 (apply (actual-value (operator exp) env)
        (operands exp)
        env))
```
This is almost the same as the application? clause of eval in The Core of the Evaluator. For lazy evaluation, however, we call apply with the operand expressions, rather than the arguments produced by evaluating them. Since we will need the environment to construct thunks if the arguments are to be delayed, we must pass this as well. We still evaluate the operator, because apply needs the actual procedure to be applied in order to dispatch on its type (primitive versus compound) and apply it.  

Whenever we need the actual value of an expression, we use
```Scheme
(define (actual-value exp env)
  (force-it (eval exp env)))
```
instead of just eval, so that if the expression's value is a thunk, it will be forced.  

Our new version of apply is also almost the same. The difference is that eval has passed in unevaluated operand expressions: For primitive procedures (which are strict), we evaluate all the arguments before applying the primitive; for compound procedures (which are non-strict) we delay all the arguments before applying the procedure.
```Scheme
(define (apply procedure arguments env)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure
          procedure
          (list-of-arg-values arguments env))); changed
        ((compound-procedure? procedure)
         (eval-sequence
          (procedure-body procedure)
          (extend-environment
           (procedure-parameters procedure)
           (list-of-delayed-args arguments env); changed
           (procedure-environment procedure))))
        (else (error "Unknown procedure type: APPLY"
                     procedure))))
```
The procedures that process the arguments are just like list-of-values from The Core of the Evaluator, except that list-of-delayed-args delays the arguments instead of evaluating them, and list-of-arg-values uses actual value instead of eval:
```Scheme
(define (list-of-arg-values exps env)
  (if (no-operands? exps)
      '()
      (cons (actual-value (first-operand exps)
                          env)
            (list-of-arg-values (rest-operands exps)
                                env))))
(define (list-of-delayed-args exps env)
  (if (no-operands? exps)
      '()
      (cons (delay-it (first-operand exps)
                      env)
            (list-of-delayed-args (rest-operands exps)
                                  env))))
```
The other place we must change the evaluator is in the handling of if, where we must use actual-value instead of eval to get the value of the predicate expression before testing whether it is true or false:
```Scheme
(define (eval-if exp env)
  (if (true? (actual-value (if-predicate exp) env))
      (eval (if-consequent exp) env)
      (eval (if-alternative exp) env))
```
Finally, we must change the driver-loop procedure to use actual-value instead of eval, so that if a delayed value is propagated back to the read-eval-print loop, it will be forced before being printed. We also change the prompts to indicate that this is the lazy evaluator:
```Scheme
(define input-prompt ";;; L-Eval input:")
(define output-prompt ";;; L-Eval value:")
(define (driver-loop)
  (prompt-for-input input-prompt)
  (let ((input (read)))
    (let ((output
           (actual-value
            input the-global-environment)))
      (announce-output output-prompt)
      (user-print output)))
  (driver-loop)))
```
With these changes made, we can start the evaluator and test it. The successful evaluation of the try expression discussed in Normal Order and Applicative Order indicates that the interpreter is performing lazy evaluation:
```Scheme
(define the-global-environment (setup-environment))
(driver-loop)
;;; L-Eval input:
(define (try a b) (if (= a 0) 1 b))
;;; L-Eval value:
ok
;;; L-Eval input:
(try 0 (/ 1 0))
;;; L-Eval value:
1
```
<h3>&nbsp;&nbsp;&nbsp;&nbsp;Representing thunks</h3>
Our evaluator must arrange to create thunks when procedures are applied to arguments and to force these thunks later. A thunk must package an expression together with the environment, so that the argument can be produced later. To force the thunk, we simply extract the expression and environment from the thunk and evaluate the expression in the environment. We use actual-value rather than eval so that in case the value of the expression is itself a thunk, we will force that, and so on, until we reach something that is not a thunk:
```Scheme
(define (force-it obj)
  (if (thunk? obj)
      (actual-value (thunk-exp obj) (thunk-env obj))
      obj))
```
One easy way to package an expression with an environment is to make a list containing the expression and the environment. Thus, we create a thunk as follows:
```Scheme
(define (delay-it exp env)
  (list 'thunk exp env))
(define (thunk? obj)
  (tagged-list? obj 'thunk))
(define (thunk-exp thunk) (cadr thunk))
(define (thunk-env thunk) (caddr thunk))
```
Actually, what we want for our interpreter is not quite this, but rather thunks that have been memoized. When a thunk is forced, we will turn it into an evaluated thunk by replacing the stored expression with its value and changing the thunk tag so that it can be recognized as already evaluated.
```Scheme
(define (evaluated-thunk? obj)
  (tagged-list? obj 'evaluated-thunk))
(define (thunk-value evaluated-thunk)
  (cadr evaluated-thunk))
(define (force-it obj)
  (cond ((thunk? obj)
         (let ((result (actual-value (thunk-exp obj)
                                     (thunk-env obj))))
           (set-car! obj 'evaluated-thunk)
           (set-car! (cdr obj)
                     result); replace exp with its value
           (set-cdr! (cdr obj)
                     '()); forget unneeded env
           result))
        ((evaluated-thunk? obj) (thunk-value obj))
        (else obj)))
```
Notice that the same delay-it procedure works both with and without memoization.
## Streams as Lazy Lists

In Section 3.5.1, we showed how to implement streams as delayed lists. We introduced special forms delay and cons-stream, which allowed us to construct a "promise" to compute the cdr of a stream, without actually fulfilling that promise until later. We could use this general technique of introducing special forms whenever we need more control over the evaluation process, but this is awkward. For one thing, a special form is not a first-class object like a procedure, so we cannot use it together with higher-order procedures. Additionally, we were forced to create streams as a new kind of data object similar but not identical to lists, and this required us to reimplement many ordinary list operations (map, append, and so on) for use with streams.  

With lazy evaluation, streams and lists can be identical, so there is no need for special forms or for separate list and stream operations. All we need to do is to arrange matters so that cons is non-strict. One way to accomplish this is to extend the lazy evaluator to allow for non-strict primitives, and to implement cons as one of these. An easier way is to recall (Section 2.1.3) that there is no fundamental need to implement cons as a primitive at all. Instead, we can represent pairs as procedures:
```Scheme
(define (cons x y) (lambda (m) (m x y)))
(define (car z) (z (lambda (p q) p)))
(define (cdr z) (z (lambda (p q) q)))
```
In terms of these basic operations, the standard definitions of the list operations will work with infinite lists (streams) as well as finite ones, and the stream operations can be implemented as list operations. Here are some examples:
```Scheme
(define (list-ref items n)
  (if (= n 0)
      (car items)
      (list-ref (cdr items) (- n 1))))
(define (map proc items)
  (if (null? items)
      '()
      (cons (proc (car items)) (map proc (cdr items)))))
(define (scale-list items factor)
  (map (lambda (x) (* x factor)) items))
(define (add-lists list1 list2)
  (cond ((null? list1) list2)
        ((null? list2) list1)
        (else (cons (+ (car list1) (car list2))
                    (add-lists (cdr list1) (cdr list2))))))
(define ones (cons 1 ones))
(define integers (cons 1 (add-lists ones integers)))
;;; L-Eval input:
(list-ref integers 17)
;;; L-Eval value:
18
```
Note that these lazy lists are even lazier than the streams of Chapter 3: The car of the list, as well as the cdr, is delayed. In fact, even accessing the car or cdr of a lazy pair need not force the value of a list element. The value will be forced only when it is really needed—e.g., for use as the argument of a primitive, or to be printed as an answer.  

Lazy pairs also help with the problem that arose with streams in Section 3.5.4, where we found that formulating stream models of systems with loops may require us to sprinkle our programs with explicit delay operations, beyond the ones supplied by cons-stream. With lazy evaluation, all arguments to procedures are delayed uniformly. For instance, we can implement procedures to integrate lists and solve differential equations as we originally intended in Section 3.5.4:
```Scheme
(define (integral integrand initial-value dt)
  (define int
    (cons initial-value
          (add-lists (scale-list integrand dt) int)))
  int)
(define (solve f y0 dt)
  (define
    y (integral dy y0 dt))
  (define dy (map f y))
  y)
;;; L-Eval input:
(list-ref (solve (lambda (x) x) 1 0.001) 1000)
;;; L-Eval value:
2.716924
```




