---
name: systematic-debugging
description: "6-phase root cause debugging: build feedback loop → reproduce → hypothesise (3-5 ranked) → instrument → fix → regression-test. Adapted from mattpocock/diagnose + obra/superpowers."
version: 2.0.0
author: Hermes Agent (adapted from obra/superpowers)
license: MIT
metadata:
  hermes:
    tags: [debugging, troubleshooting, problem-solving, root-cause, investigation]
    related_skills: [test-driven-development, writing-plans, subagent-driven-development]
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)
- Someone wants it fixed NOW (systematic is faster than thrashing)

## The Four Phases

You MUST complete each phase before proceeding to the next.

---

## Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

### 1. Read Error Messages Carefully

- Don't skip past errors or warnings
- They often contain the exact solution
- Read stack traces completely
- Note line numbers, file paths, error codes

**Action:** Use `read_file` on the relevant source files. Use `search_files` to find the error string in the codebase.

### 2. Reproduce Consistently

- Can you trigger it reliably?
- What are the exact steps?
- Does it happen every time?
- If not reproducible → gather more data, don't guess

**Action:** Use the `terminal` tool to run the failing test or trigger the bug:

```bash
# Run specific failing test
pytest tests/test_module.py::test_name -v

# Run with verbose output
pytest tests/test_module.py -v --tb=long
```

### 3. Check Recent Changes

- What changed that could cause this?
- Git diff, recent commits
- New dependencies, config changes

**Action:**

```bash
# Recent commits
git log --oneline -10

# Uncommitted changes
git diff

# Changes in specific file
git log -p --follow src/problematic_file.py | head -100
```

### 4. Gather Evidence in Multi-Component Systems

**WHEN system has multiple components (API → service → database, CI → build → deploy):**

**BEFORE proposing fixes, add diagnostic instrumentation:**

For EACH component boundary:
- Log what data enters the component
- Log what data exits the component
- Verify environment/config propagation
- Check state at each layer

Run once to gather evidence showing WHERE it breaks.
THEN analyze evidence to identify the failing component.
THEN investigate that specific component.

### 5. Trace Data Flow

**WHEN error is deep in the call stack:**

- Where does the bad value originate?
- What called this function with the bad value?
- Keep tracing upstream until you find the source
- Fix at the source, not at the symptom

**Action:** Use `search_files` to trace references:

```python
# Find where the function is called
search_files("function_name(", path="src/", file_glob="*.py")

# Find where the variable is set
search_files("variable_name\\s*=", path="src/", file_glob="*.py")
```

### Phase 1 Completion Checklist

- [ ] Error messages fully read and understood
- [ ] Issue reproduced consistently
- [ ] Recent changes identified and reviewed
- [ ] Evidence gathered (logs, state, data flow)
- [ ] Problem isolated to specific component/code
- [ ] Root cause hypothesis formed

**STOP:** Do not proceed to Phase 2 until you understand WHY it's happening.

---

## Phase 1: Build a Feedback Loop（前置核心）

> **Source: mattpocock/diagnose skill (MIT License) — 系统调试的核心是构造正确的反馈回路**
> **如果有一个快速的、可由 agent 运行的 bug 反馈信号，你就能找到根因。如果没有，无论看多少代码都救不了你。**

### 10 种构造反馈回路的方法

| # | 方法 | 适用场景 |
|---|------|---------|
| 1 | **Failing test**（在触达 bug 的接口层写单元/集成/e2e 测试） | 首选，只要有接口可用 |
| 2 | **Curl / HTTP script**（对运行的 dev server 发请求） | API 级 bug |
| 3 | **CLI invocation**（命令行 + 固定输入 + stdout diff 对比已知正确结果） | CLI 工具 |
| 4 | **Headless browser script**（Playwright/Puppeteer — 断言 DOM/console/network） | UI bug |
| 5 | **Replay captured trace**（把网络请求/payload/event log 保存到磁盘回放） | 网络依赖 bug |
| 6 | **Throwaway harness**（最小化子集 + mock 依赖） | 复杂系统 bug |
| 7 | **Property/fuzz loop**（跑 1000 次随机输入） | "有时候输出错误" bug |
| 8 | **Bisection harness**（git bisect run 用状态检查自动化） | commit 之间的回归 |
| 9 | **Differential loop**（旧版 vs 新版，diff 输出） | 版本对比 |
| 10 | **HITL bash script**（用结构化循环驱动人工，最后手段） | 需要人工交互 |

