---
type: post
title: "Possible Tailscale Security Misconfiguration Firewalls"
---
## Preface
A few months ago I installed Tailscale on an Ubuntu 24.04 machine so I could access it remotely. It works great and has been a good solution for me as an alternate to setting up a personal VPN server and enabling port forwarding. I have used UFW with this device for the host firewall. After reviewing my host firewall settings I noticed that they weren't applied to devices connected through Tailscale. 

Tailscale advertises that it requires no firewall configuration, but I had thought this meant that the installation process would update the host firewall to allow a VPN connection and the service would bypass the boarder firewall built into my router. This was only somewhat the case. It turns out that Tailscale added a rule to iptables ( the underlying firewall tool used by UFW ). This rule authorized any connection coming from my Tailscale network to any port on the machine. This rule effectively ignored my rules in UFW. 

Additionally, I tested to see if the Windows Tailscale app would act differently. It appears that it also adds a rule to the Windows firewall allowing connections from my Tailscale network to any port. I have two database servers on my Windows machine.  Both servers were reachable by a simple Netcat test ( `nc -v 100.xxx.xx.xxx port`  ). The Windows firewall profile allowed neither of these servers to receive connections. 

## Solution Tailscale ACLs
Tailscale comes with a default access control list (ACL) that governs how devices, users and groups of users can connect. The default settings are very permissive allowing all devices in your network to connect to any other device on any port. This may present security risks if one of your connected devices is compromised. Just as an example, say you have a VM or container exposed to the internet which also happens to be connected to your Tailscale network ( tailnet ). Let's also say your personal computer, a server with family documents, or a server with sensitive information is on the same tailnet. You now have no firewall protection with the default ACL. Thankfully the ACL is easy to use and Tailscale provides good documentation. You can look it up [ here ](https://tailscale.com/kb/1018/acls) . 


