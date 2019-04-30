---
title: Python 爬蟲常用技巧 (持續更新)
date: 2019-02-28 17:33:51
author: Titangene
tags:
  - Python Requests
categories:
  - Python
  - Crawler
cover_image: /images/cover/python_crawler.png
---

紀錄個人常用的 Python 爬蟲技巧，未來還會持續更新其他技巧。

<!-- more -->

## 載入常用套件

```python
import json
import shutil
import xml.etree.ElementTree as ET
from urllib.parse import urlparse, parse_qs, urlunparse

import requests
from bs4 import BeautifulSoup
from user_agent import generate_user_agent
import numpy as np
import pandas as pd
```

## 解析 HTML

```python
import requests
from bs4 import BeautifulSoup

response = requests.get(url)
soup = BeautifulSoup(response.text, 'lxml')
html = soup.find(id='some_id')
```

## 解析網址

```python
from urllib.parse import urlparse, parse_qs, urlunparse

url = 'http://xxx.com/api/data?id=123&sub_code=06A1297'
link_parse = urlparse(url)
print(link_parse)
```

輸出：

```
ParseResult(scheme='http', netloc='xxx.com', path='/api/data', params='', query='id=123&sub_code=06A1297', fragment='')
```

### 將 `ParseResult` 物件轉成網址字串

```python
url = urlunparse(link_parse)
print(url)
```

輸出：

```
'http://xxx.com/api/data?id=123&sub_code=06A1297'
```

### 解析網址 query

```python
link_query = parse_qs(link_parse.query)
print(link_query)

id = link_query['id'][0]
sub_code = link_query['sub_code'][0]
print(id, sub_code)
```

輸出：

```
{'id': ['123'], 'sub_code': ['06A1297']}
123 06A1297
```

## 隨機生成 user-agent

### fake_useragent 套件

```python
import requests
from fake_useragent import UserAgent

def set_header_user_agent():
    user_agent = UserAgent()
    return user_agent.random

user_agent = set_header_user_agent()
response = requests.get(url, headers={ 'user-agent': user_agent })
```

### user_agent 套件

```python
import requests
from user_agent import generate_user_agent

user_agent = generate_user_agent()
response = requests.get(url, headers={ 'user-agent': user_agent })
```

## 讀取表格資料

```python
import requests
from bs4 import BeautifulSoup
import numpy as np
import pandas as pd

response = requests.get(url)
soup = BeautifulSoup(response.text, 'lxml')
html = soup.find(id='table_id')
df = pd.read_html(str(html), header=0)[0]
```

## 讀取/解析 JSON

```python
import json

# 讀取
with open('data.json', encoding='utf-8') as file:
    datas = json.load(file)

# 解析
with open('data.json', 'w', encoding='utf-8') as file:
    json.dump(datas, file, ensure_ascii=False)
```

## 讀取/解析 XML

### 讀取 XML 檔

```python
import xml.etree.ElementTree as ET

tree = ET.parse('data.xml')
root = tree.getroot()
```

### 讀取 XML 字串

```python
import xml.etree.ElementTree as ET

root = ET.fromstring(some_xml_strings)
```

以下面的 XML 為範例：

```xml
<?xml version="1.0"?>
<bookstore>
  <book ISBN="10-000000-001">
    <title>Book 1</title>
    <price>300</price>
    <comments>
      <userComment rating="4">xxx...</userComment>
      <userComment rating="2">yyy...</userComment>
    </comments>
  </book>
  <book ISBN="10-000000-999">
    <title>Book 2</title>
    <price>500</price>
    <comments>
      <userComment rating="3">zzz...</userComment>
    </comments>
  </book>
</bookstore>
```

### 迴圈取得子元素

```python
for child in root:
    print(child.tag, child.attrib)
```

輸出：

```
book {'ISBN': '10-000000-001'}
book {'ISBN': '10-000000-999'}
```

### 用 index 取得某元素

```python
root[0][1].tag
root[0][1].text
root[0].attrib
```

輸出：

```
'price'
'300'
{'ISBN': '10-000000-001'}
```

