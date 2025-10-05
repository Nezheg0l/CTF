
# Lab: SQL injection attack, listing the database contents on non-Oracle databases

This write-up details the step-by-step process of exploiting a `UNION`-based SQL injection to enumerate a database, extract user credentials, and gain administrative access in the lab "[SQL injection attack, listing the database contents on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)".

---

## 🎯 Lab Objective

The goal is to identify a SQL injection vulnerability, use it to map out the database schema, find the table containing user credentials, retrieve the administrator's password, and successfully log in to the application.

## 🧐 Vulnerability Analysis

The product category filter is vulnerable to SQL injection. The backend query directly incorporates user-supplied data from the `category` parameter without proper sanitization. This allows for a `UNION` attack, enabling an attacker to append their own `SELECT` statements to the original query and exfiltrate data from any table within the database.

---

## 🚀 Exploitation Steps

The attack followed a methodical, multi-stage process to enumerate the database and extract the required information.

### 1. Confirming Column Count and Data Types

Before exfiltrating data, it's essential to determine the structure of the original query. I used the following payload to identify the number of columns and confirm they are suitable for string data:

```sql
'+UNION+SELECT+'abc','def'--
```
*   **Payload Breakdown:**
    *   `'`: Closes the original string parameter in the `WHERE` clause.
    *   `UNION SELECT 'abc','def'`: Appends a query that selects two string literals.
    *   `--`: A comment that neutralizes the rest of the original query, preventing syntax errors (works for MySQL, PostgreSQL, etc.).

The successful response confirmed that the query returns **two columns** that can both display text.

### 2. Listing Database Tables

The next step was to discover the names of the tables within the database. The `information_schema` is a standard metadata database available in most non-Oracle SQL flavors (like MySQL, PostgreSQL, MS-SQL) and is perfect for this.

```sql
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```
*   **Payload Breakdown:**
    *   `SELECT table_name, NULL`: Selects the `table_name` column from the `information_schema.tables` view. `NULL` is used as a placeholder for the second column.

This payload returned a list of all tables. A table with a promising name like `users_abcdef` was identified as the likely target.

### 3. Enumerating Columns in the Users Table

After finding the table, I needed to know the names of the columns within it to target the usernames and passwords.

```sql
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```
*   **Payload Breakdown:**
    *   `SELECT column_name, NULL`: Selects the `column_name` from the `information_schema.columns` view.
    *   `WHERE table_name='users_abcdef'`: Filters the results to show columns only from the target table.

This revealed the column names, such as `username_abcdef` and `password_abcdef`.

### 4. Retrieving Usernames and Passwords

With the table and column names known, the final step was to extract the credentials.

```sql
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
```
*   **Payload Breakdown:**
    *   This is a straightforward `SELECT` query that pulls the username and password from every row in the `users_abcdef` table. Since we have two text-compatible columns, both pieces of data can be extracted simultaneously.

This payload displayed a list of all users and their passwords.

### 5. Logging in as Administrator

From the extracted data, I located the password associated with the `administrator` user and used these credentials to log in to the application, thus completing the lab.

---

## ✅ Key Takeaways

*   **The Power of `information_schema`:** This metadata database is the primary tool for enumerating the structure (tables, columns) of non-Oracle databases.
*   **Methodical Enumeration:** Successful SQL injection attacks are often not a single payload but a sequence of queries that build on each other: find columns -> find tables -> find columns -> extract data.
*   **Payload Versatility:** The same two-column vulnerability was used to extract four different types of information, demonstrating how a single flaw can lead to a full database compromise.
