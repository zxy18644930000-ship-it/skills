Hermes 深度成长 - 知识沉淀技能

当用户提到以下意图时使用此技能：
「沉淀」「记到 Hermes」「存入知识库」「存入长期记忆」「值得记一下」
「这个以后还会用到」「下次还要用」「形成规则」「沉淀这个经验」
「不要重复踩坑」「记住这个偏好」「建立工作流」

## 核心判断标准

在沉淀任何内容前，先问自己六个问题：

1. **这是稳定事实吗？** — 不是一次性的试错结论
2. **这是长期偏好吗？** — 用户多次表达的一致偏好
3. **这是可复用流程吗？** — 以后还会重复执行的工作流
4. **这会不会泄露秘密？** — 无 CTP 账号、token、密码、raw log
5. **这是只在当前机器或会话成立的吗？** — 不是环境绑定的临时状态
6. **Hermes 已有更合适的落点吗？** — 优先补充到现有章节，不新建重复文档

六个问题有一个不稳，就不强行沉淀，宁可保留在队列 pending。

## 落点优先级

1. **稳定事实/偏好** → `docs/Hermes_深度成长沉淀.md` 直接补充现有章节
2. **可复用工作流** → 先检查是否有重叠 skill，优先 patch，不重复建
3. **不确定是否值得** → 写入 `state/hermes_growth_queue.jsonl`，由定时任务二次判断
4. **纯秘密/噪音** → 直接丢弃，不进任何库

## 执行方式

### 方式一：直接写入沉淀文件（适合明确的稳定知识）

读取 `docs/Hermes_深度成长沉淀.md`，找到最合适的章节位置，用 Edit 工具补充。
补充格式：先给结论，再给适用边界，最后给复用建议。

### 方式二：写入成长队列（适合不确定是否值得长期保留）

```bash
python3 /Users/zhangxiaoyu/trade2026/scripts/codex_hermes_sync.py \
  --mode mirror \
  --title "简短标题" \
  --body "要沉淀的内容" \
  --repo "/Users/zhangxiaoyu/trade2026"
```

### 方式三：直接记忆写入（适合用户偏好/已验证事实）

```bash
python3 /Users/zhangxiaoyu/trade2026/scripts/codex_hermes_sync.py \
  --mode memory \
  --title "简短标题" \
  --body "要记忆的内容"
```

## 禁止事项

- 严禁写入 secrets、token、cookie、CTP_* 环境变量、raw log
- 严禁写入只在当前机器/会话成立的环境绑定细节
- 严禁写入临时试错、短期故障快照
- 严禁为了"清空队列"强行沉淀不稳定内容
- 严禁建立与现有 Hermes 重复的知识树

## 判断示例

**值得沉淀**：
- "8060 默认资金口径应为 ctp_available，不应用 100 万"
- "弱腿 24h 成交额小于 10 万禁止进仓"
- "量化策略合集心跳每 1 小时巡检一次，检测到 1 小时无增长时尝试恢复 worker"

**不值得沉淀**：
- "今天在 8060 上修了一个按钮样式 bug"
- "TA607C6900 这个合约临门复核失败了"
- "remote-a 机器的 IP 是 100.102.242.67"（环境绑定细节）
- "raw log: Error: connection refused at line 123"（噪音）

## 协作约定

Claude Code 和 Codex 共用同一套 Hermes 资产：
- 长期知识：`docs/Hermes_深度成长沉淀.md`
- 成长队列：`state/hermes_growth_queue.jsonl`
- 审计记录：`state/hermes_growth_audit.jsonl`

两边都先读沉淀文件，避免重复。Claude Code 新发现的稳定结论，优先补充到现有章节，不要另建"Claude 专属"知识树。
