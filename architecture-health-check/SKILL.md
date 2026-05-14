---
name: architecture-health-check
description: "定期检查代码库架构健康度，发现深层模块机会（deep module）和架构退化。适用：每几天跑一次/代码库维护/接手新项目/重构前。触发词：'架构检查'、'代码健康'、'架构退化'、'深层模块'、'代码库诊断'。"
version: 1.0.0
author: Hermes Agent (inspired by mattpocock/improve-codebase-architecture, MIT License)
license: MIT
metadata:
  hermes:
    tags: [架构, 代码质量, 重构, 深层模块, mattpocock]
    related_skills: [systematic-debugging, writing-plans, subagent-driven-development]
    domain: 通用
    module: 软件工程
---

# Architecture Health Check — 代码库架构健康检查

> Source: mattpocock/improve-codebase-architecture (MIT License)  
> 核心思想：每次编程都在建造或拆除系统。要持续关注代码设计，而非只在"重构"时才想起。

---

## 触发条件

**触发此 Skill 的时机：**
- 每隔几天定期检查（建议每周一次）
- 接手新代码库时
- 在重大重构之前
- 用户说"架构检查"、"代码健康"
- 长时间没有关注代码架构时

> **原则：架构健康是持续维护，不是突击检查。**

---

## 核心检查维度

### 维度一：深层模块 vs 浅层模块

**深层模块（Deep Module）**：接口小，功能深。
```
✅ 好例子：Python `set` — 5个方法（add/remove/discard/union/intersection）
               但背后支持 O(1) 查找、交集、并集、差集

✅ 好例子：Unix `grep` — 3个参数（pattern, file, options）
               但支持正则、递归、多文件、上下文
```

**浅层模块（Shallow Module）**：接口大，功能浅。
```
❌ 坏例子：一个工具类有 50 个方法，每个方法只有 3 行
           或者是包装了一层但什么都没做的 facade
```

**如何发现浅层模块：**
- 一个文件超过 20 个导出函数/类
- 模块名是 `Utils`、`Helper`、`Manager`、`Service`
- 模块文档说明写的是"这个模块负责 X"而不是"这个模块做什么"
- 很多 `export { x, y, z }` 从其他模块重导出

### 维度二：模块依赖是否合理

**检查依赖方向：**
- 高层模块（业务逻辑）不应被低层模块（工具函数）污染
- 不应有循环依赖（A 引用 B，B 引用 A）
- 跨模块调用应该通过稳定的接口

**常见问题模式：**
```
❌ 跨层直接调用：业务逻辑层直接调用数据库连接层之外，还调用 UI 组件
❌ 循环依赖：utils → logger → config → utils
❌ 星形依赖：一个中心模块被所有模块引用，自己却依赖别人
```

### 维度三：命名是否反映领域

**检查清单：**
- [ ] 模块名是否用业务语言而非技术语言？
  - ❌ `DatabaseConnectionManager`
  - ✅ `OrderRepository` / `CustomerLookup`
- [ ] 函数名是否说明"做什么"而非"怎么做"？
  - ❌ `getDataFromDbById()`
  - ✅ `findCustomerById()`
- [ ] 同一个概念是否在代码库中用同一个名字？

**操作方法：**
```bash
# 列出所有顶层目录，看命名是否反映了业务领域
ls src/

# 搜索重复概念的相似命名
grep -r "customer\|client\|user\|account" src/ --include="*.py" | head -20
```

### 维度四：变更集中度

**好的架构**：一个变更应该只在少数几个模块/文件中发生。

**检查方法：**
```
1. 随机选一个业务概念（比如"订单"）
2. grep 整个代码库，找出所有包含"订单"的文件
3. 如果超过 10 个文件涉及这个概念 → 说明概念定义不集中
```

**信号说明：**
- 一个概念散落在 20 个文件里 → 当业务变更时，修改点太多
- 一个文件被 50 个其他文件引用 → 这个文件不能随便改，改了影响太大

### 维度五：接口 vs 实现耦合

**检查：**
- 模块是否暴露了内部实现细节？（返回原始对象而不是接口）
- 是否有测试可以证明模块的接口是稳定的？

---

## 架构改进机会评估

### 给每个问题打分（干预优先级）

| 问题 | 评估维度 | 分数 |
|------|---------|------|
| 浅层模块（>20导出） | 影响：新人不理解 + 修改风险高 | 8 |
| 跨层直接调用 | 影响：改动时级联失败 | 9 |
| 循环依赖 | 影响：无法单独测试模块 | 10 |
| 命名不反映领域 | 影响：代码不可读 | 5 |
| 变更集中度差 | 影响：改动成本高 | 7 |

**分数 >7 的问题 → 优先处理**  
**分数 5-7 的问题 → 列入下次迭代**  
**分数 <5 的问题 → 暂时忽略**

---

## 执行流程

### 第一步：选择检查范围

**两种模式：**
- **全量扫描**：整个代码库（适合接手新项目时）
- **增量检查**：最近变更的文件（适合定期维护）

### 第二步：运行诊断

按"核心检查维度"逐项扫描，记录问题：

```
## 架构健康报告

### 深层模块机会
- [ ] `src/utils/math.py` — 浅层工具函数，可以合并到业务模块

### 循环依赖
- [ ] `auth.py ↔ session.py` 存在循环依赖

### 命名问题
- [ ] `CustomerService` 和 `ClientManager` 指同一个概念，需统一

### 变更集中度
- [ ] "订单"概念散落在 23 个文件中，建议抽取 `domain/order.py`
```

### 第三步：与用户确认优先级

把诊断报告给用户看，问：

> "这几个问题，你觉得哪个最影响日常开发？我们可以先处理这一个。"

### 第四步：制定改进计划

对高优先级问题，用 `writing-plans` 写改进计划。  
**注意：架构改进应该用小的、增量的方式逐步做，不是一口气重构。**

---

## 什么时候不需要这个 Skill

- 代码库 < 1000 行且没有活跃开发 → 不需要
- 只有一个开发者在维护 → 架构问题还没显现
- 项目即将废弃 → 架构改进投入产出比太低

---

## 快速参考卡

```
架构健康三问：

1. 这个模块是深还是浅？
   深 = 接口小，功能强 ✅
   浅 = 接口大，功能弱 ❌

2. 依赖方向是否合理？
   业务层不应依赖工具层
   不应有循环依赖

3. 改名能一搜就找到所有相关代码吗？
   能 ✅ → 概念集中
   不能 ❌ → 概念分散

每周一次小检查
每次接手新项目全量扫描
重构前必须跑一次
```

---

## 参考来源

- mattpocock/improve-codebase-architecture (MIT License, github.com/mattpocock/skills)
- A Philosophy of Software Design, John Ousterhout — Deep vs Shallow Modules
- Extreme Programming Explained, Kent Beck — Invest in design every day
