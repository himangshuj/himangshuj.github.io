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
* How to infrastructure ?

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
This can get very religious. I have seen a lot of debates from seemingly smart people that the language they like is the best
language. It is partially true that the language the main architect is most proficient with is the best language provided it
is reasonably popular and stable. By reasonably popular, I mean how many projects and libraries exists in github. How many questions exist
in stack overflow. It is tempting to sometime go by marketing jargon of a every fancy new language and it is very important to 
resist the temptation. A few parameters I find useful to evaluate a framework are

* Does it support out of box standard authorization and authentication
* Does it generate crud apis(how easy is it to change data-model)
* Does it handle form validations
* Is it easy to hire
* Is it scalable
* Re-usable with existing code
* Project stability
* Ease of Debug/support

### Which database to use ?

It is best to stick to sql based databases. Given that a start up suffers from so many uncertainties, It is imperative to check
how thinks are doing. Moreover, you need to be able to check for any data corruption or failed code. You also want to do analytics
and error detection without using too much resources. SQL is far more evolved than any other database and we have standard open source
solutions like redash abnd metabase which can be used off the shelf to do error correction and analytics. 

### How to host and deploy ?
Refer [[https://twitter.com/mohapatrahemant/status/1102401615263223809]] , infrastructure is incredibly hard and is hostage to physical failures. You
need to use standard solutions for deploying and hosting. Besides it is possible to make mistakes during deployment . I prefer to use cloudformation
on aws backed by custom amis to do deployment. These are stable , mainstream and less error prone. I also have nothing against app engine/ digital ocean . However,
I only know one of them and like the question on `Which programming language?`, even in infra it is best to stick to known devil.





## Inspiration

* https://blog.bradfieldcs.com/you-are-not-google-84912cf44afb
* https://medium.com/@goldybenedict/single-page-applications-vs-multiple-page-applications-do-you-really-need-an-spa-cf60825232a3
* https://www.dwmkerr.com/the-death-of-microservice-madness-in-2018/
* https://twitter.com/mohapatrahemant/status/1102401615263223809?lang=en

         