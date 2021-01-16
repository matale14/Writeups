# Legacy

Running nmapAutomator with the Recon option reveals a samba vulnerability

![](images/image10.png)

Googling this CVE ID reveals that this is the infamous “Eternal Blue” exploit. 

We can also see three different versions using searchsploit:

![](images/image4.png)

Digging deeper, it is revealed that 42315.py has the most testing and supports the most target systems.

This can easily be copied via searchsploit:

![](images/image7.png)

And it seems to be working perfectly, now we can modify the code to upload a reverse shell:

![](images/image8.png)

I prepared this exploit script, assuming the target computer has python installed:

![](images/image6.png)

I tested the script and it worked perfectly. However, when adding the reverse shell to EternalBlue, nothing happens:

![](images/image9.png)


I assume the target machine does not have python, or do not have the permissions.

So I’ll create a reverse shell with metasploit instead:

![](images/image5.png)

We use Sudo to make it run as root, -p signifies we want a payload, then we specify which payload we want, set our IP and listening port, we want an exe file, so we add -f exe. And since msfvenom just outputs to the terminal we do > shell.exe to create and add that data to shell.exe

![](images/image1.png)

And it’s working.

![](images/image3.png)

I can confirm python is not installed :)

After that, we can just cd around to the user and root desktops and use “type root.txt” to see the contents:

![](images/image2.png)

EternalBlue is nearly 1k python lines long, I did not want to recreate the entire exploit from scratch in this instance. I did create my own reverse shell, but the main exploit was a bit too large. 


## Comparing to the official writeup:

The official writeup used nmap to enumerate the SMB service, and identifies the exploit available. They then use Metasploit to quickly gain access.

## How to stop this exploit:

Patch the Windows OS. As Microsoft released a hotfix for this exact exploit, MS17-010 here: https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2017/ms17-010?redirectedfrom=MSDN

## Explanation:

Highly reccomend this read by Nadav Grossman:https://research.checkpoint.com/2017/eternalblue-everything-know/

Eternalblue exploits the SMBv1 protocol in Windows machines. 

The first part of the bug exploits a buffer overflow in the `SrvOs2FeaListToNt`function, as it wrongly casts a variable.

The second bug exploits wrong parsing. And is what enables the first bug.

The third and final bug lets the attacker allocate a chunk with a specified size in the kernel non-paged pool. It causes an out of bound write to the next chunk. Which will be the first and second bugs.


