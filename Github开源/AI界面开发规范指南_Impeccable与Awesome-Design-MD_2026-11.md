# Impeccable × Awesome DESIGN.md 分析与落地指南

> 数据抓取时间：2026-11-06，来源为 GitHub REST API 与仓库原始文件（README / SKILL.md / spec / 样例 DESIGN.md）。
> Star 数会变，结论不会：这两个仓库解决的是**两个不同环节**的问题。

---

## 0. 一页结论

| | `pbakaus/impeccable` | `VoltAgent/awesome-design-md` |
|---|---|---|
| Star / Fork | 104,098 / 6,168 | 125,815 / 13,932 |
| License | Apache-2.0 | MIT |
| 创建时间 | 2025-11-16 | 2026-03-31 |
| 形态 | Agent Skill 包 + 确定性检测引擎 | 只读内容库（84 份 `DESIGN.md`） |
| 解决的问题 | Agent **不知道什么叫好设计**，做完**没人审** | Agent **不知道你的产品长什么样** |
| 在流程中的位置 | **过程**：怎么做、怎么改、怎么验 | **输入**：给它一份可读的设计规范 |
| 使用方式 | `npx impeccable install` | 复制一个 `DESIGN.md` 进项目根目录 |
| 可复用性 | 方法论（可迁移到任何项目） | 素材（一次性的起点） |

**一句话**：impeccable 是「方法论 + 质检器」，awesome-design-md 是「上下文素材库」。impeccable 的 `document` 命令产出的正是后者那种 `DESIGN.md`，所以两者天然串联。

---

## 1. 仓库一：pbakaus/impeccable

### 1.1 它是什么（不是什么）

**不是** UI 组件库、**不是** Figma 插件、**不是**代码生成器。

它是一套装进编码 Agent 的 **Skill**，由三部分组成：

| 组成 | 体量 | 作用 |
|---|---|---|
| `.agent/skills/impeccable/SKILL.md` | 11.8 KB | 总纲：触发条件、四种 surface 模式、24 个命令的路由表 |
| `reference/*.md` | 40+ 文件 | 每个命令一份 playbook。`critique.md` 45.6 KB，`new-work.md` 62.9 KB |
| `scripts/impeccable`（自包含二进制） | — | 上下文加载、token 提取、62 条规则的确定性检测器 |

**核心价值在 `reference/`**：它把「把这个做得高级点」这种模糊指令，翻译成了有步骤、有检查项、有反面清单的执行手册。

> 实用提醒：仓库体积约 877 MB（含演示素材、浏览器脚本、字体索引），**不要 clone**，用 `npx` 安装即可。

### 1.2 安装

```bash
npx impeccable install
```

安装器会：检测你装了哪些编码工具 → 问装项目级还是全局 → 在支持的工具上提供 design hook（**务必接受**）。

官方声明支持：Claude Code、Cursor、Codex CLI、GitHub Copilot、Devin、Replit Agent、Gemini CLI、Grok Build、OpenCode、Google Antigravity、Kiro、Trae、Rovo Dev、Qoder、Mistral Vibe、Pi、Hermes Agent、**DeepSeek Harness**、Veto。

装完后在 Agent 里执行 `/impeccable init`（Codex 用 `$impeccable init`）。

### 1.3 四种 surface 模式（最值得抄的设计）

它先判断「这是什么页面」，再决定「该多大胆」：

| 模式 | 场景 | 设计策略 |
|---|---|---|
| **Persuade** | 落地页、营销、定价页 | 设计就是产品，要够狠、要有观点 |
| **Operate** | 后台、控制台、编辑器、设置 | 可扫读、一致性、原生预期优先；野心体现为**精准和克制** |
| **Read** | 文档、博客、帮助中心 | 结构服务理解，再让阅读体验值得停留 |
| **Experience** | 作品集、画廊、展示 | 作品本身唱主角，界面后退 |

**模式由「surface」决定，不由「产品」决定**：一个工具的落地页仍然是 Persuade，一个时尚品牌的文档仍然是 Read。

