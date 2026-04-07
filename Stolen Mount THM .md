Stolen Mount — Forensics Writeup

⸻

1. Objective

Analyze a PCAP file capturing suspicious activity on an NFS server and recover the stolen secret data.

⸻

2. Initial Analysis
	•	Opened the provided PCAP file in Wireshark
	•	Identified that the target service was NFS (Network File System)
	•	Applied filter:

nfs

👉 Confirmed NFS traffic present (file sharing activity)

⸻

3. Identifying Data Exfiltration
	•	NFS protocol decoding was incomplete/noisy
	•	Switched to manual inspection approach

Key technique:
	•	Inspected packets
	•	Located raw hex data inside NFS responses

👉 Identified suspicious file transfer content embedded in packets

⸻

4. Data Extraction
	•	Copied the hex stream from relevant packet(s)
	•	Used CyberChef to process it

Steps in CyberChef:
	1.	Input: Raw hex data
	2.	Operation:
	•	From Hex
	3.	Output:
	•	Extracted embedded file data

⸻

5. Hash Identification
	•	Extracted content revealed an MD5 hash

Example:

<md5_hash_value>


⸻

6. Hash Cracking
	•	Used online/known lookup to resolve hash

Result:

avengers

👉 This was identified as a password

⸻

7. Archive Extraction
	•	Extracted data contained a ZIP archive
	•	Used password:

avengers

✔ Successfully decrypted ZIP file

⸻

8. QR Code Analysis
	•	Inside ZIP:
	•	Found a QR code image

Decoding:
	•	Uploaded to online QR scanner
	•	Extracted hidden content

⸻

9. Final Flag

THM{n0t_s3cur3_f1l3_sh4r1ng}


⸻

10. Attack Summary

What happened:
	1.	Attacker accessed NFS share
	2.	Retrieved backup data
	3.	Data transferred over network (captured in PCAP)
	4.	Sensitive data exposed without encryption

⸻

11. Key Forensics Techniques Used
	•	Protocol identification (NFS)
	•	Manual packet inspection
	•	Hex extraction from network traffic
	•	Data decoding using CyberChef
	•	Hash cracking
	•	Archive decryption
	•	QR code analysis

⸻

12. Security Lessons
	•	NFS traffic was unencrypted → data easily captured
	•	Sensitive backups exposed over network
	•	Weak protection (MD5 + simple password)
	•	Multi-layer obfuscation (hash → zip → QR) is not secure

⸻

13. Final Conclusion

The attacker successfully exfiltrated sensitive data from the NFS server. By analyzing raw packet data, decoding embedded content, and reversing multiple layers of encoding, the stolen information was fully reconstructed.

⸻

✅ Flag

THM{n0t_s3cur3_f1l3_sh4r1ng}