# Performance Audit —— Lighthouse 评分 90+，不破坏追踪和转化

> 从 **Google PageSpeed Insights / Lighthouse** 角度对页面进行性能审计，同时不牺牲布局、追踪、UTM、SEO 或转化。


---

## 它做什么

把 **Lighthouse 测量的东西** 和 **用户真实感受到的东西** 分开的性能审计，按以下维度排优先级：

- 对正确指标（LCP、CLS、INP、TBT、FCP）的可衡量影响。
- 现实的实现成本。
- 显式风险（破坏布局？破坏追踪？破坏转化？）。
- 修复后的测试计划。

**它不是**盲目优化。每条建议都保留：

1. 视觉布局。
2. 追踪（Pixel、GTM、GA4、Hotmart、AC、Make、转化）。
3. UTM 和 URL 参数。
4. 表单功能。
5. SEO（meta、structured data、canonical、hreflang）。
6. 基本可访问性。

## 运行模式

- **`audit`** —— 审计已有页面（URL、代码、Lighthouse 报告）。
- **`build`** —— 部署前审查代码，从一开始就保证性能。

## 何时使用

**总是**在以下情况使用：
- 需要审计页面（landing、WP/Elementor、HTML/Tailwind、Next.js、引流页）。
- 在开发新页面，想从一开始就保证好分数。
- 任何公开页面部署之前。
- 在调查 Core Web Vitals 下降。
- 在调查与速度相关的可疑转化下滑。

用 `/performance-audit` 或"审计一下这个页面的性能"来调用。

## 安装

```bash
# 1. 克隆到 ~/.claude/skills/
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/brunnocarpena/performance-audit-skill.git performance-audit

# 2.（可选但推荐）配置 PageSpeed Insights API key
# 无 key：每个 IP 大约 1 次查询/天。有 key：400 QPS / 25k 查询/天。
#
#   - https://console.cloud.google.com/apis/credentials → 创建 API key
#   - 启用 "PageSpeed Insights API" 和 "Chrome UX Report API"
#   - 在 ~/.zshrc（或 ~/.bashrc）添加：
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

验证安装：

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # 应该存在
echo $PSI_API_KEY                                # 应该显示 key（可选）
```

在任意 Claude Code 会话中：

```
/performance-audit https://example.com --mode=audit --target=mobile
```

## 何时**不要**用

- SEO 内容审计 → `seo-audit`。
- WCAG 可访问性审计 → 专门的 skill。
- 安全审计 → `security-auditor`。
- 与性能无关的视觉 bug。

## 结构

```
performance-audit/
├── SKILL.md                          # 主指令、Iron Law、workflow
├── README.md                         # 本文件
├── references/                       # 深入指南
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   如何收集 lab + field data
│   ├── severity-rubric.md            #   关键/高/中/低 评级
│   ├── safe-fixes.md                 #   按风险分类的修复
│   └── report-format.md              #   报告模板
└── checks/                           # 按领域的审计
    ├── hero-and-lcp.md
    ├── images.md
    ├── fonts.md
    ├── css.md
    ├── javascript.md
    ├── third-party-tracking.md
    ├── wordpress-elementor.md
    ├── layout-stability.md
    └── network-server.md
```

## 工作方式

1. 识别范围和模式（audit vs build）。
2. 收集证据：Lighthouse + PageSpeed + CrUX + 代码。
3. 分到 12 个类别（hero、图片、字体、CSS、JS、第三方、WP、CLS、网络）。
4. 标注受影响的指标 + 严重度 + 优先级 + 风险。
5. 推荐**安全**的修复，始终给出文件/行号和不要同时动什么。
6. 出标准化报告。
7. 部署前决策：批准 / 附保留意见 / 阻止。

## 不可违反的规则

1. **LCP 图片绝不带 `loading="lazy"`** —— 始终 `fetchpriority="high"`。
2. **未经用户明确确认绝不动追踪**。
3. **lab 和 field data 永远分开** —— Lighthouse 分数不是绝对真理。
4. **mobile 和 desktop 永远分开** —— 是 Google 不同的排名。
5. **重定向/canonical 中 UTM 绝不丢弃**。
6. **每条建议都引用文件/行号或 audit ID**。
7. **默认 mobile-first**。
8. **报告永远生成**，即使全绿。
9. **单独看 score 不能放行部署** —— field 上 Core Web Vitals "good" 才是标准。
10. **每个有风险的修复都有回滚计划**。

## 指标和阈值（2026）

| 指标 | 良好 | 需要改进 | 差 |
|------|------|----------|-----|
| LCP | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| CLS | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| INP | ≤ 200ms | 200–500ms | > 500ms |
| TBT | ≤ 200ms | 200–600ms | > 600ms |
| FCP | ≤ 1.8s | 1.8–3.0s | > 3.0s |
| TTFB | ≤ 800ms | 800–1800ms | > 1800ms |

Google 标准：**CrUX 中 75% 用户的 p75 ≥ "good"**。

## 灵感与参考

结构灵感来自：
- [brunnocarpena/congruence-skill](https://github.com/brunnocarpena/congruence-skill) —— 强制证据 + Iron Law 模式。
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) —— Chrome 团队官方为 Lighthouse + CWV 出的 Agent Skills。
- [web.dev/articles/vitals](https://web.dev/articles/vitals) —— 阈值的权威来源。
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) —— PR 中的 budget 和回归。

## 相关 skills

- **congruence** —— 校验页面上的声明是否和代码一致。如果审计改动了 copy/CTA，之后使用。
- **seo-audit** —— 技术和 on-page SEO。互补。
- **ux-ui-perfectionist** —— UI/UX 审查。如果优化触及布局，配合使用。
