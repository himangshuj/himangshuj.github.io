---
layout: post
title: How I think early stage startup should build product
image: confusion.jpg
date:  2019-04-20 12:00:00
---
## Why I have written this
I have spent better part of my professional career in some startup or other as one of
the earliest engineers. This has meant , I was thrust into the frying pan far too often
and had to figure my way out. In architecture everyone has an opinion. 

* Micro service vs monolith
* Which programming language  and framework to use ?
* Which database to use ?
* How to host ?
* How to deploy ?

All the above the age old Shakespearean dilemma of "whats in a name?" can take a lot of time
not spent coding. While I have no answer of what is the best name for a repo or a variable or function,
I have found my individual answers to a few architectural questions. 

## Constraints of a startup

* Less Time and Resource
* Lots of uncertainties 
* Lots of assumptions to validate. This is not same as lot of code to write. Wherever possible
we should not write code and use off the shelf solution.
* Crude nature of assumptions. Assumptions or hypothesis should be crude. If the
assumption requires you to have the fastest code that can be written or the most
polished possible website, the assumption cannot be the basis of a startup. In the 
initial phases of the startup we are all building the fundamental basic blocks, the core
problems addressed at this stage should be solvable using a decent product and should not
require the best ux or latency.

## My Gospel

### Monolith vs Micro service
<p>Wherever possible , you should not use microservice. Microservices significantly
increases the code complexity without any tangible benefits at early stages. A start up
should not need asymmetric scaling. The benefits of zero downtime is also over-exaggerated in context of a startup.
Having said that there is a definite use of microservice. I find them very useful when integrating with third party services. 
They insulate the core code-base from libraries brought in by third parties like SOAP dependencies etc.</p>

### Which programming language  and framework to use ?

## Inspiration

* https://blog.bradfieldcs.com/you-are-not-google-84912cf44afb
* https://medium.com/@goldybenedict/single-page-applications-vs-multiple-page-applications-do-you-really-need-an-spa-cf60825232a3
* https://www.dwmkerr.com/the-death-of-microservice-madness-in-2018/
* https://twitter.com/mohapatrahemant/status/1102401615263223809?lang=en

         