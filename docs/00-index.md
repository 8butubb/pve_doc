# PVE 服务器配置文档

> 基于 **铭凡 N5 Air** + **Proxmox VE** 的家庭服务器完整配置记录与技术分享。

## 文档说明

本文档是个人 PVE 服务器配置的系统性记录与分享平台，采用 **AI 辅助 + 人工审核** 的方式维护，确保技术准确性与实用性。

### 三大核心功能

#### 1. 配置记录中心
防止配置细节遗忘，便于长期维护与回溯：

- **硬件规格**：CPU、内存、磁盘、网络接口
- **软件版本**：PVE 系统版本、关键组件版本、已安装应用
- **网络设置**：IP 分配、端口规划、防火墙规则
- **存储配置**：LVM/RAID 设置、存储池分配、备份策略
- **VM/CT 参数**：资源分配、操作系统、网络配置、特殊设置（PCIe 直通等）

#### 2. 技术交流与学习平台
系统整理 PVE 搭建经验，便于其他用户参考：

- **搭建流程**：含详细步骤与配置示例
- **性能优化**：CPU/内存/存储/网络的调优方法
- **常见问题**：错误信息、排查过程、解决步骤

#### 3. 联系方式
欢迎通过 GitHub 进行技术交流与讨论：

- **GitHub 仓库**：[https://github.com/8butubb/pve_doc](https://github.com/8butubb/pve_doc)
- **GitHub 账号**：[https://github.com/8butubb](https://github.com/8butubb)

---

## 目录结构

```
pve_doc/
├── docs/
│   ├── 00-index.md           # 项目索引（本文档）
│   ├── hardware/             # 硬件相关
│   │   ├── 00-index.md       # 宿主机硬件选型
│   │   ├── 01-我的需求.md    # 需求与规划
│   │   └── 02-落地配置.md    # 实际硬件配置清单
│   └── network/              # 网络架构
│       ├── 00-index.md       # 网络架构概述
│       ├── 01-局域网架构.md  # 局域网访问链路
│       ├── 02-组网架构.md    # Tailscale VPN 组网
│       └── 03-公网架构.md    # 公网访问策略
└── mkdocs.yml                # MkDocs 配置
```

---

## 硬件规格

### 宿主机：铭凡 N5 Air

| 项目 | 规格 |
|------|------|
| 处理器 | AMD Ryzen 7 255（8核16线程，基础 3.8GHz / 加速 4.95GHz，TDP 45W） |
| 核显 | AMD Radeon 780M（RDNA 3.0） |
| 内存 | 16GB DDR5 5600MHz（单通道，最高支持 96GB） |
| 系统盘 | 250GB M.2 SSD × 2 |
| 数据盘 | SATA 4TB × 4 + 8TB × 1 |
| 网口 | 1× 10GbE（RTL8127）+ 1× 5GbE（RTL8126） |
| 扩展 | PCIe 4.0 x4、OCuLink、USB4 × 2 |

### 落地配置

| 组件 | 规格 |
|------|------|
| 内存 | 16GB DDR5 5600MHz（单通道） |
| 系统盘 | 250GB M.2 SSD × 2 |
| 虚拟化平台 | Proxmox VE |
| 数据盘 | SATA 4TB × 4 + 8TB × 1 |

---

## 虚拟化规划

### LXC 容器（3个）

| 用途 | 说明 |
|------|------|
| 服务容器 | 运行各类服务 |
| 开发容器 | 开发环境 |
| 网关容器 | 网络网关（部署 Tailscale + AdGuard + Lucky） |

### 虚拟机（3个）

| 系统 | 用途 | 备注 |
|------|------|------|
| DSM | 主 NAS 系统 | 核心存储服务 |
| Windows 10 | 日常使用 | 计划直通 RTX 3060 显卡 |
| FNOS | 尝鲜体验 | 轻度使用 |

### 扩展计划
- 后期通过 PCIe HBA 卡连接硬盘柜
- TrueNAS 作为冷备份方案

---

## 网络架构概览

### 核心组件

| 组件 | 部署位置 | 功能 |
|------|----------|------|
| Tailscale | network LXC | VPN 组网 |
| AdGuard Home | network LXC | 网络过滤 |
| Lucky | network LXC | SSL 终止 + 反代 |
| Cloudflare | 公网 | DNS 解析 |
| GitHub Pages | 公网 | 静态文档 |

### 职责划分

- **Cloudflare**：`*.butubb.cn` 泛解析到 192.168.1.2（Lucky）
- **AdGuard Home**：组网 DNS 出口，仅做**网络过滤**
- **Lucky**：SSL 终止（自动证书）+ Host 头反代

### 关键 IP

| 设备 | LAN IP | 用途 |
|------|--------|------|
| 路由器 | 192.168.1.1 | 网关 |
| network LXC | 192.168.1.2 | AdGuard + Tailscale + Lucky |
| 群晖 NAS | 192.168.1.50 | 主存储 |
| 绿联 NAS | 192.168.1.60 | 备份 |

### 端口规划

| 服务 | 端口 | 说明 |
|------|------|------|
| AdGuard DNS | 53 | 网络过滤 |
| AdGuard Web | 3000 | 管理界面 |
| Lucky HTTPS | 443 | 反代入口 |
| Tailscale | 41641/UDP | VPN 通信 |

---

## 软件版本

| 组件 | 版本 | 说明 |
|------|------|------|
| Proxmox VE | 待定 | 虚拟化平台 |
| Tailscale | latest | VPN 客户端 |
| AdGuard Home | latest | DNS 过滤 |
| Lucky | latest | 反向代理 |

> 实际版本号请以部署时的 `pveversion -v` 与各服务 Web 界面为准。

---

## 性能优化

### CPU
- 启用 `kvm` 硬件加速
- 虚拟机 CPU 类型选择 `host`

### 内存
- 启用 ballooning
- 关闭透明大页（THP）

### 存储
- 系统盘使用 SSD
- 数据盘使用 HDD + LVM
- 启用 `fstrim` 定期回收

### 网络
- 桥接使用 `vmbr0`
- VirtIO 半虚拟化网卡

---

## 常见问题

### Q1: PCIe 设备直通失败
- 确认 BIOS 开启 `SVM`、`IOMMU`
- 内核参数添加 `intel_iommu=on` 或 `amd_iommu=on`
- GRUB 配置：`GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"`

### Q2: LXC 容器无法启动
- 检查模板是否下载完整
- 检查存储空间是否充足
- 查看日志：`pct start <vmid> --debug`

### Q3: AdGuard Home DNS 解析失败
- 确认上游 DNS 可达（1.1.1.1）
- 确认防火墙放行 53 端口
- 检查 `/etc/resolv.conf` 配置

> 更多问题与解决方案，请参考 [Issues](https://github.com/8butubb/pve_doc/issues)。

---

## 文档维护

### 创建与修改记录

- **2026-06-12**：创建项目索引文档，梳理整体架构
- 后续更新将在 [CHANGELOG.md](https://github.com/8butubb/pve_doc/blob/main/CHANGELOG.md) 中记录

### 维护原则

1. **准确性**：所有配置需经过实际验证
2. **及时性**：配置变更后及时更新文档
3. **可读性**：使用统一的 Markdown 格式
4. **可检索**：通过目录、标签、关键词便于查找

---

## 联系方式

### 技术交流

欢迎通过 GitHub 进行技术交流与讨论：

- **GitHub 仓库**：[https://github.com/8butubb/pve_doc](https://github.com/8butubb/pve_doc)
- **GitHub 个人**：[https://github.com/8butubb](https://github.com/8butubb)
- **Issues**：[提交问题](https://github.com/8butubb/pve_doc/issues)
- **Discussions**：[参与讨论](https://github.com/8butubb/pve_doc/discussions)

### 文档站

- **在线地址**：[https://pve.doc.butubb.cn](https://pve.doc.butubb.cn)
- **源码**：本文档由 MkDocs + Material 主题构建

---

## 许可与免责声明

本文档采用 AI 辅助生成 + 人工审核的方式创作，内容仅供学习与参考。

- 文档内容遵循 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 协议
- 实际部署请根据自身环境调整参数
- 作者不对因使用本文档造成的任何损失负责
