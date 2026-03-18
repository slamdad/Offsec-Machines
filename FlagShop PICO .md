Flag Shop – picoCTF 2019 Writeup

Challenge Type
General Skills (Integer Overflow Exploit)

⸻

Objective

Purchase the “1337 Flag” priced at $100,000 despite starting with only $1,100 by exploiting a vulnerability in the program.

⸻

Step 1: Connect to the Challenge

nc 2019shell1.picoctf.com 63894


⸻

Step 2: Understand the Program Logic
	•	The program allows you to:
	•	Buy cheap flags ($900 each)
	•	Buy the “1337 Flag” ($100,000)
	•	Internally, the cost is calculated as:

total_cost = 900 * number_flags;

	•	Data type used: 32-bit signed integer

⸻

Step 3: Identify the Vulnerability

A 32-bit signed integer overflow occurs when:
	•	The result exceeds 2,147,483,647
	•	It wraps around into the negative range (two’s complement)

Effect:

account_balance -= total_cost;

If total_cost becomes negative, subtracting it will increase your balance.

⸻

Step 4: Trigger the Overflow

Choose:

2 → Buy Flags  
1 → Definitely not the flag Flag

Enter a large number of flags:

3579138

Why this works:
	•	Calculation:

900 × 3579138 = overflow → negative value


	•	Result:
	•	Instead of losing money, your balance increases massively

Example:

Balance: $1100 → ~$1,000,000,000+


⸻

Step 5: Buy the Real Flag

Now select:

2 → Buy Flags  
2 → 1337 Flag  
1 → Quantity

You now have enough balance to purchase it.

⸻

Step 6: Retrieve the Flag

Output:

YOUR FLAG IS: picoCTF{m0n3y_bag5_783740a8}


⸻

Key Concepts Learned
	•	Integer overflow in C (32-bit signed int)
	•	Two’s complement behavior
	•	Exploiting arithmetic bugs to manipulate logic
	•	Importance of validating user input and using safe data types

⸻

Summary

By exploiting integer overflow in cost calculation, the program incorrectly increases your balance. This allows purchasing an otherwise unaffordable flag.