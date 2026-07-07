# Goal: 将当前 Obsidian vault 初始化并配置为 XiRang 知识系统

---

## Agent Instructions（Agent 必读）

你必须按以下顺序执行，不可跳过任何步骤：

### 第一轮：模块选择

使用你所在平台的 ask / choice / question 机制，向用户展示以下可选模块清单（多选），让用户勾选需要启用的功能：

> 以下为 XiRang 可选功能模块。**Core（必选基础）** 会始终安装。请勾选你需要的额外模块：

| 编号 | 模块 | 简介 |
|------|------|------|
| A | 实体百科系统 | 从 source note 自动拆分独立实体为百科词条，双轴分类，wikilink 强制执行 |
| B | 知识卡片提炼 | 同一 domain 积累 ≥3 篇 source 后自动提炼为结构化知识卡片，含争议追踪 |
| C | 多源 Agent 会话同步 | 自动同步 Claude Code / Codex / Reasonix / ChatGPT 等多平台会话到知识库 |
| D | 定时流水线 | 每日自动同步+抽取+日报，每周蒸馏+健康检查+skill 优化 |
| E | 知识雷达 | 跨项目工具/技术推荐引擎，发现你可能遗漏的有价值工具 |
| F | 知识访谈 | 主动发现知识缺口，向用户精准提问并回写答案 |
| G | 记忆系统 | 短时项目 brief（14 天滚动）+ 长时记忆蒸馏 + 跨项目全局 persistent context |
| H | 论文发现 | 自动检测 DOI/arXiv/PMID，多工具链获取论文全文 |

### 第 1.5 轮：全局检索入口（可选，在模块选择后立即询问）

模块选择完毕后，在进入逐模块信息收集之前，先问用户是否需要一个全局知识库检索入口：

> **是否要创建一个全局命令，让你在任何项目目录下都能检索这个知识库？**
>
> 例如你在 `D:\Projects\MechDW\` 下写代码时，输入 `/xirang Docker 配置` 就能直接搜到知识库里的相关内容。
>
> - 是 → 请输入你希望的命令名称（默认：`xirang`）
> - 否 → 跳过，只在 vault 内使用

如果用户选择"是"，记录命令名称（记为 `$RETRIEVAL_CMD`），在实现阶段完成后执行全局注册（见 `## 全局检索入口实现`）。

### 第二轮：逐模块信息收集

用户勾选模块后，遍历每个勾选的模块，使用 ask 机制收集该模块所需的用户信息（见各模块的 `## Ask 问题` 节）。**不要在信息不全的情况下开始实现。**

### 第三轮：实现

所有信息收集完毕后，先实现 Core，再按依赖顺序实现用户勾选的模块。每完成一个模块，运行该模块的验收标准验证。

如果用户在 1.5 轮选择了全局检索入口，在 Core 实现完成后立即执行下方 `## 全局检索入口实现`。全局检索不依赖任何可选模块。

### 平台能力自检

初始化完成后，检测你当前所在平台的能力（能否注册 cron/计划任务、能否读写用户目录等），生成 Post-Setup Checklist（见文档末尾），仅列出你做不到、需要用户手动处理的事项。

---

## Core（必选基础）

所有 XiRang 实例必须包含以下内容。无论用户是否勾选任何可选模块，Core 始终初始化。

### 1. 目录结构

```text
vault/
├── raw/
│   ├── inbox/
│   │   ├── 00_webclip/          # Web Clipper 保存的网页
│   │   ├── 20_cowork/           # 对话中上传的文件（PDF/DOCX/MD/code）
│   │   ├── 30_screenshot/       # 对话中粘贴的截图
│   │   ├── 40_manual/           # 用户手动放入的资料
│   │   ├── 50_to_review/        # 待审核（不确定是否保留的材料）
│   │   └── 70_agent_chat/       # Ad-hoc agent 对话记录
│   └── archived/                # 已处理材料归档（按 YYYY-MM/ 分月）
├── wiki/
│   ├── source-notes/            # 来源笔记（轻量索引）
│   ├── persona/                 # 用户画像（动态分类子目录）
│   ├── moc/                     # Maps of Content（领域导航）
│   └── index.md                 # Vault 总索引
├── logs/
│   ├── source-registry.csv      # 来源注册表
│   ├── operation-log.md         # 操作日志
│   └── reports/                 # 报告目录
├── skills/                      # 模块化 AI 操作规则
│   ├── ingest-attachment.md     # Ingest 编排器
│   ├── lifecycle-source-note.md # Source note 模板与生成
│   ├── persona.md               # Persona 更新规则
│   ├── infra-retrieval.md       # 检索优先级规则
│   └── infra-vault-architecture.md # 知识库架构总览
├── AGENTS.md                    # AI 规则（核心入口）
├── CLAUDE.md                    # 内容为 `AGENTS.md`（兼容 Claude Code）
└── GOAL.md                      # 本文件
```

### 2. 核心文件实现

#### 2a. `AGENTS.md`

AGENTS.md 是 vault 的 AI 行为规则文件，必须包含以下规则（自包含，不依赖外部文件）：

**文件入库路径规则**：
- PDF/DOCX/MD/TXT/CSV/code → `raw/inbox/20_cowork/`
- 截图 (PNG/JPG/WEBP/GIF) → `raw/inbox/30_screenshot/`
- Web Clipper 保存 → `raw/inbox/00_webclip/`
- Agent 对话记录 → `raw/inbox/70_agent_chat/`
- 手动放入 → `raw/inbox/40_manual/`
- 不确定是否保留 → `raw/inbox/50_to_review/`，标记 needs-review
- 附件 bytes 不可访问时：明确报告限制，不得假装已保存

**Ingest 流水线**：
每收到新材料时执行完整流水线：接收分类 → 质量控制过滤 → 注册 `source-registry.csv` → 生成 source note → 更新 persona → 更新 MOC/index → 归档到 `raw/archived/YYYY-MM/` → Git 提交

