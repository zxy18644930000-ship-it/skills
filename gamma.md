Gamma 风控参考 - 计算期权对的 Greeks、Theta/Gamma 比率、盈亏平衡波动，辅助选品种和开仓决策。

脚本: `/Users/zhangxiaoyu/Scripts/gamma_monitor.py`
数据源: `~/.vntrader/database.db`
配对源: `~/Scripts/price_sum_pairs.json`

## 参数解析

用户输入的参数 `$ARGUMENTS`，格式：
- 无参数 / `扫描`：扫描工作台所有配对
- 品种代码如 `ag2604`：扫描该品种所有可交易期权对
- 两腿如 `ag2604C37600+ag2604P17000`：计算指定配对
- `对比`：所有配对按安全性排序，输出对比表

## 执行步骤

全程自动化，不要询问确认：

### 1. 运行 gamma_monitor.py

根据参数选择模式：

**模式A：扫描工作台（无参数 / `扫描`）**

```bash
cd /Users/zhangxiaoyu/Scripts && python3 -c "
from gamma_monitor import scan_all_pairs, print_pair_greeks
results = scan_all_pairs()
for pg in results:
    print_pair_greeks(pg)
"
```

**模式B：扫描品种（如 `ag2604`）**

```bash
cd /Users/zhangxiaoyu/Scripts && python3 -c "
from gamma_monitor import scan_product_pairs, print_pair_greeks
results = scan_product_pairs('ag2604')
for pg in sorted(results, key=lambda x: x.theta_gamma_ratio, reverse=True):
    print_pair_greeks(pg)
"
```

**模式C：指定配对（如 `ag2604C37600+ag2604P17000`）**

解析两腿合约代码，调用：
```bash
cd /Users/zhangxiaoyu/Scripts && python3 -c "
from gamma_monitor import calculate_pair_greeks, print_pair_greeks
pg = calculate_pair_greeks('ag2604C37600', 'ag2604P17000')
if pg: print_pair_greeks(pg)
else: print('数据不足，无法计算')
"
```

**模式D：对比排序（`对比`）**

扫描所有配对，按 Theta/Gamma 比率从高到低排序，输出简洁对比表。

### 2. 解读输出

计算完成后，用中文向用户解读关键指标：

- **Theta/Gamma 比率**：越高越安全。>50安全，20-50注意，<20危险
- **盈亏平衡波动**：期货波动多少%才会让一天的 Gamma 亏损吃掉 Theta 收入
- **GEX（Gamma Exposure）**：期货涨1%时因 Delta 变化产生的额外盈亏（元/手）
- **净Delta**：组合的方向性偏向。正=偏多，负=偏空
- **信号**：SAFE / CAUTION / DANGER

### 3. 给出交易建议

根据计算结果，结合以下逻辑给出建议：

**卖出宽跨适合的条件：**
- T/G 比率 > 20（至少 CAUTION 级别）
- 盈亏平衡波动 > 3%（期货不太可能单日波动这么多）
- DTE 合适（不要太短也不要太长，7-30天最佳）

**买入宽跨适合的条件：**
- T/G 比率 < 10（Gamma 大，波动时收益加速）
- 有明确的波动率催化剂（数据发布、地缘事件）
- DTE 较短（Gamma 放大效应更强）

**不建议交易的情况：**
- IV 极低（Vega 不利）
- 流动性差（bid-ask spread 大）
- 盈亏平衡波动 < 1%（太危险）

### 4. 教学提示

如果用户是在学习过程中使用本技能（结合 `/学习` 课程），适当解释 Greeks 的含义和交易逻辑。
