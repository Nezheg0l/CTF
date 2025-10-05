
# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

This document is a comprehensive write-up for the "[SQL injection attack, querying the database type and version on MySQL and Microsoft](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)" lab from PortSwigger Academy.

---

## 🎯 Lab Objective

The objective is to exploit a `UNION`-based SQL injection vulnerability in the product category filter to identify and display the database version string, which could be either MySQL or Microsoft SQL Server.

## 🧐 Vulnerability Analysis

The application is vulnerable to SQL injection because it fails to properly sanitize the `category` parameter. The input is directly concatenated into a SQL query, allowing an attacker to modify the query logic. This can be exploited using a `UNION` statement to extract information from the database.

---

## 🚀 Exploitation Steps

The exploitation process was carried out using Burp Suite to intercept and modify the HTTP requests.

### 1. Determining the Number of Columns

To perform a `UNION` attack, the injected query must return the same number of columns as the original query. I started by injecting payloads to determine this count.

The following payload confirmed that the query returns two columns and that both can hold string data:

```sql
'+UNION+SELECT+'abc','def'#
```

**Payload Breakdown:**
*   `'`: Escapes the `WHERE` clause's string literal.
*   `+UNION+SELECT+'abc','def'`: Appends a `UNION` query that selects two string values. This helps confirm the column count and which columns are suitable for displaying text.
*   `#`: A comment character in both MySQL and Microsoft SQL Server. This effectively nullifies the rest of the original query, preventing syntax errors.

The server responded with a `200 OK`, and the strings `abc` and `def` were rendered on the page, confirming a two-column structure.

### 2. Querying the Database Version

With the column structure identified, the next step was to retrieve the database version. Both MySQL and Microsoft SQL Server use the `@@version` system variable to store this information.

The final payload to solve the lab was:

```sql
'+UNION+SELECT+@@version,+NULL#
```

**Payload Breakdown:**
*   `+UNION+SELECT+@@version`: This selects the global `@@version` variable, which contains a detailed version string of the database software.
*   `,+NULL`: This is used as a placeholder for the second column to satisfy the requirement of matching the column count.
*   `#`: Comments out the remainder of the query.

Upon sending this payload, the application displayed the full database version string, successfully completing the lab.

---

## ✅ Key Takeaways

*   **Database-Specific Syntax is Crucial:** While the `UNION` attack methodology is similar across databases, the syntax for system information and comments varies. Here, `@@version` and the `#` comment character are specific to MySQL/Microsoft SQL Server.
*   **System Variables are Goldmines:** Variables like `@@version` provide immediate and valuable information for an attacker during the reconnaissance phase.
*   **Column Enumeration is Fundamental:** A successful `UNION` attack always starts with correctly identifying the number of columns in the original query.
