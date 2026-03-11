# 交易相关记忆

## 已完成的重要回测（避免重复）

### B020 棉花双涨历史验证
- **脚本**: `/tmp/cf_double_rise_v2.py` + `/tmp/cf_double_rise.json`
- **数据**: CF.parquet 9200万行, 2019-12~2025-08
- **结论**: 989/5041夜盘=19.6%发生双涨, 是常态非异常. DTE<15均涨+64%, DTE60+仅+14%

### B021 棉花卖出宽跨DTE矩阵
- **脚本**: `/tmp/cf_strangle_sell_dte.py` + `/tmp/cf_strangle_sell_dte.json`
- **结论**: DTE≥25进→DTE≤7出=6年100%胜率(31个合约). DTE<20进场风险陡增

### B021 双涨后卖出 止盈vs持有
- **脚本**: `/tmp/cf_double_rise_exit_v2.py` + `/tmp/cf_double_rise_sell.py`
- **结论**: 真实胜率64%(非93%, 之前有session排序bug). 止盈15%累计优于持有到收盘(+5324% vs +4650%). 止盈5%触达率65%但累计反而少赚
- **BUG教训**: session排序必须日盘在前夜盘在后(同一日历日内), 否则H1会指向过去数据

### B021 纯碱SA DTE卖出验证
- **脚本**: `/tmp/sa_strangle_sell_dte.py`
- **数据**: SA.parquet 6000万行, 2023-10~2025-08, 24个合约
- **结论**: 入25-60出0-7=95-100%胜率, 与CF一致甚至更稳. SA入20出0=100%(CF仅86%). 郑商所品种通性

### B021 SA+CF深度策略参数 (止盈止损+MAE+衰减曲线)
- **脚本**: `/tmp/sa_strategy_deep.py` + `/tmp/cf_strategy_deep.py`
- **结果**: `/tmp/sa_strategy_deep.json` + `/tmp/cf_strategy_deep.json`
- **SA策略**: 入DTE25-30, 出DTE0-3, 不设止盈止损. 均PnL+88%, MAE中位27%. 衰减加速区DTE20→14(6.1%/天)
- **CF策略**: 入DTE30-60(必须更早!), 出DTE0-7, 绝对不设止损(100%触发!). 均PnL+65%, MAE中位128%最大547%
- **核心差异**: SA温和(MAE27%), CF剧烈(MAE128%). SA可入DTE20, CF必须入DTE30+

### B024 VRP全市场ATM跨式卖出策略 (4阶段回测+Walk-Forward)
- **脚本**: `/tmp/vrp_backtest.py`(Phase1), `/tmp/vrp_backtest_dte.py`(Phase2), `/tmp/vrp_tpsl_v2.py`(Phase3), `/tmp/vrp_walkforward.py`(Phase4), `/tmp/vrp_entry_filter.py`(入场筛选)
- **数据**: 48品种/35316观测/6年(2019-10~2025-08)
- **策略**: 每日VRP(IV-RV)扫描→Top-3→卖ATM跨式→次日平仓
- **入场规则**: VRP Top-3 + **入场腿比≤1.5x**(用户经验: 超过1.5x不是跨式是赌单边)
- **TP/SL方法论(用户原创)**: 止盈=权利金/DTE×系数(theta衰减), 止损=高价腿/低价腿≥阈值(腿比失衡). 拒绝传统百分比止损
- **稳健路线(DTE30-60)**: TP0.8xθ/NoSL, OOS Sharpe **12.56**, WR 89.9%, max_loss -13.65%
- **激进路线(DTE≤7, 筛选后)**: TP0.8xθ/SL4-5x, OOS Sharpe **5.93**, WR 72.7%, max_loss -85%
- **Walk-Forward**: 衰减率1.17/0.89, 零过拟合
- **关键发现**: DTE≤7入场腿比中位1.37x, 40%超1.5x→筛选后Sharpe从5.68→5.93; DTE30-60天然平衡(仅2.4%超标)

### B037 v6全市场日内宽跨DTE分桶回测
- **脚本**: `/tmp/grid_backtest_v6.py`, 结果: `/tmp/*_grid_v6.json`
- **数据**: 48品种, 1680组合/品种(5入场×8TP×7SL×6DTE桶)
- **核心发现**: DTE是关键隐藏维度. DTE15-60为黄金区间(33/48品种最优). v5仅19个可靠品种→v6全部48个可用
- **超级品种(Sharpe≥30)**: SH烧碱43.6/ZN锌43.5/RU橡胶40.3/V-PVC34.6/PX33.7/PF短纤32.5/NI镍31.2/RM菜粕30.2/MA甲醇28.2
- **已整合到工作台**: 今日计划按钮动态计算DTE→匹配最优桶参数→Sharpe降序排列

### B018/B019 棕榈油周五夜盘
- **脚本**: `/tmp/p_allday_backtest.py` + `/tmp/p_friday_dte14.py`
- **结论**: DTE≤7周五88.9%/+23% vs 非周五75.5%/+13%, 周五有显著优势
