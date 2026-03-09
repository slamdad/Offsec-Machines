TryHackMe – The Game Binmary Analysis Writeup

Challenge Description

Cipher has hidden sensitive data inside the binary of a Tetrix game. The objective is to analyze the executable file and uncover the hidden flag embedded within the program.

Objective

Analyze the provided .exe file and extract the hidden flag from the binary.

⸻

Environment

Analysis was performed using:
	•	Kali Linux (ARM64)
	•	Standard Linux command-line tools

⸻

Initial File Inspection

The challenge provided a Windows executable file (Tetrix.exe).
Since the challenge hinted that the flag was embedded in the program code, the first step was to inspect the binary for readable strings.

To identify readable text inside the binary, the strings utility was used.

strings Tetrix.exe

This command extracts printable characters from the binary, which often reveals hidden messages, debug text, or flags used in CTF challenges.

⸻

Searching for the Flag

Because CTF flags often contain keywords such as flag, ctf, or the platform name, the output was filtered using grep.

strings Tetrix.exe | grep -i thm

The -i option ensures the search is case-insensitive.

⸻

Flag Discovery

The command returned a string containing the flag embedded in the binary.

This confirmed that the flag was directly stored inside the executable as a readable string and did not require complex reverse engineering or decoding.

⸻

Result

The flag was successfully extracted from the binary using basic static analysis techniques.

⸻

Key Takeaways
	•	Many beginner reverse-engineering challenges store flags directly in binaries.
	•	The strings command is a quick and effective way to discover embedded text.
	•	Filtering with grep helps quickly locate relevant information such as flags or secrets.

⸻

Tools Used
	•	strings
	•	grep
	•	Kali Linux terminal

⸻

Conclusion

By performing simple static analysis on the executable file and searching for relevant keywords, the hidden flag was located without requiring advanced reverse engineering tools. This demonstrates how basic binary inspection techniques can often reveal sensitive data embedded within executables.
:::