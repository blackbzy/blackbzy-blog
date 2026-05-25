---
title: 博客迁移：从vercel到vps
description: 爬虫真的是毒瘤
date: 2026-03-04
categories:
  - blog
tags:
  - blog
auther: yourdomain
update_date: 2026-03-13
pin: false
toc: true
comments: true
---

> 怎么说呢，折腾花的时间太多了，应该及时收手
{: .prompt-info }
---
# mian
事情过程： 在今年2月底一次github的push后，想看下自己博客状态，结果输入网址发现网页提示`This deployment is temporarily paused`。怀疑是vercel挂了，于是登陆vercel发现主页usage模块提示`paused` 中的`Fast Data Transfer` 在1/12这一天用了680G，平时连100mb都难达到，这时候我意识到可能被攻击了，查看了一下访问的ip分布世界各地，包含各个语种的访问者，分析了一下可能是ai爬虫导致的流量泄露。后续在ai的帮助下和vercel的客服ai沟通，结果是反复车轱辘话回复，解决不了问题，发了邮件个vercel的support团队，收到回复，后面也不了了之，这里就只能放弃vercel了，尝试用自己的vps。

```text
申诉内容：
Hi Vercel Support, I'm reporting a massive traffic anomaly on my project **yourdomain-blog-pages**. On **January 17, 2026**, my site was hit by a DDoS attack that consumed over **600GB** of bandwidth in a single day, whereas my typical monthly usage is **less than 1GB**. This has caused my project to be suspended. I have already taken proactive steps to prevent future incidents by routing my domain through **Cloudflare (Proxied)** and enabling **"Under Attack Mode"** along with **additional WAF security rules** to filter out malicious requests. Could you please review this abnormal activity, waive the bandwidth overage caused by the attack, and reactivate my project? Thank you for your assistance.

```

可能的攻击方式：
- **恶意刷流量 (CC 攻击)：** 攻击者利用代理 IP 池，不断请求你站点上的大文件（如图片、JS 脚本），迅速消耗带宽。
- **爬虫“轰炸”：** 某些不怀好心的爬虫（或是配置错误的搜索引擎爬虫）在短时间内高频抓取你的所有页面。

## 1.补足漏洞
### cloudflare防护添加
![blog-attactered-move-to-vps02.png](2026/blog-attactered-move-to-vps/blog-attactered-move-to-vps02.png)
1. 开启“**小黄云**”（我之前是因为waline的js资源加载问题关闭了，是因为和vercel冲突）
>确保你在 CF 的 DNS 记录中，对应的域名 **代理状态（Proxy status）** 是开启的（即那个**橙色的小云朵**图标）。
>ps：如果云不开（仅dns），google的analytics是没有数据的
- **立即生效的操作：** 在 CF 后台点击左侧的 **安全性 (Security) -> 仪表板 (Dashboard)**。
- 如果看到流量曲线依然陡峭，点击右侧的 **“正在受到攻击？(Under Attack Mode)”**。这会强制所有访问者通过 JavaScript 挑战，能瞬间挡住 95% 以上的恶意脚本。

2. 配置 WAF 频率限制（防刷核心）
>在 **安全性 (Security) -> WAF -> 速率限制规则 (Rate limiting rules)** 中创建规则：
- 规则名称： 防刷流量。
- 匹配条件： 所有传入请求。
- 速率限制： 设置 10 秒内超过 10 个请求。
- 操作： 选择 “阻止”。

3.  封锁特定地区或异常来源
>暂时拦截非中文地区的 IP：
- 在 **WAF -> 自定义规则 (Custom rules)** 中，设置：
  - 匹配： `Country` 不等于 `China` (包括港澳台)。
  - 操作：JS 质询。
- 这样真实用户点一下验证码就能进，但自动化攻击脚本会卡死在这里。

4.  阻止 AI 爬虫程序和爬网程序规则
>cloudflare的自带的爬虫限制规则，能放入正常的google等服务商的爬虫，限制非自然流量

5. 检查缓存命中率 (Cache Hit Ratio)
>流量消耗大，说明请求都透传到了 Vercel。
- 在 CF 的 **缓存 (Caching) -> 配置** 中，将 **浏览器缓存过期时间** 设置得长一点。
- 确保 **开发模式** 是关闭的。

