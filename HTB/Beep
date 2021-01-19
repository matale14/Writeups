# Beep

As always, I start by running nmap.

While using my default parametres -sV and -sC, I did not get any response.

So I tried -A which enables OS detection, version, detection, script scanning, and traceroute.
And I also added -v which increases verbosity. Essentially just gives more details.

```
kali@kali:~$ sudo nmap -A -v 10.129.84.71
...
...
...
Discovered open port 995/tcp on 10.129.84.71
Discovered open port 25/tcp on 10.129.84.71
Discovered open port 22/tcp on 10.129.84.71
Discovered open port 443/tcp on 10.129.84.71
Discovered open port 111/tcp on 10.129.84.71
Discovered open port 110/tcp on 10.129.84.71
Discovered open port 3306/tcp on 10.129.84.71
Discovered open port 993/tcp on 10.129.84.71
Discovered open port 80/tcp on 10.129.84.71
Discovered open port 143/tcp on 10.129.84.71
Discovered open port 10000/tcp on 10.129.84.71
Discovered open port 4445/tcp on 10.129.84.71
```
We discover some interesting ports. 80 and 443 signals a website. So lets go check that out.

In order to gain access to the webpage, I need to enable TLS 1.0 on my browser.

After doing that though, we are greeted with a login-page. Powered by Elastix.

![](images/1.png)

I try to input some common credentials like admin/admin. Not only to see if I can get lucky, but also to see if there are some error messages for username and password that can be used with Hydra to crack it.

Nothing whatsoever. So I throw on dirbuster, and start checking manually. The first thing I try is /admin. And I get greeted by a popup login window.

![](images/2.png)

I try a couple credentials again, before cancelling the popup.

This redirects me to https://10.129.84.71/admin/config.php

![](images/3.png)

This also gives us a backend version. freepbx 2.8.1.4 

So lets throw both of these into searchsploit to see if we find something useful.

```
kali@kali:~$ searchsploit Elastix
------------------------------------------------------------- ---------------------------------
 Exploit Title                                               |  Path
------------------------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting                        | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities      | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilitie | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion             | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                            | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                           | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution       | php/webapps/18650.py
------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~$ searchsploit freepbx 2.8.1.4 
------------------------------------------------------------- ---------------------------------
 Exploit Title                                               |  Path
------------------------------------------------------------- ---------------------------------
FreePBX < 13.0.188 - Remote Command Execution (Metasploit)   | php/remote/40434.rb
Freepbx < 2.11.1.5 - Remote Code Execution                   | php/webapps/41005.txt
------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

That FreePBX / Elastix RCE looks very likely. Although the FreePBX version is listed as 2.10.0, lets open it up and see what we can find.



