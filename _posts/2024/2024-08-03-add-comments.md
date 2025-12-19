---
title: blog_04_博客添加评论模块
description: 添加评论模块
date: 2024-08-03
categories:
  - blog
tags:
  - blog
author: blackbzy
update_date: 2025-12-19
pin: false
toc: true
comments: 
render_with_liquid: false
media_subpath: 
---

> 基于waline添加comments
{: .prompt-info }

---
## 1.前期试错
尝试基于valine添加comment 模块：
发现没有关于jykell的官方文档，只有[valine](https://duter2016.github.io/2019/09/18/Jekyll%E6%B7%BB%E5%8A%A0Valine%E8%AF%84%E8%AE%BA-%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5%E5%92%8C%E8%AF%84%E8%AE%BA%E5%88%97%E8%A1%A8%E5%A4%B4%E5%83%8F/) 的社区文档，于是尝试
- 修改blog项目启动之后发现没用，没用排查头绪遂放弃
## 2.尝试基于Waline添加comment 模:
这次是有[官方文档](https://waline.js.org/guide/get-started/)：jykell特殊调整内容如下
### 2.1数据库链接leanCloud
服务端基于vercel切换（基于[deta](https://waline.js.org/guide/deploy/deta.html)进行部署，也是可以的，尝试了一下没问题）
系好leancloud提供的key，在vercel 的 chipry项目填入恰当的值
```
LEAN_MASTER_KEY:
LEAN_KEY：
LEAN_ID：
```

### 2.2新增waline 模板
新建文件  _includes/comments/waline.html
```md
<script>
  (function () {
    const walineServerURL = 'https://comment.blackbzy.com/';

    // 1. 动态加载 CSS
    if (!document.getElementById('waline-style')) {
      const link = document.createElement('link');
      link.id = 'waline-style';
      link.rel = 'stylesheet';
      link.href = 'https://unpkg.com/@waline/client@v3/dist/waline.css';
      document.head.appendChild(link);
    }

    // 2. 注入评论框容器
    const walineDiv = document.createElement('div');
    walineDiv.id = 'waline';
    const $footer = document.querySelector('footer');
    if ($footer) {
      $footer.insertAdjacentElement('beforebegin', walineDiv);
    }

    // 3. 异步初始化
    import('https://unpkg.com/@waline/client@v3/dist/waline.js').then(({init, pageviewCount}) => {

      // 初始化评论
      init({
        el: '#waline',
        serverURL: walineServerURL,
        dark: 'html[data-mode="dark"]',
        reaction: true,
        pageview: true // 开启记录功能
      });

      // 初始化阅读量显示
      pageviewCount({
        serverURL: walineServerURL,
        update: true,
        selector: '.waline-pageview-count'
      });

    });
  })();
</script>

```
### 2.3配置文件增加相关代码
同时在_config.yml文件中增加waline相关配置：
```yml
comments:
  provider: waline # [disqus | utterances | giscus]
  waline:
    server: https://comments.blackbzy.com/ # Vercal 服务端地址
    placeholder: 说点什么吧！ # 空白评论框时显示的文字
    avatar: mp # 默认头像  

```
### 2.4添加页面浏览量
修改 _layouts/post.html
请找到文件中 第 85 行到第 94 行 左右的位置，也就是这部分：

```md

{% if site.pageviews.provider and site.analytics[site.pageviews.provider].id %}
<span>
<em id="pageviews">
<i class="fas fa-spinner fa-spin small"></i>
</em>
{{ site.data.locales[lang].post.pageview_measure }}
</span>
{% endif %}
将其替换为以下代码：

HTML

<span>
  <i class="far fa-eye fa-fw"></i>
  <span class="waline-pageview-count" data-path="{{ page.url }}">
    <i class="fas fa-spinner fa-spin small"></i>
  </span>
  {{ site.data.locales[lang].post.pageview_measure }}
</span>
```


### 2.5添加评论的邮箱通知
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

最后重启服务即可

---
故事未完:216
**Thoughts**:: justdoit.
