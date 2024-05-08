# Domain Discovery
### 1. Acquisition

- Which companies are bought by Walmart
- Useful site  =>  [crunchbase](https://www.crunchbase.com/)
- Google Dorks

```bash
"Acquired by Walmart"
"company. All Rights Reserved."
"© 2021 company. All Rights Reserved."
"company. All Rights Reserved" "-inurl: company"
site:corporate.walmart.com "Acquire"
```

### 2. Online Website for Reverse whois on domain properties
- [viewdns](https://viewdns.info/reversewhois/)
- [yougetsignal](https://www.yougetsignal.com/)
- [domaineye](http://domaineye.com/reverse-whois)
- [reversewhois](https://www.reversewhois.io/)
- [whoxy](https://www.whoxy.com/)
- [website.informer](https://website.informer.com)
- [bgp.he](https://bgp.he.net/)


### 3. Read hunters writeups
- [Example 1](https://samcurry.net/how-i-couldve-taken-over-the-production-server-of-a-yahoo-acquisition-through-command-injection/)
- [Example 2](https://websecblog.com/vulns/google-threadit/)

### 4. Certificate seacrh

- Certificates store valuable information *(Should done on CIDR)*
  - Subject: CN=
  - Issuer: C=, O=, CN=  *(Not Important !)*
  - X509v3 extensions - X509v3 Subject Alternative Name: DNS

```bash
echo | openssl s_client -showcerts -servername gardeshpay.ir -connect gardeshpay.ir:443 2>/dev/null | openssl x509 -inform pem -noout -text
```

- Search in sites that have already stored internet website`s certificates.
  - http://shodan.io
  - http://censys.io
  - https://crt.sh

### 5. Contact the security team         


------------------------------------------------------------------------------------------------------------

# CIDR Discovery

### 1. Reverse lookup on IP properties
1. Extract IPs for domains and subdomains (recursively)

```bash
for domain in $(subfinder -d domain.tld -silent); do echo $domain | dnsx -a -resp -silent; done
```

2. Extract IPs that do not belong to the CDNs
   - https://github.com/projectdiscovery/cdncheck
3. Extract properties of IP address with ```whois```
4. Reverse lookup on the properties to find other CIDR (IP range)
   - https://apps.db.ripe.net/docs/How-to-Query-the-RIPE-Database/#2--query-basics
   - https://www.arin.net/resources/registry/whois/rws/cli/ 

```bash
whois -h whois.ripe.net -- '-i person [name]'  # Ripe - [admin-c, tech-c, mnt-by]
whois -h whois.arin.net -- 'n ! [name]'        # Arin - [NetHandle]
```

### 2. ASN discovery

- Extract CIDR

```bash
curl -s https://api.bgpview.io/ip/161.209.100.100 | jq -r ".data.prefixes[] | {prefix: .prefix, ASN: .asn.asn}"
whois -h whois.cymru.com [IP]
```

- In Small companies ASN block does not belong to the company

```bash
prips [CIDR] | xargs -P10 -I IP bash -c "echo IP: \$(whois IP | grep netname)"   # Ripe
prips [CIDR] | xargs -P10 -I IP bash -c "echo IP: \$(whois IP | grep NetName)"   # Arin
```

- Amass (Not Suggested)

```bash
amass intel -org "walmart"
```

### Examples 

- arvancloud.com

```bash
whois $(dig A +short arvancloud.com)
# mnt-by: ArvanCloud
# nic-hdl: FAFA-RIPE, PUYA-RIPE
whois -h whois.ripe.net -- '-i mnt-by ArvanCloud'
# nic-hdl: ARMO-RIPE
whois -h whois.ripe.net -- '-i mnt-by ArvanCloud' | grep route
# CIDR (IP range)
whois -h whois.ripe.net -- '-i person FAFA-RIPE' | grep route
whois -h whois.ripe.net -- '-i person PUYA-RIPE' | grep route
whois -h whois.ripe.net -- '-i person ARMO-RIPE' | grep route
# 94.101.184.0 is a weired route
whois 94.101.184.0
# mnt-by: AbrArvan
whois -h whois.ripe.net -- '-i mnt-by AbrArvan' | grep route
# CIDR
```

- gardeshpay.ir

```bash
for domain in $(subfinder -d gardeshpay.ir -silent); do whois $(echo $domain | dnsx -a -resp-only -silent) | grep netname; done
# Find all netname

dig +short A gardeshpay.ir
# 80.75.4.105
whois $(dig A +short gardeshpay.ir) | grep netname
# danesh-novin-arsham
curl -s https://api.bgpview.io/ip/80.75.4.105 | jq -r ".data.prefixes[] | {prefix: .prefix, ASN: .asn.asn}"
# 80.75.4.0/24
prips 80.75.4.0/24 | xargs -P10 -I IP bash -c "echo IP: \$(whois IP | grep netname)"
# Find rows with netname == to danesh-novin-arsham
```

- Sempra

```bash
whois 161.209.202.104    # This IP address comes from subfinder output in CIDR Discovery
# It`s network is ARIN
# Find OrgId: SCG-1
whois -h whois.arin.net -- 'o SCG-1'
# Find this URL address that contain SCG-1 information from ARIN that contains Handle value
curl -s https://rdap.arin.net/registry/entity/SCG-1 | jq -r ".networks[].handle"
# NET-161-209-0-0-1
whois -h whois.arin.net -- 'n ! NET-161-209-0-0-1'
# CIDR: 161.209.0.0/16 , This is very large for Nmap and must devide to 255 ta /24
whois -h whois.arin.net -- 'z Sempra*'  # We should validate if the IP range belong to Sempra or NOT
whois -h whois.arin.net -- 'o Sempra*'  # Find more CIDR like SEMPR-1
curl -s https://rdap.arin.net/registry/entity/SEMPR-1 | jq -r ".networks[].handle"
# NET-206-130-144-0-1
whois -h whois.arin.net -- 'n ! NET-206-130-144-0-1'
# CIDR: 206.130.144.0/22
nmap -sS -Pn 161.209.202.0/24 -p 21,22,23,25,53,80,110,161,389,443,445,587,636,995,1025,1701,1723,2000,2483,2484,2601,3001,3128,3306,3389,3690,5060,5900,8000,8080,10443 --scrip ssl-cert -oX 161.209.202.0-24.xml
# Extract all domains and subdomains from XML file
```


------------------------------------------------------------------------------------------------------------

# Subdomain Discovery

### NMAP Certificate Search on CIDR

- The xml output should be parsed

```bash
sudo nmap -sS -Pn 45.82.138.0/23 -p 21,22,23,25,53,80,110,161,389,443,445,587,636,995,1025,1701,1723,2000,2483,2484,2601,3001,3128,3306,3389,3690,5060,5900,8000,8080,10443 --scrip ssl-cert -oX nmap.xml
```

### Providers

- crt.sh *(free)*

```bash
curl -s https://crt.sh/?q=[keyword]   # search on database
curl -s https://crt.sh/?O=[Orgname]   # search on organization
curl -s "https://crt.sh/?q=cvefix.ir&output=json" | jq -r ".[].common_name" | sort -u
curl -s "https://crt.sh/?q=cvefix.ir&output=json" | jq -r ".[].name_value" | sort -u
```

- Censys *(free api with rate limit)*

```bash
curl -s https://censys.io/api/v1/search/certificates -u "KEY:SECRET" -h "content-type: application/json" --data '{"query":"parsed.subject.organ"}'   ??????????????
```

- Github
  - https://github.com/gwen001/github-subdomains
  - Clone all company`s repositories and search in commits
  - Search in all repositories

```bash
domain.tld NOT www.
```

### Abused IP DB

```bash
curl -s "https://www.abuseipdb.com/whois/74.6.143.25" -H "user-agent: Chrome" | grep -E '<li>\w.*</li>' | sed -E 's/<\/?li>//g' | sed -e 's/$/.$1/'
```

### DNS brute force *(GOLDEN)*
- Find NS records that are not exposed and no one know about them, with combination of static (just wordlist) and dynamic (permutation of wordlist and subdomains) DNS Brute-force

1. check if ```kossher.domain.tld``` has A record or not
2. if ```True```, it answer to all DNS records: and we should 
   - Use HTTP (It is very slow)
   - Check if the IP of kossher.domain.tld is different from IP of a valid subdomain
3. if ```False```, conduct a subdomain enumeration with ```Subfinder``` & ```Abused IP``` & ```cert.sh```
4. Merge subdomain from **3** and static wordlist (below lists or any individual list) as an input for ```ShuffleDNS```
   - https://wordlists-cdn.assetnote.io/data/manual/2m-subdomains.txt
   - https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt 
5. Conduct a Name Resolution by ```ShuffleDNS``` and save the results (maybe can use ```DNSx```)
6. Merge subdomains from **3** and resolverd from **5** as input for ```DNSGen``` and ```AltDNS``` and save the result list *(permutation: dev-1.domain.tld => dev-x.domain.tld)*
7. Use ths list from **6** to conduct Name Resolution by ```ShuffleDNS``` and save the result again
8. Merge the result from **5** and **7** to generate final sundomain list



------------------------------------------------------------------------------------------------------------

# Asset Discovery

### Service Discovery

- just with HTTPX *(also we can use ```Nmap``` to find specific port)*

```bash
echo cvefix.ir | httpx -silent
```

### Virtual Host Discovery  

- Change host value of http request to localhost or IP to show virtual host on same ip

- Use these inputs as ```wordlist```:
  - Static wordlist (dev)
  - Static wordlist + domain (dev.domain.tld)
  - Subdomains that aquired from ```subfinder -d cvefix.ir -silent```
  
```bash
ffuf -w [path_to_wordlist] -u https://domain.tld -H "host: FUZZ"
# Default page's size is 10918 and we should exclude it
ffuf -w [path_to_wordlist] -u https://domain.tld -H "host: FUZZ" -fs 10918
```
- Spray all subdomains from ```subfinder``` on CIDR (range ip)

