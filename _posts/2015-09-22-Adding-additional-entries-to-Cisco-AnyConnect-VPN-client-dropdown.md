---
layout: post
title: Adding additional entries to Cisco AnyConnect VPN client dropdown 
excerpt: Being able to quickly jump from one VPN to another is nice. If you're using the Cisco AnyConnect client, learn how to add entries to the VPN dropdown in the UI.
---

I'm constantly switching between VPNs daily, and have been using the Cisco AnyConnect VPN client 
for some time now. There is a dropdown in the UI, but no where in the settings to add additional 
VPNs to that dropdown. 

![_config.yml]({{ site.baseurl }}/images/2015-09-22-vpn-ui.png)

I just never really thought to look into it. I would just type in the address 
to the VPN I was connecting to and get on with it. Then one day, I decided I'd had enough of that, so 
I did some Googling and the following is what I pieced together from assorted forum postings.

The key here is you have to create XML configuration files (yes, two files) for each connection you are 
wanting to add. These files are pretty simple and straightforward - you can make copies of the existing 
ones and then make the minor changes. The XML config files live in 
``C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client\Profile``:

![_config.yml]({{ site.baseurl }}/images/2015-09-22-xml-directory.png)

Here my existing ones I started out with are the ones that begin with "GISiAnyConnect". Make copies of those and 
rename them, following the naming scheme of your originals.

Next, make edits to your config files as follows:

{% highlight xml %}
<ServerList>
    <HostEntry>
        <HostName><name to appear in dropdown></HostName>
        <HostAddress><IP address></HostAddress>
        <PrimaryProtocol>SSL</PrimaryProtocol>
    </HostEntry>
</ServerList>
{% endhighlight %}

Change ``HostName`` and ``HostAddress`` for sure. You may have to change ``PrimaryProtocol`` from ``IPsec`` to ``SSL``, 
I did for mine. If one doesn't work, try the other.

Save your new configs, bounce the AnyConnect client, and your new VPN listings should show up in the UI dropdown.