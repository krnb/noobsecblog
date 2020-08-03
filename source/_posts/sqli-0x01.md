---
title: SQL Injection - 0x01
date: 2020-07-03 16:22:37
tags: [web attacks, sqli, sql injection]
---

# SQL Injection 0x01

<!-- 💉 Injections are pretty cool aren't they? 💉 -->

Hi, welcome to the first post of the SQL injection series. Before we dive into the "injection" part of it, let's first understand the basics of what SQL is and the usual structure of a database-driven system.

## Structured Query Language (SQL)

SQL is a database querying language which comes in various flavours like MySQL, MS-SQL, PostgreSQL, etc. In this series we'll mainly focus on MySQL

It's a tabular database system, like Microsoft Excel simply put, with rows and columns.

Database servers can have multiple databases with different user rights if required and within those custom tables are created to support the systems functionality.

An SQL database are usually used to support login forms, blogs, ecommerce websites, etc. It's not restricted to websites and can also be found in mobile applications.

## Structure Of A System
Usual structure of a database-driven system. Today n-tiered architecture are used in a dynamic system, 3-tier architecture being the simplest kind.

A three-tier architectural breakdown: 
Client (or Presentation) tier (Browser) - Renders HTML + JS
Logic tier (Code or Application Server) - PHP, ASPX, etc
Data tier (Database Server) - MySQL, MS-SQL, PostgreSQL, Oracle, etc.

Structure of a system visually:

![Structure of a system](/sqli-0x01/system_structure.png)

Client views the system via a browser which sends requests (usually just the parameters) as per the functionalities presented to the logic tier which sends the complete request (the entire query invisible to the user) to the database server.
The database server executes the query successfully, if valid, sends the results (or errors) to the logic tier. The logic tier performs any processing that's put by the developer on the results received and forwards the processed result to the end-user.

<!-- Structure explanation done -->

## Basics of SQL
We don't have to learn SQL for DBMS (DataBase Management System) purposes but we certainly need to understand how queries are built and functionality of different aspects of a query. Having this knowledge we can guess a bit better what the query is from the input and output.

### Structure of a `SELECT` statement
`SELECT` and `FROM` are required to form a SELECT query. Rest are optional.
``` SQL
# Used to retrieve rows from selected columns
SELECT <column_names/wildcard>

# Specifies the table to retrieve data from
FROM <table_name>

# Specifies the condition or logic as per which rows (data) from columns specified should be retrieved
[WHERE <condition> <operator> <condition>]

# Concatenates two SELECT queries. Number of rows fetched by both the queries should be same.
[UNION <SELECT>]

# Group or aggregate the results by a column name or position
[GROUP BY <column number/name>]

# This is the same as putting a where condition
[HAVING <condition>]

# Alter the results by the column name or position
[ORDER BY <column number/name>]

# Number of rows to display in the output
[LIMIT <offset>,<number of rows>]
```

Example of a `SELECT` query:

``` SQL
# All statements end with a semi-colon -> ;
SELECT CustomerName FROM customers;
```

### SQL Operands

We'll mostly be only working with `OR` and sometimes `AND` operators. Other operators that exist are `NOT` and `XOR`, which are not so important to us.

**`OR` logic table**:
Condition column - statement1_result `OR` statement2_result

| Condition | Result |
| --------- | ------ |
| *true* `OR` *true* | True ✔ |
| *true* `OR` *false* | True ✔ |
| *false* `OR` *true* | True ✔ |
| *false* `OR` *false* | False ❌ |

If either of the statements is true, the result will be true.

**`AND` logic table**:
Condition column - statement1_result `AND` statement2_result

| Condition | Result |
| --------- | ------ |
| *true* `AND` *true* | True ✔ |
| *true* `AND` *false* | False ❌ |
| *false* `AND` *true* | False ❌ |
| *false* `AND` *false* | False ❌ |

If both of the statements are either true or false, the result will be true.

Usage of these both will help us ensuring an SQL injection is present.

