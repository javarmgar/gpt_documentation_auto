# Facebook Sign-In integration in an Android app with a Django backend

### **Table of Contents**

1. **Introduction**
   - Overview of Facebook Authentication
   - Purpose of the Document
   - Audience

2. **Prerequisites**
   - Tools and SDKs Required
   - Basic Knowledge Assumptions (Intents, Android Manifest, Django)

3. **Facebook Developer Platform Setup**
   - Step-by-Step Guide for Creating a Facebook Developer Account
   - Registering Your Application on the Facebook Platform
   - Configuring the Facebook Login Product
   - App ID and App Secret Generation
   - Configuring OAuth Redirect URIs

4. **Understanding OAuth 2.0 in the Context of Facebook Login**
   - What is OAuth 2.0?
   - Facebook's Role as an Identity Provider
   - How Facebook Uses OAuth to Authenticate Users
   - Token Generation: Access Token, User Token, App Token
   - Deep Dive into Facebook’s Token Lifecycle

5. **Android Project Setup**
   - Adding Facebook SDK to Your Android Project
   - Configuring Android Manifest
     - Permissions Needed
     - Activity Declarations (`FacebookActivity`, `CustomTabActivity`)
   - Integrating the Facebook SDK with Android Studio

6. **Facebook Login Button Integration (Step by Step)**
   - Initializing Facebook SDK in Android
   - Adding Facebook Login Button to the Layout
   - Handling Facebook Login Callback
     - Registering `CallbackManager`
     - Retrieving Access Tokens
   - Sending Access Token to Backend

7. **Understanding Intents in the Context of External Services**
   - What Are Intents?
   - Interaction with External Entities like Facebook (Deep Linking)
   - How Intents Work Outside the App (Handling Browser and Custom Tabs)
   - Declaring Intent Filters in Android Manifest for Facebook Login
   - Chrome Custom Tabs and Why They Are Used

8. **Deep Linking in Android**
   - What Are URIs?
   - Role of URIs in Facebook Authentication
   - Using URIs in Deep Linking for Facebook Login
   - Configuring Custom URI Schemes
   - Handling URI Redirection Back to the App after Login
   - Practical Example of URI Implementation in Android

9. **Working with Access Tokens**
   - What Is an Access Token?
   - Facebook Access Token vs JWT
   - How to Retrieve Facebook User Information with Access Tokens
   - Securing Access Tokens: Best Practices
   - How Tokens Expire and How to Refresh Them

10. **Android Debugging and Testing**
    - Testing Facebook Login on Android Emulator
    - Debugging Tips for Token Generation Issues
    - Handling Login Errors and Fallbacks

11. **Django Backend Integration (Short Explanation)**
    - Verifying the Facebook Access Token
    - Creating or Logging in Users Based on the Token
    - Issuing JWT Tokens for Session Management
    - Securing API Endpoints

12. **Conclusion**
    - Recap of Facebook Login Flow
    - Key Takeaways for Android and Django Integration

13. **Appendices**
    - Facebook API References
    - Glossary of Terms (OAuth, URI, Intent, etc.)

### **Introduction**

#### 1. **Overview of Facebook Authentication**
Facebook Authentication allows users to log in to your application using their Facebook credentials. It leverages the **OAuth 2.0** protocol, which securely delegates user authentication to Facebook without requiring users to create new credentials for your app. When a user logs in through Facebook, they authenticate with Facebook’s servers, which issue an **access token**. This token can then be used to retrieve user profile information or perform additional actions.

Key benefits of Facebook Authentication:
- **Simplified User Experience**: Users can log in with their existing Facebook account, avoiding the need to create new passwords or usernames.
- **Secure**: OAuth 2.0 ensures secure handling of sensitive credentials, with Facebook handling the authentication process.
- **Widely Adopted**: Facebook is a trusted platform, and many users prefer logging in via Facebook for convenience.

#### 2. **Purpose of the Document**
This document provides a comprehensive, step-by-step guide to integrating **Facebook Login** into an Android application, along with a brief explanation of how the backend can verify and manage user sessions using Django. The document assumes minimal familiarity with advanced Android features such as deep linking and URI handling and aims to build understanding through clear, structured explanations.

The purpose is to:
- Guide developers through the configuration of Facebook Developer Platform.
- Offer in-depth explanations of Android integration with the Facebook SDK.
- Clarify how external services interact with intents and URIs.
- Provide insight into Facebook's OAuth 2.0 implementation and how tokens are managed.

