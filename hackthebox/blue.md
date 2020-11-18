Blue is named after the EternalBlue (MS17-010) exploit. 

This takes advantage of a request handling flaw in SMB servers on Windows and allows remote code execution. The exploit was leveraged by ransomware since it had such a widespread presence, and it ended up having huge impacts.

This box was really easy, but more importantly it got me reading about EternalBlue, ransomware, WannaCry, and the Shadow Brokers and the NSA's role in all of this, which ended up being quite interesting.

Ok, let's start - `nmap -A -T4 -Pn 10.10.10.40`:

<img src="/assets/images/htb-blue/nmap.png">

Samba ports are open, along with a bunch of RPC ports. SMB also shows us that the machine is running Windows 7 Professional, Build 7601, Service Pack 1. Message signing is also not required on port 139 and 445, which leaves SMB open to exploitation.

A quick Google search confirms that Windows 7 Professional 7601 SP1 is likely vulnerable to the EternalBlue exploit.

Let's boot up the exploit on Metasploit:

<img src="/assets/images/htb-blue/exploit.png">

<img src="/assets/images/htb-blue/whoami.png">

And we have system privileges. Let's get the flags:

<img src="/assets/images/htb-blue/user.png">

<img src="/assets/images/htb-blue/root.png">

Thanks for reading!