**质量控制规则**：
- 保留：可复用的技术方法论、项目设计思路、长期参考配置、skill/提示词调教、科研/教学核心讨论、重要项目决策、可复用调试经验
- 丢弃：已过时的一次性报错排查、已废弃工具版本问题、纯闲聊、重复内容、已无实际价值的临时操作、无独立信息量的 tool output 噪声
- 不确定时标记 needs-review，不擅自丢弃

**检索优先级**：
回答问题时按以下顺序检索，命中即停：
1. `wiki/cards/`（如存在）— 最高质量，跨 article 综合
2. `wiki/persona/` — 用户画像和项目信息
3. `wiki/moc/` — 领域导航
4. `wiki/source-notes/` — 来源索引
5. `logs/source-registry.csv` — 来源注册
6. 持久化记忆/用户偏好
7. Web 搜索（如需当前信息）
8. 模型训练数据（标注 `[AI 生成 — 未经验证]`）

**通用回答规则**：
- 回答时优先引用 vault 中的提炼知识
- vault 无答案时注明信息来源
- 遇到矛盾来源时标注冲突，不自行裁决

**归档规则**：
- 处理完的原始文件从 `raw/inbox/` 移入 `raw/archived/YYYY-MM/<原 inbox 子目录名>/`
- 原始文件永不删除或修改
- 归档后更新 registry 中的 `saved_path` 为新路径
- `raw/inbox` 不应长期保留已处理内容

**Git 版本管理**：
- 每次 ingest 后提交：`git add raw/archived/ wiki/ logs/ skills/ && git commit -m "chore: archive YYYY-MM-DD - processed X sources"`
- raw/inbox 不纳入 Git tracking（`.gitignore` 中排除）

**raw 只读原则**：
- `raw/archived/` 中的文件永不编辑，只能通过 wikilink 引用
- 不扫描用户目录中的无关文件

#### 2b. `CLAUDE.md`

文件内容仅一行：`AGENTS.md`（指向 AGENTS.md 的软引用，兼容 Claude Code）。

#### 2c. `logs/source-registry.csv`

CSV header 必须包含以下列：

```csv
source_id,created_at,original_name,saved_path,capture_channel,content_type,domain,project,status,confidence,source_note,notes
```

- `source_id`: `SRC-YYYYMMDD-NNNN` 格式，NNNN 为当日流水号（0001 起）
- `status`: `active` | `needs-review` | `deprecated` | `superseded`
- `confidence`: `high` | `medium` | `low` | `conflicting`
- `capture_channel`: `webclip` | `cowork` | `screenshot` | `manual` | `agent_chat` | `chatgpt` | `codex` | `reasonix` | `claude-code` 等

#### 2d. `wiki/index.md`

Vault 总索引文件。初始化时创建基础结构：

```markdown
# XiRang / 息壤 — Vault Index

## Personas
（创建后自动填充）

## Recent Sources
（创建后自动填充）

## Entry Points
- [[wiki/moc/MOC-research]]
- [[wiki/moc/MOC-teaching]]
- [[wiki/moc/MOC-engineering-projects]]
- [[wiki/moc/MOC-tools-workflows]]
- [[wiki/moc/MOC-administration]]
- [[wiki/moc/MOC-personal-reference]]
- [[wiki/moc/MOC-to-review]]
```

#### 2e. Skill 文件

**`skills/ingest-attachment.md`** — Ingest 编排器：

描述完整的 9 步流水线，包含扩展点（hook）供可选模块插入：
1. 接收与分类（保存到 inbox 对应子目录）
2. 质量控制过滤（保留/丢弃/needs-review）
3. 注册来源（写入 source-registry.csv）
4. 委托抽取（生成 source note，按 `skills/lifecycle-source-note.md` 模板）
   - [Module A hook: 生成 source note 后执行 entity split]
   - [Module H hook: 检测论文标识符]
5. Persona 更新（按 `skills/persona.md`）
   - [Module G hook: 写入项目 brief]
6. [Module F hook: Knowledge Interview 伴随检查]
7. 更新 MOC 和索引
8. 归档原始文件（移入 raw/archived/YYYY-MM/）
9. Git 提交

**`skills/lifecycle-source-note.md`** — Source note 模板：

```markdown
---
source_id: SRC-YYYYMMDD-NNNN
capture_channel: <见 registry>
content_type: <类型>
domain: <领域>
project: <关联项目>
source_path: <原始文件在 archived 中的路径>
created_at: YYYY-MM-DD
status: active
confidence: high | medium | low | conflicting
superseded_by: ""
persona_impact: confirm | contradict | expand | none
---

# Source Note: <简短描述>

## Summary
1-3 句话描述本来源的知识内容（不是元描述如 "I uploaded a file..."）。

## Quality Gate
1. Summary 描述知识内容，不能是用户的原始 prompt 或元描述
2. 如果来源是 Agent session：过滤 agent 自言自语（"Now let me..."），只提取用户意图、关键决策、解决方案
3. persona_impact 必须附理由（confirm/expand 时写明为什么），不能只写 "none"

## Persona Impact
本来源如何影响用户画像：确认了已有偏好？与已知模式矛盾？扩展了新兴趣？

## Trace
L0 → raw/inbox/XXX → source-note/SRC-XXX → [后续层在可选模块中]

## Links
- [[wiki/moc/MOC-XXX]] — 导航
- [[wiki/persona/...]] — 关联 persona

## Open Questions
（本来源遗留的、仅用户可回答的问题）
```

**`skills/persona.md`** — Persona 更新规则：

Persona 文件存储在 `wiki/persona/<category>/` 下，category 为动态分类子目录。

初始启发规则：
- 描述项目 → `project/`
- 描述人物 → `person/`
- 描述工具 → `tool/`
- 描述研究领域 → `research/`
- 描述教学 → `teaching/`

Persona 文件标准结构：
```markdown
---
type: persona
category: <分类>
status: active
privacy: internal
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Name>

## Stable Preferences
## Recurring Patterns
## Known Pitfalls
## Topic Interests
## Active Items
```

