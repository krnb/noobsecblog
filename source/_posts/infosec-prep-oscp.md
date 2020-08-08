---
title: Infosec Prep OSCP Voucher Giveaway Writeup
date: 2020-08-06 10:33:26
tags: [walkthrough, oscp-like]
---

# Infosec Prep OSCP Voucher Giveaway Writeup

## Introduction

![Challenge Preview](/infosec-prep-oscp/0_challenge1.png)

The [Infosec Prep Discord server](https://discord.gg/y3ERAXt) had recently announced an OSCP voucher giveaway except they only allowed people to enter in the giveaway pool by submitting the flag of this challenge, which I personally feel was a much better approach than before - ...clicking emojis to enter.

This machine was created by [FalconSpy](https://medium.com/@falconspy) and was uploaded on VulnHub and can be found [here](https://www.vulnhub.com/entry/infosec-prep-oscp,508/). It is like any other Boot2Root type machine and is very easy. By creating this requirement it is ensured that at least only those will be selected who are ready to get into the PwK course and not someone who still hasn't grasped the basics yet.

I have written this writeup for *absolute beginners*, with no assumptions in consideration. If you feel like you didn't understand something feel free to contact me.

## Setting The Lab

To start you download the VM, unzip it, and double-click the extracted file.
The VM starts with the network adapter connected via "Bridged" mode, which means it is directly connected to your network (maybe WiFi) like any other device. Connecting a vulnerable machine directly to the network is not such a good idea, and we can do something better to keep our network secure. We will change that setting and ensure it is connected and accessible only via our host machine by switching the network adapter to "Host-only".

Right Click on the network icon on the top and then click on Settings

![Right Clicked](/infosec-prep-oscp/1_change_adapter_vuln1.png)

Select "Host-only", and then OK to save the changes.

![Adapter Changed](/infosec-prep-oscp/1_change_adapter_vuln2.png)

In order for us to use our attacking machine, Kali in my case, to hack this vulnerable machine, we need to ensure that both these machines are in the same network. So we will change Kalis' network adapter to "Host-only" as well. Perform the same steps as above for the Kali machine or whichever OS you're using to tackle this challenge.

Once the adapters have been changed the IP addresses of both the machines would have changed. Let's find out what these new IP addresses are. 

On our attacking machine, we will enter the following command in the terminal

``` bash
ip -c --brief addr
```

![Kali IP](/infosec-prep-oscp/2_kali_ip.png)

Great, we got our IP and the network we are in. But how do we get the IP of the target machine when we do not even have access, you cannot login to the target machine if you did not know that, to it.

The Infosec Prep folks were nice enough to give you the IP directly on the login screen

![Target IP Given](/infosec-prep-oscp/3_tgt_ip1.png)

But what if that wasn't provided, how could you get the IP then? Well, since we know the network in which this machine resides, which is same as our attacking machine - `192.168.174.0/24`, we could use `nmap` to get the IP address of the target machine. Another tool you can use is - `netdiscover`, you should always mess around with different tools and identify which works better when or which one you feel more comfortable with while also getting the job done.

Before starting with any machine it's important that you create a separate directory for that machine, and save all the output in that directory. I usually create (check my mkdir alias <a href="/linux-tips/#Directories" target="_blank">here</a>) a parent directory with the machine name, and then three child directories to save files in their relevant folders.

![Creating Directories](/infosec-prep-oscp/4_mkdir.png)

## Reconnaissance

### Finding The Target

We will perform a ping sweep on our network to get the targets' IP address.

``` bash
# -sn : ping scan
# -n : do not perform DNS resolution, it saves us some time
# -oN : save output as "pingsweep" in Nmap format in the directory "nmap" 
# 192.168.174.0/24 : asking nmap to scan our entire network (/24) to identify any active hosts
nmap -sn -n -oN nmap/pingsweep 192.168.174.0/24
```

![Target IP Found](/infosec-prep-oscp/3_tgt_ip2.png)

Since we know that our IP is `192.168.174.129`, then the other IP got to that of the target machine.

### General Enumeration

Now that we have its' IP address, we will scan the machine to see which ports and their respective services are active.

Command sent:

``` bash
# -Pn : assume host as active, do not perform ping scan
# -n : do not perform DNS resolution, it saves us some time
# -oN : save output as "initial" in Nmap format in the directory "nmap" 
# 192.168.174.128 : asking nmap to scan the target machine
    # notice that we removed the network part - "/24"
nmap -Pn -n -oN nmap/initial 192.168.174.128
```

Output:

![Initial Nmap Scan](/infosec-prep-oscp/5_nmap1.png)

Great our first nmap scan is here pretty quickly with the results. It is important to note that when not specifying ports to scan, Nmap automatically takes the "top 1000 ports" and scans a target for these ports. By top 1000 ports, I do not mean ports 1 to 1000, but rather most frequent ports that Nmap has come across so far since its' inception.

From the output, we can see that there are only two ports open on this machine:

1. Port 22 - SSH
2. Port 80 - HTTP

Let's take some mental notes!
Since port 22 is open, this machine is most likely a Linux machine because SSH is a very common service that is active on Linux based hosts unlike Windows. You would have probably known that the target is based on Linux by reading the description of the machine on VulnHub or maybe just by looking at the login screen, but we'll always tackle a machine assuming we have no prior knowledge about the target other than its' IP, if at all.

Since port 80 is open, the machine must have some kinda of web server running and a website too.

Let's scan these two ports by using Nmaps' scripts, to get more information about them.

Command sent:

``` bash
# -p<xx,yy,zz,...> : Specifying ports to make Nmap scan only the ones mentioned
# -sC : Scan ports using Nmaps' default scripts to extract more information from within the services
# -sV : Scan ports to identify the versions of the services
nmap -Pn -n -p22,80 -sC -sV -oN nmap/initial_target 192.168.174.128
```

Output:

![Targeted Nmap Scan](/infosec-prep-oscp/5_nmap2.png)

Great, we got a lot of information back. Let's again make some notes.

| Port | Service | Type | Information |
| ---- | ------- | ---- | ---------- |
| 22 | SSH | Version | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 |
| 80 | HTTP | Version | Apache httpd 2.4.41 Ubuntu |
| 80 | HTTP | Web Application | WordPress 5.4.2 |
| 80 | HTTP | robots.txt | secret.txt |
| 80 | HTTP | Server Header | Default |
| 80 | HTTP | Title | OSCP Voucher | 

I listed all the information we got in a tabular format to view and refer from better. Before we start interpreting these results, let's have an all port scan run in the background as it could take some time to finish.

Command sent:

``` bash
# -p- : Scan for all the ports (1-65535) on a given host, return active
nmap -Pn -n -p- -oN nmap/allports 192.168.174.128
```

Now let us get back to the results of the initial targeted scan.

From the SSH and HTTP version we got from Nmap, we can say that this is an Ubuntu machine and we can also find out which one, which I'll show in the SSH enumeration section.

We can also look up these versions to see if there exists any publicly known exlpoits for their respective services. 

We know that a web application is used here, WordPress, to make a website.

We also found a hidden file called *secret.txt* which was in the "Disallow" list in the *robots.txt*. 
Server header being default isn't of much use here.

The title of the first page captured is not default, indicating that we may not have to look for hidden content.

### SSH Enumeration

Although we were able to get the version of the SSH doing an Nmap scan, you could achieve the same manually as well and learning and knowing how to achieve objectives manually is a crucial skill.

The process of getting the version and some basic information from a service is called Banner Grabbing, in which you connect to a service as any other normal user and let it show you some basic information which often includes version.
Sometimes on corporate services, upon connection, it shows which firm owns that service and that unauthorized access is not allowed. This [image](https://www.tecmint.com/wp-content/uploads/2012/11/SSH-Banner.png) can be taken as a example.

We can use `telnet` to get the SSH version

![SSH Version Using Telnet](/infosec-prep-oscp/6_ssh_enum1.png)

We can also use `nc` to achieve the same

![SSH Version Using Nc](/infosec-prep-oscp/6_ssh_enum2.png)

We were successfully able to get the version manually using two different commands.

Let's get the OS running from this SSH version. Since the attacking machine is connected host-only, I will do this on my host machine.

We'll search for "ubuntu openssh launchpad" on Google, and the very [first link](https://launchpad.net/ubuntu/+source/openssh) was exactly what we wanted, all the openssh versions to their respective Ubuntu versions. If you don't know what launchpad is, it is a collaborative platform which keeps track of all the Ubuntu packages and bugs and much more.

![Ubuntu Version Found](/infosec-prep-oscp/6_ssh_enum4.png)

It shows that this version of OpenSSH is in Ubuntu Focal Fossa. Another quick Google search shows that "Ubuntu Focal Fossa" is Ubuntu 20.04 LTS. There probably isn't any public exploits available for this machine as it is quite recent, but if the OS you find turns out to be a little old then you could probably search for exploits against it too.

### HTTP Enumeration

Let's move to enumerating the web manually, starting with finding the server version in a few ways.

First we'll use `curl` to find the version of the web server.

Command sent:
``` bash
curl -i http://192.168.174.128
```

![Web Server Version Found](/infosec-prep-oscp/7_http_enum1.png)

As soon as the command is executed we see a lot of data dumped on the screen, once we go all the way to the top where command was executed we see the server response details with the web server version.

Another way of finding the server version is by browsing to a non-existent page via a web browser.

![Web Server Version Found](/infosec-prep-oscp/7_http_enum2.png)

And we got the web server version.

### General Enumeration Contd.

Before we go any further, let's take a look at the all port scan and see if any new ports were found.

![All Ports Nmap Scan](/infosec-prep-oscp/5_nmap3.png)

Looks like there's an additional port found - 33060 mysqlx. Since this is a MySQL port, we won't really look into this.

## Initial Foothold - Sensitive File Leak (SSH)

In our initial targeted Nmap scan, we found that in robots.txt there's an entry in the "Disallow" list - secret.txt.
Let's take a look at it.

![secret.txt](/infosec-prep-oscp/8_secret1.png)

It looks like it is a base64 encoded string, we can say that because the string is ending with two equal signs. Let's copy its' contents (or you can download the file using `wget`) and decode it.

![secret.txt Decoded](/infosec-prep-oscp/8_secret2.png)

And the secret is out. The encoded string is nothing but a SSH private key. The key must belong to some user, so let us hunt for one.

***Note: When decoding a file, its' contents must be copied and pasted exactly as it is. No additional character or new line must be present in the pasted file. This is to ensure the decoding is done exactly as intended***

Let's take a look at the website, as there might be some information there, intentional or otherwise.

Upon browsing the index, the post gave us the user that is on the machine - *oscp*.

![SSH User](/infosec-prep-oscp/9_ssh_user.png)

Let's make the private SSH key suitable to use, by ensuring the key is not writable by any other person. To do so we change the permissions of the file using `chmod` command and setting permissions to `600`.

![SSH Key Permissions](/infosec-prep-oscp/10_ssh1.png)

I also renamed the file appropriately as per its' purpose. Now we are ready to use this key to login as "oscp."

![Initial Access Achieved](/infosec-prep-oscp/11_lowpriv.png)

We have successfully logged in as "oscp."

## Privilege Escalation

### Local Enumeration

We will use [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) to enumerate the system.

We will transfer the script to the target machine. To do so, we will host the script on our machine using python server, `pysrv` as a command does not exist, the term is an *alias*, you can check my aliases <a href="/linux-tips/#Python-Server" target="_blank">here</a>) and download it on the target using wget.

![Hosting The Script](/infosec-prep-oscp/12_linpeas1.png)

![Script Transferred](/infosec-prep-oscp/12_linpeas2.png)

Once the script is transferred, we make the script executable using the following command:

```
chmod +x linpeas.sh
```

Once the script is executable, we run it. LinPeas is a nice tool which highlights binaries if it knows the binary is vulnerable or exploitable for sure.

Once executed, it will start dumping all the information out on the screen, while going through it you will see that the bash binary is highlighted in the "SUID" section.

![Bash SUID](/infosec-prep-oscp/13_pe1.png)

Great, it looks like we have the privilege escalation binary on hand.

If you wanted to find this binary manually you could run the following command to get all the SUID binaries, and then compare those binaries against your system to find any non-standard binary or binaries that shouldn't ideally have SUID enabled.

``` bash
find / -perm -u=s -type f 2>/dev/null
```

### Getting Root Shell

Although we know which binary to target, we do not know how. Luckily for us, there happen to exist a collaborative project called [GTFOBins](https://gtfobins.github.io/) which keeps track of all such binaries which could be exploited in several ways.

We can search for "bash" in that website, and select the functionality, SUID, which we have access to.

![Bash SUID](/infosec-prep-oscp/13_pe2.png)

The first command is to make bash a SUID binary, but we already have one so we'll ignore that.
All we have to do is execute our SUID binary as - `/usr/bin/bash -p` to get root.

![Got Root](/infosec-prep-oscp/14_gotroot.png)

Let's go get our loot

![Got Flag](/infosec-prep-oscp/15_gotflag.png)

## Unintended Privilege Escalation Methods

Apart from this method, there turned out to be two more ways that one could have taken to become root. FalconSpy has written about both the unintended methods [here](https://medium.com/@falconspy/infosec-prep-oscp-vulnhubwalkthrough-a09519236025)

## Lessons Learned

1. Ensuring vulnerable systems are only accessible via our host system(s).
2. Enumeration/reconnaissance is an important phase and could be the most lengthy part when targetting a system but very much worth it.
3. Mis-configurations are a much bigger threat than some cool exploit.

## Fin

If you did not get something, or found some mistake, feel free to contact me.
Take care, and keep hacking!