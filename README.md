# Study Guide for [EECS 111 Exam 2, 2018 Spring Quarter][Course Page]

[Course Page]: http://users.eecs.northwestern.edu/~jesse/course/eecs111/

This document aims to review material from the second half of Northwestern University's EECS 111 course. I've also included some practice questions that might be good to solve before the exam. 

## Table of Contents
- [Lists](#lists)
- [Recursion](#recursion)
    - [Mutual Recursion](#mutual-recursion)
    - [Generative Recursion](#generative-recursion)
- [List Abstractions](#list-abstractions)
- [Local and Lambda](#local-and-lambda)

## Lists 
Lists are recursive data definitions. An example data definition for a list is as follows:
```racket
; A List-of-names is one of :
;- ‘()
;- (cons String List-of-names)
```

When lists are formed, one must always start with an empty list. An empty list is denoted as `‘()`. To check if a list is empty, we use the `empty?` predicate.

To form a list-of-names, we simply chain a list of `cons ` statements as follows:
```racket
(define peer-mentors 
    (cons “Rohit" 
        (cons “Jeffrey” 
            (cons “Hayden” 
                (cons “Alin” 
                     (cons “Aaron” 
                        (cons “Nila” 
                            (cons “Prashanth” 
                                (cons “Shu” ‘()))
```

The above list is equivalent to:
```racket 
(define peer-mentors 
    (list “Rohit” “Jeffrey” “Hayden” “Alin” “Aaron” “Nila” “Shu” “Prashanth”))
```

Some other list functions that you may find useful include:
* `append`: [list of X] [list of X] … -> [list of X] \
            Constructs a new list from several lists by concatenating the argument lists together.
    ```racket
    ; Example:
    (append (list 1 2) (list 3 4)) -> (list 1 2 3 4)
    ```

* `remove`: X [list of X] -> [list of X] \
            Constructs a new list wth the first occurrence of X removed.
    ```racket
    ; Example:
    (remove 2 (list 1 2 3 2 3)) -> (list 1 3 2 3)
    ```
* `length`: [list of X] -> Number \
            Returns the length of the argument list
    ```racket
    ; Example: 
    (length (list 1 2 3)) -> 3
    ```
## Recursion
**What is recursion?** \
Recursion occurs when a thing is defined in terms of *itself* or of *its type*.  

Consider the following data definition for a `TrainCar`:
```racket
; A trainCar is a (make-trainCar Number Train)
(define-struct TrainCar [capacity rest])
```

Consider the data definition for a `Train`:
```racket
A Train is one of:
; -  “caboose"
; -  (make-trainCar Number Train)
```

Here, the `Train` data-definition is defined in terms of a `Train`. Thus, the train data-definition is self-referential and said to be *recursive*.

How would we write a function to process this `Train ` data-definition? Well, we should define a **template** that closely follows the recursive data-definition of the train.

The template would be as follows:
```racket
(define (process-train a-train)
    (cond
        [(string? a-train) …]
        [else … (train-capacity a-train) …
                 … process-train (train-rest a-train)…]))
```

If our `Train` is a `String`, we know that the `Train` must be a `“caboose”`. If the `Train` is a `“caboose”`, we know we’ve reached the end of the `Train`. This is because the `Train` definition of `“caboose”` *does not* refer to the `Train` data definition. When we’ve reached the caboose, we complete some computation and terminate our `process-train` function. 

If our `Train` is not a `String`, it must be a `trainCar`. In this case, we complete some computation on the the current `trainCar` and then call `process-train` on the rest of the `Train` recursively, following the recursive data-definition of the `Train`.
 
[Lists](#lists) and 'list-like' data definitions (ex. the `Train`) are the most common recursive data-definitions that we’ve seen in class. But, they certainly are not the only recursive data-definitions. Another example of a recursive data-definition that we've seen in class is the `Alpaca`. A generalization of the `Alpaca` data-definition is a `BinaryTree.` 

In a BinaryTree, each `Node` is defined in terms of its left child `Node` and its right child `Node.` The formal data definition is
```racket
; A BinaryTree is one of:
; - "leaf"
; -  (make-Node Number BinaryTree BinaryTree)

(define-struct Node [value left right])
; A node is a (make-Node Number BinaryTree BinaryTree)
```

Some recursive data-definitions can get really complicated and difficult to work with (take a look at the data definitions in lab 6). Remember though, with recursive data definitions, you can *always* use structural decomposition to write a template that follows the recursive structure of the data-definition to process the data.

    Exercise 1:
    Consider the following BinaryTree: 

    (define EX
        (make-Node 1 (make-Node 2 (make-Node 4 "leaf" "leaf")
                                  (make-Node 5 "leaf" "leaf"))
                     (make-Node 3 "leaf" "leaf")))
                      
    Write a function to traverse EX and add every element to the list in the following order:
    a. 4 2 5 1 3
    b. 1 2 4 5 3
    c. 4 5 2 3 1

### Mutual Recursion
Sometimes many data-definitions are defined in terms of *each other*. If this is the case, the data-definitions are said to be *mutually recursive*. To be more concrete, here is a simple example of some mutually recursive data definitions (for a more complicated example, see Lab 7 for the JSON data-definition):

```racket
(define-struct Tree [value children])
; A Tree is one of:
; - “leaf"
; - (make-Tree Number Children)
; Note: A tree must have at least one child.

; Children is a [List-of Tree]
```

Here, `Tree` refers to `Children` and `Children` refers to `Tree`. `Tree` and `Children` are said to be mutually recursive. How would we write a function to process these `Tree` and `Children` data-definitions? Again, we should define *templates* that closely follows the recursive data-definitions of `Tree` and `Children`.

The templates would be as follows:
```racket
(define (process-tree a-tree)
    (cond
        [(string? a-tree) …]
        [else … (Tree-value a-tree) ...
                … (process-children (Tree-children a-tree))]))

(define (process-children some-children)
    (cond
        [(empty? some-children) …]
        [else … (process-tree (first some-children)
                 …(process-children (rest some-children)]))
```

When processing a `Tree`, we first check if the `Tree` is a "leaf". If the `Tree` is a "leaf", we complete some computation and end our recursion. If the `Tree` is not a "leaf", we complete some computation on the `Tree's` value but then call our recursive process-children function on the `Tree's` children.

When processing a `Tree`'s children, we first check if the `Children` is empty. If our list is empty, we know we've already processed all of the children so we can end our recursion. If `Children` is not empty, we process the first `Tree` in `Children` using our recursive `process-Tree` function, and recursively call `process-children` on the rest of the `Children`.

    Exercise 2:
    Write a template to process the following mutually recursive data definitions:

    ; A ListOfAlternatingNumbersandStrings is one of:
    ; - '()
    ; - (cons Number ListOfAlternatingStringsandNumbers)

    ; A ListOfAlternatingStringsandNumbers is one of:
    ; - '()
    ; - (cons String ListOfAlternatingNumbersandStrings)

Note: Example taken from Mitchell Wand's Northeastern University Lecture on Mutually-Recursive Data Definitions

### Generative Recursion
The recursion examples above are examples of *structural recursion* - that is, we exploit the structure of our data definition to organize the structure of our code. More concretely, if we are writing a function to do some computation on a list, we know we must process the first element of the list
and then the rest of the list. 

While structural recursion is a useful tool that can be used to solve some problems with recursive solutions, it *cannot* be used to solve all problems. Namely, it cannot solve problems where the input is not defined recursively. 

*Generative recursion* a general problem-solving technique that can be applied to a wide class of problems. Generative recursion generates a set of *subproblems* from the current problem at hand. The answers to these subproblems are then *combined* to provide the solution.

Here is the template for general recursion:
```racket
(define (generative-recursive-fun problem)
  (cond
    [(trivially-solvable? problem)
     (determine-solution problem)]
    [else
     (combine-solutions
       ... problem ...
       (generative-recursive-fun (generate-problem-1 problem))
       ...
       (generative-recursive-fun (generate-problem-n problem)))]))
```
See lecture 17 for some great worked examples of generative recursion (mergesort, quicksort, fibonacci). Here is an additional worked example of generative recursion:

Background: The Euclidean algorithm is an efficient way to find the greatest common divisor of two positive integers A and B. The algorithm works by repeated division. 
Read https://en.wikipedia.org/wiki/Euclidean_algorithm to understand the mechanics of the algorithm.

Now let's write a ISL function to calculate the GCD of two numbers.
```racket
; GCD : Natural Natural -> Natural
(define (GCD A B)
    (cond 
        [(zero? B) A]
        [else (GCD B (modulo A B))]))
```

**Why does this program terminate?** It isn't immediately obvious.

Run this function on a few different A and B inputs and use the DrRacket stepper. You will see that at each iteration of the algorithm, the modulo decreases by at least 1. Eventually, the modulo value reaches 0 and the program reaches its termination condition. 

Notice how each recursive call to `GCD` is generated based on the euclidean algorithm, and not the structure of the input (a natural number). Generative recursion problems are usually more challenging to solve than structural recursion problems and usually require some domain specific knowledge.



## List Abstractions 
`Map`, `filter`, and `foldr` are examples of list abstractions. They are examples of 'higher-order' functions. A higher-order function is a function that either:
- takes another function as an input
- returns a function

As we will see shortly `map`, `filter`, and `foldr` each take functions as inputs. Here are the semantics of `map`, `filter`, and `foldr`: 

* `Map`: [F: X ->Y] [List of X] -> [List of Y] \
Map applies a function to each element of a list and returns the list of results in the same order as the original list. \

    ```racket
    ; Example: Multiply by 2
    (map (lambda (x) (* x 2)) (list 1 2 3 4 5)) -> (list 2 4 6 8 10)
    ```

* `Filter`: [F: X->Boolean] [List of X] -> [List of X] \
Filter applies a function to each element of a list and returns the list of elements that the function returns true for.

    ```racket;
    ; Example: Filter even numbers
    (filter (lambda (x) (= 0 (modulo x 2))) (list 1 2 3 4 5)) -> (list 2 4)
    ```

* `Andmap`: [F: X->Boolean] [List of X] -> Boolean \
Andmap applies a function to each element of a list and 'and's each result together

    ```racket
    ; Example: Are all elements even?
    (andmap (lambda (x) (= 0 (modulo x 2))) (list 2 4 6 8)) -> #true
    ```

* `Ormap`: [F: X->Boolean] [List of X] -> Boolean \
Ormap applies a function to each element of a list and 'or's each result together

    ```racket 
    ; Example: Is there at least one even element in the list?
    (ormap (lambda(x) (= 0 (modulo x 2))) (list 1 3 5 7 2)) -> #true
    ```

* `Foldr`: [F: X Y -> Y] Y [List of X] -> Y \
Foldr applies a function to each element of a list (starting with the right) and the second argumentof foldr. It subsequently updates the second argument with this return value. Once all elements of the input list have been considered, the second argument is returned.

    ```racket
    ; Example: Sum all elements in a list
    (foldr (lambda (x y) (+ x y)) 0 (list 1 2 3 4 5)) -> 15
    ```

  Note: for each of these functions, the variable x in the lambda will be bound to single element of the list that is taken as an input.

```
Exercise 3: Use foldr to write map  

Exercise 4 Use foldr to write andmap and ormap 

Exercise 5: Use foldr to write filter

Exercise 6: When might it be useful to use map with a higher order-function that returns a function and a list to return a list of functions?
```

## Local and Lambda
### Lambda
The keyword `lambda` defines *anonymous* functions. An anonymous function is simply a function not bound to a name. Lambda functions are generally used to define functions that higher-order functions take as input. See [this](#list-abstractions) for some examples. 

Lambda functions are generally preferred to named functions in the context of higher order functions as the function definition code is kept close to its call within the higher order function. Also, lambda functions are generally used for one-off computations exclusively used in the context of the higher-order-function is defined, so it doesn't make sense for the functions to be defined globally in the program.

### Local
According to the Racket documentation, local has the following semantics:
    `(local [definition ...] body ...)` The following examples will make this syntax more clear.

ISL allows for local definitions within expressions. There are two main benefits of using a local defintion:
- Local definitions provide names for *intermediate* values. They can help break complicated expressesions into a series of simpler ones:
    ```racket 
    ; Ex. Let's write a function to take the slope of a line:
    ; slope: Number Number Number Number -> Number
    (define (slope x1 y1 x2 y2)
        (local
            [(define rise (- y2 y1)) ;definition
            (define run (- x2 x1))] ;definition
            (/ rise run))) ;body
    ```
        
- Local definitions for functions can limit the scope of helper functions.
    ```racket 
    ; Ex. Use a locally defined helper function to calculate the fatorial of a number 
    ; factorial: Number -> Number
    (define (factorial n)
        (local
            [(define (factorial-helper index accum)  ;definition
                (cond
                    [(> n i) (factorial-helper (+ index 1) (* index accum))]
                    [else accum]))]
            (factorial-helper 1 1))) ;body
    ```

    See `do-filter` in lab 8 for another example of local defintions of functions.
    There are several things to unpack in `factorial`:
    1) `Accum`  holds on to our in-progress 'answer' as we step through the factorial algorithm.
    2) Notice that `factorial-helper` references the variable `n`. `N` is a parameter of the outer `factorial` function. If `factorial-helpe`r was defined outside of `factorial`,
    it would need to take an extra parameter to have access to n. Every inner function/definition can reference the parameters of the outer function (even after the outer function returns). This is formally called **lexical scoping**.

    Here is another example of lexical scoping using lambda:
    ```racket
    (define (larger-than-predicate n)
        (lambda (v) (> v n )))

    (filter (larger-than-predicate 5) '(1 2 3 4 5 6 7 8 9 10))
    ```


### An aside - closures: 
```racket
(define (create-closure x)
    (lambda (y) (+ x y)))
```

This function returns a *function* that has access to x (because of lexical scoping). This returned function with access to x is formally called a **closure**. Not all functions returned by functions are closures, only those that have access to a variable defined outside of it. Don't worry if this doesn't quite make sense.

Let's do:
```racket
(define my-function (create-closure 5)). 
```
This definition binds the value of 5 to `x` and the name `my-function` to `(lambda(y) (+ 5 y))`. 

Now we can call my-function as such:
```racket
(my-function 10)
```
This binds the value of `y` in `create-closure` and executes the function, yielding 15.

Note: `Filter-digits` in Lab 8 returns a closure, as well.
If you're interested in learning more about closures, check out: https://en.wikipedia.org/wiki/Closure_(computer_programming)

### You DON'T NEED to know what closures are and lexical scoping are formally. Just be aware that they exist and that we've been using them.

    Exercise 7: Write another recursive version of factorial
    Exercise 8: Write factorial using foldr