每次 ingest 后检查 source note 的 `persona_impact` 字段，若 != none，追加更新条目到对应 persona 文件。

**`skills/infra-retrieval.md`** — 检索优先级：

逐层检索：cards → persona → MOC → source-notes → registry → memory → web → model（标注未验证）

**`skills/infra-vault-architecture.md`** — 知识库架构总览：

描述数据流、目录结构、检索优先级、关键文件索引、当前规模统计。

### 3. Core 验收标准

- [ ] 所有目录存在且含 `.gitkeep`（空目录保留）
- [ ] `AGENTS.md` 包含上述全部规则
- [ ] `CLAUDE.md` 内容为 `AGENTS.md`
- [ ] `logs/source-registry.csv` 含正确 header
- [ ] `wiki/index.md` 存在且含 MOC 入口列表
- [ ] 5 个 core skill 文件就位
- [ ] `.gitignore` 排除 `raw/inbox/`

---

## Optional Modules（用户按需选择）

### Module A：实体百科系统

**简介**：从 source note 中识别和提取独立实体（工具、方法论、Agent 系统、数据集、人物、概念等），每个实体创建一个独立的百科词条（article）。Source note 精简为轻量索引。全局实体注册表（entity-registry）统一管理规范名称、别名、来源追踪。所有文档中的实体引用强制执行 `[[wikilink]]`。

**价值**：知识从"以来源为中心"升级为"以实体为中心"，同一实体在多个来源中的信息自动合并，避免碎片化。

**依赖**：Core

#### Ask 问题

> 以下为「实体百科系统」的配置选项，请确认：

1. **双轴分类体系**：是否启用 category（结构类型：tool / agent-system / methodology / framework / library / platform / dataset / person / concept）× domains（语义领域：biomedical / ai4science / geospatial / neuroscience / engineering / tools-workflows / teaching / infrastructure / research / personal）？
   - 选项：是（推荐）/ 否（仅保留简化分类）
2. **Wikilink 强制执行**：是否要求所有文档中的项目名、人名、工具名必须使用 `[[wikilink]]`（通过 entity-registry 规范名称），禁止纯文本？
   - 选项：是（推荐）/ 否（允许纯文本引用）

#### 实现步骤

1. 创建目录 `wiki/articles/`（含 `.gitkeep`）
2. 创建 `wiki/entity-registry.md`：
   - 命名协议：无空格（GeoEvidence 非 Geo Evidence）、正确大小写、英语术语优先、单数形式、同实体唯一名
   - 初始分类表模板：Projects / Research / Tools / People / Teaching / Articles / Domains
   - 每个实体行格式：`| 规范名称 | Persona/Article 路径 | 别名/旧名 | 首次来源 | 来源类型 | 可信度 | 关系/Status |`
   - Domains 段列出可用领域标签
   - 使用规则：写文档时必须用规范名称的 wikilink；首次遇到新实体先注册再引用；别名统一为规范名称
3. 创建 `skills/lifecycle-entity-split.md`：
   - 触发：ingest 流程中 source note 创建后
   - Step 1: 读取 source note 的 Summary + frontmatter
   - Step 2: 识别候选实体（大写专有名词、已知工具/库/框架名、方法论/算法名、Agent 系统名、数据集/benchmark 名、人名；排除过于泛化的概念、项目内部术语、用户本人的项目名）
   - Step 3: 信息量评估（Overview ≥1-3 句 + Key Facts ≥2 条 + 独立可搜索 → 创建 article；不满足 → stub needs-review；不达标 → 保留在 source note）
   - Step 4: 分配 category 和 domains
   - Step 5: 创建/更新 article（按 `skills/lifecycle-article.md` 模板）
   - Step 6: 精简 source note（添加 `## References` 段列出所有拆出的 article wikilink）
   - Step 7: 同 source note 产出的多个 article 互相添加 Related Articles
4. 创建 `skills/lifecycle-article.md`：
   - 模板含 frontmatter（type: article, title, category, domains, aliases, sources, status, created_at, updated_at）
   - 正文结构：Overview → Key Facts（每条标注来源）→ Related Articles → Source References
   - 分类体系表（category 9 种 × domains 10 种，正交独立）
   - 创建规则：最低信息阈值（1 Overview + 2 Key Facts）、规范名（查 entity-registry）、事实引用（每条标注 source-SRC）、去重、合并
   - 与 persona / card 的边界说明
5. 创建 `skills/infra-entity-management.md`：
   - 命名协议、工作流程（提取标准化 → 重复检查 → 创建/维护）
   - 定期检查命名违规和语义重复
6. 更新 `skills/ingest-attachment.md`：在 Step 4（委托抽取）之后插入 entity split 调用
7. 更新 `skills/lifecycle-source-note.md` 模板：在 `## Summary` 后添加 `## References` 段（标准输出格式）
8. 更新 `AGENTS.md`：添加 wikilink 强制执行规则（如果用户选择启用）
9. 更新 `wiki/index.md`：添加 entity-registry 入口链接

#### 验收标准

- [ ] `wiki/articles/` 目录存在
- [ ] `wiki/entity-registry.md` 含完整的分类表和命名协议
- [ ] 3 个新 skill 文件就位
- [ ] `skills/ingest-attachment.md` 含 entity split hook
- [ ] 创建一份测试 source note 后，能成功拆分出至少 1 个 article 并注册到 entity-registry

---

### Module B：知识卡片提炼

**简介**：当同一 domain 积累 ≥3 篇 source note（或对应 ≥2 个 article）后，自动提炼为结构化知识卡片。卡片是跨 article 的综合分析（对比、方法论总结、洞察），不是 article 事实的重复。每张卡片强制包含证据表、不一致/争议点追踪、更新历史。

**价值**：知识从"离散事实"升级为"经交叉验证的结论"，矛盾自动暴露而非被掩盖。

**依赖**：Core + Module A（实体百科系统）

#### Ask 问题

