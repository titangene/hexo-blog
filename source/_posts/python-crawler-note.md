---
title: Python 爬蟲常用技巧 (持續更新)
date: 2019-02-28 17:33:51
author: Titangene
tags:
  - Python
  - Python Requests
  - Crawler
categories:
  - Python
cover_image: /images/cover/python_crawler.png
---

紀錄個人常用的 Python 爬蟲技巧，未來還會持續更新其他技巧。

<!-- more -->

## 載入常用套件

```python
import json
from urllib.parse import urlparse, parse_qs

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
from urllib.parse import urlparse, parse_qs

url = 'http://xxx.com/api/data?id=123&sub_code=06A1297'

link_parse = urlparse(url)
link_query = parse_qs(link_parse.query)

id = link_query['id'][0]
sub_code = link_query['sub_code'][0]
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

## 下載圖片

```python
import requests

urls = [
  "http://www.xxx.com/xxx.png",
  "http://www.xxx.com/yyy.png",
  "http://www.xxx.com/zzz.png"
]

index = 1

for url in urls:
    r = requests.get(url)
    with open('{}.png'.format(index), 'wb') as file:
        file.write(r.content)
    index += 1
```