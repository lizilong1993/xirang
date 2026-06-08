# Goal: 将当前 Obsidian vault 初始化并配置为 XiRang 知识系统

## Installation acceptance

1. **目录结构存在**：
   `raw/inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}`
   `raw/archived/`
   `wiki/{source-notes,persona,cards,moc}`
   `logs/`
   `logs/reports/{daily,weekly,monthly,quarterly,annual,reconciliation}`

2. **AGENTS.md 就位** — vault 根目录含自包含的 AGENTS.md，规则包括：
   - 文件/截图入库路径（含 agent_chat 渠道）
   - source note 模板（含 Persona Impact + Trace）
   - registry CSV header（含 status + confidence 列）
   - persona 更新规则
   - 检索优先级 + 通用回答规则（vault → 记忆 → web → 标注）
   - 冲突调和 + 过期标记
   - 卡片提炼规则
   - 归档规则（处理完移入 raw/archived/）
   - Git 版本管理
   - raw 只读、不扫用户目录
   - 质量控制规则（ChatGPT 等历史备份保留/丢弃标准）

3. **CLAUDE.md 就位** — 内容仅为 `AGENTS.md`

4. **来源注册表就绪** — `logs/source-registry.csv` 含 header：
   `source_id,created_at,original_name,saved_path,capture_channel,content_type,domain,project,status,confidence,source_note,notes`

5. **Vault 索引就绪** — `wiki/index.md` 存在

6. **Skills 就绪**：
   - `skills/source-note.md` — 含 confidence + superseded_by 字段
   - `skills/persona.md` — 按 persona_impact 更新
   - `skills/knowledge-retrieval.md`
   - `skills/ingest-attachment.md` — 含归档步骤
   - `skills/knowledge-card.md` — 知识卡片提炼
   - `skills/knowledge-reconciliation.md` — 冲突检测与调和
   - `skills/agent-chat-capture.md` — Agent 对话自动保存
   - `skills/skill-review.md`
   - `skills/sync-opensource.md`

## Runtime behavior expected

- 用户上传文件 → 保存到 `20_cowork/`，登记，生成 source note，归档
- 用户粘贴截图 → 保存到 `30_screenshot/`，提取界面/错误/路径等信息，生成 source note，归档
- 附件不可访问 → 明确报告限制
- Web Clipper 文件在 `00_webclip/` → 分类，生成 source note，归档
- Agent 对话保存到 `70_agent_chat/` → 走完整流水线
- intake 后评估 persona impact，更新 persona
- intake 后检测冲突 → 标记矛盾/过期

7. **设计原则 — 不依赖 Obsidian 插件做知识蒸馏**：知识蒸馏由 Agent 直接调用 LLM API 完成，不依赖 Obsidian 插件生态（如 Text Generator、Templater 等）。Obsidian 的价值在于 Markdown 可读性，而非计算能力。

## Retrieval-first public principle

XiRang 的开源项目保持为简洁公共壳层：只同步可复用的设计原则和验收标准，不提交个人 vault 内容、skills 实现、source notes、logs、raw captures 或生成索引。

检索优先于图谱可视化：系统应将信息分层为「提炼后的知识」「上下文」「证据」「过程材料」。Agent 回答问题时应优先命中提炼后的知识与项目上下文，再按需回溯 source notes 和 registry 获取证据；过程材料不应成为第一检索入口。

验收标准：
- 开源仓库根目录保持简洁，只保留项目说明、目标、许可和 agent 入口说明。
- 私有 vault 的 `skills/`、`wiki/`、`raw/`、`logs/`、实体表、检索索引、图谱辅助文件不进入开源仓库。
- README/GOAL 只同步通用思想，不复制个人路径、个人材料或具体知识库内容。