6.  静态资源跳过限制
>配合后续的oss服务使用，因为静态文件托管在CF的R2服务器上，所以博客的静态资源子域名放行，让CF去处理流量问题，因为免费额度是没有流量限制的
- 在 **WAF -> 自定义规则 (Custom rules)** 中，设置：
  - 匹配： `主机名` 等于 `静态资源域名` 。
  - 操作：跳过。 下方所有的组件勾选
## 2.vps部署blog

### 2.1.大文件oss管理
考量到vps部署存在流量带宽的限制，而且容易被访问超过流量限制导致vps服务像vercel一样挂掉，所以使用oss服务就很合适而且方便部署，推送的静态资源大小更可控，部署的方式更自由，不然1g的静态文件对于github并不友好，性能会大幅降低。
而且对于图片（二进制大文件）较多的博客，直接用 `rsync` 或 `ssh-deploy` 这种文件同步方式，由于 GitHub Actions 每次运行都是在一个“全新的、干净的”虚拟环境里，GitHub确实需要把所有图片重新扫描、对比甚至重新传输，效率会随着图片增多而下降。

OSS服务商选择Cloudflare R2？
- 域名管理方便：既然我的域名是CF管理，那么用CF的Oss也更方便我分配子域名。
- 完全免费额度大： R2 每月有 10GB 的免费存储额度，且 流量（带宽）完全免费。这意味着即便下次有人再刷你 600GB 图片流量，你也不用付一分钱，更不会被停机。
- 全球加速： 图片直接从 CF 边缘节点加载，速度比你的 VPS 快得多。

迁移步骤：
1. 在 Cloudflare 后台开启 **R2**，创建一个 Bucket
2. 将你本地 `对应需要上传的静态资源全目录资源` 上传到 R2。
3. 在 R2 绑定一个自定义域名（比如 `img.yourdomain.com`）。
4. 修改 Chirpy 引用方式，使用 Chirpy 的 `img_cdn` 配置： 在 `_config.yml` 中设置：

```yaml
cdn: https://img.yourdomain.com

```

5. 需要把原来的文档中的图片引用 `![img](/图片路径)` 替换为`![img](OSS的相对图片路径)`
6. 原来的静态资源文件夹添加到`.gitignore`

![blog-attactered-move-to-vps01.png](2026/blog-attactered-move-to-vps/blog-attactered-move-to-vps01.png)
到这里OSS部分是迁移完成了

### 2.2vps服务器配置环境
ps：服务器系统不同，命令工具也不一定相同，用自己服务器有的指令就行
服务器参数：
- 1核1g内存20g硬盘
- Operating system: **Rocky 9 x86_64**

#### 2.2.1安装基础工具

```bash
# 安装基础工具
sudo yum update -y sudo 
yum install git nginx -y
sudo apt update
# 创建并授权目录
sudo mkdir -p /var/www/blog
sudo chown -R $USER:$USER /var/www/blog

```

#### 2.2.2. 配置Nginx
首先需要在 VPS 上安装 **SSL 证书**，来确保从 Cloudflare 到你 VPS 之间的数据也是加密的。

1. 在 Cloudflare 后台生成证书
  1. 登录 Cloudflare，选择你的域名 `yourdomain.com`。
  2. 点击左侧菜单 **SSL/TLS -> 源服务器 (Origin Server)**。
  3. 点击 **创建证书 (Create Certificate)**。
  4. 保持默认设置（包含 `yourdomain.com` 和 `*.yourdomain.com`），有效期限通常选 15 年。
  5. 点击 **创建** 后，你会看到两个文本框：
    - **源证书 (Origin Certificate)**：这就是你的公钥。
    - **私钥 (Private Key)**：这是你的私钥（**注意：关闭窗口后就看不到了，请立刻复制**）。
2. 在 VPS 上存放证书
  1. 回到你的 VPS 终端，创建存放证书的目录并保存文件：

```bash
    # 创建目录
    sudo mkdir -p /etc/nginx/ssl
    # 保存证书文件
	# 使用 `vi` 或 `nano` 创建证书文件，并将 Cloudflare 页面上的 **源证书** 内容粘贴进去：
    sudo vi /etc/nginx/ssl/blog.pem
    # **保存私钥文件：** 创建私钥文件，并将 Cloudflare 页面上的 **私钥** 内容粘贴进去：
    sudo vi /etc/nginx/ssl/blog.key

```

3. 编辑blog服务的nginx的配置文件

```bash
sudo vi /etc/nginx/conf.d/blog.conf

```