Just to give a brief about what the `NOT` operator is used for, let's take an example of a table consisting of all the people living in some country. You want a list of all the people in that country whose job title is NOT *Thought Leader*. A query for that would look something like this:

``` SQL
SELECT first_name, last_name FROM population WHERE job is NOT "Thought Leader";
```

## What is SQL Injection?
SQL injection is a web based attack in which the malicious end-user enters an SQL query (in an input field or a parameter) which would append to the existing query in the logic tier of the application and this now new (malicious) query is passed on to the database which executes it, if it's a fully-working query and not broken syntactically, and returns the result back to the end-user.

![DROP Tables. ](/sqli-0x01/exploits_of_a_mom.png)
<div style="text-align: center;"><span style="font-size: small;">Credit to <a href="https://xkcd.com/327/" target="_blank">XKCD Comics</a>. If you've never checked them out, YOU SHOULD!</span></div>
<br>
Considering the above comic, although destructive and not very beneficial, it still is a SQL injection.

When a web application fails to properly sanitize the user input (parameters that are passed to the SQL statement or query), the malicious SQL query will be executed. This query will be executed with the same rights as the web server.

If a command is being executed on the system via the database server, this command will be executed on the system with the rights of whoever deployed the database server. If MySQL (mysqld) is running as root user, then the commands that will be executed on the system will be as root.

### Classic SQL Injection - Authentication Bypass
How could I possibly end this post without actually displaying an SQL injection?! And welcome to the section you were waiting for.

Let's consider the following PHP code of a login page as an example:
We are passing our input in the `user` and `pass` field
``` PHP
# Takes the user input from the login POST request
$user = $_POST['user'];
$pass = $_POST['pass'];

$query = "SELECT * FROM users WHERE username=$user AND password=$pass";
```
The user controls the SQL query parameters "username" and "password" because they can potentially send in any value and it would be passed on to those parameters.

**Let's consider a legitimate request first**:
POST login request sent by a legitimate user with `user=admin&pass=amdin` and there's a typo in the password field

The SQL query that will be built with this request would be:
``` SQL
SELECT * FROM users WHERE username="admin" AND password="amdin";
```
The query will fetch all the entries in the database that matches `username=admin` and `password=amdin`. If no entries exist, the database will return nothing and so the logic tier will receive nothing from the database. The browser will then display whatever error is coded to display that login attempt has been failed.
Maybe something like *Incorrect Password*.

**Let's consider a malicious request now**:
POST login request sent by an attacker with `user='or'1'='1';-- -&pass=lulz`

The SQL query that will be built with that request would be:
``` SQL
SELECT * FROM users WHERE username=''or'1'='1';-- - AND password=lulz';
```

By sending `'or'1'='1';-- -` in the user field in the POST request, we did not only modify the username parameter but also commented out the rest of the query that was initially present, which is the password parameter of the `WHERE` condition check. 

With this two modifications, our malicious query will always yield *true* due to the `OR` operand. Once this query is sent by the logic tier to the database server, the database server will execute it and return all the rows as the result!

If there is no check present as to how many rows the PHP code (logic tier) must recieve, it'll by default take the first row (as it can't take all) from the result received from the database server. Since this first row is very much a valid result, the logic tier would log the malicious user into the system (mostly as admin).

We've successfully performed an SQL Injection to bypass authentication mechanism!

## Fin
If you stuck around and read all the way till here, thank you! If you have any suggestions, queries or found a mistake, feel free to contact me, if you'd like me to credit you regarding it, I won't mind that.

Regarding this series...it will go in depth from the basics to as advanced as I possibly can which would very much be out of the scope of OSCP and maybe even OSWE. There would probably be a weekly update to this series or as soon as I learn enough to blog about it.
This series is not just to teach you about SQL injection but are also my personal notes if that gives you any more confidence about the quality of this.

Have a great day, take care and hack the planet!

Read the next post [SQL Injection 0x02 - Testing & UNION Attacks](/sqli-0x02)