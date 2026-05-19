---
layout: post
title: "MemOS 开源笔记：30秒自建你的私人日记/备忘录系统｜2026完整安装指南"
date: 2026-05-20
categories: [tech, tools, self-hosted]
permalink: /articles/memos-open-source-note-taking-guide-2026/
description: 开源轻量笔记 Memos 完整安装与使用指南，支持 Docker 一行部署、REST API、Markdown 编辑、标签管理，极简日记工具
---

## 引言：Memos 是什么？

在笔记应用百花齐放的今天，你是否也在寻找一款**真正轻量、完全属于自己**的日记/备忘录工具？Notion 虽强但太重，语雀虽方便但数据不在自己手里，Obsidian 虽本地但同步要折腾——而 **Memos** 的出现，完美解决了这些痛点。

Memos 是一款开源、轻量级的笔记/日记应用，受 Twitter / Weibo 的时间线启发，采用**极简信息流式布局**，让你像发微博一样记录灵感、写日记、做备忘录。它的核心哲学是：**30 秒部署，终身自己掌控**。

截至 2026 年 5 月，Memos 最新稳定版为 **v0.28.0**，GitHub Star 已突破 **35,000+**，社区活跃度极高。你可以把它理解成"你的私人微博 + 日记本 + 备忘录合集"——而且数据 100% 在你自己的服务器上。

## 核心功能一览

### ✍️ 纯 Markdown 编辑
Memos 原生支持 Markdown 语法，写标题、列表、代码块、引用、加粗斜体通通不在话下。编辑区实时预览，所见即所得。

### 🏷️ 标签系统
通过 `#标签` 语法快速分类笔记。比如 `#工作 #日记 #读书笔记` 等，右侧自动生成标签导航栏，可以按标签筛选，比文件夹管理更灵活。

### 🔍 全文搜索
支持按关键词搜索所有笔记内容，响应极快，零延迟。即使攒了几千条备忘录，也能一秒找到。

### 🔌 REST API + 第三方生态
Memos 提供完整的 REST API，开发者可以轻松集成。社区已经涌现了大量基于 Memos API 的工具：
- **Memos Deck**：卡片式看板
- **Memos Daily**：每日回顾邮件
- **Chrome 扩展**：一键保存网页片段到 Memos
- **Telegram Bot**：通过 Telegram 发消息自动写入 Memos

### 🌙 暗色模式
一键切换明暗主题，夜间写作不刺眼。

### 📱 移动端 PWA
Memos 支持 Progressive Web App，手机浏览器打开后添加到桌面，体验接近原生 App，无需安装额外客户端。

## 安装方法：30 秒部署

Memos 提供了多种部署方式，最推荐的是 **Docker 一键部署**。

### 方式一：Docker 一行命令（推荐）

```bash
docker run -d \
  --name memos \
  -p 8081:8081 \
  -v ~/.memos/:/var/opt/memos \
  neosmemo/memos:stable
```

这条命令做了三件事：
1. 后台运行 Memos 容器，命名为 `memos`
2. 将主机的 `8081` 端口映射到容器的 `8081` 端口
3. 将主机 `~/.memos/` 目录挂载为数据持久化目录

启动后，浏览器访问 `http://你的IP:8081`，注册第一个账号（自动成为管理员），即可开始使用。

### 方式二：数据目录详解

Memos 的所有数据都存储在挂载目录 `~/.memos/` 下，结构如下：

```
~/.memos/
├── memos_prod.db       # SQLite 数据库（核心数据）
├── assets/             # 上传的图片/附件
└── profile.json        # 服务配置
```

**备份只需备份 `memos_prod.db` 和 `assets/` 目录**，以后迁移数据非常方便。

### 方式三：配置 systemd 服务（开机自启）

如果你用的是 Linux 服务器，可以配置 systemd 保证 Memos 随系统自动启动：

```ini
[Unit]
Description=Memos Service
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker start -a memos
ExecStop=/usr/bin/docker stop memos
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

保存到 `/etc/systemd/system/memos.service`，然后执行：

```bash
sudo systemctl daemon-reload
sudo systemctl enable memos
sudo systemctl start memos
```

### 方式四：Nginx 反向代理 + 域名 HTTPS

如果希望用域名访问并启用 HTTPS（强烈推荐），配置 Nginx 反代：

```nginx
server {
    listen 80;
    server_name memos.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

然后用 Certbot 申请 SSL 证书：

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d memos.yourdomain.com
```

30 秒搞定 HTTPS，安全又省心。

## 使用技巧

### 💾 定期备份

用 cron 定时备份数据库：

```bash
# 每天凌晨 3 点备份
0 3 * * * cp ~/.memos/memos_prod.db ~/backups/memos_$(date +\%Y\%m\%d).db
```

备份后可以用 `scp` 或 `rclone` 同步到其他存储。

### 🔄 数据迁移

从旧服务器迁移到新服务器：

1. 在旧服务器上打包数据：`tar -czf memos-backup.tar.gz ~/.memos/`
2. 传到新服务器：`scp memos-backup.tar.gz user@新IP:~`
3. 解压到新位置：`tar -xzf memos-backup.tar.gz -C ~/`
4. 在新服务器上启动 Docker 容器挂载同样的目录

无缝迁移，数据完整保留。

### 🔐 多用户管理

Memos 支持多用户注册和登录。管理员可以在设置中开启或关闭开放注册，适合家庭或小团队共享使用。

## 对比其他笔记工具

| 特性 | Memos | Notion | Obsidian | 语雀 |
|------|-------|--------|----------|------|
| 部署方式 | 自托管（Docker） | SaaS 云端 | 本地 + 同步 | SaaS 云端 |
| 数据所有权 | ✅ 完全自控 | ❌ 云端 | ✅ 本地文件 | ❌ 云端 |
| 安装复杂度 | 极低（30秒） | 无需安装 | 需要客户端 | 无需安装 |
| 离线使用 | ❌ 需网络 | ⚠️ 有限 | ✅ 完全离线 | ❌ 需网络 |
| 移动端体验 | PWA（够用） | App 完善 | App 完善 | App 完善 |
| 笔记组织 | 标签+时间线 | 数据库+页面 | 文件+双向链接 | 目录+知识库 |
| 开放生态 | REST API + 社区工具 | 有限 API | 插件生态 | 有限 |
| 适合场景 | 日记/备忘录/随手记 | 项目管理/数据库 | 知识管理/PKM | 团队文档 |

**小结**：Memos 不是要替代 Notion 或 Obsidian，而是在"极简日记/备忘录"这个细分领域做到极致。如果你主要需求是**写日记、随手记、做备忘录**，Memos 是体验最好的选择——没有之一。

## 总结

Memos 是一个"小但强大"的开源项目。它用最简洁的方式解决了"快速记录"这个核心需求，同时通过 REST API 和社区生态保持了极强的扩展性。

如果你厌倦了笔记 App 越来越臃肿、数据越来越封闭的趋势，Memos 会给你一种久违的清爽感——**你的数据，你掌控**。

🔗 项目地址：<https://github.com/usememos/memos>
🔗 官方文档：<https://usememos.com/docs>
🔗 在线 Demo：<https://demo.usememos.com/>

现在就试试一行命令部署你的私人日记系统吧。30 秒后，你就有了一台完全属于自己的时间线笔记服务器。