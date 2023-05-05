---
layout: post
title:  "Customizing the Powershell Prompt"
date:   2021-10-22 15:30:00 +0530
categories: tweaks
tags:
- windows
- powershell
- prompt
---

Initially I was scanning through Google as usual and found some solutions. The key idea was taken from [this SOF thread][1]. One needs to use *Virtual Terminal Sequences* to do this. 

### Using *Virtual Terminal Sequences*:

From the Microsoft documentation on [this][2]:

> **Virtual terminal sequences** are control character sequences that can control cursor movement, color/font mode, and other operations when written to the output stream.

We're interested in **color/font mode** to change the appearance of our prompt. We need a sequence of the format
```
ESC[<n>m
```
There are two parts to this:
  - We need to the start with the *ASCII ESC character* (hex 0x1B). This can be achieved by writing the expression `$([char]27)` in Powershell.
  - For `<n>` we insert an appropriate integer code that corresponds to a particular formatting style. The entire list can be found in the same [doc page][3].

When we combine the two parts, we write something like:
```powershell
$([char]27)[36m
```
which applies *non-bold/bright <span style="color:rgb(88,209,235)">**cyan**</span> to foreground*. The text that follows will acquire this style. To negate all styles we use `ESC[0m`.

#### Custom colours:

With `ESC[38;2;<r>;<g>;<b>m`(`ESC[48;...`) one can use any *(r, g, b)* value for the text foreground (background). For instance, *(250, 128, 114)* is the <span style="color:rgb(250,128,114)">**salmon**</span> colour.

### Changing the prompt permanently

We modify the Powershell *profile* file to make changes permanent. ([Creating profiles][4]) We add the following lines:
```powershell
$ESC = [char]27
function prompt {
    $(if (Test-Path variable:/PSDebugContext) { '[DBG]: ' }
      else { '' }) + "$ESC[38;2;250;128;114mPS@$ESC[38;2;127;255;255m$ESC[4m" + $(Get-Location) + "$ESC[24m" +
        $(if ($NestedPromptLevel -ge 1) { '>>' }) + "> $ESC[0m"
}
```
where
  - We have stored the *ESC* character in a variable to ease our life.
  - The `prompt()` function is used to modify the prompt. The code within the `prompt()` function has been obtained from the [doc][5] on Powershell prompt.
  - The *built-in prompt* has been modified[^wrongsol] by inserting sequences at appropriate places. Currently, it should look like this:

  ![](/res/pwsh_prompt_211022/newprompt.png)

For changes to take effect immediately, source the profile using `. $profile`.






[^wrongsol]: The original SOF solution (and many other places) give this solution:
    ```powershell
    function prompt  
    {  
        "$ESC[93mPS $ESC[36m$($executionContext.SessionState.Path.CurrentLocation)$('>' * ($nestedPromptLevel + 1)) $ESC[0m"  
    }
    ```
    but `$('>' * ($nestedPromptLevel + 1))` causes issues. More specifically, the character **m** appeared automatically after the prompt when I was typing a period `.` which is undesirable behaviour. <span style="color:red">*TO BE INVESTIGATED.*</span>






[1]: <https://superuser.com/a/1259916/1171201>
[2]: <https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences>
[3]: <https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#text-formatting>
[4]: <{% post_url /pwsh_profile_211022/2021-10-22-pwsh-profile %}>
[5]: <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_prompts?view=powershell-7.1#built-in-prompt>