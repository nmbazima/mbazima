---
layout: post
comments: true
title: SQL Style for Data Science - General Rules
excerpt: Below are some general rules that I try to follow when coding in a SQL environment. There will always be exceptions, but you might want to keep these in mind when you're developing your own SQL scripts.
image: images/sql-style-guidelines-general-rules.png
date: 2018-10-04
author: pjryan126
category: techblog
tags: [sql, coding, style]
---

Below are some general rules that I try to follow when coding in a SQL 
environment. There will always be exceptions, but you might want to keep these
in mind when you're developing your own SQL scripts.

----

* [Comment your code](#comment-code)
* [Use white space to improve readability](#white-space) 
* [Use standard SQL functions](#standard-functions)

----

## Comment Your Code {#comment-code}

Commenting your code should be the best part of your job. It is your 
opportunity to tell others in the clearest terms possible what your code should 
do. In fact, it is often helpful to write your comments first before you event 
start to write your query or function. It always helps to know where you're 
going before you start your journey.  

Generally speaking, you should try to use the C-style `/*` and `*/` marks to 
open and close your comments whenever possible. Doing so will allow you to write 
multi-line comments without worrying about adding a `--` mark to every single 
line. If you are adding a quick comment on the same line as your code, though, 
you can use a `--` mark and finish with a newline.

```
/* Update North Korea's country name */
UPDATE Tools.Countries
   SET country_name = 'Little Rocket Land' -- use the unofficial name
 WHERE country_id = 113;
```

Comment style is a matter of preference. The key is that you do it consistently 
and in as a consistent a fashion as possible.
   
## Use White Space to Improve Readability {#white-space}

Do not, under any circumstances, write code like you are writing a paragraph. 
Your code should be broken up into logical parts. Clauses, for example, should 
appear on separate lines of code.

Below is an example of a simple `SELECT` query that uses white space and 
indentation judiciously.

```
SELECT
       S.state_name   AS state,
       C.country_name AS country,
       C.iso3         AS country_code
  FROM Tools.States AS S
  LEFT OUTER JOIN Tools.Countries AS C
    ON S.country_id = C.country_id
 ORDER BY
       state,
       country;
```

## Use Standard SQL Functions {#standard-functions}

To maximize the portability of your code, you should do your best to use 
standard SQL functions whenever possible. Every database management system puts 
its own spin on SQL. In SQL Server, for example, you can replace `NULL` values 
in a query with a value of your choosing with the `ISNULL()` function. However, 
that function is not available anywhere outside SQL Server, so if you write a 
query in SQL Server using the `ISNULL()` function and then try to run that 
query on a MySQL database, you are going to get an error. A better choice would 
be the `COALESCE()` function, which is available in all database management 
systems. 

```
SELECT
       country_id,
       iso,
       country_name,
       COALESCE(iso3, 'Not Available')        AS iso3,
       COALESCE(number_code, 'Not Available') AS number_code,
       phone_code
  FROM Tools.Countries
 WHERE iso <> 'US';
```

You can find a useful summary of standard and dialect-specific functions 
**[here](https://en.wikibooks.org/wiki/SQL_Dialects_Reference)**. Many of the 
system-specific functions can be quite useful, so I wouldn't say their use 
should be expressly forbidden. If you are writing code that you think might be 
used on more than one type of database management system, though, you should do 
your best to avoid them. 

{% if page.comments %}

If you have any ideas on how to improve these guidelines, please feel free to leave a comment below. I'd love to hear about it.

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://https-pjryan126-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                            
{% endif %}







