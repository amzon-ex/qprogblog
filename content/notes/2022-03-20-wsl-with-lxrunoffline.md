---
layout: post
title:  "Managing WSL distros with LxRunOffline"
date:   2022-03-20 12:00:00 +0530
categories: tweaks
tags:
    - wsl
    - lxrunoffline
---
[//]: # (The body)

**LxRunOffline** is a tool for managing WSL distributions.

Can be installed using **chocolatey**:
```powershell
choco install lxrunoffline
```
### Installing distros

Detailed information in the [github repo wiki][1]. Installation of a distro can be done as follows:
```powershell
LxRunOffline i -n {custom-distro-name} -d {install-location} -f {name-of-tar-file}
```
where the tar file has been downloaded from <https://lxrunoffline.apphb.com/download/{distro}/{version}> for the required distribution *or* is a previously exported distro (using `wsl --export` or LxRunOffline itself).

The installed distros can be viewed with `LxRunOffline l`. For detailed summary of an installed distro:
```powershell
LxRunOffline sm -n {custom-distro-name}
```

Unfortunately, **LxRunOffline** installs distros using WSL1. If one has to upgrade to WSL2, one must run
```powershell
wsl --set-version {custom-distro-name} 2
```
This conversion, however, may fail, or take an insanely long time depending on the size of the distro and other *undocumented* reasons (?) as detailed in [github:microsoft/WSL:issue#5344][2] or [github:microsoft/WSL:issue#4626][3].

For a fresh install this should work. (?) Of course, there is a lot of guessing here as I don't know the exact reasons. For me, importing an old distribution with **wsl** failed - and after installing it with **LxRunOffline**, **wsl** still failed to convert it to version 2, stating
```
Conversion in progress, this may take a few minutes.
Importing the distribution failed.
```
after some time. <span style="color:red">*TO BE INVESTIGATED.*</span>

### Setting the user

If installed this way, the distro will probably default to the `root` user. To avoid this, we can create a new user from `root`. First, we list the available users. This is stored in */etc/passwd*:
```bash
$ cat /etc/passwd
```
or to just get the usernames and the user ids:
```bash
$ awk -F: '{print $1,$3}' /etc/passwd
```
where we specify the separator `:` using `-F` and print only the first entry `$1` (which is the username) from the file. As we can see from the first output, the `root` user has id 0. The default login user usually has id 1000. There are many users with non-login shells as well, not of interest. 

Now to create an user:
```bash
$ adduser {user-name}
```
after which we will be prompted to enter the password and user information. Now we can re-list the users to check the user id (this should be 1000 if there weren't any users already).

Finally, we shutdown the wsl distro and set the default user[^setdefuser]
```powershell
LxRunOffline su -n {custom-distro-name} -v {user-id}
```
where `su` stands for *set user*. The next time we open the distro, we will login as this user by default.






[//]: # (Footnotes, if any)

[^setdefuser]: This is only for imported/custom-installed distros. For a distro installed using the default mode `wsl --install` this can be done using 
    ```powershell
    {distro-name} config --default-user {user-name}
    ```
    from Powershell/cmd.




[//]: # (Links, if any)

[1]: <https://github.com/DDoSolitary/LxRunOffline/wiki>
[2]: <https://github.com/microsoft/WSL/issues/5344>
[3]: <https://github.com/microsoft/WSL/issues/4626>