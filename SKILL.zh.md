---
name: performance-audit
description: 每当需要从 Google PageSpeed Insights / Lighthouse 的角度审计页面（landing 页、WordPress/Elementor、HTML/Tailwind、Next.js、引流页）的性能时使用，或者在开发新页面并希望从一开始就确保良好评分时使用。TRIGGER：当 agent 提到 "PageSpeed"、"Lighthouse"、"Core Web Vitals"、"LCP"、"CLS"、"INP"、"TBT"、"performance"、"页面慢"、"score 低"、"优化页面"；当会话涉及 hero、图片、字体、关键 CSS、GTM/Pixel/Analytics、缓存插件、Elementor 等文件；以及任何公开页面部署之前。SKIP：纯内部改动（API、jobs、schema）且不影响前端渲染时。
argument-hint: [url或范围?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

从 **Google PageSpeed Insights / Lighthouse** 角度对页面进行性能审计，同时不牺牲布局、追踪、UTM、SEO 或转化。

**本 skill 不是盲目的优化。** 它是一种有判断的审计，区分 Lighthouse 测量的东西和用户真实感受到的东西，优先处理真正带来影响、又不会把页面搞坏的修复。

## 核心原则

```
性能不是分数。性能是把首屏快速送达、
保持视觉稳定、并且不破坏追踪和转化。
```

一个得 100 分但表单坏掉、Pixel 不触发、UTM 丢失的页面**是一次倒退**，不是胜利。本 skill 的每一条建议都必须保留：

1. **视觉布局** —— 不引入 CLS，不破坏网格/组件。
2. **追踪** —— Meta Pixel、GTM、GA4、Hotmart、ActiveCampaign、Make、转化事件。
3. **UTM 和 URL 参数** —— 重定向/canonical 中绝不丢弃 query string。
4. **表单功能** —— 字段、校验、action、集成。
5. **SEO** —— meta 标签、structured data、hreflang、canonical、robots。
6. **基本可访问性** —— alt、label、对比度、焦点。

如果某项"优化"会把以上任何一点置于风险中，**没有明确预警就不要推荐**。

## 运行模式

本 skill 有两种模式。一开始就识别属于哪种：

### `audit` 模式 —— 审计已有页面

接受的输入：
- 公开 URL。
- PageSpeed Insights 的截图/JSON。
- Lighthouse 报告（HTML/JSON）。
- 源代码（HTML、Elementor 模板、Next.js 组件）。
- 局部代码片段（hero、head、footer）。

输出：按优先级排列的诊断 + 修复计划（见 [references/report-format.md](references/report-format.md)）。

### `build` 模式 —— 开发新页面

在代码尚未成型时**提前**应用最佳实践。在提交时审查 HTML/CSS/JS。按 [checks/](checks/) 中的检查项验证 hero、图片、字体、脚本和追踪。

输出：部署前 checklist + 当前代码中的风险点。

## Workflow

### 1. 识别范围和模式

- 如果有 URL 或 Lighthouse 报告 → `audit`。
- 如果正在编辑页面/组件文件而没有报告 → `build`。
- 如果不明确，只问**一次**："你是想审计已有页面，还是部署前审查代码？"

### 2. 收集证据

**强制前置检查 —— PageSpeed Insights API key**

任何收集之前，先检查 `$PSI_API_KEY` 是否已配置：

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY 缺失"
fi
```

**如果变量未设置，暂停并用以下文本提醒用户**（语气根据对话调整，内容保持）：

> ⚠️ 你还没有配置 PageSpeed Insights 的 API key（`$PSI_API_KEY`）。没有它，我会被限制到每个 IP 大约 1 次查询/天 keyed —— 配额很快就耗尽，逼我只能做纯静态审计（没有 score、没有 CrUX 字段数据、没有 Lighthouse audit）。
>
> **配置了 key 之后，我能精确得多：** 我可以拿到真实的 Performance Score、实际测量的 LCP/CLS/INP/TBT/FCP、按优先级排好的 opportunities、main thread 拆解、识别出的重脚本，以及最近 28 天真实用户的字段数据（CrUX）。
>
> 配置（2 分钟）：
> 1. https://console.cloud.google.com/apis/credentials → 创建 API key
> 2. 在 APIs & Services → Library 启用 **PageSpeed Insights API** 和 **Chrome UX Report API**
> 3. 在 `~/.zshrc` 添加：
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` 或打开新终端
>
> 你想现在就配置，还是希望我先做受限的静态审计？

如果用户选择配置：等待并用测试 `curl` 验证。如果选择不配置直接继续：在最终报告中标注 **"部分审计 —— 没有 PSI/CrUX 数据"**，结论仅基于静态分析（HTML/headers/assets）。

**实际收集：**

| 来源 | 提取什么 |
|------|----------|
| 公开 URL | 运行 `curl -sI` 看 headers，下载 HTML，检查 `<head>`、脚本顺序、hero 图片 |
| Lighthouse 报告 | 指标（LCP/CLS/INP/TBT/FCP/SI）、top opportunities、diagnostics、JS treemap |
| 代码 | hero、`<link>`/`<script>` 顺序、图片属性、字体、关键 CSS、插件（WP）、bundle（Next） |
| PageSpeed Insights | Lab data **和** field data（CrUX）分开，mobile **和** desktop 分开 |

WebFetch / curl 公开 URL 的方式见 [references/data-collection.md](references/data-collection.md)。PSI/Lighthouse/CrUX 的机制（lab×field、score 权重、INP、soft-nav、throttling、PSI API v5）见 [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md)。

### 3. 按类别分类

所有分析按 **12 个类别**组织（见 [checks/](checks/)）：

1. 执行摘要
2. PageSpeed 指标（LCP、CLS、INP/TBT、FCP、SI）
3. 首屏（hero、headline、CTA、关键 CSS）
4. 图片
5. 字体
6. CSS
7. JavaScript
8. 第三方与追踪
9. WordPress / Elementor
10. 视觉稳定性（CLS）
11. 网络与服务器
12. 行动计划

### 4. 评估影响和严重度

每个问题都要标注：
- **影响的指标**（LCP、CLS、INP、TBT、FCP、SI、TTFB）。
- **严重度**（关键 / 高 / 中 / 低）—— 见 [references/severity-rubric.md](references/severity-rubric.md)。
- **优先级**（P0 / P1 / P2）—— 基于影响 × 修复成本。
- **修复风险**（会破坏布局？破坏追踪？需要视觉回归？）。

### 5. 推荐安全的修复

每个问题都要给出：
- 具体改什么（文件 / 选择器 / 属性）。
- 为什么这会影响指标 X。
- 同时**不要**动什么。
- 之后如何测试（Lighthouse run、WebPageTest、DevTools Performance、视觉回归）。

安全修复 vs. 风险修复清单：[references/safe-fixes.md](references/safe-fixes.md)。

### 6. 出报告

强制模板：[references/report-format.md](references/report-format.md)。

### 7. 部署前决策

- 任何**关键**问题未解决 → **阻止部署**。
- hero/LCP 上有多个**高**严重度问题 → **阻止**，直到 top 3 修复。
- 仅有**中/低** → **附保留意见批准**。
- 全绿 + Core Web Vitals 在阈值内 → **批准**。

## 指标和阈值（快速参考）

2026 年 Core Web Vitals 数值（与 Google CrUX 对齐）。有疑问时始终对照 [web.dev/vitals](https://web.dev/articles/vitals) 验证。

| 指标 | 良好 | 需要改进 | 差 |
|------|------|----------|-----|
| **LCP**（Largest Contentful Paint） | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **CLS**（Cumulative Layout Shift） | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |
| **INP**（Interaction to Next Paint） | ≤ 200ms | 200ms – 500ms | > 500ms |
| **TBT**（Total Blocking Time, lab） | ≤ 200ms | 200ms – 600ms | > 600ms |
| **FCP**（First Contentful Paint） | ≤ 1.8s | 1.8s – 3.0s | > 3.0s |
| **Speed Index** | ≤ 3.4s | 3.4s – 5.8s | > 5.8s |
| **TTFB** | ≤ 800ms | 800ms – 1800ms | > 1800ms |

详情见 [references/metrics-glossary.md](references/metrics-glossary.md)。

## Lab Data vs. Field Data —— 永远要区分

- **Lab data（Lighthouse）** —— 固定 throttling 下的模拟（Moto G4 / 慢速 4G）。适合诊断。
- **Field data（CrUX）** —— 最近 28 天 Chrome 真实用户数据。适合真实优先级排序。
- **常见差异：** Lighthouse 显示 LCP 4s，但 CrUX 显示 2.1s —— 真实用户没问题，是 Lighthouse 分数有问题。**基于 field 决策，用 lab 验证。**
- Mobile 和 desktop **在 Google 中是分开排名的**。永远两个都要评估。

## Red Flags —— 立即 STOP

如果你发现自己在想或在推荐：

- "把所有脚本都用 `defer` 延迟" —— 没有把追踪/inline 的依赖关系理清。
- "所有图片都加 lazy-load" —— 包括 LCP 那张图。
- "Pixel/GTM 太重，直接去掉" —— 没和 marketing 对齐。
- "把 Elementor 换成原生代码" —— 没理解编辑周期。
- "把 hero 压到 30KB" —— 牺牲了 headline 的视觉质量。
- "自动删除未使用的 CSS" —— 可能破坏状态样式（hover、modal、error）。
- "Lighthouse 100 分就是目标" —— 目标是 field data 上 Core Web Vitals 都"good"。
- "迁到 Next.js / 框架 X 就解决了" —— 没有理解真正问题。

→ STOP。回到第 4 步（影响 × 风险）再推荐。

## 借口 | 现实

| 借口 | 现实 |
|------|------|
| "Lighthouse 只在慢速 mobile 跑，不真实" | 这是 Google 排名用的。Field data 会印证。 |
| "Score 80 已经挺好了" | Core Web Vitals 是二元的（过/不过）。80 可能在 LCP 不过。 |
| "我全优化了，score 反而下降了" | 你大概率把 inline 追踪或 LCP 弄坏了。重新审计。 |
| "客户要 100 分" | 100 很脆弱。目标是稳定的"全绿 CWV"。教育客户。 |
| "我用插件 X 它什么都做" | 插件做 60%、坏 30%。审计结果，不是审计插件本身。 |
| "mobile 不重要，受众都是 desktop" | Google 的 mobile-first index 按 mobile 排名。 |
| "我去掉了 custom 字体，速度上来了" | 转化也跟着掉了。优化字体，不要去掉。 |

## 按领域的 Checks

当会话触及这些文件/区域时，使用对应的 check 文件：

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) —— 首屏、LCP 图片、headline。
- [checks/images.md](checks/images.md) —— 格式、尺寸、lazy、preload、响应式。
- [checks/fonts.md](checks/fonts.md) —— Google Fonts、本地字体、`font-display`、preload、fallback。
- [checks/css.md](checks/css.md) —— render-blocking、关键 CSS、unused CSS、Tailwind、Elementor CSS。
- [checks/javascript.md](checks/javascript.md) —— defer/async、bundle、hydration、不必要的库。
- [checks/third-party-tracking.md](checks/third-party-tracking.md) —— Pixel、GTM、GA4、Hotmart、AC、Make、客服聊天。
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) —— 插件、全局 assets、缓存、LiteSpeed、Cloudflare。
- [checks/layout-stability.md](checks/layout-stability.md) —— CLS、预留空间、字体 swap、embed。
- [checks/network-server.md](checks/network-server.md) —— 缓存、CDN、Brotli、TTFB、preconnect。

