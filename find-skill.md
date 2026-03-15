---
description: 搜索并安装开源技能生态中的技能包。当用户问"有没有XX技能"、"找一个XX的skill"、"怎么做XX"时触发。
---

# Find Skills — 技能搜索与安装

从开源技能生态 (skills.sh) 搜索并安装技能包，扩展 Agent 能力。

## 使用方式

用户输入: `/find-skill [搜索关键词]`

## 执行流程

### 1. 解析用户需求

从用户输入中提取：
- **领域**: React、测试、部署、文档、设计等
- **具体任务**: 写测试、代码审查、生成changelog等
- **关键词**: 用于搜索的英文关键词

### 2. 搜索技能

用 Bash 执行搜索命令：

```bash
npx skills find [关键词]
```

常见搜索示例：
- "React性能优化" → `npx skills find react performance`
- "PR代码审查" → `npx skills find pr review`
- "生成changelog" → `npx skills find changelog`

如果第一次没找到，尝试换关键词再搜一次。

### 3. 展示结果

将搜索结果整理为：
```
找到以下技能：

1. **[技能名]** — 功能描述
   安装: `npx skills add <owner/repo@skill> -g -y`
   详情: https://skills.sh/<owner/repo/skill>

2. ...
```

### 4. 安装

用户确认后，用 Bash 执行安装：

```bash
npx skills add <owner/repo@skill> -g -y
```

`-g` 全局安装（用户级），`-y` 跳过确认。

### 5. 未找到时

如果没有匹配的技能：
1. 告知用户未找到
2. 提议用当前能力直接帮助完成任务
3. 建议用户可以自己创建: `npx skills init my-skill`

## 常见技能分类

| 分类 | 搜索关键词 |
|------|-----------|
| Web开发 | react, nextjs, typescript, css, tailwind |
| 测试 | testing, jest, playwright, e2e |
| DevOps | deploy, docker, kubernetes, ci-cd |
| 文档 | docs, readme, changelog, api-docs |
| 代码质量 | review, lint, refactor, best-practices |
| 设计 | ui, ux, design-system, accessibility |
| 效率 | workflow, automation, git |

## 搜索技巧

1. 用具体关键词："react testing" 比 "testing" 效果好
2. 换同义词：找不到 "deploy" 试试 "deployment" 或 "ci-cd"
3. 热门来源：`vercel-labs/agent-skills`、`ComposioHQ/awesome-claude-skills`

## 技能生态

- **浏览**: https://skills.sh/
- **更新检查**: `npx skills check`
- **批量更新**: `npx skills update`
