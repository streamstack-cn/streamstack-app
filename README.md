# StreamStack

**私有化部署的一体化影音管理平台** · 115 网盘 × 123 云盘 × Emby

[![Docker Pulls](https://img.shields.io/docker/pulls/streamstack/streamstack)](https://hub.docker.com/r/streamstack/streamstack)
[![Version](https://img.shields.io/docker/v/streamstack/streamstack/latest?label=version)](https://hub.docker.com/r/streamstack/streamstack/tags)
[![Platform](https://img.shields.io/badge/platform-amd64%20%7C%20arm64-blue)](https://hub.docker.com/r/streamstack/streamstack)

---

## 产品简介

StreamStack 是一款私有化部署的一体化影音管理平台，原生支持 **115 网盘**与 **123 云盘**双云存储，深度集成 **Emby** 媒体服务器。

它做的事是：将你云盘中的影视内容自动整理、规范命名，生成 Emby 可直接播放的媒体索引——整条链路全自动，无需手动干预。

它不是脚本，不是命令行工具，是一个完整的独立产品——精心打磨的界面设计、流畅的交互动效、稳定的工程架构，以及持续迭代的功能更新。

> 注：StreamStack 不提供任何影视资源。

---

## 产品亮点

| 亮点 | 说明 |
|------|------|
| STRM 智能编排 | 自动识别、规范命名、生成索引，云盘资源秒变规整媒体库，Emby 打开就能看 |
| 双云盘原生支持 | 115 和 123 云盘均为一等公民，转存、整理、同步一气呵成，不怕限速、不怕掉线 |
| 订阅地图 | 追剧全生命周期管理——自动匹配、自动入库、缺集提醒，追完还有庆祝彩蛋 |
| AI 智影助手 | 对话式交互，发现好剧、获取推荐、加入订阅，遇到难以识别的文件名 AI 自动修正 |
| 全链路自动化 | 从整理到播放，全程无需手动操作，一个 Docker 容器替代五六款工具的拼装折腾 |
| 精致界面体验 | 高级视觉动效、流体光标交互、深色星空主题，每个细节都经过精心设计 |
| 个性化观影画像 | 越用越懂你，根据你的观影习惯智能推荐，订阅选片也会优先挑你喜欢的版本 |
| 多渠道通知 | 支持 Telegram、PushDeer、企业微信，转存成功、入库完成、异常告警实时推送 |
| 媒体年报与徽章 | 年度观影报告、成就徽章系统，记录你的每一段观影旅程 |
| Emby 反代与封面生成 | 内置 Emby 反向代理统一入口，自动生成精美媒体库封面 |
| 20+ 扩展模块 | 豆瓣同步、定时清理、封面生成、种子孵化器……核心之外，持续生长的功能生态 |
| 115 库房管家 | 云盘空间全景总览，影视盘点一目了然，存储状态尽在掌握 |

---

## 功能模块

| 模块 | 说明 |
|------|------|
| 星际搜索 | 聚合多平台元数据，影视信息一搜即达，结果直观呈现 |
| 订阅地图 | 追剧自动化中枢，自动入库、缺集检测、完结庆祝，全程托管 |
| 云帆中心 | 管理 115 / 123 云盘账号，配置转存目录，查看存储空间与状态 |
| 智影助手 | 连接 Emby 与 AI，驱动 STRM 同步，智能对话与个性化推荐 |
| 传输轨迹 | 实时追踪每一次转存记录，成功 / 失败 / 处理中一目了然 |
| 系统日志 | 运行日志实时查看，问题排查一目了然 |
| 系统实验室 | 推送通知、自动清理、豆瓣同步、封面生成、库房管家等 20+ 扩展模块 |
| 源力中心 | 配置 TMDB、HDHive、MoviePilot、qBittorrent 等数据来源与下载器 |

---

## 适合哪些人

- 使用 115 或 123 云盘存储影视，并用 Emby 自建媒体库的用户
- 希望省去手动整理文件、手动重命名、手动刮削元数据繁琐流程的用户
- 想要一个稳定、好看、开箱即用的自动化影音管理工具的用户

> 如果你只使用其中一种（仅 115 / 仅 123 或仅 Emby），StreamStack 的部分功能仍然可用，但完整体验需要云盘与 Emby 搭配。

---

## 快速开始（推荐：分开版）

数据库和缓存运行在独立的 compose 文件中，与主应用解耦，可单独升级 PostgreSQL / Redis 而不影响主应用。

> 最低内存要求：1.5 GB（应用 1 GB + PostgreSQL 512 MB + Redis 256 MB）

完整部署文档请参见 [streamstack.cn/wiki](https://streamstack.cn/wiki)。

---

### 第一步：创建目录

```bash
mkdir streamstack && cd streamstack
```

### 第二步：启动数据库服务

将以下内容保存为 docker-compose.db.yml，修改密码后启动：

```yaml
services:

  db:
    image: postgres:17-alpine
    container_name: streamstack-db
    restart: unless-stopped
    ports:
      - "5432:5432"   # 如已有其他 PostgreSQL 占用 5432，改为如："5433:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data #请正确挂载数据库路径
    environment:
      - POSTGRES_USER=streamstack
      - POSTGRES_PASSWORD=streamstack       # ← 修改为强密码，需与主应用 DB_PASSWORD 完全一致
      - POSTGRES_DB=streamstack
      - TZ=Asia/Shanghai

  redis:
    image: redis:7-alpine
    container_name: streamstack-redis
    restart: unless-stopped
    ports:
      - "6379:6379"   # 如已有其他 Redis 占用 6379，改为如："6380:6379"
    command: >
      redis-server
      --requirepass streamstack
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --save ""
      --appendonly no
    # ↑ 修改 streamstack 为你的密码，需与主应用 REDIS_URL 中密码完全一致 ← 修改
```

启动数据库服务：

```bash
docker compose -f docker-compose.db.yml up -d
```

### 第三步：启动主应用

将以下内容保存为 docker-compose.yml，修改带注释的项后启动：

```yaml
services:
  streamstack:
    image: streamstack/streamstack:latest
    container_name: streamstack
    restart: unless-stopped
    ports:
      - "5717:5717"   # Web 界面端口，如需改端口修改左侧数字，如："8080:5717"
      - "9098:9098"   # Emby 反代端口，不用 Emby 功能可删除此行
    volumes:
      - ./config:/config   #请正确挂载用户配置路径
      - /your/media/path:/media             # ← 修改为你的媒体目录路径
      - /var/run/docker.sock:/var/run/docker.sock:ro

    environment:
      - TZ=Asia/Shanghai
      - ADMIN_USERNAME=admin                # ← 修改为你的用户名
      - ADMIN_PASSWORD=streamstack          # ← 修改为强密码（必须修改！）

      # PostgreSQL 连接（与 docker-compose.db.yml 中保持一致）
      - DB_HOST=127.0.0.1
      - DB_PORT=5432                        # ← 如修改过端口请同步修改
      - DB_NAME=streamstack
      - DB_USER=streamstack
      - DB_PASSWORD=streamstack             # ← 修改为与 docker-compose.db.yml 中相同的密码

      # Redis 连接（密码需与 docker-compose.db.yml 中 --requirepass 一致）
      - REDIS_URL=redis://:streamstack@127.0.0.1:6379/0  # ← 修改密码部分

      - DOCKER_UPDATE_ENABLED=1
      # - PROXY_HOST=http://192.168.1.100:7890   # ← 代理地址（可选）
      # - ACTIVATION_CODE=                        # ← VIP 激活码（可选）

    # Linux 用户必须保留；macOS / Windows Docker Desktop或其他 可删除
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

启动主应用：

```bash
docker compose up -d
```

访问 http://服务器IP:5717

---

### 密码对应关系

| docker-compose.db.yml | docker-compose.yml |
|------------------------|----------------------|
| POSTGRES_PASSWORD=xxx | DATABASE_URL=postgresql://streamstack:xxx@... |
| --requirepass xxx | REDIS_URL=redis://:xxx@... |

---

## 端口说明

| 端口 | 用途 |
|------|------|
| `5717` | Web 管理界面（必须开放） |
| `9098` | Emby 反向代理（开启 Emby 反代功能后生效） |

---

## 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `ADMIN_USERNAME` | ✅ | 管理员账号（仅首次启动生效） |
| `ADMIN_PASSWORD` | ✅ | 管理员密码，**请修改默认值** |
| `ACTIVATION_CODE` | — | VIP 激活码，不填则以免费版运行 |
| `TZ` | — | 时区，默认 `Asia/Shanghai` |
| `DB_HOST` | — | PostgreSQL 主机，不填则使用内置 SQLite |
| `DB_PORT` | — | PostgreSQL 端口，默认 `5432` |
| `DB_NAME` | — | 数据库名，默认 `streamstack` |
| `DB_USER` | — | 数据库用户名 |
| `DB_PASSWORD` | — | 数据库密码 |
| `REDIS_URL` | — | Redis 连接串，不填则使用内存缓存 |
| `PROXY_HOST` | — | HTTP 代理地址（可选） |
| `DOCKER_UPDATE_ENABLED` | — | 启用管理界面一键更新，设为 `1` |

---

## 数据持久化

```yaml
volumes:
  - ./config:/config          # 配置文件持久化（必须挂载）
  - /your/media/path:/media   # 媒体目录（STRM 写入路径）
```

> ⚠️ `./config` 目录包含 license 和应用配置，**务必挂载到宿主机**，容器重建后数据不会丢失。

---

## 更新

```bash
docker compose pull
docker compose up -d
```

或在管理界面点击「检查更新」一键完成。

---

## 系统要求

- Docker 20.10+，支持 Docker Compose v2
- 内存：最低 1.5 GB（应用 1 GB + PostgreSQL 512 MB + Redis 256 MB）
- 架构：`linux/amd64` · `linux/arm64`（群晖、威联通、树莓派、NAS 原生支持）

---

## 官方链接

- 🌐 官网：[streamstack.cn](https://streamstack.cn)
- 📖 文档：[streamstack.cn/wiki](https://streamstack.cn/wiki)
- 💬 反馈：[streamstack.cn/feedback](https://streamstack.cn/feedback)
- 🐳 Docker Hub：[hub.docker.com/r/streamstack/streamstack](https://hub.docker.com/r/streamstack/streamstack)

---

## Tags

| Tag | 说明 |
|-----|------|
| `latest` | 最新稳定版 |
| `beta` | 公测版（功能更新，可能有小问题） |
| `v0.41` 等 | 具体版本，不可变，可用于回滚 |

---

## 鸣谢

StreamStack 在设计与实现过程中参考了以下优秀开源项目，在此表示感谢：

- [jxxghp/MoviePilot](https://github.com/jxxghp/MoviePilot) — 媒体库自动化管理框架，架构设计参考
- [DDSRem-Dev/MoviePilot-Plugins · p115strmhelper](https://github.com/DDSRem-Dev/MoviePilot-Plugins/tree/main/plugins.v2/p115strmhelper) — 115 STRM 同步方案参考
- [ChenyangGao/p115client](https://github.com/ChenyangGao/p115client) — Python 115 网盘客户端接口参考
