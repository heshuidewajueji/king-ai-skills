---
name: subagent-handoff-protocol
description: Structured handoff protocol for multi-agent coordination. Covers mission brief, ownership boundaries, explicit done signals, and structured memory feedback.
---

# Subagent Handoff Protocol

## 核心问题

两个 Agent 互相交接时最容易出问题：
- 没有明确的完成信号
- 交接内容不完整
- 下一个 Agent 不知道上一个做到了哪一步

Maestro 风格的结构化交接解决这个问题。

---

## 交接协议

### 1. Mission Brief（任务发起时）

给子 Agent 的 brief 必须包含：

```
## 任务目标
[具体要做什么]

## 所有权边界
[这个 Agent 负责什么 | 不负责什么]

## 完成信号
[什么情况算完成 — 必须可验证]

## 上下文
[前期已完成的步骤 | 已知约束]
```

### 2. 交接时（上一个 → 下一个）

每个 handoff 必须包含：

```
## 已完成
- [具体输出物 + 路径]

## 进行中
- [当前状态 / 最后一个决策]

## 未完成（下一个接手）
- [还需要的步骤]

## 关键约束
- [不可违反的边界 / 用户明确要求]
```

### 3. Done 信号规范

交接时必须明确回答：
- 产出是什么？（具体文件/数据）
- 状态是什么？（pass/fail/blocked）
- 证据是什么？（路径/日志/测试结果）

不能让接收方猜测他做完了吗。

---

## 记忆反馈循环

### corrections/（纠正）

用户纠正了 Agent 的行为 → 写入 corrections 目录

格式：
```
## 纠正：<日期>
### 问题
<Agent 做错了什么>

### 正确做法
<用户要求怎么做>

### 防止重现
<以后遇到同类任务要注意什么>
```

### learnings/（学到）

成功完成了一个非平凡任务 → 写入 learnings 目录

格式：
```
## 学到：<日期>
### 任务类型
<什么类型的任务>

### 有效做法
<这次用了什么有效方法>

### 可复用场景
<以后在什么情况下用这个方法>
```

---

## 适用场景

- 同一任务需要多个子 Agent 接力
- 交接给另一个 session 继续
- subagent-driven-development 中的 two-stage review

## 相关Skill

- subagent-driven-development — 已有任务分发逻辑
- self-evaluation-checkpoint — 自检触发记忆沉淀
- systematic-debugging — 调试时的交接
