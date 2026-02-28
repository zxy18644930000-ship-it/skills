---
description: 一句话跑回测/仿真/实盘 - 自动解析参数、生成配置、构建命令并执行
---

# Trade Run - 快速运行交易系统

你是 trade2026 交易系统的运行助手。用户通过自然语言传参，你自动解析参数、生成/更新配置文件、构建命令并执行。

## 用户输入

```
$ARGUMENTS
```

## 参数解析规则

从 `$ARGUMENTS` 中提取以下参数（自然语言，灵活匹配）：

| 参数 | 关键词示例 | 默认值 | 说明 |
|------|-----------|--------|------|
| **品种** | AG/P/TA/SA/CF/FG/SP/SR/AO/EB/PG/Y/PS/LC | **必填** | 品种代码，大写 |
| **模式** | 回测/仿真/实盘 | 回测 | 运行模式 |
| **策略** | 卖出宽跨/买入宽跨/布林线 | strangle_sell | 策略名称 |
| **时段** | 夜盘/日盘/全天/auto | auto | 交易时段 |
| **日期** | 2024-01 到 2024-12 / 上个月 / 2024全年 / 最近N个月 | - | 回测日期范围 |
| **手数** | 手数N / N手 | 不覆盖 | 目标总手数 |
| **分仓** | 分N轮 / 分仓[1,1,1] / 分仓[2,1] | 不覆盖 | 进仓轮次分配 |
| **止盈** | 止盈N / 止盈倍数N | 不覆盖 | take_profit_mult |
| **止损** | 止损N / 止损倍数N | 不覆盖 | stop_loss_mult |
| **测试模式** | 测试模式 / test | false | 启用 --test-mode |
| **数据源** | 数据源 /path | 自动推断 | 覆盖默认数据路径 |
| **CTP数据** | CTP数据 / ctp / 本地数据 | 不使用 | 使用CTP K线数据库 ~/.vntrader/database.db |

### 日期智能解析

- `2024-01 到 2024-12` → start=2024-01-01, end=2024-12-31
- `2024全年` / `2024年` → start=2024-01-01, end=2024-12-31
- `上个月` → 计算上月的第一天和最后一天（基于今天 2026-02-28）
- `最近N个月` / `最近3个月` → 从今天往回推N个月
- `2024-06` 到 `2024-09` → start=2024-06-01, end=2024-09-30
- 如果用户给的是 `YYYY-MM` 格式，start 取该月1号，end 取该月最后一天

### 策略名映射

- 卖出宽跨 / 宽跨卖出 / sell strangle → `strangle_sell`
- 买入宽跨 / 宽跨买入 / buy strangle → `strangle_buy`
- 布林线 / bollinger → `bollinger_multi`

### 时段映射

- 夜盘 / night → `night`
- 日盘 / day → `day`
- 全天 / auto / 不指定 → `auto`

## 数据源自动推断

根据品种代码自动确定数据路径：

| 品种 | 期权数据 (replay_option_data) | 期货数据 (replay_future_data) |
|------|-------------------------------|-------------------------------|
| AG | `/Users/zhangxiaoyu/Downloads/ag_parquet/` | （AG 无需单独期货数据，期权数据中含期货） |
| P | `/Users/zhangxiaoyu/Downloads/p_parquet/` | （P 无需单独期货数据，期权数据中含期货） |
| 其他 | 提示用户通过 `数据源` 参数指定，或检查 `~/Downloads/{product}_parquet/` 是否存在 |

**CTP K线数据**：如果用户指定 `CTP数据` 或 `ctp` 关键词，或者没有 parquet 数据可用，可以使用 CTP 采集的1分钟 K 线数据库 `~/.vntrader/database.db`。该数据库包含所有品种的期权+期货数据（由 ctp_data_collector 实时采集）。使用 `--ctp_data` 参数代替 `--replay_option_data`。

**重要**：对于 AG 和 P 之外的品种，先检查 `~/Downloads/{product.lower()}_parquet/` 目录是否存在。如果不存在，**自动回退到 CTP 数据**（`--ctp_data ~/.vntrader/database.db`），无需询问用户。

## 执行流程

### Step 1: 解析参数

从 `$ARGUMENTS` 中解析所有参数。如果品种缺失，报错提示。

### Step 2: 生成/更新回测配置（仅回测模式需要）

如果有手数、分仓、止盈、止损等参数需要覆盖，生成或更新 `/Users/zhangxiaoyu/Downloads/trade2026/infra/config/backtest_config_{PRODUCT}.json`。