> 以下为「知识卡片提炼」的配置选项：

1. **提炼触发阈值**：同一 domain 积累多少篇 source note 后自动触发卡片提炼？
   - 选项：3 篇（推荐）/ 5 篇 / 手动触发

#### 实现步骤

1. 创建目录 `wiki/cards/`（含 `.gitkeep`）
2. 创建 `skills/lifecycle-knowledge-card.md`：
   - 卡片 frontmatter：title, created_at, updated_at, articles[], sources[], status, superseded_by, confidence
   - 正文结构：核心结论 → 支持证据表（Article/来源 | 结论 | 时间 | 一致性）→ 不一致/争议点（必须填写）→ 更新历史 → 关联
   - 操作流程：确定主题 → 检索相关 articles → 提取核心事实 → 对比一致性 → 填写争议点 → 保存
   - 与 article 的边界：article 是单实体百科，card 是跨 article 综合分析
   - 更新规则：新增 article → 追加证据表；证据矛盾 → 启用争议点 + confidence 降级；来源过时 → 标记 superseded
3. 创建 `skills/lifecycle-distill.md`：
   - 触发：定时任务（weekly maintenance Part 1）或手动
   - 流程：统计 derived_card 为空的 active source notes → 按 domain 聚类 → 对 ≥阈值的聚类创建/更新 card → 更新 source notes 的 derived_card 字段 → 更新 MOC
   - 分批处理策略：research domain 优先 → 多 source 聚集主题 → 用户关注项目
4. 更新 `skills/lifecycle-source-note.md` 模板：在 frontmatter 中添加 `derived_card` 字段（初始值 TBD）
5. 更新 `AGENTS.md`：在检索优先级中将 `wiki/cards/` 置为第一优先级；添加卡片提炼触发条件

#### 验收标准

- [ ] `wiki/cards/` 目录存在
- [ ] `skills/lifecycle-knowledge-card.md` 含完整的证据表和争议点模板
- [ ] `skills/lifecycle-distill.md` 含聚类和分批策略
- [ ] source note 模板含 `derived_card` frontmatter 字段
- [ ] 对已有 ≥3 篇同 domain source notes 能成功生成至少 1 张知识卡片

---

### Module C：多源 Agent 会话同步

**简介**：自动将多个 AI Agent 平台的会话记录同步到知识库统一处理。本地 Agent（Claude Code / Codex / Reasonix 等）直接读取 JSONL 会话文件；网页端 Agent（ChatGPT / Gemini / Claude.ai 等）推荐浏览器插件将对话保存到本地目录。

**价值**：所有 AI 对话集中管理，不会因为换了 Agent 工具而丢失上下文。

**依赖**：Core

#### Ask 问题

> 以下为「多源 Agent 会话同步」的配置选项。请如实勾选，Agent 将为每个选中的源配置同步路径。

1. **本地 Agent 工具**（Agent 可自动扫描会话目录并同步 JSONL）：
   - Claude Code（`~/.claude/projects/*/*.jsonl`）
   - Codex（`~/.codex/sessions/YYYY/MM/DD/*.jsonl`）
   - Reasonix（`~/AppData/Roaming/reasonix/projects/*/sessions/*.jsonl`）
   - Cowork（需 session_info MCP 导出）
   - Qoder Work（`~/.qoderworkcn/projects/`）
   - GitHub Copilot（终端日志 `~/AppData/Roaming/Code/copilot-terminal-output/*.txt`）
   - 以上全部

2. **网页端 AI 工具**（Agent 无法直接访问，需浏览器插件保存到本地）：
   - ChatGPT → 推荐安装「ChatGPT 对话保存助手」浏览器插件，保存路径设为 `raw/inbox/60_chatgpt/ChatGPT-Backup/`
   - Gemini → 推荐安装「Gemini Chat Exporter」浏览器插件
   - Claude.ai → 推荐使用 Claude Code 本地替代，或手动复制对话
   - 其他网页端工具（请说明）

3. **用户主目录路径**（用于扫描 Agent 会话文件）：
   - Windows: `C:\Users\<用户名>`
   - macOS: `/Users/<用户名>`
   - Linux: `/home/<用户名>`
   （Agent 会尝试自动检测，若不正确请手动输入）

#### 实现步骤

1. 创建目录 `raw/inbox/60_agent_sessions/`，为每个本地 Agent 创建子目录（如 `claude-code/`、`codex/`、`reasonix/` 等）
2. 创建目录 `raw/inbox/60_chatgpt/`（用于浏览器插件保存的 ChatGPT 对话）
3. 创建 `skills/ingest-agent-session.md`（Agent 会话路由层）：
   - 路由表：capture_channel → 对应 ingest-* skill
   - 所有 Agent 会话共性的噪声模式：工具调用、自言自语、subagent 嵌套、重复冗余、系统元数据
   - 通用抽取目标（按价值排序）：用户意图 → 最终方案 → 关键决策 → 可复用配置 → 问题诊断 → 遗留问题
   - 通用质量检查表
4. 为用户勾选的每个本地 Agent 创建对应的 ingest skill：
   - `skills/ingest-chatgpt.md` — ChatGPT 导出 JSON 解析
   - `skills/ingest-codex.md` — Codex JSONL 解析（type/payload 事件流）
   - `skills/ingest-reasonix.md` — Reasonix JSONL 解析（type/sessionId 事件流）
   - `skills/ingest-claude-code.md` — Claude Code projects JSONL 解析
   - `skills/ingest-cowork.md` — Cowork 预处理摘要
   - `skills/ingest-qoderwork.md` — QoderWork 扁平消息对象解析
   每个 ingest-* skill 必须包含：
   - 文件格式说明（JSONL 字段结构）
   - 噪声过滤规则（该平台特有的模式）
   - 抽取优先级
   - 质量门控检查表