这一条直接解决了 AI 最高频的错误：**把后台管理系统做得像营销落地页**。

### 1.4 24 个命令

| 分组 | 命令 | 说明 |
|---|---|---|
| **建立上下文** | `init` | 写 `PRODUCT.md`（产品真相） |
| | `document` | 从现有代码反推 `DESIGN.md` |
| | `extract` | 抽取可复用 token 与组件 |
| | `shape` | 写码前先规划 UX |
| **调风格** | `bolder` | 放大过于保守的设计 |
| | `quieter` | 收敛过度刺激的设计 |
| | `distill` | 删到只剩本质 |
| | `colorize` | 给灰白界面加策略性色彩 |
| | `typeset` | 修字体、层级、字号 |
| | `layout` | 修间距、节奏、视觉层级 |
| | `animate` | 加有目的的动效 |
| | `delight` | 加个性与记忆点 |
| | `overdrive` | 上 shader、弹簧物理、滚动驱动 |
| **评审** | `critique` | UX 启发式打分，排序「先修什么」 |
| | `audit` | a11y / 性能 / 主题 / 响应式，按 P0–P3 分级 |
| **收尾** | `polish` | 发布前最后一次质量过一遍 |
| | `harden` | 错误态、i18n、溢出、边界情况 |
| | `clarify` | 改写不清楚的文案与报错 |
| | `adapt` | 多端适配 |
| | `optimize` | 加载与渲染性能 |
| | `onboard` | 首启流程、空状态、激活 |
| **浏览器迭代** | `live` | 页面里选中元素，热换变体，接受后写回源码 |
| | `generate` | 对指定元素批量出方案供挑选 |
| | `craft` | 已废弃，等价于普通的新建请求 |

**用法**：`/impeccable critique 定价卡片`。目标越窄越好——**一个 section 细看胜于整页扫一眼**。

常用命令可以 pin 成独立快捷键：`/impeccable pin audit` → 之后直接 `/audit`。

### 1.5 craft floor：把「AI 味」写成可检查的条目

`reference/craft-floor.md` 是它最精华的部分。摘录：

**视觉禁令（bans，任何 brief 都不给翻案）**
- 标题上方的 **kicker / eyebrow 小字**
- 卡片套卡片

**默认不用（brief 明确要才可以）**
- 渐变文字（强调应该靠字重或字号）
- 玻璃拟态当装饰而非具体效果
- 等大卡片「图标 + 标题 + 正文」撑起整页结构
- Hero 大数字模板（大数 + 小标签 + 辅助指标 + 强调色）
- section 编号 `01 / 02 / 03`（除非顺序本身携带信息）
- 硬偏移阴影（`box-shadow: 4px 4px 0`），除非世界设定真的是新野蛮主义
- 用 unicode / emoji 冒充图标系统
- 把等宽字体当「技术感」的服装（只该用于代码、数据、度量）
- 用系统字体当自有世界的 display voice

**硬性检查（`audit` 会看，hook 会拦）**
- 正文与 placeholder 对比度 ≥ 4.5:1，大字 ≥ 3:1
- **彩色表面上，次要文字必须从该色相 tint，绝不能直接灰**
- 阴影必须带偏移 + 柔和模糊；零偏移彩色光晕 = 装饰
- 正文行宽 65–75ch；display 最大 6rem；tracking 下限 -0.04em
- 必须覆盖：hover / disabled / loading / error / empty + 键盘焦点 + **真实内容**
- **浏览器原生表面**：文字选区、caret、自定义滚动条、focus ring、下划线偏移、表格等宽数字 —— 这些出厂样式不属于任何设计系统，必须从色板里重新定义
- 动效：**一个作者署名的时刻**，不是每段都同一个入场动画
- 文案：控件命名其动作；报错命名问题与恢复路径

> 「浏览器原生表面」这一条是最高性价比的：**90% 的 AI 页面会漏，而它是最强的「这页是人做的」信号。**

### 1.6 检测器的三种跑法

