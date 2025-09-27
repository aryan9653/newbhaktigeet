**Aim:** To implement OAuth 2.0 and JWT-based API authentication in a sample web application.

### **Learning Objective**

To understand and implement a modern authentication flow that combines OAuth 2.0 for third-party login (e.g., with Google) and JSON Web Tokens (JWT) for securing internal API endpoints.

### **Tools & Prerequisites**

- **Python 3:** A recent version of Python installed.
- **Flask & Extensions:** Required Python libraries.
  - `pip install Flask Flask-Dance Flask-JWT-Extended`
- **Google Cloud Platform Account:** To create OAuth 2.0 credentials.
- **Web Browser:** Any modern browser like Chrome, Firefox, or Edge.
- **API Client:** A tool like Postman or the command-line tool `curl`.

### **Theoretical Background**

Modern applications often decouple the process of **authentication** (proving who you are) from **authorization** (what you are allowed to do). This experiment demonstrates a powerful pattern that combines two industry standards: OAuth 2.0 and JWT.

#### **1. OAuth 2.0: The Authorization Framework** 🤝

OAuth 2.0 is an **authorization framework**, not an authentication protocol. It allows a user to grant a third-party application limited access to their resources on another service, without sharing their password.

Think of it like a hotel key card system:

- **You (Resource Owner):** The guest who owns the right to access the room.
- **Your App (Client):** The hotel guest asking for a key card.
- **Google (Authorization Server):** The front desk that checks your ID and issues the key card.
- **Google User Info (Resource Server):** The hotel room door that accepts the key card.

The flow allows your application to ask Google, "Does this user grant me permission to see their email address?" Google asks you for consent, and if you agree, it gives your application a temporary access token (the key card) to fetch that specific information. Crucially, your app **never sees your Google password**.

#### **2. JSON Web Token (JWT): The Digital Passport** 🛂

A JSON Web Token (JWT) is a compact and self-contained standard for securely transmitting information between parties as a JSON object. Once our app has authenticated the user via OAuth 2.0, it can issue its own JWT. This JWT acts like a digital passport for the user within our application.

A JWT consists of three parts separated by dots (`.`):

1.  **Header:** Contains the token type (`JWT`) and the signing algorithm (e.g., `HS256`).
2.  **Payload:** Contains the "claims," which are statements about the user (e.g., their email, user ID, expiration time of the token).
3.  **Signature:** A cryptographic signature created using the header, the payload, and a secret key known only to the server.

The server signs the token, gives it to the client, and for every subsequent API request, the client includes this JWT. The server can then instantly verify the token's signature to confirm that the request is from an authenticated user and that the token hasn't been tampered with. This creates a **stateless authentication system**, as the server doesn't need to store session information.

### **Procedure / Implementation**

This practical is divided into three main parts: setting up Google credentials, developing the Flask application, and testing the entire authentication flow.

### **Part 1: Setup and Configuration**

#### **Step 1: Install Required Libraries**

Open your terminal or command prompt and run the following command to install all necessary Python packages:

```sh
pip install Flask Flask-Dance Flask-JWT-Extended
```

#### **Step 2: Configure Google OAuth 2.0 Credentials**

You need to tell Google about your application so it can issue credentials.

1.  **Go to the Google Cloud Console:** Navigate to `console.cloud.google.com`.
2.  **Create or Select a Project:** Either create a new project or select an existing one from the top dropdown menu.
3.  **Navigate to Credentials:** In the navigation menu (☰), go to **APIs & Services \> Credentials**.
4.  **Create OAuth Client ID:** Click on **+ CREATE CREDENTIALS** and select **OAuth client ID**.
5.  **Configure Consent Screen:** If prompted, you'll need to configure the OAuth consent screen.
    - Choose **External** for the User Type.
    - Fill in the required fields: App name, User support email, and Developer contact information. Click **SAVE AND CONTINUE** through the Scopes and Test Users sections.
6.  **Set Application Type:** Select **Web application** from the dropdown list.
7.  **Add Redirect URI:** This is a critical step. Under **Authorized redirect URIs**, click **+ ADD URI** and enter:
    - `http://127.0.0.1:5000/login/google/authorized`
      This tells Google where to send the user back after they have successfully authenticated.
8.  **Create and Copy Credentials:** Click the **CREATE** button. A pop-up will appear with your **Client ID** and **Client Secret**. Copy these two values and save them somewhere safe. You will need them for the application.

**⚠️ Security Warning:** Your **Client ID** and **Client Secret** are sensitive credentials. Never commit them to a public code repository (like GitHub). In a real application, use environment variables or a secrets management service.

### **Part 2: Application Development**

Create a Python file named `oauth_jwt_app.py` and add the following code.

