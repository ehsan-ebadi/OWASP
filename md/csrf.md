# Detection
- Authentication system should work with Cookies, and `SameSite: None`.
  - `SameSite=Lax/Strict`  →  Need an XSS in any subdomain.
  - `SameSite=Lax`  →  Exploit CSRF with a GET request and Top-Level-Navigation. (We can't use JavaScript)
- Find a state-changing action that alters data like *Change password*, *Send email*, *Delete email*.
  - The request should be simple and repeatable.
- Look for a lack of CSRF protections
  - Use Burp Interceptor search bar to look for `csrf` or `state`.
  - CSRF tokens are in *request headers*, *cookies*, and *URL parameters*.
  - If you find a CSRF protection present on the endpoint, try protection-bypass techniques.
- Confirm the vulnerability
  - Right click on the request in BurpSuite and select **Engagement tools** > **Generate CSRF PoC**.
  - Click on **Options** button, then select **Include auto-submit script**, then click on **Regenerate**.

# Bypass
- Modify `csrf_token`
  - remove `csrf_token` parameter.
  - blank `csrf_token` parameter.
  - Use another session’s `csrf_token`. (CSRF tokens are single-use, so you'll need to include a fresh one)
- Verb tamper
  - `POST`  →  `GET` or `PUT`, then use preveious bypasses.
  - If verb tamper works, then use other bypasses.
  - `GET`  →  `POST`, If it's not allowed, try override the method by adding the `_method` parameter to the query string.
      ```html
      GET /my-account/change-email?email=foo%40web-security-academy.net&_method=POST HTTP/1.1
      ```
- Change the `Content-type` from `application/json` to:
  ```html
  application/x-www-form-urlencoded
  text/plain; application/json                               # To bypass Middleware/Reverse-proxy, changing content-type in exploit code should not make the request preflight
  application/x-www-form-urlencoded; application/json        # To bypass Middleware/Reverse-proxy
  ```  
- Bypass Referer Header Check
  - Remove the referer header by adding a `<meta>` tag to the page hosting your request form:
    ```html
    <html>
      <meta name="referrer" content="no-referrer">
      <form method="POST" action="https://email.example.com/password_change" id="csrf-form">
        <input type="text" name="new_password" value="abc123">
        <input type='submit' value="Submit">
      </form>
      <script>document.getElementById("csrf-form").submit();</script>
    </html>
    ```
  - Place the victim domain name in the referer URL as a subdomain/pathname.
    ```html
    Referer: example.com.attacker.com
    Referer: attacker.com/example.com
    ```
- Using XSS
  - XSS allows attacker to steal the legitimate CSRF token,
  - then craft forged requests by using XMLHttpRequest,
  - then take over admin accounts. 
- If CSRF token is in headers, put it in the body. If works, try other bypasses.
  - Vice-versa will not work, because make the request preflight. 

# Escalate
- Exploit Click-jacking
  - Check click-jacking vulnerability by placing the page in an iframe by specifying its URL as the `src` attribute of an `<iframe>` tag.
    ```html
    <html>
      <head>
        <title>Clickjack test page</title>
      </head>
      <body>
        <p>This page is vulnerable to clickjacking if the iframe is not blank!</p>
        <iframe src="PAGE_URL" width="500" height="500"></iframe>
      </body>
    </html>
    ```
  - Then use clickjacking to trick users into executing the state-changing action 
- Leak User Information by changing email address of victim in the vulnerable website, to receive his billing reports. 
- Accounts take over by finding CSRF vulnerability in critical functionality, like *creates password*, *changes password*, *changes email*, *resets password*.

# Tip
- Search for same functionality of web-app in Mobile endpoints.
  - Mobile endpoints work with token (except they are web-view), If *Authentication Cookie* works in mobile endpoint, it leads to CSRF.
  - If the reuqest's data is `json`, change `Content-Type` to `application/x-www-form-urlencoded` to make the request simple.
- Hidden path/parameters mostly are not protected against CSRF.  
  - `.../user/update` is protected against CSRF, but `.../user/update/info` is not.
- JSON request with cookies (NOT token) are more likely vulnerable to CSRF.
  - The endpoint might accept other content types like `GET`.

# Resource
- https://book.hacktricks.xyz/pentesting-web/csrf-cross-site-request-forgery
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSRF%20Injection
