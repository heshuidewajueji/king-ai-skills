---
name: tbox-prd-skill
description: >
  辅助生成汽车电子 TBOX（Telematics BOX）全生命周期需求文档。
  适用客户：整车厂 OEM、Tier1 供应商（特汽/华菱/瑞驰/斯堪尼亚等）。
  当 King 说「生成 PRD」「写需求规范」「TBOX 需求」时使用。
---

# TBOX PRD Skill — 车载TBOX产品需求文档生成器

> 本 Skill 辅助生成汽车电子 TBOX（Telematics BOX）全生命周期需求文档。
> 适用客户：整车厂 OEM、Tier1 供应商（特汽/华菱/瑞驰/斯堪尼亚等）

---

## 核心能力

| 文档类型 | 输出模板 | 标准对齐 |
|---------|---------|---------|
| **FRS** 功能需求规范 | `templates/frs.md` | GB/T 32960.2 |
| **SRS** 软件需求规范 | `templates/srs.md` | GB/T 34590 / AUTOSAR |
| **SDD** 软件设计说明 | `templates/sdd.md` | AUTOSAR Classic |
| **SRS** 硬件需求规范 | `templates/hrs.md` | AEC-Q100 |
| **SRS** 信息安全规范 | `templates/sec-srs.md` | TAF 安全技术要求 |

---

## 输入信息清单（生成 PRD 前必须收集）

### 必须信息
- [ ] 客户名称 / 车型
- [ ] 目标市场（国内新能源 / 出口 / 商用车）
- [ ] 必须满足的法规（GB/T 32960 / GB/T 32960.3-2025 / TAF 安全要求）
- [ ] 通信制式（4G Cat.1 / 4G Cat.4 / 5G）
- [ ] CAN 总线速率与协议（500K / 250K / CAN FD）
- [ ] 需要接入的 ECU 清单（VCU / BMS / MCU / BCM...）
- [ ] OTA 需求（是否需要支持车厂定制 OTA）

### 可选信息
- [ ] 蓝牙版本（BLE 4.2 / 5.0）
- [ ] 定位精度要求（亚米级 / 分米级）
- [ ] 休眠功耗目标（≤2mA @ standby）
- [ ] 工作电压范围（9~36V）
- [ ] 目标 ASIL 等级（B / C）
- [ ] 客户特殊需求（数字钥匙 / E-call / 定制协议）

---

## 使用方式

### 1. 生成完整 FRS（功能需求规范）

用户提供客户和车型信息后，按以下顺序执行：

```
1. 读取 knowledge/domain.md — TBOX 功能模块体系
2. 读取 knowledge/standards.md — GB/T 32960 合规要点
3. 按 templates/frs.md 模板生成文档
4. 按 CAN 矩阵补充具体信号列表（如客户提供）
5. 补充信息安全章节（如目标市场为欧洲 → UNECE R155）
```

### 2. 生成软件 SRS

```
1. 读取 knowledge/standards.md 第3节（软件需求规范要求）
2. 按 templates/srs.md 生成
3. 对齐 FRS 中的每条功能需求（需求追踪矩阵）
```

### 3. 评审现有 PRD

```
将现有 PRD 内容粘贴，Skill 自动检查：
- 是否覆盖 GB/T 32960.2 必选功能
- 是否包含 TAF 安全技术要求章节
- 需求描述是否符合 Gherkin 格式（Given-When-Then）
- 接口定义是否完整（CAN ID / UART / 以太网）
```

---

## 输出规范

| 字段 | 要求 |
|------|------|
| 文档编号 | `{客户代号}-SRS/HRS/FRS-{版本}` |
| 保密级别 | 客户内部 / 受控分发 / 公开 |
| 计量单位 | SI 单位，CAN 信号使用原始值+物理值双注 |
| 需求 ID | `FR-XXX`（功能）/ `SR-XXX`（软件）/ `HR-XXX`（硬件）|

---

## 注意事项

- GB/T 32960 是**强标**，国内新能源必须满足，文档中必须单独章节声明
- GB/T 32960.3-2025（2026年12月1日实施）较旧版有重大变化，出口车型注意版本
- TAF 安全技术要求（2024版）是进网许可前提，文档结构需对齐其章节
- AUTOSAR Classic 是主流方案（基于OSEK/OS），不要混用 Adaptive 术语
- CAN 信号命名遵循 `{ECU}_{SignalName}_{Unit}` 格式（如 `VCU_VehSpd_kmh`）

---

*Skill 版本：v1.0*
*最后更新：2026-04-19*
*维护责任：pm-director*
