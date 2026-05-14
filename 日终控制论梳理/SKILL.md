---
title: SKILL-日终控制论梳理
type: skill
status: active
tags:
  - 系统/Skill
  - 系统/控制论
created: 2026-05-10
updated: 2026-05-11
trigger: "用户说"今天收工"、"收工"、"review today"、或 cron 每日 21:00 自动触发"
inputs: "今日工作上下文 + session_search(今日关键决策) + _memory/ 当前所有卡片 + _skills/ 当前所有技能"
outputs: "日结报告存入 30-AI与工具/03-hermes/日结报告/日结报告 YYYY-MM-DD.md；session 摘要自动生成（session_search 工具）"
---

# SKILL：日终控制论梳理

## 触发条件

- 用户说"今天收工"、"收工"、"review today"
- 每日 21:00 cron 自动触发

---

## 执行协议

### 第一步：收集今日事件

从对话历史和工作记录中提取今日所有重要操作：
- 创建/修改了哪些笔记
- 执行了哪些工具调用
- 用户提供了哪些新信息
- 沉淀了哪些经验

### 第一步附：Session 自动摘要（使用 session_search 工具）

**强制执行**：在收集今日事件时，必须先调用 `session_search` 获取今日 session 数据。

**操作步骤**：
1. 调用 `session_search(query="YYYY-MM-DD 关键决策", limit=5)` — 搜索今日关键决策
2. 调用 `session_search(query="YYYY-MM-DD 新增 Skill", limit=3)` — 搜索今日新增的 skill
3. 调用 `session_search(query="YYYY-MM-DD 待办 OR 问题", limit=3)` — 搜索待处理项

**Session 摘要输出格式**（写入日结报告对应章节）：
```
## 今日 Session 摘要

| Session 主题 | 关键决策 | 产出 |
|-------------|---------|------|
| {主题1} | {决策1} | {文件/Skill} |
| {主题2} | {决策2} | {文件/Skill} |

**待跟进**：
- {问题1}
- {问题2}
```

如果 session_search 无结果（首次启动或无历史），跳过此节但需在报告中注明「无历史 session 数据」。

### 第二步：控制论五问复盘

对每件重要事项，强制过"控制论五问"：

1. **反馈在哪？**
   - 这个事项调节了哪个系统反馈回路？
   - 是负反馈（稳定）还是正反馈（放大）？

2. **传感器和执行器是谁？**
   - 信息从哪来？（传感器）
   - 控制指令作用在哪？（执行器）
   - 有没有不可观测或不可控的关键变量？

3. **延迟与带宽？**
   - 反馈回路中有哪些显著延迟？
   - 信息采样频率够不够？

4. **扰动源是什么？**
   - 最影响系统稳定性的扰动来自哪里？
   - 系统如何抵消它？

5. **层级是否合理？**
   - 各层级的控制频率是否匹配？
   - 有没有高频事务侵入低频决策层？

### 第三步：Skill 沉淀检查

检查今日操作中：
- 有没有重复性操作可以抽象为新 Skill？
- 有没有现有 Skill 因环境变化而需要标记 `deprecated`？
- 有没有具体步骤需要写入对应 Skill 文件？

### 第四步：记忆巩固检查

用控制论六准则扫描今日所有笔记，发现：
- 可迁移的底层逻辑（存为 _memory/ 卡片）
- 已知模式的验证或反例（更新已有 _memory/ 卡片）
- 控制论维度覆盖是否完整（六个准则中哪些已有卡片、哪些缺）

六准则分类：
1. 系统与反馈
2. 稳定性与鲁棒性
3. 可观性与可控性
4. 分层与接口
5. 最优性与约束
6. 时间尺度解耦

### 第五步：生成日结报告

输出文件路径：
`30-AI与工具/03-hermes/日结报告/日结报告 YYYY-MM-DD.md`

报告格式：

```markdown
# 日结报告 YYYY-MM-DD

## 今日 Session 摘要
（来自 session_search 工具自动生成，来自今日 Hermes session 记录）

| Session 主题 | 关键决策 | 产出 |
|-------------|---------|------|
| ... | ... | ... |

**待跟进**：...

## 控制论亮点
- 今天化解了哪个控制论问题？
- 本质是调节了哪个反馈回路？

## 新增记忆（_memory/）
- [[模式名]]：[一句话描述]

## 技能变更
- 新增 Skill：[[Skill名]]
- 废弃 Skill：[[Skill名]]（deprecated）
- 更新的 Skill：[[Skill名]]

## 系统健康度
- 六准则覆盖情况：哪些已有、哪些缺
- _memory/ 和 _skills/ 边界清晰度评估
- 下一个工作日的建议
```

---

## 存储位置规范

| 内容 | 路径 |
|------|------|
| 日结报告 | `30-AI与工具/03-hermes/日结报告/日结报告 YYYY-MM-DD.md` |
| 底层逻辑卡片 | `30-AI与工具/03-hermes/_memory/` |
| 操作技能文件 | `/Users/king-macbookair/.openclaw/king-ai-skills/{skill名}/SKILL.md` |
| _memory 索引 | `30-AI与工具/03-hermes/_memory/INDEX.md` |
| _skills 索引 | `/Users/king-macbookair/.openclaw/king-ai-skills/INDEX.md` |

---

## 注意事项

- **session_search 必须执行**：「第一步附」是强制步骤，不可跳过。首次运行时如无历史数据，需在报告中注明。
- 日结报告不追求长，关键是控制论视角的洞察
- 如果当日无重大进展，报告可以从简，但必须包含"控制论五问"的通过/未通过判定
- 发现新的底层逻辑时，优先更新 _memory/ INDEX.md 的分类索引
