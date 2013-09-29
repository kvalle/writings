# Implementing (the original) Lisp in Python

> As programmers, we have a lot of fun learning new languages and techniques.
> …
> But sometimes it's good to go back to the source, to see where it all comes from and have look at the fundemental ideas.
> 
> Today I'll take a look Lisp, as originally described by John McCarthy in his 1960 paper [Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part 1][rec-func]. Despite the ideas being expressed using mathematical notation, the paper is actually highly readable, and I encourage you to have a look. 
> 
> Another great source for insights into the original Lisp ideas is the article [The Roots of Lisp][roots] by Paul Graham, where he explains McCarthy's core idea, and why it's so powerful. 
> 
> There is really nothing novel in this blogpost. The ideas all belonged to McCarthy, but sometimes it's good to study the masters. Lets explore the origins of one of our most powerful programming languages.
> 
> We'll start by briefly covering the Lisp syntax and parsing, before moving on to the implementation of the language itself.

## A short Lisp 101

Lets start out briefly explaining the syntax and core semantics of Lisp. If you're already familiar with this, please skip down to the next section.

### A wee bit of syntax

Lisp programs consists of forms called [s-expressions][s-exps], which stands for *symbolic expressions*. S-expressions are defined recursively, and consists of either a single *atom* or a list of s-expressions.

An *atom* is akin to what many modern languages would call an identifier. It consists of a series of letters or symbols — anything other than parens, single quote and whitespace, basically. Examples of atoms would be `a`, `foo/bar!` og `+`. 

(*Note that we are simplify a bit here, compared to McCarthys original description, by limiting atoms to not contain spaces, thus eliminating the need for commas separating atoms in lists. Also, his original description only allowed UPPER CASE letters for atoms, no symbols. *)

Lists use the iconic parentheses that some people love to hate. A list is simply a pair of parens enclosing a number of other s-expressions, which might be either atoms or other lists. Examples of lists include `(list of only atoms)` and `(list (with some ((nested)) (lists inside)))`.

In addition, although it's not in the original Lisp, we include a shorthand syntax `'x` for expressing `(quote x)`, where `x` is a s-expression. This way, `'(a b c)` is interpreted as`(quote a b c)` and similarly, `'foo` as `(quote foo)`.

### The basic semantics

When evaluating an s-expression `e`, the following rules apply.

If the expression is an atom its value is looked up in the environment.
Otherwise, the expression is a list like `(e0 e1 … en)`, which is evaluated as function application. How this is handled depends on the first element of the list, `e0`.

* If `e0` is one of the builtin (axiomatic) forms,  its evaluated is described below.
* If `e0` is another atom, a new list with the value of `e0` replaced by its valued looked up in the environment is constructed, and this list is evaluated.
* If `e0` is of the form `(lambda (<a1> … <an>) <body>)`, then `e1` through `en` is first evaluated. Then `body` is evaluated in an environment where each of `a1` through `an` points to the value of the corresponding `en`.
* If the expression is of form `(label <name> <lambda>)` where `lambda` is a lambda expression like the one above, then a new list with `e0` replaced by just the `lambda` is constructed. This list is then evaluated in an environment where `name` points to `e`.

### The axiomatic forms

**`(quote e)`** returns `e` without evaluating it first.

**`(atom e)`** evaluates `e` and returns the atom `t` if the resulting value is an atom, otherwise `f` is returned.

**`(eq e1 e2)`** evaluates to `t` if both `e1` and `e2` both evaluates to the same atom, otherwise `f`.

**`(car e)`** evaluates `e` (which should give a list), and returns the first element of this list.

**`(cdr e)`** is quite the opposite of `car`, returns all but the furst element of the list gotten by evaluating `e`. If the list only holds one element, `cdr` returns the atom `nil`.

**`(cons e1 e2)`** evaluates both `e1` and `e2`, and returns a list constructed with `e1` as the first element and `e2` as the rest. If `e2` evaluates to the atom `nil`, the list `(e1)` is returned.

**`(cond ((p1 e1) … (pn en)))`** is the *conditional* operator. It will evaluate predicates `p1` to `pn` in order, until one of them evaluates to `t`, at which time it will evaluate the corresponding `en` and return its value.

The evaluation rules above are, surprisingly, all we need to implement Lisp. In addition, however, I'd like to include another form that don't seem to be described by McCarthy, but which is included in Graham's article. It could strictly speaking be replaced by doing a lot of nestd `label`s but, makes things much more readable. It is the `define` form:

**`(defun name (a1 … an) <body>)`** is a way to define functions and then later use them outside of the `define` expression. It does this by adding a new binding to the environment it is itself evluated in: 
`name`→`(lambda (a1 … an) <body>)`.

With that, lets look at how to make this happen in Python.

## Implementation in Python

### The parser

The first part of creating any language is usually the parser. We need some way to go from programs as strings to some datastructure we can interpret. Such a datastructure is usually called the [Abstract Syntax Tree][ast] (AST) of the program.

Since Lisp is a language largely without syntax, with parentheses and atoms used for everything, writing the parser is relatively easy and uninteresting.
I've written [a simple parser][gh-parser] which we'll be using, which works like this:

```python
>>> from rootlisp.parser import parse
>>> program = "(lambda (x) (cons x (cons x '())))"
>>> ast = parse(program)
>>> ast
['lambda', ['x'], ['cons', 'x', ['cons', 'x', ['quote', []]]]]
```

The `parse` function takes one argument, the program string, and returns the corresponding AST. There is also an opposite funciton `unparse` which converts ASTs back into Lisp source strings.

```python
>>> from rootlisp.parser import unparse
>>> unparse(ast)
"(lambda (x) (cons x (cons x '())))"
```

The implementation of the parser is not the focus of this blogpost, so we'll skip over it here. Feel free to have a look at [the implementation][gh-parser] before we move on, if you like.

### Interpreting

Before we move on to how to evaluate the ASTs, let's define another useful function, `interpret`, which we'll be using to test our language as we go:

```python
def interpret(program, env=None):
    ast = parse(program)
    result = eval(ast, env if env is not None else [])
    return unparse(result)
