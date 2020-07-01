---
title: Hack OSCP
date: 2020-06-30 22:45:33
tags: [oscp]
---

# Hack OSCP - A n00bs Guide

Since I cleared OSCP plenty of folks asked me how to clear OSCP, and although I briefly mentioned it in my <a href="/oscp-journey" target="_blank">OSCP Journey</a> post, it was not the whole picture and also not very accessible, and so I'm writing this post.

Please note that PwK is a course you're paying for to learn from, the course teaches you almost everything you need to learn and you'll get to practice in the labs as well. There's no need for over preparation, but I understand how anxiety can be and so this post will point you to almost all the resources I could suggest to ensure you're ready. By doing the following you'll be enough prepared to head into the exam on your first day in the course if you want to, although **NOT** recommended.

## Pre-requisites
1. Linux
    1. Navigating filesystem
    2. Common commands
2. Basic bash scripting
    1. Able to read and understand a bash script
    2. Able to create scripts like pingsweep
3. Basic python scripting - same as above

The above pre-requisites are now taught well in the PWK course, but you should know these to be able to get your hands dirty for the practice below. If you don't, it's ok, I've linked resources below which will cover that.

## Basics

To get your basics on I'd highly suggest doing the [Practical Ethical Hacking](https://www.udemy.com/course/practical-ethical-hacking/) course by [Heath Adams - thecybermentor](https://twitter.com/thecybermentor/)

In this course you'll learn the basics of Linux, python scripting, Active Directory stuff, and a bunch more. It's a great start.

## Practice
For practicing something like the PwK Labs itself, I'd highly suggest going through the [HTB OSCP-Like machines](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159) and **hack every single machine on that list!**

For ease of use I'd created an Excel spreadsheet of my own where I can do all sorts of filtering to decide which machines I want to target first and to keep a track of my progress. You can find the Excel sheet [here](https://github.com/krnb/oscp-prep) 

In the Heath Adams' course you'll be hacking few machines along with him, so that should probably give you a start.
If you won't be taking that course because you know the basics and want to move on to practice immediately but lack confidence I'd suggest you follow this process:
- Select a machine (maybe the easiest when you're first starting)
- Enumerate the machine with anything and everything you know
- If you're stuck on some step X, do some research. Google is literally gonna be your best friend throughout this course and in the future.
- If you don't land anywhere and feel you have exhausted all your resources, check how IppSec did it.
    - But only look at that particular step
- Again repeat the process till you've successfully exploited it
- Document everything
    - Use cherrytree or Joplin or whatever suits your need
- Document your enumeration process
- Document why you did what you did
- Document why ABC worked but XYZ did not
- Upon exploitation of the machine, look for more info as per why something worked and other thing did not
- Make a separate document and jot down all your learnings on this machine and new enumeration or exploitation step you learned
    - Use this document to list down learnings from every machine you hack, organize it well. This will be the final result of everything you've learned so far

If it was not obvious in the above process then let me clarify something, what is important in doing all this is **BUILDING YOUR METHODOLOGY**. Learning how to hack machine xyz, or how to exploit abc vulnerability is not the point of this. It's to build a process and develop the [hackers mindset](https://www.offensive-security.com/offsec/what-it-means-to-try-harder/).

The above practice will be a bit more harder than OSCP itself, which is great. You'll come across things which are a lot more complex to deal with and easy to get overwhelmed from, like binary analysis or blind SQL injection, you don't need to get into that right now, feel free to keep those parts for later, maybe learn from them once you've cleared OSCP.
Your focus should be on learnings various things from each box, but not everything. This would also not be possible for you for various reasons other than you're a beginner.

### Buffer Overflow
Apart from all that practice you absolutely need to practice buffer overflow which holds 25% weightage of the OSCP exam.
Note: you do not need to practice them before your PWK course starts, the course does a good job in my opinion.

If you'd like a buffer overflow tutorial then you can watch thecybermentor's [Buffer Overflow Made Easy](https://www.youtube.com/playlist?list=PLLKT__MCUeix3O0DPbmuaRuR_4Hxo4m3G) series

Practice on [SLMail](https://www.exploit-db.com/apps/12f1ab027e5374587e7e998c00682c5d-SLMail55_4433.exe) and [Brainpan](https://www.vulnhub.com/entry/brainpan-1,51/) on a free Windows VM available [here](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/). Remember to run the VM in a host-only network and turn off protection mechanisms before you start practicing.

## Privilege Escalation
You might still face issues with privilege escalation even after all the practice you did above, which is fine.
I can highly recommend following courses by [Tib3rius](https://twitter.com/TibSec) 

https://www.udemy.com/course/windows-privilege-escalation/
https://www.udemy.com/course/linux-privilege-escalation/


## Epilogue
I believe this is almost everything you need. I'm sure you will crack this and I wish you all the best!
Hack the planet one step at a time! 