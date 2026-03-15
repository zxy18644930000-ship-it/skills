Tavily Search - 使用 Tavily API 进行 LLM 优化的网络搜索，返回高质量结果和内容摘要。

## 环境变量要求

- `TAVILY_API_KEY` (必填) — 格式：`tvly-xxx`，从 https://tavily.com 获取

## 参数解析

用户输入的参数 `$ARGUMENTS`，格式：
- 搜索关键词，如 `python async patterns`
- 可附加选项：`-n 10`（结果数）、`--deep`（深度搜索）、`--topic news`（新闻）等

## 执行步骤

全程自动化，不要询问确认。

### 0. 环境检查

```bash
echo "TAVILY_API_KEY=${TAVILY_API_KEY:+已配置}"
```

如果未配置，提示用户前往 https://tavily.com 获取 API Key 并 `export TAVILY_API_KEY="tvly-xxx"` 到 `~/.zshrc`。

### 1. 执行搜索

脚本路径：`~/.claude/scripts/tavily-search.mjs`

```bash
node ~/.claude/scripts/tavily-search.mjs "搜索关键词" [选项]
```

### 选项说明

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `-n <数量>` | 结果数量 (1-20) | 10 |
| `--depth <模式>` | 搜索深度：ultra-fast / fast / basic / advanced | basic |
| `--topic <主题>` | 主题：general / news | general |
| `--time-range <范围>` | 时间范围：day / week / month / year | 无 |
| `--include-domains <域名>` | 只搜这些域名（逗号分隔） | 无 |
| `--exclude-domains <域名>` | 排除这些域名（逗号分隔） | 无 |
| `--raw-content` | 包含完整页面内容 | false |
| `--json` | 输出原始JSON | false |

### 搜索深度对比

| 深度 | 延迟 | 相关性 | 适用场景 |
|------|------|--------|----------|
| ultra-fast | 最低 | 较低 | 实时聊天、快速查询 |
| fast | 低 | 良好 | 需要速度的场景 |
| basic | 中等 | 高 | 通用搜索（推荐） |
| advanced | 较高 | 最高 | 深度研究、精确查找 |

### 使用示例

```bash
# 基础搜索
node ~/.claude/scripts/tavily-search.mjs "python async patterns"

# 更多结果
node ~/.claude/scripts/tavily-search.mjs "React hooks tutorial" -n 10

# 深度搜索
node ~/.claude/scripts/tavily-search.mjs "machine learning" --depth advanced

# 新闻搜索
node ~/.claude/scripts/tavily-search.mjs "AI news" --topic news

# 限定域名
node ~/.claude/scripts/tavily-search.mjs "Python docs" --include-domains docs.python.org

# 最近一周
node ~/.claude/scripts/tavily-search.mjs "期权交易策略" --time-range week
```

### 2. 解读结果

搜索完成后，用中文向用户总结关键发现，包括：
- 最相关的几条结果及其核心内容
- 如果有 AI 生成的答案，优先展示
- 根据用户的搜索意图，给出进一步建议

### Tips

- 查询保持在400字符以内，像搜索引擎查询那样简洁
- 复杂问题拆分为多个子查询效果更好
- 用 `--include-domains` 限定可信来源
- 用 `--time-range` 获取最新信息
