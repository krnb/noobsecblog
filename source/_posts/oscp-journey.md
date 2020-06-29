---
title: OSCP Journey
date: 2020-06-27 10:26:58
tags: [oscp]
---

# OSCP Journey
From a persistent n00b who couldn't even hack a medium difficulty machine on his own to cracking OSCP in 4 months!

## Background
I wanted to do the PwK course and clear OSCP since past couple years but haven't been able to due to reasons. Two of my certifications were going to expire in August '20, and I had to do a certification to renew them, a perfect opportunity. Before I started with the PwK course, I had very little experience with hacking or even CTF for that matter. My college studies or previous certifications weren't of any help here either. But I decided to take a deep breath, and dive in.

## Prep
My course and lab access started in January, it was the old course, I completed all of my exercises and was only able to crack 14 machines in the Public Network. Not good by any standards, but it was just a start. I was glad I was able to do enough to make my lab report. Once my lab access got over, I took a break of 2 months, focussed on college.

Two months later, in March, I decided to start preparing for the exam. I came across [OSCP-like machines list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit) by [TJ_Null](https://twitter.com/TJ_Null) which looked quite promising and so signed-up for the HackTheBox VIP platform.

I went through every machine (approximately 50) in two months. I watched IppSec's video of the machine I'm targetting, and hacked it and continued that for almost every machine. I made a report for each machine I hacked and documented each step including why certain steps were taken, why it worked and why something I was trying did not, and the reasoning behind it. This was essential because I was under a time crunch, needed to build my hacking methodology, and make it as robust as I possibly can. Apart from watching IppSec's video, I read walkthroughs by various people of the same machine to get more information, their thought process and learnings out of it. All my learnings can be found <a href="/oscp-cheatsheet" target="_blank">here</a>

**HACK EVERY SINGLE MACHINE ON THAT LIST!!!** You'll thank yourself later.

And that was it, and that's all the practice you need apart from buffer overflow. For the buffer overflow, practice on [SLMail](https://www.exploit-db.com/apps/12f1ab027e5374587e7e998c00682c5d-SLMail55_4433.exe) and [Brainpan](https://www.vulnhub.com/entry/brainpan-1,51/) on a free Windows VM available [here](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/). Remember to run the VM in a host-only network and turn off protection mechanisms. My buffer overflow cheatsheet can be found <a href="/bof" target="_blank">here</a>

## Exam
I had scheduled my exam at 3.30 AM, everyone is asleep, can peacefully work on my stuff without any external disturbances. Proctoring started at 3.15 AM, took a little time but went smoothly. Started with my exam at 3.45 AM.
I decided to first get done with Buffer Overflow, easy 25 points. Started AutoRecon on the other 25 point machine in the background. Ran first few commands, and suddenly I'm not receiving any output from the debugging machine, it was completely frozen. Huh? I thought it could be some issue with `rdesktop`, tried to kill it, it won't die. Checked my connection, it was re-connecting. Ugh. Killed the connection, tried to connect to the VPN again, it was trying to connect. I looked down at my taskbar at the internet connection icon, and there it was - my worst nightmare, the internet has crashed. It was 4 AM and I won't be getting any support from the ISPs' end. FUCK. I took my phone, turned the hotspot on, connected to the proctoring software, informed the proctor that my internet has crashed and I'm changing my location. They informed me that they would pause the VPN, I agreed.

I called a bunch of folks where I could continue the rest of the exam from, some places are completely locked down, can't go there. Can go to a relative's place, 50 minutes away by car, their building has a bunch of Covid-19 cases in there, and a whole lot in a hospital near that place. Fuck me. Looked for cabs, luckily I found one, booked it immediately, packed my stuff. It was 5 AM, I've got nothing and chances are I could catch the virus, not the best of odds.

<img src="https://media.giphy.com/media/xU1spRleFHmtjvskXw/giphy.gif" alt="Too stressed" width="300" height="250" />

### Cracking BOF - 25 points
It was 6.20 AM, I finally reached the place, sanitized myself, and unpacked. Showed the proctor my environment again, got the green light to continue with my exam. Started AutoRecon on the 25 pointer again, focussed on BOF, and cracked it by 7 AM. I've got 25 points in 3.5 hours, 45 points more, any 3 more boxes to crack. It was time to focus on the other 25 pointer, luckily AutoRecon was done. Ran AutoRecon on one of the 20 pointer machine. Checked my directory for the 25 pointer recon results....aaaand it's gone. The new scan command probably overwrote it. *Internal screaming*. Chucked AutoRecon entirely, moved to nmapAutomator. Best move so far.

Ran nmapAutomator on the 25 pointer machine, it printed out its finding as it went on, perfect. Poked at the open ports found one by one, looked for publicly known exploits against any service or name I was coming across using `searchsploit` and Google, and ran enumeration scripts. Consecutive scanning proved a whole lot better than parallel scanning. Couldn't find anything on the 25 pointer, moved on. Ran nmapAutomator on the 20 point machine, poked open ports one by one, repeated the enumeration process, again found nothing. Scanned the other 20 point machine, poked open ports, repeated the enumeration process, landed no where. It was noon by then and I only had 25 points so far. Needed some fresh air.

<img src="https://media.giphy.com/media/l41Yxij9zr2UI8wx2/giphy.gif" alt="Scrambling" width="500" height="300" />

### Cracking 10 pointer
I took a break for few minutes, I realized I had not looked at the 10 pointer machine yet, thought "welp here goes nothing." Got back from the break, scanned 10 pointer machine, performed the enumeration process, found the vunlerable thing! ╰(*°▽°*)╯. Pwned the machine in 20 minutes. Checked enumeration results of the other machines again, nothing clicked. It was 1 PM and I had 35 points, 35 more to go. 

### Cracking 20 pointer
Took a lunch break for 30 minutes. Checked the enumeration results on one of the 20 pointer, saw a promising exploit, there were 2 versions: manual and metasploit. I was not sure if I should blow my metasploit chance yet. In retrospect, I should've and you'll soon realize why. I was up for around 10 hours now, and I was already tired cause of all the gymnatics and so stressed you could see it physically. I checked the manual exploit, it was written in broken english but the steps were obvious, tried to follow it, did not work, but had a strong feeling that it would (hint: msf module). I tried various patterns of it without achieving anything for a **couple hours**. I got so tired I execute the simplest thing I could imagine, should have done that as my first try, and decided to move on if that won't work. IT WORKED! Realized why my previous exploits weren't working and felt like a total moron. 

Updated my exploit according to the new information, got a low shell but was highly unustable. Checked the MSF module, decided to make my exploit as per the MSF module instead of using Metasploit itself. It worked! Got a low shell, escalated my privileges in 5 minutes. Done. It was 7 PM, and I had 55 points. Time for another break.

### Cracking 25 pointer
Got back from the break, checked my enumeration results again, realized this is all the data I had to work with. Again checked for publicly known exploits for a particular thing, but I did not know it's version. I was looking for exploits with that exact name of the thing for too long (bad move), with no results. Felt like giving up, tried the generic name exploit and IT WORKED! I already had that information since 7:30 AM, I wasted away all these hours *sigh*. Got a low shell by midnight.

Ran the local enumeration script on the system, pretty quickly identified a lot of rabbit holes. But coud not find the way to escalate my privileges, spend 2 more hours just scrambling. Realized the privilege escalation was right in front of my eyes, again it wasn't anything complex. Some non-standard application on system. Looked up publicly known exploit, one entry "Software name privilege escalation". *facepalm*. 

Tried to get the exploit to work with just 40 minutes to go. Unable to get it to execute as per what I wanted. Something was off. Tried to follow the exploit steps manually, I could follow them, but it wasn't working. Maybe a rabbit hole? Nope, it couldn't be, it was the only thing that stood out, or atleast that's what I felt. Realized it was based on something, modified that, followed the steps again. Get a message on the side from proctor that my time is up and that VPN has been closed. I couldn't escalate my privileges in time, neither could I say I found the right thing.

24 hours - 65 points
<img src="https://media.giphy.com/media/jOpLbiGmHR9S0/giphy.gif" alt="Almost had it" width="500" height="300"/>

## Reporting
I wrote both my exam report and lab report and submitted them as per instructions by 2:30 AM the next day. I was hoping to get the 5 bonus points and clear the certification. Surprisingly, I didn't have to wait long and get my result in 2 days. I've successfully obtained the certification!

<img src="https://media.giphy.com/media/l4EpkVLqUj8BI7OV2/giphy.gif" alt="The victory is mine!" width="500" height="300" />

## Takeaway 
Things to keep in mind:
* I want you to know that this exam is *EASY*. I know the margin by which I cleared, yet still I'll say the same. If it wasn't for my overthinking and external failure, it'd have been a walk in the park
* This exam is *INTENDED* to be completed in 12 hours. Good thing we have 24

### Improved Enumeration Process
* nmapAutomator each IP consecutively
    * `nmapAutomator $IP All`
* Check each port for as much info as you can
* Run the usual enumeration scripts against the services and system
    * Use different wordlists
* `searchsploit` each service, web application, or odd software you come across
    * Look at valuable exploits from the entire result
    * Something that can give you either more information or increase your privileges
        * Arbitary file upload
        * Arbitary file download
        * Code execution
        * LFI
        * RFI
        * SQLi
    * Once these exploits have been identified,
        * Either try them all out one by one
        * Or filter by version, then try whatever is left

### Do
* Stay calm
* Take breaks frequently
* Stay structured
* Sleep
* Understand the exploit
    * Make a checklist of it's underlying requirement for it to work
    * Execute the simplest command you can execute first
    * Make a note of parameters it's using

### Don't
* Scan all IPs at once - overloading network will give you incomplete results
* Panic
* Overlook exploits because names do not match *exactly* (This costed me over 5 hours)
* Be afraid to use your Metasploit card (This costed me 4 hours)