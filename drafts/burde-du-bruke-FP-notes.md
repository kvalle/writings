---
title: Should _your_ team use functional programming?
date: null
description: ""
---

Functional programming (FP) is all the rage these days, with new languages, frameworks and libraries popping up all the time. You've probably seen both blog posts and conference talks proclaiming the death of OOP, and that functional is the new way.

The interest in FP is strong in BEKK. We have a special interest group dedicated to FP, a separate one for the functional language Elm, and [we use FP a fair bit in our projects](https://blogg.bekk.no/hvor-funksjonelle-er-vi-7414246ca1ae) (_link in norwegian_). And as practice lead for functional programming in BEKK, I'm obviously a proponent of FP. I believe there are many benefits to taking a functional approach, and have helped introduce FP at the client I'm currently working with.

But is FP the right answer for everyone? Should _your_ team be making the switch to functional programming?

**The answer is, of course, it depends!**

All languages and programming paradigms have their strengths and weaknesses. You need to beware of these to make the right choice. There are good arguments both for and against FP, and you need to find out which apply to your situation.

### Consider the problem

The first consideration is naturally if FP is the right tool for your particular job.

There are some situations where it is very hard to argue for FP. Say you are building something with hard realtime requirements, working on a low level close to the hardware. You're going to use a low level language like C or maybe even assembly, and probably won't even be reading a post like this.  

Or maybe you know you'll be doing something where you'll have to be using some particular library or other code integration. Say you have to be using [D3](https://d3js.org/) extensively. Then you might be best off by sticking to JS altogether.

	Ritkig språk for problemet
		Trenger å lage fancy grafer med D3? JAvascript, eller noe med tett og enkel JS interop er veien å gå?
		Taper du millionger, eller blir hengt ut i avisa, hvis frontenden din tryner? Velg noe med gode kjøretidsgarantier.
		Noe som kjører både på server og i klienten? Ikke Elm
		Utvikling av mobil-app? JavaScript, eller kanskje Kotlin/Swift?

### Consider the people

	Riktig språk for teamet man jobber i
		Hvis ingen, eller bare noen få, i teamet er begeistret for å lære noe funksjonelt kommer det ikke til å fungere.

	Noen ganger veier det ene mye tyngre enn det andre
	Hvilket hensyn er viktigst der du er?

### False concerns

Hard to find people. Not really.

		Vanskelig å finne folk:
		Kommer an på. Men flere har sett at det er _enklere_ å finne folk etter at de tok i bruk Elm

FP is abstract and hard, and requires tons of math. Not true.

		Funksjonell programmering er abstrakt og vanskelig, og krever mye matte
		Usant. Men mange vil nok ønske å lære mer om sånt når de kommer inn i denne typen språk.
		Elm er et eksempel på et språk/community som aktivt unngår de "vanskelige" uttrykkene, med fokus på å gjøre ting enkelt.

### Hybrid FP

Even if you find yourself "stuck" with an non-functional language, all is not lost.

### So, should you learn FP?

Yes. Regardless of whether you find FP to be a good fit for your particular problem and your people, I believe you should spend the time and effort to learn to use the functional paradigm.

It will [make you a better programmer](https://blogg.bekk.no/learn-new-languages-4f90456a287c) to learn a new way of thinking, regardless of whether you use it in your day-to-day work. And it will make you better able to determine if and when your project could benefit from the change.
