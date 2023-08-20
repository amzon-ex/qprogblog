---
layout: post
title: "Ask SSH key passphrase on demand"
date: 2022-11-03 02:06
categories: workflow
tags:
    - ssh
    - keychain
    - wsl
---
[//]: # (The body)

Our core focus here is to be able to ssh without typing passphrases a million times, but not be forced to enter passphrases on login, if they're never required for the session. The job of an `ssh-agent` is to remember our passphrases. For a discussion on the merits and demerits of different approaches, [this stackoverflow answer][stof-ssh-types] is insightful.

## Start `ssh-agent` on login

There are many ways to start an `ssh-agent` on login. Here we explore two ways: one using a script and the other using small tool named `keychain`.

### Using a script

We can start the `ssh-agent` on login by adding a few lines to our profile files. This follows a modified version of [this stackoverflow answer][stof-autoagent]. This could be added to the */etc/profile.d* folder as a script, but it seems like it [doesn't work for zsh][asku-zshprof]. So we add this to our *~/.zprofile*. 

```shell {title="~/.zprofile"}
# Start the SSH agent only if not running
[[ -z $(ps | grep ssh-agent) ]] && echo $(ssh-agent) > /tmp/ssh-agent-data.sh

# Identify the running SSH agent
[[ -z $SSH_AGENT_PID ]] && source /tmp/ssh-agent-data.sh > /dev/null

# Kill ssh-agent on logout
trap 'test -z $SSH_AGENT_PID && eval $(/usr/bin/ssh-agent -k)' 0
```
Here we just check if the agent is already running - if not, we run it. Finally, when logging out, we kill the `ssh-agent`.

### Using `keychain`

The advantage of using [keychain][keychain-page] is it handles all the complications of adding keys and persisting them through logins if required. We will use it in a very limited manner, to only start `ssh-agent` reliably and persisting through logins if so desired.

We will first install `keychain`:
```shell
sudo apt-get install keychain
```
Now we will modify *~/.profile*, *~/.bash_profile* or *~/.zprofile* (whichever is appropriate) and add the following:
```shell {title="~/.zprofile"}
eval $(keychain -q --clear --eval --agents ssh)
```
This will start `keychain` on login and it will handle the addition of `ssh-agent`. We wouldn't have to worry about spawning multiple agents at the same time. `keychain` is designed to persist the ssh keys even after logout, until a reboot. However, we have used the `--clear` option, which clears ssh keys on logout.[^kc-clear] This is safer than the standard behaviour of keychain. The other options:
- `-q` suppresses info output of `keychain`.
- `--eval` prints lines to be evaluated on **stdout** (which is why we put `eval` in front)
- `--agents` specifies the kind of agent to start. 

## Adding keys on-the-fly

One could also provide names of keys to the script or `keychain`, to be automatically added on login. However, we wouldn't exploit this behaviour. We would instead use configure the `ssh-agent` to [automatically add keys][ssh-autokeys] to it.[^ssh-autokeys] To do so, we edit the configuration file *~/.ssh/config* (and create it if it does not exist):
``` {title="~/.ssh/config"}
Host *
	IdentityFile /path/to/file
	AddKeysToAgent yes
```
Here we add this for all hosts. It's definitely possible to do [advanced configurations][ssh-config] if required. One probably does not need to provide the `IdentityFile` if the name of the private key file is one of the "standard" names[^std-id-files]. Whether the ssh key is being picked up can be seen in the `ssh` output when used:
```shell
ssh -vvvv git@github.com
```
The `AddKeysToAgent` option is key here. *If* an `ssh-agent` is running (which we've ensured in the previous step), it adds a key to the agent when one is used. The [ssh changelog][ssh-v7_2] explains:

>\* ssh(1): Add an AddKeysToAgent client option which can be set to
   'yes', 'no', 'ask', or 'confirm', and defaults to 'no'.  When
   enabled, a private key that is used during authentication will be
   added to ssh-agent if it is running (with confirmation enabled if
   set to 'confirm').

More details can be found in the [ssh configuration page][ssh-conf-addkeys]. 

We're essentially done here. Now passphrases will be asked when required and once given, will be remembered by the `ssh-agent` for the session. 








[//]: # (Footnotes, if any)

[^kc-clear]: But lets cron jobs still use ssh keys after logout. 
[^ssh-autokeys]: This is a *recent* (!) addition (2016, SSH v7.2).
[^std-id-files]: These files are named after the protocol, for instance *id_rsa* or *id_ed25519*. It is possible to override this behaviour by adding the option `IdentitiesOnly yes` to the config file.





[//]: # (Links, if any)

[stof-ssh-types]: <https://unix.stackexchange.com/a/90869/428839>
[keychain-page]: <https://www.funtoo.org/Funtoo:Keychain>
[ssh-autokeys]: <https://unix.stackexchange.com/a/319964/428839>
[ssh-config]: <https://linuxize.com/post/using-the-ssh-config-file/>
[stof-autoagent]: <https://stackoverflow.com/a/24347344/12983399>
[asku-zshprof]: <https://askubuntu.com/a/476435/1049026>
[ssh-v7_2]: <https://www.openssh.com/txt/release-7.2>
[ssh-conf-addkeys]: <https://man.openbsd.org/ssh_config#AddKeysToAgent>