Python Wrangling - picoCTF Writeup
===================================

Category: General Skills
Difficulty: Easy

Overview
--------
This challenge involves decrypting an encrypted flag using a Python script,
a password file, and an encrypted flag file.

Files Provided
--------------
- ende.py       (encryption/decryption script)
- pw.txt        (contains the password)
- flag.txt.en   (encrypted flag)

Solution Steps
--------------

1. Download the files using wget:
   wget <link_to_ende.py>
   wget <link_to_pw.txt>
   wget <link_to_flag.txt.en>

2. Read the password:
   cat pw.txt
   Output: aa821c16aa821c16aa821c16aa821c16

3. Decrypt the flag:
   python3 ende.py -d flag.txt.en

   When prompted for the password, enter:
   aa821c16aa821c16aa821c16aa821c16

   Or pass it directly:
   python3 ende.py -d flag.txt.en $(cat pw.txt)

4. The decrypted flag is printed to stdout.

Flag
----
picoCTF{4p0110_1n_7h3_h0us3_aa821c16}

Key Takeaways
-------------
- ende.py uses the cryptography.fernet library for symmetric encryption.
- The -d flag tells the script to decrypt, -e to encrypt.
- Always review scripts before running them.
- Use $(cat pw.txt) to pipe file contents directly as command arguments.
- Use python3 explicitly as the script uses sys.stdout.buffer.write()
  which behaves differently in Python 2.