(按 `i` 进入输入模式，粘贴下面的内容，按 `Esc` 后输入 `:wq` 保存退出)
以下的证书目录和上面的是对应的，如要修改请同步：


```nginx
# 80 端口只负责重定向
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;# yourdomain.com 换成你的域名
    return 301 https://$host$request_uri;
}

# 443 端口负责处理业务
server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # 证书路径
    ssl_certificate     /etc/nginx/ssl/blog.pem;
    ssl_certificate_key /etc/nginx/ssl/blog.key;

    # SSL 性能与安全优化
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    root /var/www/blog;
    index index.html;

    location / {
        error_page 405 =200 $uri;
        try_files $uri $uri/ /index.html =404;
	      # 在 server 或 location / 块中添加
	      proxy_buffer_size 128k;
	      proxy_buffers 4 256k;
	      proxy_busy_buffers_size 256k;
	       #禁用响应缓冲防止截断
        proxy_buffering off;
    }
    
    location /assets/img/favicons/ {
      alias /var/www/blog/assets/img/favicons/;
       # 1. 强制声明类型，防止 Nginx 因为不认识后缀而报 403 或下载
       types {
          application/manifest+json webmanifest;
          }
          default_type application/manifest+json;
          # 2. 允许所有请求
          allow all;
          satisfy any; # 如果有全局认证，这一行能跳过

          # 3. 跨域头（PWA 必须）
          add_header Access-Control-Allow-Origin *;
           add_header Cache-Control "no-cache"; # 调试期间防止被 Service Worker 缓存坑
    }
    
    location = /assets/img/favicons/site.webmanifest {
      allow all;
      types {
          application/manifest+json webmanifest;
        }
      # 确保 alias 路径绝对正确
      alias /var/www/blog/assets/img/favicons/site.webmanifest;
    }

}

```

nginx需要配合CF做域名解析才能完成跳转：
- 登录 Cloudflare，进入你的域名。
- **DNS -> 记录 (Records)**。
- 点击 添加记录 (Add record)：
  - 类型 (Type): `A`
  - 名称 (Name): `@` (代表主域名) 或 `www`
  - IPv4 地址: 填入你的 VPS 公网 IP。
  - 代理状态 (Proxy status): 务必开启“小黄云”。
- **把vercel相关的dns记录删掉**，否则会影响后续的域名解析
- 配置加密模式 (SSL/TLS) -> **完全 (Full)**


```bash
# **保存后重启 Nginx：**
sudo nginx -t  # 检查有没有语法错误
sudo systemctl restart nginx

```

nginx重启后访问域名看是否是403：代表从域名到服务器的链路通了

需要开放防火墙的 443 端口：

```bash
# 检查防火墙是否开启
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

```

>我在这步遇到端口占用的问题

```bash
# 排查路径，看错误日志排查配置文件和端口
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[root@safe-powder-1 ~]# sudo systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[root@safe-powder-1 ~]# sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@safe-powder-1 ~]# sudo ss -tulpn | grep -E '80|443'
udp   UNCONN 0      0                  *:443              *:*    users:(("sui",pid=64262,fd=13))
tcp   LISTEN 0      4096               *:443              *:*    users:(("sui",pid=64262,fd=11))

```

把另一个服务的端口修改就没有问题了
ps：s-ui服务的入站端口不一定非要443，可以用其他代替，只是隐蔽性不强了，修改完端口tls的密钥（这里端口不用改，踩了小坑，改了就连不了了，这是入口服务强制的端口校验）需要重新生成才能起效。

### 2.3 配置github
**window系统**操作方式：其他系统自行修改命令
只剩最后一步：**把 GitHub 上的代码自动“推”到你的 VPS 上。**
使用 **GitHub Actions** 配合 **SSH** 来完成这个自动化闭环。

1.在 GitHub 仓库配置“钥匙” (Secrets)
GitHub Actions 需要权限才能登录你的 VPS。为了安全，不要在脚本里写密码。

```bash
    # 以下命令`$HOME`需改为你windows的 C:\Users\用户名
    # 在你的本地电脑（或 VPS）生成一对 SSH 密钥
    ssh-keygen -t rsa -b 4096 -f $HOME\.ssh\github_deploy_key
    #`C:\Users\你的用户名\.ssh\` 文件夹下会有两个文件，
    # 手动复制的方式添加白名单
    # 公钥
    cat $HOME\.ssh\github_deploy_key.pub
    # 私钥
    cat $HOME\.ssh\github_deploy_key
    

```

