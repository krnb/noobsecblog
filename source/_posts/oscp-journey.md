---
title: oscp-journey
date: 2020-06-27 10:26:58
tags: [oscp]
---

# OSCP Journey
From a persistent n00b who couldn't even hack a medium difficulty machine on his own to cracking OSCP in 4 months!

## Background
I wanted to do the PwK course and clear OSCP since past couple years but haven't been able to due to reasons. Two of my certifications were going to expire in August '20, and I had to do a certification to renew them, a perfect opportunity. Before I started with the PwK course, I had very little experience with hacking or even CTF for that matter. My college studies or previous certifications weren't of any help here either. But I decided to take a deep breath, and dove in.

## Prep
My course and lab access started in January, it was the old course, I completed all of my exercises and was only able to crack 14 machines in the Public Network. Not good by any standards, but it was just a start. I was glad I was able to do enough to make my lab report. Once my lab access got over, I took a break of 2 months, focussed on college.

Two months later, in March, I decided to start preparing for the exam. I came across [OSCP-like machines list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit) by TJ_Null which looked quite promising and so signed-up for the HackTheBox VIP platform.

I went through the list, watched IppSec's video of the machine I'm targetting, and hacked it and continued that for almost every machine. I made a report for each machine I hacked and documented each step including why certain steps were taken and the reasoning behind it. This was essential as I was under a time crunch, needed to build my hacking methodology, and make it as robust as I possibly can. Apart from watching IppSec's video, I read walkthroughs by various people of the same machine to get more information or learnings out of it. 

And that was it, and that's all the practice you need.
**HACK EVERY SINGLE MACHINE ON THAT LIST!!!** You'll thank yourself later.

## Exam
I had scheduled my exam at 3.30 AM, everyone is asleep, can peacefully work on my stuff without any external disturbances. Proctoring started at 3.15 AM, took a little time but went smoothly. Started with my exam at 3.45 AM.
I decide to first get done with Buffer Overflow, easy 25 points. Start AutoRecon on the other 25 point machine in the background. Run first few commands, and suddenly no output from the debugging machine, completely frozen. Huh? Maybe some issue with rdesktop, try to kill it, it won't die. Check my connection, seems to be re-connecting. Ugh. Kill the connection, try to connect to the VPN again, it's trying to connect. I look down at my taskbar at the internet connection icon, and there it was my worst nightmare, the internet has crashed. It's 4 AM, I won't be getting any support from the ISPs' end. FUCK. I take my mobile, turn the hotspot on, connect to the proctoring software, inform the proctor that my internet has crashed and I'm changing my location. They inform me they would pause the VPN, I agree.

I called a bunch of folks where I could continue the rest of the exam from, some places are completely locked down, can't go there. Can go to a relative's place, 50 minutes far by car, their building has a bunch of Covid-19 cases in there, and a whole lot in a hospital near that place. Fuck me. Look for cabs, luckily I find one, book it immediately, pack my shit. It's 5 AM, I've got nothing, chances are I could catch the virus, not the best of odds.

<img src="https://media.giphy.com/media/xU1spRleFHmtjvskXw/giphy.gif" width="300" height="250" />

### Cracking BOF - 25 points
It's 6.20, I finally reach the place, sanitize myself, unpack. Show the proctor my environment again, get the green light to continue with my exam. Start AutoRecon on 25 point again, focus on BOF, crack it by 7 AM. 25 points in 3.5 hours, 45 points more, any 3 more boxes to crack. Time to focus on the other 25 point box, luckily AutoRecon is done. Run AutoRecon on a 20 point machine. Check my directory for the 25 point recon results....it's gone. The new scan probably overwrote it. *Internal screaming*. Chuck AutoRecon entirely, move to nmapAutomator. Best move so far.

Run nmapAutomator on 25 point machine, it prints its finding as it goes on, perfect. Poke open ports one by one, searchsploit-ing any service or name I'm coming across, running enumeration scripts. Consecutive scanning proved a whole lot better than parallel scanning. Can't find anything on this, move on. Run nmapAutomator on a 20 point machine, poke open ports one by one, repeat the enumeration process, again nothing. Scan the other 20 point machine, poke open ports, repeat the enumeration process, land no where. It's noon now, I only have 25 points so far. Maybe time to consider life choices?

<img src="https://media.giphy.com/media/l41Yxij9zr2UI8wx2/giphy.gif" widtg="500" height="300" />

### Cracking 10 pointer
I take a break for few minutes, I realize I haven't looked at 10 point machine yet, welp here goes nothing. Get back from break, scan 10 pointer machine, perform the enumeration process, found the vunlerable thing ╰(*°▽°*)╯. Pwned the machine in 20 minutes. Check enumeration results again, nothing clicks. It's 1 PM and I have 35 Points, 35 more to go. 
### Cracking 20 pointer
Take a lunch break for 20  minutes. Check the enumeration on a 20 pointer, see a promising exploit, there are 2 versions: manual and metasploit. I'm not sure if I should blow my metasploit yet. In retrospect, I should've and you'll realise soon why. I'm up for around 10 hours now, and I'm already tired cause of all the mental gymnatics and so stressed you can see it physically. I check the manual exploit, it's written in broken english and steps are obvious, try to follow it, doesn't work, but have a strong feeling this will work(hint: msf module). I try various patterns of it without achieving anything for a **couple hours**. I get so tired I execute the simplest thing I can imagine, and decide to move on if that won't work. IT DOES! Realize why my previous stuff wasn't working and feel like a total moron. 

Update my exploit according to the new information I found, get a low shell but highly unustable. Check the MSF module, make my exploit as per the MSF module. It works! Get a low not so unstable shell, send myself another shell within the low shell I got. Escalated privileges in 5 minutes. Done. It's 7 PM, and I've 55 points. Time for another break.
### Cracking 25 pointer
Get back from the break, check my enumeration results again, realized this is all the data I've to work with. Again check for publicly known exploits for a particular thing, but I don't know it's version, have the approximate date of the machine. I was looking for exploits with that exact name of the thing for too long, with no results. Feel like giving up, try the generic name exploit and IT WORKED! I already had this result by 7:30 AM, I wasted away these many hours *sigh*. Got a low shell by midnight.

Ran the local enumeration script on the system, pretty quickly identified a lot of rabbit holes. But can't find the way to escalate privileges, spend 2 more hours just scrambling. Realize the privilege escalation was right in front of my eyes, again nothing complex that I was looking for. Some non-standard application on system. Look up publicly known exploit, one entry "Software name local privilege escalation". *facepalm*. 

Try to get the exploit to work with just 40 minutes to go. Unable to get it to execute as per what I want. Something's off. Try to follow the exploit steps manually, can follow them, but it's not working, maybe a rabbit hole? Nope, it's not, it is the only thing that stands out. Realize it's based on something, modified that, followed the steps again. Get a message on the side from proctor that my time is up and that VPN has been closed. I couldn't escalate my privileges in time.

24 hours - 65 points
<img src="https://media.giphy.com/media/jOpLbiGmHR9S0/giphy.gif" width="500" height="300"/>

## Reporting
I write both my exam report and lab report, submit it by 2:30 AM the next day, hoping to get the 5 bonus points and clear the certification. Surprisingly, I didn't have to wait long and get my result in 2 days. I've successfully obtained the certification!

<img src="https://media.giphy.com/media/l4EpkVLqUj8BI7OV2/giphy.gif" width="500" height="300" />

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