#### 3. **Audience**
This document is written for **developers unfamiliar with the Facebook SDK** and those who may have limited experience with Android’s external service interaction mechanisms (e.g., intents, deep linking, URIs). Developers with a basic understanding of Android components, such as intents, will learn how to extend their knowledge to interact with services like Facebook. The document is designed to be beginner-friendly, focusing on building foundational knowledge before progressing into more complex concepts, such as token management and backend authentication processes.

### Key Documentation Guidelines:
- **Clear Structure**: The content is organized to guide the developer from basic to advanced concepts.
- **Step-by-Step Guidance**: Each task is broken down into actionable steps.
- **Avoid Assumptions**: The document assumes little prior knowledge of advanced topics and provides detailed explanations of key concepts.
  
### **Prerequisites**

#### 1. **Tools and SDKs Required**
To implement Facebook Login in your Android app, you’ll need the following:
- **Android Studio**: IDE for Android development.
- **Facebook SDK for Android**: The SDK enables Facebook login and provides access to user data.
- **Django** (Optional): For backend integration, use Django and libraries such as `requests` for HTTP calls.

#### 2. **Basic Knowledge Assumptions**
- **Intents**: You should understand how intents work internally in Android for launching activities within an app. This guide will explain how intents interact with external services like Facebook.
- **Android Manifest**: You need basic knowledge of configuring the `AndroidManifest.xml` file to declare activities, permissions, and intent filters.
- **Django**: Basic familiarity with creating endpoints and handling HTTP requests is assumed.


### **Facebook Developer Platform Setup**

