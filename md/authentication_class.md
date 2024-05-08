# Basic
- **Cookie** vs **Session**
  - Cookie’s data is saved in user’s browser, but Session’s data is saved in the server. _(Session’s id is saved in user’s browser)_
  - Mostly Cookies are handled by web applications, Sessions are handled by web servers.
  - Sessions are destroyed (not in server side) by closing browsers, but Cookies not.
  - Users can alter Cookie’s data, but Only Session’s id, because the data is saved server-side.
- In Authentication by **Cookie**,
  - Information can be saved in _only Cookie_, _only Session_, _Both Cookie and Session_.
  - Need to make extra effort to mitigate CSRF attacks
  - Authentication state is saved in the Session. (checked in every request)
  - Re-Authentication data is saved in the Cookie. (checked only if the Session is not present)
- In Authentication by **Token**,
  - Information can be saved in _LocalStorage_ or _SessionStorage_. (No information is saved in server-side)
  - CORS and CSRF are not security issue.
 
# Detection
### Improper Token Generation
- Check sufficient entropy/randomness.
- Try to guess or brute force authentication token.

### Insecure Email Verification
- In account creation or recovery process, email verification is often used to confirm a user's email address.

### Weak Password Reset Mechanisms (Forgot Password)
- Try to predict or guess reset link.
- Check if the reset password link doesn't expire after a reasonable time or can be reused. (low severity)
- In a scenario, the reset-password request is `POST /forgot-password?temp-forgot-password-token`, by removing `temp-forgot-password-token` from both URL and request body, we can change the username, and reset password of another user.

### Insecure Magic Links
- Also known as one-time login links or passwordless authentication links.
- Try to tamper the link's parameter for unauthorized access or other security breaches.

### Two-Factor Authentication (2FA) Flaws
- Check for flaw in the recovery or reset process for second-factor devices, such as SMS-based 2FA.
- An authentication flow is `/login`  →  `/login2` (2FA)  →  `/my-account?id=wiener`, we can manually change the URI when we are in `/login2` to `/my-account?id=carlos` to bypass 2FA.

### One-Time Password (OTP) Flaws
- Check for weak key generation, lack of proper entropy, or predictable OTP generation.
- Bypass rat-limit, by send the values as an array:
  ```bash
  POST /api/v1/otp/verify HTTP/1.1
  {"code": 334}  →  {"code": [0001,0002,0003,....,9999]}
  ```

### Weak Passwords
- Try to brute force or dictionary attack.
- Pay attention to different error that the web-app shows for incorrect username and incorrect password.

# Methodology
### Login
- Fuzz to discover hidden parameter to find Open Redirect or XSS.
- Response manipulation on legacy web-app, that may leads to a new page to more test (if there is JavaScript to handle authorization)
- Login with an email (`myemail@gmail.com`) address, then try to register by a ssame email with small change.
  ```bash
  my%00email@gmail.com
  my.email@gmail.com
  my.email+123@gmail.com
  ```
- Change the Email address, then use the old code to verify.
- Send HTTP request to the Login/Registration endpoint while authenticated and fuzz for hidden parameter.

### Registration
- Fuzz to discover hidden parameter to find Open Redirect or XSS.
- Register with special accounts:
  ```bash
  noreplay@github.com
  support@company.com
  ```
### Forgot password
- Fuzz to discover hidden parameter and try to manipulate.
- Change the `Host` header in the forgot password HTTP request or use double host.
  ```html
  Host: cvefix.ir
  Host: cvefix.ir/target.tld
  X-Forwarded-Host: cvefix.ir
  X-Host: cvefix.ir

  Host: target.tld
  Host: cvefix.ir
  ```
- Add `referrer` and `origini`, too.
- Collect the token for further analysis.
- Try Parameter Pollution, works here
  ```bash
  email=victim@gmail.com&email=attacker@gmail.com                  # &
  email=victim@gmail.com%20email=attacker@gmail.com                # %20
  email=victim@gmail.com|email=attacker@gmail.com                  # |
  email="victim@gmail.com%0a%0dcc:email=attacker@gmail.com         # %0a%0dcc:
  email="victim@gmail.com%0a%0dbcc:email=attacker@gmail.com        # %0a%0dbcc:
  email="victim@gmail.com",email="attacker@gmail.com"              # ,
  {"email":["victim@gmail.com","attacker@gmail.com"]}              # JSON
  ```
- Check if password reset link is not expire. (Low severity)

### Two-Factor authentication
- Response manipulation, e.g. change param value from `false` to `true`.
- Login with the credentials, send request to all features without performing 2FA.
  - Make request to an API which returns user's information
  - Make request to update user's information. (`PATCH`)

# Tip
### Login Bypass
- Check `*` for both username and password fields.

# Resource
- https://book.hacktricks.xyz/pentesting-web/login-bypass
- https://book.hacktricks.xyz/pentesting-web/captcha-bypass
- https://book.hacktricks.xyz/pentesting-web/account-takeover
- https://book.hacktricks.xyz/pentesting-web/2fa-bypass
