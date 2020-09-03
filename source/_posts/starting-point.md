---
title: Getting Into Cybersecurity - Red Team Edition
date: 2020-08-31 15:48:04
tags:
---

# Getting Into Cybersecurity - Red Team Edition

## Introduction

I came across this question and was asked so often that I decided to turn it into a blog post.

Since finance is one of the biggest barrier that one could face, every resource listed here will be either completely free or at least provide a good amount of free resources/content.

## Disclaimer

The following suggestions will be more towards getting into penetration testing or red teaming but if that is not your thing or rather you do not know what your thing is I would highly recommend reading through [Lesley Carharts'](https://twitter.com/hacks4pancakes/) [Starting an InfoSec Career - The Megamix](https://tisiphone.net/2015/10/12/starting-an-infosec-career-the-megamix-chapters-1-3/) series. The series is quite extensive and you would also get a look at different specializations in infosec from the amazing people who are working in it; it is fantastic. Also, Lesley is one of the most talented person in the infosec community and you gotta follow her on [Twitter](https://twitter.com/hacks4pancakes/).

I have myself barely just gotten my feet wet in the massive ocean that is infosec, but I know a little and would love to share whatever I know if it helps you out. In no shape or form is this gonna be THE ULTIMATE guide to follow, there are a TON of options and you must research on your own to find what works best for you ¯\\\_(ツ)_/¯

All this will only help you get started, perhaps even make you ready for an entry level job, but that is the most all these courses, CTFs, and trainings can provide you. Not so surprisingly, experience is the most important thing in this field and is what will help you get through your future hurdles, and you will build that in time.

## Who This Post Is For?

Anyone who is just starting out in the industry, not sure where to begin, basically a noob. The following suggestions are as per my experiences, these are the things I did, and it has worked out for me, clearly.

## No IT Experience

Let's say you found this field very interesting and want to dive into it but do not have any IT or computer experience before. I would suggest doing the [Google IT Support Professional Certificate](https://www.coursera.org/professional-certificates/google-it-support) from Coursera. This course will cover all the basics you need to know before you get started with cybersecurity. The course provides 7 day free trial, so hack your way through it for a free certificate :)

## Let's Begin

Ok so you decided to be a pentester, let's get started!

*Note: None of the following needs to be done in a particular order, you can do things in any order whichever makes you feel more comfortable*

### Google

This might seem like a little obvious or feel like I'm making fun of you, I'm not. Google will be your best tool throughout your journey regardless of what you intend on doing. Learn how to look up things properly, learn some [Google Dorks](https://securitytrails.com/blog/google-hacking-techniques) to look for specific things. Google every issue or doubt you come across, there is a very high chance someone else went through that too.

### Note Taking

Before you start with anything, ensure that you take plenty of notes. There are a ton of notes out there, but creating your own notes and hacking sheets will help you the most. I use following applications (both are free):

1. [Notion](notion.so/) - General note taking of almost everything I learn
2. Joplin - Jotting down hack steps of each machine I tackle on HackTheBox

### Linux

There's no getting rid of the penguin. Ok, maybe there is, but learning Linux will open up a lot of the playground with which you can learn how to hack. Not to mention, a majority of the servers are infact Linux-based, along with almost all the IoT devices. There is a lot you can do with this.

Start by learning the basics of [Linux](https://www.youtube.com/playlist?list=PL6gx4Cwl9DGCkg2uj3PxUWhMDuTw3VKjM)

Some commands/tools that I feel you absolutely must learn are:

 - [ ] vi/vim - learn from `vimtutor` command on Linux
 - [ ] cut
 - [ ] sed
 - [ ] awk
 - [ ] curl
 - [ ] grep
 - [ ] tmux - learn from [here](https://www.youtube.com/watch?v=Lqehvpe_djs)

Once you've done that you could head on to [OverTheWire](https://overthewire.org/wargames/) and start by playing the Bandit series to not only practice Linux in a CTF-style environment but you'll also learn some cool tricks which may help you in the future.

Once you have grasped the basics, you should head on to bash scripting. Bash scripting, in my opinion, is quite underrated. Linux commands can do a ton of stuff individually, and you automate it using bash scripts you can achieve pretty amazing stuff like automating your exploitation process, or even enumeration.

You can learn bash scripting from [here](https://www.youtube.com/playlist?list=PLS1QulWo1RIYmaxcEqw5JhK3b-6rgdWO_)

### Python

I would also highly recommend learning Python. Like bash, python will come in handy in terms of automation, in some places bash will feel more "right" to use, other times python.

You can start learning python from [here](https://www.youtube.com/playlist?list=PL6gx4Cwl9DGAcbMi1sH6oAMk4JHw91mC_)

One of the libraries that I would highly recommend learning is the *requests* library.

Python for hacking/exploitation is no different than any other python. Python is python. You just need to apply the concepts in different ways. Some of the projects/scripts that I would recommend making are:

1. pingsweep
2. portscanner
3. bruteforcer
4. bruteforcer bypassing CSRF tokens
5. SQL injection automation

You can achieve the same in bash :)

### Other Languages

As Python can help in automating things, each language has its' own pros and cons.
Other languages that you should consider learning are:

 - [ ] C - Useful for general binary exploitation purposes - [resource 1](https://www.youtube.com/playlist?list=PL0170B6E7DD6D8810), [resource 2](https://beej.us/guide/bgc/html/single/bgc.html)
 - [ ] C# - Useful for Windows exploitation - Google
 - [ ] C++ - Useful for Windows exploitation - Google
 - [ ] PowerShell - Useful for Windows exploitation - Google 

Web languages that you should be able to read at the least are:
 - [ ] PHP
 - [ ] JavaScript

If you're wondering why did I not mention Java, the reason is quite simple, I absolutely despise it. If you want to you can learn it enough to be able to read it and understand what the code is doing.

### Computer Networks

Learning computer networks in essential. It might feel like a lot but in time you will get a hang of it. You do not need to take a graduate level networking course or something, but you should know the following at the least:

 - [ ] OSI model
 - [ ] How TCP and UDP work
 - [ ] TCP three way handshake
 - [ ] Some important protocols work - HTTP, SSH (these are the ones at top of my head right now)
 - [ ] Subnetting

This [YouTube playlist](https://www.youtube.com/playlist?list=PL6gx4Cwl9DGBpuvPW0aHa7mKdn_k9SPKO) covers most of the topics if not all. Google things to learn more.

To understand the flow of connections and learn more about protocols, learn [Wireshark](https://www.youtube.com/playlist?list=PL6gx4Cwl9DGBI2ZFuyZOl5Q7sptR7PwYN), and tcpdump.

Want to learn networking a little more "properly"? Go through this [Guide to Network Programming](https://beej.us/guide/bgnet/html/)


### Web Application Exploitation

With Linux, Python, and computer networks are out of the way, you now know some good amount of basics to go ahead with.

Next you could start learning how to exploit web applications.

The best tool, in my opinion, that you should learn how to use as proficiently as possible is [PortSwiggers' BurpSuite](https://portswigger.net/burp), which is a web proxy tool.

One of the best resources currently out there is PortSwiggers' [Web Security Academy](https://portswigger.net/web-security). It goes through a lot of web exploits which are quite realistic, you could start with XSS or SQL injection. It also provides a free lab to practice each kind of exploit.

You can practice your python skills here as well.

Another resource that you should check out is [OWASP](https://owasp.org/), Open Web Application Security Project. OWASP provides brilliant projects like [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) and [OWASP TOP 10](https://owasp.org/www-project-top-ten/). You should go through them thoroughly.

You can also leverage two different vulnerable labs to practice your web hacking skills - [DVWA](http://www.dvwa.co.uk/) and [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)

You can also practice on [OverTheWire - Natas](https://overthewire.org/wargames/natas/)

### Capture The Flag

After learning all that the only way to learn more is by hacking. Some of the CTF websites that you can get your hands dirty are:

 - [ ] [HackTheBox](https://www.hackthebox.eu/) - free + paid (personal favourite)
 - [ ] [VulnHub](https://www.vulnhub.com/) - completely free
 - [ ] [TryHackMe](https://www.tryhackme.com/) - free + paid
 - [ ] [OverTheWire](https://overthewire.org/wargames/) - completely free
 - [ ] [Hacker101 CTF](https://ctf.hacker101.com/) - completely free

## Proof of Knowledge

There are multiple ways of showing that you know something - certificate of completion, a social media post of "I hacked machine X", and so on.
What stands out the most is making a blog and writing about what you learned, write about something you hacked and explain the vulnerability as deep as possible and at the same time suggest remediations to fix the vulnerability.

Create a GitHub repo to store all your scripts there, and display those as well!

### Certifications

If you're considering getting certified or going for some certifications, I have stated which certification will provide you what:

1. CompTIA Security+ : to understand the basics of security theory of different verticles (*recommended*)
2. Offensive Security Certified Professional (OSCP) : a stamp that you now know the basics (*recommended*)
3. Certified Ethical Hacker (CEH) : for HR purposes only

Certifications are expensive, so if you do intend on getting one somehow while managing your financial problems, get only OSCP.

## Fin

Before you go, read [this](https://twitter.com/z3roTrust/status/1296666120426942470).

I believe this is more than enough to get anyone into pentesting, if you have any doubt or suggestion, feel free to contact me :)
Take care and hack the planet!