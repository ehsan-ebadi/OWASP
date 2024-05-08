# backup_fuzzer.py
```python
#!/usr/bin/pyhon

import time
from itertools import product
import argparse

'''
wp-config.php generates

wp-config.php.1
wp-config.php.bak
wp-config.php.20140909
wp-config.php_08082014
.wp-config.php
.wp-config.php.swp
wp-config.txt
'''

preffixes = set(['', '.', '_', '~'])
date_seps = set(['', '.', '-', '_'])
extensions = set(['', '.swp', '.tmp', '~', '.zip', '.tar.gz', '.tgz', '.tar.bz2', '.rar', '.7z', '.bak', '.0', '.1', '.2', '.old'])


if __name__ ==  '__main__':

    parser = argparse.ArgumentParser()
    parser.add_argument('-nd', '--no-dates', help='Disable YYYYMMDD and YYYYDDMM dates as file suffixes (default False)', default=False, action='store_true')
    parser.add_argument('-bd', '--basic-dates', help='Use basic dates variation only', default=False, action='store_true')
    parser.add_argument('-cs', '--case-sensitive', help='Enable case sensitiveness (default False)', default=False, action='store_true')
    parser.add_argument('-y', '--years', help='Past years to include in dates (default 1)', default=1, type=int)
    parser.add_argument('word') 
    args = parser.parse_args()

    original = args.word.strip()
    splitted = original.split('.')
    if len(splitted) > 1:
        original_name = '.'.join(splitted[:-1])
        original_ext = '.' + splitted[-1]
    else:
        original_name = splitted[0]
        original_ext = ''

    dates = set([''])
    if not args.no_dates:
        current = time.localtime()
        for d in product(range(current.tm_year-args.years, current.tm_year), [str(i).zfill(2) for i in range(1,13)], [str(i).zfill(2) for i in range(1,31)]):
            dates.add('%s%s%s' % d)
            if not args.basic_dates:
                dates.add('%s%s%s' % (d[0],d[2],d[1]))
                dates.add('%s%s' % (d[2],d[1]))
                dates.add('%s%s' % (d[1],d[2]))
                dates.add('%s%s%s' % tuple(reversed(d)))
                dates.add('%s%s%s' % tuple(reversed((d[0],d[2],d[1]))))
        for d in product([str(i).zfill(2) for i in range(1,current.tm_mon+1)], [str(i).zfill(2) for i in range(1,31)]):
            dates.add('%s%s%s' % (current.tm_year, d[0], d[1]))
            if not args.basic_dates:
                dates.add('%s%s%s' % (current.tm_year, d[1], d[0]))
                dates.add('%s%s' % (d[0], d[1]))
                dates.add('%s%s' % (d[1], d[0]))
                dates.add('%s%s%s' % tuple(reversed((current.tm_year, d[0], d[1]))))
                dates.add('%s%s%s' % tuple(reversed((current.tm_year, d[1], d[0]))))

    filenames = set([original_name])
    if args.case_sensitive:
        filenames.add(original_name.upper())
        filenames.add(original_name.lower())
        filenames.add(original_name[0].upper() + original_name[1:])

    duplicates = set()
    for result in product(preffixes, filenames, ('', original_ext), date_seps, dates, extensions):
        if result[4] == '':
            string = '%s%s%s%s%s' % tuple([result[i] for i in [0,1,2,4,5]])
        else:
            string = '%s%s%s%s%s%s' % result

        if string not in duplicates:
            print (string)
            duplicates.add(string)
```

# nmap_certificate_parser.py
```python

import sys
import xmltodict

xml_file = sys.argv[1]

with open(xml_file, 'r') as file:
    xml_data = file.read()

result = xmltodict.parse(xml_data)

host = result['nmaprun']['host']

if (type(host)) != list:
    address = host['address']
    hostnames = host['hostnames']
    ports = host['ports']['port']

    print ("[+] Address:" + "\n")
    print (str(address) + "\n\n")

    print ("[+] Hostname:" + "\n")
    print (str(hostnames) + "\n\n")

    for port in ports:
        if 'script' in port:
            output = port['script']['@output']
            tables = port['script']['table']
            for table in tables:
                if table['@key'] == 'subject':
                    subject = table['elem']
                    print ("[+] Subject:" + "\n")
                    print (str(subject) + "\n\n")

                elif table['@key'] == 'issuer':
                    issuer = table['elem']
                    print ("[+] Issuer:" + "\n")
                    print (str(issuer) + "\n\n")

                elif table['@key'] == 'extensions':
                    print ("[+] Extensions:" + "\n")
                    extensions = table['table']
                    for extension in extensions:
                        print (" - " + str(extension['elem']) + "\n")
```

# selenium.py
```python
# https://github.com/SeleniumHQ/seleniumhq.github.io/tree/trunk/examples/python/tests
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

options = webdriver.ChromeOptions()
options.add_argument('--headless')
driver = webdriver.Remote(
    command_executor='http://localhost:4444',
    options=options
)

# Navigate to the Twitter login page
driver.get("https://twitter.com/login")
print ("===============1===============")

# Enter the username and password and submit the form
username_field = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located(("name", "text"))
)
username_field.send_keys("targethunter001")
print ("=============2=================")
next_button = driver.find_element(By.XPATH, "//span[text()='Next']")
next_button.click()
print ("================3==============")
username_field = driver.find_element("name", "password")
username_field.send_keys("!QAZ2wsx#EDC")
next_button = driver.find_element(By.XPATH, "//span[text()='Log in']")
next_button.click()
print ("===============4===============")

# Wait for the page to load
time.sleep(5)

driver.save_screenshot('screenshot.png')


# driver.get("https://twitter.com/hacker0x01")
# tweets = driver.find_elements_by_xpath("//div[@data-testid='tweet']")
# for tweet in tweets:
#     print(tweet.text)
driver.quit()





# Scroll down to load all tweets
# for i in range(3):
#     driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
#     time.sleep(2)




















from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

options = webdriver.ChromeOptions()
options.add_argument('--headless')
driver = webdriver.Remote(
    command_executor='http://localhost:4444',
    options=options
)

driver.get("https://twitter.com/i/flow/login")

driver.implicitly_wait(3)

username_field = driver.find_element(by=By.NAME, value="text")
next_button = driver.find_element(by=By.XPATH, value="//span[text()='Next']")

username_field.send_keys("targethunter001")
next_button.click()

driver.implicitly_wait(3)

password_field = driver.find_element(by=By.NAME, value="password")
login_button = driver.find_element(by=By.XPATH, value="//span[text()='Log in']")

password_field.send_keys("!QAZ2wsx#EDC")
login_button.click()

time.sleep(3)

driver.get("https://twitter.com/hacker0x01")

time.sleep(5)
driver.save_screenshot('screenshot1.png')

tweets = driver.find_element(by=By.CSS_SELECTOR, value='a[class*="css-4rbku5"]')
href_value = tweets.get_attribute("href")
print (href_value)
# for tweet in tweets:
#     href_value = tweets.get_attribute("href")
#     print(href_value)

driver.save_screenshot('screenshot2.png')

driver.quit()
```
