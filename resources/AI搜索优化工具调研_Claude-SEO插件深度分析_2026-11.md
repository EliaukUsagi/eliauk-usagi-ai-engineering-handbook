# Claude SEO 插件调研报告

> 调研时间：2026-11-06 ｜ 仓库：`AgriciDaniel/claude-seo`
> 数据来源：GitHub 搜索 API + 仓库原始文件（README / AGENTS.md / ARCHITECTURE.md / COMMANDS.md / SKILL.md / CHANGELOG / requirements.txt / hooks）

---

## 0. 三十秒结论

| 问题 | 答案 |
|---|---|
| 这是什么 | 给 **Claude Code** 用的免费开源 **SEO 审计插件**：26 个子技能 + 19 个专职 agent + 60 个 Python 脚本 |
| 干什么用 | 丢一个网址进去，10–15 分钟跑完全站 SEO 审计（含 AI 搜索可见性），输出分级行动清单 |
| Star / Fork | **26,951 / 3,496**（MIT，2026-02-07 发布 v1.0.0，当前 v2.5.0） |
| 值不值得看 | **值得**。它是目前「AI Agent + 垂直领域专业知识」这个范式里做得最完整的样板之一 |
| 要不要直接用 | 如果你用 Claude Code：**直接装**。如果你用别的 harness（含 DeepSeek Harness）：**要移植，成本中等** |
| 最大的坑 | README 营销味很重，部分数据自述无法验证；无 API key 时 CWV 只是实验室估算 |

---

## 1. 基础盘

| 项 | 值 |
|---|---|
| 仓库 | `AgriciDaniel/claude-seo` |
| Star / Fork | 26,951 / 3,496 |
| License | MIT（Copyright 2026 agricidaniel） |
| 首次发布 | 2026-02-07（v1.0.0） |
| 当前版本 | v2.5.0（2026-10-22） |
| 最近提交 | 2026-11-01 |
| 项目地址 | https://github.com/AgriciDaniel/claude-seo ｜ https://claude-seo.md |
| 作者 | Agrici Daniel，AI Workflow Architect |
| 技术栈 | Python 3.11–3.14 + Playwright（自带隔离运行时）+ Markdown Skill |
| 体量 | 约 7.6 MB / 467 个文件 |

**增长曲线值得注意**：9 个月从 0 到 2.7 万 star，曾上 GitHub Trending 日榜第 12。同类里排第一，第二名 `zubair-trabzada/geo-seo-claude` 只有 1.15 万。

---

## 2. 它到底解决什么问题

传统 SEO 工具的痛点是**给一堆告警，不给判断**。Screaming Frog 能爬 50 万页，但不会告诉你「先修哪个」「为什么」「怎么验证修好了」。

这个项目赌的是另一件事：**LLM 擅长解释和排序，不擅长高速爬取**——所以它把爬取压缩到「最多 500 页」，把算力花在**分析、归因、排优先级**上。

它覆盖六块：

1. **技术 SEO**：可爬取、可索引、sitemap、robots.txt、重定向、Core Web Vitals
2. **内容质量**：E-E-A-T（经验/专业/权威/可信）、薄内容、主题覆盖
3. **结构化数据**：检测、校验、生成 Schema.org
4. **AI 搜索（GEO）**：AI Overviews、ChatGPT、Perplexity、Gemini 引用你页面的可能性
5. **本地 & 电商**：Google Business Profile、评价、地图包、商品页
6. **国际化**：hreflang、i18n

另有 `/seo agentic`——检查**AI Agent 能不能使用你的网站**（Lighthouse Agentic Browsing X/N、可访问性树、llms.txt、Markdown 版本、WebMCP）。这是 2026 年才出现的新赛道，它做得比较早。

---

## 3. 架构解剖

