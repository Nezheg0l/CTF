
# Lab: SQL Injection attack, querying the database type and version on Oracle

A detailed write-up for the "[SQL injection attack, querying the database type and version on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)" lab from PortSwigger Academy.

---

## 🎯 Lab Objective

The goal of this lab is to exploit a SQL injection vulnerability in the product category filter to retrieve and display the Oracle database's version string.

## 🧐 Vulnerability Analysis

The application is vulnerable because it uses user-supplied input from the `category` parameter directly within a SQL query without proper sanitization or parameterization. This allows an attacker to break out of the original query and append a `UNION` statement to run their own queries.

---

## 🚀 Exploitation Steps

The exploitation process involves two main steps: determining the number of columns returned by the original query and then crafting a payload to extract the database version.

### 1. Determining the Number of Columns

The first step in a `UNION` attack is to match the number and data types of the columns in the original query. I used Burp Suite to intercept the request and inject payloads into the `category` parameter.

To find the column count, I used the following payload:

```sql
'+UNION+SELECT+'abc','def'+FROM+dual--
```

**Payload Breakdown:**
*   `'`: Closes the original query's string literal.
*   `+UNION+SELECT+'abc','def'`: Appends a `UNION` statement selecting two string literals.
*   `+FROM+dual`: `dual` is a special one-row, one-column table present by default in Oracle databases. It is a convenient way to run a `SELECT` statement that doesn't need data from a real table.
*   `--`: Comments out the rest of the original query to avoid syntax errors.

This payload returned a `200 OK` response and displayed the strings `abc` and `def` on the page, confirming that the original query returns **two columns**, both of which can hold string data.

### 2. Querying the Database Version

Now that the structure is known, we can replace the placeholder values (`'abc'`, `'def'`) with a query to fetch the database version. In Oracle, this information is stored in the `v$version` view.

The final payload is:

```sql
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

**Payload Breakdown:**
*   `+UNION+SELECT+BANNER,+NULL`: We select the `BANNER` column, which contains the database version string.
*   `+FROM+v$version`: The query targets the `v$version` system view.
*   `,+NULL`: As we need to match two columns, `NULL` is used as a placeholder for the second column.

This payload successfully executed, and the database version string was displayed on the page, solving the lab.

---

## ✅ Key Takeaways

*   A `UNION` attack requires matching the number and data types of columns from the original query.
*   Oracle databases have unique features, such as the `dual` dummy table and system views like `v$version`.
*   The `BANNER` column within `v$version` is the standard place to find the full version string in an Oracle database.