```bash
# ① Agent 内（安装时勾选 hook）
#    每改一个 UI 文件自动扫，findings 直接回喂给 Agent，在它说「完成了」之前拦住

# ② 命令行 / CI
npx impeccable detect src/                      # 扫源码（JSX/TSX 只跑模式类规则）
npx impeccable detect http://localhost:5173     # 扫渲染页（对比度、小字号类规则必须这样跑）
npx impeccable detect --json .                  # 机器可读
npx impeccable detect --github src/             # GitHub Actions 注解
npx impeccable detect --list-rules              # 列出全部 62 条规则
# 退出码：2 = 有 findings，1 = 目标扫描失败

# ③ Chrome 扩展
#    把同样的 findings 以 overlay 形式盖在任意真实页面上

# 例外管理（例外必须写理由，否则规则会烂掉）
npx impeccable ignores add-value overused-font Inter --reason "品牌字体"
<!-- impeccable-disable overused-font: 导出的品牌文档 -->
```

**关键坑**：只扫源码目录会漏掉一半规则。对比度、极小字号这类规则**依赖渲染结果**，CI 里必须额外扫一次 dev server 的 URL。

### 1.7 其他值得知道的

- **`live` 模式**：打开 dev server 加 overlay，选元素、换方向、原地热换，接受后写回源码。支持 Vite / Next / SvelteKit / Astro / Nuxt / 纯 HTML。
- **`doctor`**：检查 `PRODUCT.md` / `DESIGN.md` / surface brief / hook 配置与当前版本之间的漂移。
- **安全注意**：live 模式的 Apply 在无人轮询时，会调用 PATH 里的 `codex`（或 `claude`）CLI，且**关闭审批与沙箱**（`--dangerously-bypass-approvals-and-sandbox`）。在陌生仓库里用之前先看清楚。

---

## 2. 仓库二：VoltAgent/awesome-design-md

### 2.1 DESIGN.md 是什么

