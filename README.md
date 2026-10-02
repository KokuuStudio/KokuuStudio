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

---

## 技术栈

```
前端     原生 HTML / CSS / JavaScript · React · Canvas 像素动画
后端     Laravel 10 · PHP 8.1+ · SQLite（WAL）/ MySQL
插件     Blessing Skin 插件 API（Hook 注入 · 原生配置机制）
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

**[工作室官网](https://kokuu.org)** · **[皮肤站内核](https://github.com/KokuuStudio/kokuu-skin-server)** · **[插件套件](https://github.com/KokuuStudio/kokuu-home)**

*雨落下之后，就交给时间。*

</div>
