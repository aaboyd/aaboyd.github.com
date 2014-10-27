---
layout: post
title: "Enable WoWLAN ( Wake on Wireless LAN ) on Windows 8.1"
tags : [C#, windows]
description : Wake computer from sleep using WoWLAN on Windows 8.1
---

## What is WoWLAN?

WoWLAN or Wake on Wireless LAN provides network enabled devices the ability to be brought out of sleep by other devices on the network.  WOL or Wake On Lan has been around for a while and used in many different scenarios.  WoWLAN really just adds WOL functionality to wireless interfaces instead of just hard wired ethernet connections.

## Who uses WoWLAN?

The first time I was asked about WoWLAN was at work, when a customer wanted to wake up their Mac Mini with [iRule](http://iruleathome.com).  Mac Mini's are used a fair amount as HTPC's and keeping them awake isn't a very big deal, but allowing it to sleep is the default functionality.

<div class="alert warning">I know this article is about setting up WoWLAN on Windows 8.1, but this was one of the reason's I started looking into it.  The walkthrough will not show you how to set it up on a Mac Mini.</div>

The previous scenario was a long time ago and I hadn't really thought about it much until recently.  I am on my computer a lot, at my desk, on the couch, and anywhere else you can take a computer.  I have a desktop computer and a Macbook Pro.  I remote desktop into my desktop often, but you can't remote desktop unless the computer is awake.  So, I started poking around to get WoWLAN to work the way I want.

I'm sure there are other scenarios where this could and will be useful.

## Setting up WoWLAN

#### Open Device Manager, find your wireless interface, and open the Preferences

![Device Manager](/assets/img/wowlan/device_manager.png)

#### Navigate to the Advanced tab and ensure the "Wake on Magic Packet" is enabled.

![Wireless Interface Advanced Properties](/assets/img/wowlan/properties_advanced.png)

#### Navigate to the Power Management tab, and check the box for Allow the device to wake the computer and Only allow a magic packet to wake the computer.

![Wireless Interface Power Management Properties](/assets/img/wowlan/properties_power_management.png)

#### Get the Devices MAC Address by running ```ipconfig /all```.  The output should contain a Physical Address for each interface you have.

![IPCONFIG output](/assets/img/wowlan/mac_address.png)

## Sending the Packet

There are multiple tools that I have seen out there to send out WOL or Magic Packets.  Since I am a developer, I decided to just whip something up real quick to help me send out the packet.

<div class="alert secondary">I have a project on <a href="https://github.com/aaboyd/WakeUp">Github</a> with the source code for the wakeup utility I wrote to send out the WOL packets.</div>

From my macbook pro, I use mono to execute the program.

{% highlight bash %}
mono /path/to/exe/wakeup.exe 0C-8B-FD-24-1A-8B
{% endhighlight %}

### The Code

{% highlight csharp %}
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Text.RegularExpressions;
using System.Linq;

public class Program
{
    public static void Main(String[] args)
    {
        if(args.Length != 1)
        {
            Console.WriteLine("Usage: wakeup.exe [mac address]");
            System.Environment.Exit(-1);
        }
            
        // strip non-hex characters
        var macAddress = new Regex("[^a-fA-F0-9]").Replace(args[0], "");  

        // build magic packet
        string hexMagicPacket = String.Concat(Enumerable.Repeat("FF", 6)) +
                                String.Concat(Enumerable.Repeat(macAddress, 16));
        
        // hex string to byte array
        byte[] magicPacket = Enumerable.Range(0, hexMagicPacket.Length)
                     .Where(x => x % 2 == 0)
                     .Select(x => Convert.ToByte(hexMagicPacket.Substring(x, 2), 16))
                     .ToArray();

        // send packet to broadcast address at port 9
        UdpClient udpClient = new UdpClient();
        udpClient.Connect(IPAddress.Broadcast, 9);
        udpClient.Send(magicPacket, magicPacket.Length);

        Console.WriteLine("Magic Packet Sent");
    }
}
{% endhighlight %}