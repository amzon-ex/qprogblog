---
title: Client addresses in dockerized AdGuard Home without host mode networking
date: 2024-12-06 00:22
categories: workflow
tags:
    - networking
    - docker
    - homelab
---
[//]: # (The body)

## The Problem

I'm running [AdGuard Home][agh] (AGH) on my teeny-weeny home server via docker. More specifically, I'm using [rootless docker][rl-dock] as is recommended for security purposes (==insert citations==). With my initial setup, only the gateway address of the default container (say, `172.xx.0.1`) shows up in AGH query log. I want the specific client making the request to show up - and the commonly recommended solution online is to use **host** or **macvlan** mode networking. The macvlan mode is out of the question, as the [docs][mvl] clearly state at the time of writing:

> The `macvlan` driver is not supported in rootless mode.

Moreover, I'm unwilling to run AGH with host mode networking, as this complicates port mapping. However, the bigger problem here is - if I try to run AGH in host mode, it doesn't work - **all** the ports that you'd have expected to show up listening for traffic when you run, say,
```shell
sudo netstat -lntpu
```
are absent! So, what's happening here?

As it turns out, using rootless docker [is at odds with][rl-host-err] host mode networking - the rootless docker daemon is nested in a namespace *within* the `rootlesskit` network namespace. The recommended solution is to forward ports - but that's where I currently am, and don't want to be. The other solution is to run AGH with "rootful" docker - something I'm also trying to avoid.

If we take a closer look at the problem (which I did after stumbling upon [this github issue][git-issue]), we can refine the issue we're dealing with. Rootless docker uses network and port drivers for networking - available configurations are documented [here][rl-net]. The default for most systems is the `slirp4netns` network driver with the `builtin` port driver. Here lies the key issue - the `builtin` port driver does not support *source IP propagation*. Thus, when the container receives requests from outside, it seems like all the requests are coming through the gateway - the requesting IP address is not forwarded (==need to revise this==).

## The solution

The way out, in short, is use another configuration of network and port drivers. Looking at the table previously linked, it seems like [`bypass4netns`][b4ns] is the fastest solution, but it's experimental and does not have docker integration as of now, so we will avoid sweating too much.

### Using `slirp4netns`

My first approach was to use `slirp4netns` as the port driver. The advantage of this method is it doesn't require installing anything else - it's present on the system. The downside is that the port throughput speed is quite poor. For this, we add this to *~/.config/systemd/user/docker.service.d/override.conf* (and create it if required):
```ini
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=slirp4netns"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
```
Now we restart the daemon:
```shell
systemctl --user daemon-reload
```
and the docker service:
```shell
systemctl --user restart docker
```
This immediately solved my immediate problem - I was now able to see the addresses of clients making DNS requests in AdGuard Home!

Unfortunately, I ran into quite a few troubles trying this method out. I host [**homepage**][hmpg] as a dashboard for my server. It makes HTTP requests to APIs of running services to display their status or information from them in widgets. When I started using `slirp4netns` as the port driver, 
- Homepage started taking forever to load, with docker container statuses showing as `unknown` for a long time before they started loading.
- Many of my service widgets started throwing API errors, mostly with the code 500 and the message `getaddrinfo EAI_AGAIN`. [This answer][eai-err] on stackoverflow makes me think this might be because my requests are timing out. It could also be that my reverse proxy is not functioning as expected (I use [NPM][npm] as a reverse proxy to assign domain names to ports). 

#### Note about privileged ports

By default, rootless containers cannot bind to *privileged ports* (<1024). If one has [used][setcap] 
```shell
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
```
to get around this problem, they will face this problem again when using `slirp4netns` as their port driver. This is because `slirp4netns` [ignores the cap][capignore]. To solve this, we should use the other recommended solution - edit */etc/sysctl.conf* and add
```ini {title="/etc/sysctl.conf"}
net.ipv4.ip_unprivileged_port_start=0
```
and run
```shell
sudo sysctl --system
```
Even though I resorted to this method, I wonder if it is more insecure in general, since it involves editing system network configuration instead of adding capabilities to `rootlesskit` in a controlled way. 
### Using `pasta`

I splashed some olive oil and chopped garlic onto my container, and the pasta was ready... uhm, wait - 

`pasta`, which is part of the [`passt`][passt] project, can be used as a replacement for `slirp4netns` in our case. Network throughput does not improve, but source IP propagation comes as a bonus. However, we must install `passt` (which provides `pasta`) first. I'm on Ubuntu, so I run
```shell
sudo apt install passt
```
Now, we put the following in *~/.config/systemd/user/docker.service.d/override.conf*:
```ini
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=pasta"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=implicit"
```
We use the `implicit` port driver in this case. Then, the standard jam:
```shell
systemctl --user daemon-reload
systemctl --user restart docker
```
That's it - client addresses show up - and in this case, nothing breaks in **homepage**.

## References

More information on network and port drivers can be found at
- [rootlesskit/docs/network.md at v2.0.0 · rootless-containers/rootlesskit](https://github.com/rootless-containers/rootlesskit/blob/v2.0.0/docs/network.md)
- [rootlesskit/docs/port.md at v2.0.0 · rootless-containers/rootlesskit](https://github.com/rootless-containers/rootlesskit/blob/v2.0.0/docs/port.md)



[//]: # (Footnotes, if any)

[^fn]: Footnote





[//]: # (Links, if any)

[agh]: <https://adguard.com/en/adguard-home/overview.html>
[rl-dock]: <https://docs.docker.com/engine/security/rootless/>
[mvl]: <https://docs.docker.com/engine/network/tutorials/macvlan/>
[rl-host-err]: <https://docs.docker.com/engine/security/rootless/#--nethost-doesnt-listen-ports-on-the-host-network-namespace>
[git-issue]: <https://github.com/nextcloud/all-in-one/issues/4621#issue-2279565577>
[rl-net]: <https://docs.docker.com/engine/security/rootless/#networking-errors>
[b4ns]: <https://github.com/rootless-containers/bypass4netns>
[hmpg]: <https://gethomepage.dev/>
[eai-err]: <https://stackoverflow.com/a/40182520/12983399>
[npm]: <https://nginxproxymanager.com/>
[setcap]: <https://docs.docker.com/engine/security/rootless/#exposing-privileged-ports>
[capignore]: <https://github.com/rootless-containers/slirp4netns/issues/251#issuecomment-761415404>
[passt]: <https://passt.top/passt/about/>