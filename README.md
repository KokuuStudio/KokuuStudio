<div align="center">

<img src="profile-assets/banner.png" alt="KOKUU 谷雨工作室" width="100%">

# KOKUU 谷雨工作室

**雨落下之后，就交给时间。**

Minecraft 服务器运营 · 皮肤站与登录验证 · 积分中台 · 平台化开发

[![官网](https://img.shields.io/badge/官网-kokuu.org-0FA3A3?style=flat-square)](https://kokuu.org)
[![论坛](https://img.shields.io/badge/论坛-chat.kokuu.org-8B7FD4?style=flat-square)](https://chat.kokuu.org)
[![皮肤站](https://img.shields.io/badge/皮肤站-auth.kokuu.org-8B7FD4?style=flat-square)](https://auth.kokuu.org)

</div>

---

## 关于

我们是一个小工作室，主线业务：

| | |
| --- | --- |
| **Minecraft 服务器** | 运营与维护，非商业化，用爱发电 |
| **皮肤站 KokuuSkin** | 基于 Blessing Skin Server 的自建皮肤站与登录验证服务 |
| **积分中台** | 游戏内货币与网站积分的桥接，唯一账本在皮肤站 |
| **KokuuPanel** | 自研 MC 服务器管理平台，一个平台管完所有服 |

技术偏好：**能不改内核就不改**。定制全部走插件机制实现，
以便上游更新时能直接 `git pull`，不必反复重套改动。

---

## 活跃仓库（4 个）

### [kokuu-auth](https://github.com/KokuuStudio/kokuu-auth) · 皮肤站 + 积分经济 monorepo

一个仓库装下整条皮肤站与积分链路，各组件完整提交历史保留（git subtree）：

| 路径 | 来源（原仓库） | 说明 |
| --- | --- | --- |
| `server/` | kokuu-skin-server | 定制版 Blessing Skin 内核：补齐核心表索引、修复路由缓存 bug、SQLite WAL 调优、低配部署指南 |
| `plugins/kokuu-credit` | kokuu-credit | 积分**唯一账本**（`users.score` + 流水），同时是两站账号互通的 OAuth2 授权服务端 |
| `plugins/kokuu-exchange` | kokuu-exchange | 积分兑换游戏货币：订单状态机 + 幂等键 + 超时自动退款 |
| `plugins/kokuu-coupon` | kokuu-coupon | 兑换码与抽奖：积分产出端，兑换码发积分 / 发抽奖次数，抽奖吐回积分 |

### [kokuu-panel](https://github.com/KokuuStudio/kokuu-panel) · MC 服务器管理平台

综合性 Minecraft Java 版服务器管理平台：

- 节点纳管（多服实时状态 / TPS / MSPT / 内存指标）
- 玩家档案（在线列表 / IP 历史 / 上下线记录）
- 跨服一致封禁（权威在平台，按皮肤站 `pid` 锚定，改名绕不过）
- LuckPerms 组与权限管理界面
- 经济模块（不自建账本，接 kokuu-credit 唯一账本）
- 全量操作审计 + 浏览器实时 WebSocket

`legacy/` 目录保留两个前身仓库的完整历史：
[`legacy/credit-admin/`](https://github.com/KokuuStudio/kokuu-panel/tree/main/legacy/credit-admin)
（经济桥接中间件 + 管理台）、
[`legacy/exchange-bridge/`](https://github.com/KokuuStudio/kokuu-panel/tree/main/legacy/exchange-bridge)
（MC 服务端队列执行器插件）。

### [kokuu-site](https://github.com/KokuuStudio/kokuu-site) · 工作室官网

<https://kokuu.org> · 零依赖单文件静态站。

### KokuuStudio · 本仓库

GitHub 组织门面与导航。

---

## 已归档（8 个）

以下仓库是想法验证阶段的产物，已归档为只读。归档只是收起，代码与历史都在，
随时可解开：

- **论坛生态**：`kokuu-forum-points`（论坛积分互通）、`kokuu-forum`（论坛聚合卡片）、`kokuu-core`（共享内核，仅被前者依赖）
- **视觉与主题**：`kokuu-theme`（Flarum 主题）、`kokuu-home` / `kokuu-ui`（皮肤站主题双件套）、`kokuu-quote` / `kokuu-quote-flarum`（提瓦特一言，两平台版）

> 归档原因：当前主线聚焦「皮肤站积分经济 + MC 平台」，论坛与主题方向暂停。
> 其中 `kokuu-quote-flarum` 的 335 条词库、`kokuu-theme` 的 5010 行视觉系统
> 是完整的可复用资产，方向重启时直接解开归档。

---

## 三个关键设计

### 唯一账本

账本只有皮肤站一份，事件源只做「加分事件来源」：

- 事件源经 HMAC 签名 + `event_id` 幂等键 + `nonce` 防重放推送
- 写入 `users.score` 并留 `credit_ledger` 流水，失败落 `pending` 由定时任务补推
- **事件源侧没有任何扣分入口** —— 建角色、衣柜、兑换走皮肤站内核原有逻辑
- 账本只有一份，就不会有「两边余额对不上」的问题

### 账号互通

用标准 OAuth2 授权码 + PKCE，客户端作消费方、皮肤站（Passport）作授权服务器，
`scope` 收窄到 `User.Read`，`client_secret` 不进任何前端资源。

跨站用户映射走显式绑定表，**不靠 email 唯一映射** —— email 会变。

### 积分与游戏货币的边界

游戏端与网站端**必须成对使用**，队列 key 两边要填一致。
有个隐蔽坑：皮肤站侧 `phpredis` 的 `OPT_PREFIX` 会给键名偷偷加
`blessing_skin_database_` 前缀，而 MC 端是裸 Jedis 读字面量键名 ——
两侧永久错位且**完全静默**。详见 `plugins/kokuu-exchange/README.md` 踩坑节。

---

## 技术栈

```
前端     Vue 3 · Vite · Element Plus · React · 原生 HTML / CSS / JS · Canvas 像素动画
后端     Node.js · Fastify · Laravel 10 · PHP 8.1+ · SQLite（WAL）/ MySQL · node:sqlite
插件     Blessing Skin 插件 API（Hook 注入）· Flarum 扩展 · Spigot/Paper 插件（Java 8 字节码）
互通     OAuth2 授权码 + PKCE S256 · Passport · HMAC 签名 · Redis 队列 / WebSocket Agent RPC
游戏     Vault / EssentialsX 经济 · 1.12.2 ~ 最新（一个 jar 通吃）
部署     Docker Compose · nginx + php-fpm · OPcache
```

### 兼容范围

| 项目 | 支持 |
| --- | --- |
| Minecraft | **1.12.2 ~ 最新**（编译为 Java 8 字节码，一个 jar 通吃） |
| Java 运行时 | 8 / 11 / 17 / 21 |
| 服务端类型 | Spigot / Paper / Purpur |
| 经济插件 | 任何经 **Vault API** 接入的后端（EssentialsX / CMI 等） |

> **为什么压到 Java 8**：字节码兼容是**单向**的 —— Java 8 字节码能在所有新运行时上跑，
> 反过来则直接 `UnsupportedClassVersionError`。构建机用 JDK 17，也压到 8 编译。

---

## 一起玩

Minecraft 服务器长期开放，欢迎加入。

- 入服方式见官网 <https://kokuu.org>
- 论坛 <https://chat.kokuu.org>
- 皮肤站 <https://auth.kokuu.org>

---

<div align="center">

**[工作室官网](https://kokuu.org)** · **[论坛](https://chat.kokuu.org)** · **[皮肤站 + 积分](https://github.com/KokuuStudio/kokuu-auth)** · **[MC 管理平台](https://github.com/KokuuStudio/kokuu-panel)**

*雨落下之后，就交给时间。*

</div>
