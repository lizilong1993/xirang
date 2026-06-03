# XiRang / 息壤

基于 Obsidian 的第二知识系统，AI 自动维护。

## 每次对话强制触发规则

**你必须在每次对话的第一条消息时，自动执行以下完整流水线：**

1. 扫描 `raw/inbox/{00_webclip,20_cowork,30_screenshot,40_manual}` 中是否有新文件
2. 检查当前对话中用户是否发送了文件/截图/文本/网页
3. 对每个新来源执行：登记 `logs/source-registry.csv` → 生成 `wiki/source-notes/` → 更新 `wiki/persona/` → 更新 MOC 和 `wiki/index.md`
4. 向用户输出总结报告

**完整流水线口诀**：原始聊天/文本/截图/文件/网页 → raw inbox → Agent 登记 → 分类 → source note → MOC → Persona

这适用于任何 agent（Claude Code、Codex、Claude Cowork 等）的任何新会话。

## 文件入库

- PDF/DOCX/MD/代码 → `raw/inbox/20_cowork/`
- 截图/图片（软件界面、错误、终端）→ `raw/inbox/30_screenshot/`
- Web Clipper 保存 → `raw/inbox/00_webclip/`
- 手动放入 → `raw/inbox/40_manual/`
- 附件 bytes 不可访问时：明确报告，不得假装保存

## Source note

见 `skills/source-note.md`

## Persona

见 `skills/persona.md`。`wiki/persona/persona-<project>.md` 记录用户偏好和模式。

## 检索优先级

见 `skills/knowledge-retrieval.md`：vault → web → 标注。

## Registry

`logs/source-registry.csv` header：
`source_id,created_at,original_name,saved_path,capture_channel,content_type,domain,project,status,source_note,notes`

## Skills 定时优化

见 `skills/skill-review.md`。每周分析对话中反复强调但未固化的规则，补充到对应 skill。如 skill 有持续更新应同步修改 AGENTS.md。

## 同步开源

见 `skills/sync-opensource.md`。本机项目每次优化时 trigger 此 skill：
- 凝练优化思想，更新 xirang 的 `GOAL.md` 验收标准
- 同步更新 xirang 的 `README.md` / `README.en.md`
- 不复制文件，不暴露个人信息

## 禁止

修改/删除/移动 raw 文件。扫描 Desktop/Downloads。创建工程脚手架。
