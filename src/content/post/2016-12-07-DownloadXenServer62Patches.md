+++
categories = ["Linux"]
date = "2016-12-07T18:03:31+08:00"
description = "Download XenServer 6.2 Patches"
keywords = ["Linux"]
title = "DownloadXenServer62Patches"

+++
The script is listed as following, use python2 for running it:    

```
from bs4 import BeautifulSoup
import urllib2
import re

## Pages contains the download page url
url_list = [
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=0",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=10",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=20",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=30",
]

for url in url_list:
  #response = urllib2.urlopen('https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=0')
  response = urllib2.urlopen(url)
  
  ### Fetch the urllib2 result.
  html = response.read()
  # print len(html)
  
  ### Make soup of the html.
  soup = BeautifulSoup(html, 'html.parser')
  # print soup.prettify()
  
  ### Get all of the download page link:
  for i in soup.findAll('h4'):
    for a in i.findAll('a'):
      #print a['href']
      page_response = urllib2.urlopen(a['href'])
      page_html = page_response.read()
      page_soup = BeautifulSoup(page_html, 'html.parser')
      for elem in page_soup(text=re.compile(r"window.location.href")):
        prefix = elem.string.find("supportkc")
        #print prefix
        url_begin_pos = elem.string.find("window.location.href") + 24
        url_end_pos = elem.string.find("zip") + 3
        if prefix > 0:
          print "https://support.citrix.com"+elem.string[url_begin_pos:url_end_pos]
        else:
          print elem.string[url_begin_pos:url_end_pos]
```
The total patches are around 2G size.    

The correct version:    

```
from bs4 import BeautifulSoup
import urllib2
import re

## Pages contains the download page url
url_list = [
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=0",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=10",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=20",
"https://support.citrix.com/search?searchQuery=%3F&lang=en&sort=cr_date_desc&ct=Hotfixes&ctcf=Recommended&prod=XenServer&pver=XenServer+6.2.0&st=30",
]

for url in url_list:
  response = urllib2.urlopen(url)
  
  ### Fetch the urllib2 result.
  html = response.read()
  # print len(html)
  
  ### Make soup of the html.
  soup = BeautifulSoup(html, 'html.parser')
  # print soup.prettify()
  
  ### Get all of the download page link:
  for i in soup.findAll('h4'):
    for a in i.findAll('a'):
      print a['href']
      print a.text
      page_response = urllib2.urlopen(a['href'])
      page_html = page_response.read()
      page_soup = BeautifulSoup(page_html, 'html.parser')
      try:
        print page_soup.findAll('blockquote')[1].text
      except:
        pass
      else:
        pass
      try:
        print page_soup.find_all('span',attrs={'class':'block-quote'})[1].text
      except:
        pass
      else:
        pass
      for elem in page_soup(text=re.compile(r"window.location.href")):
        prefix = elem.string.find("supportkc")
        #print prefix
        url_begin_pos = elem.string.find("window.location.href") + 24
        url_end_pos = elem.string.find("zip") + 3
        if prefix > 0:
          print "https://support.citrix.com"+elem.string[url_begin_pos:url_end_pos]
        else:
          print elem.string[url_begin_pos:url_end_pos]
```
Manually get the result, and this time we get the uuid.    

a tip for reverse the output is using `tac` rather than `cat`.    
