I figured things would be smooth if I followed Hack The Box's difficulty ranking from easy to hard. So, here we are with the first box, Lame.

Running `nmap -A -T4 -Pn 10.10.10.3` (just running on default ports for the easier boxes to save time) gives us:

<img src="/assets/images/htb-lame/nmap.png">

Okay, lots of things open. First thing of note is that anonymous FTP login is allowed. That's a red flag, but we still won't be able to actually own the system unless there's an exploit for that implementation of FTP. 

Moving on - Samba's also open, looks promising. I'll probably try messing with this first.

We have SSH available too, but I'll leave that as backup, since SSH doesn't usually do much in terms of exploitation. (Also, a more comprehensive scan I ran earlier showed distccd open on port 3632. There's a distccd exploit that can be used to root this box too. Maybe a full port scan was a better idea after all.)

Googling for vsFTPd 2.3.4 exploits lands us on a backdoor command exec exploit. Next, we have to check what version of SMB the box is running, and for that I used `smb_version` from Metasploit's auxiliary modules:

<img src="/assets/images/htb-lame/smb_version.png">

Cool, it worked. We have version 3.0.20 on Debian. I found a "username map script" exploit allowing us to run arbitrary commands by taking advantage of a config option. Let's run it through Metasploit:

<img src="/assets/images/htb-lame/usermap_exploit.png">

We got root! Let's find the files with `updatedb` and `locate root.txt user.txt`

`user.txt` is in `/home/makis` and `root.txt` is in `/root`:

<img src="/assets/images/htb-lame/user.png">

<img src="/assets/images/htb-lame/root.png">

Thanks for reading, stick around for the next one!