**回路迭代原则：**
- **更快？** — 缓存 setup，跳过无关初始化，缩小测试范围
- **更准？** — 断言具体症状，不是"没崩溃"就完了
- **更确定？** — 固定时间，seed RNG，隔离文件系统，冻结网络

> 30 秒的不稳定回路和没有差不多。2 秒的确定性回路是调试超能力。

**不确定 bug（Non-deterministic）：** 目标是**提高复现率**，不是完美复现。50% 的 flaky bug 可以调试；1% 不行。持续提高复现率直到可调试。

**真的无法构造回路时**：明确说出来，列出试过的方法，问用户要复现环境访问权限或抓取的 artifact（HAR 文件、日志 dump、core dump）。**在有回路之前，不要进入 Phase 2。**

---

## Phase 2: Reproduce

**跑回路，看 bug 出现。确认：**
- [ ] 回路产生**用户描述的**失败模式——不是相近但不同的失败
- [ ] 失败在多次运行中可复现（或达到可调试的频率）
- [ ] 精确症状已捕获（错误信息、错误输出、慢的时间）

> **Wrong bug = wrong fix.** 在完美复现之前，不要进入下一步。

---

## Phase 3: Hypothesise（关键——生成 3-5 个假设再验证）

> **Source: mattpocock/diagnose skill — 单一假设生成会锚定在第一个看似合理的想法上**

**生成 3-5 个假设，按可能性排序，展示给用户后再测试。**

### 每个假设必须可证伪

```
If <原因> 是问题，那么 <测试> 会让 bug 消失 / <测试> 会让 bug 恶化
```

如果无法陈述预测，这个假设只是"感觉对"——丢弃或深化它。

### 假设格式示例

```
假设 #1（最可能）：数据库连接池耗尽
  依据：请求堆积 + 最近部署了新的连接初始化逻辑
  预测：减少并发请求数 → 错误消失

假设 #2：Redis 缓存失效导致大量回源
  依据：错误出现在缓存过期时间点附近
  预测：清除缓存 → 错误消失

假设 #3：API 超时阈值过低
  依据：慢查询日志时间戳与错误时间吻合
  预测：调高超时阈值 → 错误减少
```

> 把排序结果展示给用户再测试。用户往往有领域知识，能立即重新排序（"我们刚部署了 #3 的变更"）。如果用户离线，按自己的排序继续。

---

## Phase 4: Instrument

每个探针必须对应 Phase 3 的特定预测。**一次只改一个变量。**

**工具优先级：**
1. **Debugger / REPL 调试** — 一个断点胜过十个日志
2. **定向日志** — 在区分假设的边界处加日志

**调试日志标签系统：**
```
[DEBUG-a4f2]
```
所有带标签的日志在最后清理时只需 `grep` 一次即删除。无标签的日志保留。

**性能回归**：日志通常不可信。先建立基准测量（timing harness / `performance.now()` / profiler / query plan），再 bisect。**先测量，后修复。**

---

## Phase 5: Fix + Regression Test

**在修复之前写回归测试——但前提是有一个正确的测试 seam（接口边界）。**

测试接口边界判断：
- **好的测试**：通过公共接口验证真实 bug 模式，测试失败 = 行为损坏
- **坏的测试**：mock 内部依赖，测试私有方法，通过外部手段验证

> **错误的测试 = 虚假信心。** 改了一个内部函数名测试就失败，但行为没变——那这个测试在测实现，不是测行为。

---

## Phase 6: Verify + Regression-test

修复后必须验证：
- [ ] 原反馈回路现在 Pass
- [ ] 相关功能的测试套件全部通过
- [ ] 调试日志标签 `[DEBUG-xxxx]` 全部清理

> **修完后立即跑回归测试。** 等到下一轮测试再发现新 bug = 浪费半小时。

---

## Phase 2: Pattern Analysis

**Find the pattern before fixing:**

### 1. Find Working Examples

- Locate similar working code in the same codebase
- What works that's similar to what's broken?

**Action:** Use `search_files` to find comparable patterns:

```python
search_files("similar_pattern", path="src/", file_glob="*.py")
```

### 2. Compare Against References

