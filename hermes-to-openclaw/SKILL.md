---
name: hermes-to-openclaw
description: 将任务转发给 OpenClaw Agent 执行。当需要项目管理、TBOX 相关任务执行、飞书群推送时使用此 skill。
---

# Hermes → OpenClaw 任务转发

当 Hermes 分析用户需求后，需要 OpenClaw 执行时，有两种方式：

## 方式一：MCP 调用（需要先配置）

**前提：** OpenClaw 的 `serve` 插件已启用（`plugins.allow` 包含 `"serve"`）

```
Hermes → MCP → OpenClaw
```

配置步骤：
1. 在 `~/.openclaw/openclaw.json` 的 `plugins.allow` 添加 `"serve"`
2. `hermes mcp add openclaw --command openclaw --args serve`
3. 重启 hermes

## 方式二：任务文件（当前使用）

写入任务到 `/Users/king-macbookair/.openclaw/incoming-tasks/task_YYYYMMDD_HHMMSS.json`

## 使用场景

- 用户需要项目管理操作（如任务状态更新）
- 需要在飞书群推送消息
- TBOX 项目管理任务
- 需要 OpenClaw 的 PM Agent 处理的任务

## 执行方式

写入任务到 `/Users/king-macbookair/.openclaw/incoming-tasks/task_YYYYMMDD_HHMMSS.json`

## 文件格式

```json
{
  "id": "task_YYYYMMDD_HHMMSS",
  "timestamp": "ISO 时间",
  "user": "Hermes 飞书用户 ID",
  "message": "原始用户消息",
  "task_type": "pm|message|analysis",
  "target_agent": "main|pm-director|pm-huaying|pm-ruichi|pm-scania",
  "priority": "high|medium|low",
  "instructions": "给 OpenClaw 的详细执行指令"
}
```

## 示例

用户说：「更新海格项目状态」

→ 写入任务文件，target_agent=pm-huaying，instructions=更新海格项目当前状态

## 重要提示

- 所有新知识/洞察也要同步写入 Obsidian Vault
- 不要等待 OpenClaw 执行结果，直接告诉用户「已转发给执行层」
- 如果需要回复用户，先在 Hermes 完成，OpenClaw 做后台执行
