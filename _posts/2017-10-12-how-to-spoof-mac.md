---
layout: post
title: "Find and spoof mac addresses in your lan"
date: 2017-10-12 00:36:21 +0530
categories: linux hack
author: uttarayan21
published: true
image: assets/images/hack.jpg
title_color: white
comments: true
excerpt: "Use the network scanning tool nmap to search people in your lan and spoof their mac addresses"
---

### **Important** _Disclaimer_

#### **Please read carefully**

_Any actions or activities related to the material contained within this Website is solely your own responsibility. The misuse of information from this website might be criminally offensive and if that is your intention **please don't proceed**. The author of this site will be in no way responsible for your own actions._

<br/>

#### Note that all these were done in an Ubuntu 17.04 machine these won't work in windows

Now with that aside we are gonna use nmap to scan your network.

<br/>

First we are gonna install nmap in your linux distro.

For ArchLinux or Arch Based distros (Manjaro,Antegros)


{% highlight bash %}
sudo pacman -Sy nmap
{% endhighlight %}


For Debian based distros (Ubuntu,Linux Mint,elementaryOS,Kali Linux,).


{% highlight bash %}
sudo apt-get install nmap
{% endhighlight %}


For Fedora and Similar distros


{% highlight bash %}
sudo yum install nmap
{% endhighlight %}

<br/>

If your distro doesn't have nmap on its repository then follow the guide [here](https://www.pentestgeek.com/tools/how-to-install-nmap)

<br/>

After you have installed nmap in your system we are ready to scan your network.

The IP addresses of your local network usually range from 192.168.0.100 to 192.168.0.255
You can find out your IP by

{% highlight bash %}
ifconfig
{% endhighlight %}
which gives the following result
{% highlight bash %}
natsu@uttarayan21-PC:~$ ifconfig
enp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.105  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::428d:5cff:fe46:7a24  prefixlen 64  scopeid 0x20<link>
        ether 40:8d:5c:46:7a:24  txqueuelen 1000  (Ethernet)
        RX packets 276718  bytes 323628974 (323.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 195558  bytes 43844165 (43.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 60130  bytes 21465808 (21.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 60130  bytes 21465808 (21.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
{% endhighlight %}

<br/>

My PC is connected to my router bt the interface enp2s0 and my IP is 192.168.0.105.
Your interface might be different though for example something like eth0 or enp1s2.
Also if you are using _WiFi_ then the your interface will be something like _wlan0_


<br/>



<br/>

So we run the command

{% highlight bash %}

sudo nmap -sS 192.168.0.1-255

{% endhighlight %}

to scan my local network from IP 192.168.0.1 to 192.168.0.255
This will display what hosts are active in my local network and their mac addresses.

<br/>

However this might take a very long time so we first do a normal scan to find out who are active and then proceed onto scanning the mac addresses.

<br/>

So we first normally san the network and log the results to a file by

{% highlight bash %}
sudo nmap 192.168.0.1- | tee nmap_report.log
{% endhighlight %}

_Note that 192.168.0.1- is not a typo it's a way of writing that I want to scan the whole network starting from 192.168.0.1_

<br/>

This will also take a fair bit of time.

However this might not show all the hosts since this is a quick scan.

Once its done you can open up the file to see the currently active hosts in your network.

{% highlight bash %}
cat nmap_report.log
{% endhighlight %}

On my network it gives

{% highlight bash %}
Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-12 23:31 IST
Nmap scan report for dd-wrt.com (192.168.0.1)
Host is up (0.011s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
53/tcp  open  domain
80/tcp  open  http
443/tcp open  https

Nmap scan report for uttarayan21-PC (192.168.0.105)
Host is up (0.000026s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
5500/tcp open  hotline

Nmap scan report for uttarayan21-Android (192.168.0.110)
Host is up (0.028s latency).
All 1000 scanned ports on uttarayan21-Android (192.168.0.110) are closed

Nmap done: 255 IP addresses (3 hosts up) scanned in 11.85 seconds
{% endhighlight %}

<br/>

Now suppose I want to scan the host 192.168.0.110 which is actually my android phone.

So I type in

{% highlight bash %}
sudo nmap -sS 192.168.0.110
{% endhighlight %}

and the output is
{% highlight bash %}
natsu@uttarayan21-PC:~$ sudo nmap -sS 192.168.0.110
[sudo] password for natsu:

Starting Nmap 7.40 ( https://nmap.org ) at 2017-10-12 23:44 IST
Nmap scan report for uttarayan21-Android (192.168.0.110)
Host is up (0.18s latency).
All 1000 scanned ports on uttarayan21-Android (192.168.0.110) are closed
MAC Address: D8:9A:34:72:E3:9E (Beijing Shenqi Technology)

Nmap done: 1 IP address (1 host up) scanned in 101.04 seconds
{% endhighlight %}

So nmap gets the mac address of my android phone
Same with the other devices just scan their IP with nmap and you'll find their mac addresses.

<br/>

Also nmap has another feature which can tell you approximately the operating system of the host devces.

Just add a **-O** flag to the command when you scan the network to know the OS.

<br/>
<br/>

Now proceeding into spoofing

There are multiple ways to do this.

I'll be giving two processes here but they aren't the only possible ones.

<br/>

1. __Using network manager__

Open Settings > Network

Choose your default network and paste the mac address there

<br/>

2. __Using /etc/network/interfaces__

If you have use /etc/network/interfaces to  setup your network like I do then you can easily change it by adding
{% highlight bash %}
hwaddress 11:22:33:44:55:66
{% endhighlight %}

to the end of your interface.


It'll work in both static and dhcp IP configuration.

<br/>

3. __Using macchanger__
First of all install macchanger using your default package manager or compile from source (available [here](https://github.com/alobbs/macchanger.git))

Then open up a terminal and type in
{% highlight bash %}
sudo macchanger -m 11:22:33:44:55:66 eth0
{% endhighlight %}

Replace `11:22:33:44:55:66` with the mac address you want to clone and replace `eth0` with your default interface

or you can also use
{% highlight bash %}
macchanger -r eth0
{% endhighlight %}

to just set a random mac address

And thats it !! Now you can intercept all the data packets from and to the device you just cloned the mac from
