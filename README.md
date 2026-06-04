# XiRang / 息壤

**AI-maintained second knowledge system for Obsidian**
**由 AI 自动维护的 Obsidian 第二知识系统**

> Capture everything. Let knowledge grow.
> 捕获一切资料，让知识自行生长。

XiRang（息壤）是一个面向 Obsidian + AI Agent 的本地第二知识系统。用户在对话中上传的文件、粘贴的截图、Web Clipper 保存的网页和手动放入的资料被自动归档到 raw inbox，再由 AI Agent 自动登记、分类、生成 source note、维护 MOC，逐步编译成可追溯的知识网络。

**本版本新增**：知识冲突检测与调和机制、自动知识卡片提炼、Agent 对话自动保存、原始文件安全归档与 Git 版本管理。

## Why XiRang

大多数人用 Obsidian 收藏大量资料，但很少回去整理。XiRang 让 AI Agent 替你完成这个环节：你只管丢进来，系统自动处理。

## How it works

```
你粘贴截图 / 上传文件 / 保存网页 / Agent 对话结束
       ↓
  raw/inbox/{cowork,screenshot,webclip,agent_chat}
       ↓
  AI Agent: 登记 → 分类 → source note → 冲突检测 → 卡片提炼 → MOC → Persona
       ↓
  原始文件归档 raw/archived/ → Git 提交
```

### Key Features

- **Persona 层** — 每次 intake 更新用户画像（偏好、模式、踩坑记录），源自 TencentDB Agent Memory 的四层记忆金字塔
- **知识冲突调和** — 自动检测新来源与已有知识的事实矛盾，标记过期/冲突，保持知识库准确
- **知识卡片提炼** — 同一 domain 积累 ≥3 篇 source note 后自动提炼为结构化知识卡片
- **Skill 每周自优化** — `skills/` 目录存放模块化规则，每周定时任务回顾对话、凝练不足、自动优化，规则持续进化

## Minimal setup

将本仓库文件放入你的 Obsidian vault 根目录。

如果你使用 Claude Code，执行：

```bash
/goal follow "/path/to/your/vault/GOAL.md"
```

如果你使用 Claude Cowork、Codex 或其他 agent，把 `GOAL.md` 内容粘贴给 agent，要求它在 vault 根目录执行。

## Inbox structure

```
vault/
├── raw/
│   ├── inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}/
│   └── archived/YYYY-MM/  # 处理后归档（Git tracked）
├── wiki/source-notes/  # 来源笔记（含 Persona Impact + Trace + Confidence）
├── wiki/persona/       # 用户画像
├── wiki/cards/         # 多来源提炼的知识卡片（v2 新增）
├── wiki/moc/           # 内容地图（v2 新增）
├── logs/source-registry.csv  # 含 status + confidence 状态列
├── logs/reports/       # 定时报告（daily/weekly/monthly/quarterly/annual + reconciliation）
├── skills/             # 模块化 AI 操作规则
├── AGENTS.md           ← AI 规则（核心）
├── CLAUDE.md           ← 指向 AGENTS.md
└── GOAL.md
```

## Screenshot ingestion

XiRang 的主路径不需要外部截图工具。直接把截图粘贴或上传到对话中，AI Agent 应将其保存到 `raw/inbox/30_screenshot/`，提取界面状态、错误、路径等信息，生成 source note。

如果当前 Agent 无法访问附件 bytes，它必须明确报告，不得假装已保存。

## Web Clipper setup

安装 [Obsidian Web Clipper 浏览器扩展](https://obsidian.com/clipper)，创建或编辑模板：

- Note location：`raw/inbox/00_webclip`
- Note name：`{{date|date:"YYYY-MM-DD"}} - {{title}}`
- Note content 建议 frontmatter：

```markdown
---
capture_channel: webclip
processing_status: new
domain: unknown
project: unknown
source_url: "{{url}}"
captured_at: "{{date|date:"YYYY-MM-DD HH:mm"}}"
---

# {{title}}

Source: {{url}}

{{content}}
```

Web Clipper 只负责保存原始网页。分类和编译由 XiRang agents 完成。

## Agent behavior

| 你做的事 | AI 自动响应 |
|----------|-------------|
| 在对话中上传文件 | 保存到 `20_cowork/`，登记来源，生成 source note，归档 |
| 在对话中粘贴截图 | 保存到 `30_screenshot/`，提取关键信息，生成 source note，归档 |
| Web Clipper 保存网页 | 扫描 `00_webclip/`，分类，生成 source note，归档 |
| Agent 对话结束 | 自动保存到 `70_agent_chat/`，走完整流水线 |
| 放置资料到 inbox | 扫描并处理，冲突检测，卡片提炼，归档，Git 提交 |
| 提问 | 优先检索 vault → 持久化记忆 → web → 模型标注生成 |

## Credits

- **Andrej Karpathy** — LLM Wiki 理念启发本项目的知识组织方式
- **TencentDB Agent Memory** — 四层记忆金字塔（L0-L3）设计启发 Persona 层集成
- [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) — 上游参考方案
- [obsidian-skills](https://github.com/kepano/obsidian-skills) — 上游参考方案
- **实践启发** — 知识冲突调和、卡片提炼、Agent 对话保存、归档流程等优化来自本机长期运行与迭代的实践经验

## License

MIT
