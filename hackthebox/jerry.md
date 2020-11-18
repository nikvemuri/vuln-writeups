Ok, next up is Jerry. 

Let's run `nmap -A -T4 -Pn 10.10.10.95`:

<img src="/assets/images/htb-jerry/nmap.png">

Didn't find anything except a web server on HTTP. More specifically, Apache Tomcat 7.0.88. Visiting the URL (`10.10.10.95:8080`) gives us a default Tomcat landing page:

<img src="/assets/images/htb-jerry/landing.png">

Wonder what else they left default. 

Tomcat has a server manager web app which can be accessed straight from this landing page by going to "Manager App". We need to be authenticated to access it:

<img src="/assets/images/htb-jerry/login.png">

I found a list of about 20 default Tomcat credentials on GitHub, and just tried the first one to see what the next page looks like:

<img src="/assets/images/htb-jerry/401.png">

So we get a 401, as expected. But if you look closely, they've given us the default login: `tomcat` and `s3cret`! (This wouldn't be too hard to find by just reading the documentation, and in the worst case scenario we could just use Burp and run a brute force attack for the 20ish default credentials.)

Attempting to log in using `tomcat` and `s3cret`:

<img src="/assets/images/htb-jerry/loggedin.png">

It worked! There are a number of things on the manager page, but "Deploy" looks interesting. Tomcat uses the WAR format, and we can deploy WAR files through the manager. 

<img src="/assets/images/htb-jerry/deploy.png">

Can we get a reverse shell through a custom WAR file?

I used `msfvenom` to create a jsp reverse shell payload which connects back to my IP (10.10.16.95) and listens on port 4444, all in a WAR file called `shell.war`:

<img src="/assets/images/htb-jerry/msfvenom.png">

Next, upload `shell.war` to the web server's base directory through the "Deploy" part of the manager.

Now, we just need to listen on port 4444
Listen using `nc -nvlp 4444`, and browse to the `/shell` file on browser. Once that loads, it would have connected back to our machine through port 4444 and loaded up a shell:

<img src="/assets/images/htb-jerry/root.png">

And we have root. Looks like both flags are in one file this time:

<img src="/assets/images/htb-jerry/flags.png">

It's pretty obvious that using default credentials is a horrible idea, especially on a large network. The Mirai botnet is an example of how badly default credentials can be exploited. Credential stuffing and password spraying are really easy to pull off as well. I'm not saying passwords that we set ourselves are much better, they're not. But at least it introduces variation and makes the attacker take some time to pull off a brute force attack.

The point is, change your password.

Thanks for reading!
