---
layout: post
title: Programming Language For Startups
image: mexican-standoff.jpg
date:  2020-10-03 10:00:00
description: "A correct choice of programming language and stack is not 
 sufficient to ensure a successful startup, but a wrong choice is a significant 
 impediment. Over the years, I have had the opportunity to architect several products with both success and failure.
 After every failure, I wonder what I did wrong and after every success I exclaim in disbelief. As a software engineer,
 I like to find patterns across data, and I am taking my judgemental eyes across a few websites I admire"  
---
Over the last decade and a half, I have had the opportunity to architect many products from scratch. As the mast of my blog will say, I have tasted my share of victories and defeats. In all these endeavors, I have never stopped loving programming and wanting to be a better programmer. Quarantine is 2020, and I wanted to use my free time dissect architectures of websites that inspire than to look at my follies. 

Today, centering a div and choosing a programming language are some of the most challenging computer science problems. Like all faith matters, it has led to many battles and fights between well-meaning and smart people. In the past few years, numerous studies on the relative quality of programming languages. PYPL, TIOBE, Stack Overflow regularly comes up with a score for programming languages. All these metrics have their merits, but I wanted to do a more qualitative study of programming languages. 

In the thirty years since Time Bernard Lee created WorldWideWeb,
 many companies operating on the web have made it to the list of most valuable companies.
  In my research, I found many articles on companies'
   current architecture and how they solved significant scaling problems. 
   I was more interested in the early days of these companies. 
    I chose fourteen companies that inspired me.  Collecting data is not enough. 
    I wanted to find patterns.  
    As Leo Tolstoy once said, *"All successful websites are alike;all
     unsuccessful websites are unsuccessful in their own way."*
     
| Product | Year Made | Primary Language | Language/Framework Founded | Hypothesis on why did they choose it ? |
| ------- | ------- | ------- | ------- | ------- |
| MS DOS | 1982 | 8086 Assembly | 1972 | Microsoft had to deliver a new operating system for IBM in limited time. Instead of building from scratch , they modified the 8 bit CP/M for 16 bit PCs.  |   
| Yahoo Directory | 1994 | Html | 1991 | No definitive article on the initial architecture of Yahoo but based on architecture of related WWW Virtual Library, it can be guessed it was pure static pre generated html.  |
| Amazon.com | July 1995 | Perl | 1987 | At that time, websites were text heavy and perl was possibly the easiest way to manipulate text |
| Google.com | July 1996 | Python | 1990 | In 1996, python was probably the easiest way to crawl the web as compared to any other language. At it's heart, google is a crawler |
| Youtube.com | 2005 | Php | 1995 | By 2005, Php was the reigning champion for web programming.  |
| Facebook.com | 2003 | Php | 1995 | In 2003, Php was the reigning champion for web programming.  |
| LinkedIn.com | 2002 | Java Servlet | 2002 | Around 2002-2003, Sun was spending a record 500 m $ in marketing java. This created an ecosystem of performant backend.   |
| Docusign | 2003 | dot net | 2002 | Docusign started as docutouch in 2000. At that time, they were signing microsoft word. Hence, they continued leveraging their strengths in microsoft stack.   |
| Paypal | 2002 | dot net | 2002 | Paypals payment code started as client side application and migrating to dot net for the web world was easy.  |
| Shopify.com | 2006 | Ruby on Rails | 2004 | They built the entire website in two months time for their own store and then decided to create a platform. |
| Twitter.com | 2006 | Ruby on Rails | 2004 | The app was rapidly prototyped in a few days by a few friends at work and ROR was the fastest way to prototype in 2006  |
| Dropbox.com | 2008 | Python | 1990 | At its core, dropbox is a distributed file server. Dropbox founders liked python. They needed to work well both as desktop and web server and python fits the bill better than ruby or php at that time. |
| UberCab.com | 2009 | Python | 1990 | Around 2005 saw the release of Django one of the post popular web frameworks   |
| Stripe.com | 2010 | Ruby on Rails | 2010 | In Patrik's own words, there was little to choose between Ruby and Python . They knew ruby better . They also punted on solving scalability problems.  |

To sum up, I will borrow Patrick's answer as to why he chose ruby. "It's reasonably powerful, it has a large community, there are lots of actively-maintained libraries, and it's sufficiently widely deployed that you're unlikely to hit big, unexpected problems"  We can use the same justification for any language. 

## References 
* http://vlib.org/admin/history
* https://www.theregister.com/2003/06/09/sun_preps_500m_java_brand/
* http://emoney.allthingsd.com/20101213/e-commerce-assistant-shopify-raises-7-million-in-first-round/
* https://medium.com/@mittalyashu/why-did-twitter-switch-from-ruby-on-rails-dac66150044d
* http://www.internetnews.com/bus-news/article.php/411921/Online+Signatures+DocuTouch+Introduces+DocuSign.htm
* https://www.quora.com/What-programming-language-did-Elon-Musk-use-at-PayPal
* https://thehustle.co/proof-that-your-favorite-startup-started-out-awful
* https://www.quora.com/Why-did-Stripe-choose-to-use-Ruby-for-its-backend-language