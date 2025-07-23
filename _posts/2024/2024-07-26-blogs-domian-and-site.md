---
title: blog_03_bolg添加域名和服务部署
description: 域名和服务部署
date: 2024-07-26
categories:
  - blog
tags:
  - blog
auther: blackbzy
update_date: 2025-01-01
pin: false
toc: true
comments: 
render_with_liquid: false
media_subpath: 
---

> 记录博客的建站和域名选择
{: .prompt-info }

---

# mian

## 0.域名备案会存在的问题
1. 不用之后会被赌博网站利用，而监管的无能，反而会查到正常备案的用户
2. 备案之后会被各种网络监察抽查，带博客就不行，评论开放不行，标题不行 。没有明确的标准那就只能被当做鸡宰
3. 搞海外主机才是王道

所以出于备案和网络监查这些存在不公开的潜规则的可能，选择境外的服务器和域名。
## 1.域名选择
1. 基于相对便宜可靠的原则在[domain](https://www.domain.com/) 和 [godaddy](https://cart.godaddy.com/) 中选择，价格差不多，所以选择最大的运营商[godaddy](https://cart.godaddy.com/) ：3年`.com`域名+安全保护 80$
xxx.com

1. 需要配置dns：基于下面的服务器项目搭建完成
	1. vercel的projects中选择搭建好的项目，在setting中选择Domains选项中黏贴上对应的域名
	2. vercel的projects中选择搭建好的项目，DNS Records中有提供对应的IP，记录一下IP
	3. 到godaddy的域名项目中，选择dns标签，修改类型为A的数据，填入刚才记录的IP保存
2. 等待vercel的自动刷新即可，刷新成功尝试登入
## 2.服务器选择
[vercel](https://vercel.com/) ，hobby账户可以有免费的额度，对于博客来说足够
1. 登入vercel ，参考[如何在vercel中部署jykell项目](https://vercel.com/guides/deploying-jekyll-with-vercel) ，选择jykell，然后登录git账号，新建仓库（这个后面可以删除，换上自己选择的主题的仓库和分支），选择对应的仓库。点确认，其他都是默认会自动部署
2. 访问服务的地址（需要翻墙），一般这个都会成功。然后可以配置域名，基于上一步去操作
3. 解绑这个项目的仓库，换上自己选择的主题的仓库和分支然后确认
这个时候会报错：
```sh
Error: No Output Directory named "_site" found after the Build completed. You can configure the Output Directory in your Project Settings.
```

导致这个报错的原因是：
没有执行 bundle install命令

于是我选择对应的Project的setting项，在修改Build Command的默认配置，勾选override项，点击保存：
```sh
# 原有配置
jekyll build
# 修改后
bundle exec jekyll build
```
4. 选择对应的Project，选最上面的deployments，会发现最新的一条记录是报错的，点进去，点击reDeploy按钮。
5. 等待部署完成，访问域名成功

完结(⌐■_■)

## 3.设置cloudflare解析dns
vercel的访问不稳定，容易国内之间访问挂掉，所以再加cloudflare解析会比较稳
多次添加dnssec找不到添加按钮，结果发现[官方文档](https://www.godaddy.com/zh-sg/help/what-is-dnssec-6135)一句话：
```md
 **注意：**如果您的域名已在 GoDaddy 注册，但未使用我们的域名服务器，则可以在您的域名中手动[添加 DS 记录](https://www.godaddy.com/zh-sg/help/add-a-ds-record-23865)。

### 正确切换dns方式
1. 登录[cloudflare](https://dash.cloudflare.com/)
2. 点击添加站点，填入域名，选择免费的套餐
3. 选择域名进行配置
4. 点击dns标签下的设置，开启dnssec
5. 切换导godaddy，更换godaddy的dns的 域名服务器设置，替换为cloudflare的
```
kristina.ns.cloudflare.com
roman.ns.cloudflare.com
```
2. 添加ds记录
至此配置完成等待dns的自动更新（vercel的代理配置因为我在之前配置过，dns服务器有记录，这次没有重复配置）
### 切换后续
切换域名到cloudflare，访问报错：www.blackbzy.com 将您重定向的次数过多。
请教了gpt得到如下解决方案：
```yml
- **SSL/TLS 设置冲突：**
    
    - **Cloudflare SSL/TLS 模式：** 如果你在 Cloudflare 中将 SSL/TLS 设置为 “Flexible” 模式，而你的服务器已经配置了 HTTPS，这可能会导致重定向循环。原因是 Cloudflare 在“Flexible”模式下会尝试通过 HTTP 与服务器通信，而服务器可能会重定向到 HTTPS，导致无限重定向循环。
    - **解决方法：** 将 Cloudflare 的 SSL/TLS 模式切换为 “Full” 或 “Full (Strict)” 模式。如果服务器已经正确配置了 SSL 证书，建议使用 “Full (Strict)” 模式。
```

于是在cloudflare配置ssl为 完全（严格）模式。
重新访问成功。

---
故事未完:181
**Thoughts**:: justdoit.
