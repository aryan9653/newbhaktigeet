# Experiment 01: To install and set up Burp Suite, Mutillidae, and Kali Linux for web application security testing.

**Disclaimer:** The tools and techniques described in this experiment are for educational purposes only. They should be used exclusively within a controlled, legal, and authorized environment, such as your own virtual machine setup. Attempting to use these tools on any system or application without explicit permission from the owner is illegal and unethical.

---

### **1. Aim**

To install and configure a complete, self-contained environment for web application security testing using Kali Linux, Burp Suite, and the Mutillidae II deliberately vulnerable web application.

---

### **2. Tools & Components**

- **Kali Linux:** A Debian-based Linux distribution specifically designed for penetration testing and security auditing. It comes pre-packaged with a vast array of security tools, saving significant setup time.
- **Burp Suite (Community Edition):** An industry-standard integrated platform for performing security testing of web applications. It acts as a man-in-the-middle proxy, allowing you to intercept, inspect, and modify traffic between your browser and a web server.
- **Mutillidae II:** A free, open-source, and deliberately vulnerable web application (DVWA). It provides a safe and legal environment to practice identifying and exploiting common web application vulnerabilities.

---

### **3. Theory**

#### **3.1 The Need for a Dedicated Testing Environment**

A security testing environment is an isolated setup that mimics a real-world production system but is safely sandboxed. This allows security professionals and students to conduct penetration tests, vulnerability assessments, and other security experiments without risking damage to live systems or data. Our environment will use Kali Linux as the attacker machine, Burp Suite as our primary analysis tool, and Mutillidae as our target application, all running locally.

#### **3.2 Kali Linux: The Penetration Tester's OS**

Kali Linux is the operating system of choice for cybersecurity professionals. It consolidates hundreds of powerful security tools into a single, cohesive platform. By using Kali, you avoid the tedious process of finding, installing, and configuring these tools individually. It includes utilities for every stage of a penetration test, from information gathering and vulnerability analysis to exploitation and post-exploitation.

#### **3.3 Burp Suite: The Web Proxy**

Burp Suite is an essential tool for any web application security tester. It functions as an **intercepting proxy**, sitting between the user's browser and the target web application. This allows you to see and manipulate all HTTP/S requests and responses in transit. Its key modules include:

- **Proxy:** The core of Burp Suite. It allows you to intercept, view, and modify all traffic passing through it in real-time.
- **Spider:** A tool to automatically crawl a web application, discovering and mapping its content and functionality.
- **Repeater:** A simple but invaluable tool for manually modifying and reissuing individual HTTP requests and analyzing the application's response. This is useful for testing for flaws by tweaking request parameters.
- **Intruder:** A powerful tool for automating custom attacks. It can be used to perform fuzzing, brute-force attacks on login forms, and enumerate identifiers.
- **Sequencer:** A tool for analyzing the randomness of session tokens or other important data items that are intended to be unpredictable.
- **Scanner (Pro Version Only):** An automated vulnerability scanner that can discover a wide range of security flaws.

#### **3.4 Mutillidae: The Safe Target**

To learn how to find and exploit vulnerabilities, you need a safe and legal target. Mutillidae is a web application packed with known vulnerabilities, mirroring common flaws found in real-world applications (based on the OWASP Top 10 and more). Practicing on Mutillidae ensures you understand how vulnerabilities work in a hands-on way without breaking the law or causing harm.

---

### **4. Detailed Steps for Installation and Setup {this has iso setup, you can also directly use a prebuilt kali image}**

#### **Step 1: Kali Linux Installation (Virtual Machine)**

**Note:** The recommended approach for beginners is to use a virtual machine (VM). This isolates the testing environment from your main operating system (host OS) and makes it easy to revert to a clean state if something goes wrong.

1.  **Download Virtualization Software:**

    - If you don't already have one, download and install a hypervisor like [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads) (free) or [VMware Workstation Player](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html) (free for non-commercial use).

