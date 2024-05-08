# Intro
- Map the application as most as possible:
  - Extract URLs and JS files
  - Extract Parameters
  - Extract file names
  - Extract endpoints
  - Discover hidden parameters, files and endpoints

# Google Dorks
- To search in third parties

```bash
site:http://ideone.com | site:http://codebeautify.org | site:http://codeshare.io | site:http://codepen.io | site:http://repl.it | site:http://justpaste.it | site:http://patebin.com | site:http://jsfiddle.net | site:http://trello.com "cvefix.ir"
```

- To search for indexes with ```inurl```, ```ext```, ```link```, ```site```, ```intitle```

```bash
site:domain.tld inurl:&
site:domain.tld ext: (php|aspx|asp|txt|jsp|html|xml|bak)
site:domain.tld inurl: (unsubscribe|register|feedback|signup|join|contact [profile user |comment|api|developer|affiliate|upload|mobile|upgrade|password) inurl:&
site:domain.tld ext:xml | exticonf | ext:cnf | extireg | ext:inf | ext:irdp | exticfg | ext:txt | extiora | ext:ini
site:domain.tld ext:sql | ext:dbf | ext:mdb | ext:log
site:domain.tld ext:bkf | ext:bkp | ext:bak | ext:old | ext:backup
site:domain.tld inurl:login | inurlisignin | intitle:Login | intitle: signin | inurl:auth
site:domain.tld ext:doc | ext:docx | ext:odt | ext:pdf | ext:irtf | ext:sxw | extipsw | ext:ppt | ext:pptx | ext:ipps | ext:csv
site:domain.tld ext:action | ext:struts | ext:do
site:domain.tld filetype:wsdl | filetype:WSDL | ext:svc | inurl:wsdl | Filetype: ?wsdl | inurl:asmx?wsdl | inurl:jws?wsdl | intitle:_vti_bin/sites.asmx?wsdl | inurl:_vti_bin/sites.asmx?wsdl
site:domain.tld filetype:config
site:x.domain.tld
site:x.*x.domain.tld
site:.s3.amazonaws.com "domain.tld"
site:stackoverflow.com "domain.tld"
link:domain.tld
site:.domain.tld intitle:"Welcome to Nginx"
```

# Search in Github, Bitbucket, ...

# Determine the Framework / CMS
- The Wappalizer is a good tool, working as CLI and browser extension.

```bash
wappalyzer https://cvefix.ir | jq
```

# Using Archives (WayBack Machine, OTX, etc) 
- To find old functionalities that are still exist but remove from the UI
- https://archive.org *(a great source to travel in time)*
- https://commoncrawl.org *(a great source of sites which have already been crawled)*

# Crawl web applications
- Manual browsing
- From JS files 

