---
title: powerview-cheatsheet
date: 2020-12-23 22:30:19
tags: [cheatsheet, powerview, active directory]
---

# PowerView Cheatsheet

This cheatsheet corresponds to an older version of PowerView deliberately as this is the version that was used in Pentester Academys' CRTP certification course. This cheatsheet will be updated to the latest version of PowerView soon.

## Helpful Commands

Commands to help use PowerView even better.

| Command | Description |
| ------------- | ------- | 
| `Set-MpPreference -DisableRealTimeMonitoring $true` | Disable Windows Defender real time monitoring |
| `Set-MpPreference -DisableIOAVProtection $true` | Disable Windows Defender scanning for all files downloaded |

Disabling Defender even if for a small amount of time puts the assets at risk, instead one could opt for bypassing AMSI using [this resource](https://amsi.fail/).

## Recon

### General Domain Enumeration

| Command | Description |
| ------- | ----------- |
| `Get-NetDomain [-Domain <target>]` | Get basic information of the domain |
| `Get-DomainSID` | Get the domains' SID |
| `Get-DomainPolicy [-Domain <target>]` | Get list of policies in the domain |
| `(Get-DomainPolicy)."kerberos policy"` | Get the Kerberos policy |
| `Get-NetDomainController [-Domain <target>]` | Get list of domain controllers |

### User Enumeration

| Command | Description |
| ------------- | ------- | 
| `Get-NetUsers [-Username <username>]` | Get a list of users |
| `Get-UserProperty [-Properties <property>]` | Gets a list of all user properties. Select a property to get its' value of every user |
| `(Get-NetUsers \| select name).count` | Get a count of users present |
| `Get-NetUsers \| select name,badpwdcount \| where badpwdcount` | Get-UserProperty alternative |
| `Get-UserProperty -Properties logoncount \| where logoncount \| sort logoncount -Descending` | Get users sorted with most logoncounts |
| `Get-LoggedOnLocal [-ComputerName <name>]` | Get who is logged on locally where |

### Computer Enumeration

| Command | Description |
| ------------- | ------- |
| `Get-NetComputer [-FullData] [-Ping] [-OperatingSystem "*server*"] [-Domain <target>]` | Get a list of all the computer OUs |
| `Get-NetComputer -FullData \| select name,logoncount,operatingsystem,badpwdcount` | Get list of computers with required info |

### Group Enumeration

| Command | Description |
| ------------- | ------- |
| `Get-NetGroup [-Domain <target>] [-FullData] [-GroupName "*admin*"] [-Username 'user_name']` | Get AD groups data either all or of a user |
| `Get-NetGroupMember [-GroupName 'group_name'] [-Recurse]` | Get members of a group |

### Share Enumeration

| Command | Description |
| ------------- | ------- | 
| `Invoke-ShareFinder -ExcludeStandard -ExcludeIPC -ExcludePrint` | Find interesting shares |

### GPO Enumeration
| Command | Description |
| ------------- | ------- | 
| `Get-NetGPO [-ComputerName <hostname.domain>]` | List all GPOs in the domain |
| `Get-NetGPOGroup` | Find interesting GPOs |
| `Find-GPOComputerAdmin [-ComputerName <FQDN_computer>]` | List users of local group using GPO |

### OU Enumeration

| Command | Description |
| ------------- | ------- | 
| `Get-NetOU [-FullData]` | Get OUs |
| `(Get-NetOU -Name 'test').gplink` | Get gplink of an OU to get GPOs applied to it |
| `Get-NetGPO -GPOName '{asd21321asdsd3as2d1}'` | Get GPO of a gplink |
| `((Get-NetOU -FullData <OU_NAME>).gplink -split "cn=" -split ",")[1] \| Get-NetGPO` | Get GPO of an OU using gplink |

### ACL Enumeration

| Command | Description |
| ------------- | ------- | 
| `Get-ObjectAcl [-Name 'domain admins'] -ResolveGUIDS` | Get the ACEs for a group |
| `Invoke-ACLScanner -ResolveGUIDS` | Find interesting ACEs |
| `Invoke-ACLScanner -ResolveGUIDS \| ?{$_.IdentityReference -match 'svcadmin'}` | Find interesting rights owned by 'svcadmin' (can be done for groups) |

### Trust Enumeration

| Command | Description |
| ------------- | ------- | 
| `Get-NetDomainTrust [-Domain <target>]` | Map all the domain trusts |
| `Get-NetForest [-Forest <target>]` | Get information about the specified forest |
| `Get-NetForestDomain [-Forest <target>]` | Get all the domains of a forest |

### User Hunting

| Command | Description |
| ------------- | ------- | 
| `Find-LocalAdminAccess` | Get list of all machines where current user has local admin access |
| `Invoke-EnumerateLocalAdmin` | Find all admins on all computers |
| `Invoke-UserHunter [-GroupName <group_name>] [-CheckAccess]` | Find machines where a domain admin has a session, checkaccess tells you if you also have access to that machine |


## Finding Sessions

| Command | Description |
| ------------- | ------- | 
| `Get-NetSession [-ComputerName <comp_name>]` | Get list of active sessions on a system |
| `Get-LoggedOnLocal [-ComputerName <comp_name>]` | Get list of users logged on a system |


## Privilege Escalation

| Command | Description |
| ------------- | ------- | 
| `Get-NetComputer -Unconstrained` | Find computers to perform uncontrained delegation |
