---
title: "Ssl Expire Check"
date: 2021-07-16T17:31:26+08:00
description: "SSL Expire Check"
tags: ["python","SSL"]
categories: ["devops"]
keywords: ["python","ssl"]
draft: false
isCJKLanguage: true
---

先从阿里云导出域名列表

```python
import pandas as pd
import socket
import ssl
import datetime

excel_path = './domain.io.xlsx'
d = pd.read_excel(excel_path, sheet_name="domain.io")
d1 = d[(d.记录类型 != 'MX') & (d.记录类型 != 'TXT')].主机记录
name=[]
for index, row in d1.iteritems():
     name.append(row)
    
domain = [i + ".domain.io" for i in name]


def ssl_expiry_datetime(hostname):
    ssl_date_fmt = r'%b %d %H:%M:%S %Y %Z'

    context = ssl.create_default_context()
    conn = context.wrap_socket(
        socket.socket(socket.AF_INET),
        server_hostname=hostname,
    )
    # 3 second timeout because Lambda has runtime limitations
    conn.settimeout(3.0)

    conn.connect((hostname, 443))
    ssl_info = conn.getpeercert()
    # parse the string from the certificate into a Python datetime object
    return datetime.datetime.strptime(ssl_info['notAfter'], ssl_date_fmt)

for i in domain:
    try:
        t=ssl_expiry_datetime(str(i))
        print(t,end=' ')
        print(i)
    except Exception as e:
        print(e,end=' ')
        print(i)
```
