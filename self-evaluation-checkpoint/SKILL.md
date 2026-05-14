---
name: self-evaluation-checkpoint
description: Self-evaluation checkpoint every N tool calls. Assesses progress, decides whether to create or update a skill, and checks for errors or drift.
---

# Self-Evaluation Checkpoint

## 核心机制

每 15 步 tool call 触发一次自检暂停。

**目的：**
1. 这步有用吗？
2. 要不要沉淀成 Skill？
3. 有没有错误或偏离？

---

## 检查流程

### Step 1: 进度评估

回顾最近 5-10 步 tool calls：
- 当前任务完成度多少？
- 走的路和原计划一致吗？
- 有没有绕远路或重复劳动？

### Step 2: 结果检查

最近一步成功了吗？
- 成功 → 继续
- 失败 → 回滚到上一个确认状态

### Step 3: Skill 沉淀判断

这轮工作值不值得沉淀？

**沉淀条件**（满足任一）：
- 同一类任务成功执行 3次或以上
- 发现新的有效工作流（可以复用于其他场景）
- 用户纠正了错误方向（防止重蹈覆辙）

**不沉淀**：
- 一次性探索
- 用户明确说"仅这次"
- 纯查询类任务

### Step 4: 错误检测

检查是否出现：
- 同一错误被重复触发
- 工具返回意外格式
- 上下文明显偏离任务目标

---

## 输出

自检后给用户一个简洁状态：

```
[自检 15步] 完成度: 60% | 状态: 正常 | 发现可复用流程: 1个
```

或

```
[自检 15步] 问题: 重复查询失败 | 建议: 回退一步更换策略
```

---

## 与 Skill Factory 的区别

| | Self-Eval Checkpoint | Skill Factory |
|--|---------------------|---------------|
| 触发 | 每15步自动 | 同一任务3次以上 |
| 作用 | 评估加决策 | 自动生成Skill |
| 复杂度 | 低 | 高 |
| 当前实现 | 手动半自动 | 未实现 |

---

## 相关Skill

- book-to-skill-distiller — 人工沉淀流程
- systematic-debugging — 发现bug后的处理
- subagent-driven-development — 子任务进度管理