2.  **Download the Kali Linux ISO:**

    - Navigate to the official Kali Linux download page: [https://www.kali.org/get-kali/](https://www.kali.org/get-kali/).
    - Download the **"Installer"** image recommended for your system's architecture (usually 64-bit).

3.  **Create a New Virtual Machine:**

    - Open your virtualization software (e.g., VirtualBox).
    - Click "New" to create a new VM.
    - **Name:** `Kali Linux`
    - **Type:** `Linux`
    - **Version:** `Debian (64-bit)`
    - **Memory Size:** Allocate at least 2048 MB (2 GB) of RAM. **4096 MB (4 GB) is recommended** for better performance.
    - **Hard Disk:** Select "Create a virtual hard disk now". Choose `VDI (VirtualBox Disk Image)` and `Dynamically allocated`. Set the size to at least **30 GB**.

4.  **Install Kali Linux on the VM:**
    - Select your newly created Kali VM and click "Start".
    - You will be prompted for a startup disk. Click the folder icon and select the Kali Linux ISO file you downloaded.
    - The Kali installer will boot. Choose **"Graphical Install"** and follow the on-screen instructions. You will set your language, location, keyboard layout, and create a user account with a password.
    - When prompted for disk partitioning, you can safely choose **"Guided - use entire disk"** as it will only use the virtual disk you created.
    - After the installation completes, the VM will reboot. You can now log in to your Kali Linux desktop.

#### **Step 2: Burp Suite Configuration**

Burp Suite comes pre-installed on Kali Linux. The main task is to configure Firefox to route its traffic through Burp's proxy.

1.  **Launch Burp Suite:**

    - Go to the `Applications` menu (top-left corner).
    - Navigate to `Web Application Analysis` -> `Burp Suite`.
    - A dialog will pop up. You can leave the defaults ("Temporary project") and click "Next", then "Start Burp".

2.  **Configure Firefox Proxy Settings:**

    - Open the Firefox browser in Kali.
    - Click the hamburger menu (☰) in the top-right corner and go to `Settings`.
    - In the "General" tab, scroll down to the **"Network Settings"** section and click "Settings...".
    - Select the **"Manual proxy configuration"** radio button.
    - Set **HTTP Proxy:** `127.0.0.1`
    - Set **Port:** `8080`
    - Check the box **"Also use this proxy for all protocols"**.
    - Click "OK".

3.  **Install Burp's CA Certificate (Crucial for HTTPS):**

    - To intercept encrypted HTTPS traffic without constant browser security errors, you must install Burp's certificate.
    - With Burp running and the proxy configured, navigate to the special address `http://burpsuite` in Firefox.
    - You should see a page with a "CA Certificate" link in the top-right corner. Click it to download the `cacert.der` file.
    - In Firefox's `Settings`, go to `Privacy & Security`.
    - Scroll down to the **"Certificates"** section and click "View Certificates...".
    - In the Certificate Manager window, go to the **"Authorities"** tab and click "Import...".
    - Select the `cacert.der` file you just downloaded.
    - A dialog will pop up. Check the box **"Trust this CA to identify websites."** and click "OK".

4.  **Verify the Setup:**
    - In Burp Suite, go to the `Proxy` -> `Intercept` tab. Ensure "Intercept is on".
    - In Firefox, try to visit any website (e.g., `https://example.com`).
    - The browser should appear to be loading, and the request details should appear in the Burp Suite Intercept tab. You can click "Forward" to send the request and "Intercept is off" to let traffic pass through while still logging it in the `HTTP history` sub-tab.

#### **Step 3: Mutillidae Installation & Setup**

1.  **Open a Terminal:**

    - Click the terminal icon in the top toolbar or find it in the Applications menu.

2.  **Start Required Services:**

    - The Apache web server and MySQL database server must be running. Start them with the following commands:

    ```bash
    sudo service apache2 start
    sudo service mysql start
    ```

3.  **Download Mutillidae:**

    - Use `git` to clone the Mutillidae repository from GitHub.

    ```bash
    git clone [https://github.com/webpwnized/mutillidae.git](https://github.com/webpwnized/mutillidae.git)
    ```

4.  **Move Mutillidae to the Web Root:**

    - The Apache server serves files from the `/var/www/html/` directory. Move the downloaded files there.

    ```bash
    sudo mv mutillidae /var/www/html/
    ```

5.  **Set Permissions (Important Fix):**

    - The web server needs permission to write configuration files. A common point of failure is incorrect permissions.

    ```bash
    sudo chmod -R 777 /var/www/html/mutillidae
    ```

    _Note: `777` permissions (read, write, execute for everyone) are insecure for a production server but are acceptable for this local, isolated learning environment._

6.  **Initialize the Database:**
    - Open Firefox and navigate to `http://localhost/mutillidae`.
    - You will be redirected to the setup page. Click the button labeled **"Setup / reset the database"**.
    - This will create the necessary database and tables for the application to function. If you see a success message, the installation is complete and you can browse the vulnerable application.

---

### **5. Conclusion**

This experiment successfully established a fully functional and isolated web application security testing lab. By installing Kali Linux in a virtual machine, we have access to a comprehensive suite of security tools. Configuring Burp Suite as a proxy allows us to intercept and analyze web traffic, a fundamental skill in web security. Finally, deploying Mutillidae provides a safe and legal target to practice identifying and exploiting common web vulnerabilities. This setup is the foundation for all subsequent web security experiments.

---

### **6. Theory Questions & Answers**

1.  **What are the different modules provided by Burp Suite for testing web vulnerabilities?**

    - Burp Suite provides several key modules: **Proxy** (to intercept and modify traffic), **Spider** (to map application content), **Repeater** (to manually reissue and modify requests), **Intruder** (to automate customized attacks like brute-forcing), and **Sequencer** (to analyze token randomness). The Professional version also includes the **Scanner** module for automated vulnerability detection.

2.  **Explain how a browser is configured to work with Burp Suite’s proxy.**

    - To work with Burp Suite, a browser's network settings must be manually configured to route all its traffic to the address and port where Burp's proxy listener is running. By default, this is `127.0.0.1` (localhost) on port `8080`. This configuration tells the browser to send all HTTP and HTTPS requests to Burp Suite instead of directly to the internet. For HTTPS traffic to be intercepted without errors, Burp's custom CA certificate must also be imported into the browser's certificate trust store.

3.  **What is the purpose of Mutillidae in web security testing?**
    - Mutillidae serves as a deliberately vulnerable web application (DVWA). Its purpose is to provide a safe, legal, and controlled environment for security professionals and students to practice their skills. Instead of illegally testing on real websites, users can attack Mutillidae to learn how to identify, exploit, and remediate common security flaws like SQL Injection, Cross-Site Scripting (XSS), and insecure authentication in a hands-on manner.

---

### **7. Learning Outcomes**

- Demonstrate the ability to successfully install and configure Kali Linux within a virtualized environment.
- Understand and apply the necessary steps to configure a web browser to proxy traffic through Burp Suite, including the installation of the CA certificate for HTTPS interception.
- Successfully deploy the Mutillidae II vulnerable web application on a local Apache/MySQL server, creating a complete and secure testing environment for web application security.
