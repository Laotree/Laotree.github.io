---
layout: post
title: 通过公网回家的一种方法
---

{{ page.title }}
================

<p class="meta">12 February 2026 - Shanghai</p>

##  安全第一

如果希望从公网回家，安全是第一位的

**在黑暗森林中，需要小心谨慎地开窗户**

如果决定尝试这个方法，请务必自检是否做到了一些基础的安全设置

- 强密码，强密码，强密码
- 两步验证，两步验证，两步验证
- 账号隔离，账号隔离，账号隔离

## 它是怎么工作的

这个方案基于 Cloudflare Tunnel，核心思路是：在家里的 NAS 上运行一个客户端，通过 Cloudflare 的边缘网络建立一条反向隧道，这样你就能从公网通过一个子域名访问家中的 NAS，而不需要公网 IP、不需要端口转发、不需要 DDNS。

整体分为两条流量路径：

**管控流量** — 客户端注册、获取 Tunnel Token：

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────────────┐
│   客户端     │────▶│    服务平台       │────▶│   Cloudflare Tunnel  │
│  (NAS 上)   │◀────│  (API Server)    │◀────│                      │
└─────────────┘     └─────────────────┘     └──────────────────────┘
```

**业务流量** — 实际访问数据，不经过服务平台：

```
┌──────────┐     ┌───────────────────┐     ┌──────────────┐     ┌─────────┐
│  用户设备  │────▶│  Cloudflare Edge  │────▶│ cloudflared  │────▶│   NAS   │
│ (浏览器)  │◀────│  (公网边缘节点)    │◀────│ (家庭网络内)  │◀────│ (本地)  │
└──────────┘     └───────────────────┘     └──────────────┘     └─────────┘
```

业务流量直接走 Cloudflare 边缘节点到你家里的 cloudflared 进程，全程 TLS 加密，服务平台不参与也看不到。

## 获取客户端源代码

[LetMeHomeClient](https://github.com/Laotree/LetMeHomeClient)

```bash
git clone https://github.com/Laotree/LetMeHomeClient
cd LetMeHomeClient
```

## 使用 Docker 部署（推荐）

### 1. 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env` 文件，填入以下信息：

```bash
# 服务平台地址
SERVER_URL=http://your-server:8000

# 邀请码（首次注册时必填，之后可留空）
INVITE_CODE=你的邀请码

# NAS 本地访问地址
NAS_URL=http://192.168.1.100:5000

# 自定义子域名前缀（可选）
SUBDOMAIN=
```

其中 `NAS_URL` 填你 NAS 管理界面的局域网地址，比如群晖默认是 `http://192.168.x.x:5000`。

### 2. 启动客户端

```bash
docker compose up -d
```

客户端首次启动会使用邀请码向服务平台注册，获取 Tunnel Token 并持久化到 Docker Volume 中。后续重启会跳过注册，直接使用已保存的 Token 启动隧道。

## 非 Docker 方式

如果不想用 Docker，确保系统上有 `curl` 和 `jq`，然后直接运行：

```bash
bash client.sh register \
  --server http://your-server:8000 \
  --invite-code YOUR_CODE \
  --nas-url http://192.168.1.100:5000
```

其他常用命令：

```bash
bash client.sh status      # 查看隧道状态
bash client.sh stop        # 停止隧道
bash client.sh uninstall   # 卸载客户端
```

## 最后

这个方案的好处是简单——不需要折腾公网 IP、不需要配置路由器端口转发、不需要自己管理证书。Cloudflare Tunnel 帮你处理了 TLS 和路由，你只需要在 NAS 上跑一个容器就行了。

但还是那句话，**安全第一**。开放任何远程访问之前，请确保你的 NAS 本身是安全的。
