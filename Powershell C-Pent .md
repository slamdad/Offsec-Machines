

# **Overall Explanation of What This Lab Is Doing**

This lab teaches how PowerShell and WinRM (Windows Remote Management) are used for system administration, enumeration, remote command execution, and how execution policies can be bypassed.
It follows three phases:

1. **Exploration & Enumeration (Exercise 1):**
   You learn how to inspect the Windows environment using PowerShell and CMD.
   This includes listing users, processes, services, and understanding how help, filtering, and pipelines work.

2. **Remote Administration with WinRM & Python (Exercise 2):**
   You configure WinRM on Windows Server 2019, allow remote access, and then use Python (`winrm` module) from Ubuntu to run commands on the Windows machine.
   This demonstrates how attackers or admins can remotely execute commands and retrieve process or event log data.

3. **Execution Policy Bypass Techniques (Exercise 3):**
   You create a simple PowerShell script (`test.ps1`) and set the machine to a restrictive execution policy.
   Then you use multiple known bypass techniques to execute the script anyway.
   This proves that **ExecutionPolicy is not a security boundary**, and demonstrates stdin execution, `Invoke-Expression`, inline commands, remote downloads, and process-scoped overrides.

---

# **Exercise 1 — PowerShell Enumeration & Local User Management**

## **Open elevated PowerShell**

Right-click *Windows PowerShell* → *Run as Administrator*.

## **Show basic help pager**

```powershell
help | more
```

## **List local users**

```powershell
net user
```

## **Create a new user**

```powershell
net user cpent password /add
```

## **Add the user to Administrators**

```powershell
net localgroup administrators cpent /add
```

## **Check users again**

```powershell
net user
```

---

## **Process enumeration using Command Prompt**

Open CMD:

### **Show tasklist help**

```cmd
tasklist /?
```

### **Show processes + services**

```cmd
tasklist /svc | more
```

### **Search for svchost.exe**

```cmd
tasklist /svc | findstr svchost.exe | more
```

---

## **Execution Policy Basic Commands**

### **Check current policy**

```powershell
Get-ExecutionPolicy
```

### **Set to Unrestricted**

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope LocalMachine
# type Y when prompted
```

---

## **PowerShell Help Exploration**

### **Commands beginning with “a”**

```powershell
get-help a*
```

### **List last 10 running services**

```powershell
Get-Service | Where-Object { $_.Status -eq "running" } | Select-Object -Last 10
```

---

## **Create a local user using PowerShell cmdlet (modern API)**

### **Check syntax**

```powershell
New-LocalUser /?
```

### **Example user creation**

```powershell
$pwd = Read-Host -AsSecureString "Enter Password"
New-LocalUser -Name "cpent2" -Password $pwd -FullName "cpent test user"
```

---

## **PowerShell Help: Detailed / Full / Example**

```powershell
Get-Help Copy-Item
Get-Help Copy-Item -Detailed | more
Get-Help Copy-Item -Full | more
Get-Help Copy-Item -Examples
```

---

# **Exercise 2 — WinRM Setup + Python Remote Execution**

## **Enable WinRM**

```powershell
winrm quickconfig
# type y
```

If the firewall blocks because the network is Public:

### **Change to Private network**

```powershell
Set-NetConnectionProfile -NetworkCategory Private
```

---

## **Configure WinRM for Basic Auth (lab settings)**

```powershell
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/client/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

### **Allow all trusted hosts**

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

### **Verify configuration**

```powershell
winrm get winrm/config | more
```

### **Check listener on port 5985**

```powershell
Get-NetTCPConnection | Where-Object -Property LocalPort -EQ 5985
```

---

# **Python Script #1 — Basic WinRM Command Execution (testingrm.py)**

```python
import winrm

# Windows target
ip = "192.168.177.20"

# WinRM session
session = winrm.Session(
    target=ip,
    auth=('Administrator', 'Pa$$w0rd')
)

# Run a basic command
result = session.run_cmd('hostname')

print(result.std_out.decode())
```

---

# **Python Script #2 — Fetch Process IDs (Extended)**

```python
import winrm

ip = "192.168.177.20"
session = winrm.Session(ip, auth=('Administrator', 'Pa$$w0rd'))

command = 'Get-Process | Select-Object Name, Id'
result = session.run_ps(command)

print(result.std_out.decode())
```

---

# **Python Script #3 — Pull Latest Event Logs**

```python
import winrm

ip = "192.168.177.20"
session = winrm.Session(ip, auth=('Administrator', 'Pa$$w0rd'))

# Get all logs → ForEach → get last 10 entries from each log
command = """
Get-EventLog -List | ForEach-Object {
    Get-EventLog -LogName $_.Log -Newest 10
}
"""

result = session.run_ps(command)
print(result.std_out.decode())
```

---

# **Exercise 3 — PowerShell Execution Policy Bypass**

## **Set policy to Restricted**

```powershell
Set-ExecutionPolicy -ExecutionPolicy Restricted
# type Y
```

### **Verify**

```powershell
Get-ExecutionPolicy
```

---

## **Create test script `test.ps1`**

Contents:

```powershell
Write-Host "Testing Execution Bypass"
calc.exe
```

Store on Desktop.

---

## **Try to run normally (blocked)**

```powershell
cd Desktop
.\test.ps1
```

---

# **Bypass Methods Demonstrated**

## **1. Execute commands manually**

```powershell
Write-Host "This is a bypass test"
calc.exe
```

---

## **2. Pipe script text to PowerShell stdin**

```powershell
Get-Content .\test.ps1 | powershell.exe -noprofile -
```

---

## **3. Download + execute in memory**

```powershell
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('test.ps1')"
```

---

## **4. Inline command execution**

```powershell
powershell -command "Write-Host 'This is a bypass test' ; calc.exe"
```

---

## **5. Invoke-Expression with Get-Content**

```powershell
Get-Content .\test.ps1 | Invoke-Expression
```

---

## **6. Launch PowerShell with ExecutionPolicy Bypass**

```powershell
powershell -ExecutionPolicy Bypass -File .\test.ps1
```

---

## **7. Set ExecutionPolicy only for current process**

```powershell
Set-ExecutionPolicy Bypass -Scope Process
```

### **Verify**

```powershell
Get-ExecutionPolicy -List | Format-Table -AutoSize
```

### **Run script**

```powershell
.\test.ps1
```

---