- If implementing a pattern, read the reference implementation COMPLETELY
- Don't skim — read every line
- Understand the pattern fully before applying

### 3. Identify Differences

- What's different between working and broken?
- List every difference, however small
- Don't assume "that can't matter"

### 4. Understand Dependencies

- What other components does this need?
- What settings, config, environment?
- What assumptions does it make?

---

## Phase 3: Hypothesis and Testing

**Scientific method:**

### 1. Form a Single Hypothesis

- State clearly: "I think X is the root cause because Y"
- Write it down
- Be specific, not vague

### 2. Test Minimally

- Make the SMALLEST possible change to test the hypothesis
- One variable at a time
- Don't fix multiple things at once

### 3. Verify Before Continuing

- Did it work? → Phase 4
- Didn't work? → Form NEW hypothesis
- DON'T add more fixes on top

### 4. When You Don't Know

- Say "I don't understand X"
- Don't pretend to know
- Ask the user for help
- Research more

---

## Phase 4: Implementation

**Fix the root cause, not the symptom:**

### 1. Create Failing Test Case

- Simplest possible reproduction
- Automated test if possible
- MUST have before fixing
- Use the `test-driven-development` skill

### 2. Implement Single Fix

- Address the root cause identified
- ONE change at a time
- No "while I'm here" improvements
- No bundled refactoring

### 3. Verify Fix

```bash
# Run the specific regression test
pytest tests/test_module.py::test_regression -v

# Run full suite — no regressions
pytest tests/ -q
```

### 4. If Fix Doesn't Work — The Rule of Three

- **STOP.**
- Count: How many fixes have you tried?
- If < 3: Return to Phase 1, re-analyze with new information
- **If ≥ 3: STOP and question the architecture (step 5 below)**
- DON'T attempt Fix #4 without architectural discussion

### 5. If 3+ Fixes Failed: Question Architecture

**Pattern indicating an architectural problem:**
- Each fix reveals new shared state/coupling in a different place
- Fixes require "massive refactoring" to implement
- Each fix creates new symptoms elsewhere

**STOP and question fundamentals:**
- Is this pattern fundamentally sound?
- Are we "sticking with it through sheer inertia"?
- Should we refactor the architecture vs. continue fixing symptoms?

**Discuss with the user before attempting more fixes.**

This is NOT a failed hypothesis — this is a wrong architecture.

---

## Red Flags — STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Pattern says X but I'll adapt it differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when already tried 2+)**
- **Each fix reveals a new problem in a different place**

**ALL of these mean: STOP. Return to Phase 1.**

**If 3+ fixes failed:** Question the architecture (Phase 4 step 5).

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question the pattern, don't fix again. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence, trace data flow | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare, identify differences | Know what's different |
| **3. Hypothesis** | Form theory, test minimally, one variable at a time | Confirmed or new hypothesis |
| **4. Implementation** | Create regression test, fix root cause, verify | Bug resolved, all tests pass |

## Hermes Agent Integration

### Investigation Tools

Use these Hermes tools during Phase 1:

- **`search_files`** — Find error strings, trace function calls, locate patterns
- **`read_file`** — Read source code with line numbers for precise analysis
- **`terminal`** — Run tests, check git history, reproduce bugs
- **`web_search`/`web_extract`** — Research error messages, library docs

### With delegate_task

For complex multi-component debugging, dispatch investigation subagents:

```python
delegate_task(
    goal="Investigate why [specific test/behavior] fails",
    context="""
    Follow systematic-debugging skill:
    1. Read the error message carefully
    2. Reproduce the issue
    3. Trace the data flow to find root cause
    4. Report findings — do NOT fix yet

    Error: [paste full error]
    File: [path to failing code]
    Test command: [exact command]
    """,
    toolsets=['terminal', 'file']
)
```

### With test-driven-development

When fixing bugs:
1. Write a test that reproduces the bug (RED)
2. Debug systematically to find root cause
3. Fix the root cause (GREEN)
4. The test proves the fix and prevents regression

## Real-World Impact

From debugging sessions:
- Systematic approach: 15-30 minutes to fix
- Random fixes approach: 2-3 hours of thrashing
- First-time fix rate: 95% vs 40%
- New bugs introduced: Near zero vs common

**No shortcuts. No guessing. Systematic always wins.**
