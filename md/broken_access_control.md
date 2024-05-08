# Hunt
### Find unprotected endpoints
- Check `/robots.txt` file
- Check `js` files

### Privilege escalation
- Change parameters of both POST and GET request (like `id`)
- Search every endpoint to find all parameters (like `roleid: 2`) and use it in other requests.
- Seacrh all endpoints (like blog posts) to find GUID of other users.


# Bypass
### URL-based access control
- Use `X-Original-Url` header to bypass internal network access limitation. Path should be defined in header and not in `GET / HTTP/2`.
```bash
GET / HTTP/2
...
X-Original-Url: /admin
```

### Method-based access control
- Change method from `POST` to `GET`
