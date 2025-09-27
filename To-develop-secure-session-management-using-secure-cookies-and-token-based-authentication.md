# Experiment 7: Secure Session Management

**Aim:** To develop secure session management using secure cookies and token-based authentication.

---

### **Learning Objective**

To understand and implement secure session management techniques using both HttpOnly/Secure cookies and JSON Web Tokens (JWT) to protect against common session-based attacks.

---

### **Tools Required**

- **Python 3:** A recent version of Python.
- **Flask:** A micro web framework for Python. (`pip install Flask`)
- **Flask-JWT-Extended:** An extension for Flask that adds JWT support. (`pip install Flask-JWT-Extended`)
- **Web Browser:** Any modern browser like Chrome, Firefox, or Edge with developer tools.
- **API Client:** A tool like [Postman](https://www.postman.com/) or a command-line tool like `curl` to test the API.

---

### **Theoretical Background**

Session management is a fundamental concept in web application development. Because the HTTP protocol is inherently stateless, applications need a way to identify and maintain a user's state across multiple requests. When a user authenticates (e.g., logs in), the server establishes a "session" and provides the client with a unique identifier. This identifier, whether a session ID or a token, is then sent back by the client with every subsequent request, allowing the server to recognize the user.

However, if not implemented correctly, session management can become a major security loophole, exposing the application to severe vulnerabilities such as:

- **Session Hijacking:** An attacker steals a user's valid session identifier and uses it to impersonate the user, gaining unauthorized access to their account and data.
- **Cross-Site Scripting (XSS):** If an attacker can inject malicious scripts into a web page viewed by other users, they can use that script to steal session cookies.
- **Cross-Site Request Forgery (CSRF):** An attacker tricks a logged-in user into performing an unwanted action on a web application in which they're currently authenticated.

To counter these threats, developers employ robust session management strategies. This experiment explores two popular and secure methods:

#### **1. Secure Cookies (Stateful Sessions)**

This is the traditional approach where the server creates and manages the session state.

- **How it works:** Upon successful login, the server generates a unique session ID and stores it alongside the user's session data (like username, role, etc.) on the server-side (e.g., in memory, a database, or a cache). This session ID is then sent to the client and stored in a cookie. For every new request, the browser automatically sends this cookie back, and the server uses the ID to look up the user's session data.
- **Security Measures:** To secure this process, we configure specific attributes on the cookie:
  - `HttpOnly`: This is a crucial flag that prevents client-side scripts (like JavaScript) from accessing the cookie. It is a primary defense against XSS attacks aimed at stealing session cookies.
  - `Secure`: This flag ensures that the browser will only send the cookie over an encrypted HTTPS connection. This prevents attackers from intercepting the cookie in transit over unencrypted networks (man-in-the-middle attacks).
  - `SameSite`: This attribute helps mitigate CSRF attacks. It controls whether a cookie is sent with cross-origin requests. It has three possible values:
    - `Strict`: The cookie is only sent for requests originating from the same site.
    - `Lax`: The cookie is sent with same-site requests and with top-level navigations from external sites (e.g., clicking a link). This is a good balance between security and usability.
    - `None`: The cookie is sent with all requests, but it **must** be used in conjunction with the `Secure` attribute.

#### **2. Token-Based Authentication (Stateless Sessions with JWT)**

This is a modern, stateless approach that is particularly popular with APIs and Single-Page Applications (SPAs).

- **How it works:** Instead of storing session information on the server, the server generates a JSON Web Token (JWT) upon successful login. A JWT is a compact, self-contained JSON object that contains all the necessary information about the user (known as "claims"). This token is cryptographically signed by the server using a secret key. The token is then sent to the client, which typically stores it in memory (e.g., a JavaScript variable) or `localStorage`/`sessionStorage`.
- - **Authentication Flow:** For subsequent requests to protected resources, the client sends the JWT in the `Authorization` header, usually with the `Bearer` schema (e.g., `Authorization: Bearer <token>`). The server receives the request, extracts the token, and verifies its signature using the secret key. If the signature is valid, the server trusts the information within the token and processes the request. Because the server doesn't need to look up session data, this approach is "stateless," making it highly scalable.

---

### **Procedure / Implementation**

This experiment is divided into two parts. First, we will implement a traditional session using secure cookies. Second, we will build a small API that uses JWT for authentication.

**Disclaimer:** The following code examples use hardcoded secret keys and user credentials for educational purposes. In a real-world application, you must use environment variables or a secrets management system for keys and a secure database for user authentication.

---

### **Part 1: Implementing Secure Session Management with Cookies**

In this part, we'll create a Flask application that uses session cookies and configures them with security attributes.

#### **Step 1: Install Flask**

If you haven't already, open your terminal or command prompt and install Flask:

```sh
pip install Flask
```

#### **Step 2: Create the Flask Application**

Create a Python file named `secure_cookie_app.py` and add the following code:

```python
from flask import Flask, session, request, redirect, url_for, render_template_string
from datetime import timedelta
import os

app = Flask(__name__)

app.secret_key = os.urandom(24)

app.config.update(
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SECURE=True,

    SESSION_COOKIE_SAMESITE='Lax',
    PERMANENT_SESSION_LIFETIME=timedelta(minutes=30)
)

LOGIN_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <h2>Login Page</h2>
    <form method="post">
        Username: <input type="text" name="username" required>
        <button type="submit">Login</button>
    </form>
</body>
</html>
"""

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        if request.form.get('username') == 'admin':
            session['user'] = 'admin'
            session.permanent = True 
            return redirect(url_for('dashboard'))
        else:
            return "Invalid credentials!", 401
    return render_template_string(LOGIN_TEMPLATE)

@app.route('/dashboard')
def dashboard():
    if 'user' in session:
        return f"<h1>Welcome to the Dashboard, {session['user']}!</h1>"
    else:
        return redirect(url_for('login'))

@app.route('/logout')
def logout():
    session.pop('user', None) # Remove the user from the session
    return redirect(url_for('login'))


if __name__ == '__main__':
    app.run(debug=True, ssl_context='adhoc', port=5000)

```

#### **Step 3: Run the Application and Test**

1.  **Run the script** from your terminal:
    ```sh
    python secure_cookie_app.py
    ```
2.  **Open your browser** and navigate to `https://127.0.0.1:5000/login`. Your browser will likely display a security warning because we are using a self-signed certificate. Proceed to the site.
3.  **Log in** with the username `admin`.
4.  **Inspect the cookie:**
    - Right-click on the page and select "Inspect" to open the Developer Tools.
    - Go to the "Application" tab (in Chrome) or "Storage" tab (in Firefox).
    - On the left pane, expand the "Cookies" section and select the URL (`https://127.0.0.1:5000`).
    - You will see a cookie named `session`. Click on it.
    - Observe the details of the cookie. You will see that the `HttpOnly` and `Secure` checkboxes are ticked, and the `SameSite` attribute is set to `Lax`.
    -

---

### **Part 2: Implementing Secure Session Management with JWT**

Here, we will create a simple API where authentication is handled by JSON Web Tokens.

#### **Step 1: Install Required Libraries**

Install both Flask and the Flask-JWT-Extended library:

```sh
pip install Flask Flask-JWT-Extended
```

#### **Step 2: Create the JWT Flask Application**

Create a new Python file named `jwt_app.py` and add the following code:

```python
from flask import Flask, jsonify, request
from flask_jwt_extended import create_access_token, jwt_required, get_jwt_identity, JWTManager
from datetime import timedelta
import os

app = Flask(__name__)

app.config["JWT_SECRET_KEY"] = os.urandom(24)
app.config["JWT_ACCESS_TOKEN_EXPIRES"] = timedelta(minutes=15)

jwt = JWTManager(app)

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get("username", None)
    password = request.json.get("password", None)
    if username != 'apiuser' or password != 'password123':
        return jsonify({"msg": "Bad username or password"}), 401

    access_token = create_access_token(identity=username)
    return jsonify(access_token=access_token)

@app.route('/profile')
@jwt_required()
def profile():
    current_user = get_jwt_identity()
    return jsonify(logged_in_as=current_user), 200

if __name__ == '__main__':
    app.run(debug=True, port=5001)
```

#### **Step 3: Test the API with `curl` or Postman**

1.  **Run the script** from your terminal:

    ```sh
    python jwt_app.py
    ```

2.  **Login to get a token:** Open a new terminal and use `curl` to send a POST request to the `/login` endpoint.

    ```sh
    curl -X POST http://127.0.0.1:5001/login \
    -H "Content-Type: application/json" \
    -d '{"username": "apiuser", "password": "password123"}'
    ```

    The server will respond with a JSON object containing your access token. It will look something like this:

    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTY2ODU4Mz..."
    }
    ```

3.  **Access the protected endpoint:**

    - Copy the `access_token` value from the response above.
    - Now, make a request to the `/profile` endpoint, passing the token in the `Authorization` header. Replace `YOUR_TOKEN_HERE` with the token you copied.

    <!-- end list -->

    ```sh
    curl -X GET http://127.0.0.1:5001/profile \
    -H "Authorization: Bearer YOUR_TOKEN_HERE"
    ```

    If the token is valid, you will receive a success response:

    ```json
    {
      "logged_in_as": "apiuser"
    }
    ```

    - If you try to access the endpoint without a token or with an invalid token, you will get a `401 Unauthorized` error.

---

### **Learning Outcome**

Through this experiment, you have successfully configured and deployed two different secure session management mechanisms. You have implemented stateful sessions using secure cookie attributes (`HttpOnly`, `Secure`, `SameSite`) in Flask to protect a traditional web application. Additionally, you have implemented a stateless session mechanism using JSON Web Tokens (JWT), demonstrating how to secure API endpoints and manage authentication without relying on server-side session storage.

### **Conclusion**

This experiment demonstrated two robust and widely-used methods for securing user sessions in web applications. Configuring cookies with `HttpOnly`, `Secure`, and `SameSite` attributes is a fundamental and critical defense for traditional web applications against session hijacking and CSRF. For modern APIs and single-page applications, stateless JWTs offer a scalable, flexible, and secure alternative. A thorough understanding and correct implementation of these techniques are essential for preventing common session-based attacks and for ensuring the integrity and confidentiality of user data.
