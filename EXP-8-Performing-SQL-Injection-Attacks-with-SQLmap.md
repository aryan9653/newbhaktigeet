## Experiment 8: Performing SQL Injection Attacks with SQLmap

**Aim:** To perform an automated SQL Injection (SQLi) attack using the tool SQLmap to bypass authentication and extract sensitive data from the underlying database.

---

### 1. Theory

**SQL Injection (SQLi)** is a code injection technique used to attack data-driven applications. It occurs when an attacker inserts or "injects" malicious SQL statements into an entry field for execution by the backend database. A successful SQLi attack can read sensitive data, modify database records, execute administration operations on the database (like shutdown), and in some severe cases, issue commands to the operating system.

The vulnerability stems from applications that construct SQL queries by concatenating user-supplied input directly into the query string. For example, a login query might look like this:

`SELECT * FROM users WHERE username = 'USER_INPUT' AND password = 'PASSWORD_INPUT';`

If the application does not sanitize the inputs, an attacker can manipulate the query structure. By entering `' OR '1'='1' --` in the username field, the query becomes:

`SELECT * FROM users WHERE username = '' OR '1'='1' --' AND password = '...';`

The `'1'='1'` condition is always true, and the `--` comments out the rest of the query. This causes the `WHERE` clause to be true for every row, logging the attacker in as the first user in the database without needing a valid password.

**SQLmap Overview**

SQLmap is a powerful, open-source penetration testing tool that automates the process of detecting and exploiting SQL injection vulnerabilities. It can:
* Identify vulnerable parameters in web applications.
* Perform various types of injection attacks (e.g., boolean-based blind, time-based blind, error-based, UNION query-based).
* Enumerate databases, tables, columns, and data.
* Dump entire database tables or specific entries.
* In some cases, gain an operating system shell or database console access.

---

### 2. Requirements

* **Tool:** SQLmap (pre-installed in Kali Linux or available for download).
* **Vulnerable Application:** Damn Vulnerable Web Application (DVWA).
* **Web Browser:** To capture a valid HTTP request.
* **Local Server Environment:** A server stack (e.g., XAMPP, Docker) to host DVWA.

---

### 3. Disclaimer

> **⚠️ Educational Use Only:** SQLmap is a potent attack tool. Use it **only** against systems you are explicitly authorized to test. Unauthorized use is illegal and can cause significant damage.

---

### 4. Procedure

This procedure uses **Damn Vulnerable Web Application (DVWA)** with the security level set to **Low**.

1.  **Setup and Navigate:**
    * Log in to your local DVWA instance.
    * Navigate to the **SQL Injection** page. This page features a simple form to look up a user by their ID.

2.  **Get a Valid Request and Cookie:**
    * To use SQLmap effectively on an authenticated page, you need to provide it with a valid session cookie.
    * In your browser, open the developer tools (usually F12) and go to the "Network" tab.
    * On the SQL Injection page in DVWA, enter a user ID of `1` and click "Submit".
    * In the Network tab, find the request to the `/vulnerabilities/sqli/` page.
    * You need two pieces of information from this request:
        * **The Full URL:** `http://<DVWA-IP>/vulnerabilities/sqli/?id=1&Submit=Submit#`
        * **The Session Cookie:** From the request headers, copy the entire `Cookie:` header value. It will look something like `security=low; PHPSESSID=a1b2c3d4e5f6g7h8`.

3.  **Run SQLmap to Enumerate Databases:**
    * Open a terminal on your attacking machine (e.g., Kali Linux).
    * Use the following command to make SQLmap test the `id` parameter and list the available databases. Replace the URL and cookie with your own values.
        ```bash
        sqlmap -u "http://<DVWA-IP>/vulnerabilities/sqli/?id=1" --cookie="security=low; PHPSESSID=<YOUR-SESSION-ID>" --dbs
        ```
    * SQLmap will analyze the parameter and should confirm it is vulnerable. It will then list all the databases on the server, including `dvwa` and `information_schema`.

4.  **Enumerate Tables in the `dvwa` Database:**
    * Now, instruct SQLmap to list the tables within the `dvwa` database.
        ```bash
        sqlmap -u "http://<DVWA-IP>/vulnerabilities/sqli/?id=1" --cookie="security=low; PHPSESSID=<YOUR-SESSION-ID>" -D dvwa --tables
        ```
    * SQLmap will list the tables, and you should see a `users` table, which is a high-value target.

5.  **Enumerate Columns in the `users` Table:**
    * Next, find out what columns are in the `users` table.
        ```bash
        sqlmap -u "http://<DVWA-IP>/vulnerabilities/sqli/?id=1" --cookie="security=low; PHPSESSID=<YOUR-SESSION-ID>" -D dvwa -T users --columns
        ```
    * This will show columns like `user_id`, `first_name`, `last_name`, `user`, and `password`.

6.  **Dump Data from the `users` Table:**
    * Finally, tell SQLmap to extract (dump) the contents of the `user` and `password` columns.
        ```bash
        sqlmap -u "http://<DVWA-IP>/vulnerabilities/sqli/?id=1" --cookie="security=low; PHPSESSID=<YOUR-SESSION-ID>" -D dvwa -T users -C user,password --dump
        ```
    * SQLmap will retrieve the data and display it in your terminal. It will also recognize that the passwords are MD5 hashes and ask if you want to try and crack them using a dictionary attack. You can choose 'Y' to attempt this.

---

### 5. Conclusion

This experiment showcased the efficiency and power of automated SQL injection tools like SQLmap. By providing a target URL and a valid session cookie, we were able to confirm a vulnerability and systematically extract sensitive information—usernames and password hashes—from the database with just a few commands. This underscores the critical need for developers to implement **parameterized queries (prepared statements)** instead of using dynamic SQL, as this is the primary and most effective defense against all forms of SQL injection.