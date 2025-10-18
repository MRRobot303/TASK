# Cyber Security Task 2

# . Objective

To demonstrate an end-to-end web application security testing workflow using Burp Suite by:

- Intercepting a login request,
- Manually probing via Repeater,
- Decoding/mentally parsing encoded data,
- Automating credential fuzzing with Intruder,
- Comparing responses with Comparer to confirm findings.

Outcome: identify whether an authentication bypass (or other anomaly) could be discovered using simple payloads/wordlists, and document the methodology and results.

---

# 2. Test Environment & Setup

- Burp Suite: Community Edition (listener: `127.0.0.1:8080`).
- Browser: Firefox with proxy set to `127.0.0.1:8080`.
- Burp CA certificate installed and trusted in the browser.
- Target: Authorized vulnerable practice site (lab URL used during testing: *REDACTED — replace with exact lab URL*).
- Wordlist used for Intruder (small test list):
    
    ```
    password
    123456
    admin
    guest
    welcome
    qwerty
    letmein
    secret
    test
    
    ```
    
- All actions performed during the test were logged using Burp’s HTTP history and saved for reporting.

---

# 3. Methodology (Step-by-step)

## 3.1 Proxy & Certificate

- Configured browser proxy to `127.0.0.1:8080`.
- Imported Burp CA to avoid SSL/TLS errors.
- Verified capture by browsing the target site with Proxy → Intercept ON.

![Screenshot_2025-10-18_05_32_22.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/578f70e5-cf4e-4247-9c9e-9d53091e4ffb.png)

![Screenshot_2025-10-18_05_33_14.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/600fe78d-c0af-45b4-9601-a56018b60d55.png)

## 3.2 Intercepting a Login Request

- Performed a login attempt on the target.
- Burp captured the request (POST `/login` with `username` and `password` form fields).
- Inspected headers and body for CSRF tokens and session cookies.

**Captured request (example):**

![Screenshot_2025-10-18_06_14_45.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/5afb652e-b37d-400a-a8dd-a13750c4944d.png)

## 3.3 Manual Probing with Repeater

- Sent the captured request to Repeater.
    
    ![Screenshot_2025-10-18_06_22_31.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/5f1487f4-3f29-41e7-8b6c-652ffac36d60.png)
    
- Performed basic tests:
    - Replace `password` with `'` (single quote) to test for SQL errors.
    - Try `password=' OR '1'='1` for a classic SQL auth bypass test (lab-only).
- Observed response bodies, HTTP status codes, and any redirect behavior.

**Observed behavior:**

- Typical failed login responses returned the same page with error message.
- One probe returned a different response (different length and content), indicating potential change in server response.

## 3.4 Decoder Use

- When encoded values (Base64/URL-encoded) were found in tokens or parameters, sent them to Decoder.
- Decoded token contents to inspect structure (e.g., `dXNlcjpwYXNz` → `user:pass`), modified, and re-encoded for testing where applicable.

## 3.5 Intruder Setup & Attack

- Sent the login request to Intruder.
- In Positions tab:
    - Cleared default selections.
    - Marked only the `password` value as payload position.
    - Attack type: **Sniper**.
- In Payloads tab:
    - Loaded the small password wordlist.
- Launched attack.

![Screenshot_2025-10-18_06_35_47.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/47c806ab-9e37-4b20-adb7-ef0bea8d95f7.png)

## wordlist

![Screenshot_2025-10-18_06_40_46.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/987928a6-0ac5-4f6e-bcc4-5f907566385c.png)

## 3.6 Results Analysis

- Sorted Intruder results by **Length**.
- Identified one response with a significantly different length compared to failed attempts.
- Retrieved that response and a known-failed response and sent both to Comparer.
- Performed “Words” comparison to highlight differences.

**Key indicators of successful login / anomaly:**

- Different response body content (e.g., presence of “Welcome”, user-specific content).
- Presence of `Set-Cookie` header with a new session ID in the suspected-success response.
- Redirect to a dashboard URL (HTTP 302 or different Location header).

![Screenshot_2025-10-18_06_42_02.png](Cyber%20Security%20Task%202%20290496433d3b80798d76c1fdf83d879e/4ff77c67-95c6-4627-8d09-36dc283c054c.png)

---

# 4. Findings

> Summary: Using the simple wordlist and Intruder Sniper attack on the password parameter, an anomalous response was identified that suggests a successful login (or at least a distinct server response) for one payload.
> 

**Severity (in lab context):** For a production site this would be a critical authentication issue (possible weak credential or logic vulnerability); for a lab it demonstrates how differences in response can reveal authentication success.

---

---

---

# 10. Short conclusion paragraph

> Using Burp Suite Community Edition, I performed an end-to-end web application assessment on an authorized vulnerable practice site. I intercepted a login request, manually probed inputs in Repeater (including SQLi-style payloads), decoded encoded tokens where present, automated a small password fuzzing attack using Intruder, and used Comparer to confirm a suspected successful response. Sorting Intruder results by response length allowed quick identification of an anomalous response that contained dashboard content and a session cookie, demonstrating how observable response differences can reveal authentication weaknesses. Recommendations include normalizing responses, enforcing strong passwords and account lockout, and improving session handling.
>