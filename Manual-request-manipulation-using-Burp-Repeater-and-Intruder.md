# Experiment 03: Manual request manipulation using Burp Repeater and Intruder.

**Disclaimer:** The techniques described here involve sending modified and potentially malicious requests. They must only be used on the Mutillidae II application running within your isolated virtual machine. Using these techniques on any other application without explicit permission is illegal.

---

### **1. Aim**

Manual request manipulation using Burp Repeater and Intruder.

---

### **2. Learning Objective**

To understand how to use Burp Repeater for detailed, manual analysis of an application's responses to unexpected input, and to use Burp Intruder to efficiently test a parameter with a list of payloads to discover vulnerabilities like username enumeration.

---

### **3. Tools Required**

- Kali Linux Virtual Machine
- Burp Suite (Community or Professional Edition)
- Mutillidae II (running on the local Apache server)

---

### **4. Theory**

#### **4.1 What is Burp Repeater?**

Burp Repeater is a tool for manual, fine-grained testing. Its purpose is to take a single HTTP request, allow you to modify it in any way, and send it over and over again. By observing how small changes to the request affect the server's response, you can probe for a wide variety of input-based vulnerabilities.

Repeater is your digital scalpel. It is used to:

- Confirm findings from the automated scanner.
- Test for logical flaws that an automated scanner would miss.
- Manually explore the functionality of an endpoint.
- Craft a proof-of-concept for a suspected vulnerability.

The workflow is simple: **Capture Request -> Send to Repeater -> Modify -> Send -> Analyze Response -> Repeat.**

#### **4.2 What is Burp Intruder?**

Burp Intruder is a powerful tool for automating customized attacks. While Repeater is for manually testing a single request, Intruder is for sending thousands of requests based on a single template. You define "payload positions" within a request (e.g., a username or password parameter) and then provide a list of "payloads" (e.g., a list of common passwords, numbers, or fuzz strings). Intruder then iterates through your payload list, sending a new, slightly different request for each one.

Intruder is a brute-force tool used for:

- **Fuzzing:** Sending a wide range of unexpected and malformed data to see if the application crashes or behaves unexpectedly.
- **Enumeration:** Discovering valid usernames, file paths, or directories by trying a list of possibilities.
- **Brute-force attacks:** Guessing passwords or other credentials for a given username.

The most common attack type is **Sniper**, which inserts one payload from a single list into one defined position per request.

---

### **5. Procedure Part A: Probing with Burp Repeater**

In this part, we will test the login page for a classic SQL Injection vulnerability.

1.  **Prepare the Environment:**

    - Start your Kali VM, Apache/MySQL services, and Burp Suite.
    - Ensure Firefox is proxied through Burp.

2.  **Capture a Login Request:**

    - In Firefox, navigate to `http://localhost/mutillidae`.
    - Go to `OWASP 2017` -> `A1: Injection` -> `SQLi - Bypass Authentication` -> `Login`.
    - In Burp, go to **Proxy -> Intercept** and turn **Intercept off**. Then go to the **HTTP history** sub-tab.
    - In Firefox, enter any fake credentials like `test` / `test` and click the "Login" button.
    - In Burp's **HTTP history**, find the `POST /mutillidae/index.php?page=login.php` request. You'll see the credentials you entered in the request body at the bottom.

3.  **Send to Repeater:**

    - Right-click on that `POST` request in the history list.
    - Select **"Send to Repeater"**.
    - A notification will appear, and the **Repeater** tab will turn orange. Click on it.

4.  **Analyze in Repeater:**

    - You will see the captured request in the left-hand panel. Click the **"Send"** button.
    - The server's response appears in the right-hand panel. You should see an "Invalid credentials" message in the rendered HTML. This is our baseline.

5.  **Test for SQL Injection:**
    - Now, we will send an unexpected character. In the left panel, find the `username` parameter.
    - Change its value from `test` to a single quote: `username='`
    - Click **"Send"** again.
    - Examine the new response on the right. Scroll through the HTML. You will now see a detailed **MySQL database error message**. This error confirms that the application is vulnerable to SQL Injection, as our input broke the underlying database query.

---

### **6. Procedure Part B: Enumeration with Burp Intruder**

Here, we will use Intruder to discover valid usernames on the `user-info.php` page.

1.  **Capture a Target Request:**

    - In Firefox, navigate to `http://localhost/mutillidae/index.php?page=user-info.php`.
    - In the text box, enter a username like `admin` and click "View Account Details".
    - Go back to Burp's **Proxy -> HTTP history** tab. Find the `POST` request to `user-info.php`.

2.  **Send to Intruder:**

    - Right-click on that `POST` request and select **"Send to Intruder"**.
    - The **Intruder** tab will turn orange. Click on it.

3.  **Configure Positions:**

    - Go to the **Intruder -> Positions** tab.
    - Burp Intruder automatically adds payload markers (`§...§`) to what it thinks are interesting parts of the request. We only want to test the username.
    - Click the **"Clear §"** button on the right.
    - Now, in the request editor, highlight the value of the `username` parameter (`admin`).
    - Click the **"Add §"** button. The request should now look like `username=§admin§`. This ensures our attack only targets this one spot.

4.  **Configure Payloads:**

    - Go to the **Intruder -> Payloads** tab.
    - Under "Payload sets", the "Payload type" should be **"Simple list"**.
    - Under "Payload Options [Simple list]", click **"Add"** and enter the following usernames, one by one:
      - `admin`
      - `john`
      - `jeremy`
      - `adrian`
      - `hacker`
      - `nonexistentuser`

5.  **Launch the Attack:**

    - Click the **"Start attack"** button in the top-right corner.
    - A new "Attack" window will open, and Intruder will send a request for each payload in your list.

6.  **Analyze the Results:**
    - In the attack window, you will see a table of results.
    - Click on the header of the **"Length"** column to sort the results by the size of the response.
    - Notice that the responses for most usernames have the same, smaller length.
    - However, the requests for `admin`, `john`, `jeremy`, and `adrian` have a significantly **different (larger) length**. This indicates that the application responded differently because these are **valid usernames**. By observing this difference, we have successfully enumerated users.

---

### **7. Conclusion**

This experiment provided hands-on experience with two of Burp Suite's most powerful manual testing tools. We used **Burp Repeater** to perform precision testing, modifying a request with a single quote to confirm a SQL injection vulnerability by analyzing the unique error in the response. Subsequently, we used **Burp Intruder** to automate a user enumeration attack, demonstrating how to configure payload positions and lists to rapidly identify valid usernames by analyzing differences in response lengths. Mastering Repeater and Intruder is fundamental to moving beyond automated scanning and performing effective, in-depth web application security assessments.

---

### **8. Learning Outcome**

- Learned to send requests from other Burp Suite tools to Burp Repeater and Intruder.
- Successfully manipulated HTTP request parameters in Burp Repeater and analyzed server responses to identify a vulnerability.
- Configured a Burp Intruder "Sniper" attack to target a specific parameter.
- Created a payload list and launched an automated attack to enumerate valid usernames based on differential server responses.