```

This function combines the power of `parse` and `eval`. `interpret` takes a Lisp program as a string, and parse and evaluate it. Since the result might be a Lisp data structure (or even valid Lisp code), we "unparse" it back to it's corresponding source string. 

### The Evaluator

With parsing out of the way, and armed with the `interpret` function to test our code, it's time to have a look at the core of the langage, the `eval` function. It looks like this:

```python
def eval(e, a):
    if is_atom(e): return lookup(e, a)
    elif is_atom(e[0]):
        if e[0] == "quote": return quote(e)
        elif e[0] == "atom": return atom(e, a)
        elif e[0] == "eq": return eq(e, a)
        elif e[0] == "car": return car(e, a)
        elif e[0] == "cdr": return cdr(e, a)
        elif e[0] == "cons": return cons(e, a)
        elif e[0] == "cond": return cond(e, a)
        elif e[0] == "defun": return defun(e, a)
        else: return call_named_fn(e, a)
    elif e[0][0] == "lambda": return apply(e, a)
    elif e[0][0] == "label": return label(e, a)
```

As you can see, `eval` takes two arguments `e` and `a`. `e` is one of the ASTs returned by `parse`, `a` holds a list of associations which represents bindings from atoms to values in the environment. 

This covers all the cases we need to cover to implement the Lisp. Now lets look at the implementation of each in turn. Keep the structure of `eval` above in mind when we go through each case below.

### Evaluatings atoms

The first case we need to cover is when the evaluated expression is an atom. Atoms are represented by strings in our AST, so we just need to check the type of `e`.

```python
def is_atom(e): 
    return isinstance(e, str) 
```

The value of an atom is whatever it is bound to in the environment, so we do a lookup of `e` in `a`.

```python
def lookup(e, a):
    for x, value in a:
        if x == e:
            return value
```

Lets test this<sup>[1](#footnote-1)</sup>:

```python
>>> from rootlisp.lisp import interpret
>>> 
>>> env = [('foo', 'bar')]
>>> interpret('foo', env)
'bar'
```

### Evaluating `quote`

Implementing `quote` is perhaps the simplest part of the whole language.
Since atoms are handled, we now know we are evaluating a list, and check if the first element in the list is `quote`. If it is, simply return whatever is the next element of the list *without* evaluating it.

```python
def quote(e):
    # e -> (quote e1)
    return e[1]
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