```text
claude-seo/
├── .claude-plugin/          # marketplace.json + plugin.json（版本、分类、关键词）
├── skills/                  # 26 个子技能（Markdown + YAML frontmatter）共 6937 行
│   ├── seo/                 # 总调度器（325 行）+ references/ 13 份手册
│   ├── seo-audit/           # 全站审计（并行 subagent）
│   ├── seo-technical/       # 技术 SEO 9 大类（295 行）
│   └── ... 共 26 个
├── agents/                  # 19 个专职 agent（Markdown 角色定义）
├── scripts/                 # 60 个 Python 执行脚本
├── extensions/              # 9 个可选 MCP 扩展
├── hooks/                   # PostToolUse 钩子：改文件后校验 schema
├── data/                    # google-updates.json（Google 官方更新库）
├── schema/                  # templates.json（结构化数据模板）
├── tests/                   # 77 个测试文件，785 个测试函数
└── docs/                    # 8 份文档，COMMANDS.md 27KB
```

### 3.1 三层结构：Skill 指挥、Agent 干活、Script 出数据

这是它最值得抄的设计：

| 层 | 载体 | 职责 | 特点 |
|---|---|---|---|
| **Skill** | `SKILL.md` | 定义「什么时候用、按什么步骤做」 | 纯文本，任何 harness 都能读 |
| **Agent** | `agents/*.md` | 一个角色 + 一个模型 + 一套检查项 | 可并行，可指定 Opus/Sonnet |
| **Script** | `scripts/*.py` | 真正去抓页面、调 API、算分数 | 确定性，不靠 LLM 猜 |

**关键约束**：Skill 里明确写了「**永远不要用裸 Python 解释器调用脚本**」，必须走统一启动器：

```bash
"${CLAUDE_PLUGIN_ROOT}/scripts/claude-seo" run fetch_page.py https://example.com --json
```

这个启动器负责：找隔离的 venv、检查依赖、错误提示。**这是把「LLM 会瞎调命令」这个风险工程化解决掉的正确姿势。**

### 3.2 19 个 agent（含模型分工）

| Agent | 角色 | 模型 | 命令 |
|---|---|---|---|
| `seo-technical` | 技术 SEO 负责人 | Sonnet | `/seo technical` |
| `seo-content` | 内容编辑 | **Opus** | `/seo content` |
| `seo-geo` | AI 搜索分析师 | **Opus** | `/seo geo` |
| `seo-cluster` | 主题策略师 | **Opus** | `/seo cluster` |
| `seo-sxo` | 搜索体验设计师 | **Opus** | `/seo sxo` |
| `seo-drift` | 变更监控 | **Opus** | `/seo drift` |
| `seo-schema` | 结构化数据工程师 | Sonnet | `/seo schema` |
| `seo-sitemap` / `seo-performance` / `seo-visual` | sitemap / CWV / 视觉 QA | Sonnet | `/seo audit` |
| `seo-local` / `seo-maps` / `seo-backlinks` / `seo-ecommerce` / `seo-google` / `seo-agentic` / `seo-flow` | 各自领域 | Sonnet | 各自命令 |

**成本设计**：5 个需要判断力的用 Opus，其余用 Sonnet。README 明说了「一次全站 audit 比单页检查更费你的 Claude 额度」，并且**允许你自己改 `agents/` 里任何 agent 的模型**。

### 3.3 9 个可选扩展（都不装也能用）

`ahrefs`、`dataforseo`、`seranking`、`profound`、`firecrawl`、`bing-webmaster`、`matomo`、`unlighthouse`、`banana`（AI 生图）。

设计上是 **MCP 镜像**：`skills/seo-dataforseo` 是扩展开关，装了才有数据。

---

## 4. 命令全景（30 个核心命令，6 组）

<details>
<summary><b>① 审计与规划（5）</b></summary>

- `/seo setup` — 创建/刷新隔离 Python 运行时和 Chromium
- `/seo doctor` — 只检查不修改
- `/seo audit <url>` — 全站审计，并行 agent
- `/seo page <url>` — 单页深度分析
- `/seo plan <type>` — 战略规划（saas / local / ecommerce / publisher / agency）
</details>

<details>
<summary><b>② 技术 SEO（7）</b></summary>

