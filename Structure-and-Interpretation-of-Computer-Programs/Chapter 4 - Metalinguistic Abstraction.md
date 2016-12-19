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

* For self-evaluating expressions, such as numbers, eval returnsthe expression itself.
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

















