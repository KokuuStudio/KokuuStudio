<div align="center">

<img src="profile-assets/banner.png" alt="KOKUU 谷雨工作室" width="100%">

# KOKUU 谷雨工作室

**雨落下之后，就交给时间。**

Minecraft 服务器运营 · 皮肤站与登录验证 · 前端与插件开发

[![官网](https://img.shields.io/badge/官网-kokuu.org-0FA3A3?style=flat-square)](https://kokuu.org)
[![皮肤站](https://img.shields.io/badge/皮肤站-KokuuSkin-8B7FD4?style=flat-square)](https://github.com/KokuuStudio/kokuu-skin-server)

</div>

---

## 关于

我们是一个小工作室，做三件事：

| | |
| --- | --- |
| 🎮 **Minecraft 服务器** | 运营与维护，非商业化，用爱发电 |
| 🧩 **皮肤站 KokuuSkin** | 基于 Blessing Skin Server 的自建皮肤站与登录验证服务 |
| 🎨 **前端与插件开发** | 为 Blessing Skin 生态开发主题与功能插件 |

技术偏好：**能不改内核就不改**。所有视觉定制都走插件机制实现，
以便上游更新时能直接 `git pull`，不必反复重套改动。
---

## 项目

### KokuuSkin 皮肤站

基于 [Blessing Skin Server](https://github.com/bs-community/blessing-skin-server)
的定制发行版，**内核保持与官方 `dev` 分支一致**，主题定制全部以插件实现。

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-skin-server](https://github.com/KokuuStudio/kokuu-skin-server)** | 定制版内核。补齐核心业务表索引、修复上游路由缓存 bug、SQLite WAL 调优，附低配服务器部署指南 |
| **[kokuu-site](https://github.com/KokuuStudio/kokuu-site)** | 工作室官网 · <https://kokuu.org> · 零依赖单文件静态站 |

### 主题插件套件

一套完整的主题插件，**按序安装即全套生效**。

| 插件 | 说明 |
| --- | --- |
| **[kokuu-home](https://github.com/KokuuStudio/kokuu-home)** | 全站主题。Apple 扁平首页（MC 场景像素轮播）、浅色/深色双主题、登录页重皮肤、后台外壳统一、首页可视化配置 |
| **[kokuu-ui](https://github.com/KokuuStudio/kokuu-ui)** | 后台界面重塑。Apple 扁平设计系统、DOM 级外壳重建、流畅动画、提瓦特元素点缀 |
| **[kokuu-quote](https://github.com/KokuuStudio/kokuu-quote)** | 提瓦特一言。顶栏展示挪德卡莱「空月之歌」台词，可后台增删 |

**设计原则**

- 不修改内核任何模板，全部通过 `Hook` 注入
- 不重写 React 业务逻辑，只替换「舞台」（数据与交互零风险）
- 零第三方前端依赖，无 CDN，不请求外部资源
- 像素插画全部由代码生成（纯 Python 手写 PNG 编码）

**安装顺序**：`kokuu-home` → `kokuu-ui` → `kokuu-quote`
（Blessing Skin 会自动禁用依赖未满足的插件，故须按序启用）

### 论坛 KokuuForum

自建 [Flarum](https://flarum.org) 论坛，与皮肤站同源，
负责社区讨论，并把「发帖」变成皮肤站积分的来源。

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-forum-points](https://github.com/KokuuStudio/kokuu-forum-points)** | Flarum 扩展。监听发帖/回复推送到皮肤站记账（论坛只加分不扣分），并作为 OAuth2 客户端支持用皮肤站账号一键登录 |
| **[kokuu-credit](https://github.com/KokuuStudio/kokuu-credit)** | 皮肤站插件。积分**唯一账本**（`users.score` + 流水），同时是账号互通的 OAuth2 授权服务端 |
| **[kokuu-forum](https://github.com/KokuuStudio/kokuu-forum)** | 皮肤站插件。仪表盘论坛聚合卡片，含每个帖子最近评论摘要与置顶帖公告 |

**积分兑换（游戏货币）**

论坛积分可以换成 MC 服务端的游戏内货币（经 Vault / EssentialsX 发放）。
采用「**直连发放**」模式：玩家在皮肤站点一下就生成订单，
MC 端插件轮询领取并就地发放，**不需要进游戏输任何兑换码**。

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-exchange](https://github.com/KokuuStudio/kokuu-exchange)** | 皮肤站插件。兑换页 + 订单状态机 + 幂等键；订单强绑定 `players` 表的具体角色，超时未领取自动退款 |
| **[exchange-bridge](https://github.com/KokuuStudio/exchange-bridge)** | MC 服务端插件（Java）。BRPOP 队列领取订单，切主线程调 Vault 经济接口 `depositPlayer()` 发放，结果回传 |

> 两端**必须成对使用**，且队列 key 两边要填一致。
> ⚠️ 有个隐蔽坑：皮肤站侧 `phpredis` 的 `OPT_PREFIX` 会给键名偷偷加
> `blessing_skin_database_` 前缀，而 MC 端是裸 Jedis 读字面量键名 ——
> 两侧永久错位且**完全静默**。详见两个仓库的 README。

**积分互通设计**

论坛只做「加分事件来源」，账本只有皮肤站一份：

- 发帖 / 回复由论坛侧扩展上报，带 HMAC 签名 + `event_id` 幂等键 + `nonce` 防重放
- 写入 `users.score` 并留`credit_ledger` 流水，失败落`pending` 由定时任务补推
- **论坛侧没有任何扣分入口**——建角色、衣柜、兑换仍走皮肤站内核原有逻辑
- 账本只有一份，就不会有「两边余额对不上」的问题

**账号互通设计**

用标准 OAuth2 授权码 + PKCE，论坛作客户端、皮肤站（Passport）作授权服务器，
`scope` 收窄到 `User.Read`，`client_secret` 不进任何前端资源。

跨站用户映射走显式绑定表（`skin_uid → forum_user_id`），
**不靠 email 唯一映射**——email 会变，靠email 的话用户改一次邮箱就再也认不出自己。

SSO 自动建号的用户**论坛密码留空**，密码只由皮肤站掌握，
因此无法用论坛原生登录表单独立登录（已用对照组 + 实验组双向实测确认）。

评估过内核市场的 `forum-integration` 并**否决**：它跨库裸写论坛表、
靠 hasher 类型猜对面框架、硬编码字段、密码 hash 跨库搬运会静默不一致。

---

## 技术栈

```
前端     原生 HTML / CSS / JavaScript · React · Canvas 像素动画
后端     Laravel 10 · PHP 8.1+ · SQLite（WAL）/ MySQL
论坛     Flarum 1.8（PHP 扩展）
插件     Blessing Skin 插件 API（Hook 注入 · 原生配置机制）
互通     OAuth2 授权码 + PKCE S256 · Passport · HMAC 签名
部署     nginx + php-fpm · OPcache · config/route 缓存
工具     Python 3（手写 PNG 编码，无第三方库）
```

### 性能取向

服务器配置不高，所以在性能上做了针对性取舍：

- 核心业务表补齐索引（上游仅有主键，查询会全表扫描）
- 修复上游重复路由名导致 `route:cache` 无法启用的问题
- SQLite 开 WAL 模式 + 页缓存
- 随仓库分发前端构建产物，部署无需 Node 环境

> 详细调优见 [部署与优化指南](https://github.com/KokuuStudio/kokuu-skin-server/blob/main/KOKUUSKIN-%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97.md)

---

## 一起玩

Minecraft 服务器长期开放，欢迎加入。

- 入服方式见官网 <https://kokuu.org>
- 皮肤上传：<https://github.com/KokuuStudio/kokuu-skin-server>

---

<div align="center">

**[工作室官网](https://kokuu.org)** · **[皮肤站内核](https://github.com/KokuuStudio/kokuu-skin-server)** · **[插件套件](https://github.com/KokuuStudio/kokuu-home)** · **[论坛积分](https://github.com/KokuuStudio/kokuu-credit)**

*雨落下之后，就交给时间。*

</div>
