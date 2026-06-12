# 网络架构概述

## 核心理念

**Cloudflare 泛域名 + Lucky 反向代理 + AdGuard 网络过滤**：

- **Cloudflare**：`*.butubb.cn` 泛解析到 Lucky 网关 IP（192.168.1.2）
- **AdGuard Home**：组网设备 DNS 出口，仅做**网络过滤**（广告/恶意域名拦截），不解析内网域名
- **Lucky**：SSL 终止（自动申请证书）+ Host 头反代

## DNS 与代理职责

| 组件 | 职责 |
|------|------|
| Cloudflare | `*.butubb.cn` → 192.168.1.2 |
| AdGuard Home | 网络过滤，透传查询到 Cloudflare |
| Tailscale | VPN 组网 |
| Lucky | SSL 终止 + 反代网关 |

## 完整访问链路

```
客户端输入 https://dsm.butubb.cn
       │
       ▼
系统 DNS 查询
       │
       ▼
AdGuard Home → 仅过滤 → 透传到 Cloudflare
       │
       ▼
Cloudflare → *.butubb.cn 泛解析 → 192.168.1.2
       │
       ▼
客户端访问 192.168.1.2:443
       │
       ▼
Lucky → SSL 终止 → Host 头匹配 → 反代到 192.168.1.50:5000
       │
       ▼
群晖 NAS 返回数据
```

## "network" 容器三大服务

```
┌──────────────────────────────────────┐
│       "network" LXC (192.168.1.2)    │
├──────────────────────────────────────┤
│  Tailscale  │  AdGuard  │  Lucky     │
│  VPN 节点   │  网络过滤 │  反代网关  │
│  100.64.1.2 │  透传 CF  │  :443 自动证书 │
└──────────────────────────────────────┘
```

## Cloudflare 配置

```
A      *.butubb.cn                  192.168.1.2
CNAME  pve.doc.butubb.cn            username.github.io
```

## Lucky 反代配置

```
规则 1：
  域名：dsm.butubb.cn
  后端：192.168.1.50:5000
  SSL：✓（Lucky 自动申请）

规则 2：
  域名：router.butubb.cn
  后端：192.168.1.1:80
  SSL：✓
```

## 核心组件

| 组件 | 部署位置 | 功能 |
|------|----------|------|
| Tailscale | network LXC | VPN 组网 |
| AdGuard Home | network LXC | 网络过滤 |
| Lucky | network LXC | SSL 终止 + 反代 |
| Cloudflare | 公网 | DNS 解析 |
| GitHub Pages | 公网 | 静态文档 |
| 群晖 NAS | 局域网 | 数据存储 |
| 绿联设备 | 局域网 | 数据备份 |

## 文档目录

- [局域网架构](01-局域网架构.md)
- [组网架构](02-组网架构.md)
- [公网架构](03-公网架构.md)