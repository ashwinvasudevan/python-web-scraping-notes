# Resource 1: Python Web Scraping, SE, Katharine Jarmul, Richard Lawson
> https://www.dlutkn.top/2018/11/14/Ebooks-about-ML-Python/pws.pdf

# Chapter 1:
## Is scraping legal? 
If the data scraped is being used for personal use and within fair use of copyright laws, then there is no problem. If the the data is going to be republished or if the scrapping is aggressive to take down the  website or if the content is copyrighted or if the scraped violates TOS, there would be certainly legal issues. 

### What should be done to avoid legal issues?
- Read the website's TOS
- Define a user agent to identify your crawler
- Make only a reasonable amount of requests.
- Make sure the data is not copyrighted

## Background research
Before crawling, understand the site's structure
* Check robots.txt
    *  There might be traps set for crawlers, you can get IP banned very easily
* Sitemaps are provided by websites to help crawlers locate updated content without needing to crawl the entire website again to check for updates
* Sitemaps needed to be treated carefully, sometimes they might be out of date or incorrect.

## Estimating number of pages in a website
* Use site:website to check the number of pages crawled by google's crawler
* Use site:website/blog/ to restrict results to a certain part of the website

## Finding the tech stack
* Use [detectem]( https://github.com/alertot/detectem) or even [Wappalyzer](https://github.com/AliasIO/Wappalyzer)
* If the site uses Jquery and Modernizer mostly the entire page will not be loaded by JS. 
* If the site uses Angular or React, mostly the content is loaded dynamically. 
* If the site uses ASP.net you need to use sessions and form submissions to crawl. 

## Finding the owner
* Use whois (python-whois) to find the emails and the nameservers 
* Be careful scraping domains owned by google, they block web crawlers. 

## Scraping vs Crawling
* Scraping is built to target a particular website.
* Crawling targets all the websites, more general

## Downloading a page
```python
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError

def download(url):
    print("Downloading, ", url)
    try:
        html = urllib.request.urlopen(url).read()
    except (HTTPError, URLError, ContentTooShortError) as e:
        print("Download failed: ", e.reason)
        html = None
    return html
```

## Downloading a page with retries
```python
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError

def download(url, num_of_retries = 2):
    print("Downloading, ", url)
    try:
        html = urllib.error.urlopen(url).read()
    except (URLError, HTTPError, ContentTooShortError) as e:
        print("Downloading failed: ", e.reason)        
        html = None
        if num_of_retries > 0:
            if hasattr(e, 'code') and 500 <= e.code <= 600:
               return download(url, num_of_retries - 1)
    
    return html
```
Some pages block the default urllib user agent, therefore, define a user agent and then use it

```python
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError
 
 def download(url, user_agent = "wswp", num_of_retries = 2):
     print("Downloading, ", url)
     request = urllib.request.Request(url)
     request.add_header('User-agent', user_agent)
     try:
         html = urllib.error.urlopen(url).read()
     except (URLError, HTTPError, ContentTooShortError) as e:
         print("Downloading failed: ", e.reason)        
         html = None
         if num_of_retries > 0:
             if hasattr(e, 'code') and 500 <= e.code <= 600:
                return download(url, num_of_retries - 1)
     return html
```

## Crawling a website using it's sitemap

* `.*` is greedy search, will match the last instance 
* `.*?` is non greedy, will match until the first occurance

```python
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError
import re

def download(url, charset = 'utf-8', user_agent = 'bswp', n_retries = 2):
    print("Downloading: ", url)
    request = urllib.request.Request(url)
    request.add_headers('User-agent', user_agent)

    try:
        resp = urllib.request.urlopen(request)
        cs = resp.headers.get_content_charset()
        if not cs:
            cs = charset
        html = resp.read().decode(cs)
    except (URLError, HTTPError, ContentTooShortError) as e:
        print("Downloading failed: ", e.reason)
        html = None
        if n_retries > 0:
            return download(url, n_retries = n_retries - 1)

    return html

def get_sitemap(url):
    print("Downloading sitemap: ", url)
    sitemap = download(url)
    links = re.findall('<loc>(.*?)</loc>', sitemap)
    for link in links:
        html = download(link)
```

Crawling a website using the ID's with max_retries = 5
```python
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError
import itertools, re

def download(url, user_agent = 'bswp', charset = 'utf-8', n_retry = 2):
    print("Downloading: ", url)
    request = urllib.request.Request(url)
    request.add_headers('User-agent', user_agent)
    try:
        resp = urllib.request.urlopen(request)
        cs = resp.headers.get_content_charset()
        if not cs:
            cs = charset
        html = resp.read().decode(cs)

    except (URLError, HTTPError, ContentTooShortError) as e:
        print("Downloading failed: ", e.reason)
        html = None
        if n_retry > 0:
            return download(url, n_retry = n_retry - 1)

    return html

def crawl_using_ids(url, max_errors = 5)
    for page in itertools.count(1):
        pg_url = "{}{}".format(url,page)
        html = download(pg_url)
        if html is None:
            error_c += 1
            if error_c == max_errors:
                break
        else:
            error_c = 0
```

## Link Crawlers
Link crawlers are used to obtain all the links which fit a pattern 
```python
import re
import urllib.request
from urllib.parse import urljoin
from urllib.error import URLError, HTTPError, ContentTooShortError
from urllib.parse import urljoin

def download(url, user_agent='wswp', num_retries=2, charset='utf-8'):
    print('Downloading:', url)
    request = urllib.request.Request(url)
    request.add_header('User-agent', user_agent)
    try:
        resp = urllib.request.urlopen(request)
        cs = resp.headers.get_content_charset()
        if not cs:
            cs = charset
        html = resp.read().decode(cs)
    except (URLError, HTTPError, ContentTooShortError) as e:
        print('Download error:', e.reason)
        html = None
        if num_retries > 0:
            if hasattr(e, 'code') and 500 <= e.code < 600:
                # recursively retry 5xx HTTP errors
                return download(url, num_retries - 1)
    return html

def get_links(html):
    webpage_regex = re.compile("""<a[^>]href=["'](.*?)["']""")
    return webpage_regex.findall(html)

def link_crawler(start_url, link_regex):
    crawl_queue = [start_url]
    seen = set(crawl_queue)
    while crawl_queue:
        url = crawl_queue.pop()
        html = download(url)
        if not html:
            continue
        links = get_links(html)
        print(links)
        for link in links:
            if re.search(link_regex, link):
                abs_link = urljoin(start_url, link)
                if abs_link not in seen:
                    seen.add(abs_link)
                    crawl_queue.append(abs_link)

link_crawler('http://example.webscraping.com', r'/(index|view)/.*')
```
## Advanced Link crawler with the following support
* proxy
* robot.txt 
* avoid spider_trap
* throttle

```python
import re
import urllib.request
from urllib.error import HTTPError, URLError, ContentTooShortError
from urllib.parse import urlparse, urljoin
from urllib import robotparser
import time
class Throttle:
    def __init__(self, delay):
        self.delay = delay
        self.domains = {}

    def wait(self, url):
        domain = urlparse(url).netloc
        last_accessed = self.domains.get(domain)

        if last_accessed is not None and self.delay > 0:
            sleep_secs = self.delay - (time.time() - last_accessed)
            if sleep_secs > 0:
                time.sleep(sleep_secs)
        self.domains[domain] = time.time()

def download(url, user_agent = 'wbsp', proxy = None, charset = 'utf-8', max_retries = 5):
    print("Downloading: ", url)
    request = urllib.request.Request(url)
    request.add_header('User-agent', user_agent)
    try:
        if proxy:
            proxy_helper = urllib.request.ProxyHandler({'http':proxy})
            opener = urllib.request.build_opener(proxy_helper)
            urllib.request.install_opener(opener)
        resp = urllib.request.urlopen(request)
        cs = resp.headers.get_content_charset()
        if cs is None:
            cs = charset
        html = resp.read().decode(cs)
    except (URLError, HTTPError, ContentTooShortError) as e:
        print("Downloading failed: ", e.reason)
        html = None
        if hasattr(e, 'code') and 500 <= e.code <= 600:
            if max_retries > 0:
                return download(url, max_retries = max_retries - 1)
    return html

def get_all_links(html):
    webpage_regex = re.compile(r"""<a[^>]href=["'](.*?)["']""", re.IGNORECASE)
    return webpage_regex.findall(html)

def get_robot_parser(robot_url):
    rp = robotparser.RobotFileParser()
    rp.set_url(robot_url)
    rp.read()
    return rp

def link_and_crawl(start_url, link_regex, robot_url = None, max_depth = 5, delay = 3, user_agent = 'wbsp') :
    crawl_queue = [start_url]
    seen = {}
    if robot_url is None:
        robot_url = "{}/robots.txt".format(start_url)
    rp = get_robot_parser(robot_url)
    throttle = Throttle(delay)
    while crawl_queue:
        url = crawl_queue.pop()
        if rp.can_fetch(user_agent, url):
            depth = seen.get(url, 0)
            if depth == max_depth:
                continue
            throttle.wait(url)
            html = download(url)
            if not html:
                continue
            for link in get_all_links(html):
                if re.search(link_regex, link):
                    abs_link = urljoin(start_url, link)
                    if abs_link not in seen:
                        seen[abs_link] = depth + 1
                        crawl_queue.append(abs_link
        else:
            print("Download restricted by robots.txt")
link_and_crawl('http://example.webscraping.com/', link_regex = r"""(/view/.*?)""")
```
## Using requests for the same

```python
def download(url, user_agent = 'wsd', proxy = None, max_retries = 5):
    print("Downloading: ", url)
    headers = {'User-Agent': user_agent}
    try:
        resp = requests.get(url, headers = headers, proxies = proxies)
        html = resp.text
        if resp.status_code >= 400:
            print("Downloading failed: ",resp.text)
            html =None
            if max_retries and 500 <= resp.status_code < 600:
                return download(url, max_retries - 1)
    except request.exceptions.RequestExceptions as e:
        print("Download Error", e.reason)
        html = None
    return html
```
# Chapter 2:
* Use requests for any website, donot use urrlib.request

## Three different ways to scrap a website

### Regular Expressions
`re.findall(r"""<tr id="places_area__row">(.*?)</tr>""")`
* Use the parent too, nodes with IDs are better

### BeautifulSoup
