# StreamStack

**智能媒体库管理工具** · 115网盘 × Emby 一体化解决方案

[![Docker Pulls](https://img.shields.io/docker/pulls/streamstack/streamstack)](https://hub.docker.com/r/streamstack/streamstack)
[![Version](https://img.shields.io/docker/v/streamstack/streamstack/latest?label=version)](https://hub.docker.com/r/streamstack/streamstack/tags)
[![Platform](https://img.shields.io/badge/platform-amd64%20%7C%20arm64-blue)](https://hub.docker.com/r/streamstack/streamstack)

---

## 产品简介

StreamStack 是一款面向影视爱好者的媒体库自动化管理工具，专为 **115网盘 + Emby** 用户设计。

无需手动整理文件，StreamStack 自动完成从内容归档、元数据刮削到 Emby 入库的完整链路——打开 Emby，你的媒体库已经整理好了。

---

## 核心功能

### 🗂 STRM 智能同步
将 115网盘内容自动生成 STRM 文件并同步到 Emby，无需本地下载，直接在 Emby 中流畅播放网盘资源。支持增量同步、目录监控、自动刷新媒体库。

### 🔍 内容监控与自动归档
监控指定内容，自动归档到 115 网盘目标目录，全程无人值守。支持关键词过滤、分辨率偏好等精细规则。

### 🎬 Emby 深度集成
- 自动触发 Emby 刮削与媒体库刷新
- Emby 反向代理（端口 9098），统一入口
- 海报封面自动获取与缓存

### 📦 转存轨迹管理
完整记录每一条归档操作的历史，支持按标题、来源、状态筛选，随时追溯。

### 🔧 可视化配置管理
Web 界面配置所有参数，无需修改配置文件。115账号、Emby 连接、目录映射、归档规则，全部在界面内完成。

### 📊 系统监控与日志
实时查看任务运行状态、错误日志、同步进度，内置健康检查接口。

### 🔄 一键在线更新
管理员界面直接拉取最新版本，无需 SSH 登录服务器，`docker compose pull` 自动完成。

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
      - "5432:5432"       # 如与其他 PostgreSQL 冲突，改为 "5433:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data   # 数据库文件持久化（必须）
    environment:
      - POSTGRES_USER=streamstack
      - POSTGRES_PASSWORD=your_db_password  # 修改为强密码，需与主应用 DATABASE_URL 一致
      - POSTGRES_DB=streamstack
      - TZ=Asia/Shanghai
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U streamstack -d streamstack"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: streamstack-redis
    restart: unless-stopped
    ports:
      - "6379:6379"       # 如与其他 Redis 冲突，改为 "6380:6379"
    command: >
      redis-server
      --requirepass your_redis_password
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --save ""
      --appendonly no
    # 修改 your_redis_password，需与主应用 REDIS_URL 中密码一致
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "your_redis_password", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
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
      - "5717:5717"     # Web 界面，如需改端口修改左边数字，如 "8080:5717"
      - "9098:9098"     # Emby 反向代理（不用 Emby 功能可删除此行）
    volumes:
      - ./config:/config                              # 配置与数据持久化（必须）
      - /your/media/path:/media                       # 媒体目录，改为实际路径
      - /var/run/docker.sock:/var/run/docker.sock:ro  # 一键更新所需
      - .:/project:ro
    environment:
      - TZ=Asia/Shanghai
      - APP_PORT=5717

      # 管理员账号（仅首次启动生效，之后修改无效）
      - ADMIN_USERNAME=admin                          # 修改为你的用户名
      - ADMIN_PASSWORD=your_admin_password            # 必须修改为强密码

      # PostgreSQL 连接（格式：postgresql://用户名:密码@地址:端口/数据库名）
      # 密码需与 docker-compose.db.yml 中 POSTGRES_PASSWORD 完全一致
      - DATABASE_URL=postgresql://streamstack:your_db_password@host.docker.internal:5432/streamstack

      # Redis 连接（密码需与 docker-compose.db.yml 中 --requirepass 完全一致）
      - REDIS_URL=redis://:your_redis_password@host.docker.internal:6379/0

      # 激活码（可选，不填则以免费版运行）
      # - ACTIVATION_CODE=

      # 代理（可选）
      # - PROXY_HOST=http://192.168.1.100:7890

      # 一键更新
      - DOCKER_UPDATE_ENABLED=true
      - DOCKER_COMPOSE_FILE=/project/docker-compose.yml
      - DOCKER_COMPOSE_SERVICE=streamstack

    # Linux 必须保留；macOS / Windows Docker Desktop 可删除此段
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