5. 更新 `skills/ingest-attachment.md` Step 1：添加 `raw/inbox/60_agent_sessions/` 和 `raw/inbox/60_chatgpt/` 到接收目录表
6. 更新 `skills/ingest-attachment.md` Step 4：Agent session 来源委托到 `skills/ingest-agent-session.md`
7. 更新 `AGENTS.md`：添加各源路径和同步规则；添加网页端插件推荐；添加 agent 对话中识别 `/remember` 等记忆指令的规则
8. 为网页端工具输出插件安装指南（写入 `logs/operation-log.md` 或直接告知用户）

#### 验收标准

- [ ] `raw/inbox/60_agent_sessions/` 含所有选中的 Agent 子目录
- [ ] `raw/inbox/60_chatgpt/` 目录存在（如用户选了 ChatGPT）
- [ ] `skills/ingest-agent-session.md` 路由表覆盖所有选中的 Agent
- [ ] 每个选中的 Agent 有对应的 ingest-* skill 文件
- [ ] `skills/ingest-attachment.md` 已更新接收目录和路由
- [ ] 对各 Agent 源路径能成功读取到至少 1 个会话文件（或明确报告源目录为空/不存在）
- [ ] 网页端工具用户收到插件安装指南

---

### Module D：定时流水线

**简介**：每日自动执行：同步所有 Agent 源 → 抽取新会话 → 注册 → source note → 冲突检测 → 短时记忆 → persona 更新 → 日报。每周自动执行：长时记忆蒸馏 → 知识蒸馏 → 健康检查（断链/孤立笔记/质量抽检）→ 实体维护 → skill review → 知识雷达 → 语义 MOC 刷新。

**价值**：知识库从"手动维护"变为"自动运转"，不需要记住要去整理。

**依赖**：Core + 建议同时启用 Module A（实体系统）、Module B（卡片提炼）、Module C（会话同步）、Module G（记忆系统）

#### Ask 问题

> 以下为「定时流水线」的配置选项：

1. **定时任务注册能力**：你当前使用的 Agent 平台能否自动注册操作系统级定时任务？
   - 选项：能（Claude Code 可用 cron schedule 等）/ 不能 / 不确定
   - （如果"不能"或"不确定"：Agent 会在 Post-Setup Checklist 中提醒你手动添加 Windows 计划任务或 cron）
2. **日报生成时间偏好**：每天什么时候生成日报？
   - 选项：晚间 22:00（推荐，汇总全天）/ 早晨 08:00（汇总前一天）/ 自定义时间
3. **周报生成时间偏好**：每周什么时候执行综合维护？
   - 选项：周日 22:00（推荐）/ 周一 08:00 / 自定义时间

#### 实现步骤

1. 创建目录 `schedule/` 及子目录：
   - `schedule/xirang-daily-pipeline/`（含 `SKILL.md`）
   - `schedule/xirang-weekly-maintenance/`（含 `SKILL.md`）
2. 创建目录 `logs/reports/daily/` 和 `logs/reports/weekly/`
3. 编写 `schedule/xirang-daily-pipeline/SKILL.md`：
   - 完整的日报生成流水线（参考现有实现，但泛化为通用版本）：
   - 阶段 1：同步所有 Agent 源 → 扫描 inbox → 分类路由 → 注册 → source note → entity split（如有 Module A）→ 冲突检测 → 归档 → Git
   - 阶段 2：Persona 更新 → 短期记忆抽取（写入项目 brief，如有 Module G）→ 日报生成
   - 日报格式：今日操作概要 + 今日知识精华（top-3）+ 来源明细 + Persona 变更 + 待处理提醒
   - 补扫机制：所有源扫描结果为 0 时自动回溯 72 小时
4. 编写 `schedule/xirang-weekly-maintenance/SKILL.md`：
   - Part 0：长期记忆蒸馏（如有 Module G）
   - Part 1：知识蒸馏（如有 Module B）— 统计未蒸馏 source notes → 按 domain 聚类 → 创建/更新 card
   - Part 2：健康检查 — 断链检测 + 孤立笔记 + 质量抽检（随机 10 个 source note）+ 规模统计 + Inbox 积压
   - Part 3：实体维护（如有 Module A）— 命名规范检查 + 引用一致性 + 孤立/未注册实体检测
   - Part 4：Skill Review — 扫描全部 skill 文件 + AGENTS.md，从本周操作日志识别需固化的规则
   - Part 5：知识雷达（如有 Module E）
   - Part 6：语义 MOC 刷新 — 重新生成 articles 索引 + 更新 entity-registry + 更新规模数字
5. 注册定时任务（如果平台支持）：
   - Claude Code: 使用 `/cron` 或 plan 机制注册 daily + weekly schedule
   - 其他平台：在 Post-Setup Checklist 中给出具体的手动配置指令
6. 更新 `AGENTS.md`：添加定时任务触发规则

#### 验收标准

- [ ] `schedule/xirang-daily-pipeline/SKILL.md` 含完整的每日流水线
- [ ] `schedule/xirang-weekly-maintenance/SKILL.md` 含 6 个 Part 的完整流程
- [ ] `logs/reports/daily/` 和 `logs/reports/weekly/` 目录存在
- [ ] 定时任务已注册（或在 Post-Setup Checklist 中给出配置指南）
- [ ] 手动触发一次日报流程，能成功生成日报文件

---

### Module E：知识雷达

**简介**：跨项目工具/技术推荐引擎。扫描所有 article 构建工具索引，为每个活跃项目生成画像（技术栈、痛点、开放问题），然后交叉匹配发现"在 A 项目中用过但对 B 项目也有价值的工具/技术"，生成推荐报告。

**价值**：你今天学到的某个工具可能就是另一个项目的关键拼图——知识雷达帮你发现这些连接。

**依赖**：Core + Module A（实体百科系统，因为依赖于 article 中的工具/技术实体）

#### Ask 问题

> 以下为「知识雷达」的配置选项：

1. **推荐频率**：多久生成一次跨项目推荐？
   - 选项：每周（推荐，跟随周报）/ 每月 / 手动触发
2. **推荐数量上限**：每次报告最多推荐多少条？
   - 选项：15 条（推荐）/ 30 条 / 不限

