---
layout: post
title:  "Customizing the Terminal Prompt with oh-my-posh"
date:   2021-10-22 16:00:00 +0530
categories: tweaks
tags:
    - windows
    - powershell
    - prompt
    - oh-my-posh
---

The module **oh-my-posh** is the Windows equivalent of **oh-my-zsh** for Linux. It can help create pretty prompts which may provide visual aids with *git* (among other things?[^obs1]). I stumbled upon this idea on [this blog][1], but really, it's a very commonly used module. I'm going to use the **Windows Terminal**.

### Installing a NerdFont

**oh-my-posh** uses glyphs that are not present in standard fonts. To get them, we need to install one of many *NerdFonts* from the [website][2]. **Windows Terminal** by default uses *Cascadia* fonts.  I prefer to use *Fira Code* instead. So we can just search for *Fira Code* in the website (or download any font - doesn't really matter). The fonts will be downloaded in a *zip* format.

*Install* and check the name of the font in the font file. In my case, it is "FiraCode Nerd Font". Now we can go to Terminal settings by pressing `Ctrl+Shift+,` which will open the settings in JSON format, and add this to the `"profiles": "defaults":` key:
```
"font": 
            {
                "face": "FiraCode Nerd Font"
            }
```
This is the place where we can adjust the size and style of the font if desired, e.g `"size": 10`. Save and Terminal refreshes, now using the installed NerdFont.

### Shell-independent installation in Windows

From [this guide][3]. First, we need to install **oh-my-posh**. We can do this using **winget**, the package manager CLI in Windows:
```
winget install JanDeDobbeleer.OhMyPosh
```
Now we will have the `oh-my-posh.exe` application which we can use to customize our prompt. For WSL, this will be `oh-my-posh-wsl` instead. Themes are present at the location *~\AppData\Local\Programs\oh-my-posh\themes*. We can list all the theme names:
```
ls ~\AppData\Local\Programs\oh-my-posh\themes\
```
which are just JSON files with the name format **.omp.json*. To actually *view* the themes in action, however, we can run the following command:
```powershell
Get-ChildItem -Path "~\AppData\Local\Programs\oh-my-posh\themes\*" -Include '*.omp.json' | Sort-Object Name | ForEach-Object -Process {
    $esc = [char]27
    Write-Host ""
    Write-Host "$esc[1m$($_.BaseName)$esc[0m"
    Write-Host ""
    oh-my-posh --config $($_.FullName) --pwd $PWD
    Write-Host ""
}
```
which basically loops through all the JSON files in the folder and prints a custom prompt for each. To actually change the prompt, we can run
```powershell
oh-my-posh --init --shell pwsh --config ~/AppData/Local/Programs/oh-my-posh/themes/jandedobbeleer.omp.json | Invoke-Expression
```
in Powershell, or 
```bash
eval "$(oh-my-posh-wsl --init --shell bash --config $USERPROFILE/AppData/Local/Programs/oh-my-posh/themes/jandedobbeleer.omp.json)"
```
on *bash* in WSL.[^envvarnote]

To make changes permanent, we need to add this command to the profile of the respective shells (*$profile* for Powershell and *.bashrc* for bash).

#### Remarks

Even though it seems like a PITA[^pita] to write such a complicated path again and again (especially in WSL), once we've written it, all we have to do is modify the name of the theme (here `jandedobbeleer`) and not touch the path at all. However, if we are to [modify these themes][4] to our liking, it is wise to copy the JSON files to another location and make necessary changes, and *then* make the *shell profile* point to that location.

### Only for Powershell

To start with, we run:
```powershell
Install-Module oh-my-posh -Scope CurrentUser
```
in Powershell. This installs the **oh-my-posh** module for the current user (admin privileges not required?).






[^obs1]: For instance, it marks the root folder of this Jekyll website with a little ruby icon (Jekyll uses Ruby and the root folder has a *Gemfile*) or a Python symbol when python files are present.

[^envvarnote]: The environment variable `$USERPROFILE` does not exist by default. It has to be *forwarded* from Windows to WSL via the `WSLENV` variable:
    ```powershell
    setx WSLENV USERPROFILE/up
    ```
    and after a session restart, the Windows environment variable `%USERPROFILE%` containing the path to the user profile is available at `$USERPROFILE` in WSL (translated to a Linux-style path, the `/p` switch does this). ([Source][5])

[^pita]: Pain-In-The-Ass.







[1]: <https://zimmergren.net/making-windows-terminal-look-awesome-with-oh-my-posh/>
[2]: <https://www.nerdfonts.com/font-downloads>
[3]: <https://ohmyposh.dev/docs/windows>
[4]: <https://ohmyposh.dev/docs/configure>
[5]: <https://superuser.com/a/1546688/1171201>