# XiRang / 息壤

[English](./README.md)

**由 AI 自动维护的 Obsidian 第二知识系统**

> 捕获一切资料，让知识自行生长。

文件、截图、网页、Agent 对话——丢进来就行。XiRang 自动走完登记、分类、提炼、归档全程。不依赖任何 Obsidian 插件。

**模块化设计**。Core 给你一套可用的 ingest 流水线。8 个可选模块（实体百科、知识卡片、多源 Agent 同步、定时流水线、知识雷达、知识访谈、记忆系统、论文发现）初始化时按需勾选，Agent 问清楚再动手。

## 工作流

```
粘贴截图 / 上传文件 / 保存网页 / Agent 对话结束
       ↓
  raw/inbox/{cowork,screenshot,webclip,agent_chat}
       ↓
  AI Agent: 登记 → 分类 → source note → Persona → MOC → 归档
       ↓  （模块扩展：entity split → card → radar → interview → ...）
  原始文件归档 raw/archived/ → Git 提交
```

### Core（必装）

- **自动流水线** — 接收 → 质量控制 → 注册 → source note → Persona → MOC → 归档 → Git，不需要手动操作
- **Persona 层** — 每次 intake 更新用户画像（偏好、模式、踩坑记录），按项目分类维护
- **检索优先** — 回答时按序命中：cards → persona → source-notes → registry → memory → web
- **安全归档** — raw/archived/ 不可变，Git 追踪

### 可选模块

| 模块 | 说明 |
|------|------|
| 实体百科系统 | 从 source note 拆出独立实体创建百科词条，双轴分类，强制 wikilink |
| 知识卡片提炼 | 同 domain ≥3 篇 → 结构化卡片，带证据表和争议追踪 |
| 多源 Agent 同步 | 自动同步 Claude Code / Codex / Reasonix / ChatGPT 等多平台会话 |
| 定时流水线 | 每日自动同步+抽取+日报，每周蒸馏+健康检查+skill review |
| 知识雷达 | 跨项目工具/技术推荐引擎 |
| 知识访谈 | 发现知识缺口，精准提问并回写答案 |
| 记忆系统 | 短时项目 brief + 长时蒸馏 + 跨项目 persistent context |
| 论文发现 | 检测 DOI/arXiv/PMID，多工具链获取全文 |

## 初始化

把仓库文件放到你的 Obsidian vault 根目录。

**Claude Code**：

```bash
/goal follow "/path/to/your/vault/GOAL.md"
```

**其他 agent**（Codex、Cowork、Reasonix 等）：把 GOAL.md 内容粘贴给 agent，告诉它在 vault 根目录执行。

## Core 目录结构

模块会额外添加目录（如 `wiki/articles/`、`wiki/radar/`、`schedule/`），详见 GOAL.md。

```
vault/
├── raw/
│   ├── inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}/
│   └── archived/YYYY-MM/
├── wiki/source-notes/
├── wiki/persona/
├── wiki/moc/
├── wiki/index.md
├── logs/source-registry.csv
├── logs/reports/
├── skills/
├── AGENTS.md           ← AI 规则入口
├── CLAUDE.md           ← 指向 AGENTS.md
└── GOAL.md
```

## 截图处理

直接把截图贴到对话里。Agent 保存到 `30_screenshot/`，提取界面信息（错误、路径、状态），生成 source note。

如果 Agent 读不到附件，它会告诉你——不会假装存了。

## Web Clipper

安装 [Obsidian Web Clipper](https://obsidian.com/clipper)，配置：

- Note location：`raw/inbox/00_webclip`
- Note name：`{{date|date:"YYYY-MM-DD"}} - {{title}}`

推荐 frontmatter：`capture_channel`、`processing_status`、`domain`、`project`、`source_url`、`captured_at`。

## Agent 行为（Core）

| 你做的事 | Agent 的响应 |
|----------|-------------|
| 上传文件 | 保存到 `20_cowork/`，登记，source note，归档 |
| 粘贴截图 | 保存到 `30_screenshot/`，提取信息，source note，归档 |
| Web Clipper 保存网页 | 扫描 `00_webclip/`，分类，source note，归档 |
| 资料丢进 inbox | 扫描处理，归档，Git 提交 |
| 提问 | 查 vault → 记忆 → web → 模型知识（标注） |

模块会扩展流水线：A 加 entity split，B 在 ≥3 篇时提炼卡片，C 同步多平台会话，以此类推。

## Credits

- **Andrej Karpathy** — LLM Wiki 概念启发
- **TencentDB Agent Memory** — 记忆金字塔启发 Persona 层
- **isEris** — Obsidian 插件 vs IDE+API 蒸馏方案对比验证
- [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) — 上游参考
- [obsidian-skills](https://github.com/kepano/obsidian-skills) — 上游参考
- **[Reasonix](https://reasonix.ai)** — Agent 编排框架，日常流水线工具
- **DeepSeek** — GOAL.md 模块化与交互逻辑设计
- 长期迭代实践 — 冲突调和、卡片提炼、会话保存、安全归档

## License

MIT