**Important:** Replace the placeholder values for `client_id` and `client_secret` with the actual credentials you copied from the Google Cloud Console.

```python
import os
from flask import Flask, redirect, url_for, jsonify
from flask_dance.contrib.google import make_google_blueprint, google
from flask_jwt_extended import JWTManager, create_access_token, jwt_required, get_jwt_identity

app = Flask(__name__)
# This secret key is for signing the Flask session cookie, not the JWT.
app.secret_key = os.urandom(24)
# This secret key is for signing the JWT.
app.config["JWT_SECRET_KEY"] = os.urandom(24)
jwt = JWTManager(app)

# This line allows OAuthlib to work with HTTP locally for development.
# In production, you MUST use HTTPS.
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'

# Create a Flask-Dance blueprint for Google OAuth
google_bp = make_google_blueprint(
    # Replace with your Client ID from Google Cloud Console
    client_id="YOUR_GOOGLE_CLIENT_ID_HERE",
    # Replace with your Client Secret from Google Cloud Console
    client_secret="YOUR_GOOGLE_CLIENT_SECRET_HERE",
    redirect_url="/login/google/authorized",
    # Scopes define what information we are asking for from Google
    scope=["profile", "email"]
)

# Register the blueprint with the main Flask app
app.register_blueprint(google_bp, url_prefix="/login")

@app.route("/")
def home():
    """Home page with a link to log in via Google."""
    return '<a href="/login/google"><h1>Login with Google</h1></a>'

@app.route("/login/google/authorized")
def authorized():
    """Callback route that Google redirects to after authentication."""
    # Check if the user authorized the request
    if not google.authorized:
        # If not, redirect them back to the Google login page
        return redirect(url_for("google.login"))

    # If authorized, fetch the user's info from Google's userinfo endpoint
    resp = google.get("/oauth2/v2/userinfo")
    assert resp.ok, resp.text
    user_info = resp.json()
    user_email = user_info["email"]

    # Create a JWT for our internal API, using the user's email as the identity
    access_token = create_access_token(identity=user_email)

    # Return the JWT to the user
    return jsonify(message="Login Successful!", access_token=access_token)

@app.route("/api/profile")
@jwt_required() # This decorator protects the endpoint
def profile():
    """A protected API endpoint that requires a valid JWT."""
    current_user = get_jwt_identity() # Get the identity from the JWT
    return jsonify(logged_in_as=current_user, message="You have access to this protected data!")

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

### **Part 3: Testing the Flow**

Now, let's test the complete authentication and authorization process.

1.  **Run the Application:** Open your terminal in the same directory as your `oauth_jwt_app.py` file and run it:

    ```sh
    python oauth_jwt_app.py
    ```

2.  **Initiate Login:** Open your web browser and navigate to `http://127.0.0.1:5000/`. You will see a simple page with a "Login with Google" link. Click it.

3.  **Authenticate with Google:** You will be redirected to the standard Google sign-in page. Log in with your Google account and grant the application permission to access your profile and email.

4.  **Receive the JWT:** After you grant permission, Google will redirect you back to your application's callback URL (`/login/google/authorized`). Your browser will now display a JSON response containing your JWT `access_token`. It will look something like this:

    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVza...",
      "message": "Login Successful!"
    }
    ```

    **Copy the entire long string** value of the `access_token`.

5.  **Access the Protected API:** Open a new terminal window (or use an API client like Postman). Use the `curl` command to make a request to the protected `/api/profile` endpoint.

    - Replace `YOUR_TOKEN_HERE` with the actual JWT you just copied.
    - The token must be sent in the `Authorization` header with the `Bearer` scheme.

    <!-- end list -->

    ```sh
    curl -X GET [http://127.0.0.1:5000/api/profile](http://127.0.0.1:5000/api/profile) \
    -H "Authorization: Bearer YOUR_TOKEN_HERE"
    ```

    If the token is valid, you will receive a successful JSON response:

    ```json
    {
      "logged_in_as": "your.email@gmail.com",
      "message": "You have access to this protected data!"
    }
    ```

### **Learning Outcome**

You have successfully performed an experiment to integrate third-party authentication using the OAuth 2.0 framework with Google. Upon successful user verification, you generated a JSON Web Token (JWT) and used it to implement secure, stateless authentication for a protected API endpoint. This process effectively separates the initial service authentication (handled by Google) from the ongoing application authorization (handled by JWTs).

### **Conclusion**

This experiment successfully demonstrates a modern and robust authentication pattern by leveraging the combined strengths of OAuth 2.0 and JWT. Delegating user authentication to a trusted provider like Google enhances security (by not handling passwords) and improves the user experience. The subsequent use of JWTs for internal API security enables a scalable and stateless architecture, which is a highly effective and standard practice for building secure web services today.
