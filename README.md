<div align="center">

# 📬 回信计数器

**一个支持多人实时共享的网页计数器。零构建、零运行时依赖。**

[![CI](https://github.com/mingongye/jishuqi/actions/workflows/ci.yml/badge.svg)](https://github.com/mingongye/jishuqi/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![零依赖](https://img.shields.io/badge/runtime_dependencies-0-success.svg)](package.json)

[快速开始](#-快速开始) · [功能特性](#-功能特性) · [键盘快捷键](#-键盘快捷键) ·
[项目结构](#-项目结构) · [自行部署](#-自行部署) · [更新指南](CONTRIBUTING.md)

</div>

---

## 这是什么

一个打开网页就能用的计数器，**所有打开同一页面的人共享同一个数字**。

典型场景：活动现场统计签到人数、群里统计报名、值班时统计处理量、投票统计。
一个人按加号，其他人的屏幕**同时**跳动——不需要刷新，不需要服务器部署，
把 HTML 丢到任意静态托管（GitHub Pages / Vercel / 对象存储）即可对外分享。

## 特点

|              |                                                                         |
| ------------ | ----------------------------------------------------------------------- |
| **实时同步** | 基于 Firebase Realtime Database，任意一端操作，全端即时可见             |
| **可撤回**   | 所有操作（含清零、导入）都进历史栈，`Ctrl+Z` 可逐步回退                 |
| **步进可调** | `±1 / ±5 / ±10` 一键切换，选择被记住                                    |
| **直接设值** | 输入目标数字立即生效，适合开局直接定基准值                              |
| **快照分享** | 把当前计数编码成一段同步码，通过聊天工具发给对方导入即可对齐            |
| **双主题**   | 浅色/深色自动跟随系统，可手动覆盖                                       |
| **无障碍**   | 语义化标签、`aria-live` 播报、键盘全操作、尊重 `prefers-reduced-motion` |
| **离线可见** | 最近计数缓存在本地，网络抖动时不会白屏回到 0                            |
| **无构建**   | 原生 HTML/CSS/JS，无打包器、无框架、无 `npm install` 即可运行           |

## 快速开始

### 直接用（无需任何构建）

```bash
git clone https://github.com/mingongye/jishuqi.git
cd jishuqi
open index.html          # 双击 index.html 也可以
```

> 页面通过 `file://` 直接打开也能正常工作（脚本刻意使用普通 `<script>` 而非 ES Module，
> 以避开 `file://` 协议下的 CORS 限制）。

### 本地起服务（推荐，便于多标签页联调同步）

```bash
npm run dev              # http://localhost:4173
```

然后打开两个标签页，就能看到**一边改、另一边同步跳动**的效果。

### 多人一起用

1. 把本仓库部署到任意静态托管（详见 [自行部署](#-自行部署)）
2. 把链接发给别人
3. 所有人打开同一个链接即自动接入同一个计数

## 键盘快捷键

| 按键                    | 作用           |
| ----------------------- | -------------- |
| `空格` / `↑` / `+`      | 按当前步进增加 |
| `↓` / `-` / `Backspace` | 按当前步进减少 |
| `Ctrl/⌘ + Z`            | 撤回上一次操作 |
| `Ctrl/⌘ + 0`            | 清零（可撤回） |

> 输入框获得焦点时快捷键自动让位，不会误触发。

## 项目结构

```
jishuqi/
├── index.html            # 页面结构（语义化 + 无障碍标注）
├── css/
│   ├── tokens.css        # 设计令牌：所有颜色/阴影/圆角的唯一定义处
│   ├── base.css          # reset、骨架、无障碍工具类
│   └── components.css    # 组件样式（不含任何色值字面量）
├── js/
│   ├── config.js         # 全局配置：Firebase 凭据、路径、上限、里程碑
│   ├── utils.js          # 纯函数：同步码编解码、数字解析、本地存储
│   ├── store.js          # 数据层：唯一持有计数与历史，封装 transaction
│   ├── ui.js             # UI 层：渲染与交互反馈，不做数据计算
│   ├── shortcuts.js      # 键盘绑定，独立成模块
│   └── app.js            # 入口：只负责组装
├── assets/               # favicon / PWA manifest
├── tools/                # 零依赖校验脚本（CI 使用）
└── .github/workflows/    # 持续集成
```

**分层原则**：`store` 不认识 DOM，`ui` 不做业务计算，`utils` 不持有状态。
新增功能时先想清楚它属于哪一层，几乎不需要跨层修改。

## 自行部署

仓库内置了一套 Firebase 配置，可直接使用。若要部署自己的实例：

1. 在 [Firebase 控制台](https://console.firebase.google.com/) 新建项目
2. 左侧 **实时数据库 (Realtime Database)** → 创建数据库 → 选区域
3. **项目设置 → 常规** → 复制 Web 配置
4. 打开 `js/config.js`，替换 `firebase` 字段
5. 数据库规则设为：

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

6. 部署静态托管：

```bash
# GitHub Pages：把仓库 Settings → Pages → Source 选 main 分支根目录即可
# 任意静态服务
npx serve .
```

> **说明**：当前配置为公开测试模式，任何人都可读写 `counter` 节点。
> 这适合内网、小圈子、一次性活动等轻量场景；若要长期公开运营，
> 建议先阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 中的加固建议。

## 常见问题

**Q：点了按钮没反应 / 顶部显示"本地模式"？**
A：`js/config.js` 里的 Firebase 配置未填写或无效。此时页面会自动读取本地快照，
避免白屏，但不会跨设备同步。填写配置后刷新即可。

**Q：同步码怎么用？**
A：点 `🔄` 生成当前计数的同步码 → 通过任意渠道发给对方 → 对方粘贴到文本框 →
点 `📥` 导入。全局计数会被覆盖，且这一步可以 `Ctrl+Z` 撤回。

**Q：多人同时操作会冲突吗？**
A：所有写操作都走 Firebase `transaction()`，服务端串行执行，不会出现数字错乱。

**Q：为什么不用框架？**
A：这类工具的核心诉求是「打开就能用、十年后还能打开」。不引入构建链，
就没有依赖腐烂、没有版本升级负担，把 HTML 传到任何地方都能运行。

## 参与贡献

见 [CONTRIBUTING.md](CONTRIBUTING.md) —— 包含**代码结构约定、提交前检查清单、
常见修改场景的操作指引**。

## 许可

[MIT](LICENSE) © mingongye