### Resource discovery
- Resources that can not be found in crawl *(like https://cvefix.ir/123.php that there is no link to it in the website)* 

### Hakrawler
- A good crawler written in GoLang

### GoSpider
- A great crawler written in GoLang, supports third parties such as Archive.org, CommonCrawl.org, VirusTotal.com, AlienVault.com
- Run GoSpider efficiently
 
```bash
ggospider(){
    host=$(echo $1 | unfurl format %d)
    gospider --site $1 --random-delay 2 --other-source --include-other-source --depth 4 --user-agent "Mozilla/5.0 (Macintosh; Intel Mac 0S X 10_15_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0.3 Safari/605.1.15" --quiet --robots —-sitemap —-json | grep -v '\[url\]' | jq -r ".output" | grep —Eiv '\.(css|jpg|jpeg|png|svg|ing|gif |mp4|flv|ogy|webm|webp|mov|mp3|mda|mdp|ppt|pptx|scss|tif|tiff|ttf|otf|woff|woff2|bmp|ico|eot|htc|swf|rtf|image)' | grep $host | sort -u
}

ggospider https://icolab.info
```

### GAU
- A tool (passive, but supports active mode) inspired by waybackurls which fetches known URLs from third parties, such as Archive.org, CommonCrawl.org, AlienVault.com

### LinkFinder
- A python script written to discover endpoints and their parameters in JavaScript files
- https://github.com/GerbenJavado/LinkFinder

### Robofinder
- A tool to fetch contents of all ```robots.txt``` from https://arvhive.org and list the URLs *(I should develop it)*
- The ```robots.txt``` file contain file names which might not be linked in the main web application.
- Flow
  - Mkae HTTP request to http://web.archive.org to get all indexes (https://site.tld/robots.txt)
  - Request to all indexes to fetch contents of saved ```robots.txt```
  - Extract all path from all indexes
  - Make a unique and nice output
  
### FLinks
- A tool to merge Robofinder and GoSpider results and show the nice output. it can be used for mass hunting.
- Features
  - Single or several target inputs
  - Bad extensions filtering (```jpg```, ```png```, ...)
  - Removing out of scope results
- Flow
  - ```Target``` > ```Flinks``` > ```GoSpider``` / ```RoboFinder``` / ```X``` > ```output```

### Pegex
- An alternative for LinkFinder which has many enhancements.
- Pegex is a tool to edltract relative and absolute URLs from a target. It's a research based tool which should be completed in a long priod time. You should find a way to extract all possible paths with minimum false positive. I give you a base regex, you can expand it:

```bash
(?:"|'|\\n|\\r|\n|\r)(((?:[a-zA-Z]{1,10}:\/\/|\/\/)[^"'\/]{1,}\.[a-zA-Z]{2,}[^"']{0,})|((?:\/|\.\.\/|\.\/)[^"'><,;| *()(%%$^\/\\\[\]][^"'><,,;|()]{1,})|([a-zA-Z0-9_\-\/]{1,}\/[a-zA-Z0-9_\-\/]{1,}\.(?:[a-zA-Z]{1,4}|action)(?:[\?|\/][^"|']{0,}|))|([a-zA-Z0-9_\-]{1,}\.(?:php|asp|aspx|cfm|pl|jsp|json|js|action|html|htm|bak|do|txt|xml|xls|xlsx)(?:\?[^"|^']{0,}|)))(?:"\'|\\n|\\r|\n|\r)
```
- Flow
  - DNS resolution of URLs
  - HTTP request to URLs
  - Saving the JavaScript files
  - Making the results beatufy
  - Extracting the URLs from files

### BurpSuite
- Manually browsing the website, generating a site map
- Using spidering feature in the BurpSuite (not recommended by me)

### Comparison among crawler tools
- To compare tools, create a lab with an index.html and many js files with different path structure inside it to test the tool.

| Name             | Passive | Active Crawling | JS parser             | DOM Parser | Old robots.txt |
|------------------|---------|-----------------|-----------------------|------------|----------------|
| GoSpider         | True    | True            | True                  | False      | False          |
| Hakrawler (bad)  | False   | True            | False                 | False      | False          |
| GAU              | True    | False           | False                 | False      | False          |
| Manual BurpSuite | False   | True            | True(with extensions) | True       | False          |
| FLinks           | True    | True            | True                  | False      | True           |


### Final flow
- The final flow to automate recon process is something like this
  - ```Target``` > ```Flinks``` > ```Pegex```
  - ```Flinks``` & ```Pegex``` & ```Parsing``` > ```URLS``` > ```HTTPX```
- The results is a bunch of URLs of the target containing
  - Absolute and relative paths
  - File names (acceptable extensions)
  - Parameter names
- The results can be leveraged to mass or manual hunting
- In the web application recon process, you might save some information to
  - Use for the current target
  - Use later on other targets
 


### Saving the information
- A database should be designed to save the information. The information can be extracted from the results of different tools. | recommend to use


# Fuzzing on various properties *(GOLDEN)*
- The key success to achieve vulnerabilty discovery Is parameter discovery. Let's see what zseanos says:


> I created "InputScanner" so I could easily scrape each endpoint for any input name/id listed on the page, test them & note down for future reference. I then used Burp Intruder again to quickly test for common > parameters found across each endpoint discovered & test them for multiple vulnerabilities such as XSS. This helped me identify lots of vulnerabilities across wide-scopes very quickly with minimal effort. I > > define the position on endpoint and then simply add discovered parameters onto the request, and from there I can use Grep to quickly check the results for any interesting behaviour


- Bullets
  - Hidden parameter discovery
  - Hidden endpoint discovery
  - Hidden file discovery
- Requirements
  - Wordiist
  - Tool

### Tools
- FFUF
- GoBuster
- KiteRunner
- IIS-Shortname-Scanner

### Fuzzing files and directories
- Before fuzz, make sure that you can find an existing file then fuzz. Let's call this method *Verify Method*. As instance, there is a file on this site:

```bash
GET /file.ext  =>  200
```

- Make a list which includes the ```file.ext``` with many lines to check if there is a limitation mechanism

```python
wlist_maker(){
  seq 1 400 > list.tmp
  echo $1 >> list.tmp
  seq 401 800 >> list.tmp
  echo $1 >> list.tmp
}
```

- Run the fuzzer
```bash
ffuf —w list.tmp -u "https://site.com/FUZZ" -ac
```

- You can use ```wlist_maker```

```python
wlist_maker(){
  seq 1 100 > list.tmp
  echo $1 >> list.tmp
  seq 101 200 >> list.tmp
  echo $1 >> list.tmp
}
```

- Important note in fuzzing
  - Make your own wordlist
  - Fuzz based on the context, for example
    - Skip file fuzz on web applications run by Express
    - Skip endpoint fuzzing on rest APIs, except the last part
      - We can not find ```api``` by ```https://site.com/FUZZ```
      - We should find ```api``` with ```KiteRunner``` and then ```https://site.com/api/FUZZ```
  - Change the ```User-Agent``` header to avoid blocking
  - Tune the threads to avoid banning
  - Watch out the CDN before fuzzing
    - Because CDN monitor request/seconds and block the requests 

### Wordlist
- You should generate your own wordlist, or use public ones:

##### Endpoints, Directories and files
- Build your own file name, directory and endpoint lists from various targets, you can use GMiner
- You should always be creative to create wordiist. | wil introduce some other methods
  - Use https://www.toptal.com/developers/gitignore to generate a sensitive wordlist
  - Use public Swaggers (inurl:/api/docs/ intitle:"swagger") to generate various wordiists (parameter and endpoint)
- Public wordlists
  - SecLists
  - https://wordists.assetnote.io/


##### Parameters 
- Build your own parameter list from various targets:
  - Collect filenames from the target
  - Collect keys of the JSON object from server's responses
  - Collect HTML form names, ids and etc
  - Collect JavaScript variables
  - Collect JSON object in JavaScript files
- Also You can
  - Merge your parameter list with the good public ones
  - Extract parameter list from other sources such as Github or any similar repository
- Examples

```html
<!DOCTYPE html
<html>
<head>
    <meta charset="utf-g">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>To Discover :)</title>
</head>
<script type="text/javascript">
    var test_var 'something';
    var test_obj = {"obj_key1": "val1", "obj_key2": "val2"};
    var empty_var = '' ;
</script>
<script src='path/js_file.js'></script>
<body>
<form method="POST">
    <input type="input" name="user_id_i" id='user_id'>
    <input type="input" name="rdt_to" id='redirect' value='/information'>
    <input type="submit" name="check">
</form>
<a href="path2/test.php?paraml=valuel">path2m param1</a>
<a href="path2/?param2=value2">path2, param2</a>
</body>
</html>
```
```bash
---------------------------------- OUTPUT ----------------------------------
test_var
test_obj
obj_keyl
obj_key2
empty_var
user_id_i
user_id
rdt_to
redirect
param
param2
```

```json
{
    "data": {
        "count": 220,
        "items": [
            {
                "dest_comment": null,
                "dest_user": {
                    "first_name": "\u0639\u0628\u062f\u0627\u0644\u0631\u062d\u0645\u0646",
                    "id": 1,
                    "last_name": "",
                    "mobile": "09148430585"
                },
"discount": null,
"discount_prsentage": null,
"discounted_price": null,
"dst_card_fullname": null,
"extra_field": null,
"hash_id": "265316873436",
"id": 1056732,
"credit": false,
"is_success": false,
"lost_points": null,
"message": "\u062f\u0631\u062e\u0648\u0627\u0633\u062a",
"owner": "09169189680",
"service": "\u062a\u0631\u0627\ud6a9\u0646\u0634\ud6a9\ud6cc",
"service_id": 5,
"source_user": null,
"src_comment": null,
"timestamp_date": "1634309813",
"tr_amount": 10000,
"tr_code": null,
"2021-10-15T18:26:53.261663+03:30",
"tr_origin": "",
"tr_status": "",
"tr_status_color": "",
"r_status_fa": "",
"r_status_id": "",
"r_type": "wallet",
"tr_type_fa": "" ,
"tracking_number": null,
```

```bash
---------------------------------- OUTPUT ----------------------------------
grep -o '\".*\"' response | sed -r 's/"|:.%//g' | sort -u
count
data
dest_comment
dest_user
discount
discount_prsentage
discounted_price
dst_card_fullnamd
extra_field
first_name
hash_id
id
is_credit
is_success
items
last_name
lost_points
message
mobile
next
owner
previous
service
service_id
source_user
src_comment
success
timestamp_date
tr_amount
tr_code
tr_date
tr_dest
tr_origin
tr_status
tr_status_color
tr_status_fa
tr_status_id
tr_type
tr_type_fa
tracking_number
```
- FallParams
  - It's a script to extract all parameters from given URLs
  - Modes
    - HTML parsing
      - ```<input>``` tag, ```id``` attribute
      - ```<input>``` tag, ```name``` attribute
      - ```<a>``` tag, ```href``` attribute
    - Javascript parsing
      - Variable names
      - JSON objects
  - Flow
    - ```Burp SiteMap``` & | ```Flinks``` & | ```Single Page``` > ```FallParams``` > ```HTTP Req``` > ```Extract```
    - ```FallParams``` > ```Extract```

##### GMiner
- A client-server tool to collect the various information from target. This information includes
  - Parameter lists
  - File names
  - Endpoints
  - RAW HTTP request
- Flow
  - ```Web``` > ```Manual Burp crawling``` > ```GMiner Burp Extension``` > ```Parsing``` > ```Database```

- The way of extracting the information:
  - Parameters
    - application/x-www-form-urlencoded
    - multipart/form-data
    - applicationfjson
    - application/xml
    - Extracting all keys from responses (HTML forms, Js variables and etc)
  - File names
    - Extracting all files from requests Ad
    - Extracting all files from responses
    - Exclude list based on the extension in requests
    - Exclude list based on the content-type in responses
  - Paths
    - Exclude list based on the content-type in the responses
    - Extracting all paths from responses
  - Full path
    - Extracting HTTP method + full path + parameter from requests
  - Counting the file names, paths and keys
  - Pushing RAW HTTP request

### Fuzzing endpoints
- In order to fuzz an endpoint, use
  - FFUF to fuzz the last part of the endpoint
  - KiteRunner to fuzz whole endpoint *(https://github.com/assetnote/kiterunner)*
- Using ```kite``` to generate wordlist:

```bash
apt install golang-go
git clone https://github.com/assetnote/kiterunner
cd kiterunner
make build
ln -s $(pwd)/dist/kr /usr/local/bin/kr
kr kb compile routes/small.json small.kite
```

- Using ```kite``` to fuzz endpoints

```bash
kr scan https://api.site.com -w routes-small.kite --kitebuilder-full-scan
```

### Fuzzing parameters
- To fuzz on parameters, you can add bunch of parameters to speed up the fuzzing:

```bash
GET /paraml=valuel&paran2=value2&param3=value3&...
```

- The following tools can be used to hidden parameter discovery
  - Arjun
  - https://github.com/xnl-h4ck3r/GAP-Burp-Extension
  - x8
    - more optional, more stable
    - Decrease the chunk to 20 ~ 30 to avoid drop requests

##### x8 Wordlists
- https://github.com/the-xentropy/samlists
- https://github.com/s0md3v/Arjun/tree/master/arjun/db
- https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content/BurpSuite-ParamMiner
- https://github.com/Impact-I/x8-Burp/tree/main/src
- https://github.com/s0md3v/Arjun/tree/master/arjun/db
- https://github.com/PortSwigger/param-miner/blob/master/resources/params
- https://wordlists.assetnote.io/
- https://gist.github.com/nullenc0de/9cb36260207924f8e1787279a05eb773

### Where to fuzz
- Fuzz the endpoints which return information
- Fuzz the files without any parameter
- Fuzz all endpoints and files to discover extra parameters
- Root or other directories (file and directory fuzz)

### Methodology
- Crawl the target and save all file names and paths - FLinks and Manual crawling *(Do no forget to use Verify Method)*
- Detect the wilb application technologies, then
  - Conduct fuzz on the full and relative endpoint to discovery hidden endpoint
  - Conduct fuzz on the file name and directories to discover hidden files and directories
  - Generate a fuzz list based on the target's site map and fuzz - backupKiller
- Crawl the target and save all parameters - FallParams
- Conduct parameter fuzzing to discover hidden parameters

### Examples
##### Examples 1: a rest API endpoint

```bash
/api/user/699201852
```

- Fuzz:

```bash
/api/user/699201852/FUZZ // words
/api/user/FUZZ/699201852 // words
/api/FUZZ/699201852  // words
/api/user?FUZZ // params NOT with ffuf, we should do it by x8
/api/user/699201852?FUZZ // params
```

##### Example 2: Single PHP page

```bash
/change_password.php
```

- Fuzz

```bash
/FUZZ.php  // file names, words
/change_password.phpFUZZ // "~", ".1", ".2", ".3", ".4", ".5", ".old", ".bk", ".bak"
/FUZZ.phpFUZZ
/change_password.php?FUZZ
```

##### Example 3: an Apache root directory (you can code something like backupKiller)

```bash
/FUZZ.FUZZ
// "7z", "back", "backup", "bak", "bck", "bz2", "copy", "gz", "old", "orig", "rar", "sav", "save", "tar", "tar.bz2", "r.gzip", "tgz", "tmp", "zip"
```

### BackupKiller
- It`s a tool to generate backup files on the URLs given. For example:

```bash
https://cvefix.ir/wp-admin/admin-ajax.php
https://cvefix.ir/wp-comments-post.php
https://cvefix.ir/wp-Login.php?action=lostpassword
https://cvefix.ir/wp-Login.php?redirect_to=https%3%2F%2Fcvefix.ir%2Fwp-admin%2F&reauth=1
https://cvefix.ir/xmlrpc.pho?rsd
```

- Output

```bash
https://cvefix.ir/wp-login.php~
https://menoryLeaks.ir/wp-login.php.1
https://cvefix.ir/wp-login.php.2
https://cvefix.ir/wp-login.php.3
https://cvefix.ir/wp-login.php.4
https://cvefix.ir/wp-login.php.5
https://cvefix.ir/wp-login.php.old
https://cvefix.ir/wp-login.php.bk
https://cvefix.ir/wp-login.php.bak
https://cvefix.ir/.wp-login.php.swp.
https://cvefix.ir/wp-admin/admin-ajax.php~
https://cvefix.ir/wp-admin/admin-ajax.php.1
https://cvefix.ir/wp-admin/admin-ajax.php.2
https://cvefix.ir/wp-admin/admin-ajax.php.3
https://cvefix.ir/wp-admin/admin-ajax.php.4
https://cvefix.ir/wp-admin/admin-ajax.php.5
https://cvefix.ir/wp-admin/admin-ajax.php.old
https://cvefix.ir/wp-admin/admin-ajax.php.bk
https://cvefix.ir/wp-admin/admin-ajax.php.bak
https://cvefix.ir/wp-admin/.admin-ajax.php.swp
https://cvefix.ir/xmlrpc.php~
https://cvefix.ir/xmlrpx.php.1
https://cvefix.ir/xmlrpx.php.2
https://cvefix.ir/xmlrpx.php.3
https://cvefix.ir/xmlrpx.php.4
https://cvefix.ir/xmlrpx.php.5
https://cvefix.ir/xmlrpx.php.old
https://cvefix.ir/xmlrpx.php.bk
https://cvefix.ir/xmlrpx.php.bak
https://cvefix.ir/.xmlrpx.php.swp
https://cvefix.ir/wp-comments-post.php~
https://cvefix.ir/wp-comments-post.php.1
https://cvefix.ir/wp-comments-post.php.2
https://cvefix.ir/wp-comments-post.php.3
https://cvefix.ir/wp-comments-post.php.4
https://cvefix.ir/wp-comments-post.php.5
https://cvefix.ir/wp-comments-post.php.old
https://cvefix.ir/wp-comments-post.php.bak
https://cvefix.ir/wp-comments-post.php.bak
https://cvefix.ir/.wp-comments-post.php.swp
```

- Usage

```bash
cat list.tmp | httpx -mc 200 -silent
```

- Flow
  - ```SiteMap``` & ```Flinks``` & ```X``` > ```backupKiller``` > ```URLs```
 

# Backup Fuzzer
- It's a nice flow I personally discovered when I was testing a target and found backup by accident.
- The flow is simple, I wrote a script to do
  - Gathering all targets from my repository
  - Generating wordlist based on the target domain
  - Sending HTTP request to check if the backup file is existing or not
- Generating based on the domain:

```bash
domain.tld -> generate ->

domain.tld.(rar|tar.gz|7z|gzip|back)
www.domain.tld.(rar|tar.gz|7z|gzip|back)
www.domain.(rar|tar.gz|7z|gzip|back)
```
