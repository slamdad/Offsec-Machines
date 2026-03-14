TryHackMe — Compiled
CTF Writeup
by slamdad

Challenge Info
Platform
TryHackMe
Challenge
Compiled
Difficulty
Easy
Category
Reverse Engineering
Tools Used
Ghidra, strings, file
Objective
Find the password by analyzing a compiled binary

Overview
This challenge provides a compiled ELF binary that cannot be executed on the AttackBox. The goal is to reverse engineer it statically to find the correct password. The key skills tested are binary analysis, understanding of scanf format strings, and XOR-based logic in decompiled code.

Step 1 — File Identification
First, identify the binary type:
file Compiled.Compiled
Output confirms it is a 64-bit ELF Linux executable. Since execution is not allowed, we proceed with static analysis.

Step 2 — Strings Enumeration
Run strings to gather readable text from the binary:
strings Compiled.Compiled | grep -i "thm\|pass\|correct"
This reveals several interesting strings:
	•	Password:
	•	DoYouEven%sCTF
	•	__dso_handle
	•	_init
	•	Correct!
	•	Try again!
	•	Strings is for Noob
The format string DoYouEven%sCTF and the comparison targets __dso_handle and _init are strong hints toward the password.

Step 3 — Ghidra Static Analysis
Import the binary into Ghidra, run auto-analysis, then navigate to the main function via Symbol Tree → Functions → main.
The decompiled output reveals:
__isoc99_scanf("DoYouEven%sCTF", local_28);
iVar1 = strcmp(local_28, "__dso_handle");
if ((-1 < iVar1) && (iVar1 < 1)) { printf("Try again!"); }
iVar1 = strcmp(local_28, "_init");
if (iVar1 == 0) { printf("Correct!"); }
The logic is:
	•	Input is read using scanf with format string DoYouEven%sCTF
	•	If captured value equals __dso_handle → wrong
	•	If captured value equals _init → correct

Step 4 — scanf Format String Analysis
The critical insight is understanding how scanf processes DoYouEven%sCTF:
	•	DoYouEven is a literal prefix — the input must start with this
	•	%s captures everything after into local_28 until whitespace
	•	CTF in the format string is never matched because %s is greedy and consumes it
This means if we input DoYouEven_init, scanf captures _init into local_28.
The trailing CTF does NOT need to be in the input.

Step 5 — Constructing the Password
To satisfy strcmp(local_28, "_init") == 0, local_28 must equal _init.
Since scanf captures everything after DoYouEven into local_28, the correct input is:
DoYouEven_init
Answer
Password / Flag
DoYouEven_init
Key Concept
scanf greedy %s consumes trailing literal characters
Vulnerability Class
Hardcoded credentials in binary

Key Takeaways
	•	Always run strings first — fast and often sufficient for easy challenges
	•	Ghidra's decompiler is essential when strings alone don't reveal the logic
	•	scanf format strings with %s followed by literals are tricky — %s is greedy
	•	Known plaintext analysis applies here: the format string reveals expected input structure
	•	ELF binaries can be fully analyzed statically even without execution

TryHackMe | Compiled | Writeup by slamdad
