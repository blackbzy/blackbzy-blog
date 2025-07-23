---
title: blog_05_bolg添加标识
description: 真人创作标识
date: 2024-12-18
categories:
  - blog
tags:
  - blog
auther: blackbzy
update_date: 2024-12-18
pin: false
toc: true
comments: 
render_with_liquid: false
---

> 修修补补，添加标识
{: .prompt-info }

---

# mian
在ai时代，互联网的内容变得杂草丛生，越来越多重复内容的文章被ai源源不断的生成，中文网络的内容渐渐的变得越来越低质量，且由于各大社区渐渐的开始排斥搜索引擎的捕捉索引，某些社区更是对于被分享到社区外的内容进行限流和热度降级，所以其实开源的互联网生态变的越来越封闭。而个人博客这片自留地希望只是被ai帮助，而不是被占领。

我对自己输出的内容的添加真人创作的标识是为了让游客能更放松的方式进行游览评价。也希望这能拉近博主与他人之间的距离，
## 1.资源准备
1. 下载[静态资源](https://notbyai.fyi/help/#)

2. 将静态资源放到对应的目录下： assets/attachments/2024/对应的文件夹（英文名）
## 2.文件创建
1. 在 /_includes 文件夹下创建 notbyai-logo.html 文件
```html
<!-- _includes/notbyai-logo.html -->
<a href="https://notbyai.fyi/" >
    <img src="{{ site.url }}/assets/attachments/2024/notbyai/Chinese (CN)/Written By Human/Written-By-Human-Not-By-AI-Badge-white.png" alt="非ai创作" style="height: 30px; max-width: 100%;">
</a>
```
2. 在 _includes/footer.html 文件中`<footer>`标签内位置添加以下代码
```html
  <!-- 加非ai标识 -->
  {% include notbyai-logo-cn-write.html %}
```
## 3.测试（针对jykell框架）
1. 控制台执行 `bundle exec jekyll s`查看页面效果，调整logo尺寸
![img](assets/attachments/2024/blogs-notbyai/blogs-notbyai01.png)

完结(⌐■_■)



---
故事未完:353
**Thoughts**:: justdoit.
