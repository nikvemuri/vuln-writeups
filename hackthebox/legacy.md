Legacy was another really simple box, highlighting more weaknesses of SMB.

`nmap -A -T4 -Pn 10.10.10.4`:

<img src="/assets/images/htb-legacy/nmap.png">

Looks like a Windows XP machine. The only open ports (139 and 445) are running Windows SMB implementations. 

<img src="/assets/images/htb-legacy/smb_version.png">

Booting up `scanner/smb/smb_version` in Metasploit's auxiliary modules and running it against the box doesn't actually give us the version of SMB. But, it does tell us that the system is on Service Pack 3.

Googling for Windows XP SP3 SMB vulnerabilities returns a stack corruption exploit (MS08-067). Let's run it in Metasploit:

<img src="/assets/images/htb-legacy/exploit.png">

<img src="/assets/images/htb-legacy/getuid.png">

Looks like we're done - we got a meterpreter shell as root. Here are the flags:

<img src="/assets/images/htb-legacy/flags.png">

Thanks for reading!
