# Experiment 4: Web Application Reconnaissance & Information Gathering

> **⚠️ Disclaimer:** This experiment is for educational purposes only. All steps and commands should only be performed on web applications that you have explicit, written permission to test. [cite_start]For this guide, we will use **scanme.nmap.org**[cite: 32], a domain provided by the Nmap project specifically for safe and legal security testing.


## 🎯 Aim
To perform reconnaissance and gather information on a web application. 


## 🧠 Theory

[cite_start]**Reconnaissance** (or "recon") is the foundational, information-gathering phase of any security assessment or attack. [cite: 13] [cite_start]The primary goal is to collect as much data as possible about a target to understand its structure, technology stack, and potential weaknesses. [cite: 14] [cite_start]Think of it like a military scout surveying the landscape before planning an operation; the more information you have, the higher your chances of success. [cite: 15, 16]

### [cite_start]Types of Reconnaissance [cite: 17]

* [cite_start]**Passive Reconnaissance**: This method involves gathering information from publicly available sources without directly interacting with the target's systems. [cite: 18] [cite_start]It is a stealthy and safe approach that utilizes resources like search engines (Google), public records (`whois`), and social media. [cite: 19]
* [cite_start]**Active Reconnaissance**: This method involves directly probing the target's systems to elicit a response. [cite: 20] [cite_start]While this approach yields more detailed technical data like open ports and running services, it is much "louder" and can be detected by security measures like Firewalls and Intrusion Detection Systems (IDS). [cite: 21]

### Key Information to Gather

[cite_start]The objective is to build a complete profile of the target. [cite: 22] Key data points include:
* [cite_start]IP addresses and domain ranges [cite: 24]
* [cite_start]Domain registration details (`whois`) [cite: 25]
* [cite_start]DNS records (e.g., `A`, `MX`) [cite: 26]
* [cite_start]Subdomains (e.g., `api.example.com`, `dev.example.com`) [cite: 27]
* [cite_start]Technologies used (web server, frameworks, CMS) [cite: 28]
* [cite_start]Open ports and the services running on them [cite: 29]


## 🛠️ Prerequisites
* A Linux-based operating system (like Kali Linux) or Windows with WSL2.
* Command-line tools installed: `whois`, `nslookup` (or `dig`), `nmap`, and `curl`.
* A web browser with Developer Tools.
* (Optional) A browser extension like **Wappalyzer**.


## 🔬 Procedure

### Step 1: Passive Reconnaissance (Domain & DNS)

Start by gathering basic information from public records without touching the target's server.

1.  [cite_start]**Whois Lookup**: Use the `whois` command to find domain registration data, including contact information and name servers. [cite: 34]
    ```bash
    whois scanme.nmap.org
    ```
2.  [cite_start]**DNS Lookup**: Use `nslookup` or `dig` to find the IP address (`A` record) and other DNS information associated with the domain. [cite: 34]
    ```bash
    nslookup scanme.nmap.org
    ```

### Step 2: Google Dorking

[cite_start]Leverage advanced search operators ("dorks") to find information indexed by search engines that isn't immediately visible on the website. [cite: 35]

* **Find specific filetypes**:
    ```
    site:example.com filetype:pdf
    ```
* **Find potential login pages**:
    ```
    site:example.com intitle:"admin login"
    ```
* **Find specific URLs**:
    ```
    site:example.com inurl:"dashboard"
    ```

### Step 3: Discover Subdomains

[cite_start]Identify all subdomains to expand the potential attack surface. [cite: 45] You can use online tools for this.

1.  Go to a website like **DNSdumpster.com**.
2.  Enter the target domain (`scanme.nmap.org`) and search.
3.  The tool will map out DNS servers, MX records, and known subdomains.

### Step 4: Active Scanning (Ports & Services)

[cite_start]Now, we shift to active reconnaissance by directly probing the target server with **Nmap**. [cite: 47] [cite_start]This scan will identify open ports and the services running on them. [cite: 48]

* Run a fast scan that also attempts to determine service versions (`-sV`) and runs against common ports (`-F`).
    ```bash
    nmap -sV -F scanme.nmap.org
    ```
    
### Step 5: Identify Web Technologies

[cite_start]Analyze the application to identify its technology stack (server, framework, etc.). [cite: 50]

1.  **Using a Browser Extension**: Install an extension like **Wappalyzer**. [cite_start]Navigate to the target website, and the extension's icon will show you the technologies it detects. [cite: 51]
    2.  **Using `curl`**: You can inspect the HTTP response headers directly from the command line. Look for headers like `Server`, `X-Powered-By`, or unique `Set-Cookie` names that can reveal the technology.
    ```bash
    curl -I [http://scanme.nmap.org](http://scanme.nmap.org)
    ```


## 🎓 Learning Outcome
[cite_start]Upon completing this experiment, you will understand the principles of reconnaissance and its critical role as the first phase of an ethical hacking engagement. [cite: 114] [cite_start]You will also gain practical experience using various tools and techniques to gather technical and infrastructure-related information about a target web application from public and direct sources. [cite: 115]


## 🏁 Conclusion
Reconnaissance is a fundamental and indispensable phase in web application security testing. This experiment demonstrated that by combining passive techniques (like `whois` and Google Dorking) with active methods (like Nmap scanning), an ethical hacker can build a comprehensive profile of a target. The information gathered—from subdomains and IP addresses to the specific software versions in use—directly informs all subsequent phases of a penetration test, enabling a more focused and effective security assessment.
