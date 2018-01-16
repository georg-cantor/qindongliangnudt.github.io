```
layout: post
title: "Python网络爬虫笔记-1"
date: 2018-01-16 21:51:59 +0800
categories: 读书笔记
tags: Python Web urllib BeautifulSoup
author: Qindongliang


* content
{:toc}
```


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 什么是Web Scraping](#1-什么是web-scraping)
* [2. 为什么我们需要Web Scraping？](#2-为什么我们需要web-scraping)
* [3. 关于本书](#3-关于本书)
* [4. 第一个Web Scraper](#4-第一个web-scraper)
		* [4.1 Connecting](#41-connecting)
		* [4.2 关于BeautifulSoup的介绍](#42-关于beautifulsoup的介绍)
		* [4.3 Connecting Reliably-如何可靠地连接](#43-connecting-reliably-如何可靠地连接)

<!-- /code_chunk_output -->




## 1. 什么是Web Scraping

General consensus today seems to favor **web scraping**, so that is the term I’ll use throughout the book, although I will occasionally refer to the web-scraping programs themselves as **bots**.

同义词：
- screen scraping,
- data mining,
- web harvesting

定义：
In theory, web scraping is the practice of gathering data through **any means other than** a program interacting with an API (or, obviously, through a human using a web browser).
除了使用API或者人肉使用浏览器之外所有在网上获取数据的方法。

实现方法：
This is most commonly accomplished by writing **an automated program** that queries a web server, requests data (usually in the form of the HTML and other files that comprise web pages), and then parses that data to extract needed information.


## 2. 为什么我们需要Web Scraping？
If the only way you access the Internet is through a browser, you’re missing out on a huge range of possibilities.
如果上网活动都是用浏览器完成的，那么很遗憾，错过了很多可能性。
web scrapers are excellent at gathering and processing large amounts of data (among other things).
想要大量处理数据，必须得使用Web Scraper。

API？
不能太依靠API：
- 如果网站有提供你所需要的API，那非常好！
- 但是很多网站并没有提供API或者提供你所需要的API
- 各种原因：你所需要的数据很小众，站点没有精力开发API

即使有API可用，也有几个问题：
- 一般使用API进行数据获取，数据量和速率受限
- 数据格式固定，不一定满足需求


只要你可以用浏览器获得的数据，那么就一定可以用Web Scraper获得（Python Script），进一步，可以把获取的数据存入数据库，再进一步，你可以对这些数据做任何你想做的处理。

Web Scraping可以做什么呢？
- market forecasting
- machine language translation
- medical diagnositics
- art world：
    + The 2006 project “We Feel Fine” by Jonathan Harris and Sep Kamvar, scraped a variety of English-language blog sites for phrases starting with “I feel” or “I am feeling.” This led to a popular data visualization, describing how the world was feeling day by day and minute by minute.


## 3. 关于本书

This book is designed to serve not only as **an introduction to web scraping**, but as a **comprehensive guide** to scraping almost every type of data from the modern Web.

技术书籍通常会专注于某种技术或者某个领域，但是因为Web Scraping是一种十分综合的技术，所以本书会涉及到：
- database
- web server
- HTTP
-HTML
- Internet Security
- Image Processing
- data science
- other tools

其中：
- 第一部分：专注于web scraping and web crawling in depth。a small handful of libraries

- 第二部分：additional subjects that the reader might find useful when writing web scrapers.一些额外的专题，你会看到很多延伸阅读的推荐。


## 4. 第一个Web Scraper

#### 4.1 Connecting

涉及到的步骤：
This chapter will 
- （1）. start with the basics of sending a GET request to a web server for a specific page, 
- （2）. reading the HTML output from that page, and 
- （3）. doing some simple data extraction in order to isolate the content that we are looking for.

代码：
```python
from urllib.request import urlopen
html = urlopen("http://pythonscraping.com/pages/page1.html") 
print(html.read())
```

关于urllib：

urllib is a **standard Python library** (meaning you don’t have to install anything extra to run this example) and contains functions for requesting data across the web, handling cookies, and even changing metadata such as headers and your user agent. We will be using urllib extensively throughout the book, so we recommend you read the Python documentation for the library。
- 标准库，不需要额外下载
- 请求数据，处理cookies，改变metadata...
- 本书会一直使用urllib，所以有空的话，看看它的[文档](https://docs.python.org/3/library/urllib.html)。

关于urlopen：

urlopen is used to open a remote object across a network and read it. Because it is a fairly generic library (it can read HTML files, image files, or any other file stream with ease), we will be using it quite frequently throughout the book.

#### 4.2 关于BeautifulSoup的介绍
> “Beautiful Soup, so rich and green, 
Waiting in a hot tureen!
Who for such dainties would not stoop? 
Soup of the evening, beautiful Soup!”

The BeautifulSoup library was named after a **Lewis Carroll** poem of the same name in **Alice’s Adventures in Wonderland**. In the story, this poem is sung by a character called the Mock Turtle (itself a pun on the popular Victorian dish Mock Turtle Soup made not of turtle but of cow).
安装：
```zsh
$pip install beautifulsoup4
```
简短的实例：
```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
html = urlopen("http://www.pythonscraping.com/pages/page1.html") 
bsObj = BeautifulSoup(html.read(), "html.parser")
print(bsObj.h1)
```
输出：
```html
<h1>An Interesting Title</h1>
```

注意：以下四个等价
```
bsObj.h1
bsObj.html.body.h1
bsObj.body.h1
bsObj.html.h1
```



#### 4.3 Connecting Reliably-如何可靠地连接

悲惨的情景：
>One of the most frustrating experiences in web scraping is to go to sleep with a scraper running, dreaming of all the data you’ll have in your database the next day—only to find out that the scraper hit an error on some unexpected data format and stopped execution shortly after you stopped looking at the screen. 

让我们回顾一下我们的第一个例子：
```python
html = urlopen("http://www.pythonscraping.com/pages/page1.html")
```
已知，至少有2种情形会导致以上语句失败：
- 在服务器上没有找到所需要的页面--HTTP error
    + “404 Page Not Found,”
    + “500 Internal Server Error,” etc.
- 服务器没有找到--`urlopen`返回`None Object`

如果是HTTP ERROR，则可以如下处理：
```python
try:
    html = urlopen("http://www.pythonscraping.com/pages/page1.html")
except HTTPError as e: 
    print(e)
    #return null, break, or do some other "Plan B"
else:
    #program continues. Note: If you return or break in the 
    #exception catch, you do not need to use the "else" statement
```

服务器没有找到的话，如下处理：
```python
if html is None:
    print("URL is not found")
else:
    #program continues
```

即使成功地从服务器获取到了相应的页面，也会有一些意外发生：
可能页面的内容不是我们期望的那样。
每次获取一个BeautifulSoup object的tag的时候，最好加上检查，确保这个tag真实存在。如果强行获取一个不存在的tagBeautifulSoup将会返回`None Object`，再想去获取某个`None Object`的tag就会返回属性错误（AttributeError）：
```python
AttributeError: 'NoneType' object has no attribute 'someTag'
```
如何避免：
```python
try:
    badContent = bsObj.nonExistingTag.anotherTag
except AttributeError as e: 
    print("Tag was not found")
else:
    if badContent == None:
        print ("Tag was not found")
    else: 
        print(badContent)
```

在以上的交代之后，我们的示例代码应该怎么写呢？

```python
from urllib.request import urlopen 
from urllib.error import HTTPError 
from bs4 import BeautifulSoup
def getTitle(url): 
    try:
        html = urlopen(url) 
    except HTTPError as e:
        return None 
    try:
        bsObj = BeautifulSoup(html.read(), "html.parser")
        title = bsObj.body.h1 
    except AttributeError as e:
        return None 
    return title
title = getTitle("http://www.pythonscraping.com/pages/page1.html") 
if title == None:
    print("Title could not be found") 
else:
    print(title)
```

作者评论道：
>In this example, we’re creating a function getTitle, which returns either the title of the page, or a None object if there was some problem with retrieving it. Inside getTi tle, we check for an HTTPError, as in the previous example, and also encapsulate two of the BeautifulSoup lines inside one try statement. An AttributeError might be thrown from either of these lines (if the server did not exist, html would be a None object, and html.read() would throw an AttributeError). We could, in fact, encompass as many lines as we wanted inside one try statement, or call another func‐ tion entirely, which can throw an AttributeError at any point.
When writing scrapers, it’s important to think about the overall pattern of your code in order to handle exceptions and make it readable at the same time. You’ll also likely want to heavily reuse code. Having generic functions such as getSiteHTML and getTi tle (complete with thorough exception handling) makes it easy to quickly and reliably scrape the web.




