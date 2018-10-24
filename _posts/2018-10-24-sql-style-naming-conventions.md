---
layout: post
comments: true
title: SQL Style for Data Scientists - Naming Conventions
excerpt: Here are some general rules that I try to follow when naming objects
 in a SQL database. There will always be exceptions, but you might want to 
 keep  these in mind when you're working in your own SQL environment.
image: images/sql-style-guidelines-general-rules.png
date: 2018-10-24
author: pjryan126
category: techblog
tags: [sql, coding, style]
---

This is Part Three of a four-part series on SQL coding style. 

Part One was a brief **[introduction](https://pjryan126.github
.io/sql-style-intro/)** to what I'm trying to accomplish with these 
articles. 

Part Two provided some **[general rules](https://pryan126.github
.io/sql-style-general-rules) that I try to follow when coding in a SQL 
environment. 

In Part Three below, I will discuss naming conventions that I try to 
follow when naming objects in a SQL environment. 

As with all of the rules in this series of articles, The rules below are 
derived from Joe Celko's **[SQL Programming Style](https://www.amazon.com/Celkos-Programming-Kaufmann-Management-Systems/dp/0120887975/ref=sr\_1\_1?ie=UTF8&qid=1516645111&sr=8-1&keywords=Joe+Celko+SQL+Programming+Style)** 
and also appear in Simon Holywell's 
**[SQL Style Guide](http://www.sqlstyle.guide/)**. There will always be 
exceptions, but you might want to keep these rules in mind when you're 
developing your own SQL scripts.

## General Rules {#general-rules}

### Names Should Follow the SCUD Principle {#scud}

When naming objects in SQL, there are four competing factors that you should always consider. In general, your names should be:

* **S**hort
* **C**onsistent
* **U**nique
* **D**escriptive

Sometimes, these factors are at odds with each other. For example, naming a column `id` because it contains the id value for each widget in your `widgets` table might offer an extremely short name, but if you named every id in your database `id`, those columns names wouldn't be unique, and they wouldn't be very descriptive of what the column data actually represent.

A better name for the column would be `widget_id`. It might not be as short as `id`, but it's short enough, and it's certainly more descriptive and unique. And if you used same pattern for all id columns in your database (*i.e.*, `<tablename>_id`), you will add a strong degree of consistency to your naming convention. 

### Avoid Special Characters and White Space {#special-characters}

Limit your names to letters, numbers and underscores. Special characters (*e.g.*, "@" and "#") can have clearly defined use cases in certain SQL products, so using them to name your objects can lead to unintended consequences and unnecessarily limit the portability of your code. White space might make names easier to read, but you would have to surround your names with delimiters whose level of support varies across SQL products. 

Furthermore, be sure to start your names with a letter and end them with either a letter or a number. All SQL products require the first character of names to be a letter, and some forbid the use of an underscore at the end of names. 

----

**Exceptions:** 

Some SQL products require a special character at the start of certain types of objects. For example, scalar variables in SQL Server must start with a "@".

Additionally, you will often have to generate results in a format that is readable by non-technical business professionals. In that case, it might be acceptable to use delimited alias column names (*e.g.* `"Country Name"` instead of `country_name`) for your query output. Even then, however, you should try to use double-quotes as your delimiters, because they are the most widely supported across SQL products.

----

### Separate Words with Underscores {#use-underscores}
If your names consist of multiple words, separate them with underscores. It will greatly improve readability.

```
CREATE TABLE Client_Accounts (
       account_nbr   varchar(10)  PRIMARY KEY,
       account_type  varchar(64)  NOT NULL,
       client_name   varchar(128) NOT NULL
);
```

### Treat Names as Being Case-Insensitive {#case-insensitive}

SQL products apply case-sensitivity rules in a variety of conflicting ways. You can generally avoid these conflicts -- while adhering to the [SCUD Principle](#scud) -- by assuming that your SQL product is case-insensitive (*i.e.*, `Country` is equivalent to `country`).

### Avoid Prefixes {#avoid-prefixes}

Prefixes will only make your code harder to read. In most cases, you don't need to know whether an object is a table or a view, and the time and effort that you save by keeping your code readable will more than make up 
for any time it will take for you to look up a few object types. 

You might find exceptions in situations where there are no better means of distinguishing between objects. For example, if you are writing a query that takes a table of source data and returns a new table with some original and some transformed colums, you might need a way to distinguish between the source and transformed columns.  In that case, a prefix might be a quick and dirty way to do it, but even then, you should still ask yourself if there is a more elegant solution to solving your problem. 

----

## Tables (and Other Schema Objects) {#tables}

### Capitalize Names {#capitalize-names}

Capitalizing table names provides a fast and easy method of distinguishing tables from attributes when scanning your code, and it reinforces their place over attributes in the hierarchical structure of your database. 

### Use Standard, Collective or Plural Nouns {#scp-nouns}

If a table stores industry standard information, use the industry standard name. For example, if you are storing a table of values from the NAICS database, call the table `NAICS`.

If the data is unrelated to an industry standard, use a collective noun whenever possible. If a collective noun is not a reasonable option, name the table with a plural noun. Tables are collective sets of information, not individual entities. Naming them with collective nouns forces the user to consider that fact when they are interacting with the tables. 

As an example, both `Staff` and `Employees` are both acceptable table names, but `Staff` should be preferred over `Employees.` In contrast, `Team` and `Players` are also both acceptable table names, but `Players` would be preferrable over `Team` if you are storing information for more than one team in your database. 

----

**Exceptions:** 

If you have a table whose purpose is to store a single row of data, you should use a singular noun to name that table.

----

### Derive Aliases from Table Names {#derive-aliases}

When aliasing a table name in a SQL query, use an alias that relates in some way to its corresponding object. If a table is named `Staff`, for example, use `S` as the alias, not `A`.

```
SELECT 
       S.name AS employee,
       D.name AS department
  FROM Staff AS S
  LEFT OUTER JOIN Department AS D
    ON S.department_id = D.department_id;
```

### Try to use Descriptive Names for Relationship Tables {#rel-tables}

In general, it is better to give relationship tables descriptive names. For example, if you are modeling student enrollment at a local college, you should name the table joining students to courses something like `Enrollment`, as opposed to `StudentsCourses`. 

----

**Exceptions:** 

There will be times when a descriptive term for the relationship simply doesn't exist, in which case, concatenating joined table names might be your best option. 

----

## Columns {#columns}

### Use Singular Nouns {#singular-nouns}

Columns store scalar values, and you should therefore reference them using singular nouns.

### Use Lowercase Letters {#use-lowercase}

Users naturally conceptualize schema objects (e.g., schemas, tables, views and stored procedures) as something akin to proper nouns, so it is best to capitalize their names. But lowercase letters are easier to read, so it's a good idea to maximize their use when naming scalars such as columns. 

### Use Standardized Postfixes {#postfixes}

Define a standard set of postfixes for your names and run with them. In general, using full words whenever possible is more descriptive and therefore preferred, but in some cases, utilizing abbreviated postfixes can make writing code faster and reading your code easier. Below is a non-exhaustive set of postfixes that uses full words and some abbreviations.

|Postfix   | Description |
|--------- | ----- |
| _addr    | address |
| _code    | a standard code from a trusted source (e.g., `zip_code`) |
| _date    | date or datetime |
| _desc    | a description of something |
| _id      | identifier |
| _img     | an image or a URI to an image |
| _name    | self-explanatory |
| _nbr     | number |
| _status  | a state of being, often nominal in value (e.g., 'shipped', 'delivered') |
| _type    | something that has a common meaning (e.g., 'transaction_type' |

----

## Stored Procedures and Table-Valued Functions {#sp-tvf}

### Use Verbs {#use-verbs}

Generally speaking, stored procedures and table-valued functions perform actions. It is therefore more descriptive to use active verbs in their names whenever possible. 
