---
date: 2013-11-10
title: Implementing (the original) Lisp in Python
description: "To understand what Lisp is all about, let's go back to the roots and
    have a look at John McCarthy's original Lisp. By implementing it in Python we see
    how, with just a few basic operators and a simple concept of functions, we're able
    to implement the rest of the language within itself."
---

# Implementing (the original) Lisp in Python

As programmers we take great pride in keeping up with new developments in libraries, languages and tools (and usually have a lot of fun doing it as well).
If we stop learning, we know we'll soon end up like the Cobol-programmers of today.
And this is of course a good thing — there's always something new and exciting to learn, something on which to sharpen our skills.

But sometimes I also find that it pays off to take a look at the old, rather than the new.
It's good to go back to the roots, to see where it all comes from, and to have look at the fundemental ideas.
Today we'll take a trip back to 1960, to the origins of Lisp as described by John McCarthy in his paper [Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part 1][rec-func].

There is really nothing novel in this blogpost.
The ideas all belonged to McCarthy, but sometimes it's good to just study the masters.
Lets explore the origins of one of our most powerful (family of) programming languages.
We'll start by briefly covering the Lisp syntax and semantics, before moving on to an implementation of the language itself in Python.

## A short Lisp 101

Lets start out by explaining the basics of Lisp. If you're already familiar with this, I urge you to move on to the next section instead.

I have tried to keep this section brief. If you would like something a bit more elaborate, I really recommend [The Roots of Lisp][roots] by Paul Graham, where he essentially describes McCarthy's Lisp by example and then show an implementation in Common Lisp.

### A wee bit of syntax

Lisp programs consists of something called [s-expressions][s-exps], which stands for *symbolic expressions*. S-expressions are defined recursively, and consists of either a single **atom** or a **list**, which contains other s-expressions.

An *atom* is akin to what many modern languages would call an identifier. It consists of a series of letters or symbols — anything other than parentheses, single quote and whitespace, basically. Examples of atoms would be `a`, `foo/bar!` or `+`.