2.在 VPS 上操作（通过当前的 SSH 连接）

```bash
# 确保 .ssh 目录存在
mkdir -p ~/.ssh
# 编辑白名单文件，粘入公钥，保存（i -> esc -> :wq）
vi ~/.ssh/authorized_keys

```

3.在 GitHub 填写 Secret
1. 进入 GitHub 仓库的 **Settings -> Secrets and variables -> Actions**：
1. **新建 `SSH_PRIVATE_KEY`：** 填入**私钥**内容
2. **新建 `REMOTE_HOST`：** 填入你的 VPS 公网 IP。
3. **新建 `REMOTE_USER`：** 填入 `root`。

### 2.4. 修改blog项目（chipry主题）
在项目`.github/workflows/`下，新建文件 `deploy.yml`

```yml
name: Deploy Chirpy to VPS  
  
on:  
  push:  
    branches: [ blog ]  
  
jobs:  
  build-and-deploy:  
    runs-on: ubuntu-latest  
  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v4  
        with:  
          fetch-depth: 0  
          submodules: true # 确保拉取了 assets/lib 子模块  
  
      - name: Setup Ruby  
        uses: ruby/setup-ruby@v1  
        with:  
          ruby-version: '3.3'  
          bundler-cache: true  
  
      - name: Setup Node  
        uses: actions/setup-node@v4  
        with:  
          node-version: '20' # Chirpy 建议使用 v20 或更高  
  
      - name: Build Frontend Assets  
        run: |  
          npm install  
          # 这一步会生成编译 CSS 所需的临时文件和 vendors 路径  
          npm run build  
  
      - name: Build Jekyll Site  
        run: |  
          # 使用 --future 确保 2026 年的文章能被渲染  
          bundle exec jekyll build --future  
  
          # 【调试命令】如果这里还是 0.7s，看这一步输出  
          echo "Checking generated posts..."  
          find _site/posts -name "*.html" | head -n 5  
        env:  
          JEKYLL_ENV: production  
  
      - name: Clean and Copy to VPS  
        uses: appleboy/ssh-action@master  
        with:  
          host: ${{ secrets.REMOTE_HOST }}  
          username: ${{ secrets.REMOTE_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          script: |  
            rm -rf /var/www/blog/*  
  
      - name: Deploy to VPS  
        uses: appleboy/scp-action@master  
        with:  
          host: ${{ secrets.REMOTE_HOST }}  
          username: ${{ secrets.REMOTE_USER }}  
          key: ${{ secrets.SSH_PRIVATE_KEY }}  
          source: "_site/*"  
          target: "/var/www/blog"  
          strip_components: 1

```

推送到 GitHub 触发部署
在本地执行：

```bash
git add .
git commit -m "Add GitHub Actions for VPS deployment"
git push 

```

- 查看一下github仓库的action日志，看静态文件是否正常推送。
- 回到你的 VPS 终端


```bash
# 查看文件是否已推送
ls /var/www/blog

```

- 网页访问博客看博客是否已经能看到主页

## 3.waline迁移（目前废案）
其实本地已经没问题了，但是vps端可能是nginx或者CF的限制，导致js加载一直有问题，没有办法具体锁定问题的位置，所以评论模块在线上的版本一直不展示，所以以前的评论可能要被暂时丢弃了，后面再想办法迁移。
如果是使用国内服务器的版本应该是没有我这样的问题的，可以照抄。

#### 3.1.服务部署
在 VPS 上使用 Docker 部署
**LeanCloud 数据库要下线了**，所以切到自己部署的mysql。
1. 创建一个 `docker-compose.yml`：

```yml
services:
  waline:
    image: lizheming/waline:latest
    container_name: waline-app
    restart: always
    ports:
      - "8450:8450"
    depends_on:
      - db
    environment:
      # --- 数据库配置 ---
      - MYSQL_HOST=db
      - MYSQL_PORT=3306
      - MYSQL_DB=waline
      - MYSQL_USER=waline_user
      - MYSQL_PASSWORD=mysqlpassword
      # --- 基础设置 ---
      - JWT_SECRET=0Nqxvu2Nxy3jARPU5g9muf95
      - SITE_NAME=大湿的菜园子
      - SITE_URL=https://www.yourdomain.com
      - AUTHOR_EMAIL=你的邮箱
      - SERVERURL=https://comments.yourdomain.com
      # --- SMTP 邮件提醒 ---
      - SMTP_SERVICE=163
      - SMTP_USER=你的邮箱
      - SMTP_PASS=邮箱smtp码
      - SMTP_SECURE=true
      - SMTP_PORT=465

  db:
    image: mysql:5.7
    # 确保 command 与 image 对齐，且前面只有空格没有 Tab
    command: --default-authentication-plugin=mysql_native_password
    container_name: waline-db
    restart: always
    environment:
      - MYSQL_DATABASE=waline
      - MYSQL_USER=waline_user
      - MYSQL_PASSWORD=mysqlpassword
      - MYSQL_ROOT_PASSWORD=mysqlpassword
    volumes:
      - ./mysql_data:/var/lib/mysql

```