#### 实现步骤

1. 创建目录 `wiki/radar/`
2. 创建 `skills/lifecycle-knowledge-radar.md`：
   - Phase 1：项目画像构建 — 扫描 active project persona + 对应 card，提取 {project_name, domain, tech_keywords[], pain_points[], active_questions[]}，输出 `wiki/radar/project-profiles.md`
   - Phase 2：工具索引构建/增量更新 — 扫描 `wiki/articles/` 中 category ∈ {tool, library, framework, platform, methodology, agent-system} 的 article，输出 `wiki/radar/tool-index.md`（含 last_scan_date 增量策略）
   - Phase 3：交叉匹配与推荐 — 排除已用工具 → 按 domain 相关性 + 痛点匹配 → LLM 评估 → 排序 → top-N 推荐，输出 `wiki/radar/recommendations-YYYY-MM-DD.md`
   - 推荐质量规则：不推荐自身、溯源必须、推荐理由具体、区分建议动作（集成/评估/仅了解）、去重
3. 更新 `schedule/xirang-weekly-maintenance/SKILL.md`：Part 5 调用知识雷达
4. 更新 `wiki/index.md`：添加 radar 入口

#### 验收标准

- [ ] `wiki/radar/` 目录存在
- [ ] `skills/lifecycle-knowledge-radar.md` 含三个 Phase 的完整流程
- [ ] 手动执行一次雷达扫描，能生成 `project-profiles.md`、`tool-index.md` 和至少一份 `recommendations-*.md`
- [ ] 推荐报告中的每条推荐引用具体 source note wikilink

---

### Module F：知识访谈

**简介**：主动发现知识库中仅用户可回答的信息缺口，按优先级（P0-P3）生成精准问题，向用户提问并将回答立即回写到对应文件。分两种模式：Ingest 伴随模式（每次 ingest 后检查 ≤2 个高价值缺口就立刻问）和定时扫描模式（每周汇总所有缺口，过滤后取前 5 个提问）。

**价值**：不再被动等待用户主动输入——系统主动识别"这个信息只有你知道"，然后精准提问。

**依赖**：Core

#### Ask 问题

> 以下为「知识访谈」的配置选项：

1. **访谈频率**：定时扫描模式的触发频率？
   - 选项：每周一次（推荐，跟随周报）/ 每两周一次 / 手动触发
2. **优先补齐的信息类型**：你希望优先追问哪类缺口？
   - 选项：个人身份与核心关系 / 活跃项目关键信息 / 工具配置与工作流 / 全部（推荐）
3. **敏感信息偏好**：涉及个人信息（姓名、联系方式、教育经历等）时？
   - 选项：正常提问（存入 persona，标记 privacy: sensitive）/ 跳过，不主动问 / 提问但让我确认是否保存

#### 实现步骤

1. 创建 `logs/knowledge-gaps.md`（缺口队列文件）：
   - 结构：Active（按 P0/P1/P2/P3 分组）→ Resolved → Deferred
   - 每条缺口格式：`- [ ] GAP-YYYYMMDD-NNN | topic | 问题摘要 | source: 发现路径 | target: 回写路径`
2. 创建 `skills/lifecycle-interview.md`：
   - 核心原则：先挖后问（检索已有资料后再决定是否提问）、问少问准（每次 ≤5 个问题）、不问可推的、不问无关的、不丢知识（立即回写）
   - 知识缺口来源（P0-P3 优先级表）
   - 过滤规则：原始资料可抽取 → 不提问改为执行抽取；可合理推断 → 不提问改为标注 `confidence: inferred`；近期已问过 → 30 天内不再问；与活跃上下文无关 → 降低优先级
   - 问题生成规则：每个问题含 id / source / target / priority / topic / question / context / ask_mode
   - 定时扫描流程：8 步（扫描 P0 → P1 → P2 → 过滤 → 排序 → 选取 top-5 → 记录 → 输出）
   - Ingest 伴随模式：在 `skills/ingest-attachment.md` Step 6 后插入，≤2 个缺口立即问，>2 个记录到 gaps queue
   - 回答处理：立即回写到 target 文件 → 更新 gaps queue → 更新 entity-registry → 用 wikilink → 更新 frontmatter
3. 更新 `skills/ingest-attachment.md`：在 Step 5 之后插入 Knowledge Interview 伴随检查（Step 6）
4. 更新 `skills/persona.md`：person 类型 persona 额外支持"待补全"段（含 `- [ ]` checklist）
5. 更新 `AGENTS.md`：添加知识访谈触发规则和缺口处理规则

#### 验收标准

- [ ] `logs/knowledge-gaps.md` 含 Active / Resolved / Deferred 三段结构
- [ ] `skills/lifecycle-interview.md` 含完整的缺口来源表、过滤规则、问题生成规则
- [ ] `skills/ingest-attachment.md` 含 Step 6 伴随访谈 hook
- [ ] 执行一次定时扫描，能发现至少 1 个知识缺口并生成格式化问题

---

### Module G：记忆系统

**简介**：三层记忆体系：
1. **短时记忆**：每日流水线自动为每个活跃项目生成 brief card（`card-agent-brief-{project}.md`），记录今日工作/新发现/阻塞项，保留 14 天滚动窗口
2. **长时记忆**：每周蒸馏 — 从各项目 brief 中提取跨天出现的稳定事实，晋升到对应 persona 文件
3. **全局持久上下文**：跨项目通用的环境配置、个人偏好、知识库规则提取到 `card-agent-persistent-context.md`

**价值**：Agent 的"记忆"从单次对话扩展到跨会话、跨项目，长期记住用户偏好和环境信息。

**依赖**：Core + 建议同时启用 Module A（实体系统，brief 中引用 article wikilink）

#### Ask 问题

> 以下为「记忆系统」的配置选项（通常无需修改）：

1. **短时记忆保留天数**：项目 brief 保留多少天的滚动窗口？
   - 选项：14 天（推荐）/ 7 天 / 30 天

