---
layout: post
title:  "Adding a Powershell Profile"
date:   2021-10-22 15:00:00 +0530
categories: tweaks
tags:
    - windows
    - powershell
    - profile
---

The Powershell profile is equivalent to the *~/.bashrc* file for *bash* in Linux. It helps load a desired configuration when a Powershell instance is run.

To test if there is a profile:
```powershell
Test-Path $profile
```
The variable `$profile` refers to the profile. If absent, this returns `False`. If `False`, create one:
```powershell
New-Item -path $profile -type file -force
```
A typical (user) profile path looks like:
```powershell
$home\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```
where for me, `$home = C:\Users\Ayon\` - *Ayon* is the *username*.

Once created, we can use an editor to edit the file. I use *Sublime Text* (cli: *subl*) for the purpose:
```powershell
subl $profile
```

### Script running permissions:

Running scripts on Powershell might be disabled by default. From [this thread][1], we can use Powershell as Admin (*Win+X->A*) to change the *execution policy* of scripts:
```powershell
Set-ExecutionPolicy RemoteSigned
```
This worked for my use case. The `Unrestricted` flag can be used instead, but I haven't researched about when I should do so. <span style="color:red">*TO BE INVESTIGATED.*</span>





[1]: <https://answers.microsoft.com/en-us/windows/forum/all/whats-wrong-with-my-windows-powershell/f05e72f2-a429-4ee0-81fb-910c8c8a1306>