2. 链接VPS 终端：

```bash
# 1. 创建目录 
mkdir -p /opt/waline 
# 2. 进入目录 
cd /opt/waline 
# 3. 创建并编辑文件 ，粘贴上面的docker-compose.yml内容
vim docker-compose.yml

```

3. vps端安装基础工具，如果已经有docker工具，忽视这条


```bash
# 移除旧版本（如果有）
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-engine
           
# 安装工具包
yum install -y yum-utils

# 添加 Docker 官方仓库 (使用阿里云镜像加速)
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装 Docker 引擎和 Compose 插件
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker 
# 验证安装是否成功 
docker --version 
docker compose version
# 再次运行你的 Waline
cd /opt/waline
docker compose up -d
# 查看日志确认是否正常启动
docker logs waline-app
docker logs -f waline-app

```

####  3.2. 数据迁移
mysql启动好之后，接下来进入进入 MySQL 容器：


```bash
docker exec -it waline-db mysql -uwaline_user -pmysqlpassword waline

```

执行表sql[官方的sql](https://waline.js.org/guide/database.html#mongodb)，我选择的[mysql ](https://github.com/walinejs/waline/blob/main/assets/waline.sql)
执行exit命令退出。


```bash
exit
# 重启 Waline
docker compose restart waline

```

5. 导入数据 先从leancloud下载数据。
   ![blog-attactered-move-to-vps01.png](2026/blog-attactered-move-to-vps/blog-attactered-move-to-vps03.png)
   再把数据转换
- leancloud导出数据转换python脚本：

```py
import json

# 读取 LeanCloud 导出的原始文件
with open('Comment_20260305_135514.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

# 适配你的表结构字段：createdAt, updatedAt, insertedAt, sticky
sql_template = "INSERT INTO `wl_Comment` (`url`, `nick`, `mail`, `link`, `ua`, `ip`, `comment`, `status`, `sticky`, `createdAt`, `updatedAt`, `insertedAt`) VALUES \n"
values = []

for item in data:
    # 适配 boolean 类型的 sticky
    sticky_val = "1" if item.get('sticky') else "0"
    
    # 转义单引号防止 SQL 报错
    def escape(val):
        return str(val).replace("'", "''") if val else ""

    val = "('{url}', '{nick}', '{mail}', '{link}', '{ua}', '{ip}', '{comment}', '{status}', {sticky}, '{createdAt}', '{updatedAt}', '{insertedAt}')".format(
        url=escape(item.get('url', '')),
        nick=escape(item.get('nick', '匿名')),
        mail=escape(item.get('mail', '')),
        link=escape(item.get('link', '')),
        ua=escape(item.get('ua', '')),
        ip=escape(item.get('ip', '')),
        comment=escape(item.get('comment', '')),
        status=escape(item.get('status', 'approved')),
        sticky=sticky_val,
        # 转换时间格式，去掉 T 和 Z
        createdAt=item['createdAt'].replace('T', ' ').replace('Z', ''),
        updatedAt=item['updatedAt'].replace('T', ' ').replace('Z', ''),
        insertedAt=item.get('insertedAt', item['createdAt']).replace('T', ' ').replace('Z', '')
    )
    values.append(val)

with open('import_comments.sql', 'w', encoding='utf-8') as f:
    f.write(sql_template + ",\n".join(values) + ";")

print("转换完成！生成了适配你表结构的 import_comments.sql")

```

获得import_comments.sql脚本


```bash
# 清除旧数据
docker exec -it waline-db mysql -uwaline_user -pmysqlpassword waline -e "TRUNCATE TABLE wl_Comment;"
# 检查并修正 SQL 文件编码
file -i import_comments.sql
# 应该输出: import_comments.sql: text/plain; charset=utf-8
# 生成 `import_comments.sql` 后，将其上传到 VPS
docker cp import_comments.sql waline-db:/tmp/
# 强制以 utf8mb4 编码执行导入 
docker exec -it waline-db mysql -uwaline_user -pmysqlpassword waline \ --default-character-set=utf8mb4 \ -e "SET NAMES utf8mb4; SOURCE /tmp/import_comments.sql;"

```


至此数据迁移完成
#### 3.2.配置waline的nginx配置
nginx的waline配置文件waline.conf，放到


```xml
# 频率限制：定义一个 10MB 的区域，每秒只允许 5 个请求
limit_req_zone $binary_remote_addr zone=waline_limit:10m rate=5r/s;

server {
    listen 80;
    server_name comments.yourdomain.com;

    # 所有的 HTTP 请求重定向到 HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name comments.yourdomain.com;

      # 证书路径
    ssl_certificate     /etc/nginx/ssl/blog.pem;
    ssl_certificate_key /etc/nginx/ssl/blog.key;

    # SSL 性能与安全优化
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    location / {
        # 应用频率限制
        limit_req zone=waline_limit burst=10 nodelay;

        proxy_pass http://127.0.0.1:8450;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 解决 WebSocket 问题（Waline 实时预览需要）
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

```


在vps终端执行：


```bash
sudo nginx -s reload

```


nginx重启之后访问博客，页面应该正常展示了。
可惜我只能本地展示。。。
![blog-attactered-move-to-vps01.png](2026/blog-attactered-move-to-vps/blog-attactered-move-to-vps04.png)
接下来准备切换评论组件，增加邮件发送的功能，现在就先这样紫啦。


## 4. 博客评论重新配置为giscus

### 4.1.仓库处理
#### 4.1.1 安装giscus
- **访问 GitHub App 页面**： 直接打开 [github.com/apps/giscus](https://github.com/apps/giscus)。
- **点击安装 (Install)**： 点击页面上的 **Install** 按钮。
- **选择仓库范围**：
  - **All repositories**：授权给你的所有仓库（省事，但权限给得多）。
  - **Only select repositories**（**推荐**）：在下拉列表里只勾选你那个 **Chirpy 博客的仓库**。作为后端开发，我们通常遵循“最小权限原则”。
- **确认授权**： 点击底部的 **Install & Authorize**。

#### 4.1.2 仓库配置
1. 进入你存放博客源码的 **GitHub 公共仓库**。
2. 点击 **Settings** -> **General**。
3. 在 **Features** 栏目下，勾选 **Discussions**。

### 4.2.获取配置参数
1. 访问 [giscus.app](https://giscus.app/zh-CN)。
2. 输入你的 `仓库名称` (例如：`yourname/yourname.github.io`)。
3. 配置映射关系（推荐选“Discussions 标题包含页面路径”）。
4. 分类选 **Announcements**。
5. 页面下方会生成一段脚本，记录其中的 `data-repo-id` 和 `data-category-id`。
6. 修改 `_config.yml` 中的 `comments` 部分：

```yaml
comments:
  active: giscus # 指定使用 giscus
  giscus:
    repo: 'yourname/yourname.github.io'
    repo_id: '你的repo_id'
    category: 'Announcements'
    category_id: '你的category_id'
    mapping: 'pathname'
    input_position: 'bottom'
    lang: 'zh-CN' # 设置语言
    
``` 

接下来重启新推送项目即可。
![blog-attactered-move-to-vps01.png](2026/blog-attactered-move-to-vps/blog-attactered-move-to-vps05.png)

[waline在国内依然适用]({% post_url 2024/2024-08-03-add-comments %})

### 4.3. 如何注册 GitHub 账号

1. 打开 [GitHub 官网 (github.com)](https://github.com/)，点击右上角的 **Sign up** 按钮。
  1. 如果无法打开，可能是网络问题，可以选择下载工具[watt Toolkit](https://steampp.net/),然后选择其中的网络加速，选择github，点击开始加速
2. 输入注册信息。。。接下来就是按照操作引导一步步注册就行：欢迎来到开源世界(╹ڡ╹ )
3. 完成真人验证 (CAPTCHA)+邮箱验证

ps: GitHub 强制开启了 **2FA (双重身份验证)**。注册后可能需要绑定一个手机号或身份验证器（如 Microsoft Authenticator），否则账号可能会被限制。


---
故事未完:63
**Thoughts**:: justdoit.
