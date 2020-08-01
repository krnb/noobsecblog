---
title: Linux Tips And Tricks
date: 2020-07-29 16:05:00
tags: [linux, misc]
---

# Linux Tips And Tricks To Improve Your Workflow
A blog post covering some commands and aliases to help make your life easier.
Tl;dr at the bottom for a quick reference.

## Shortcut Keys

### Command shortcuts

Cycle through the last arguments of previous commands - `ALT + .`

![Argument Cycling](/linux-tips/alt.gif)

Run the last command again - `!!`

![Running Again](/linux-tips/runagain.png)

Run the last command with sudo - `sudo !!`

![Run Again With Sudo](/linux-tips/runagain1.png)

### Going back and forth
Go back where you came from
`cd -` : go to previous working directory

Go to home directory
`cd` : takes you home

![Directory Switching](/linux-tips/goback.png)

### Moving around
Instead of holding down on arrow keys, make use of the following:

`CTRL + arrow` left or right to move around words
`CTRL + a` : move to start of the command
`CTRL + e` : move to the end of the command
`ALT + backspace` from the last character of a word to remove it
`CTRL + delete` from first char of a word to delete it

![Moving Around](/linux-tips/moving.gif)

### Looking up faster

`CTRL + r` : recursively search the history

![Recursive Search](/linux-tips/rev1.png)

Once you found the command, you can either press Enter to execute the command. But let's say you don't want to execute it as is and want to add some arguments to that command. You can press Tab to put it on your screen to work with.

![Working On Previous Command](/linux-tips/rev2.png)

### Clearing the screen

`CTRL + l` : clear the screen

## Aliases

### System alias

Sudo alias is required to be able to use sudo functionality with aliases.

``` bash
alias sudo = "sudo "
```
Example is covered in the python server section

### Directories

Automatically make parent directories if need be

``` bash
alias mkdir='mkdir -p'
```

![Creating Parent Child Directories](/linux-tips/mkdir1.png)

Make multiple child directories.

![Creating Multiple Child Directories](/linux-tips/mkdir2.png)

### View Files

View all files with human readable file sizes.

``` bash
alias ll='ls -alh'
```

### Launching VPNs

Launching HTB VPN with a single command

``` bash
alias htblab='sudo openvpn ~/Documents/noobsecdotnet.ovpn'
```

![Connection Initiated](/linux-tips/htb1.png)

![Connection Established](/linux-tips/htb2.png)

If you're working with similar lab or a course like offsecs' PWK, you could make an alias like that. Also you can launch the lab from anywhere, with a single command which is nice.

Below are the aliases I used in my PWK lab for connection purposes:
``` bash
alias pwklab='sudo openvpn ~/Documents/pwk/pwk.ovpn'
alias rdpwin='rdesktop -g 75% -u OS-noob -p supersecurepassword 10.10.10.10'
```

### Get IPs

Lists only IPs of all the active connections.

``` bash
alias myip="echo;ip -c a | grep -w 'inet' | cut -d'/' -f1;echo"
```

![IP List](/linux-tips/myip.png)

If you don't like just printing out the IPs and want their respective interfaces as well, you could use the following:

``` bash
alias myip="echo;ip -c --brief addr | awk '{print \"\t\" \$1,\"\t\",\$3}';echo"
```

![IP List w/ Interfaces](/linux-tips/myip1.png)

### Python Server

``` bash
alias pysrv="python3 -m http.server"
```
By default the server would start on port 8000

![Python Default Server](/linux-tips/pysrv1.png)

Arguments can be appended as normal, if port requires sudo, command can be executed with sudo as well (this is where the sudo alias comes in handy)

![Python Server On Port 80](/linux-tips/pysrv2.png)

### SMB Server

``` bash
alias smbsrv="sudo impacket-smbserver"
```
![SMB Default Server](/linux-tips/smbsrv.png)

## Exports

Exports can work as your constant variables, if there are certain wordlists that you often use, typing out the entire path of the wordlist can get cumbersome, this is where exports can give you a hand.

``` bash
export WORD1="/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt"
```

To utilize the export you just created, you'll use it as any other bash variable. Let's take an example of `gobuster`:

```
gobuster dir -u http://10.10.10.13 -w $WORD1
```

Similarly more exports can be set up to help improve your workflow.

## TL;DR

List of all the **commands**:

| Command | Function |
| -------- | --------- |
| `ALT + .` | Cycle through the last argument |
| `!!` | Execute previous command |
| `sudo !!` | Execute previous command with sudo |
| `cd -` | Go to the previous working directory |
| `cd` | Go to home |
| `CTRL + left/right arrow` | Move left or right between words |
| `CTRL + a` | Move to the start of the command |
| `CTRL + e` | Move to the end of the command |
| `CTRL + DEL` | Delete words |
| `ALT + Backspace` | Remove words |
| `CTRL + r` | Search history recursively |
| `CTRL + l` | Clear screen |


**Aliases** :

| Alias | Function |
| ----- | -------- |
| `alias sudo='sudo '` | To be able to use sudo with aliases |
| `alias mkdir='mkdir -p'` | Make parent-child directories |
| `alias ll='ls -lah'` | List all files with human readable sizes |
| `alias htblab='sudo openvpn ~/Documents/noobsecdotnet.ovpn'` | Launch HTB VPN |
| `alias pwklab='sudo openvpn ~/Documents/pwk/pwk.ovpn'` | Launch PWK VPN |
| `alias rdpwin='rdesktop -g 75% -u OS-noob -p supersecurepassword 10.10.10.10'` | Launch test Windows machine |
| `alias myip="echo;ip -c a \| grep -w 'inet' \| cut -d'/' -f1;echo"` | List of IPs |
| `alias myip="echo;ip -c --brief addr \| awk '{print \"\t\" \$1,\"\t\",\$3}';echo"` | List of IPs with their interfaces |
| `alias pysrv="python3 -m http.server"` | Launch python server |
| `alias smbsrv="sudo impacket-smbserver"` | Launch SMB server |

**Exports** :

| Exports | Function |
| ------- | -------- |
| `export WORD1="/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt"` | Accessing wordlists easily |
