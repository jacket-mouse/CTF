# baby_web

开始时为1.php，调成index.php还是显示1.php，但看了题解才发现，我们得到了index.php在网络中，响应头里有flag，只不过界面通过location响应头进行了重定向。

# Training-WWW-Robots

一看名字就知道和robots.txt有关，网页也给了提示，用了御剑工具进行扫描，果然扫到了，里面是

```txt
User-agent: *
Disallow: /fl0g.php

User-agent: Yandex
Disallow: *
```

直接出结果