- `/seo technical <url>` — 9 大类技术审计
- `/seo sitemap <url|generate>` — 分析或生成 XML sitemap
- `/seo images <url>` — 图片优化
- `/seo hreflang <url>` — 多语言 SEO
- `/seo programmatic <url>` — 程序化 SEO
- `/seo google [cmd]` — GSC / PageSpeed / CrUX / Indexing / GA4 / PDF 报告
- `/seo drift baseline|compare|history <url>` — 漂移监控（SQLite 快照）
</details>

<details>
<summary><b>③ 内容与 E-E-A-T（6）</b></summary>

- `/seo content <url>` — E-E-A-T 与内容质量
- `/seo content-brief <topic>` — 内容简报（关键词、大纲、内链）
- `/seo cluster <keyword>` — 基于 SERP 的语义聚类
- `/seo sxo <url>` — 搜索体验优化（页面类型、用户故事、画像）
- `/seo flow [stage]` — FLOW 框架（CC BY 4.0）
- `/seo competitor-pages <url>` — 竞品对比页生成
</details>

<details>
<summary><b>④ Schema 与 AI 搜索（3）</b></summary>

- `/seo schema <url>` — 检测、校验、生成 Schema.org
- `/seo geo <url>` — AI Overviews / 生成式引擎优化
- `/seo agentic <url>` — Agent 可读性（Lighthouse Agentic X/N、llms.txt、Markdown、WebMCP）
</details>

<details>
<summary><b>⑤ 本地 / 外链 / 电商（4）</b></summary>

- `/seo local <url>` — GBP、引用、评价、地图包
- `/seo maps [cmd]` — 地图情报（geo-grid、GBP 审计、竞品半径）
- `/seo backlinks <url>` — 外链画像（Moz / Bing / Common Crawl）
- `/seo ecommerce <url>` — 电商 SEO
</details>

<details>
<summary><b>⑥ 扩展（5）</b></summary>

- `/seo image-gen`、`/seo seranking`、`/seo profound`、`/seo bing`、`/seo unlighthouse`
</details>

---

## 5. 一次 `/seo audit` 到底做了什么

```text
1. 渲染首页        render_page.py --mode auto --json
                   → 原始 HTML + 渲染后 HTML + 正文 + SPA 判定 + 可访问性树
2. 识别业务类型    SaaS / 本地服务 / 电商 / 出版 / 代理商 / 其他
3. 爬站            最多 500 页，遵守 robots.txt
                   5 并发 / 1 秒间隔 / 30 秒超时 / 最多 3 跳重定向
4. 分派 agent      基础 8 个 + 按条件追加（本地→seo-local，电商→seo-ecommerce，
                   有 Google 凭证→seo-google，有基线→seo-drift…）最多 17 个并行
5. 打分            SEO Health Score 0–100
6. 落盘            {domain}-audit/
7. 出报告          优先级行动清单 + 可选 PDF/HTML
```

### 5.1 评分权重（这是它最诚实的部分）

| 类别 | 权重 |
|---|---|
| 内容质量 | 23% |
| 技术 SEO | 22% |
| 站内 SEO | 20% |
| Schema / 结构化数据 | 10% |
| 性能（CWV） | 10% |
| **AI 搜索就绪度** | **10%** |
| 图片 | 5% |

内容和技术占 45%，**AI 搜索只给 10%**——因为 Google 自己说了「为 AI 搜索做优化仍然是 SEO」。这个权重分配是克制的，没有为了卖点把 GEO 吹成全部。

### 5.2 输出文件

```text
{domain}-audit/
├── FULL-AUDIT-REPORT.md        # 完整发现
├── ACTION-PLAN.md              # 按 Critical > High > Medium > Low 排序
├── audit-data.json             # 结构化信封，供报告生成器消费
├── findings/                   # 每个专项一份：technical.md / content.md / schema.md ...
├── screenshots/                # 桌面 + 移动端截图
└── *.pdf                       # 可选，A4 企业报告（含目录、仪表盘、路线图）
```