### 搜尋指定元素

使用 `root.iter()`：

```python
for comment in root.iter('userComment'):
    print(comment.attrib)
    print('comment: {}'.format(comment.text))
```

輸出：

```
{'rating': '4'}
comment: xxx...
{'rating': '2'}
comment: yyy...
{'rating': '3'}
comment: zzz...
```

使用 `root.findall()`：

```python
for book in root.findall('book'):
    ISBN = book.get('ISBN')
    title = book.find('title').text
    print(ISBN, title)
```

```
10-000000-001 Book 1
10-000000-999 Book 2
```

## 下載圖片

`response.raw` 是 file-like 物件，預設不會解壓縮 response (使用 gzip 或 deflate，參考至 [Requests 原始碼](https://github.com/kennethreitz/requests/blob/master/requests/utils.py#L808))，可以透過在 `requests.get()` 方法中，新增參數 `stream=True` 來強制解壓縮，並且可以避免立即將大的 response 內容讀入記憶體內，接著使用 `shutil.copyfileobj()` 讓 Python 將 串流資料轉成檔案物件。

```python
import requests
import shutil

url = "http://www.xxx.com/xxx.png"
response = requests.get(url, stream=True)
file_name = url.split('/')[-1]
with open(file_name, 'wb') as file:
    shutil.copyfileobj(response.raw, file)
```

如果要下載檔案大的圖片，可參考下個段落「[下載大檔案](#下載大檔案)」。

:::info
`shutil.copyfileobj(fsrc, fdst[, length])`：將 file-like 物件 `fsrc` 的內容複製到 file-like 物件 `fdst`。`length` 參數 (int) 是 buffer 的大小。如果 `length` 為負數則代表是複製資料，而不以 chunk 的形式循環原始資料；預設是資料以 chunk 的形式讀取，以避免不受控制的記憶體消耗。請注意，如果 `fsrc` 物件的當前檔案位置不為 0，則只複製從當前檔案位置到檔案末端的內容。

> 參考至 [shutil - High-level file operations - Python 3.7.3 documentation](https://docs.python.org/3/library/shutil.html#shutil.copyfileobj) 官方文件。
> :::

> 參考來源：[python - How to download image using requests - Stack Overflow](https://stackoverflow.com/questions/13137817/how-to-download-image-using-requests)

## 下載大檔案

以迭代的方式取得，預設會以每 128 byte 為一個 chunk 來讀取資料 (參考至 [Requests 原始碼](https://github.com/kennethreitz/requests/blob/master/requests/models.py#L688))：

```python
import requests

response = requests.get(url, stream=True)
file_name = url.split('/')[-1]
with open(file_name, 'wb') as file:
    for chunk in response:
        file.write(chunk)
```

若要自訂 chunk 大小，可使用 [`Response.iter_content()`](https://github.com/kennethreitz/requests/blob/master/requests/models.py#L729) 方法自訂：

```python
import requests

file_name = url.split('/')[-1]
with requests.get(url, stream=True) as response:
    response.raise_for_status()
    with open(file_name, 'wb') as file:
        for chunk in response.iter_content(chunk_size=8192):
            if chunk: # filter out keep-alive new chunks
                file.write(chunk)
                # file.flush()
```

> 參考來源：[Download large file in python with requests - Stack Overflow](https://stackoverflow.com/questions/16694907/download-large-file-in-python-with-requests)

若想要以一次一行的方式迭代 response 資料，可使用 [`Response.iter_lines()`](https://github.com/kennethreitz/requests/blob/master/requests/models.py#L784) 方法，此方法預設一個 chunk 的大小為 512 byte，若要設定分隔符號，可加上 `delimiter` 參數：

```python
import json
import requests

r = requests.get(url, stream=True)
for line in r.iter_lines():
    # filter out keep-alive new lines
    if line:
        print(json.loads(line))
```

> 參考來源：[Python Requests 庫：HTTP for Humans - 再見紫羅蘭 - 博客園](https://www.cnblogs.com/linxiyue/p/3980003.html)
