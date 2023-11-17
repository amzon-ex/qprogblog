---
title: SSH through an intermediate host
date: 2023-11-11 19:21
categories: workflow
tags:
  - ssh
  - commandline
  - password
---
[//]: # (The body)

When we have to connect via SSH to a remote machine, but we have to go through an intermediate host, it can feel tedious. There are many efficient ways to do it. This workflow derives from various tips picked up from various sources. This workflow assumes we've already set up ssh keys. It might also be worth looking at [[notes/2022-11-03-ssh-keychain|how to ask for ssh passphrases on demand]].

## Add public keys to remote host

To avoid the pain of having to type passphrases all the time, we use ssh keys. After generating the public-private key pair, we need to copy and authorize the public key to our remote host, which we wish to connect to. We assume that we have an account on the remote host and can log in to it using a password. For other use cases, [this tutorial][copy-keys] is useful.

We run the `ssh-copy-id` tool, which creates all necessary directories, files and sets appropriate permissions.
```shell
ssh-copy-id -i /path/to/key.pub username@hostname
```
The way to manually do this would be to append the public key string to a file named *authorized_keys* in the *.ssh* folder on the remote host and set appropriate permissions - `700` for *.ssh* and `600` for *authorized_keys*. 

## Access remote host via jump host

To start with, we want to connect with our remote host directly, without ssh-ing multiple times through all intermediate hosts (which we call **jump hosts**). We mostly follow [this guide][ssh-jump-host]. Instead of ssh-ing into the jumphost, and then ssh-ing into the destination, we use the switch `-J` and run
```shell
ssh -J user@jumphost username@destination
```
It is possible to add multiple jumphosts using commas, as `-J user1@jumphost1,user2@jumphost2, ...`. To avoid having to do this all the time, we can put this in the ssh configuration file. The user config is typically located at *$HOME/.ssh/config* and this is the one we will modify.
``` {title="~/.ssh/config"}
Host somename
	User user
	Hostname destination
	ProxyJump user@jumphost
```
Now, whenever we run
```shell
ssh somename
```
The connection will automatically be routed through the jumphost. Once this is set up, we can even use `scp` to move files around with ease, like so:
```shell
scp /path/to/file somename:/dest/loc/of/file
```
without the config file set up, we'd have to mention jumphosts with the `-J` key, as usual.

We could be done here - but there's a convenience issue. Since we have more than one host we want to ssh to in this discussion, we would have to create key pairs for each connection pair, or propagate the keys in our source machine to all machines in the chain of access. However, even after that has been done, the `ssh-agent` on each machine has to be running in order to authenticate, say, file transfers without repeatedly asking for passwords/passphrases. 

### Agent forwarding

The solution to the above-described problem is to use **agent forwarding** (a great explanation can be found [here][agent-fwd]). If the connection goes like
```mermaid
flowchart LR
A[source A] --> X[jumphost X] --> Y[destination Y]
```
If agent forwarding is set up, when X connects to Y is asked for authentication, X will forward this request to the `ssh-agent` running in A. This eliminates both the need to propagate our private key to all the hosts in the network AND type passphrases for each successive connection, or get the `ssh-agent` running on all the intermediate hosts.
To do this, we use the `-A` switch:
```shell
ssh -A -J user@jumphost username@destination
```
or add it to the config file for convenience:
``` {title="~/.ssh/config"}
Host somename
	User user
	Hostname destination
	ProxyJump user@jumphost
	ForwardAgent yes
```
which will automatically use agent forwarding when we connect to `somename`. 






[//]: # (Footnotes, if any)

[^fn]: Footnote





[//]: # (Links, if any)

[copy-keys]: <https://linuxhandbook.com/add-ssh-public-key-to-server/>
[agent-fwd]: <http://www.unixwiz.net/techtips/ssh-agent-forwarding.html>
[ssh-jump-host]: <https://tailscale.com/learn/access-remote-server-jump-host/>