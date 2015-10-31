---
layout: post
title: Getting around a VPN tunnel to access the internet 
excerpt: VPNs sometimes kill your internet access; use a local VM to VPN from and keep your internet access available.
---

Unless the network you are accessing has split tunneling enabled, chances are connecting to the 
network via VPN is going to kill your local internet connection - meaning no email, chat, or even worse, 
no Google. I've had this problem several times and just dealt with it. Recently, it got to be a 
problem as I needed to be able to demo some applications internally while on a remote server on a 
VPN, but that VPN connection killed my internet. After doing some research and finding out that there 
is basically nothing I can do about the VPN settings myself, I decided to 
take matters into my own hands.
 
First, some assumptions: you can install VirtualBox and have access to a licensed copy 
of Windows (Windows 7 in my case). The solution? I installed VirtualBox on my local PC, created a Windows 
VM with very minimal resources, and installed the VPN client onto the new VM. Now, to VPN into the 
remote network, I fire up VirtualBox, start my VM, and then on the VM connect to the VPN and Remote 
Desktop into the remote server. I can now work on the remote server on the VPN and still have my 
local internet connection up along with email and chat.

I was concerned about the what the performance and draw speeds would be in a setup like this, but 
so far, the performance has been about as good through the VM as it is from my local PC.