---

## 6. 方法论：10 原则框架

它不只是一堆检查项。`skills/seo/references/thinking-framework.md` 定义了一套四阶段十原则的**综合框架**：

| 阶段 | 原则 |
|---|---|
| **PERCEIVE** | OBSERVE（外部）· OBSERVE（内部）· LISTEN |
| **ANALYZE** | THINK · CONNECT（横向）· CONNECT（系统） |
| **VALIDATE** | FEEL · ACCEPT |
| **ACT** | CREATE · GROW |

一句话原则：

> **没有走完四个阶段的，只是 finding，不是 recommendation。**

其中「OBSERVE（内部）」特意要求**先审计自己的假设**——比如「不要假设首页代表整个站点，很多流量来自程序化页或分类页」。这种「先质疑自己」的元认知步骤，在同类工具里很少见。

---

## 7. 安装与运行成本

### 7.1 标准安装（Claude Code）

```bash
# 在 Claude Code 里执行三条
/plugin marketplace add AgriciDaniel/claude-seo
/plugin install claude-seo@agricidaniel-claude-seo
/seo setup
```

### 7.2 手动安装（macOS / Linux）

```bash
git clone --depth 1 https://github.com/AgriciDaniel/claude-seo.git
bash claude-seo/install.sh
```

> 官方特意不建议 `curl | bash`，理由是 Claude Code 自身的安全护栏会把 `irm | iex` 标记为供应链风险。**这一点很专业。**

### 7.3 环境要求

- Claude Code **≥ 1.0.33**
- Python **3.11–3.14**（3.10 已于 2026-10-01 EOL；3.15 暂不支持，因为 Google Ads 客户端声明 `<3.15`）
- `/seo setup` 自动装 Playwright Chromium，**不污染全局环境**

### 7.4 成本

- 插件本身：**免费**，MIT，无按站收费、无用量限制、无追踪
- 真实成本：**Claude 额度**。全站 audit 会并行跑 17 个 agent（5 个 Opus + 其余 Sonnet），比单页检查贵很多
- 可选扩展：需自备 Ahrefs / DataForSEO / SE Ranking / Profound 等账号
- Google API（PSI / CrUX / GSC / GA4）：**免费**，key 存在你本机

---

## 8. 工程化程度（这块超出预期）

这是我给它加分的核心原因——**它不是一个 prompt 合集，是一个正经软件项目**：

| 证据 | 说明 |
|---|---|
| 785 个测试函数 / 77 个测试文件 | README badge 写 1077 passing（含参数化展开） |
| 3 条 CI | `ci.yml`、`v2.yml`、`windows-smoke.yml` |
| 依赖安全审计 | requirements.txt 每条都注明对应 CVE 与修复版本 |
| ruff 固定版本 | `ruff==0.17.1`，注释写明「防止新版本无故让 CI 变红」 |
| GitHub Actions 固定 SHA | `actions/checkout@3d3c42e...` |
| Hook | `PostToolUse(Edit|Write)` → `validate-schema.py`，改完自动校验 |
| 移植性检查 | `scripts/portability_check.py` 验证 SKILL.md 是否只用了可移植子集 |
| 文档 | 8 份（架构 / 命令 / 安装 / MCP / 迁移 / 排障 / 公私分流） |
| 安全 | `SECURITY.md` 10KB、`PRIVACY.md` 6.5KB，逐条列明第三方数据流向 |

**「PRIVACY.md 里有一张表，列出每个扩展会发什么数据、发给谁、对方隐私政策链接」**——这个颗粒度，大部分商业工具都做不到。

---

## 9. 局限与风险（必须知道的）

