I wanted to try my luck on an active box. Went with the first one, OpenAdmin.

Starting with `nmap -A -T4 -Pn 10.10.10.171`:

<img src="/assets/images/htb-openadmin/nmap.png">

System's running Ubuntu. We have SSH on 22, and an Apache 2.4.29 HTTP server on 80.

The web server just has the default landing page, and we don't have the password for SSH. I checked Google for Apache 2.4.29 vulnerabilities and [found a local privilege escalation exploit](https://www.exploit-db.com/exploits/46676). But we don't have local access to the machine - we need something remote. So this won't work.

I ran `nikto` and `dirbuster` next to dig into the server:

<img src="/assets/images/htb-openadmin/nikto.png">

Nikto showed some warnings, but nothing of major note.

<img src="/assets/images/htb-openadmin/dirbuster.png">

Okay, here we have something. The `/music/` and `/artwork/` subdirectories lead to template-looking pages which are pretty but don't have much information.

`/ona/`, on the other hand, opens up OpenNetAdmin (version 18.1.1 as we can see on the page).

Tried a few links, looked at the page source, didn't find anything. Searching for ONA 18.1.1 vulnerabilities lands us on a [sweet remote code execution exploit](https://www.exploit-db.com/exploits/47691). It's rated as critical. Copied the script into a .sh file, and ran it against the ONA subdirectory:

<img src="/assets/images/htb-openadmin/expsh.png">

It worked, but we don't have root yet. The Apache privilege escalation exploit from earlier might be worth revisiting, but for now let's look around in the files.

After a lot of manual searching (reading the ONA documentation and `grep`ing for what I needed might have been smarter), I found a database configuration file:

<img src="/assets/images/htb-openadmin/database.png">

In here, there's a password - "n1nj4W4rri0R!". Probably an SSH password. But we don't know whose password that is. Looking `/etc/passwd` us a list of users, out of which only 2 looked like _non-program_ users: _jimmy_ and _joanna_. SSHing into `jimmy@10.10.10.171` with the password `n1nj4W4rri0R!` works, and we're in a shell as jimmy.

running `find / -type f -user jimmy` (find files which user jimmy owns/has access to) and scrolling through gives us this:

<img src="/assets/images/htb-openadmin/findjimmy.png">

The `www-data` user didn't have access to this "internal" folder, which can be verified by running `find / -type f -user www-data` in the previous shell. `logout.php` and `index.php` look fairly plain, but `main.php` is interesting:

<img src="/assets/images/htb-openadmin/php.png">

Looks like upon loading this file, the web server print's user joanna's RSA key (which we can use to SSH in as her). Let's try `curl`ing this:

<img src="/assets/images/htb-openadmin/curl.png">

We get a 404, even though the file clearly exists. At this point I had the usual almost-done-but-stuck period. Eventually I noticed localhost Port 80 at the bottom of the output and tried to see if any other local ports were open using `netstat`:

<img src="/assets/images/htb-openadmin/netstat.png">

I tried `curl` with localhost:3306, but that showed the same error. localhost:52846 did work though, and the RSA key pops up:

<img src="/assets/images/htb-openadmin/curl_success.png">

It's not just an RSA key though, at the bottom we see 'Don't forget your "ninja" password'. Sure enough, copying the key into a file called joanna_rsa and running `ssh -i joanna_rsa joanna@10.10.10.171` asks for a "passphrase for key" as well (note you have to limit the permissions on the SSH key, like: `chmod 400 joanna_rsa`).

I used John the Ripper to extract a password from the key. First, the key has to be in proper format for `john`; this can be done with `sudo /usr/share/john/ssh2john.py joanna_rsa > joanna_rsa.hash`. Then using the "rockyou.txt" wordlist which is already in Kali, we run `john`:

<img src="/assets/images/htb-openadmin/john.png">

There's a password - "bloodninjas". And it works:

<img src="/assets/images/htb-openadmin/joanna.png">

We can get the user flag, but we're still not root:

<img src="/assets/images/htb-openadmin/whoami.png">

`sudo -l` lists which commands joanna can run without a superuser password. Looks like she can only run the preinstalled `nano` on a specific file, `/opt/priv`. Opening this file up, we see that it's empty, but nano has a handy command at the bottom for reading files:

<img src="/assets/images/htb-openadmin/nanoread.png">

Let's try reading root.txt from the default location on Linux boxes:

<img src="/assets/images/htb-openadmin/roottxt.png">

<img src="/assets/images/htb-openadmin/rootflag.png">

There it is!


This one was a lot more fun than the retired ones, and I liked how the exploitation leveraged multiple services on the machine. Definitely felt more CTF-style than the others I've done so far, though. Thanks for reading!
