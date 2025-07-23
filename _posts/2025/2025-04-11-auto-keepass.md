---
title: 密码管理-keepass云托管（坚果云）
description: 半自动化的密码本管理,和多端同步
date: 2025-04-11
categories:
  - tool
tags:
  - tool
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
---

> 做好备份和安全管理，找回服务和记忆有时也并不可靠，尤其在有些特殊情况需要单独记起密码，具有相应的使用场景
{: .prompt-info }

# 1.准备阶段
1. 下载 [KeePassXC](https://keepassxc.org/) （选择对应你电脑系统的版本） 版本
2. 下载 [Keepassxc-browser（链接是chrome，其他浏览器可通过谷歌查询）](https://chromewebstore.google.com/detail/keepassxc-browser/oboonakemofpalcgghocfoadofidjkkk?hl=zh-CN&utm_source=ext_sidebar) 这款插件 
	1. 注意：不同浏览器到对应的商店下载，下载github最新的插件会导致Keepassxc-browser不跳出关联本地的KeePassXC窗口
3. 注册[坚果云](https://www.jianguoyun.com/) 
以下是下载链接：

| 平台    | 客户端                                                                                                        |
| ------- | ------------------------------------------------------------------------------------------------------------- |
| windows | [KeePass【https://keepass.info/download.html】](https://keepass.info/download.html)                           |
| 安卓端  | [Keepass2Android【https://github.com/PhilippC/keepass2android】](https://github.com/PhilippC/keepass2android) |
| 浏览器  | [keepassxc-browser【https://keepassxc.org/download/#browser】](https://keepassxc.org/download/#browser)       |
| IOS     | 自行在应用商店搜索                                                                                            |
|         |                                                                                                               |

# 2.实际操作
注意：**电脑的KeePassXC不能和云同步**，需要手动上传文件，所以是半自动。
## 半自动实现
1. 【电脑端KeePassXC】创建KeePass 数据库，参考 [KeePass 使用入门教程](https://post.smzdm.com/p/489402/)。
	1. **密码**需要记牢，这是你唯一能打开这个库的方法
2. 【坚果云】在坚果云上创建一个同步专用文件夹（建议勾选“默认不同步到本地”选项）。

![img](assets/attachments/2025/auto-keepass/auto-keepass01.jpg)

3. 【坚果云】将 KeePass 数据库文件上传到该文件夹。
4. 【坚果云】设置 WebDAV 权限，记录下框里面信息
5. 【手机端KeePassXC】 选择 打开文件--》HTTPS（webDav）--》填入第4步【坚果云】记录的信息--》输入第1步记录的密码 ，这样就完成了云端和手机本地的同步

![img](assets/attachments/2025/auto-keepass/auto-keepass02.jpg)

## 电脑端的KeePassXC和Keepassxc-browser自动填充
1. 重新打开【电脑端KeePassXC】并输入密码打开刚才的数据库
2. 打开浏览器并启用Keepassxc-browser插件
	1. 这时候会跳出输入ID的弹框，创建一个名称（Key Name）。就会自动关联
3. 测试：在KeePassXC配置了对应域名的账号密码，打开对应的网站测试即可

>参考资料：
>1. https://www.cnblogs.com/xututu6/p/18646095
>2. https://segmentfault.com/a/1190000041381245


---
故事未完:99
**Thoughts**:: justdoit.