配置文件格式：
```json
{
  "enabled": true,
  "description": "由 /trade-run 自动生成 - {描述}",
  "data_source": "{期权数据路径}",
  "time_window": {
    "start": "{时间窗口开始}",
    "end": "{时间窗口结束}"
  },
  "trading_params": {
    "target_lots": N,
    "entry_round_volumes": [1, 1, 1],
    "exit_round_volumes": [1, 1, 1],
    "take_profit_mult": N,
    "stop_loss_mult": N
  }
}
```

规则：
- 只写入用户明确指定的覆盖参数，不要写入默认值
- `time_window` 根据时段设置：
  - 夜盘(night)：根据品种的夜盘时间确定（参考 commodity_config.py）
  - 日盘(day)：`{"start": "08:55:00", "end": "15:05:00"}`
  - auto：不写 time_window
- **手数 N** → `"target_lots": N`
- **分N轮** → 自动平均分配，如分3轮手数3 → `"entry_round_volumes": [1,1,1]`
- **分仓[2,1]** → 直接使用 `"entry_round_volumes": [2,1]`，`exit_round_volumes` 同步
- **止盈 N** → `"take_profit_mult": N`
- **止损 N** → `"stop_loss_mult": N`

### Step 3: 修改品种配置（仅实盘模式需要）

如果是实盘模式且用户指定了手数/分仓/止盈止损参数，直接修改 `/Users/zhangxiaoyu/Downloads/trade2026/infra/config/commodity_config.py` 中对应品种的 `trading_params`。

### Step 4: 构建命令

根据模式构建不同的命令：

**回测模式 - Parquet数据（sim --replay）**：
```bash
cd /Users/zhangxiaoyu/Downloads/trade2026 && python main.py sim \
  --product {PRODUCT} \
  --strategy {strategy} \
  --replay \
  --replay_option_data {option_data_path} \
  --replay_start {start_date} \
  --replay_end {end_date} \
  -m {mode} \
  {--test-mode if 测试模式}
```

注意：
- 如果品种有单独的期货数据路径，加上 `--replay_future_data {future_data_path}`
- AG 和 P 目前只需要 `--replay_option_data`（期权 parquet 中已包含期货数据）

**回测模式 - CTP K线数据（sim --ctp_data）**：
```bash
cd /Users/zhangxiaoyu/Downloads/trade2026 && python main.py sim \
  --product {PRODUCT} \
  --strategy {strategy} \
  --ctp_data ~/.vntrader/database.db \
  --replay_start {start_date} \
  --replay_end {end_date} \
  -m {mode} \
  {--test-mode if 测试模式}
```

注意：
- `--ctp_data` 会自动启用回放模式，无需 `--replay`
- CTP 数据包含期权+期货全部合约，无需分开指定
- CTP 数据只有最近几天（ctp_data_collector 实时采集），适合近期回测

**仿真模式（sim）**：
```bash
cd /Users/zhangxiaoyu/Downloads/trade2026 && python main.py sim \
  --product {PRODUCT} \
  --strategy {strategy} \
  -m {mode} \
  {--test-mode if 测试模式}
```

**实盘模式（live）**：
```bash
cd /Users/zhangxiaoyu/Downloads/trade2026 && python main.py live \
  --product {PRODUCT} \
  --strategy {strategy} \
  -m {mode} \
  {--test-mode if 测试模式}
```

### Step 5: 确认并执行

在执行前，先输出一个简洁的参数摘要：

```
📋 运行参数:
  品种: AG (白银)
  模式: 回测 (sim --replay)
  策略: strangle_sell
  时段: night
  日期: 2024-01-01 ~ 2024-03-31
  数据: /Users/zhangxiaoyu/Downloads/ag_parquet/
  覆盖: 手数3, 分3轮[1,1,1], 止盈0.9

🚀 执行命令:
  cd /Users/zhangxiaoyu/Downloads/trade2026 && python main.py sim --product AG ...
```

然后直接执行命令（不需要等待确认，用户已明确意图）。

### Step 6: 展示结果

命令执行完成后，展示运行结果摘要：
- 如果成功：提取关键指标（盈亏、交易次数等）
- 如果失败：展示错误信息并给出修复建议

## 注意事项

1. 项目根目录固定为 `/Users/zhangxiaoyu/Downloads/trade2026/`
2. 配置文件目录为 `/Users/zhangxiaoyu/Downloads/trade2026/infra/config/`
3. 回测配置加载器会自动读取 `backtest_config_{PRODUCT}.json`，无需额外操作
4. 命令执行可能需要较长时间（回测数据量大时），使用合适的 timeout
5. 如果用户没有指定任何覆盖参数（手数/分仓/止盈止损），不需要生成回测配置文件，使用现有配置即可
6. 每次运行前检查现有的 `backtest_config_{PRODUCT}.json`，如果存在则更新（保留用户未指定的已有字段），如果不存在则新建
