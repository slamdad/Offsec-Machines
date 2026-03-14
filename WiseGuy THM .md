TryHackMe – WiseGuy v1.3 Writeup

Overview

This challenge focuses on basic XOR cryptography and recovering an encryption key from known plaintext. The service provides an XOR-encrypted hex string that contains a flag. By analyzing the source code and using the known flag format, the encryption key can be recovered.

⸻

1. Source Code Analysis

The provided Python server code performs the following steps:
	•	Generates a random 5-character key
	•	XORs the flag with the key
	•	Encodes the result in hex
	•	Sends the encrypted data to the user
	•	Asks the user to submit the encryption key

Important section of the code:

for i in range(0,len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

Key observations:
	•	Encryption method: XOR
	•	Key length: 5 characters
	•	Flag format: THM{…}

The XOR uses a repeating key (key[i % len(key)]).

⸻

2. Connect to the Service

Connect to the challenge server:

nc <IP> 1337

Example output:

This XOR encoded text has flag 1: 157b7e3d4670525f2842044b4707423507502d55005d4175572d7f4a2e6333474a7643334b7c344b
What is the encryption key?

The encrypted flag is provided as hexadecimal.

⸻

3. Exploiting Known Plaintext

The flag format starts with:

THM{

Convert that to ASCII bytes:

54 48 4d 7b

Take the first 4 bytes of ciphertext and XOR them with the known plaintext to recover the key characters.

Example approach in CyberChef:
	1.	Take the first 8 hex characters of the ciphertext.
	2.	Decode from Hex.
	3.	XOR with the string THM{.

This reveals the first part of the key:

PK01


⸻

4. Recovering the Full Key

From the code we know:

Key length = 5 characters

We currently have:

PK01?

Brute force the last character by trying values.

After testing possibilities:

PK04

This is the correct key.

⸻

5. Retrieve Flag 2

Reconnect to the service and provide the key:

nc <IP> 1337

Input:

PK04

Server response:

Congrats! That is the correct key! Here is flag 2: <flag>


⸻

6. Key Concepts Learned
	•	XOR encryption
	•	Known plaintext attacks
	•	Repeating XOR keys
	•	Hex decoding
	•	Using CyberChef for cryptographic analysis

⸻

Final Result

Encryption Key:

PK04

Use this key to obtain Flag 2 from the server.

⸻

If you want, I can also show a much faster method using Python or CyberChef in one step (no manual guessing), which is useful for similar CTF crypto challenges.