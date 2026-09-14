# 分区纹理示例主题（theme-zones）——LinkDesk 插件仓

> **本文件是给在这个仓里干活的 AI 看的**（Claude Code / Codex / Cursor / …）。人看 `README.md`。
> 插件身份的唯一来源 = `plugin.json` 顶层的 `pluginId`（本仓：`theme-zones`）。当前版本 `1.0.2`。

## 1. 这是什么

分区纹理示例主题——演示 **per-surface 背景**两条路：纸纹分区（`surface.texture` 平铺纹理）与影像分区（`background.mode: zones` 连续切片）。

**用户在哪看到它**：外观主题：在设置 → 外观里切换（本仓声明了**两只**主题）。

**🔴 本仓就是这只插件的真源。** 插件源码后来从壳仓整体外移，现在**壳仓没有它的源码**了——
改这只插件，只能在这个仓里改；壳仓那边只有它**已发布的产物**（`bundled-plugins/` 里的 zip 或官方目录的条目）。

> 这只插件**不随软件出厂**（`seed: false`）：用户从插件市场装上它。发版照常走下面的两步。

## 2. 铁律（破了就坏插件，或者坏壳）

1. **颜色一律走 `var(--xxx)`** —— 禁止硬编码 hex，否则切主题时你的 UI 不跟。
2. **UI 文案一律走 `t()`**（key = 原文；英文译文放 `i18n/en.json`）—— 禁止硬编码显示字符串。
3. **系统能力只走 `window.linkdesk.*`** —— 不要 `import` 壳内部（`@src/core/...`）；SDK 的 lint 会判这条。
4. **插件身份只来自 `plugin.json` 的声明** —— 不要让任何人从目录名 / 文件位置去推断它是什么。
5. **右键菜单声明式**（`contributes.menus` ＋ `<ContextMenu>`）；**弹窗 portal 到 `document.body`**；**持久化走 `window.linkdesk.configuration`**（不要 `localStorage`）。
6. **keep-alive：每个标签页始终挂载** —— 不要用 `isActive` 把内容整块 blank 掉；它只用来 gate「聚焦才跑」的副作用。

## 3. 本仓的结构与关键路径

两份配方在**仓根**：`paper-zones.json` / `image-zones.json`。
🔴 **已知偏差**：作者文档要求「配方只住 `themes/`、不许摊到仓根」，本仓是外移前的历史形态。改它要动路径 ＋ 发版，记在账上。

**本仓没有 `src/`** —— 它是「数据插件」：能力全在 `plugin.json` 的声明 ＋ 数据文件里。

**本仓没有 `i18n/`** —— 文案 key 就是中文原文，英文由语言包插件（`lang-defaults`）提供。

- **全仓唯一声明两只主题**的插件（`contributes.themes` 两条）——它是「一只插件可以带多套主题」的活样本。
- 资源：`resources/paper-texture.svg`（纸纹平铺图）、`resources/zones-bg.svg`（分区切片图）、`icon.svg`。

## 4. 规矩去哪找

- **在线（作者文档，按「我想做什么」组织）**：<https://github.com/Encaron/linkdesk/tree/electron/docs/03-plugin-authoring>，从 `00-readme.md` 进。
- **离线（永远可用，不用联网）**：`node_modules/@linkdesk/plugin-sdk/schemas/plugin.schema.json` —— **字段级权威**；同目录的 `theme.schema.json` 管主题配方。
- **编辑器补全**：`plugin.json` 的 `$schema` 指向它，打字就有补全与诊断。
- **改完自查**：`npm run validate`（清单 / 格式 / 声明的文件在不在）＋ `npm run lint`（SDK 规则腿）。
- 中文版作者文档（维护者面原文）：<https://github.com/Encaron/linkdesk/tree/electron/docs/03-插件制造>

## 5. 本仓的命令

```bash
npm run build      # 产出 `<pluginId>.linkdesk-plugin`（装进 LinkDesk / 发布都用它）
npm run validate   # 校验 plugin.json / 主题配方 / 声明的字典文件真的在
npm run lint       # SDK 规则腿（硬编码颜色 / 字号 / 4px 网格 / 自定义 eslint 规则）——**只报告、不拦**
npm run publish    # 发版到本仓自己的 GitHub Release（**上架两步里的第一步**）
npm run verify     # 🔴 **交付前严格腿** = 本仓 CI 跑的那条（lint 判红 + 跨插件 import + 字典完整性 + 声明自洽）
```

## 6. 发布与版本纪律

**🔴 上架是两步，别只做第一步**：`npm run publish` 只写**本仓自己的** GitHub Release
（只有手动加过本仓地址的人看得见）；**第二步是收录进官方目录**——只有收录之后，
默认设置的全体用户才找得到它。两步都做完，才算「上架」。

**版本号**：`plugin.json` 的 `version` 与 `package.json` 的 `version` **必须同值**；
且每次 bump 都要在 `CHANGELOG.md` 里配一段 `## v<新版本>（YYYY-MM-DD）`——
缺了这段，市场详情页的「更改日志」页签会是空的。

## 7. 本仓自带的门禁

- `.github/workflows/ci.yml` —— push / PR 时跑 `validate` → `verify` → `build` → `test`。
- `scripts/ci-verify.mjs`（`npm run verify`）—— **严格腿**：SDK lint 全腿**判红** ＋ 跨插件 import ＋
  字典完整性 ＋ 声明自洽。⚠️ SDK 自带的 `npm run lint` 是**只报告不拦**的（那是有意给作者本地留的），
  **别把 `verify` 里的这段删了换成 `npm run lint`**。
- 🔴 **壳仓的 `npm run check` 够不着本仓**（源码搬出去之后就不在它的扫描域里了）——
  本仓的绿灯只由本仓的这两条腿给出。
