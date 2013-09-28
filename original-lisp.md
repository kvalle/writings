# Implementing (the original) Lisp in Python

As programmers, we have a lot of fun learning new languages and techniques.
...
But sometimes it's good to go back to the source, to see where it all comes from and have look at the fundemental ideas.

Today I'll take a look Lisp, as originally described by John McCarthy in his 1960 paper [Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part 1][rec-func]. Despite the ideas being expressed using mathematical notation, the paper is actually highly readable, and I encourage you to have a look. 

Another great source for insights into the original Lisp ideas is the article [The Roots of Lisp][roots] by Paul Graham, where he explains McCarthy's core idea, and why it's so powerful. 

There is really nothing novel in this blogpost. The ideas all belonged to McCarthy, but sometimes it's good to study the masters. Lets explore the origins of one of our most powerful programming languages.

We'll start by briefly covering the Lisp syntax and parsing, before moving on to the implementation of the language itself.

## Lisp 101

Lets start out briefly explaining the syntax and core semantics of Lisp. If you're already familiar with this, please skip down to the next section.

### A wee bit of syntax

Lisp programs consists of forms called [s-expressions][s-exps], which stands for *symbolic expressions*. S-expressions are defined recursively, and consists of either a single *atom* or a list of s-expressions.

An *atom* is akin to what many modern languages would call an identifier. It consists of a series of letters or symbols — anything other than parens, single quote and whitespace, basically. (*Note: we simplify a bit here, compared to McCarthys original description, by limiting atoms to not contain spaces, thus eliminating the need for commas separating atoms in lists. Also, we allow lower case letters…*)

Lists use the iconic parentheses that some people love to hate. A list is simply a pair of parens enclosing a number of other s-expressions, which might be either atoms or other lists.

Here are some examples of valid s-expressions:

```text
foo
another-atom
(list of only atoms)
(list (with some ((nested)) (lists inside)))
```

In addition, although it's not in the original Lisp, we include a shorthand syntax `'x` for expressing `(quote x)`, where `x` is a s-expression. This way, `'(a b c)` is interpreted as`(quote a b c)` and similarly, `'foo` as `(quote foo)`.

### The basic semantics

When evaluating an s-expression `e`, the following rules apply.

If the expression is an atom its value is looked up in the environment.
Otherwise, the expression is a list like `(e0 e1 ... en)`, which is evaluated as function application. How this is handled depends on the first element of the list, `e0`.

* If `e0` is one of the builtin (axiomatic) functions,  its evaluated is described below.
* If `e0` is another atom, a new list `e'` with the value of `e0` replaced by its valued looked up in the environment is constructed, and this list is evaluated.
* If `e0` is of the form `(lambda (<a1> ... <an>) <body>))`, then `e1` through `en` is first evaluated. Then `body` is evaluated in an environment where each of `a1` through `an` points to the value of the corresponding `en`.
* If the expression is of form `(label <name> <lambda>)` where `lambda` is a lambda expression like the one above, then a new list with `e0` replaced by just the `lambda` is constructed. This list is then evaluated in an environment where `name` points to `e`.

### The axiomatic functions

**`(quote e)`** returns `e` without evaluating it first.

**`(atom e)`** evaluates `e` and returns the atom `t` if the resulting value is an atom, otherwise `f` is returned.

**`(eq e1 e2)`** evaluates to `t` if both `e1` and `e2` both evaluates to the same atom or both empty lists, otherwise `f`.

**`(car e)`** evaluates `e` (which should give a list), and returns the first element of this list.

**`(cdr e)`** is quite the opposite of `car`, returns all but the furst element of the list gotten by evaluating `e`.

**`(cons e1 e2)`** evaluates both `e1` and `e2`, and returns a list constructed with `e1` as the first element and `e2` as the rest.

**`(cond ((p1 e1) ... (pn en)))`** is the *conditional* operator. It will evaluate predicates `p1` to `pn` in order, until one of them evaluates to `t`, at which time it will evaluate the corresponding `en` and return its value.

`(defun name e)` is 

## The parser

The first part of creating any language is usually the parser. We need some way to go from programs as strings to some datastructure we can interpret. Such a datastructure is usually called the [Abstract Syntax Tree][ast] (AST) of the program.

Since Lisp is a language largely without syntax, with parentheses and atoms used for everything, writing the parser is relatively easy and uninteresting.
I've written [a simple parser][gh-parser] which we'll be using, which works like this:

