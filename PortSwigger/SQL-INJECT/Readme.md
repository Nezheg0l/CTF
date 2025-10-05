# PortSwigger Academy: SQL Injection Labs

Welcome to my repository of solved SQL Injection labs from the PortSwigger Web Security Academy. This directory serves as a detailed log of my journey, documenting the techniques, payloads, and methodologies I used to tackle each challenge.

The goal is not just to find the flag, but to understand the vulnerability at its core. Each entry links to a detailed write-up.

---

## 🚀 Completed Labs

Here is a list of the labs I have successfully completed (Practitioner level and above). Each one includes a link to the detailed write-up.

| Lab Title                                                                                               | Difficulty   | Key Technique Explored                               |
| :------------------------------------------------------------------------------------------------------ | :----------: | :--------------------------------------------------- |
| [SQL injection attack, querying the database type and version on Oracle](https://github.com/Nezheg0l/CTF/tree/main/PortSwigger/SQL-INJECT/querying%20the%20database%20Oracle/Readme.md)| `PRACTITIONER🟩` | `UNION` attack, retrieving version strings from `v$version`. |
| [SQL injection attack, querying the database type and version on MySQL and Microsoft](https://github.com/Nezheg0l/CTF/blob/main/PortSwigger/SQL-INJECT/querying%20the%20database%20MySQL%20and%20Microsoft/Readme.md)                                  | `PRACTITIONER🟩` | `UNION` attack, retrieving version strings from `@@version`. |
| [SQL injection attack, listing the database contents on non-Oracle databases](https://github.com/Nezheg0l/CTF/blob/main/PortSwigger/SQL-INJECT/listing%20the%20database%20on%20non-Oracle%20databases/Readme.md)                                  | `PRACTITIONER🟩` | `UNION` attack, retrieving user and pass strings from `information_schema.tables`. |

### How to Navigate
Each lab has its own dedicated directory. Inside, you'll find a `README.md` file that contains a comprehensive write-up covering:
*   **Vulnerability Analysis:** A breakdown of why the vulnerability exists.
*   **Exploitation Steps:** The step-by-step process of exploiting the flaw.
*   **Payloads:** The exact payloads used, with explanations.
*   **Mitigation:** A brief on how to prevent such vulnerabilities.

---

## ⚔️ My Philosophy

"The best way to learn defense is to master the offense. To protect a system, you must first understand how to break it."

This repository is a testament to that belief. It's a continuous effort to sharpen my skills, dive deep into vulnerabilities, and think like an attacker to ultimately build more secure systems.

Got questions or suggestions? Feel free to open an issue or connect with me.

---
*This repository is for educational purposes only.*
