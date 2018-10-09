---
layout: post
comments: true
title: SQL Style for Data Science
excerpt: Opinions on SQL style are freely available, but they often vary, and I sometimes find it hard to keep them all straight. For that reason, I have begun to compile a basic set of style guidelines that I hope to commit to memory and use when I'm coding in SQL. 
image: images/sql-style-guidelines.png
date: 2018-10-03
author: pjryan126
category: techblog
tags: [sql, coding, style]
---

## Introduction

I spend enough time in SQL environments to worry not only about how my code 
functions but also about how it looks. When it comes to personal projects, 
formatting and style often give way to my concerns over performance and my
rate of development, but when I'm coding in a collaborative environment or 
in situations where my code may be subject to some sort of review, the way 
that I write the code can matters just as much as what it actually does. 

Opinions on SQL style are freely available, but they often vary, and quite 
frankly, I sometimes find it hard to keep them straight. If I were a SQL admin 
working exclusively in a SQL environment, I might be more motivated to learn 
the rules of SQL style -- and the exceptions to those rules -- but as a data 
scientist, I already have too many rules (and exceptions) in my head as it is. 
For that reason, I have begun to compile a more manageable set of style guidelines 
that I can commit to memory and use when I'm coding in SQL. 

These guidelines won't teach you how to program in SQL. If you don't already know 
how to do that, I suggest that you at least learn the basics before worrying about 
your coding style. The purpose of these guidelines is to provide you with simple 
instructions for writing clean, readable, repeatable SQL code. They aim follow 
principles set forth in Joe Celko's **[SQL Programming Style](https://www.amazon.com/Celkos-Programming-Kaufmann-Management-Systems/dp/0120887975/ref=sr\_1\_1?ie=UTF8&qid=1516645111&sr=8-1&keywords=Joe+Celko+SQL+Programming+Style)**, 
but they may not follow them exactly. If you would like more detailed 
information on a particular topic, you will definitely want to take 
a look at what Celko has to say. 

The guidelines are also heavily inspired and informed by Simon Holywell's 
**[SQL Style Guide](http://www.sqlstyle.guide/)**. Holywell's guidelines are 
extremely well structured and also seek to apply the Celko standards, so if you 
disagree with any of my suggestions, you might want to consider taking
Holywell's advice instead. 

Finally, if you're feeling lazy and simply need help with style on a discrete
piece of code, take a look at **[poorsql.com](http://poorsql.com)**. It's a 
useful tool (as long as you're not worried about providing your code to an 
outside party for processing), and it can get you at least most of the way 
to clean, readable SQL code. 

I'll be pushing out my guidelines over the next few weeks. For now, please take
a look at some general style rules that I try my best to employ when coding in 
SQL **[here](https://pjryan126.github.io/sql-style-general-rules/)**. 