# 7 simple steps to implementing a programming language

> *It's easier than you think. Promise.*

I've always liked to play with programming and learning new concepts. 
There's a lot to be gained by learning a new language, especially from an unfamiliar paradigm. 
For me, however, some concepts only really first clicked when I started to make simple languages from scratch. 
I didn't fully understand closures until I had implemented them myself.

The problem with making a language is that if you don't know what you're doing, then there is *a lot* of trying and failing before you get anywhere. 
It was for me, anyway. 
I felt there was a need for something to help a prospective language maker on those first steps. 
Therefore I made [this simple tutorial](https://github.com/kvalle/diy-lisp).

The tutorial is a hans-on, test driven guide to implementing a language. 
The tests are all provided, and each step is explained. 
All you need to do is make each test pass in turn.

So what are the "seven steps" promised in the title? Well, I lied a little. It's actually "seven parts" which constitute the tutorial. Each consist of a set of tests guiding you to implement one aspect of the language. Start with the [README file](https://github.com/kvalle/diy-lisp/blob/master/README.md), and all is explained from there.

Here is a sneak peak of what the final language will look like:

```scheme
(define fact 
    ;; Factorial function
    (lambda (n) 
        (if (eq n 0) 
            1 ; Factorial of 0 is 1
            (* n (fact (- n 1))))))

(fact 5) ;; this should evaluate to 120
```

Depending on your level of familiarity with the concepts, the tutorial should take anywhere from a few hours to a day or more to complete. 
By the end, however, you will have implemented a language with proper lexical scoping, first class functions, and a simple Lisp-like syntax. 
More importantly, you'll be done making your first language done, and ready to continue exploring.

Have fun!

*Oh, by the way, if you do follow the tutorial and implement a language, I would love to hear your feedback.*