## 不可违反的规则

1. **绝不在 LCP 图片上推荐 lazy-load** —— 始终 `fetchpriority="high"` + 不加 `loading="lazy"`。
2. **不经用户确认绝不移动或删除追踪脚本** —— 可能破坏归因。
3. **始终区分 lab 与 field data** —— 不把 Lighthouse 分数当成绝对真理。
4. **始终分开报告 mobile 和 desktop** —— 是两套不同排名。
5. **绝不用 `display: none` 来掩盖 CLS 问题** —— 通过预留空间来修。
6. **绝不在重定向或 canonical 优化中丢弃 UTM/query string**。
7. **每条建议都要指明文件/行号（当适用时）** —— 不要笼统的"优化下 CSS"。
8. **所有分析默认 mobile-first** —— desktop 是补充。
9. **报告永远生成** —— 即使全绿。
10. **单独看 score 不是放行标准** —— field 上的 Core Web Vitals 才是。

## 何时**不要**用

- SEO 内容/关键词审计 → 用 `seo-audit`。
- 深度可访问性审计（WCAG AA/AAA）→ 专门的 a11y skill。
- Header 安全审计 → `security-auditor` / `seguranca`。
- 与性能无关的视觉渲染 bug → 普通调试。
- 后端 / API 重构 → 不在范围内。

## 相关 skills

- **congruence** —— 校验 landing 上的声明是否和代码真实行为一致。在 performance-audit 改动了 copy/CTA 之后使用。
- **seo-audit** —— 技术与 on-page SEO 审计。互补。
- **ux-ui-perfectionist** —— UI/UX 审查。如果优化触及布局，配合使用。
- **ai-seo** —— 针对 LLM search / AI Overviews 的优化。互补。

## 为什么有这个 skill

性能优化已经变成了民俗：团队换框架、买插件、开 CDN 的"涡轮模式"，分数涨了 5 分，转化却跌了 20%。本 skill 的存在是为了**把 Lighthouse 测量的东西和用户真实感受到的东西分开**，并保证每一次修复都：

- 在报告/代码里有具体证据。
- 优先级基于影响 × 成本。
- 有针对布局、追踪和功能的回归测试。
- 不是 cargo cult（"听说 defer 能改善"）。

最终目标不是分数。是**那种对真实用户加载够快、保住转化、还顺便通过 Core Web Vitals 的页面**。
