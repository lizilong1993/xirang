# Goal: 将当前 Obsidian vault 初始化并配置为 XiRang 知识系统

## Installation acceptance

1. **目录结构存在**：
   `raw/inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review}`
   `wiki/{source-notes,persona,cards,moc}`
   `logs/`

2. **AGENTS.md 就位** — vault 根目录含自包含的 AGENTS.md，规则包括：
   - 文件/截图入库路径
   - source note 模板（含 Persona Impact + Trace）
   - registry CSV header
   - persona 更新规则
   - 检索优先级（vault → web → 标注）
   - raw 只读、不扫用户目录
   - 质量控制规则（ChatGPT 等历史备份保留/丢弃标准）

3. **CLAUDE.md 就位** — 内容仅为 `AGENTS.md`

4. **来源注册表就绪** — `logs/source-registry.csv` 含 header：
   `source_id,created_at,original_name,saved_path,capture_channel,content_type,domain,project,status,source_note,notes`

5. **Vault 索引就绪** — `wiki/index.md` 存在

## Runtime behavior expected

- 用户上传文件 → 保存到 `20_cowork/`，登记，生成 source note
- 用户粘贴截图 → 保存到 `30_screenshot/`，提取界面/错误/路径等信息，生成 source note
- 附件不可访问 → 明确报告限制
- Web Clipper 文件在 `00_webclip/` → 分类，生成 source note
- intake 后评估 persona impact，更新 persona

## Constraints

- 不要创建 scripts、docs、examples、workflows、vendor 等目录
- AGENTS.md 自包含，不依赖外部分散的 rule 文件
- 保持最小化

## 定时凝练与优化规则

每周通过定时任务调用 `skills/` 目录中的 skill 文件来优化规则：

1. agent 读取 `skills/` 下的所有 skill 文件
2. 回顾过去一周的实际操作，从对话中提炼出值得固化的规则
3. 发现规则与实际操作有偏差时修改对应 skill
4. 如果发现skills内容修改确实导致CLAUDE.md 需要同步更新时，动态修改 CLAUDE.md；否则保持 CLAUDE.md 最小化
5. 修改过的 skill 写入 `logs/reports/skill-review-YYYY-MM-DD-weekly.md`
6. 不把 skill 的内容直接嵌入 CLAUDE.md；保持简洁索引风格

验收指标：
- `skills/` 目录存在且包含所有必要 skill 文件
- 定时任务能正常读取和对比规则与执行情况
- skill 如有持续更新则 CLAUDE.md 随之变更部分内容

## 不含错误表述

- 不存在"必须用 ShareX"、"截图必须先手动保存"、"AI 不能自动保存截图"等表述
