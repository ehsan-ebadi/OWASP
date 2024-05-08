# Basic
- User is logged-in at site _A_, he also wants to be logged-in at site _B_.
  - There should be a **Token** to transfer authenticaition between two different origin. (one-time-use, unique, not predictable)
  - Site _B_ should verify the token.
  - There should be a `redirect_uri` parameter to redirect the user from site _A_ to site _B_.
- SSO Implementations are _Redirect_, _CORS_, _JSONP_, _oAuth_, _SAML_ 

# Detection
- At first, determined the exact flow.
- Find a parameter (like `redirect_uri`) that redirect user to the _sso.target.tld_.
  ```bash
  https://sso.target.tld/auth/issue_token?redirect_uri=https://cvefix.ir/log
  ```
- `redirect_uri` is limited to _*.target.tld_  →  Search for an Open Redirect.
  ```bash
  https://sso.target.tld/auth/issue_token?redirect_uri=https://subdomain.target.tld/logout?r=https://cvefix.ir/log
  ```
- SSO works by XHR request  →  Search for CORS Misocnfiguration.
- SSO works by XHR request + CORS is configured safely _(*.target.tld)_  →  Search for an XSS.
