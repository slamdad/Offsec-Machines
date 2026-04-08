Sneaky Patch — Kernel Rootkit Forensics Writeup

⸻

🧠 Challenge Summary

A high-value Linux system is suspected to be compromised with a kernel-level backdoor.
Traditional tools show nothing unusual, indicating possible rootkit-level persistence.

Goal:

Identify the backdoor and extract the flag.

⸻

🔍 Step 1 — Initial Enumeration

Started with standard process inspection:

ps aux
top
ss -tulnp

Observation:
	•	Nothing clearly malicious
	•	Only expected services (VNC, systemd, etc.)

👉 Indicates:

Possible kernel-level hiding (rootkit)

⸻

🔍 Step 2 — Kernel Module Inspection

lsmod

Output included:

spatch 12288 0

Initial thoughts:
	•	spatch is unusual
	•	Not a common kernel module

⸻

🔍 Step 3 — Verify Module Legitimacy

Attempted to check kernel logs:

sudo dmesg | grep -i spatch

Output:

spatch: loading out-of-tree module taints kernel.
spatch: module verification failed: signature and/or required key missing


⸻

🚨 Key Finding
	•	Out-of-tree module → not part of official kernel
	•	Unsigned → failed verification
	•	Kernel tainted → system integrity compromised

👉 Strong indicator of malicious kernel module

⸻

🔍 Step 4 — Locate Module File

modinfo -n spatch

Path:

/lib/modules/.../spatch.ko


⸻

🔍 Step 5 — Static Analysis (strings)

strings $(modinfo -n spatch)


⸻

🔥 Critical Findings

1. Backdoor interface

cipher_bd

2. Command hint

[CIPHER BACKDOOR] Format: echo "COMMAND" > /proc/cipher_bd

👉 Confirms:

The module creates a /proc-based backdoor

⸻

3. Suspicious execution capability

/bin/sh
call_usermodehelper_exec

👉 Means:

Kernel module can execute user-space commands (RCE)

⸻

4. Output file reference

/tmp/cipher_output.txt


⸻

🔍 Step 6 — Flag Discovery

Searching for secrets:

strings $(modinfo -n spatch) | grep secret

Output:

[CIPHER BACKDOOR] Here's the secret: 54484d7b73757033725f736e33346b795f643030727d0a


⸻

🔐 Step 7 — Decode the Flag

The string is hex encoded.

Decode using CyberChef / CLI:

echo 54484d7b73757033725f736e33346b795f643030727d0a | xxd -r -p


⸻

Final Flag:

THM{sup3r_sn34ky_d00r}


⸻

🧠 Technical Analysis

What the rootkit does:
	•	Loads as unsigned kernel module (spatch)
	•	Creates hidden interface:

/proc/cipher_bd


	•	Accepts commands via:

echo "COMMAND" > /proc/cipher_bd


	•	Executes commands using:

call_usermodehelper_exec()


	•	Writes output to:

/tmp/cipher_output.txt



⸻

Rootkit Capabilities
	•	Kernel persistence
	•	Command execution (RCE)
	•	Hidden control channel (/proc)
	•	Possible syscall hooking

⸻

🧠 Key Lessons

1. Trust nothing in user-space
	•	ps, lsmod can be manipulated

⸻

2. Always check kernel logs

dmesg

→ Revealed unsigned module

⸻

3. /proc is a goldmine
	•	Custom entries = strong compromise indicator

⸻

4. Static analysis works
	•	strings quickly exposed:
	•	backdoor interface
	•	flag

⸻

5. Kernel rootkits often:
	•	Hide processes/modules
	•	Expose control via /proc or /sys

⸻

🎯 Final Conclusion
	•	Malicious module: spatch
	•	Backdoor interface: /proc/cipher_bd
	•	Flag (decoded):

THM{sup3r_sn34ky_d00r}


⸻

If needed, next level can include:
	•	Reversing the .ko with Ghidra
	•	Detecting hidden modules without lsmod
	•	Memory forensics with Volatility