---
title: 开发_07_Linux
description: Linux相关
date: 2025-08-07
categories:
  - develop
tags:
  - develop
author: blackbzy
update_date: 2026-03-25
pin: false
toc: true
comments: true
image:
  path: tech/linux.png
  alt: linux
---

> 使用才能记得住
{: .prompt-info }


## Linux常用命令
### 1. 文件与目录操作 (基础中的基础)

|**命令**|**用途**|**常用参数/示例**|
|---|---|---|
|**`ls`**|列出文件|`ls -al` (显示所有文件及详细信息)|
|**`cd`**|切换目录|`cd ~` (回家目录), `cd ..` (上级目录)|
|**`pwd`**|显示当前完整路径|迷路时必敲|
|**`mkdir`**|创建目录|`mkdir -p a/b/c` (递归创建多级目录)|
|**`rm`**|删除文件/目录|`rm -rf` (**慎用！** 强制递归删除)|
|**`cp`**|复制|`cp -r` (复制文件夹)|
|**`mv`**|移动或重命名|`mv old.txt new.txt`|

---

### 2. 文件内容查看与编辑

- **`cat`**: 一次性查看全文本（适合小文件）。
- **`more` / `less`**: 分页查看长文本（`less` 更好用，支持上下滚动和搜索）。
- **`tail`**: **程序员最爱**。`tail -f app.log` 实时滚动查看日志更新。
- **`grep`**: 强大的文本过滤。`grep "ERROR" app.log` 快速定位错误。
- **`vim`**: 终端里的编辑器王者。
  - `i` 进入编辑，`Esc` 退出编辑。
  - `:wq` 保存退出，`:q!` 不保存强制退出。

---

### 3. 系统监控与进程管理

当你的程序“卡死”或者服务器变慢时：

- **`top`**: 相当于任务管理器，实时查看 CPU、内存占用。
- **`ps -ef`**: 查看系统当前运行的所有进程。通常配合 `grep` 使用：
  - `ps -ef | grep java` (看看我的 Java 程序还在不在)
- **`kill`**: 结束进程。`kill -9 <PID>` (强制杀死)。
- **`df -h`**: 查看磁盘剩余空间。
- **`free -m`**: 查看内存使用情况。

---

### 4. 网络与传输

- **`ifconfig` / `ip addr`**: 查看本机 IP 地址。
- **`netstat -ntlp`**: 查看哪些端口正在被占用。
- **`curl`**: 发送网络请求，测试接口神器。`curl -X POST http://localhost:8080/api`
- **`scp` / `rsync`**: 在不同服务器之间拷贝文件。

---

### 5. 权限管理 (经常遇到的坑)

如果你遇到 `Permission denied`：
- **`chmod`**: 修改权限。`chmod 777 file` (全开放)，`chmod +x script.sh` (增加执行权限)。
- **`chown`**: 修改所属用户/组。`chown root:root file`。
- **`sudo`**: 以超级管理员身份运行。

---

提高效率的小技巧：**管道符 `|`**
Linux 命令最强大的地方在于组合。比如你想找出日志里包含 "Exception" 的最后 50 行：
`tail -n 500 app.log | grep "Exception" | tail -n 50`

故事未完:84
**Thoughts**:: justdoit.
