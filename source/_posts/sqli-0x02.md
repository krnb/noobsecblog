---
title: SQL Injection 0x02 - Testing and UNION Attacks
date: 2020-07-10 22:31:47
tags: [sqli, sql injeciton, web attacks]
---

# SQL Injection 0x02 - Testing and UNION Attacks

## Introduction

Things covered in this blog post are:
- Testing
- UNION based attacks
- Optimizing injections

## Vulnerability Identification
To be able to identify potential SQL injection points, it's essential that the application and its functionality is browsed through and understood, making note of all the pages and functions that might be interacting with the database. Some of the example of where an application might be interacting with the database would be:
- authentication forms
    - login
    - password reset
- e-commerce platforms
    - product display

Make note of all the user input fields, parameters being passed in requests. These would be the testing points for SQL injection.

## Vulnerability Testing
Once all the user input fields and parameters have been identified, each field and parameter needs to be tested individually. Along with these, HTTP headers like User-Agent and Cookie should also be on your list to test for. User-Agent and/or cookies might be getting tracked and written onto a table for analytics purposes or tracking users.

Best way to test is to use something a SQL database would treat it as a part of the query itself.

``` SQL
'       # a single quote, treated as a string terminator
;       # a semi-colon, treated as a query terminator
#       # a comment, comments out anything that comes after that character in that line
-- -    # a comment, same as above
/**/    # a comment, starts with /* , ends with */ , and can contain anything in between 
AND 1=1 # an and statement, 1=1 tests for true condition. Use 1=2 to test for false conditions
OR 1=1  # an or statement, 1=1 tests for true condition, use 1=2 to test for false condition
sleep(5)# delays the response by the number of seconds mentioned in the sleep command. 
```
The objective of these tests is for the existing query to return true, false or some kinda subtle changes to the webpage or bring out the errors.

### Examples
All examples here are from HackTheBox - Charon machine.

Normal looking blog post
![Normal Post](/sqli-0x02/1_normal.png)

1. Single-quote - `'`
Add a single quote at the end of the parameter or in an input field.
Eg: http://example.com/blog.php?id=11'

![Single Quote Test](/sqli-0x02/2_single_quote.png)

The post broke due to the semi colon, indicating that the parameter id could potentially be vulnerable to SQL injection

2. Semi-colon - `;`
Add a semi-colon at the end of a parameter or in an input field 
Eg: http://example.com/blog.php?id=11;

![Semi Colon Test](/sqli-0x02/3_semi_colon.png)

This indicates that even though the query may be getting terminated, the query is working just as intended.

3. Comments - `-- -`
Comments passed onto the parameter or the input field needs to be database specific. MySQL uses the above mentioned comments.
Eg: http://example.com/blog.php?id=11--%20-

![](/sqli-0x02/4_comments.png)

Even after commenting out the rest of the query, it is working as intended.

4. And operator - `AND`
AND operator should be passed with some additional condition in parameters or input fields.

![](/sqli-0x02/5_and.png)

This means we can probably append any SQL statement to the `id` parameter. Since a post with id=11 exists and 1=1, it comes out to be true and the post loads. If the same is tried with some random `id` value, while 1=1, the post will not load.

5. OR operator - `OR`
`OR` operator should be passed with some additional condition in parameters or input fields.

![](/sqli-0x02/6_or.png)

Unlike `AND`, whatever value you put in either the `id` field or the condition after `OR`, as long as one of them is `true` the requested blog post or the first post in the blog will load.

6. Sleep - `sleep(seconds)`
When output doesn't show that something is vulnerable to the SQL injection, doesn't mean that it's not. It could be blind.
At times like those a `sleep` statement could be passed like `id=11 AND sleep(20)` and look at the amount of time it takes to send the response back, if it's anywhere from 18-22 seconds, you have a SQL injection on your hand.

## Vulnerability Exploitaion - UNION Attacks

Once an SQL injection has been identified and tested, it is time to exploit it. An SQL injection exploitation is usually used in order to find something juicy - credentials, customer details, credit card numbers.

This post, as mentioned above, will cover UNION injections. Let's continue with the example above.

### Backend Query

We've successfully found that the `id` parameter is vulnerable to SQL injection. The way the code might be working at the backend would be that the PHP takes the id parameter from a GET request and then passes it to some internal variable, which puts it into a SQL query. 
If we had to guess the query it is executing it could be something like below:
``` SQL
SELECT title, author, date, post FROM blogpost WHERE id=11
```

With this guess we can estimate that this particular table has 5 columns. Let's test it out.

