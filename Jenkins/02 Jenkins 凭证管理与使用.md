---
title: 'Jenkins凭证管理与使用'
tags:
  - Jenkins
categories:
  - Jenkins
date: 2022-09-01 16:03:00
top_img: transparent
cover: https://img1.baidu.com/it/u=1706157547,2217937168&fm=253&fmt=auto&app=138&f=JPG?w=889&h=500
---

## 1. 凭证介绍

`Jenkins`凭证是指很多可以和 `Jenkins`进行交互的第三方站点和应用程序，如 GitHub，SonarQube，Jira，Docker 等为 `Jenkins` 专门配置的凭证，通过这样的专门的凭证可以将 `Jenkins` 对第三方站点和应用程序的可用功能区间锁定在特定的范围内。

## 2. 凭证的种类

`Jenkins`可以存储以下类型的凭证：

![image-20220901160442938](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901160442938.png)

- Username with password (用户名和密码) ：可以是单独的用户名、密码，也可以是冒号分隔的字符串（格式为 username:password），如 GitHub 的用户名和密码。
- GitHub App ： Github App 可以通过 Github 提供的认证信息去调用 Github API。GitHub端的设置路径在 GitHub 的 Settings / Developer Settings 下。
- SSH Username with private key （ SSH 用户名与私有密钥）：一个 SSH 公钥/私钥对。
- Secret file（加密文件）：本质上是文件中的加密内容。
- Secret text（加密文本）：API令牌（例如 GitHub 个人访问令牌）。
- Certificate （证书）：一个PKCS＃12证书文件和可选密码。

## 3. 凭证的安全性

为了最大程度地提高安全性，在`Jenkins` 中配置的凭证会以加密形式存储在 `Jenkins` 实例上（基于`Jenkins` 实例 ID 进行加密），并且`Jenkins Job`中仅通过凭证 ID 来使用。

这最大程度地减少了将实际凭证本身暴露给 `Jenkins` 用户的机会，并阻碍了将凭证从一个`Jenkins` 实例复制到另一个实例的可能。

## 4. 配置凭证

任何具有"凭证">"创建"权限的`Jenkins` 用户都可以将凭证添加到 `Jenkins`。`Jenkins` 用户可以使用 Administer 权限来配置这些权限。

同时，如果你的`Jenkins`实例的"全局安全配置"页面的"授权策略"被设置为默认的"登录用户可以执行任何设置"或"任何人可以做任何设置"，则任何`Jenkins`用户都可以添加和配置凭证。

![image-20220901161052227](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901161052227.png)

## 5.创建凭证

选择适合的凭证类型

![image-20220901161419196](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901161419196.png)

将凭证本身添加到所选凭证类型的相应字段中

- 加密文本：复制加密文本并将其粘贴到"Secret"字段中。
- 用户名和密码：在相应字段中指定凭证的用户名和密码
- 加密文件：点击选择文件按钮旁边的文件字段选择秘密文件上传到`Jenkins`。
- 带有私钥的 SSH 用户名-在其各自的字段中指定凭证 Username， Private Key 和可选的 Passphrase。
  注意：直接选择 Enter 可让您复制私钥的文本并将其粘贴到生成的 Key 文本框中。
- 证书：指定证书和可选的密码。选择上传 PKCS＃12 证书，可以通过出现的上传证书按钮将证书作为文件上传。

## 6.最佳实践

>  创建 Username and password 凭证

![image-20220901161739424](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901161739424.png)

> 创建  SSH Username with private key  凭证

![image-20220901161942028](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901161942028.png)

> 为了便于管理和使用， 强烈建议使用统一的约定来指定 credential ID

![image-20220901162234872](https://picture-typora-bucket.oss-cn-shanghai.aliyuncs.com/typora/image-20220901162234872.png)