The next case, `atom` determines whether the value of `e` is atomic or not.

```python
def atom(e, a):
    # e -> (atom e1)
    val = eval(e[1], a)
    return 't' if is_atom(val) else 'f'
```

Our Lisp does not have any boolean datatypes, so we simply return the atoms `t` or `f` depending on whether `e` is an atom or not.

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
def eq(e, a):
    # e -> (eq e1 e2)
    v1, v2 = eval(e[1], a), eval(e[2], a)
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


The `car` and `cdr` forms both evaluate the argument, expecting the resulting value to be a list. `car` then returns the first item of this list, while `cdr` returns the rest of the list.

```python
def car(e, a):
    # e -> (car e1)
    return eval(e[1], a)[0]

def cdr(e, a):
    # e -> (cdr e1)
    lst = eval(e[1], a)
    return 'nil' if len(lst) == 1 else lst[1:]
```

Notice that if the list contains only one element, `cdr` return the atom `nil` which in Lisp represents the empty list.

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
def cons(e, a):
    # e -> (cons e1 e2)
    rest = eval(e[2], a)
    if rest == 'nil':
        rest = []
    return [eval(e[1], a)] + rest
```

Notice that once again, we treat `nil` as the empty list.

```python
>>> interpret("(cons 'a '(b c))")
'(a b c)'
>>> interpret("(cons 'a 'nil)")
'(a)'
```

### Evaluating `cond`

The expression passed as the argument to `cond` is a list of lists, where each of the sub-lists is expected to contain two elements.

```python
def cond(e, a):
    # e -> (cond ((p1 e1) (p2 e2) ...))
    for p, e in e[1:]:
        if eval(p, a) == 't':
            return eval(e, a)
```

We evaluate the first element of each of the sublists in turn, until one evaluates to `t`. When the first `t` is found, the second element of that list is evaluated and returned.

```python
>>> program = """
...   (cond ((eq 'a 'b) 'first)
...         ((atom 'a) 'second))
... """
>>> interpret(program)
'second'
```

### Evaluating `defun`

As noted above, this form isn't one of those specified by McCarthy, but we include it anyway to make the language easier to use. Evaluating a `defun` expression simply extends the environment where it is called with a `label` structure containing a `lambda`. You will see how these are evaluated below.

```python
def defun(e, a):
    # e -> (defun my-fun (a1 ...) body)
    name, params, body = e[1], e[2], e[3]
    label = ["label", name, ["lambda", params, body]]
    a.insert(0, (name, label))
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

To round of the case when the first element in the evaluated list `e` is an atom, we simply look up this atom up in the environment, expecting to find a function. A new list with this function as the first element is then evaluated.

```python
def call_named_fn(e, a):
    # e -> (my-fun e1 ...)
    fn = lookup(e[0], a)
    return eval([fn] + e[1:], a)
```

Lets try to call the `pair` function we defined with `defun` above.

```python
>>> env
[('pair', ['label', 'pair', ['lambda', ['x', 'y'], ['cons', 'x', ['cons', 'y', ['quote', 'nil']]]]])]
>>> interpret("(pair 'a 'b)", env)
'(a b)'
```

### Evaluating `lambda` application

In the above example, `foo` is looked up in the environment and a new s-expression is evaluated. This new expression holds a function rather than an atom as the first element. Thus, the first element looks something like `(lambda (list of parameters) body)`. The rest of the elements in `e` are the arguments to the function.

```python
def apply(e, a):
    # e -> ((lambda (a1 ...) body) e1 ...)
    fn, args = e[0], e[1:]
    _, params, body = fn
    evaluated_args = map(lambda e: eval(e, a), args)
    a = zip(params, evaluated_args) + a
    return eval(body, a)
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

The `lambda` functions above is enough to define normal non-recursive functions. This could have been dealt with by using the [Y-combinator][y-combinator], but McCarthy introduces the `label` notation instead.

This evaluation case considers expressions on the form `((label name lambda-expression) arguments) the arguments)`.

```python
def label(e, a):
    # e -> ((label name (lambda (p1 ...) body)) arg1 ...)
    _, f, fn = e[0]
    args = e[1:]
    return eval([fn] + args, [(f, e[0])] + a)
