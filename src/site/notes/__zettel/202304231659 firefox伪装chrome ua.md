---
{"dg-publish":true,"permalink":"/__zettel/202304231659 firefox伪装chrome ua/","title":202304231659,"tags":["firefox","chrome","browser","ua","useragent"],"created":"2023-04-23T16:59:37+08:00"}
---


许多网站强制建议使用chrome打开，但其实firefox能够完美渲染该页面。所以需要强制更改firefox的ua伪装chrome：

1. firefox输入about:config
2. 搜索general.useragent.override，没有则新建之，值类型为字符串
3. 打开chrome看下它的ua，输入about:version，或者直接使用 `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36`
4. 复制chrome的ua到新建的值中


see: https://sspai.com/post/75349