---
name: skill-manage
description: >
  Skills 自动分类与同步管理。当 King 说「整理 Skills」或「同步 Skills」时使用。
  功能：扫描新增 Skills、自动分类、GitHub 同步、NAS 备份。
---

# Skill-Manage

> Skills 自动分类与同步管理

## 触发词

当 King 说「整理 Skills」或「同步 Skills」时使用。

## 功能

### 1. 扫描新增 Skills
扫描 `~/.openclaw/king-ai-skills/` 目录，找出最近新增或修改的 Skills。

### 2. 自动分类
按以下规则自动归类：

```yaml
规则:
  obsidian_related:
    关键词: [obsidian, vault, markdown, note, canvas, bases]
    分类: "# 核心技能/"

  professional:
    关键词: [prd, tbox, pm, project, career]
    分类: "# 专业技能/"

  general:
    关键词: [tutor, simplify, review, security, defuddle]
    分类: "# 通用技能/"

  manage:
    关键词: [manage, sync, skill-manage]
    分类: "# 管理技能/"
```

### 3. GitHub 同步
```bash
cd ~/.openclaw/king-ai-skills
git add -A
git commit -m "Skills 更新: $(date '+%Y-%m-%d %H:%M')"
git push
```

### 4. NAS 备份
```bash
rsync -avz ~/.openclaw/king-ai-skills/ /Volumes/NAS/ai-skills-backup/
```

## 执行流程

1. 扫描 Skills 目录
2. 显示新增/修改列表
3. 自动分类（如有需要）
4. GitHub 提交推送
5. NAS 备份（如可用）
6. 汇总报告

## 注意事项

- 推送前先检查 `gh auth status`
- NAS 离线时跳过备份，不阻塞主流程
- 分类规则存储在 `~/.openclaw/king-ai-resources/scripts/classifier.yaml`
