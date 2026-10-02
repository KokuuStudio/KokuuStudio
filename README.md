<div align="center">

<img src="profile-assets/banner.png" alt="KOKUU 谷雨工作室" width="100%">

# KOKUU 谷雨工作室

**雨落下之后，就交给时间。**

Minecraft 服务器运营 · 皮肤站与登录验证 · 前端与插件开发

[![站点](https://img.shields.io/badge/站点-kokuu.org-0FA3A3?style=flat-square)](https://kokuu.org)
[![皮肤站](https://img.shields.io/badge/皮肤站-KokuuSkin-8B7FD4?style=flat-square)](https://github.com/KokuuStudio/kokuu-home)

</div>

---

## 关于

我们是一个小工作室，做三件事：

| | |
| --- | --- |
| 🎮 **Minecraft 服务器** | 运营与维护，非商业化，用爱发电 |
| 🧩 **皮肤站 KokuuSkin** | 基于 Blessing Skin Server 的自建皮肤站与登录验证服务 |
| 🎨 **前端与插件开发** | 为 Blessing Skin 生态开发主题与功能插件 |

技术偏好：**能不改内核就不改**。所有定制都走插件机制实现，
以便上游更新时能直接 `git pull`，不必反复重套改动。

---

## 项目

### KokuuSkin 皮肤站插件套件

为 [Blessing Skin Server](https://github.com/bs-community/blessing-skin-server) 打造的一整套主题插件。
**零依赖安装，一个插件装好即全套生效。**

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

### 其他

| 仓库 | 说明 |
| --- | --- |
| **[kokuu-site](https://github.com/KokuuStudio/kokuu-site)** | 工作室官网 · <https://kokuu.org> · 零依赖单文件静态站 |

---

## 技术栈

```
前端     原生 HTML / CSS / JavaScript · Canvas 像素动画
后端     Laravel 10 · PHP 8.1+ · SQLite / MySQL
插件     Blessing Skin 插件 API（Hook 注入 · 原生配置机制）
工具     Python 3（手写 PNG 编码，无第三方库）
```

---

## 一起玩

Minecraft 服务器长期开放，欢迎加入。

- 服务器地址与入服方式见官网 <https://kokuu.org>
- 皮肤上传：<https://github.com/KokuuStudio/kokuu-home>

---

<div align="center">

**[工作室官网](https://kokuu.org)** · **[皮肤站插件](https://github.com/KokuuStudio/kokuu-home)**

*雨落下之后，就交给时间。*

</div>
