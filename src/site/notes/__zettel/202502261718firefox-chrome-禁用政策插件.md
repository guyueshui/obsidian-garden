---
{"dg-publish":true,"permalink":"/__zettel/202502261718firefox-chrome-禁用政策插件/","title":202502261718,"tags":["firefox","chrome","技巧"],"created":"2025-02-26T17:18:01+08:00"}
---

看到浏览器中“由贵组织管理”让人很恼怒，它装的插件一般是用来监控你的浏览记录和浏览内容的。所以，在这种情况下，你等于没穿衣服在网上冲浪。我们必须强烈抵制这种行为。

![](/img/user/assets/image-20250226.172211.319.png)
![](/img/user/assets/image-20250226.172316.624.png)

网上的一些方法多半是删除注册表，但是公司电脑往往不给你管理员权限，因此行不通。所以我们要另辟蹊径。

chrome禁用政策插件
---
打开你的profile目录，一般为
```
C:\Users\用户名\AppData\Local\Google\Chrome\User Data\Default\Extensions
```
删掉这个插件文件夹（打开开发者模式可以看到插件id，在插件目录里面对应一个插件目录）
![](/img/user/assets/image-20250226.172638.664.png)

firefox禁用政策插件
--
firefox就更牛了，官方文档直接教你怎么禁用。
https://support.mozilla.org/en-US/kb/cannot-remove-add-on-extension-or-theme

具体来说，还是先查看插件。地址栏输入`about:support`，可以看到配置文件夹目录，打开它，里面有个extensions目录存放插件文件。
![](/img/user/assets/image-20250226.173020.014.png)

再往下翻一翻，可以看到插件信息，
![](/img/user/assets/image-20250226.173106.605.png)

可以看到插件ID，删除对应文件即可。

see also [202412112322firefox重启保留标签页](202412112322firefox重启保留标签页.md)