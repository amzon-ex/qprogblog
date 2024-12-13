---
title: DNS setup in home server with AdGuard Home
date: 2024-12-10 10:45
draft: false
categories: workflow
tags:
    - DNS
    - networking
    - homelab
---


[//]: # (The body)

The home server I have been setting up recently uses [AdGuard Home][agh] (AGH) as a DNS server and DNS-level adblocker. There are three things I wanted to achieve with my setup:
1. Use AdGuard Home as a DNS server with *upstream* queries via DNS-over-HTTPS (DoH),
2. Use [Nginx Proxy Manager][npm] (NPM) as a reverse proxy to assign *local* domain names to services running on different ports of my server, and
3. Resolve local hostnames with `.lan` as the TLD.

## AdGuard Home as DNS server with DoH

This is pretty straightforward - use upstream servers that support DoH. As of now, I do not care to encrypt my local DNS queries (client → AGH). I do not want to set up DNS servers on each device I use, so I set it directly on the router, which runs [OpenWRT][owrt]. However, to avoid a DNS loop - I want the DHCP server on my router to point to AGH as a DNS server, instead of indicating it in the DNS settings of my router. A comprehensive guide on OpenWRT DHCP configuration can be found [here][owrt-dhcp].

I first log into my router interface via SSH. To check the current DHCP config, we use
```shell
uci show dhcp
```
We check for any pre-set `lan.dhcp_option`. If set, we can delete it by running
```shell
uci -q delete dhcp.lan.dhcp_option
```
Now we can assign custom DNS servers by setting DHCP option 6[^dhcp-6] ([reference][dhcp-opt]):
```shell
uci add_list dhcp.lan.dhcp_option="6,ip.of.dns.srv"
uci commit dhcp
```
and then we restart **dnsmasq**:
```shell
service dnsmasq restart
```
For clients to start using the new DNS server and newly assigned DNS addresses, we would need to reconnect them, or restart the router.

## NPM to assign domain names to local services

This scheme is pretty simple too - say we wish to host service **foo** and **bar** at `foo.hlab.com` and `bar.hlab.com` respectively, i.e. all services get a suffix `hlab.com`. We achieve this using *DNS Rewrites* on AGH. We go to *Filters > DNS rewrites* and add a rule:

1. Wildcard domain name: `*.hlab.com`.
2. IP address or domain name: `ip.of.npm.srv`

where `ip.of.npm.srv` is the IP address where NPM is running and listening on port 80. Now we go to NPM and add a *Proxy Host*:

1. In "Domain Name(s)" we enter the service address: `foo.hlab.com`.
2. In "Forward Hostname / IP" we enter the server address where **foo** is running: `ip.of.foo.srv`.
3. In "Port" we enter the port where a WebUI or similar of **foo** is hosted. 

I have a fairly simple setup with a single laptop acting as a server, so for me, `ip.of.dns.srv`,`ip.of.npm.srv` and `ip.of.foo.srv` are all the same. 

Anyway - we're done! Now when we type `foo.hlab.com` in the address bar, the domain name will be sent to AGH for resolution, which will send the request to NPM. NPM will appropriately redirect to the port on the server **foo** is running.

## Local hostnames with `.lan` as the TLD

The downside of switching to AGH as the DNS server is that we lose the ability to refer to devices on the server with their hostnames, like `mypc.lan`. Of course, `.lan` is an example - you could use (almost) anything else. To achieve this in OpenWRT, one goes to *Network > DHCP and DNS > General* and changes the value of *Local Domain*.

>[!note]
>Not all TLDs are recommended, or work well and may depend on your use case. This [wiki][tld-rsv] lists some TLDs that are reserved in the *global* domain name system. This [wiki][domain-rsv] lists some domain names one should avoid using. [This][tld-reddit] reddit post details issues with using certain domain names that you might run into. For instance, if using [Let's Encrypt][lenc] for SSL certification (common if using NPM), one will run into a conflict with `.lan` TLDs. Moreover, your browser might refuse to work well with some (or many) TLDs, ending up searching instead of a name lookup.

When OpenWRT is the DNS server, you could access machines via `hostname.lan` (the TLD can be customized). This is handy if you do not want to remember IP addresses, or if your addresses are expected to be dynamically assigned by DHCP, so will keep changing. 

With AGH, it is often advised to put the names and their corresponding records in *Custom Filtering Rules* or *DNS Rewrites*. I didn't want to do this because I want the hostnames that are available on the network to be used automatically, as was the case with OpenWRT as the DNS server. I do not wish to write a rule for every single IP on my network. 

The solution, as it turns out, is pretty simple. In AGH, we can specify different upstream servers for specific domains, at different levels of specificity. The one we're interested in would require adding a line to our upstream DNS servers section:
```
1.1.1.1
...
[/lan/] ip.of.owrt.rtr
```
Now all `*.lan` requests are sent to `ip.of.owrt.rtr` - which is the IP address of our OpenWRT router. This is **dnsmasq**-like syntax and can be adapted depending upon the level of specificity desired - examples are in the [wiki][agh-ups-dom].




[//]: # (Footnotes, if any)

[^dhcp-6]: For IPv4 networks: Carries the IP address(es) of the DNS servers that the client uses for name resolution.





[//]: # (Links, if any)

[agh]: <https://adguard.com/en/adguard-home/overview.html>
[npm]: <https://nginxproxymanager.com/>
[owrt]: <https://openwrt.org/>
[owrt-dhcp]: <https://openwrt.org/docs/guide-user/base-system/dhcp_configuration#providing_custom_dns_with_dhcp>
[dhcp-opt]: <https://www.incognito.com/tutorials/dhcp-options-in-plain-english/>
[tld-rsv]: <https://en.m.wikipedia.org/wiki/Top-level_domain#Reserved_domains>
[domain-rsv]: <https://en.m.wikipedia.org/wiki/Special-use_domain_name>
[tld-reddit]: <https://www.reddit.com/r/selfhosted/comments/17wj2qz/what_toplevel_domain_do_you_use_in_your_local/>
[lenc]: <https://letsencrypt.org/>
[agh-ups-dom]: <https://github.com/AdguardTeam/Adguardhome/wiki/Configuration#specifying-upstreams-for-domains>