*(Note that we simplify a bit here, compared to McCarthys original description. By limiting atoms to not contain spaces, we eliminating the need for commas separating atoms within lists. In the original Lisp, atoms were also limited to upper case letters, a restriction I don't see the need to enforce here.)*

*Lists* use the parenthesis syntax that have become so iconic to Lisp. A list is expressed by a pair of parentheses enclosing a number of elements. Each element is another s-expression, which might be either an atom or a list. Here are a copule of examples:

> ```no-highlight
> (list of only atoms)
> (another list (with some ((nested)) (lists inside)))
> ```

In addition, although it's actually not in the original Lisp, we include a shorthand syntax `'x` which is evaluated as `(quote x)`, where `x` is any s-expression. This way, `'(a b c)` is interpreted as`(quote (a b c))` and similarly, `'foo` as `(quote foo)`. We will later see why this turns out to be useful.

### The basic semantics

When evaluating an s-expression `e`, the following rules apply.

- If `e` is an atom its value is looked up in the environment.
- Otherwise, the expression is a list like `(<e0> <e1> … <en>)`, which is evaluated as a function application. How this is handled depends on the first element of the list, `e0`.
    * If `e0` is the name of one of the builtin (axiomatic) forms, it is evaluated as described below.
    * If `e0` is any other atom, its value is looked up. A new list, with the value of `e0` replacing the first element is then evaluated.
    * If `e0` is not an atom, but a list of the form `(lambda (<a1> … <an>) <body>)`, then `e1` through `en` is first evaluated. Then `body` is evaluated in an environment where each of `a1` through `an` points to the value of the corresponding `en`. This constitutes a call to an anonymous function (i.e. a `lambda` function).
    * If `e0` is of form `(label <name> <lambda>)` where `lambda` is a lambda expression like the one above, then a new list with `e0` replaced by just the `lambda` is constructed. This list is then evaluated in an environment where `name` points to `e`. The `label` notation is how we solve the problem of defining recursive functions.

### The axiomatic forms

The axiomatic forms are the basis on which the rest of the language rests. They behave as follows:

<dl>
    <dt>`(quote e)`</dt>
    <dd>
        returns `e` without evaluating it first.
    </dd>

    <dt>`(atom e)`</dt>
    <dd>
        evaluates `e` and returns the atom `t` if the resulting value is an atom, otherwise `f` is returned. (Since we have no boolean type in our language, these two atoms are treated as *true* and *false*, respectively.)
    </dd>

    <dt>`(eq e1 e2)`</dt>
    <dd>
        evaluates to `t` if both `e1` and `e2` evaluates to the same atom, otherwise `f`.
    </dd>

    <dt>`(car e)`</dt>
    <dd>
        evaluates `e`, which is expected to give a list, and returns the first element of this list.
    </dd>

    <dt>`(cdr e)`</dt>
    <dd>
        is the opposite of `car`. It returns all but the first element of the list gotten by evaluating `e`. If the list only holds only one element, `cdr` instead returns the atom `nil`.
    </dd>

    <dt>`(cons e1 e2)`</dt>
    <dd>
        evaluates both `e1` and `e2`, and returns a list constructed with the value of `e1` as the first element and the value of `e2` as the rest. If `e2` evaluates to the atom `nil`, the list `(e1)` is returned.
    </dd>

    <dt>`(cond (p1 e1) … (pn en))`</dt>
    <dd>
        is the *conditional* operator. It will evaluate predicates `p1` to `pn` in order, until one of them evaluates to `t`, at which time it will evaluate the corresponding `en` and return its value.
    </dd>
</dl>

The evaluation rules above are, surprisingly, all we need to implement Lisp. In addition, however, I'd like to include another form that isn't explicitly described by McCarthy, but which is included in Graham's article. It could strictly speaking be replaced by doing a lot of nested `label`s but, but this would make things a lot less readable.

<dl>
    <dt>`(defun name (a1 … an) <body>)`</dt>
    <dd>
        is a way to define functions and then later use them outside of the `define` expression. It does this by adding a new binding to the environment it is itself evluated in: `name`→`(lambda (a1 … an) <body>)`.
    </dd>
</dl>


## Implementation in Python

Now, with the syntax and core semantics of the language outlined, lets look at how to make this happen in Python.

### The parser

The first step when implementing a language is usually the parser. We need some way to go from programs as strings to some datastructure we can interpret. Such a datastructure is usually called the [Abstract Syntax Tree][ast] (AST) of the program.

Since Lisp is a language largely without syntax, with parentheses and atoms used for everything, writing the parser is relatively easy and uninteresting. This is not what I want to focus on in this blogpost, so we'll skip over the details here. Feel free to have a look at [the code for the parser][gh-parser] before we move on, if you like.

The parser works like this:

```python
>>> from rootlisp.parser import parse
>>> program = "(lambda (x) (cons x (cons x '())))"
>>> ast = parse(program)
>>> ast
['lambda', ['x'], ['cons', 'x', ['cons', 'x', ['quote', []]]]]
```

The `parse` function takes one argument, the program string, and returns the corresponding AST. There is also an opposite function `unparse` which converts ASTs back into Lisp source strings.

```python
>>> from rootlisp.parser import unparse
>>> unparse(ast)
"(lambda (x) (cons x (cons x '())))"
```

### Interpreting

Before we move on to evaluating the ASTs, let's define another useful function, `interpret`, which we'll be using to test our language as we go:

```python
def interpret(program, env=None):
    ast = parse(program)
    result = eval(ast, env if env is not None else [])
    return unparse(result)
```

The function combines the power of `parse` and `eval`. `interpret` takes a Lisp program as a string, parses it, and finally evaluates the parsed expression. Since the evaluated expression might be a Lisp data structure (and even valid Lisp code) we "unparse" it back to it's corresponding source string. This is done to hide our internal ASTs from the user of the Lisp.

### The Evaluator

With parsing out of the way, and armed with the `interpret` function to test our code, it's time to have a look at the core of the langage, the `eval` function. It looks like this:

```python
def is_atom(exp):
    """Atomes are represented by strings in our ASTs"""
    return isinstance(exp, str)

def eval(exp, env):
    """Function for evaluating the basic axioms"""
    if is_atom(exp): return lookup(exp, env)
    elif is_atom(exp[0]):
        if exp[0] == "quote": return quote(exp)
        elif exp[0] == "atom": return atom(exp, env)
        elif exp[0] == "eq": return eq(exp, env)
        elif exp[0] == "car": return car(exp, env)
        elif exp[0] == "cdr": return cdr(exp, env)
        elif exp[0] == "cons": return cons(exp, env)
        elif exp[0] == "cond": return cond(exp, env)
        elif exp[0] == "defun": return defun(exp, env)
        else: return call_named_fn(exp, env)
    elif exp[0][0] == "lambda": return apply(exp, env)
    elif exp[0][0] == "label": return label(exp, env)
```

As you can see, `eval` takes two arguments `exp` and `env`. `exp` is one of the ASTs returned by `parse`, `env` holds a list of associations which represent bindings from atoms to values in the environment.

We have now covered all the cases we need in order to implement the Lisp. Lets look at the implementation of each in turn. Keep the structure of `eval` in mind when we go through each case.

### Evaluating atoms

The first case we need to cover is when the evaluated expression is an atom.
The value of an atom is whatever it is bound to in the environment, so we do a lookup of the atom in `env`.

```python
def lookup(atom, env):
    for x, value in env:
        if x == atom:
            return value
```

Lets have a look at how this works in the REPL:

```python
>>> from rootlisp.lisp import interpret
>>>
>>> env = [('foo', 'bar')]
>>> interpret('foo', env)
'bar'
```

### Evaluating `quote`

The next form is `quote`, which is incredibly easy to implement: all we need to do is simply to return whatever the argument was, *without* evaluating it first.

```python
def quote(exp):
    # (quote e1)
    return exp[1]
```

And it works as expected:

```python
>>> interpret('(quote a)')
'a'
>>> interpret("'a")
'a'
>>> interpret("'(a (b (c) d))")
'(a (b (c) d))'
```

### Evaluating `atom`

The next case, `atom` determines whether the value of `exp` is atomic or not.

```python
def atom(exp, env):
    # (atom e1)
    val = eval(exp[1], env)
    return 't' if is_atom(val) else 'f'
```

Our Lisp does not have any boolean datatypes, so we simply return the atoms `t` or `f` depending on whether `exp` is an atom or not.

```python
>>> interpret("(atom 'a)")
't'
>>> interpret("(atom '(a b c))")
'f'
>>> interpret("(atom (atom 'a))")
't'
```

### Evaluating `eq`

The `eq` function is defined as `t` if the value of its two arguments evaluates to the same atom.

```python
def eq(exp, env):
    # (eq e1 e2)
    v1 = eval(exp[1], env)
    v2 = eval(exp[2], env)
    return 't' if v1 == v2 and is_atom(v1) else 'f'
```

```python
>>> interpret("(eq 'a 'a)")
't'
>>> interpret("(eq 'a 'b)")
'f'
>>> interpret("(eq '(a) '(a))")
'f'
```

### Evaluating `car` and `cdr`

The `car` and `cdr` forms both evaluate the argument, expecting the resulting value to be a list. `car` returns the first item of the list; `cdr` returns the rest of the list, i.e. everything but the first element.

```python
def car(exp, env):
    # (car e1)
    return eval(exp[1], env)[0]

def cdr(exp, env):
    # (cdr e1)
    lst = eval(exp[1], env)
    return 'nil' if len(lst) == 1 else lst[1:]
```

Notice that if the list contains only one element, `cdr` return the atom `nil` which represents the empty list.

```python
>>> interpret("(car '(a b c))")
'a'
>>> interpret("(cdr '(a b c))")
'(b c)'
>>> interpret("(cdr '(a))")
'nil'
```

### Evaluating `cons`

`cons`, short for *construct*, returns a list constructed with the value of the first argument as the first element, and the value of the second argument as the rest of the list.

```python
def cons(exp, env):
    # (cons e1 e2)
    rest = eval(exp[2], env)
    if rest == 'nil':
        rest = []
    return [eval(exp[1], env)] + rest
```

Once again, we treat `nil` as the empty list.

```python
>>> interpret("(cons 'a '(b c))")
'(a b c)'
>>> interpret("(cons 'a 'nil)")
'(a)'
```

### Evaluating `cond`

The expressions passed as arguments to `cond` are all lists of two elements. We evaluate the first element of each of the sublists in turn, until one evaluates to `t`. When the first `t` is found, the second element of that list is evaluated and returned.

```python
def cond(exp, env):
    # (cond (p1 e1) (p2 e2) …)
    for p, e in exp[1:]:
        if eval(p, env) == 't':
            return eval(e, env)
```

Like in McCarthy's original Lisp, our `cond` is also undefined for cases where no `p` expressions evaluate to `t`.

```python
>>> program = """
...   (cond ((eq 'a 'b) 'first)
...         ((atom 'a) 'second))
... """
>>> interpret(program)
'second'
```

### Evaluating `defun`

As noted above, the `defun` form isn't one of those specified by McCarthy, but we include it anyway to make the language easier to use. Evaluating a `defun` expression simply extends the environment where it is called with a `label` structure containing a `lambda`. The evaluation of `lambda`s and `label`s is described below.

```python
def defun(exp, env):
    # (defun my-fun (a1 …) body)
    name, params, body = exp[1], exp[2], exp[3]
    label = ["label", name, ["lambda", params, body]]
    env.insert(0, (name, label))
    return name
```

To see what's happening, lets look at the environment after evaluating a `defun` form.

```python
>>> env = []
>>> interpret("""
...     (defun pair (x y)
...         (cons x (cons y 'nil)))
... """, env)
'pair'
>>> env
[('pair', ['label', 'pair', ['lambda', ['x', 'y'], ['cons', 'x', ['cons', 'y', ['quote', 'nil']]]]])]
```

### Evaluating function calls

To round of the case when the first element in `exp` in `eval` is an atom, we simply look this atom up in the environment, expecting to find a function. A new list with this function as the first element is then evaluated instead.

```python
def call_named_fn(exp, env):
    # (my-fun e1 …)
    fn = lookup(exp[0], env)
    return eval([fn] + exp[1:], env)
```

Lets try testing this by calling the `pair` function we defined with `defun` above.

```python
>>> env
[('pair', ['label', 'pair', ['lambda', ['x', 'y'], ['cons', 'x', ['cons', 'y', ['quote', 'nil']]]]])]
>>> interpret("(pair 'a 'b)", env)
'(a b)'
```

### Evaluating `lambda` application

In the example above, `pair` is looked up in the environment and a new s-expression is evaluated. This new expression holds a function rather than an atom as the first element. (Actually, it holds a `label` with a function, but the `label` is stripped away in an intermediate step as explained bellow.) Thus, we end up evaluating an expression where the first element looks something like `(lambda (list of parameters) body)`. The rest of the elements in `exp` are the arguments to the function. The `apply` function evaluates such expressions.

```python
def apply(exp, env):
    # ((lambda (a1 …) body) e1 …)
    fn, args = exp[0], exp[1:]
    _, params, body = fn
    evaluated_args = map(lambda e: eval(e, env), args)
    new_env = zip(params, evaluated_args) + env
    return eval(body, new_env)
```

The first line separates the lambda-expression `fn` and the arguments. The function `fn` is then split further into its list of parameters and the body. The arguments are then each evaluated, before they are merged with the corresponding parameters and put into the environment. Finally, the body of the function is evaluated in this new environment.

```python
>>> program = """
...   ((lambda (x y) (cons x (cdr y)))
...       'z
...       '(a b c))
... """
>>> interpret(program)
'(z b c)'
```

### Evaluating `label` application

The `lambda` syntax above is fine for defining normal non-recursive functions. It is also expressive enough to make recursive functions using the [Y-combinator][y-combinator], but for this McCarthy introduces the `label` notation instead (which arguably is a lot easier to understand).

This evaluation case considers expressions on the form `((label name lambda-expression) arguments)`.

```python
def label(e, a):
    # ((label name (lambda (p1 …) body)) arg1 …)
    _, f, fn = e[0]
    args = e[1:]
    return eval([fn] + args, [(f, e[0])] + a)
```

We handle this by extending the environment such that `name` points to the first element of `e`, i.e. the `label` expression. The `lambda` function is then applied to the rest of the elements of `e` (the arguments) in this environment, and the value returned.

Lets se an example:

```python
>>> program = """
...   ((label greet (lambda (x)
...                   (cond ((atom x)
...                           (cons 'hello (cons x 'nil)))
...                         ('t (greet (car x))))))
...    '(world))
... """
>>> interpret(program)
'(hello world)'
```

## Taking the Lisp for a test run

And with that, we have enough of Lisp implemented to be able to start using it. Lets define a few functions.

We start with something simple, a function for checking whether lists are empty or not.

```common-lisp
(defun null (x)
    (eq x 'nil))
```

`null` returns `t` for any list that is without elements, and `f` otherwise.

```common-lisp
> (null '(foo bar))
f
> (null (cdr '(a)))
t
```

We might also define the common logical operators.

```common-lisp
(defun and (x y)
    (cond (x (cond (y 't) ('t 'f)))
          ('t 'f)))

(defun or (x y)
    (cond (x 't)
          ('t (cond (y 't) ('t 'f)))))

(defun not (x)
    (cond (x 'f)
          ('t 't)))
```

Both `and`, `or` and `not` works as one would expect.

```common-lisp
> (not 'f)
t
> (not (and 't (or 't 'f)))
f
```

Further we can define some functions working on lists. First `append`, which takes two lists as arguments, returning their concatination.

```common-lisp
(defun append (x y)
    (cond ((null x) y)
          ('t (cons (car x) (append (cdr x) y)))))
```

A couple of tests shows that it works:

```common-lisp
> (append '(1 2 3) '(a b c))
(1 2 3 a b c)
> (append 'nil '(a b))
(a b)
```

Another useful function is `zip`, which takes two lists as arguments, returning a list of pairs where each pair consists of the corresponding elements from each of the argument lists.

```common-lisp
(defun pair (x y)
  (cons x (cons y 'nil)))

(defun zip (x y)
  (cond ((and (null x) (null y)) 'nil)
        ((and (not (atom x)) (not (atom y)))
         (cons (pair (car x) (car y))
               (zip (cdr x) (cdr y))))))
```

The helper function `pair` is simply used as a convenience for creating lists of two elements.

```common-lisp
> (zip '(a b c) '(1 2 3))
((a 1) (b 2) (c 3))
```

## Completing the language

These functions are all nice and well, but one thing is still lacking.
One of the central concepts in Lisp is that *code is data*, and vice versa.
We already have `quote` which enables us to convert code into lists, but we still need some way to evaluate lists as if they were Lisp code again.

Our Lisp cannot do this yet. But, fortunately, we have enough pieces to be able to implement it within the Lisp itself!

Before we go on, lets just define a few shorthand notations for working with combinations of `car` and `cdr`. These will help keep the code a bit more concise and readable as we move on.

```common-lisp
(defun caar (lst) (car (car lst)))
(defun cddr (lst) (cdr (cdr lst)))
(defun cadr (lst) (car (cdr lst)))
(defun cdar (lst) (cdr (car lst)))
(defun cadar (lst) (car (cdr (car lst))))
(defun caddr (lst) (car (cdr (cdr lst))))
(defun caddar (lst) (car (cdr (cdr (car lst)))))
```

Next, we need a function to help us look up values from an environment.

```common-lisp
(defun assoc (var lst)
  (cond ((eq (caar lst) var) (cadar lst))
        ('t (assoc var (cdr lst)))))
```

`assoc` takes two arguments: the variable we wish to look up, `var`, and a list of bindings, `lst`. The bindings in `lst` are lists of two elements, and `assoc` simply returns the second element of the first pair where the first element is the same as `var`.

```common-lisp
> (assoc 'x '((x a) (y b)))
a
> (assoc 'y '((x a) (y b)))
b
```

With this, we are ready to implement the `eval` function:

```common-lisp
(defun eval (exp env)
  (cond
    ((atom exp) (assoc exp env))
    ((atom (car exp))
     (cond
       ((eq (car exp) 'quote) (cadr exp))
       ((eq (car exp) 'atom)  (atom (eval (cadr exp) env)))
       ((eq (car exp) 'eq)    (eq   (eval (cadr exp) env)
                                    (eval (caddr exp) env)))
       ((eq (car exp) 'car)   (car  (eval (cadr exp) env)))
       ((eq (car exp) 'cdr)   (cdr  (eval (cadr exp) env)))
       ((eq (car exp) 'cons)  (cons (eval (cadr exp) env)
                                    (eval (caddr exp) env)))
       ((eq (car exp) 'cond)  (evcon (cdr exp) env))
       ('t (eval (cons (assoc (car exp) env)
                        (cdr exp))
                  env))))
    ((eq (caar exp) 'label)
     (eval (cons (caddar exp) (cdr exp))
            (cons (pair (cadar exp) (car exp)) env)))
    ((eq (caar exp) 'lambda)
     (eval (caddar exp)
            (append (zip (cadar exp) (evlis (cdr exp) env))
                     env)))))

(defun evcon (c env)
  (cond ((eval (caar c) env)
         (eval (cadar c) env))
        ('t (evcon (cdr c) env))))

(defun evlis (m env)
  (cond ((null m) 'nil)
        ('t (cons (eval  (car m) env)
                  (evlis (cdr m) env)))))
```

As you see, the Lisp version of `eval` is very similar to the one we implemented in Python, both in structure and how it works. Lets see a couple of examples.

```common-lisp
> (eval '(cons x '(b c)) '((x a) (y b)))
(a b c)
> (eval '(f '(bar baz)) '((f (lambda (x) (cons 'foo x)))))
(foo bar baz)
```

We have the Lisp implemented in terms of the Lisp itself.

## Summary

We have now seen a full implementation of the original Lisp, and the final result is, of course, [available on github][gh-root].

The core of the language is pretty small. Given only a handful of axiomatic forms, implemented in Python, we were actually able to implement the rest of the language in itself. This implementation even included an `eval` function, able to interpret any new Lisp code (which makes this an example of a [meta-circular interpreter][meta-circular]).

Of course, while being a neat little language, our Lisp is missing a lot of features we expect in programming languages today. For example, it has no side effects (no IO), no types other than atoms (e.g. no numbers, strings, etc), no error handling, and it has dynamic rather than lexical scoping. The behaviour is also undefined for incorrect programs, and as an effect of this the error messages (which would bubble up from Python) can be rather strange and uninformative at times.

Most of this could easily be rectified, though. Either from within the Lisp itself, or by changing the Python implementation. To learn about some of the improvements that could be made, notably *lexical scoping* and *mutable state*, I reccomend to have a look at [The Art of the Interpreter][art-interpreter] by Steele and Sussman.

I hope this blogpost have peaked your interest in how programming languages work, and that you find the implementation of Lisp as delightful as I do.

[rec-func]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.111.8833&rep=rep1&type=pdf
[roots]: http://www.paulgraham.com/rootsoflisp.html
[ast]: http://en.wikipedia.org/wiki/Abstract_syntax_tree
[s-exps]: http://en.wikipedia.org/wiki/S-expression
[quoting]: http://en.wikipedia.org/wiki/Lisp_(programming_language)#Self-evaluating_forms_and_quoting
[y-combinator]: http://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator
[gh-root]: https://github.com/kvalle/root-lisp
[gh-parser]: https://github.com/kvalle/root-lisp/blob/master/rootlisp/parser.py
[gh-tests]: https://github.com/kvalle/root-lisp/tree/master/tests
[art-interpreter]: http://18.7.29.232/handle/1721.1/6094
[meta-circular]: http://en.wikipedia.org/wiki/Meta-circular_evaluator
