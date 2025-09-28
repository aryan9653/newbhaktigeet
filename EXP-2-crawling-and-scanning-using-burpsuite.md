# Experiment 02: Crawling and scanning a web application using Burp Spider and Scanner.

**Disclaimer:** This experiment must be conducted only on the Mutillidae II application running in your local, isolated virtual machine. Using web crawlers and vulnerability scanners on any website without explicit, written permission from the owner is illegal and unethical.

---

### **1. Aim**

Crawling and scanning a web application using Burp Spider and Scanner.

---

### **2. Learning Objective**

To understand and utilize Burp Spider to discover the content and functionality of a target web application and to use Burp Scanner's capabilities to automatically identify potential security weaknesses.

---

### **3. Tools Required**

- Kali Linux Virtual Machine
- Burp Suite (Community or Professional Edition)
- Mutillidae II (running on the local Apache server)

---

### **4. Theory**

#### **4.1 What is Web Application Crawling (Spidering)?**

Crawling, also known as spidering, is the process of systematically browsing a website to identify and map all of its pages, links, directories, and functionality. The goal is to build a complete "site map" of the application. This map is crucial for a security tester because it reveals the entire **attack surface**—all the different endpoints and parameters that can be tested for vulnerabilities.

Burp Suite performs crawling in two ways:

- **Passive Crawling:** As you manually browse the web application, Burp Suite passively logs every page you visit and analyzes the responses to find links to other pages, building a site map in the background.
- **Active Crawling (Spidering):** You can instruct Burp Spider to take over, automatically following every link it finds on a page, submitting forms with dummy data, and recursively exploring the application to discover content you might have missed manually.

#### **4.2 What is Vulnerability Scanning?**

Vulnerability scanning is the process of actively probing a web application to find security flaws. After crawling has identified _what_ to test, the scanner sends a large number of crafted requests designed to trigger known vulnerability patterns, such as SQL injection, Cross-Site Scripting (XSS), command injection, and more. It then analyzes the application's responses to determine if a vulnerability is present.

- **Burp Scanner (Professional Edition):** The full-featured **Active Scanner** is a core component of Burp Suite Professional. It is a powerful, automated tool that can detect a wide range of vulnerabilities with high accuracy.
- **Passive Scanning (Community Edition):** The free Community Edition of Burp Suite can still perform **passive scans**. It won't send new, malicious requests, but it will analyze all the normal traffic passing through the proxy for issues that can be detected without active probing. Examples include finding verbose error messages, identifying insecure cookie flags, or discovering information disclosure.

---

### **5. Procedure**

#### **Step 1: Prepare the Environment**

1.  Start your Kali Linux VM.
2.  Open a terminal and ensure your web and database services are running:
    ```bash
    sudo service apache2 start
    sudo service mysql start
    ```
3.  Launch Burp Suite from `Applications` -> `Web Application Analysis` -> `Burp Suite`.
4.  Open Firefox and ensure its proxy is configured to `127.0.0.1:8080`.
5.  Navigate to `http://localhost/mutillidae` to confirm the application is running.

#### **Step 2: Set the Target Scope**

Setting a scope is crucial to ensure your tools only test the intended application and not other sites you might visit.

1.  In Burp Suite, go to the **Proxy** -> **HTTP history** tab.
2.  Find a request to `http://localhost` in the list.
3.  Right-click on it and select **"Add to scope"**.
4.  Burp will ask if you want to stop logging out-of-scope items. It's helpful to click **"Yes"**.
5.  Go to the **Target** -> **Scope** tab to confirm that `http://localhost` is now listed.

#### **Step 3: Passive Crawling (Manual Discovery)**

1.  In Firefox, browse the Mutillidae application. Click on various links in the OWASP Top 10 menu (e.g., A1 - Injection, A2 - Broken Authentication).
2.  As you click around, switch back to Burp Suite and look at the **Target** -> **Site map** tab.
3.  You will see the site map for `http://localhost` being populated with every page and resource you access. This is passive crawling.

#### **Step 4: Active Crawling with Burp Spider**

Now we will let Burp automatically find more content.

1.  In the **Target** -> **Site map** tab, right-click on the top-level entry for `http://localhost`.
2.  From the context menu, select **"Scan"**.
3.  A dialog box will appear. Select **"Crawl"** and keep the default settings. Click OK.
    - **Note:** In older versions of Burp, this option was called "Spider this host".
4.  Burp will now start actively crawling the application. You can monitor its progress in the **Dashboard** tab under "Tasks".
5.  Once the crawl is complete, revisit the **Target** -> **Site map**. You will likely see many new grayed-out entries that the Spider discovered but you haven't visited yourself.

#### **Step 5: Performing a Scan**

**For Burp Suite Community Edition Users (Passive Scan):**
The passive scan has already been happening!

1.  Go to the **Dashboard** tab. Look at the **"Issue Activity"** panel.
2.  Alternatively, go to the **Target** -> **Site map** tab. Select the `http://localhost` entry.
3.  In the bottom panel, click on the **"Issues"** tab.
4.  Burp Community will report any passive vulnerabilities it has found during your browsing and crawling, such as "Cookie without HttpOnly flag set" or "Cross-domain script include."

**For Burp Suite Professional Edition Users (Active Scan):**

1.  In the **Target** -> **Site map** tab, right-click on `http://localhost`.
2.  Select **"Scan"**.
3.  This time, choose **"Crawl and Audit (Intelligent)"**. This will perform both a crawl and a full active vulnerability scan.
4.  Click OK to start the scan.

#### **Step 6: Analyze the Results**

1.  The primary place to view findings is the **Dashboard** tab, which shows a summary of all discovered vulnerabilities, categorized by severity (High, Medium, Low, Information).
2.  For details, select `http://localhost` in the **Target** -> **Site map** and view the **"Issues"** tab.
3.  Click on any reported issue (e.g., "SQL Injection").
4.  The right-hand panel will provide a detailed **Description** of the vulnerability, the specific **Request** and **Response** that triggered it, and valuable **Remediation** advice.

---

### **6. Conclusion**

In this experiment, we successfully mapped the attack surface of the Mutillidae web application using both passive and active crawling techniques with Burp Suite. We learned how to define a target scope to focus our testing efforts. By analyzing the traffic, Burp Suite's passive scanning capabilities identified several low-severity issues. Furthermore, we outlined the procedure for launching an active scan with Burp Suite Professional, which would automate the process of sending malicious payloads to discover more critical vulnerabilities like SQL Injection and XSS. This process of crawling and scanning is a fundamental first step in any web application security assessment.

---

### **7. Learning Outcome**

- Learned to effectively define a target scope in Burp Suite to focus testing activities.
- Successfully used passive and active crawling (Spider) to build a comprehensive site map of a web application.
- Understood how to view and interpret the results from Burp's passive scanner.
- Gained knowledge of how to initiate an active vulnerability scan and analyze its detailed findings for vulnerability details and remediation guidance.
