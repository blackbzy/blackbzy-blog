---
title: blog_02_jykell主题模板配置
description: 模板配置
date: 2024-07-03
categories:
  - blog
tags:
  - blog
auther: blackbzy
update_date: false
pin: false
toc: true
comments: 
render_with_liquid: false
media_subpath: 
---

# mian
qr:metadata编辑对象格式数据目前不支持即，需要编辑完成再添加
```yml
image:
  path: /commons/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
```

## 1.metadata的含义：
```yaml
title: 笔记 # 标题 
tags: # 标签
  - blog
date: # 日期
update_date: false # 更新日期
categories: # 类型（归类汇总用）
  - blog
pin: true # 是否展示
toc: true # 是否展示右侧面板
math: false # 是否用到matrix
mermaid: false # 是否用到图表语法
render_with_liquid: # 是否用到render渲染语法
media_subpath: /posts/20180809 # 图片等资源的相对路径
image: # 封面图片
  path: /commons/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA # 模糊效果
  alt: Responsive rendering of Chirpy theme on multiple devices. # 悬浮展示

```

## 2.模板配置的策略
方便不同类型文章的输出。且适配jykell语法，能够正常的在页面渲染


tips:
logo生成：准备大小为 512x512 或更大的方形图片（PNG、JPG 或 SVG），然后转到在线工具 [**Real Favicon Generator**](https://realfavicongenerator.net/) 并单击 Select your Favicon image 按钮上传您的图片文件。

---
**Thoughts**:: 模板格式方便调试.
