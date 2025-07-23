---
title: blog_01_jykell博客搭建部署
description: 博客搭建
date: 2024-06-27
categories:
  - blog
tags:
  - blog
author: blackbzy
update_date: 2025-07-23
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
基于：[chirpy](https://chirpy.cotes.page/) Jekyll Theme,黑白主题，简洁，布局我比较中意
## 1.基于github fork创建
环境: **windows11** 
准备工作参考 [Jekyll Docs](https://jekyllrb.com/docs/installation/) 
开始：
- fork出自己的仓库 USERNAME.github.io
- 拉到本地跑cmd：
```sh
bundle
```

遇到报错（已挂节点）
解决方法：
```sh
Could not fetch specs from https://rubygems.org/ due to underlying error
<IO::TimeoutError: Failed to open TCP connection to rubygems.org:443 (Blocking
operation timed out!) 
```
国内外搜索都找不到合理的解决方案，还是gpt解决了问题：
```markdown
1. 检查防火墙
2. 设置代理
3. 更新ssl
4. 使用不同的 gem 源（使用该方法解决问题）
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
bundle config mirror.https://rubygems.org https://gems.ruby-china.com/
5. 更新 RubyGems
gem update --system
```
- Configuration 配置调整： 
文件_config.yml 根据注释自定义调整
难点一：
comments 评论配置：disqus、giscus国内访问困难，所以选择valine。初版不做太多自定义，目前不支持，所以先不做设置，今后参考[nihil博客](https://nihil.cc/)
- 运行项目：
```sh
# 先安装
bundle add webrick
bundle install
# 再启动
bundle exec jekyll s
```
就此启动成功

## 2.部署github action
如果您已将 `Gemfile.lock` 提交到存储库，并且您的本地计算机未运行 Linux，请转到站点的根目录并更新锁定文件的平台列表：

```
$ bundle lock --add-platform x86_64-linux
```

接下来根据配置github action ，使用默认Jekyll的部署action，然后自动部署成功。

访问页面报错
```
Failed to open TCP connection to rubygems.org:443
```
排查问题和询问gpt排查皆没用。
自己回想改的配置文件哪有问题，发现_config.yml 文件中的url属性应当为默认值，而我配置了github的域名导致访问失败
置空之后保存更新，启动，访问成功
```yml
# Fill in the protocol & hostname for your site.
# e.g. 'https://username.github.io', note that it does not end with a '/'.
url: ""
```

至此博客搭建完成。

## 3.后续维护博客时遇到的问题汇总
### 3.1 oss白嫖失败
博客主要面对的是国内的使用者，但是项目部署在vercel上，会出现加载慢的问题，但是使用国内的oss存在需要域名的问题，微软和aws的服务又不能稳定访问，也不支持长期的免费额度，暂时观望

### 3.2 页面加载刷白光
![](/assets/attachments/blog/error-white-flash.gif)
这个问题是我博客的一个浏览者提出的，后面好几次试图解决，但是没找到原因。

现在我找了个时间一个一个排查原因，现在发现是`/assets/js/data`目录下的js文件缺失导致。

缺失原因：Chirpy 的 JS 文件是通过构建工具（如 npm + webpack）编译出来的，它们不会直接出现在项目中，必须手动构建一次。我在项目初期构建过，但是随着原项目更新，我fork的项目再merge之后，没有及时更新项目依赖，导致`.js`文件依赖未及时更新。

解决方案：
1. [安装 Node.js](https://nodejs.org/zh-cn/download) 环境
2. 安装前端依赖，执行命令
```bash
npm install
```
如果存在以下报错,是node和npm版本不够，需要升级：
```bash
npm WARN EBADENGINE Unsupported engine { 
npm WARN EBADENGINE   package: 'stylelint-config-recommended-scss@15.0.1',
npm WARN EBADENGINE   required: { node: '>=20' },
npm WARN EBADENGINE   current: { node: 'v19.8.1', npm: '9.5.1' }
npm WARN EBADENGINE }   
```

- 下载 [nvm-windows](https://github.com/coreybutler/nvm-windows/releases) 安装程序，选择nvm-windows
- 安装完成后，重启终端（建议重启电脑以刷新环境变量），然后在命令行中运行：
```bash
nvm install 20.17.0
nvm use 20.17.0
node -v
npm -v
```

3. 在项目目录下执行以下命令，构建 JS 文件
```bash
npm install
npm run build
```
最后运行项目，看是否还有报错。

### 3.3项目`git commit`内容规范化
符合以下格式
```bash
git commit -m "feat: 初始化迁移"
```

| type       | 说明         |
| ---------- | ---------- |
| `feat`     | 添加功能       |
| `fix`      | 修复 bug     |
| `docs`     | 仅文档变更      |
| `style`    | 格式（空格、分号等） |
| `refactor` | 代码重构       |
| `test`     | 添加或修改测试    |
| `chore`    | 杂项（构建工具等）  |

### 3.4 页面右侧目录失效 ：即文章内容下的未渲染
![toc-error.png](/assets/attachments/blog/toc-error.png)
排查方式：
1. Chirpy 的 TOC 需要你在文章头部设置中启用`toc: true `
2. 文章是否包含 2 级或以上标题
3. 是否构建了 post.min.js，在以下目录：`assets/js/dist/post.min.js`
4. _config.yml 中是否禁用了 TOC
5. 控制台是否有 JS 报错:post.min.js
最后都没有问题，问题出现在。。。。我的文章的头部 YAML内容
```yaml
title: 科幻之书Ⅱ
description: 科幻的力量在于为现实生活找到可能的出口，可能性！！！
date: 2025-07-06
categories:
  - read
tags:
  - read
## 对的就是 auther 拼写错了，导致下面的toc属性无法识别，然后渲染不出来
auther: blackbzy
update_date: false
pin: false
toc: true
comments: true
image:
  path: assets/attachments/2025/science-fiction/science-fiction02.png
  alt: 封面
```
#### 3.4.1 后续vercel部署是`assets/js/dist/theme.min.js`文件404

| 检查项                                           | 状态 |
|-----------------------------------------------| -- |
| GitHub 仓库根目录包含 `assets/js/dist/theme.min.js`？ | ✅  |
| 文件名大小写匹配？                                     | ✅  |
| vercel 输出目录配置为 `.`？                           | ✅  |
| 不存在 `_site/` 子目录部署？                           | ✅  |
| 请求路径和文件路径完全一致？                                | ✅  |
| 访问 Vercel 默认子域名看看能不能打开 JS                     | ✅  |

最后检查出来是 域名 绑定vercel的问题，因为我的的是托管在cloudflare上，dns配置成代理的话，会导致CNAME 配置指向错误:即你现在的 DNS 设置阻止了它的 bot 和静态资源加速、JS 加载等机制

修改方式：在cloudflare上修改dns记录

| 主机记录（Host） | 记录类型  | 值                     |
| ---------- | ----- | --------------------- |
| @          | CNAME | cname.vercel-dns.com. |
| www        | CNAME | cname.vercel-dns.com. |


## 未完待续
- [x] [01_模板配置](/posts/blog-template)
- [x] [独立域名和服务器](/posts/blogs-domian-and-site)
- [ ] 双语切换（待定）参考[双语使用方式](https://aursus.github.io/hexo-bilingual)
- [x] [评论服务加上：waline](/posts/add-comments)
- [ ] 学习front_end的语言，自己进行theme的调整
- [ ] 静态资源的cdn和oss服务托管

---
故事未完:179
