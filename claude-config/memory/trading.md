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

### B041 多品种日内宽跨 v1→v2 DTE分桶回测
- **脚本**: `~/Scripts/multi_trade_allmarket.py`(v1), `~/Scripts/multi_trade_allmarket_v2.py`(v2)
- **结果**: `~/Scripts/multi_trade_results_v2/` (39品种JSON + _ALL_SUMMARY)
- **报告**: `~/Scripts/multi_trade_v2_report.html`
- **v2新增**: DTE分桶 (1,7]/(7,15]/(15,30]/(30,60] 作为网格搜索维度
- **核心发现**: DTE 30-60 确认为黄金区间(36/39品种最优), 平均PnL提升+97% vs v1

### B042 腿背离均值回归策略 全市场回测 (重大发现!)
- **脚本**: `~/Scripts/divergence_backtest.py` + `~/Scripts/divergence_report.py`
- **结果**: `~/Scripts/divergence_results/` (52品种JSON + _ALL_SUMMARY)
- **报告**: `~/Scripts/divergence_report.html`
- **信号**: 高价腿方向 ≠ Sum方向 + IV(邻档OTM)不动 → 真背离
- **策略A(均值回归/卖Sum)**: 51/52品种胜出(98%), 多品种216组参数100%盈利
- **策略B(动量/买Sum)**: 1/52胜出(ad, 仅3天数据, 不具代表性)
- **S级(PnL>3000+WR>65%)**: lc(碳酸锂,28350), ni(镍,6708)
- **A级(PnL>1000+WR>60%)**: cu/ru/OI/p/ag/br/CF/zn/si/eb/ps/y/pg
- **通用最优参数**: LB10, Comp≥1.0, Hold60m, TP0.5-1.0x, SL2.0x
- **IV代理**: 用邻档OTM行权价(非ATM!), 处理非均匀行权价间距(cu等)
- **结论**: 背离是均值回归信号, 实盘应卖出Sum; 动量策略在绝大多数品种亏损

## 执行器架构决策

### 2026-03-12 废弃渐进式第二腿（关键决策）
- **背景**: 第二腿有独立的渐进式三阶段（phase1=中间价3min→phase2=贴近bid3min→phase3=sweep对手价）
- **问题**: spread=1时加速切换（5+5=10秒就进phase3 sweep），CF605C17000排了11秒就被主动扫到ask=150
- **决策**: 完全废弃渐进式。第二腿与第一腿走完全相同的逻辑（跟随L1 + queue_jump + 单量检测sweep）
- **commit**: `8f13224` — 删除 `_second_leg_phase` / `second_leg_phase1_seconds` / `second_leg_phase2_seconds` 所有引用

### 2026-03-12 修复分轮进仓两腿不平衡
- **症状**: 进6手总有一腿差3手（如C=9,P=6）
- **根因**: `_is_entry_complete()` 用绝对CTP持仓检查，起始持仓=3+目标=3→立即pass，Round2在Round1第二腿未成交就开始
- **修复**: 记录进仓开始时的起始持仓(`_entry_start_call_volume/put_volume`)，用增量判断完成度
- **commit**: 同 `8f13224`

## 实盘观察与策略洞察

### B042v2 夜盘23:00+ 循环ATM跨式卖出 (AG+AO)
- **脚本**: `/tmp/late_night_loop_v2.py`, 结果: `/tmp/late_night_loop_v2.json`, 报告: `/tmp/late_night_loop_report.html`
- **数据**: AG 632夜(2019-2025), AO 234夜(2024-2025)
- **策略**: 23:15起入场ATM跨式卖出→TP/SL/FC出场→冷却5min→再入场循环
- **AG最优**: +15m/TP0.8θ/SL5.0x, Sharpe=7.0, WR=79%, PnL=+1.42%, 3轮/晚
- **AG过滤≤3tick后**: WR=75%, PnL=+1.26% — **衰减仅11%, 真实可靠**
- **AG DTE30-60**: Sharpe=7.4, WR=80%, PnL=+1.50% — 黄金区间
- **AO最优**: +15m/TP0.8θ/SL3.0x, Sharpe=6.3, WR=70%, PnL=+0.98%
- **AO过滤≤3tick后**: WR=54%, PnL=+0.33% — **刷单问题严重, P50仅2tick**
- **结论**: AG可用于实盘循环做(DTE30-60, 23:15入场, 1手); AO刷单风险高, 需谨慎
- **关键发现**: SL越宽越好(5.0x>3.0x>2.0x), 夜盘波动有限误杀少; TP0.8θ最优平衡点

### B043 MA纠缠度 vs 趋势持续性 — 宽跨卖出安全过滤器 (已完成)
- **脚本**: `~/Scripts/B043_ma_tangle.py`(Phase1), `~/Scripts/B043_ma_tangle_direct.py`(Phase2)
- **OTM报告**: `~/Scripts/ma_tangle_otm_results/multidim_report.html`
- **ATM报告**: `~/Scripts/ma_tangle_direct_results_v2/multidim_report.html`
- **Phase 1 (间接验证 — 期货波幅)**: 47品种全市场完成
  - **实用参数**: lookback=10, threshold=7 (70%) — 47/47品种成立
  - trending状态未来30bar波动 / entangled状态波动 = 1.2~5.6x
- **Phase 2a (ATM跨式)**: 41品种, 222,425笔 → 23/41(56%)成立
- **Phase 2b (OTM宽跨4-8%虚值, 最终版)**: 44品种, 228,830笔
  - **30m 信念成立: 28/44 (64%)**, 60m: 28/44 (64%) — OTM比ATM更有效
  - **Switch效应: 15个品种** — 纠缠入场赚钱, 趋势入场亏钱(过滤器反转盈亏方向)
  - **双时间框架一致: 26个品种** — 30m+60m同时成立
  - **多维综合评分(满分100)**: 尾部保护30+PnL优势25+Switch20+双TF15+胜率10
  - **S级(≥60分,14个)**: Y豆油80.7/OI菜油79.6/EB苯乙烯78.7/AG白银77.0/PB铅76.9/SA纯碱76.4/LC碳酸锂76.3/M豆粕74.3/UR尿素73.2/PG液化气70.0/SN锡68.7/CF棉花62.5/SF硅铁61.6/ZN锌60.3
  - **A级(45-59,7个)**: RM菜粕/SH烧碱/MA甲醇/NI镍/TA-PTA/I铁矿/L塑料
  - **AG特殊**: 尾部保护59.3x(趋势最差-267% vs 纠缠-28%), Switch效应(纠缠+0.17% vs 趋势-0.09%)
  - **反向品种(别用过滤器)**: JD鸡蛋(-2.59%)/SM锰硅(-1.88%)/CJ红枣(-1.50%)/SI工业硅(-0.98%)/AO氧化铝(-0.84%)/PK花生(-0.65%)
- **核心结论**: OTM宽跨下过滤器效果远强于ATM(64% vs 56%, Switch从1→15个). S级品种必须启用过滤器. 核心价值=尾部风险保护+Switch效应, 非均值PnL提升

### B018/B019 棕榈油周五夜盘
- **脚本**: `/tmp/p_allday_backtest.py` + `/tmp/p_friday_dte14.py`
- **结论**: DTE≤7周五88.9%/+23% vs 非周五75.5%/+13%, 周五有显著优势
