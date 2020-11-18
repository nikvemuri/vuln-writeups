Wanted a break from Hack The Box, so I downloaded the Kioptrix series from [Vulnhub](https://www.vulnhub.com/). This write-up is for [Kioptrix: Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/).

On a side note, [this compilation](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159), along with [abatchy's post on OSCP-like VulnHub VMs](https://www.abatchy.com/2017/02/oscp-like-vulnhub-vms), are great resources for OSCP prep. I'll probably make a detailed post of all the resources I use by the time I'm done with the exam.

Ok, let's begin. First, download the VM, extract it and set it up. Not going into much detail here, but I'm using VMWare Player, and I set the networking on both my Kali box and Kioptrix to NAT. Now just start up the VM and let it boot, and once the login screen pops up you can just minimize it and continue on the attacking box.

Kioptrix is set to be automatically assigned an IP through DHCP. `sudo ifconfig` on Kali shows our IP:

<img src="/assets/images/kioptrix-level1/ifconfig.png">

Using `netdiscover -i eth0` (the argument specifies which adapter to search on) we can look for the machine on our NAT network:

<img src="/assets/images/kioptrix-level1/netdiscover.png">

Looks like it's 192.168.183.131 (the address right after Kali).

Ping just to make sure it's responsive:

<img src="/assets/images/kioptrix-level1/ping.png">

`nmap -A -T4 -p- 192.168.183.131`:

<img src="/assets/images/kioptrix-level1/nmap.png">

We have SSH on 22, Apache 1.3.20 on 80/443, RPC on 111, and SMB on 139. Also something on 32768, probably another RPC. Also, looks like the machine is on Red Hat Linux.

`ssh 198.168.183.131` gives us a "connection refused" error, so it doesn't seem brute-forceable. Didn't find any exploits for OpenSSH 2.9p2 either. Moving on.

`smbclient -L \\\\198.168.183.131\\` failed, but `enum4linux 192.168.183.131` did run. It didn't give us any more information, but running a packet capture analysis in Wireshark and looking at the TCP stream conversations quickly gives us Samba 2.2.1a. `searchsploit` shows some interesting exploits that look promising:

<img src="/assets/images/kioptrix-level1/smbexploits.png">

Let's keep enumerating. Port 80/443 are running Apache 1.3.20, and visiting the web server gives us a default landing page:

<img src="/assets/images/kioptrix-level1/website.png">

`nikto -host 192.168.183.131`:

<img src="/assets/images/kioptrix-level1/nikto.png">

Lots of potential exploits. Let's keep going with `dirbuster`:

<img src="/assets/images/kioptrix-level1/dirbuster.png">

Looks like the server is running `mod_ssl` and `mod_perl`, as well as MRTG.

<img src="/assets/images/kioptrix-level1/mrtg.png">

Maybe there are some ways to exploit this. Nothing on searchsploit so we'll move on for now. Looks like MRTG is a traffic monitor, that's probably how the /usage applet works.

In the `mod_ssl` pages, we find the `mod_ssl` version:

<img src="/assets/images/kioptrix-level1/mod-version.png">

For which there are some sweet exploits:

<img src="/assets/images/kioptrix-level1/mod-exploits.png">

At the bottom of /usage, there's also this:

<img src="/assets/images/kioptrix-level1/webalizer.png">

Found something interesting for Webalizer: <https://www.cvedetails.com/cve/CVE-2002-0180/>. Might be another way in.

Ok, lastly, some exploits for Apache 1.3.20 which might be worth looking into:

<img src="/assets/images/kioptrix-level1/apache-exploits.png">

(Spoiler: eliminated all of them, as they were based on `mod_include`, Win32, `mod_mylo`, and `mod_rewrite`, which this server isn't using. It does look like the site is vulnerable to cross-site scripting, however this doesn't really help us here.)

That leaves us with the `mod_ssl` buffer overflow exploit. This is the updated version: <https://www.exploit-db.com/exploits/47080>. Let's try it.

Download the C file: `searchsploit -m 47080 && mv 47080.c OpenFuck.c`

`apt-get install libssl-dev` and `gcc -o OpenFuck OpenFuck.c -lcrypto` to compile the program into a binary.

<img src="/assets/images/kioptrix-level1/run-exploit.png">

Following is a list of target IDs. Ours is Red Hat, and based on the Apache version:

<img src="/assets/images/kioptrix-level1/targets.png">

0x6a didn't work, so:

`./OpenFuck 0x6b 192.168.183.131 443 -c 40`:

<img src="/assets/images/kioptrix-level1/root.png">

(I was actually stuck on this part for a while. I was only getting a shell as the `apache` user, and it didn't look like the exploit was working. I spent a while looking for privesc methods but then realized that my college internet went down and the exploit couldn't connect to packetstorm to finish its second stage. Staged payloads are fun!)

There are potentially 2 other ways that I found to exploit this box - one was the Samba 2.2.1a buffer overflow, and the other was the Webalizer buffer overflow. I didn't try either of them, but considering there are "multiple ways to own this box", I'm pretty sure they must lead somewhere.

Thanks for reading!