| 风险 | 说明 | 应对 |
|---|---|---|
| **无 key 时 CWV 是估算** | 没接 CrUX 只有实验室数据，与真实用户数据可能差很多 | 接免费 Google API：`/seo google setup` |
| **爬取上限 500 页** | 大站会漏 | 大站用 Screaming Frog 做全量爬取，这里做分析与排序 |
| **JS 站点可能噪声** | 需要滚动/点击才加载的内容会不准 | 已知限制，README 有写 |
| **自述数据不可验证** | 「12 周 10.8K 点击」是作者自有站点的自述 | 当参考，别当基准 |
| **README 营销味重** | 有 $106,500 的代理费对比图（标注了 illustrative） | 只看机制，忽略话术 |
| **商业化存在** | MIT 公共版 + Skool 付费社区给「私有抢先版」 | 公共版功能完整，不影响使用 |
| **供应链** | install.sh 会下 Python 包、装 Chromium | 读脚本再跑；它自己也建议这么做 |
| **数据流向** | 被审计页面内容会发给 Anthropic（和其他 Claude Code 使用一样） | 审内部/客户站点前先确认合规 |
| **强绑定 Claude Code** | plugin marketplace、hooks schema、`${CLAUDE_PLUGIN_ROOT}` 都是 Claude 专有 | 见下一节移植方案 |

---

## 10. 在你的环境里怎么用（DeepSeek Harness）

它**不是** Claude Code 独占。仓库根目录有 `AGENTS.md`，专门写了 8 种 harness 的加载方式，并提供了 `portability_check.py`。

已验证的支持矩阵：

| Harness | 加载方式 |
|---|---|
| Cursor | 把 `skills/` + `agents/` 软链或复制到 `.cursor/rules/`，命令当文本 prompt |
| Cursor Cloud | 推送仓库，启动时自动读 `AGENTS.md` |
| Google Antigravity | 工作区指向仓库根，先读 `AGENTS.md` |
| Gemini CLI | `gemini init`，对话里 `activate_skill <name>` |
| Grok Build | 直接打开仓库，`grok inspect` 验证发现 |
| OpenAI Codex CLI | 读项目根 `AGENTS.md`，工具名自动别名 |
| Cline | 读 `AGENTS.md`，subagent 降级为上下文内展开 |
| Aider | 读 `AGENTS.md`，**不支持 subagent 分派**，全部 inline |

### 10.1 移植到 DeepSeek Harness 的具体步骤

**能移植的**：`skills/*/SKILL.md`（Markdown + 通用 frontmatter）、`agents/*.md`、`scripts/*.py`

**不能直接移植的**：`hooks/hooks.json`（PostToolUse schema 是 Claude Code 专有）、`/plugin marketplace` 安装、`${CLAUDE_PLUGIN_ROOT}` 变量展开

```bash
# 1) 取代码（不要 clone 整个历史）
curl -L https://codeload.github.com/AgriciDaniel/claude-seo/tar.gz/refs/heads/main -o cs.tgz
mkdir -p claude-seo && tar xzf cs.tgz -C claude-seo --strip-components=1

# 2) 建隔离运行时（Python 3.11+）
cd claude-seo && python3.11 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt && playwright install chromium

# 3) 验证可移植性
./scripts/claude-seo run portability_check.py

# 4) 把 skills/ 复制到你的 skill 目录（项目级 .agent/skills/ 或全局）
# 5) 改写 SKILL.md 里的 ${CLAUDE_PLUGIN_ROOT} → 你的绝对路径
#    参照 install.sh 第 138–140 行的做法：只改写本次安装过来的文件

# 6) 工具名映射：Read/Write/Edit/Bash/Glob/Grep/WebFetch
#    → 你的 harness 对应工具（DSH 里是 read/write/edit/bash/glob/grep/web_fetch）

# 7) 丢弃 hooks/，或改成你自己的 post-edit 钩子调 validate-schema.py
```

**移植后的降级**：并行 subagent 数量取决于你的 harness；没有 subagent 就退化为串行（和 Aider 一样）。核心价值（评分逻辑、检查清单、方法论）不受影响。

---

## 11. 与上一份报告的关系

两份报告在「对外产品页」这个交叉点上是叠加的：

