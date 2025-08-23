---
{"dg-publish":true,"permalink":"/zettel/202412222011termux-baidupcs/","title":202412222011,"tags":["baidupcs","termux","android"]}
---

有两种方式编译android版本的baidupcs

1. termux直接配好go环境，直接编译
2. 电脑上下载[androind ndk](https://developer.android.com/ndk/downloads/index.html?hl=zh-cn)，指定编译架构后编译

我最后使用第一个方案成功。第二个虽然编译成功，放到termux里面运行还是会报[dns](https://github.com/qjfoidnh/BaiduPCS-Go/issues/107)的问题。最后work的编译命令行

```bash
GOOS=android GOARCH=arm64 CGO_ENABLE=1 go build  
```

如果下载依赖过慢，设置一下GOPROXY环境变量，

```bash
$ go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
$ go env | grep GOPROX
GOPROXY='https://mirrors.aliyun.com/goproxy/,direct'
```

> [!tip] 关于登录百度账号
> 建议直接使用bduss的方式登录，
>
>     ./BaiduPCS-GO login -bduss <your_bduss_string>
> 账号密码貌似登录不上。bduss获取方式为，打开浏览器，登录pan.baidu.com，查看cookie，找到bduss对应的value即可。


参考

- 使用方案二的参考：https://web.archive.org/web/20190830041727/http://www.iikira.com/2017/08/09/golang-compile-jc-2/
- 报错`invalid reference to runtime.rawbyteslice`修改方案：https://github.com/qjfoidnh/BaiduPCS-Go/pull/344/files
- baidupcs项目地址：https://github.com/qjfoidnh/BaiduPCS-Go