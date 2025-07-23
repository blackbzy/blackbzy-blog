---
title: blog_04_博客添加评论模块
description: 添加评论模块
date: 2024-08-03
categories:
  - blog
tags:
  - blog
auther: blackbzy
update_date: 2025-02-28
pin: false
toc: true
comments: 
render_with_liquid: false
media_subpath: 
---

> 基于waline添加comments
{: .prompt-info }

---
# mian
1. 尝试基于valine添加comment 模块：
发现没有关于jykell的官方文档，只有[valine](https://duter2016.github.io/2019/09/18/Jekyll%E6%B7%BB%E5%8A%A0Valine%E8%AF%84%E8%AE%BA-%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5%E5%92%8C%E8%AF%84%E8%AE%BA%E5%88%97%E8%A1%A8%E5%A4%B4%E5%83%8F/) 的社区文档，于是尝试
	1. 修改blog项目启动之后发现没用，没用排查头绪遂放弃
##  2. 尝试基于Waline添加comment 模:
这次是有[官方文档](https://waline.js.org/guide/get-started/)：jykell特殊调整内容如下
	1. 数据库保持leanCloud，服务端基于vercel切换（基于[deta](https://waline.js.org/guide/deploy/deta.html)进行部署，也是可以的，尝试了一下没问题）
	2. Waline部署完成，然后要在blog中添加的官方配置，单独建一个文件`_includes/head.html` ，内容如下：
```md
`<!-- waline 评论框 start -->

<div id="waline"></div>

<!-- waline 评论框 end -->
<!-- 引入 waline 样式 -->
<link rel="stylesheet" href="https://unpkg.com/@waline/client@v2/dist/waline.css">
<!-- 引入 waline 模块并初始化 -->
  <script type="module">
    import { init } from 'https://unpkg.com/@waline/client@v2/dist/waline.mjs';
    const locale = {
      placeholder: '{{ site.comments.waline.placeholder }}',
    };
    init({
      el: '#waline',
      dark: 'auto', // 自动暗黑模式
      reaction: true,  // 文章反应
      search: false,  // 表情包搜索
      serverURL: '{{ site.comments.waline.server }}',
      // 设置 emoji 为微博与哔哩哔哩
      emoji: [
      '//unpkg.com/@waline/emojis@1.1.0/weibo',
      '//unpkg.com/@waline/emojis@1.1.0/bilibili',
      '//unpkg.com/@waline/emojis@1.1.0/tw-emoji',
      '//unpkg.com/@waline/emojis@1.1.0/qq',
      ],
      dark: "__waline__css__",
      locale,
    });
    let head = document.getElementsByTagName("head")[0];
    let css = head.lastChild;
    let cssContent = css.textContent.replace("__waline__css__", "");
    let cssContentPerferredDark = "@media (prefers-color-scheme: dark){html:not([data-mode])" + cssContent + "}";
    let cssContentSelectedDark = "html[data-mode=dark]" + cssContent;
    css.textContent = cssContentPerferredDark;
    let style = document.createElement('style');
    style.textContent = cssContentSelectedDark;
    head.appendChild(style);
  </script>`
```

同时在_config.yml文件中怎加waline相关配置：
```yml
comments:
  provider: waline # [disqus | utterances | giscus]
  waline:
    server: https://comments.blackbzy.com/ # Vercal 服务端地址
    placeholder: 说点什么吧！ # 空白评论框时显示的文字
    avatar: mp # 默认头像  

```

最后提交代码部署完成。

## 添加评论的邮箱通知
[官方参考](https://waline.js.org/guide/features/notification.html)
在vercel对应的容器中添加以下字段的环境变量：
```
SMTP_SERVICE: SMTP 邮件发送服务提供商。
SMTP_USER: SMTP 邮件发送服务的用户名，一般为登录邮箱。
SMTP_PASS: SMTP 邮件发送服务的密码，一般为邮箱登录密码，部分邮箱(例如 163)是单独的 SMTP 密码。
SMTP_SECURE: 是否使用 SSL 连接 SMTP。
SITE_NAME: 网站名称，用于在消息中显示。
SITE_URL: 网站地址，用于在消息中显示。
AUTHOR_EMAIL: 博主邮箱，用来接收新评论通知。如果是博主发布的评论则不进行提醒通知。
```
![](assets/attachments/blog/blog01.png)

最后重启日期即可

---
故事未完:216
**Thoughts**:: justdoit.