| | `impeccable` | `claude-seo` |
|---|---|---|
| 管什么 | 页面**看起来**好不好 | 页面**能不能被找到** |
| 输出 | `PRODUCT.md` / `DESIGN.md` | `ACTION-PLAN.md` / schema / llms.txt |
| 验证 | 62 条设计规则检测 | SEO Health Score + 优先级清单 |
| 共同点 | **都把「机器需要理解的东西」写成文件** | 同左 |

有意思的是，`claude-seo` 的 `/seo geo` 和 `/seo agentic` 生成的 `llms.txt`、Markdown 版本、WebMCP 描述，本质上是「**给 AI 读的 UI**」——和 `DESIGN.md` 是同一族思路：**降低机器理解你的成本**。

---

## 12. 竞品对比

| 仓库 | Star | 定位 | 差异 |
|---|---|---|---|
| **AgriciDaniel/claude-seo** | 27.0k | 全栈 SEO 插件 | 最完整，26 skills / 19 agents，工程化最强 |
| zubair-trabzada/geo-seo-claude | 11.5k | GEO 优先 | 专注 AI 搜索，可引用性/品牌信号更深 |
| coreyhaines31/marketingskills | 58.8k | 通用营销技能 | 覆盖 CRO/文案/SEO/分析，SEO 只是其中一块 |
| kostja94/marketing-skills | 1.1k | 160+ 营销技能 | 广度大，深度浅 |
| Hainrixz/claude-seo-ai | 82 | SEO + GEO 双评分 | 小，轻量 |
| huifer/claude-code-seo | 111 | Next.js SEO 助手 | 绑定 Next.js 项目 |

**选型建议**：要「一次完整审计」选 claude-seo；要「只做 AI 搜索」选 geo-seo-claude；要「营销全链路」选 marketingskills。

---

## 13. 落地清单

**如果你只想快速试一次**

1. 装 Claude Code（≥1.0.33）
2. `/plugin marketplace add AgriciDaniel/claude-seo` → `/plugin install` → `/seo setup`
3. `/seo page https://你的站点/一个落地页` —— 先跑单页，看输出质量
4. 满意再 `/seo audit`，注意 agent 数量对额度的消耗

**如果你想长期用**

1. `/seo google setup` 接免费 Google API（否则 CWV 是估算）
2. `/seo drift baseline <url>` 存基线，之后每周 `/seo drift compare`
3. 把 `ACTION-PLAN.md` 转成团队 issue，按 Critical → High 排
4. 接 CI：每次发版跑一次 `seo-technical` + `seo-schema`

**如果你想把它当「AI 垂直应用」的样板学**

重点看四个文件：

| 文件 | 学什么 |
|---|---|
| `skills/seo/SKILL.md` | 如何写一个**总调度器**：路由、命令表、编排逻辑、运行时约束 |
| `docs/ARCHITECTURE.md` | 如何用三层（Skill / Agent / Script）分工，避免让 LLM 干它不擅长的事 |
| `scripts/claude-seo`（启动器） | 如何把「LLM 会瞎调命令」这个风险工程化解决 |
| `AGENTS.md` | 如何做**跨 harness 可移植**，而不是绑定单一平台 |

---

## 14. 一句话评价

> 它是「**提示词工程 → 软件工程**」这个转变的一个高质量样例：把领域知识写成可移植的 Markdown，把确定性工作交给脚本，把判断工作交给不同价位的模型，并且用测试、CI、依赖审计和隐私清单把它做成了能长期维护的东西。

**扣分项**：README 营销密度过高、自述数据不可验证、强绑定 Claude Code 生态、无 key 时精度打折。

---

## 15. 参考链接

- 仓库：https://github.com/AgriciDaniel/claude-seo
- 官网：https://claude-seo.md
- 命令全集：`docs/COMMANDS.md`
- 架构：`docs/ARCHITECTURE.md`
- 方法论：`skills/seo/references/thinking-framework.md`
- 隐私：`PRIVACY.md`（含第三方数据流向表）
- 同族项目：Claude Blog / Claude Banana / ChatGPT SEO / FLOW