#### 1. **Step-by-Step Guide for Creating a Facebook Developer Account**
1. **Visit Facebook for Developers**: Go to [Facebook for Developers](https://developers.facebook.com/).
2. **Create an Account**: If you don’t already have an account, sign up with your Facebook credentials.
3. **Create a New App**: After logging in, click “My Apps” and then “Create App”. Select the app type (e.g., **Consumer**) and follow the prompts.

#### 2. **Registering Your Application on the Facebook Platform**
Once your app is created, you’ll need to configure it:
1. **App Dashboard**: Navigate to the app dashboard.
2. **Add a Product**: Under the “Products” menu, click “Set up” next to **Facebook Login**.

#### 3. **Configuring the Facebook Login Product**
1. **Client OAuth Settings**: In your app’s settings, go to **Facebook Login > Settings**.
2. **Enable OAuth Login**: Enable “Client OAuth Login”, “Web OAuth Login”, and “Embedded Browser OAuth Login”.
3. **Set OAuth Redirect URIs**: Specify the URIs where Facebook will redirect users after they authenticate (e.g., `fb[YOUR_APP_ID]://authorize`).

#### 4. **App ID and App Secret Generation**
1. **App ID**: This is automatically generated when you create your app. It uniquely identifies your app in Facebook’s system.
2. **App Secret**: This is a private key that your backend uses to verify requests and communicate securely with Facebook. It must remain confidential.

#### 5. **Configuring OAuth Redirect URIs**
- **Purpose**: OAuth Redirect URIs handle redirection from Facebook back to your app after successful authentication.
- **How to Set Them**: In the **OAuth settings**, define a URI (e.g., `fb[YOUR_APP_ID]://authorize`) in your Android app and backend. These URIs act as entry points, allowing Facebook to redirect users back to your app or web endpoint post-authentication.

By the end of this setup, your app is registered with Facebook, and you’re ready to integrate Facebook Login into your Android project.

### **Understanding OAuth 2.0 in the Context of Facebook Login**

#### 1. **What is OAuth 2.0?**
OAuth 2.0 is an open standard for secure authorization, allowing third-party applications to access user data from another service without requiring the user to share their credentials. Instead, an authorization server (like Facebook) issues an **access token**, which the third-party app (your Android app) can use to interact with Facebook’s API.

OAuth 2.0 operates using four main roles:
- **Resource Owner**: The user who owns the data.
- **Client**: The app requesting access (your Android app).
- **Authorization Server**: Facebook’s server that issues tokens.
- **Resource Server**: Facebook’s API, which provides the data after successful authorization.

#### 2. **Facebook's Role as an Identity Provider**
In the context of Facebook Login, Facebook acts as both the **authorization server** and **identity provider**. This means Facebook is responsible for authenticating the user’s identity and generating tokens to prove the user’s authentication status. When your app integrates Facebook Login, it leverages Facebook’s OAuth 2.0 system to authenticate users, rather than building a custom username-password system.

#### 3. **How Facebook Uses OAuth to Authenticate Users**
Here’s a step-by-step breakdown of how Facebook uses OAuth 2.0 for authentication:
1. **User Interaction**: The user clicks the Facebook login button in your app.
2. **Authorization Request**: Your app sends an authorization request to Facebook, asking for access to certain permissions (e.g., email, public profile).
3. **User Consent**: Facebook asks the user to log in (if they aren’t already) and consent to the permissions requested by your app.
4. **Access Token Exchange**: After the user consents, Facebook generates an **access token** and sends it back to your app. This token is proof that the user has authenticated with Facebook and granted your app access.
5. **Token Usage**: Your app uses the access token to make API requests to Facebook’s resource servers, retrieving data like the user’s profile or email.

#### 4. **Token Generation: Access Token, User Token, App Token**
Facebook issues several types of tokens during the OAuth process, each with a specific role:

- **Access Token**: This is the main token that allows your app to make authenticated API calls to Facebook. It typically has a limited lifespan (e.g., an hour).
- **User Token**: A special type of access token that grants your app access to a user's Facebook profile and data.
- **App Token**: Used for accessing app-level data (e.g., app settings or metrics). This is usually not relevant for authenticating users, but important for backend operations.

#### 5. **Deep Dive into Facebook’s Token Lifecycle**
Facebook’s tokens follow a specific lifecycle, which helps balance security and user convenience:
1. **Access Token Expiry**: Access tokens have a limited lifespan (typically around 1-2 hours). This ensures that if a token is compromised, the damage is minimal.
2. **Refreshing Tokens**: After the access token expires, the app can either ask the user to re-authenticate or use a **refresh token** (if available) to obtain a new access token without user intervention.
3. **Revoking Tokens**: The user can revoke token access via their Facebook settings, which prevents your app from using expired or invalidated tokens.

By following this lifecycle, Facebook ensures that access to a user’s data remains secure and time-limited, but can be refreshed as needed for ongoing sessions.

---

### **Android Project Setup**

#### 1. **Adding Facebook SDK to Your Android Project**
To integrate Facebook Login, you need to add the Facebook SDK to your project. Follow these steps:
1. Open your `build.gradle` file and add the following dependency:
   ```gradle
   implementation 'com.facebook.android:facebook-login:[5,6)'
   ```
2. Sync your project with Gradle to download the SDK.

#### 2. **Configuring Android Manifest**
After adding the SDK, you need to configure the `AndroidManifest.xml` to ensure your app can handle Facebook Login properly.

##### 2.1. **Permissions Needed**
You need to declare the necessary permissions for internet access, as Facebook authentication requires an active internet connection:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

##### 2.2. **Activity Declarations**
There are two critical activities that need to be declared for the Facebook SDK:
- **FacebookActivity**: This activity handles Facebook’s login UI.
- **CustomTabActivity**: This activity is required for Chrome Custom Tabs, which Facebook uses for secure authentication.

Add the following to your `AndroidManifest.xml` inside the `<application>` tag:
```xml
<activity android:name="com.facebook.FacebookActivity"
    android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
    android:label="@string/app_name" />
<activity android:name="com.facebook.CustomTabActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="@string/fb_login_protocol_scheme" />
    </intent-filter>
</activity>
```
- **Why These Are Needed**: `FacebookActivity` handles the login dialog and `CustomTabActivity` is required for secure authentication with Custom Tabs, which prevents phishing attacks by controlling the login experience.

#### 3. **Integrating the Facebook SDK with Android Studio**
- Ensure that the `FacebookSdk.sdkInitialize(getApplicationContext())` is called in your `MainActivity`'s `onCreate` method to initialize the SDK:
  ```java
  FacebookSdk.sdkInitialize(getApplicationContext());
  ```

---

### **Facebook Login Button Integration (Step by Step)**

#### 1. **Initializing Facebook SDK in Android**
In your `MainActivity`, you need to initialize the Facebook SDK before using it:
```java
FacebookSdk.sdkInitialize(getApplicationContext());
```
This ensures that the SDK is ready for use throughout your app.

#### 2. **Adding Facebook Login Button to the Layout**
Add the **Facebook Login Button** to your XML layout file:
```xml
<com.facebook.login.widget.LoginButton
    android:id="@+id/login_button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```
- **Why**: This button provides a simple UI for triggering the Facebook login flow.

#### 3. **Handling Facebook Login Callback**
When a user clicks the login button, Facebook returns the login result via a callback. You need to capture this result:
```java
LoginButton loginButton = findViewById(R.id.login_button);
loginButton.registerCallback(callbackManager, new FacebookCallback<LoginResult>() {
    @Override
    public void onSuccess(LoginResult loginResult) {
        // Handle successful login
    }

    @Override
    public void onCancel() {
        // Handle login cancellation
    }

    @Override
    public void onError(FacebookException exception) {
        // Handle login errors
    }
});
```
- **Why**: Handling the callback allows you to respond to the login result—whether it’s success, failure, or cancellation.

#### 4. **Registering CallbackManager**
You need to create and register a `CallbackManager` to handle the result of the Facebook login process:
```java
CallbackManager callbackManager = CallbackManager.Factory.create();
```
This will route the result of the Facebook login attempt back to your app.

#### 5. **Retrieving Access Tokens**
After successful login, you can retrieve the access token:
```java
AccessToken accessToken = loginResult.getAccessToken();
```
- **Why**: The access token is a key to the Facebook API, allowing your app to request the user’s profile data.

#### 6. **Sending Access Token to Backend**
Once you have the access token, you need to send it to your backend for verification and further processing (e.g., generating a JWT):
```java
private void sendAccessTokenToBackend(String accessToken) {
    // Code to send the token to your Django backend
}
```
- **Why**: The backend must verify the token and authenticate the user session.

---

### **Understanding Intents in the Context of External Services**

#### 1. **What Are Intents?**
Intents in Android are messaging objects that allow apps to request actions from other components (activities, services, etc.). An intent can launch an activity, start a service, or communicate with other apps.

#### 2. **Interaction with External Entities like Facebook (Deep Linking)**
When integrating Facebook Login, intents facilitate interaction between your app and Facebook's login service through **deep linking**. A deep link allows Facebook to redirect users back to your app after they complete authentication on an external browser or app.

#### 3. **How Intents Work Outside the App (Handling Browser and Custom Tabs)**
When handling external services like Facebook, Android uses **implicit intents** to communicate with activities outside your app. For Facebook login, the intent opens a web browser (often in a **Chrome Custom Tab**) to securely display Facebook’s login page. Once the user completes login, the intent uses a **URI** to return the user to your app, carrying the authentication result.

#### 4. **Declaring Intent Filters in Android Manifest for Facebook Login**
An **intent filter** specifies the types of intents your app can respond to. In the case of Facebook Login, you need an intent filter to handle the URI returned after authentication. This is how your app knows when and how to handle the redirect from Facebook:
```xml
<activity android:name="com.facebook.CustomTabActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="@string/fb_login_protocol_scheme" />
    </intent-filter>
</activity>
```
This configuration lets your app recognize the custom URI scheme and respond appropriately.

#### 5. **Chrome Custom Tabs and Why They Are Used**
Chrome Custom Tabs allow your app to open web pages in a browser without losing control over the user experience. Facebook Login uses Custom Tabs to display the Facebook login page securely. They provide a seamless user experience by maintaining app-like interactions (e.g., color customization) while ensuring the authentication happens in a secure browser environment. This prevents apps from tampering with the authentication process, improving security.

---

### **Deep Linking in Android**

#### 1. **What Are URIs?**
A **Uniform Resource Identifier (URI)** is a string that identifies a resource, such as a webpage or an API endpoint. In the context of Android, URIs are used to deep link into an app from an external service like Facebook.

#### 2. **Role of URIs in Facebook Authentication**
In Facebook Login, URIs are essential for redirecting users back to your app after they authenticate on Facebook’s servers. Facebook uses a custom URI scheme (e.g., `fb[APP_ID]://authorize`) to return the user to your app, passing back the login result or token through the URI.

#### 3. **Using URIs in Deep Linking for Facebook Login**
Deep linking via URIs allows external services, like Facebook, to invoke specific activities within your app. In Facebook Login, this means that after successful authentication, Facebook can send the user back to a specific screen in your app (e.g., the home screen) using a pre-configured URI.

#### 4. **Configuring Custom URI Schemes**
In your `AndroidManifest.xml`, configure the custom URI scheme Facebook will use to redirect users to your app:
```xml
<activity android:name="com.example.MyActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="fb[YOUR_APP_ID]" />
    </intent-filter>
</activity>
```
- **Scheme**: The `android:scheme` specifies the URI scheme (e.g., `fb[YOUR_APP_ID]`). This acts as a trigger for the activity when the matching URI is invoked.

#### 5. **Handling URI Redirection Back to the App after Login**
When Facebook redirects back to your app after authentication, the **intent filter** and URI scheme ensure that the appropriate activity is launched. You can then retrieve the login result or access token from the intent’s data.

In your `MainActivity`, override the `onNewIntent` method to handle the data passed via the URI:
```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    Uri data = intent.getData();
    if (data != null && data.getScheme().equals("fb[YOUR_APP_ID]")) {
        // Handle Facebook login result here
    }
}
```
This code captures the URI, validates the scheme, and processes the login result.

#### 6. **Practical Example of URI Implementation in Android**
Here’s how you can set up deep linking for Facebook Login in practice:
1. **Declare the custom URI scheme** in `AndroidManifest.xml`.
2. **Override `onNewIntent`** in your main activity to handle redirection.
3. **Extract the login result** from the intent and process the data (e.g., retrieve the access token).

This approach ensures seamless interaction between Facebook and your app, allowing users to authenticate via Facebook and return directly to your app afterward.

---

### **Working with Access Tokens**

#### 1. **What Is an Access Token?**
An **Access Token** is a short-lived token that grants your app permission to access a user’s Facebook data on their behalf. It is issued by Facebook after successful authentication and used to make API requests to access profile data, email, or other user information. It serves as proof that the user has authorized your app to interact with their Facebook account.

#### 2. **Facebook Access Token vs JWT**
- **Facebook Access Token**: This token is specific to Facebook’s OAuth 2.0 system and is used to interact with Facebook’s APIs. It is usually short-lived (e.g., 1-2 hours).
- **JWT (JSON Web Token)**: JWT is a general-purpose token format that your backend can issue after validating the Facebook access token. JWT tokens are often used for managing sessions within your app and can be long-lived or short-lived, depending on your use case.

#### 3. **How to Retrieve Facebook User Information with Access Tokens**
Once your app receives an access token, you can use it to request information from the Facebook Graph API:
```java
GraphRequest request = GraphRequest.newMeRequest(
    accessToken,
    new GraphRequest.GraphJSONObjectCallback() {
        @Override
        public void onCompleted(JSONObject object, GraphResponse response) {
            // Handle the user information (e.g., name, email)
        }
    });
Bundle parameters = new Bundle();
parameters.putString("fields", "id,name,email");
request.setParameters(parameters);
request.executeAsync();
```
In this example, the Graph API retrieves the user’s ID, name, and email.

#### 4. **Securing Access Tokens: Best Practices**
- **Use HTTPS**: Always transmit access tokens over secure HTTPS connections to prevent interception.
- **Token Expiration**: Rely on short-lived tokens to limit exposure. Once an access token expires, users need to refresh or reauthenticate.
- **Store Tokens Securely**: Use secure storage mechanisms in your app (e.g., `SharedPreferences` with encryption) to store access tokens.

#### 5. **How Tokens Expire and How to Refresh Them**
Facebook access tokens have an expiration time. You can check the expiration time using the `AccessToken.getExpires()` method. If a token expires, the user needs to refresh the token, either by:
1. **Requesting a new token** using Facebook’s token refresh mechanism.
2. **Re-authenticating the user**, where the login flow is repeated to obtain a fresh token.

---

### **Android Debugging and Testing**

#### 1. **Testing Facebook Login on Android Emulator**
To test Facebook login on an Android emulator:
1. Install the Facebook app on the emulator, as Facebook login often relies on it.
2. Ensure the emulator has Google Play Services installed for proper OAuth handling.
3. Use a real Facebook account for testing, not a developer account.

#### 2. **Debugging Tips for Token Generation Issues**
- **Check App Setup**: Ensure that the **App ID** and **App Secret** are correctly configured in both Facebook’s developer platform and your Android app.
- **Inspect Network Calls**: Use Android Studio’s **Network Profiler** to verify that requests to Facebook’s Graph API are being made correctly and that access tokens are being exchanged without errors.
- **Log Callbacks**: Ensure proper callback handling by logging responses from the Facebook login process. This helps identify if the token retrieval is successful or if there’s a problem with the user’s permissions.

#### 3. **Handling Login Errors and Fallbacks**
If Facebook login fails, handle the errors gracefully:
- **Connection Issues**: Display a user-friendly message when internet connectivity is a problem.
- **Invalid Permissions**: Ensure that your app requests the correct Facebook permissions (like `email` and `public_profile`) and handle scenarios where users decline permissions.
- **Error Handling**: Use the `onError()` callback from the `FacebookCallback<LoginResult>` to manage login failures and guide the user through retrying or troubleshooting.

### **Django Backend Integration (Short Explanation)**

#### 1. **Verifying the Facebook Access Token**
When your Android app sends the Facebook access token to the Django backend, the backend must verify the token to ensure it's legitimate. This is done by calling Facebook's token verification API:
```python
response = requests.get(f'https://graph.facebook.com/debug_token?input_token={token}&access_token={app_access_token}')
```
This verifies that the token was issued by Facebook for your app.

#### 2. **Creating or Logging in Users Based on the Token**
Once the token is verified, the backend can fetch the user's profile data:
```python
user_info = requests.get(f"https://graph.facebook.com/me?access_token={token}&fields=id,name,email").json()
```
If the user exists in your database, log them in. If they are new, create a user account using the details from Facebook.

#### 3. **Issuing JWT Tokens for Session Management**
Once the user is authenticated, generate **JWT tokens** for managing their session in your app:
```python
access_token = jwt.encode(payload, secret_key, algorithm='HS256')
refresh_token = jwt.encode(refresh_payload, secret_key, algorithm='HS256')
```
JWT tokens allow your app to maintain user sessions without having to reauthenticate with Facebook for each API request.

#### 4. **Securing API Endpoints**
Secure your backend API endpoints by requiring valid JWT tokens for access. Before processing requests, ensure that the token is valid and belongs to the authenticated user:
```python
decoded_token = jwt.decode(access_token, secret_key, algorithms=['HS256'])
```
This guarantees that only authenticated users can access your secured routes.

---

### **Conclusion**

#### 1. **Recap of Facebook Login Flow**
The Facebook login flow begins with the Android app, where the user authenticates through Facebook. Upon successful authentication, Facebook issues an access token. This token is sent to the Django backend for verification, after which the backend either logs the user in or creates a new user account. Finally, the backend issues JWT tokens for session management.

#### 2. **Key Takeaways for Android and Django Integration**
- **Android**: Manages the Facebook login UI and retrieves the access token.
- **Django Backend**: Verifies the token, authenticates the user, and issues JWT tokens for future API interactions. Securing your endpoints with JWT tokens ensures that only authenticated users can interact with your application.

### **Appendices**

#### 1. **Facebook API References**
Below are the key API endpoints and documentation links that are helpful when working with Facebook Login and its integration with Android and Django:
- **Facebook Graph API Documentation**: The primary API for interacting with Facebook user data, such as retrieving profile information or user permissions. It includes endpoints for querying user data and verifying access tokens.
  - [Facebook Graph API Documentation](https://developers.facebook.com/docs/graph-api)
- **Facebook Login for Android**: This reference outlines how to integrate Facebook Login into Android applications, including SDK setup, token handling, and permissions.
  - [Facebook Login for Android](https://developers.facebook.com/docs/facebook-login/android)
- **Facebook Debug Token API**: Used to verify access tokens and retrieve information such as token expiration and user IDs.
  - [Facebook Debug Tool](https://developers.facebook.com/tools/debug/accesstoken/)

#### 2. **Glossary of Terms (OAuth, URI, Intent, etc.)**

- **OAuth 2.0**: An open standard for authorization that allows third-party apps to access user data without requiring them to share their credentials. Facebook Login uses OAuth 2.0 to issue access tokens.
  
- **Access Token**: A token provided by Facebook after a user authenticates, allowing the app to request data on behalf of the user. This token is time-limited and should be verified by the backend.

- **JWT (JSON Web Token)**: A compact, URL-safe token format used for securely transmitting information between parties. It is commonly used for session management in apps after verifying a user's identity.

- **URI (Uniform Resource Identifier)**: A string that identifies a resource on the internet. In the context of Android, URIs are used in deep linking to redirect users to specific app activities after authentication (e.g., Facebook Login redirection).

- **Intent**: An Android messaging object that is used to request an action from another component, such as launching an activity. For Facebook Login, intents help facilitate the flow between the app and external services like Facebook for authentication.

- **Intent Filter**: A declaration in `AndroidManifest.xml` that allows your app to respond to specific intents, such as opening a custom URI scheme for Facebook login redirection.

- **Chrome Custom Tab**: A feature in Android that allows web content to be securely displayed in a Chrome-based browser experience within your app. It is often used for OAuth-based logins like Facebook Login to improve security by controlling the browser environment.
