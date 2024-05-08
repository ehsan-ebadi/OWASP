# Vulnerabilities
### Mass Assignment
- In modern web-app, check endpoints that make **INSERT** or **UPDATE**.

### Broken Object Level Authorization
- Change the id
  ```html
  /shops/{shopID}
  (Resquest Body)
  {"id": 2}
  ```
- Bypass
  ```bash
  {"id": [2]}                 # String to Array
  {"id": {"id": 2}}           # Object in Object
  {"id":1, "id": 2}           # Parameter Pollution (Replace "1" with a legitimate id)
  {"id":2, "id": 1}
  {"id": "*"}
  ```

### Broken Function Level Authorization
- Change the endpoint
  - _/api/v1/users/`me`_  →  _/api/v1/users/`all`_
- Fuzz (_/api/v3/login_)
  - _/api/`FUZZ`/login_  →  `/api/mobile/login`
  - _/api/v3/`FUZZ`_  →  `/api/v3/login`
  - _/api/`FUZZ`_  →  `/api/hidden_route`
- Verb Tamper
  - `GET /api/v1/users/433`  →  POST, PUT, DELETE

### Broken User Authentication
- Find an API to bypass authentication. (Narrow Recon)
  ```bash
  /api/system/verification-codes/{smsToken}
  ```

### Security Misconfiguration
- Change the `Content-type`
  - `application/json`  →  `application/xml`
    - JSON requests are not simple, so we need to change it to be able for some attacks like CSRF. 
  - `application/x-www-form-urlencoded`  →  `application/json`
- Cookie instead of token
  - There is two Change_Password functionality in Mobile and Web endpoint of a company.
    - Mobile: Work with **Authentication Token** in `target.tld/panel/changePassword`
    - Web: Work with **CSRF Token** + **Authentication Cookie** in `target.tld/api/v2/changePassword`
  - Now, there is a CSRF vulnerability if Mobile endpoint accept Authentication Cookie.
