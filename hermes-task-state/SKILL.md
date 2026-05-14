---
name: hermes-task-state
description: Hermes跨天任务状态持久化 — 解决Agent跨天"失忆"问题，会话开始时自动恢复任务进度
triggers:
  - session_start
  - task_update
  - cron_daily
tags: [memory, task, persistence, cross-day]
version: 1.0.0
---

# Hermes Task-State — 跨天任务状态持久化

## 目的
解决 Hermes 跨天任务"失忆"问题。任务状态保存到文件，第二天自动恢复。

## 文件结构
```
~/.hermes/task-state/
├── current-tasks.md      # 当前进行中任务
├── project-checkpoints.md # 项目进度检查点
└── task-history.md       # 任务完成历史
```

## 核心文件格式

### current-tasks.md
```markdown
# 当前任务状态
最后更新：{{date}}

## 进行中任务
| # | 任务 | 项目 | 进度 | 下一步 | 状态 |
|---|------|------|------|--------|------|
| 1 | 任务描述 | 项目名 | 30% | 下一步行动 | ⏳进行中 |

## 阻塞/等待
| # | 问题 | 影响 | 等待方 |
|---|------|------|--------|
| 1 | 待XX反馈 | 影响进度 | XX |

## 已完成（今日）
- 完成的任务简述
```

## 触发时机

### 1. 会话开始时（自动加载）
每次会话开始，读取 `~/.hermes/task-state/current-tasks.md`，如果有进行中任务，显示任务列表。

### 2. 任务有重大进展时（立即写入）
```bash
python3 << 'EOF'
from hermes_tools import write_file
from datetime import datetime
path = os.path.expanduser("~/.hermes/task-state/current-tasks.md")
# 更新任务进度
EOF
```

### 3. 会话结束前（必须写入）
将当前任务状态写入 `current-tasks.md`。

### 4. 每日 Cron 自动检查
每日 22:00 自动读取任务状态并更新。

## 核心操作

### 读取当前任务
```python
from hermes_tools import read_file
tasks = read_file("~/.hermes/task-state/current-tasks.md")
```

### 更新任务状态
```python
from hermes_tools import write_file
from datetime import datetime

def update_task_state(tasks_md_content):
    header = f"# 当前任务状态\n最后更新：{datetime.now().strftime('%Y-%m-%d %H:%M')}\n\n"
    write_file("~/.hermes/task-state/current-tasks.md", header + tasks_md_content)
```

### 标记任务完成
在 current-tasks.md 中移动到"已完成（今日）"列表。

## 重要规则
- ❌ **禁止**：任务状态只放在对话里，不写文件
- ✅ **必须**：任何进行中任务必须写入 `~/.hermes/task-state/`
- ✅ **必须**：任务完成后标记为已完成

## 需求编译阶段追踪（requirement-compiler联动）

当进行需求编译任务时，使用 P0→P1→P2→P3→P4 阶段追踪：

```markdown
## 进行中需求编译

| 阶段 | 状态 | 说明 |
|------|------|------|
| P0 基线 | ✅ | Dumb Baseline + 角色设定 |
| P1 澄清 | 🔄进行中 | 视角分布验证（>25%）|
| P2 生成 | ⏳待开始 | 收敛条件：覆盖度/重复率/轮次上限 |
| P3 门禁 | ⏳待开始 | 模糊词/可测试性/宪法合规 |
| P4 审查 | ⏳待开始 | Verifier + 基线对比 |
```

### 阶段流转规则
- P0→P1：角色设定完成，视角分布验证通过
- P1→P2：收敛条件满足（覆盖度>80%，重复率<20%）
- P2→P3：生成完成，进入三重门禁
- P3→P4：三重门禁全部通过
- P4→完成：Verifier审查通过

### 阶段阻塞检测
如果在某阶段阻塞，自动记录到「阻塞/等待」：
```markdown
## 阻塞/等待
| 阶段 | 问题 | 影响 | 解决方案 |
|------|------|------|----------|
| P2 | 覆盖度仅60% | 无法进入P3 | 增加需求维度X |
```

## 交付自检清单

每次交付前必须逐项核对：

- [ ] 无自行脑补或默认推演
- [ ] 无冗余功能/抽象/预留设计
- [ ] 无超范围改动
- [ ] 无口语化/客套/AI套路表达
- [ ] 不确定内容已明确标注
- [ ] 结果符合初始验收标准（含负面路径验证）
- [ ] 若发现已交付内容违反以上约束：立即修正并说明，不掩饰

## Cron 集成
```yaml
# 每日 22:00 检查任务状态
cronjob:
  name: "Hermes Task-State Check"
  schedule: "0 22 * * *"
  prompt: "读取 ~/.hermes/task-state/current-tasks.md，如有进行中任务，整理今日进度并更新文件"
  deliver: "origin"
```