```

We handle this by extending the environment `name` pointing to the first element of `e`, the `label`-expression. The `lambda` function is then applied to the rest of the elements of `e`, the arguments, and the value returned.

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

## Implementing the rest in Lisp

> Implemeting a few functions in our new language

```common-lisp
(defun null (x)
    (eq x 'nil))

(defun and (x y)
    (cond (x (cond (y 't) ('t 'f)))
          ('t 'f)))

(defun not (x)
    (cond (x 'f)
          ('t 't)))

(defun append (x y)
    (cond ((null x) y)
          ('t (cons (car x) (append (cdr x) y)))))

(defun list (x y)
  (cons x (cons y 'nil)))

(defun zip (x y)
  (cond ((and (null x) (null y)) 'nil)
        ((and (not (atom x)) (not (atom y)))
         (cons (list (car x) (car y))
               (zip (cdr x) (cdr y))))))
```

### Implementing the language within itself

These functions are all nice and well, but one of the nice things about Lisp, as we all know is that you are supposed to be able to construct datastructures (lists) that represent valid Lisp code, and evaluate them within the language itself. Our Lisp cannot do that yet!

Well, we can in fact add this feature with nothing but the Lisp itself. Lets first define a few helper functions.

We start by writing a few shorthand functions for combinations of `car` and `cdr`.

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
(defun assoc (x y)
  (cond ((eq (caar y) x) (cadar y))
        ('t (assoc x (cdr y)))))
```

With this, we are ready to implement the `eval` function:

```common-lisp
(defun eval (e a)
  (cond
    ((atom e) (assoc e a))
    ((atom (car e))
     (cond
       ((eq (car e) 'quote) (cadr e))
       ((eq (car e) 'atom)  (atom   (eval (cadr e) a)))
       ((eq (car e) 'eq)    (eq     (eval (cadr e) a)
                                    (eval (caddr e) a)))
       ((eq (car e) 'car)   (car    (eval (cadr e) a)))
       ((eq (car e) 'cdr)   (cdr    (eval (cadr e) a)))
       ((eq (car e) 'cons)  (cons   (eval (cadr e) a)
                                    (eval (caddr e) a)))
       ((eq (car e) 'cond)  (evcon (cdr e) a))
       ('t (eval (cons (assoc (car e) a)
                        (cdr e))
                  a))))
    ((eq (caar e) 'label)
     (eval (cons (caddar e) (cdr e))
            (cons (list (cadar e) (car e)) a)))
    ((eq (caar e) 'lambda)
     (eval (caddar e)
            (append (zip (cadar e) (evlis (cdr e) a))
                     a)))))

(defun evcon (c a)
  (cond ((eval (caar c) a)
         (eval (cadar c) a))
        ('t (evcon (cdr c) a))))

(defun evlis (m a)
  (cond ((null m) 'nil)
        ('t (cons (eval  (car m) a)
                  (evlis (cdr m) a)))))
```

As you see, the Lisp version of `eval` is very similar to the one we implemented in Python.

> ...

## Summary

The final result is [available on github][gh].

Just like the original this is a small, although neat, language with a lot features we expect in todays languages missing. It has no side effects (no IO), no types other than atoms (e.g. no numbers, strings, etc), there's no error handling, and it has dynamic rather than lexical scoping. The behaviour is also undefined for incorrect programs, and as an effect of this the error messages (from Python) can be rather strange and uninformative at times.

Still, it's remarkable how simple the implementation is, and

[rec-func]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.111.8833&rep=rep1&type=pdf
[roots]: http://www.paulgraham.com/rootsoflisp.html
[ast]: http://en.wikipedia.org/wiki/Abstract_syntax_tree
[s-exps]: http://en.wikipedia.org/wiki/S-expression
[quoting]: http://en.wikipedia.org/wiki/Lisp_(programming_language)#Self-evaluating_forms_and_quoting
[y-combinator]: http://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator
[gh-root]: https://github.com/kvalle/root-lisp
[gh-parser]: https://github.com/kvalle/root-lisp/blob/master/rootlisp/parser.py
[gh-tests]: https://github.com/kvalle/root-lisp/tree/master/tests

**Footnotes**

1. <a id="footnote-1"></a>Of course, implementing this you would not only rely on poking around in the REPL, but write [unit tests][gh-tests] beforehand or as you go.