#### 实现步骤

1. 创建 `wiki/cards/card-agent-persistent-context.md`（全局持久上下文）：
   - 章节：👤 个人信息 / 🛠️ 开发偏好 / 🖥️ 环境配置 / 🔧 工具链 / 🧠 知识库规则
   - 每节含子条目，每条标注来源和更新时间
2. 更新 `schedule/xirang-daily-pipeline/SKILL.md`：
   - 步骤 9.5：短期记忆抽取 — 从今日 source notes 按 project 分组写入 `wiki/cards/card-agent-brief-{project}.md`，格式：`## YYYY-MM-DD` → `### 今日工作` / `### 新发现/决策` / `### 阻塞/待续`，保留最近 N 天
   - 步骤 9.5 也包含记忆指令识别：如果 agent 对话中出现 `/remember` 或「记住」「以后都这样」等记忆指令，按维度归类写入对应 brief 或 persistent-context
3. 更新 `schedule/xirang-weekly-maintenance/SKILL.md`：
   - Part 0a：项目级蒸馏 — 读取各 brief 最近 7 天区段 → 逐条评估 → 满足晋升条件（一周后仍成立/跨多天出现/涉及方向或状态变化）的合并到对应 persona
   - Part 0b：全局蒸馏 — 从各 brief 提取跨项目通用事实 → 更新 persistent-context 对应章节
4. 更新 `AGENTS.md`：添加记忆指令识别规则和记忆维度归类表

#### 验收标准

- [ ] `wiki/cards/card-agent-persistent-context.md` 含 5 个标准章节
- [ ] 每日流水线含步骤 9.5（短期记忆抽取）
- [ ] 每周维护含 Part 0a 和 0b（长时记忆蒸馏）
- [ ] 手动写入一条测试 brief 后，执行蒸馏能将其晋升到 persona 或 persistent-context

---

### Module H：论文发现

**简介**：ingest 过程中自动检测论文引用（DOI / arXiv ID / PMID / PMCID / 论文标题），通过多工具链获取全文并关联到知识库。支持精确匹配原则（不降级到相似论文）和标题规范化消噪。

**价值**：你浏览的论文不再"看过就忘"——系统自动抓取全文、建立索引、关联到相关项目。

**依赖**：Core

#### Ask 问题

> 以下为「论文发现」的配置选项：

1. **文献管理工具**：你是否使用 Zotero？
   - 选项：是，已安装 Zotero 桌面版 + 浏览器 Connector / 是，但只用浏览器版 / 否（不使用）
2. **Zotero MCP**：如果使用 Zotero，是否启用了 Zotero MCP 插件（Agent 可直接操作 Zotero）？
   - 选项：是 / 否 / 不确定
3. **论文获取工具**：你的环境中有哪些论文获取工具可用？
   - 选项：paper-fetch / gs-skills / wos-skills / asta-skill / science-skills / deep-research / 以上都没有（Agent 会尝试直接访问 DOI 着陆页）

#### 实现步骤

1. 创建 `skills/lifecycle-paper-discovery.md`：
   - 获取策略优先级：精确标识符（DOI/arXiv/PMID）→ 标题精确匹配 → 已安装工具 → Zotero MCP → 手动路径
   - Paper-Exact 原则：不自动降级，不确定时记录 candidate 不标记已获取
   - 浏览器降级：工具被反爬阻断时记录而非替换
   - 标题规范化：尾部标点、版本后缀、周围散文、剪藏残留
   - 获取结果记录表：detected_paper / normalized_identifiers / normalized_title / tool_used / result / rejection_reason / fulltext_location / next_step
2. 更新 `skills/ingest-attachment.md` Step 4：如果来源涉及论文（PDF、学术文章链接），额外调用 `skills/lifecycle-paper-discovery.md`
3. 如果用户启用 Zotero MCP：
   - 在 `skills/lifecycle-paper-discovery.md` 中添加 Zotero MCP 优先路径（检查是否已存在 → 附加全文 → 放入对应 collection）
4. 更新 `skills/lifecycle-source-note.md` 模板：在 `## Trace` 段记录论文获取结果

#### 验收标准

- [ ] `skills/lifecycle-paper-discovery.md` 含完整的获取策略优先级和标题规范化规则
- [ ] `skills/ingest-attachment.md` 含论文检测 hook
- [ ] 对一个含 DOI 的测试来源，能成功识别论文并记录获取结果（获取成功或明确记录失败原因）

---

## 全局检索入口实现

如果用户在 1.5 轮选择了全局检索入口，记录的命令名称为 `$RETRIEVAL_CMD`（默认为 `xirang`）。在 Core 实现完成后，按以下策略配置：

### 检测目标

创建一个可在任意工作目录下调用的检索命令，功能等价于：在知识库 vault 目录下按 `skills/infra-retrieval.md` 定义的检索优先级搜索并返回结果。

### 平台适配策略

Agent 检测当前所在平台，按以下顺序尝试：

1. **Claude Code 全局命令**：如果当前 Agent 是 Claude Code，在 `~/.claude/` 下注册全局 CLAUDE.md 或使用 `/init` 注册全局 slash command，内容为：
   ```markdown
   ## 全局知识库检索
   当你收到以 `$RETRIEVAL_CMD` 开头的查询时，先读取 <VAULT_PATH>/skills/infra-retrieval.md，按检索优先级在 <VAULT_PATH> 中搜索回答。
   ```

2. **Reasonix 全局 workspace**：如果当前 Agent 是 Reasonix，在全局 reasonix.toml 中注册自定义命令，或创建全局 skill 文件。

3. **通用 shell 方案**（兜底）：在用户 PATH 中创建名为 `$RETRIEVAL_CMD` 的可执行脚本：
   - Windows: `%USERPROFILE%\bin\$RETRIEVAL_CMD.bat` 或 `%USERPROFILE%\AppData\Local\Microsoft\WindowsApps\$RETRIEVAL_CMD.ps1`
   - macOS/Linux: `~/.local/bin/$RETRIEVAL_CMD`
   脚本内容：切换到 vault 目录，调用当前 Agent 执行检索。

