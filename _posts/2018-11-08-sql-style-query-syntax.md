---
layout: post
comments: true
title: SQL Style for Data Scientists - Query Syntax
excerpt: Here are some general rules that I try to follow when writing queries
 in a SQL database. There will always be exceptions, but you might want to 
 keep  these in mind when you're working in your own SQL environment.
image: images/sql-style-guidelines-general-rules.png
date: 2018-11-08
author: pjryan126
category: techblog
tags: [sql, coding, style]
---

This is Part Four of a four-part series on SQL coding style. 

Part One was a brief **[introduction](https://pjryan126.github
.io/sql-style-intro/)** to what I'm trying to accomplish with these 
articles. 

Part Two provided some **[general rules](https://pryan126.github
.io/sql-style-general-rules)** that I try to follow when coding in a SQL 
environment. 

Part Three discussed some simple **[naming conventions](https://pjryan126.github.io/sql-style-naming-conventions)**
 that I try to follow when creating objects in a SQL environment. 

In part Four, I will provide a few basic rules to follow when writing your SQL 
queries. 

As with all of the rules in this series of articles, The rules below are 
derived from Joe Celko's **[SQL Programming Style](https://www.amazon.com/Celkos-Programming-Kaufmann-Management-Systems/dp/0120887975/ref=sr\_1\_1?ie=UTF8&qid=1516645111&sr=8-1&keywords=Joe+Celko+SQL+Programming+Style)** 
and also appear in Simon Holywell's 
**[SQL Style Guide](http://www.sqlstyle.guide/)**. There will always be 
exceptions, but you might want to keep these rules in mind when you're 
developing your own SQL scripts.

## Use Uppercase Letters for Reserved Words {#use-uppercase}

Separate reserved words from other words in your query by using uppercase 
letters. It makes it easier for a reader to identify the defined objects in 
your query.

```
SELECT
       U.user_id    AS user_id,
       U.user_name  AS username,
       U.email      AS email,
  FROM Auth.Users U
 WHERE U.user_name LIKE 'R%';
```

## Right-Align Root Reserved Words {#align-reserved}

Reserved words that start separate lines should be right-aligned with 
reserved words that start other lines. For example, "SELECT", "FROM", 
"WHERE", and "ORDER" in the `ORDER BY` clause in a `SELECT` query should all 
end at the same character boundary on their lines. This creates a **river** 
of whitespace that helps to improve readability, implicitly indicates a 
priority level among the clauses, and provides a visual method of ordering 
clauses and values. Columns and other object names should appear to the right
 of this river. 

For the same reason, all instances of `AS` in a `SELECT` clause should be 
aligned with each other. 

Finally, join clauses should start to the right of the river on a newline in 
the `FROM` clause. This helps the reader in understanding that columns are 
being selected from a relation of joined objects.

```
SELECT
       U.user_id    AS user_id,
       U.user_name  AS username,
       U.email      AS email,
       CASE
            WHEN P.is_admin = 1 THEN 'Administrator'
            ELSE 'User'
        END         AS 'Role'
  FROM Auth.Users   AS U
  LEFT OUTER JOIN Auth.Permissions  AS P
    ON U.user_id = P.user_id
 WHERE U.user_name LIKE 'R%'
   AND P.is_admin IS NOT NULL
 ORDER BY username;
```

## Separate Parts of your Queries with Newlines {#use-newlines}

The reader should consider each part of a query in turn. To assist the reader
 with this process, you should generally try to separate parts of your 
 queries with vertical space by adding newlines:

* before `AND` or `OR`
* after semicolons
* after each keyword definition
* after a comma when separating multiple items into a logical group
* to separate code into related sections  
  

```
INSERT INTO Addresses (address1, address2, city, state, zip)
VALUES ('123 Main St.', 'Apt. 4F', 'New York', 'NY', '10038')
       ('456 Elm St.', NULL, 'New York', 'NY', '10038');
``` 
```
UPDATE Addresses
   SET address2 = 'Apt A'
 WHERE address_id = 2;
```
```
SELECT A.address1,
       A.address2,
       A.city,
       A.state,
       A.zip
  FROM Addresses AS A
 WHERE A.city = 'New York';
```
```
SELECT 
       B.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM Champions AS C
         WHERE C.last_name = B.last_name
           AND C.is_confirmed = TRUE) AS last_championship_year
  FROM Boxers AS B
 WHERE B.last_name IN (
         SELECT C.last_name
           FROM Champions C
          WHERE YEAR(championship_date) > 2010
       );
 ```
               