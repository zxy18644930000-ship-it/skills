# User Preferences

- **Language**: User communicates in Chinese (Mandarin), respond in Chinese
- **Autonomy**: User prefers full autonomy — don't ask for confirmation, just execute
- **对话记忆（事件驱动写入）**: 不等对话结束，有结论就立刻写。触发时机：①话题切换时存上一个话题的结论 ②任务完成时（回测/bug修复/功能上线）③产生关键决策或推翻旧认知时 ④用户说"记住这个"。MEMORY.md 当索引（控制100行内），详细内容按主题分文件

# 记忆文件索引

| 文件 | 内容 |
|------|------|
| `trading.md` | 已完成回测(B018-B037)、策略结论、实盘教训 |
| `data_structure.md` | 历史数据路径和格式（期权/期货/CTP） |
| `construction.md` | 建筑劳务功效管理系统（规划阶段） |
| `课程知识库.md` | 期权量化课程（28课时，0/28进度） |
| `health.md` | 健康相关：体检结论、用药、健康决策 |
| `conversations.md` | 对话摘要流水账（日期+一句话） |

# Skills Repo 同步

当用户推送代码时，同时同步技能和知识库到 GitHub skills 仓库：
- **仓库**: https://github.com/zxy18644930000-ship-it/skills
- **本地**: `~/skills/`
- **源文件**: 技能 `~/.claude/commands/*.md` + 知识库 `~/Scripts/price_sum_knowledge.json` + `~/Scripts/price_sum_pairs.json`

# 统一工作台架构原则

**价格之和工作台 (port 8052) 是所有交易辅助工具的唯一入口。**
- **文件**: `~/Scripts/price_sum_workbench.py`
- **已集成**: 期权对图表、智能选对、布林上轨预警、卖出顾问

# 知识库 — 查理·芒格式多元思维栅格

- **路径**: `~/Scripts/price_sum_knowledge.json`（v2.0, 37个信念）
- **配对**: `~/Scripts/price_sum_pairs.json`
- 任何领域的洞察都要沉淀，鼓励跨学科关联

# 期权DTE估算

**不要用简化公式！** 各交易所规则不同，详见知识库 `expiration_calendar`。

604合约(从2026-02-28算):
- 郑商所: 3月11日 | 大商所: 3月17日 | 上期所: 3月25日 | 能源中心: 3月13日 | 广期所: 3月6日
