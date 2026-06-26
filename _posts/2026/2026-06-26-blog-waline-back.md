---
title: waline评论区回归
description: 一言难尽，折腾半天被一个小问题卡住
date: 2026-06-26
categories:
  - blog
tags:
  - blog
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
---
> 外国的开源产品在项目的架构文件中少用中文。。
{: .prompt-info }

# mian
先说下原因：
1. 基于github 的 discus出于墙的原因，国内访问很不稳定，对于没有工具的朋友来说很不友好，而且数据放在仓库的- Discussions，很难在后期迁移数据
2. discus需要有github账号才能评论，我的博客大概率也不是程序员看。。。
所以思虑良久还是得切回[waline](https://waline.js.org/)，那么之前的评论刚好我也备份了，可以同步回来。之前放弃是因为通过vercel部署的没有问题，可以正常访问，不过博客被攻击之后，vercel欠费了，基本是瘫痪了，只能部署到自己的vps服务器上，所以通过github action 部署不出意外是出意外了，一直解决不了评论加载不出来，显示那个html页面异常。

那之前为啥切别的discus呢，[那就是这个故事了]({% post_url 2026/blog-attactered-move-to-vps %}) 

现在重新走一遍流程，看看是啥问题，基础的ngnix搭建和配置不再赘述，已经在被攻击的[那就是这个故事了]({% post_url 2026/blog-attactered-move-to-vps %}) 里面讲了，这里只针对waline+mysql的vps部署。

## 部署-基于docker
ps：服务器系统不同，命令工具也不一定相同，用自己服务器有的指令就行
服务器参数：
- 1核1g内存20g硬盘
- Operating system: **Rocky 9 x86_64**

docker-compose文件：包含了waline服务和mysql库
这里的配置需要把你的域名代替domain部分，mysql的密码也是要改成自己的。

```

version: "3.8"

services:
  waline:
    image: lizheming/waline:latest
    container_name: waline

    restart: unless-stopped

    depends_on:
      - mysql

    ports:
      - "8360:8360"

    environment:
      TZ: Asia/Shanghai

      # 数据库
      MYSQL_HOST: mysql
      MYSQL_PORT: 3306
      MYSQL_DB: waline
      MYSQL_USER: waline_user
      MYSQL_PASSWORD: MYSQL_ROOT_PASSWORD

      # Waline
      JWT_SECRET: 123xv123xy3jARPU5g9mu123

      SITE_NAME: 大湿的菜园子
      SITE_URL: https://www.doman.com
      SERVER_URL: https://comments.doman.com

      # ⭐ 新增下面这行：允许跨域的域名白名单（不要带 http/https，用英文逗号隔开）
      ALLOWED_DOMAINS: "*"
      
      # 邮件通知
      SMTP_SERVICE: 163
      SMTP_USER: doman@163.com
      SMTP_PASS: password

      AUTHOR_EMAIL: doman@163.com

      # 评论审核（可选）
      COMMENT_AUDIT: false

    volumes:
      - ./waline-data:/app/data

  mysql:
    image: mysql:8.0
    container_name: waline-mysql

    restart: unless-stopped

    command:
      --default-authentication-plugin=mysql_native_password

    environment:
      MYSQL_ROOT_PASSWORD: MYSQL_ROOT_PASSWORD

      MYSQL_DATABASE: waline

      MYSQL_USER: waline_user
      MYSQL_PASSWORD: MYSQL_ROOT_PASSWORD

    volumes:
      - ./mysql-data:/var/lib/mysql

    ports:
      - "3306:3306"

```


```shell

# 完全重启
docker compose down
docker compose up -d
# 查看是否重启成功
docker ps
docker logs -f waline
docker logs -f waline-db
#补充命令：重启单个服务
docker compose restart waline
docker restart waline-app

```

## 配置waline的nginx
上面的服务启动成功，接下来需要配置访问，域名依旧是在cloudflare上加一个配置就行


waline.conf 文件：处理接口跳转和跨域的一些问题

```

limit_req_zone $binary_remote_addr zone=waline_limit:10m rate=10r/s;

server {
    listen 80;
    server_name comments.doman.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name comments.doman.com;

    ssl_certificate     /etc/nginx/ssl/comments/comments.pem;
    ssl_certificate_key /etc/nginx/ssl/comments/comments.key;

    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # ===== API（不限制太狠）=====
    location /api/ {
       # 1. 拦截浏览器的 OPTIONS 预检请求，直接在 Nginx 层返回 204 成功，不扔给后端
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '$http_origin' always;
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE' always;
            add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,Accept,Origin' always;
            add_header 'Access-Control-Allow-Credentials' 'true' always;
            add_header 'Content-Length' 0;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            return 204;
        }

        # 2. 清除后端可能自带的、残缺的跨域头，防止浏览器报“重复/冲突”错误
        proxy_hide_header 'Access-Control-Allow-Origin';
        proxy_hide_header 'Access-Control-Allow-Credentials';
        proxy_hide_header 'Access-Control-Allow-Methods';
        proxy_hide_header 'Access-Control-Allow-Headers';

        # 3. 强行注入统一的跨域头（$http_origin 动态匹配来源，比 * 更安全且支持 Credentials）
        add_header 'Access-Control-Allow-Origin' '$http_origin' always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE' always;
        add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,Accept,Origin' always;


        # 针对原本的限流，如果嫌烦可以先注释掉它
        # limit_req zone=waline_limit burst=20 nodelay;

        proxy_pass http://127.0.0.1:8360;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # ===== 前端资源（不做限流）=====
    location / {
        
        proxy_pass http://127.0.0.1:8360;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

重启ngnix

```shell
sudo nginx -t          # 检查语法是否正确
sudo nginx -s reload   # 平滑重启生效

```

## 数据库处理
### 执行建表
执行下表sql[官方的sql](https://waline.js.org/guide/database.html#mongodb)，我选择的[mysql ](https://github.com/walinejs/waline/blob/main/assets/waline.sql)

mysql启动好之后，接下来进入进入 MySQL 容器：

```Bash
docker exec -it waline-mysql mysql -uwaline_user -pMYSQL_ROOT_PASSWORD waline
USE waline;
SHOW TABLES LIKE '%comment%';
SELECT * FROM wl_Comment;

```

执行完成，敲`exit`命令退出。

```Bash
exit
# 重启 Waline
docker compose restart waline

```

### 迁移数据 
如果你之前没有存量的评论数据没必要走这一步。
sql转换交给了gpt，把要执行的数据放到 `import_comments.sql` 文件，上传到docker-compose同一目录，执行以下命令，将数据导入数据库

```shell
# 检查并修正 SQL 文件编码
file -i import_comments.sql
# 应该输出: import_comments.sql: text/plain; charset=utf-8
# 生成 `import_comments.sql` 后，将其上传到 VPS
docker exec -i waline-mysql mysql -uwaline_user -pMYSQL_ROOT_PASSWORD --default-character-set=utf8mb4 waline < import_comments.sql
```

## 处理waline相关问题
### 每一个域名或者说网站都需要证书，这里是我评论站的证书获取方式

```sh
#安装 EPEL 源（CentOS 需要这个源来下载 certbot）
sudo dnf install epel-release -y
# 如果提示 dnf command not found，说明系统比较老（如 CentOS 7），请换用 yum：
# sudo yum install epel-release -y

#安装 Certbot 及其 Nginx 插件
sudo dnf install certbot python3-certbot-nginx -y
# 老系统请用：
# sudo yum install certbot python3-certbot-nginx -y

#运行 Certbot 自动申请并配置证书
sudo certbot --nginx -d comments.doman.com

#VPS 系统是最新的 **CentOS Stream 9、Rocky Linux 9 或 AlmaLinux 9**。在大版本 9 的系统里，官方和 EPEL 源已经彻底废弃了通过 `dnf/yum` 直接安装 Certbot 的方式，而是全面转向了 **acme.sh** 脚本或者 **Snapd**
#终极方案：使用 `acme.sh` 申请证书
curl https://get.acme.sh | sh -s email=domain@domain.com
source ~/.bashrc
#让 acme.sh 自动读取 Nginx 配置并申请证书
acme.sh --issue -d comments.doman.com --nginx
#**如果这一步报错提示类似 "nginx command not found"：** 说明你的 Nginx 路径不在环境变量里。别慌，直接用**独立 Web 模式**验证（申请前需要先**临时关闭 Nginx**）：
systemctl stop nginx
acme.sh --issue -d comments.doman.com --standalone
systemctl start nginx
#证书申请成功后，我们需要把它拷贝到 Nginx 能够读取的独立目录下。 我们直接把证书放到一个新的统一目录（比如 `/etc/nginx/ssl/comments/`）下，这样绝对不会弄乱你原有的博客证书
# 创建一个新的证书存放目录
mkdir -p /etc/nginx/ssl/comments/

# 安装并拷贝证书，同时指定更新后自动重启 Nginx
acme.sh --install-cert -d comments.doman.com \
--key-file       /etc/nginx/ssl/comments/comments.key  \
--fullchain-file /etc/nginx/ssl/comments/comments.pem \
--reloadcmd     "systemctl force-reload nginx"

#修改你的 Nginx 评论配置
# ssl_certificate     /etc/nginx/ssl/blog.pem;     <-- 这是老的，删掉或注释
    # ssl_certificate_key /etc/nginx/ssl/blog.key;    <-- 这是老的，删掉或注释

    # ===== 换成下面这两行刚刚申请好的新证书 =====
    ssl_certificate     /etc/nginx/ssl/comments/comments.pem;
    ssl_certificate_key /etc/nginx/ssl/comments/comments.key;


systemctl restart nginx
# 如果ngnix启动状态下安装失败，则按照下面的命令重新安装下

#独立模式重新申请与安装 临时停掉 Nginx（（让出 80 端口给 acme.sh 验证）
systemctl stop nginx
#强行重新申请证书
acme.sh --issue -d comments.doman.com --standalone --force
systemctl start nginx

#正确安装并拷贝证书文件
acme.sh --install-cert -d comments.doman.com \
--key-file       /etc/nginx/ssl/comments/comments.key  \
--fullchain-file /etc/nginx/ssl/comments/comments.pem \
--reloadcmd     "systemctl force-reload nginx"

#检查 Nginx 虚拟主机配置
ssl_certificate /etc/nginx/ssl/comments/comments.pem; ssl_certificate_key /etc/nginx/ssl/comments/comments.key;

```

### 在chipry中引入
这部分在[添加waline]({% post_url 2024-08-03-add-comments %}) 已经写了，不再赘述，这里主要是解决之前的问题：
浏览器报以下错，导致评论无法加载：

```sh
book-youmingxiantu/:1  GET https://www.doman.com/Human.png 404 (Not Found)
book-youmingxiantu/:1 Uncaught SyntaxError: Unexpected end of input (at book-youmingxiantu/:1:22139)

```

不是 Nginx 的问题，也不是 Cloudflare 的问题，而是 VPS 磁盘上的这个网页文件，本身就是一个天生的“半成品”， 它的物理内容在写到 `</script>` 时就戛然而止了，后面彻底空了，难怪线上无论怎么折腾网络层，浏览器拿到的永远是残缺的 22139 个字符。经过排查发现截断的地方刚好是中文注释处，也就是说，又是中文搞的鬼。。。。其实最开始折腾这个博客的时候，本地无法启动，也是因为文件名称有中文，或者是metadata中有中文。
以下是调整过的`waline.html`

```html
<script type="module">  
  (function () {  
    const walineServerURL = 'https://comments.doman.com';  
  
    if (!document.getElementById('waline-style')) {  
      const link = document.createElement('link');  
      link.id = 'waline-style';  
      link.rel = 'stylesheet';  
      link.href = 'https://unpkg.com/@waline/client@v3/dist/waline.css';  
      document.head.appendChild(link);  
    }  
  
    const initWaline = () => {  
      if (document.getElementById('waline')) return;  
  
      const walineDiv = document.createElement('div');  
      walineDiv.id = 'waline';  
      walineDiv.style.marginTop = '2rem';  
  
      const $anchor = document.querySelector('.post-tail-wrapper') || document.querySelector('footer');  
      if ($anchor) {  
        $anchor.insertAdjacentElement('beforebegin', walineDiv);  
      } else {  
        document.querySelector('main')?.appendChild(walineDiv);  
      }  
  
      import('https://unpkg.com/@waline/client@v3/dist/waline.js')  
        .then((module) => {  
          const {init, pageviewCount} = module;  
  
          if (typeof init === 'function') {  
            init({  
              el: '#waline',  
              serverURL: walineServerURL,  
              dark: 'html[data-mode="dark"]',  
              reaction: true,  
              pageview: true,  
              emoji: [  
                '//unpkg.com/@waline/emojis@1.4.0/weibo',  
                '//unpkg.com/@waline/emojis@1.4.0/bmoji'  
              ]  
            });  
          }  
        })  
        .catch(err => {  
          /* 块级注释在压缩时是绝对安全的，不会误伤后续代码 */          
          console.error('Waline 模块加载失败:', err);  
        });  
    };  
  
    if (document.readyState === 'loading') {  
      document.addEventListener('DOMContentLoaded', initWaline);  
    } else {  
      initWaline();  
    }  
  })();  
</script>

```

同步要在配置文件`_config.yml`中调整

```yml
comments:  
  # Global switch for the post-comment system. Keeping it empty means disabled.  
	provider: waline # [disqus | utterances | giscus]  
	  # The provider options are as follows:
	waline:  
	  serverURL: 'https://comments.xxx.com/' # 必須是你 VPS 的新地址，注意帶上 https://  # --- 重要：清理舊配置 ---  # 如果下面有 appID 或 appKey，請務必刪除或留空，MySQL 模式不需要它們  
	  # appID: ''  
	  # appKey: ''  # --- 其他可選配置 ---  pageview: true # 是否開啟閱讀量統計  
	  comment: true  # 是否開啟評論數統計  
	  lang: 'zh-CN'  
	  placeholder: '來都來了，不說點什麼嗎？'
```

### 部署问题
然后对应项目中的`deploy.yml` 也需要调整

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
      # ==========================================  
      # 终极修复：使用原生的、带严格校验的 rsync 部署到 VPS      # ==========================================      
      - name: Deploy to VPS via Rsync  
        run: |  
          # 1. 将 GitHub Actions 里的私钥写入临时文件，用于认证  
          mkdir -p ~/.ssh  
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa  
          chmod 600 ~/.ssh/id_rsa  
  
          # 2. 绕过首次连接时的主机密钥确认（防止卡住）  
          ssh-keyscan -H ${{ secrets.REMOTE_HOST }} >> ~/.ssh/known_hosts  
  
          # 3. 【新增核心步骤】通过 SSH 让 VPS 自动安装 rsync（CentOS/Rocky 系列）  
          # 使用 dnf/yum 静默安装，如果已安装则自动跳过  
          ssh ${{ secrets.REMOTE_USER }}@${{ secrets.REMOTE_HOST }} "sudo dnf install rsync -y || sudo yum install rsync -y"  
  
          # 3. 使用 rsync 进行严格的覆盖同步（--delete 会自动清理 VPS 目录上旧的、不用的垃圾文件）  
          # -a: 归档模式，保持所有文件属性和完整性  
          # -v: 打印同步详情  
          # -z: 传输时压缩，加快速度  
          rsync -avz --delete _site/ ${{ secrets.REMOTE_USER }}@${{ secrets.REMOTE_HOST }}:/var/www/blog/

```

修正了博客项目中的waline的问题（把中文注释删掉），再推送启动就成功了。

这下又能行了，哦耶。
![comment-01.png](blog/comment-01.png)


---
故事未完:177
**Thoughts**:: justdoit.