```python
>>> from rootlisp.parser import parse
>>> program = "(lambda (x) (cons x (cons x '())))"
>>> parse(program)
['lambda', ['x'], ['cons', 'x', ['cons', 'x', ['quote', []]]]]
>>> 
```

The `parse` function takes one argument, the program string, and returns the corresponding AST.

## The Evaluator

The other major part of the implementation is the `eval` function. The `eval` function looks like this:

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

The `eval` function takes two arguments, an expression `e` formed as one of the ASTs returned by `parse`, and a list of associations `a` which acts as the environment binding atoms to values. These are all the cases we need to cover to implement Lisp. 

Before we move on, let's look at another useful function, `interpret`, which we'll be using to test our language:

```python
def interpret(program, env=None):
    ast = parse(program)
    result = eval(ast, env if env is not None else [])
    return unparse(result)
```

This function combines the power of `parse` and `eval`. `interpret` takes a Lisp program as a string, and parse and evaluate it. Since the result might be a Lisp data structure (or even valid Lisp code), we "unparse" it back to it's corresponding source string. (This is part of the parser, and not very interesting, but feel fre to [have a look][gh-parser] it you like.)

Now, lets look at each in turn.

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

Lets test this:

```python
>>> from rootlisp.lisp import interpret
>>> 
>>> env = [('foo', 'bar')]
>>> interpret('foo', env)
'bar'
```

### Evaluating `quote`

The next branch of the `eval` function is perhaps even simpler than the first one.
since atoms are handled, we now know we are working with a list expression.

If the first element in the list is the atom `quote`, we simply return the next element of the list, skipping evaluation.

```python
def quote(e):
    return e[1]
```

And it works like expected:

```python
>>> interpret('(quote a)')
'a'
>>> interpret("'a")
'a'
>>> interpret("'(a (b (c) d))")
'(a (b (c) d))'
```

### Evaluating `atom`

```python
def atom(e, a):
    a = eval(e[1], a)
    return 't' if a == [] or is_atom(a) else 'f'
```

### Evaluating `eq`

```python
def eq(e, a):
    a, b = eval(e[1], a), eval(e[2], a)
    if a == b and (is_atom(a) or a == []):
        return 't'
    else:
        return 'f'
```

### Evaluating `car` and `cdr`

```python
def car(e, a):
    return eval(e[1], a)[0]

def cdr(e, a):
    return eval(e[1], a)[1:]
```

### Evaluating `cons`

```python
def cons(e, a):
    return [eval(e[1], a)] + eval(e[2], a)
```

### Evaluating `cond`

```python
def cond(e, a):
    for p, e in e[1:]:
        if eval(p, a) == 't':
            return eval(e, a)
```

### Evaluating `defun`

```python
def defun(e, a):
    name, params, body = e[1], e[2], e[3]
    label = ["label", name, ["lambda", params, body]]
    a.insert(0, (name, label))
    return name
```

### Evaluating function calls

```python
def call_named_fn(e, a):
    fn = lookup(e[0], a)
    return eval([fn] + e[1:], a)
```

### Evaluating `lambda` application

```python
def apply(e, a):
    fn, args = e[0], e[1:]
    _, params, body = fn
    evaluated_args = map(lambda e: eval(e, a), args)
    a = zip(params, evaluated_args) + a
    return eval(body, a)
```

### Evaluating `label` application

```python
def label(e, a):
    _, f, fn = e[0]
    args = e[1:]
    return eval([fn] + args, [(f, e[0])] + a)
```


## Summary

The final result is [available on github][gh].

Just like the original this is a small, although neat, language with a lot features we expect in todays languages missing. It has no side effects (no IO), no types other than atoms (e.g. no numbers, strings, etc), there's no error handling, and it has dynamic rather than lexical scoping.

Still, it's remarkable how simple the implementation is, and

[rec-func]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.111.8833&rep=rep1&type=pdf
[roots]: http://www.paulgraham.com/rootsoflisp.html
[ast]: http://en.wikipedia.org/wiki/Abstract_syntax_tree
[s-exps]: http://en.wikipedia.org/wiki/S-expression
[quoting]: http://en.wikipedia.org/wiki/Lisp_(programming_language)#Self-evaluating_forms_and_quoting
[gh-root]: https://github.com/kvalle/root-lisp
[gh-parser]: https://github.com/kvalle/root-lisp/blob/master/rootlisp/parser.py
[gh-tests]: https://github.com/kvalle/root-lisp/tree/master/tests
