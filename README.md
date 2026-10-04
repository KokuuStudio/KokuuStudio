<div align="center">

<img src="profile-assets/banner.png" alt="KOKUU 谷雨工作室" width="100%">

# KOKUU 谷雨工作室

**雨落下之后，就交给时间。**

Minecraft 服务器运营 · 皮肤站与登录验证 · 积分中台 · 前端与插件开发

[![官网](https://img.shields.io/badge/官网-kokuu.org-0FA3A3?style=flat-square)](https://kokuu.org)
[![论坛](https://img.shields.io/badge/论坛-chat.kokuu.org-8B7FD4?style=flat-square)](https://chat.kokuu.org)
[![皮肤站](https://img.shields.io/badge/皮肤站-auth.kokuu.org-8B7FD4?style=flat-square)](https://github.com/KokuuStudio/kokuu-skin-server)

</div>

---

## 关于

我们是一个小工作室，做四件事：

| | |
| --- | --- |
| **Minecraft 服务器** | 运营与维护，非商业化，用爱发电 |
| **皮肤站 KokuuSkin** | 基于 Blessing Skin Server 的自建皮肤站与登录验证服务 |
| **论坛 KokuuForum** | 基于 Flarum 的自建论坛，与皮肤站同源，账号与积分互通 |
| **积分中台** | 游戏内金币 / 点券与网站积分的双向桥接，带独立管理台 |

技术偏好：**能不改内核就不改**。所有视觉定制都走插件机制实现，
以便上游更新时能直接 `git pull`，不必反复重套改动。

---

## 项目

### 积分中台

把「游戏里的金币 / 点券」和「网站上的积分」打通，两个方向都能走。

```
┌────────────────────────┐┌────────────────┐┌──────────────────────┐
│    中间件 + 管理台     │<═════>│   Redis 队列   │<═════>│       MC 插件        │
│ 皮肤站 / 论坛（可选）  │<═════>│  双向 · 幂等   │<═════>│ Vault / EssentialsX  │
└────────────────────────┘└────────────────┘└──────────────────────┘
```

四个方向：**①** 积分换金币 · **②** 金币回流换积分 · **③** 管理员改余额 / 限流 · **④** 资产下发

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-credit-admin](https://github.com/KokuuStudio/kokuu-credit-admin)** | 中间件 + 管理台（Vue 3 / Node.js）。账户查询、全站流水多维筛选、数值调整、兑换比例与限额配置、金币回流开关与实时统计 |
| **[exchange-bridge](https://github.com/KokuuStudio/exchange-bridge)** | MC 服务端插件（Java）。BRPOP 队列领取订单，切主线程调 Vault / EssentialsX 发放金币与点券；金币自动回流按比例抽走零头换积分 |

**低耦合**是这一块的设计目标：

- 可插拔后端。整套系统只有 `server/src/runtime.js` **一个文件**知道「皮肤站存在」。
  `BACKEND=standalone` 时不装皮肤站也能完整跑（兑换、限流、流水、管理台全套），
  连 `mysql2` 都不会被加载 —— 模块图上根本不存在。
- **UUID 作为身份锚点**。玩家名可变且可抢注，改了名找不到人，重名了更糟（找到别人）。
  MC UUID 由服务器首次进服生成，不可变、离线模式下同样稳定。
- **账本三原则**：事务 + `SELECT ... FOR UPDATE` 行锁；`event_id` 唯一索引做幂等；
  余额一律从库里读，不信任调用方传的余额。
- 独立管理台，带完整 Docker 一键部署与安装引导，可脱离本工作室环境复用。

**为什么用 Redis 队列而不是 HTTP 回调**：主流经济插件（Vault / EssentialsX）
没有 REST 接口，发放货币这个动作只能由服务端 Java 代码执行。
Redis 队列不需要对外暴露端口，BRPOP「取走即消失」天然互斥，队列本身即防重放。

### KokuuSkin 皮肤站

基于 [Blessing Skin Server](https://github.com/bs-community/blessing-skin-server)
的定制发行版，**内核保持与官方 `dev` 分支一致**，主题定制全部以插件实现。

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-skin-server](https://github.com/KokuuStudio/kokuu-skin-server)** | 定制版内核。补齐核心业务表索引、修复上游路由缓存 bug、SQLite WAL 调优，附低配服务器部署指南 |
| **[kokuu-core](https://github.com/KokuuStudio/kokuu-core)** | 共享内核。不含业务功能，只做「被依赖方」—— 统一论坛 API 的超时、降级、字段兜底 |
| **[kokuu-site](https://github.com/KokuuStudio/kokuu-site)** | 工作室官网 · <https://kokuu.org> · 零依赖单文件静态站 |

### 论坛 KokuuForum

自建 [Flarum](https://flarum.org) 论坛，与皮肤站同源，
负责社区讨论，并把「发帖」变成皮肤站积分的来源。

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-forum](https://github.com/KokuuStudio/kokuu-forum)** | 皮肤站插件。仪表盘论坛聚合卡片，含每个帖子最近评论摘要与置顶帖公告 |
| **[kokuu-forum-points](https://github.com/KokuuStudio/kokuu-forum-points)** | Flarum 扩展。监听发帖 / 回复推送到皮肤站记账（论坛只加分不扣分），并作为 OAuth2 客户端支持用皮肤站账号一键登录 |
| **[kokuu-theme](https://github.com/KokuuStudio/kokuu-theme)** | 论坛专属主题。《原神》「蒙德天空岛」视觉系统，7 个 Less 文件 5010 行 + 1507 行动效，**内核文件覆写 0 个** |
| **[kokuu-quote-flarum](https://github.com/KokuuStudio/kokuu-quote-flarum)** | 提瓦特一言。335 条内置词库、17 个主题标签，每页一处，点击换一句 |

### 积分与兑换（皮肤站侧）

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-credit](https://github.com/KokuuStudio/kokuu-credit)** | 皮肤站插件。积分**唯一账本**（`users.score` + 流水），同时是账号互通的 OAuth2 授权服务端 |
| **[kokuu-exchange](https://github.com/KokuuStudio/kokuu-exchange)** | 皮肤站插件。兑换页 + 订单状态机 + 幂等键；订单强绑定 `players` 表的具体角色，超时未领取自动退款 |
| **[kokuu-coupon](https://github.com/KokuuStudio/kokuu-coupon)** | 兑换码与抽奖。管理员批量生成兑换码，可直接发积分或发抽奖次数，抽奖再吐回积分形成闭环 |

### 主题插件套件

一套完整的皮肤站主题，**按序安装即全套生效**。

| 插件 | 说明 |
| --- | --- |
| **[kokuu-home](https://github.com/KokuuStudio/kokuu-home)** | 全站主题。Apple 扁平首页（MC 场景像素轮播）、浅色 / 深色双主题、登录页重皮肤、后台外壳统一、首页可视化配置 |
| **[kokuu-ui](https://github.com/KokuuStudio/kokuu-ui)** | 后台界面重塑。Apple 扁平设计系统、DOM 级外壳重建、流畅动画、提瓦特元素点缀 |
| **[kokuu-quote](https://github.com/KokuuStudio/kokuu-quote)** | 提瓦特一言（皮肤站侧）。顶栏展示挪德卡莱「空月之歌」台词，可后台增删 |

**设计原则**

- 不修改内核任何模板，全部通过 `Hook` 注入
- 不重写 React 业务逻辑，只替换「舞台」（数据与交互零风险）
- 零第三方前端依赖，无 CDN，不请求外部资源
- 像素插画全部由代码生成（纯 Python 手写 PNG 编码）

**安装顺序**：`kokuu-home` → `kokuu-ui` → `kokuu-quote`
（Blessing Skin 会自动禁用依赖未满足的插件，故须按序启用）

---

## 三个关键设计

### 唯一账本

论坛只做「加分事件来源」，账本只有皮肤站一份：

- 发帖 / 回复由论坛侧扩展上报，带 HMAC 签名 + `event_id` 幂等键 + `nonce` 防重放
- 写入 `users.score` 并留 `credit_ledger` 流水，失败落 `pending` 由定时任务补推
- **论坛侧没有任何扣分入口** —— 建角色、衣柜、兑换仍走皮肤站内核原有逻辑
- 账本只有一份，就不会有「两边余额对不上」的问题

### 账号互通

用标准 OAuth2 授权码 + PKCE，论坛作客户端、皮肤站（Passport）作授权服务器，
`scope` 收窄到 `User.Read`，`client_secret` 不进任何前端资源。

跨站用户映射走显式绑定表（`skin_uid → forum_user_id`），
**不靠 email 唯一映射** —— email 会变，靠 email 的话用户改一次邮箱就再也认不出自己。

SSO 自动建号的用户**论坛密码留空**，密码只由皮肤站掌握，
因此无法用论坛原生登录表单独立登录（已用对照组 + 实验组双向实测确认）。

评估过内核市场的 `forum-integration` 并**否决**：它跨库裸写论坛表、
靠 hasher 类型猜对面框架、硬编码字段、密码 hash 跨库搬运会静默不一致。

### 分布式边界

游戏端与网站端**必须成对使用**，队列 key 两边要填一致。
有个隐蔽坑：皮肤站侧 `phpredis` 的 `OPT_PREFIX` 会给键名偷偷加
`blessing_skin_database_` 前缀，而 MC 端是裸 Jedis 读字面量键名 ——
两侧永久错位且**完全静默**。详见两个仓库的 README。

---

## 技术栈

```
前端     Vue 3 · Vite · Element Plus · React · 原生 HTML / CSS / JS · Canvas 像素动画
后端     Node.js · Express · Laravel 10 · PHP 8.1+ · SQLite（WAL）/ MySQL
插件     Blessing Skin 插件 API（Hook 注入）· Flarum 扩展 · Spigot/Paper 插件（Java 8 字节码）
互通     OAuth2 授权码 + PKCE S256 · Passport · HMAC 签名 · Redis 队列
游戏     Vault / EssentialsX 经济 · MySQL · 1.12.2 ~ 最新（一个 jar 通吃）
部署     Docker Compose · nginx + php-fpm · OPcache
工具     Python 3（手写 PNG 编码，无第三方库）
```

### 兼容范围

| 项目 | 支持 |
| --- | --- |
| Minecraft | **1.12.2 ~ 最新**（编译为 Java 8 字节码，一个 jar 通吃） |
| Java 运行时 | 8 / 11 / 17 / 21 |
| 服务端类型 | Spigot / Paper / Purpur |
| 经济插件 | 任何经 **Vault API** 接入的后端（EssentialsX / CMI 等） |

> **为什么压到 Java 8**：MC 的 Java 版本随版本走（1.12.2 是 Java 8，1.18+ 是 17，
> 1.20.5+ 是 21），而字节码兼容是**单向**的 —— Java 8 字节码能在所有新运行时上跑，
> 反过来则直接 `UnsupportedClassVersionError`。构建机用 JDK 17，也压到 8 编译。

### 性能取向

服务器配置不高，所以在性能上做了针对性取舍：

- 核心业务表补齐索引（上游仅有主键，查询会全表扫描）
- 修复上游重复路由名导致 `route:cache` 无法启用的问题
- SQLite 开 WAL 模式 + 页缓存
- 随仓库分发前端构建产物，部署无需 Node 环境
- 依赖不可用时接口**主动超时**而非无限排队 —— Redis 挂掉不应该拖垮整个管理台

> 详细调优见 [部署与优化指南](https://github.com/KokuuStudio/kokuu-skin-server/blob/main/KOKUUSKIN-%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97.md)

---

## 一起玩

Minecraft 服务器长期开放，欢迎加入。

- 入服方式见官网 <https://kokuu.org>
- 论坛 <https://chat.kokuu.org>
- 皮肤上传：<https://github.com/KokuuStudio/kokuu-skin-server>

---

<div align="center">

**[工作室官网](https://kokuu.org)** · **[论坛](https://chat.kokuu.org)** · **[积分中台](https://github.com/KokuuStudio/kokuu-credit-admin)** · **[皮肤站内核](https://github.com/KokuuStudio/kokuu-skin-server)** · **[插件套件](https://github.com/KokuuStudio/kokuu-home)**

*雨落下之后，就交给时间。*

</div>