格式由 **Google Stitch** 提出，规范仓库为 [google-labs-code/design.md](https://github.com/google-labs-code/design.md)（28,754 star，Apache-2.0，2026-04-10 创建）。

它的定位一句话讲清：

| 文件 | 谁读 | 定义什么 |
|---|---|---|
| `AGENTS.md` | 编码 Agent | 项目**怎么构建** |
| `DESIGN.md` | 设计 Agent | 项目**应该长什么样** |

**就是纯 markdown + YAML front matter**，无 Figma 导出、无 JSON schema、无特殊工具链。扔进项目根目录，任何 Agent 都能读。LLM 最擅长读 markdown，所以这东西没有解析成本。

### 2.2 文件结构（规范强制）

**两层**：
1. YAML token（文件顶部 `---` 围栏，或各 `##` 章节内嵌 ` ```yaml ` 代码块）
2. Markdown 正文（`##` 章节，讲「为什么」）

```yaml
version: <string>          # 可选，当前 "alpha"
name: <string>
description: <string>
omitted: <string[]>        # 可选，声明故意省略的章节

colors:
  <token-name>: <CSS 颜色，hex/rgb/oklch/命名色均可>
typography:
  <token-name>:
    fontFamily / fontSize / fontWeight / lineHeight / letterSpacing / fontFeature / fontVariation
rounded:
  <scale-level>: <Dimension>
spacing:
  <scale-level>: <Dimension | number>
components:
  <component-name>:
    <token-name>: <值 或 {path.to.token} 引用>
```

**Token 类型**：Color / Dimension（`px em rem`）/ Token Reference（`{colors.primary}`）/ Typography 对象。

**组件可用属性**：`backgroundColor`、`textColor`、`typography`、`rounded`、`padding`、`size`、`height`、`width`。变体（hover/active/pressed）写成同一组件的独立条目。

**8 个章节顺序强制**（可省略，但出现的必须按序）：

| # | 章节 | 别名 |
|---|---|---|
| 1 | Overview | Brand & Style |
| 2 | Colors | |
| 3 | Typography | |
| 4 | Layout | Layout & Spacing |
| 5 | Elevation & Depth | Elevation |
| 6 | Shapes | |
| 7 | Components | |
| 8 | Do's and Don'ts | |

**未知内容的消费行为**：未知章节 → 保留、不报错；未知色/排版 token → 接受；未知组件属性 → 接受但告警；**重复章节标题 / 重复顶层 YAML key → 直接报错拒收**。

### 2.3 为什么这份语料值得看

抽查 `design-md/linear.app/DESIGN.md`（24 KB）：

```yaml
colors:
  canvas: "#010102"        # 整个合集里最深的暗面
  primary: "#5e6ad2"       # Linear 薰衣草蓝，唯一彩色强调
  ink-muted: "#d0d6e0"
  hairline: "#23252a"

typography:
  display-xl:
    fontFamily: Linear Display
    fontSize: 80px
    fontWeight: 600
    lineHeight: 1.05
    letterSpacing: -3.0px
```

**这才是重点**：不是「深色、科技感、紫色点缀」这种形容词，而是 `#010102` + `-3px` tracking + `1.05` 行高。

> AI 拿到形容词只能猜，拿到数字才能复现。

### 2.4 合集构成

84 个品牌目录，每个含 `DESIGN.md` + `README.md`。分布大致：

- **开发工具 / AI**：Linear、Vercel、Stripe、Raycast、Notion、Claude、Cohere、Cursor、Figma、Framer、Supabase、Resend、Sentry、PostHog、Mintlify、Warp、Mistral、Replicate、Together.ai、x.ai、Ollama、OpenCode
- **消费 / 品牌**：Apple、Nike、BMW、Ferrari、Bugatti、Starbucks、Spotify、PlayStation、Nintendo、Pinterest
- **金融**：Binance、Coinbase、Kraken、Revolut、Wise、Mastercard、Mercury
- **其他**：Airbnb、Uber、Shopify、Slack、Meta、NVIDIA、SAP、Oracle、Dell、HP、IBM

### 2.5 三个必须知道的坑

**① 商业化导流**
每个品牌目录的 `README.md` 现在只剩一句「详细设计系统已移至 https://getdesign.md/...」。`DESIGN.md` 正文目前仍完整保留在仓库里（我抽查是完整的 24 KB），但这是典型 freemium 结构，**未来可能被裁剪**。社区已有人做 `Meliwat/awesome-design-md-pre-paywall` 快照 fork。

**行动建议**：把需要的文件复制到自己仓库并版本化，不要每次都从上游拉。

**② 这是「逆向分析」，不是官方设计系统**
文件由社区从公开网站扒出并逆向整理。**定制字体（如 Linear Display）、插画、logo、摄影素材都不能直接用。** 只借：色板结构、尺度关系、间距节奏、组件规则。

**③ 同质化风险**
如果整个社区都在用这 84 份文件，做出来的东西会越来越像。**它是起点，不是终点。**

### 2.6 生态里顺带值得收的

| 仓库 | Star | 用途 |
|---|---|---|
| [bergside/design-md-chrome](https://github.com/bergside/design-md-chrome) | 3.1k | Chrome 扩展，从任意网站抽 token 生成 DESIGN.md |
| [VoltAgent/official-design-md](https://github.com/VoltAgent/official-design-md) | 530 | **第一方**发布的 DESIGN.md，比逆向版靠谱 |
| [Meliwat/awesome-ios-design-md](https://github.com/Meliwat/awesome-ios-design-md) | 632 | 移动端 200 份，含 SwiftUI / Jetpack |
| [fchangjun/awesome-design-md-cn](https://github.com/fchangjun/awesome-design-md-cn) | 149 | 中文场景补充 |
| [kzhrknt/awesome-design-md-jp](https://github.com/kzhrknt/awesome-design-md-jp) | 1.0k | 日文 UI 场景 |
| [Paidax01/web-to-design-md](https://github.com/Paidax01/web-to-design-md) | 446 | 任意官网 → design.md |

配套 CLI（来自规范仓库）：

```bash
npx @google/design.md lint DESIGN.md                 # 校验结构 + token 引用 + WCAG 对比度，输出结构化 JSON
npx @google/design.md diff DESIGN-v1.md DESIGN-v2.md  # 对比两版，检测 token 与文案回归
npx @google/design.md export ...                      # 导出
```

`lint` 的 JSON 输出可直接进 CI：

```json
{ "findings": [{ "severity": "warning", "path": "components.button-primary",
  "message": "contrast ratio 15.42:1 — passes WCAG AA." }],
  "summary": { "errors": 0, "warnings": 1, "infos": 1 } }
```

---

## 3. 两者的关系

```
        ┌──────────────────────────────────────────┐
        │  输入（上下文）                            │
        │  DESIGN.md  ← awesome-design-md 提供语料   │
        │  PRODUCT.md ← 你自己 / impeccable init     │
        └───────────────────┬──────────────────────┘
                            ↓
        ┌──────────────────────────────────────────┐
        │  过程（方法论）                            │
        │  shape → 实现 → critique → audit           │
        │       → polish → harden                    │
        │  ← impeccable 提供 playbook                │
        └───────────────────┬──────────────────────┘
                            ↓
        ┌──────────────────────────────────────────┐
        │  验证（下限）                              │
        │  62 条检测规则 + design.md lint + 截图      │
        └──────────────────────────────────────────┘
```

**不是二选一**：只装 impeccable → 有方法但没有你的产品语境，产出仍会同质化；只用 DESIGN.md → 有规范但没人执行、没人审。

---

## 4. 落地路线

### 4.0 一个前提认知

大部分人以为要「装个工具」，其实**真正决定效率的是把隐性知识写成文件**。这两个仓库最大的价值，是示范了「该写什么文件、写到什么颗粒度」。

### 4.1 阶段 0：建立三个「真相文件」

```text
your-repo/
├── AGENTS.md              # 怎么构建：命令、目录约定、测试方式、禁止事项
├── PRODUCT.md             # 为谁做：用户、场景、目标、约束、术语
├── DESIGN.md              # 长什么样：tokens + 8 章节规范
├── docs/decisions/        # 为什么这么做：ADR，防止 AI 反复推翻旧决策
└── .github/workflows/design.yml
```

**`AGENTS.md` 原则：只写 Agent 猜不到的东西。**
不要写「写干净代码」。要写：
- 用 pnpm 不用 npm
- 组件必须从 `src/ui/index.ts` barrel export
- 新增页面必须同步 `i18n/{zh,en,ja,ko}.json` 四个文件
- 禁止内联颜色值，一律走 token

**`PRODUCT.md` 原则：只写确认过的事实。**
让 impeccable 的 `init` 采访你生成，它只问三个核心问题：**谁、在什么场景、要完成什么工作**。它明确说了：init 阶段**不问**视觉方向、配色、字体——那些属于 DESIGN.md。

**`DESIGN.md` 原则：把「感觉」翻译成「数字」。**

| ❌ 写了等于没写 | ✅ 可执行 |
|---|---|
| 现代简约 | 正文 16px/1.6，行宽 68ch，卡片间距 24px |
| 圆角要柔和 | 圆角统一 8px，仅 modal 用 12px |
| 科技感 | 主色 `oklch(62% 0.18 250)`，饱和度不超过 60% |
| 有品质感 | 阴影 `0 1px 2px rgba(0,0,0,.06), 0 8px 24px -8px rgba(0,0,0,.12)` |

### 4.2 阶段 1：生成你自己的 DESIGN.md

```bash
# 情况 A：已有项目 → 让 agent 从代码反推
/impeccable document

# 情况 B：从零开始 → 从合集里挑一份「气质参考」
curl -O https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md/linear.app/DESIGN.md

# 情况 C：从现有网站逆向
#   用 bergside/design-md-chrome 扩展，或 Paidax01/web-to-design-md

# 然后必须改三件事，让它变成你自己的：
#   1. colors      → 换成你的品牌色（哪怕让 AI 重新生成一套）
#   2. typography  → 换成你能合法使用的字体（Google Fonts / 自托管 / 系统栈）
#   3. Do's and Don'ts → 换成你团队的真实禁令

# 校验
npx @google/design.md lint DESIGN.md
```

### 4.3 阶段 2：装工具链（约 30 分钟）

```bash
# Agent 侧
npx impeccable install
#   → 选 project scope（团队共享）
#   → 接受 design hook（这是最关键的开关）

# CI 侧：.github/workflows/design.yml
name: design
on: [pull_request]
jobs:
  design-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx @google/design.md lint DESIGN.md
      - run: npx impeccable detect --github src/
      - name: start dev server & scan rendered page
        run: |
          npm ci && npm run dev & 
          npx wait-on http://localhost:5173
          npx impeccable detect --github http://localhost:5173
```

**Hook 是最有价值的部分**：Agent 每改一个 UI 文件自动扫，findings 直接回喂，**在它说「完成了」之前拦住**。没有 hook，检测器只在你记得手动跑的时候才跑。

### 4.4 阶段 3：把「一次对话」变成「一条流水线」

每个 UI 需求走这个 loop，**不要跳步**：

```text
shape（先规划，别急着写码）
  ↓
生成实现
  ↓
critique（UX 打分，看哪里最该修）
  ↓
audit（a11y / 性能 / 响应式，P0–P3）
  ↓
按 P0 → P3 修
  ↓
polish（收尾）
  ↓
harden（错误态 / i18n / 溢出 / 空态）
  ↓
截图对比 + 人工终审
```

**核心原则：让 Agent 自己看渲染结果。**
它写完就说「做好了」毫无意义。必须给它截图能力（Playwright MCP / Chrome DevTools MCP / impeccable 的 `live`），让它在 **1440 / 768 / 375** 三个宽度各看一次。impeccable 的 SKILL.md 原话：

> "Verify by looking at the rendered result at the sizes it ships at, and batch the checks."

### 4.5 阶段 4：把团队审美固化成规则

**每次发现 AI 犯同样的错，就把它变成一条规则**——这是「以后更规范」的唯一机制：

| 发现的重复问题 | 固化到哪 |
|---|---|
| 又用了渐变文字 | `DESIGN.md` → `## Do's and Don'ts` |
| 又在标题上加 eyebrow 小字 | 同上，写成硬 ban |
| 又是三张等宽卡片 | `AGENTS.md` 的禁止模式 |
| 灰字放彩色底上 | 检测器内置规则；**确保 CI 扫渲染页**才生效 |
| 字体用了 Inter | `npx impeccable ignores add-value overused-font Inter --reason "品牌字体"` |
| 新增页面忘了 i18n | `AGENTS.md` 写死；加 lint 脚本 |

**规则必须留例外通道**（带理由），否则团队会绕过整个系统。

### 4.6 阶段 5：守住三条底线

1. **DESIGN.md 是约束，不是创意来源。**
   每次新项目都从它出发，但第一个决策必须是「这个产品和别人哪里不一样」，写进 `PRODUCT.md`。

2. **不要直接复用品牌资产。**
   色板结构可以借，字体/插画/logo 不行——既是法律问题，也是同质化问题。

3. **62 条检测是下限，不是上限。**
   全绿只代表「不像 AI 垃圾」，不代表「好看」。好看仍然需要人的判断。

---

## 5. 模板

### 5.1 `AGENTS.md`（精简版，20 行以内）

```markdown
# AGENTS.md

## 命令
- 安装：`pnpm i`
- 开发：`pnpm dev`（:5173）
- 测试：`pnpm test`，单文件 `pnpm test <path>`
- 类型检查：`pnpm typecheck`（提交前必须过）

## 约定
- 包管理器：pnpm。不要生成 package-lock.json。
- 组件：全部从 `src/ui/index.ts` 导出，不要深层 import。
- 颜色/间距/圆角：只能用 `src/styles/tokens.css` 中的 CSS 变量，禁止内联魔法值。
- 文案：所有面向用户的字符串走 `t()`，同步 `i18n/{zh,en}.json`。
- 新增路由：必须同时在 `src/routes/index.ts` 注册并加一条 e2e 冒烟。

## UI 改动验收（缺一不可）
- [ ] 1440 / 768 / 375 三个宽度截图正常
- [ ] hover / disabled / loading / error / empty 五态齐全
- [ ] 键盘可达，focus ring 可见
- [ ] `npx impeccable detect http://localhost:5173` 无 P0/P1
```

### 5.2 `DESIGN.md` 最小骨架

```markdown
---
name: <你的产品名>
colors:
  primary: "#..."
  on-primary: "#..."
  canvas: "#..."
  surface-1: "#..."
  ink: "#..."
  ink-muted: "#..."
  hairline: "#..."
typography:
  display: { fontFamily: <可用字体>, fontSize: 48px, fontWeight: 600, lineHeight: 1.1, letterSpacing: -1.2px }
  body:    { fontFamily: <可用字体>, fontSize: 16px, fontWeight: 400, lineHeight: 1.6 }
rounded:
  sm: 6px
  md: 10px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 40px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    rounded: "{rounded.md}"
    padding: 12px
---

## Overview
<一段话：这是什么气质的界面，为谁，在什么场景用。>

## Colors
<每个 token 的语义与使用边界。>

## Typography
<层级关系、行宽上限、什么时候用 display。>

## Layout
<栅格、容器宽度、间距节奏。>

## Elevation & Depth
<阴影策略；有几种层级，分别什么时候用。>

## Shapes
<圆角体系，何时破例。>

## Components
<关键组件的构成规则。>

## Do's and Don'ts
- Don't: 渐变文字
- Don't: 标题上方的 kicker 小字
- Don't: 卡片套卡片
- Don't: 彩色底上直接放灰字
- Do: 动效只做一个作者署名的时刻
- Do: 主题化选区颜色 / 滚动条 / focus ring
```

---

## 6. 反模式清单

| 反模式 | 为什么错 |
|---|---|
| 把这两个 star 数当质量背书 | impeccable 2025-11 才创建，一年涨到 10 万 star，热度真实但生态在快速变化，命令与行为可能随版本变 |
| 直接 clone impeccable | 仓库约 877 MB，用 `npx` 即可 |
| DESIGN.md 抄完不改 | 你会得到和几千人一样的界面，而且侵权风险 |
| 只在 CI 扫源码目录 | 对比度、小字号类规则需要渲染页，必须额外扫 URL |
| 为了让 CI 绿掉而加一堆 ignore | 规则烂掉之后整个系统就失效了；例外必须写理由 |
| 让 Agent 自己说「做完了」 | 必须给它截图能力和明确的验收清单 |
| 只装工具不改流程 | 工具是下限，流程（shape → critique → polish → harden）才是产出质量来源 |

---

## 7. 今天就能做的 5 件事

1. **在主力项目跑 `npx impeccable install`**，选项目级 + 接受 hook，然后 `/impeccable init`
2. **从 awesome-design-md 挑一份最接近你产品气质的 DESIGN.md**，改 color / typography 两节，跑 `lint`
3. **写 10 行 `AGENTS.md`**：只写 Agent 猜不到的命令和约定
4. **对现有页面跑一次 `/impeccable critique <某个页面>`**，看它列的第一条是什么——通常会认同
5. **CI 加两行**：`design.md lint DESIGN.md` + `impeccable detect --github`（再加扫 dev server URL 那一步）

---

## 8. 参考链接

- [pbakaus/impeccable](https://github.com/pbakaus/impeccable) · [官方文档](https://impeccable.style/docs)
- [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) · [getdesign.md](https://getdesign.md)
- [google-labs-code/design.md](https://github.com/google-labs-code/design.md)（格式规范）
- [Google Stitch DESIGN.md 概述](https://stitch.withgoogle.com/docs/design-md/overview/)
- [bergside/design-md-chrome](https://github.com/bergside/design-md-chrome)（网页 → DESIGN.md）
- [VoltAgent/official-design-md](https://github.com/VoltAgent/official-design-md)（第一方语料）
