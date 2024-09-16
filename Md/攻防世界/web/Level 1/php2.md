# PHP2

知识点：phps文件&url解码

```php
<?php
if("admin"===$_GET[id]) {
  echo("<p>not allowed!</p>");
  exit();
}

$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "admin")
{
  echo "<p>Access granted!</p>";
  echo "<p>Key: xxxxxxx </p>";
}
?>

Can you anthenticate to this website?
```

简单点来说，phps文件就是php的源码，但用户在浏览器上直接访问php文件是渲染后的，看不到源码，为了让其能够看到，就有了phps文件。

根据源码很好知道，两次判定一次解码，但不要忘了后端会自动进行一次解码，加上代码里的，所以有两次解码，我们进行编码也要编两次。

这次知道了：

`%ASCII`就可以url编码，但不仅符号，普通字母也可以用，但数字如果用的话，会变成字符形式，而不是数字。