### Number of Columns

We'll send a set of `UNION` requests to identify number of columns like follows:
```
http://10.10.10.31/singlepost.php?id=11 UNION SELECT 1-- -
```

![UNION Testing](/sqli-0x02/7_union_1.png)

We can rule out that the table does not have just 1 column, and repeat the process by adding another number.

![UNION Worked](/sqli-0x02/8_union_done.png)

After few iterations we found that the table indeed has 5 columns, our estimation was correct.

The reason the earlier UNION query did not work is because, a UNION of two tables has to have *equal* number of columns for it to be considered as a working query.

We can send in anything in the `SELECT` part of the `UNION` query since it is not taking anything from a table. We used numbers but you can also send in alphabets; if you do go for alphabets, do not forget to add quotes.

We do not know which column number (from our UNION) is going to which part of the blog post - title, author, date, post.
To do that we can ask the webpage to load a non-existent post (a random `id`), which would result as `false` thus not loading anything, while our `UNION` is present. This will lead to our injected query getting printed out.

![Column Position Found](/sqli-0x02/8_union_ok.png)

We can see that fifth column printed out first as title of the post, third and second column as author and date respectively in tiny font, and then the fourth column which has quite some space to put text in as it is used for the post of the blog.

Now that we have column position on hand, we can start dumping data out on the screen.

### Enumerating Database

Let's look at the version of the MySQL database. We can use MySQL internal functions at our advantage here.

![DB Version](/sqli-0x02/11_version.png)

Let's check out the name of the database we're using. In database systems, you can use multiple databases (within a single database management system like MySQL) under different user contexts.

![Database Found](/sqli-0x02/12_blog_union_database.png)

We are using a database called "freeeze"

Let's take a look at the user we are running as.

![User Found](/sqli-0x02/13_blog_union_user.png)

We are running as "freeeze@localhost". Not so useful, let's move on.

### Enumerating Tables

Now we want to know what tables exist in the database we are using.

In SQL we can list down tables and columns that exist in the database. In MySQL, there are a set of tables called as ***information_schema*** tables which holds all the "metadata" or useful information for the database to function properly.

The most commonly used tables of ***information_schema*** set are:
1. *information_schema.tables* - A table containing information about all the tables in the database
2. *information_schema.columns* - A table containing information about all the columns of all the tables in the database

We'll essentially send in a query as follows:
``` SQL
SELECT table_name FROM information_schema.tables WHERE table_schema=database()
```
This will `SELECT` (print out) all the tables `FROM` the table called *information_schema.tables* `WHERE` those tables are in the current `database()`

On the above query I'm using a function called `GROUP_CONCAT()`. This function not only concatenates the entire output, it also places it in a single row. You can use this function wherever there is a `LIMIT` on the backend and you can't print everything out in a single command.  

In the form of request it will look like below
``` SQL
http://10.10.10.31/singlepost.php?id=191+UNION+SELECT+1,2,3,4,GROUP_CONCAT(0x7c,table_name,0x7c)+FROM+information_schema.tables+WHERE+table_schema=database()--+-
```

With group concat, you can pass in hex characters or even additional ascii to provide your results more distinction and even aesthetics.

Output of the above request

![Table Found](/sqli-0x02/15_blog_union_current_db_table.png)

Apparently this particular database only had one table, and we can guess that it's not of any use. But let's take a look anyway.

### Enumerating Columns
We leverage the use of *information_schema.columns* table to get all the columns from the table *blog* we found earlier.

Request sent: 

``` SQL
http://10.10.10.31/singlepost.php?id=191+UNION+SELECT+1,2,3,group_concat(0x7c,column_name,0x7c),5+FROM+information_schema.columns+WHERE+table_name='blog'--+-
```

![Columns Found](/sqli-0x02/16_columns_blog.png)

### Getting Contents
Before printing out everything, let's find out the number of rows present in the table. On the website, it showed only 3 posts, it could be a possibility that there are hidden posts present here.

Let's take a count of `id` column as that column has to be present in a table.

Request sent:
```
http://10.10.10.31/singlepost.php?id=191+UNION+SELECT+1,2,3,4,count(id)+FROM+blog--+-
```

![Number Of Rows](/sqli-0x02/17_row_count.png)

Unfortunately there are only 3 posts present and there is no hidden content available here.

## Note

Coming up in next 24 hours:
- another example (UNION attack) is .
- a summary + cheatsheet.

If some part of it feels unexplained or you did not understand, feel free to contact me :)