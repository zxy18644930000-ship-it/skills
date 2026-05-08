Hermes - 爱马仕历史对话记忆

当用户提到以下意图时使用此技能：
「查看 Hermes」「读取爱马仕」「历史对话」「之前聊过什么」
「爱马仕记了什么」「看看之前说过什么」「我们之前讨论过」

## 读取顺序

按以下顺序查找，先找到先使用：

1. `docs/Hermes_深度成长沉淀.md` — 长期稳定知识摘要
2. `state/hermes_growth_queue.jsonl` — 待处理队列（有结论但未沉淀的 pending 项）
3. `state/hermes_growth_audit.jsonl` — 审计记录（已沉淀项的操作历史）
4. `state/hermes_route_audit.jsonl` — 路由记录
5. `state/hermes_sync_audit.jsonl` — 同步历史
6. `state/hermes_sync_backlog.jsonl` — 同步积压

## 读取方式

### 沉淀文件（结构化，可直接 grep）
```bash
cat /Users/zhangxiaoyu/docs/Hermes_深度成长沉淀.md
```

### 队列/审计文件（JSONL，逐条读取）
```bash
# 看最近10条审计记录（最新在前）
tail -n 10 /Users/zhangxiaoyu/trade2026/state/hermes_growth_audit.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    try:
        d = json.loads(line.strip())
        print(f\"{d.get('updated_at','?')} | {d.get('title','?')[:50]} | {d.get('action','?')}\")
    except: pass
"

# 看待处理队列
tail -n 20 /Users/zhangxiaoyu/trade2026/state/hermes_growth_queue.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    try:
        d = json.loads(line.strip())
        print(f\"[{d.get('status','?')}] {d.get('title','?')[:60]}\")
    except: pass
"
```

## 常用查找

```bash
# 搜索沉淀文件中的特定主题
grep -i "8060\|弱腿\|临门\|持仓" /Users/zhangxiaoyu/docs/Hermes_深度成长沉淀.md

# 搜索关键词在哪条队列记录里
grep -i "关键词" /Users/zhangxiaoyu/trade2026/state/hermes_growth_queue.jsonl

# 查看今天有没有新的沉淀操作
tail -n 50 /Users/zhangxiaoyu/trade2026/state/hermes_growth_audit.jsonl | python3 -c "
import sys, json
from datetime import date
today = date.today().isoformat()
for line in sys.stdin:
    try:
        d = json.loads(line.strip())
        at = d.get('updated_at','')
        if today in at:
            print(f\"{at} | {d.get('action','?')} | {d.get('title','?')[:40]}\")
    except: pass
"
```

## 输出格式

读取后，用简洁的方式汇报：
- 找到几条相关记录
- 关键结论是什么
- 来源是哪个文件

如无任何记录，说明 Hermes 里没有相关内容，直接告知用户。