4. **如果 Agent 无法写入全局路径**：在 Post-Setup Checklist 中输出手动配置指引。

### 验收标准

- [ ] 在 vault 外任意目录执行 `$RETRIEVAL_CMD 某关键词`，能返回知识库中的相关内容
- [ ] 如果 Agent 无法自动配置，Post-Setup Checklist 中给出了准确的手动配置步骤

---

## Design Principles（设计原则）

### 1. 不依赖 Obsidian 插件做知识蒸馏

知识蒸馏由 Agent 直接调用 LLM API 完成，不依赖 Obsidian 插件生态（如 Text Generator、Templater、Dataview 的复杂查询逻辑等）。Obsidian 的价值在于 Markdown 可读性和 `[[wikilink]]` 双向链接，而非计算能力。

### 2. 四层知识架构

```
raw（不可变证据链）
  → source-notes（轻量索引 + citation，以来源为中心）
    → articles（独立实体百科词条，以实体为中心）
      → cards（跨 article 综合分析，以洞察为中心）
```

下层不包含上层内容，上层只引用下层做溯源。不跨层跳跃（如从 source note 直接跳到 card 而不经过 article）。

### 3. 检索优先于图谱可视化

系统将信息分层为"提炼后的知识"→"上下文"→"证据"→"过程材料"。Agent 回答问题时优先命中提炼后的知识与项目上下文，再按需回溯 source notes 和 registry 获取证据。raw 层不应成为第一检索入口。

### 4. Source note 是轻量索引，不是知识容器

Source note 的根本身份是"指向 raw 的证据链索引"，不是"知识本身"。实体级知识拆分到 article，跨 article 洞察升级为 card。Source note 只保留 citation + Summary + References。

### 5. 原始文件永不删除或修改

`raw/archived/` 中的文件是只读证据链。即使内容过时或错误，也只标记 superseded，不删除原文件。

---

## Retrieval-first Public Principle（开源壳层原则）

XiRang 的开源项目保持为简洁公共壳层：只同步可复用的设计原则和验收标准，不提交个人 vault 内容、skills 实现细节、source notes、logs、raw captures 或生成索引。

验收标准：
- 开源仓库根目录保持简洁，只保留 GOAL.md、README、LICENSE、AGENTS.md 骨架
- 私有 vault 的 `skills/`、`wiki/`、`raw/`、`logs/`、实体表、检索索引、图谱辅助文件不进入开源仓库
- README/GOAL 只同步通用思想，不复制个人路径、个人材料或具体知识库内容
- 每个可选模块只描述 "是什么、怎么实现、如何验证"，不包含任何具体知识内容

---

## Post-Setup Checklist（初始化完成后输出）

Agent 在完成所有模块的初始化后，必须执行以下检查和输出：

### 1. 平台能力检测

检测你当前所在 Agent 平台的能力：

| 能力 | 检测方式 | 如果不可用 |
|------|----------|------------|
| 定时任务注册 | 尝试注册 cron/schedule | 提醒用户手动添加 |
| 用户目录读写 | 尝试读取用户 home 目录 | 提醒用户手动复制文件 |
| 浏览器插件安装 | 不适用（Agent 无法操作浏览器） | 提醒用户手动安装 |

### 2. 生成用户待办清单

根据检测结果，输出仅包含 Agent **无法**自动完成的事项：

```
## 你需要手动完成的事项

### 定时任务
- [ ] 如果当前 Agent 平台不支持自动注册定时任务：
  请手动添加以下定时任务：
  Windows：打开"任务计划程序" → 创建基本任务
    - 日报：每天 [用户选择的时间] 触发，执行 "<agent命令> /schedule xirang-daily-pipeline"
    - 周报：每周日 [用户选择的时间] 触发，执行 "<agent命令> /schedule xirang-weekly-maintenance"
  macOS/Linux：添加 crontab
    - 0 [时间] * * * cd /path/to/vault && <command>

### 浏览器插件
- [ ] 安装 Obsidian Web Clipper 浏览器扩展（用于网页裁剪）
  链接：https://obsidian.md/clipper
  - Note location：raw/inbox/00_webclip
  - Note name：{{date|date:"YYYY-MM-DD"}} - {{title}}
- [ ] [仅当用户勾选 Module C 且使用 ChatGPT 时]
  安装「ChatGPT 对话保存助手」浏览器插件
  保存路径设为：<vault>/raw/inbox/60_chatgpt/ChatGPT-Backup/
- [ ] [仅当用户勾选 Module C 且使用 Gemini 时]
  安装「Gemini Chat Exporter」浏览器插件

### Obsidian 配置（可选）
- [ ] 安装 Dataview 插件（用于 MOC 动态查询）
- [ ] 安装 Obsidian Git 插件（或确保 vault 目录已 git init）

### 全局检索
- [ ] [如果 Agent 无法自动配置全局检索命令]
  手动创建全局检索入口：
  - Windows：在 `%USERPROFILE%\bin\$RETRIEVAL_CMD.bat` 中创建脚本，内容为切换到 vault 目录后调用 Agent 检索
  - macOS/Linux：在 `~/.local/bin/$RETRIEVAL_CMD` 创建可执行脚本
  Agent 会在上方给出具体的脚本内容。
```

### 3. 模块状态汇总

输出已安装模块清单及当前状态：

```markdown
## XiRang 初始化完成

### 已启用模块
| 模块 | 状态 | 备注 |
|------|------|------|
| Core | ✅ | 基础流水线就绪 |
| A: 实体百科 | ✅ | entity-registry 含 N 个初始实体 |
| ... | ... | ... |

### 下一步
1. 尝试上传一个文件或粘贴一张截图，观察 ingest 流水线是否正常工作
2. [如果有定时任务] 等待第一次日报自动生成
3. 阅读 AGENTS.md 了解系统规则
```
