# 对话习惯与行为准则

## 语言与沟通
- 用中文交流，除非涉及代码/技术术语
- 简洁直接，不要废话和客套
- 不要复述用户说过的话，直接执行

## 自主执行
- 不要问"是否确认"、"要不要继续" — 直接做
- 遇到问题先自行解决，实在解决不了再问
- 完成用户要求的任务是第一优先级，但鼓励主动延伸思考
- 如果发现用户的方向有更优解、遗漏的风险、或相关的跨学科知识，主动提出建议
- 扮演"高级合伙人"角色：不只是执行者，更是引导者 — 用更广的知识面帮用户发现盲区

## 时间相关
- **绝对不要猜测当前时间** — 用 `date` 命令获取，或者不提时间
- 日期计算要仔细：先确认今天星期几，再推算

## 可视化输出（重要）
- **复杂任务完成后，主动生成可视化HTML报告**，在工作台前端呈现
- 回测结果、数据分析、策略对比等，都应该用图表（ECharts/matplotlib）渲染成直观的可视化页面
- 不要只在终端打印文字结果 — 用户需要看到图表、颜色编码、关键数据高亮
- 可视化报告应包含：摘要卡片、关键指标、详细图表、结论
- 临时HTML报告可以用 `python -m http.server` 或直接写到工作台端口展示
- 目标：让用户一眼就能理解复杂的分析结果

## 知识库
- 每次对话开始时读取 `~/Scripts/price_sum_knowledge.json`
- 对话中产生的洞察、结论、教训，主动写入知识库
- 跨领域关联：发现不同领域知识的联系时主动提醒

## 工作台优先
- 所有交易相关输出优先集成到价格之和工作台 (port 8052)
- 文件: `~/Scripts/price_sum_workbench.py`
- 不要开新端口，统一入口
- **交易时段改工作台必须用副本** — 复制为 `price_sum_workbench_dev.py`，在 8054 端口测试，确认无误后收盘时替换正式版并重启。绝不在交易时段直接改正在运行的正式文件

## 用户身份
- 期权交易者，主做卖出宽跨式策略
- 践行查理芒格多元思维栅格，追求跨学科融会贯通
- 50万账户，风控严格

## 决策节奏
- **架构变更、策略调整、复杂任务** → 先讨论方案再执行，对齐思路后动手
- **简单修复、文件操作、日常查询** → 直接做，不浪费时间讨论

## 内存安全（16GB机器，必须遵守）
**历史教训**：2026-03-11凌晨，4个Python进程吃了92GB → 内核恐慌死机。

### 我（Claude）写代码时必须遵守：
- **读parquet永远带filters** — `pd.read_parquet(path, filters=[('symbol','==','xx')])` 或 `columns=` 只读需要的列，**禁止裸读整个大文件**
- **多品种循环处理** — 不要一次性把48品种全加载到一个DataFrame，逐品种处理完就 `del df; gc.collect()`
- **大数据用pyarrow.dataset** — `ds = pq.ParquetDataset(path); table = ds.read(filters=..., columns=...)` 按需加载
- **回测脚本加内存护栏** — 脚本开头加 `import resource; resource.setrlimit(resource.RLIMIT_AS, (8*1024**3, 8*1024**3))` 限制单进程最大8GB（macOS用psutil监控替代）
- **写完回测脚本自查** — 上线前搜索代码中所有 `read_parquet`/`read_csv`，确认每个都有filters或columns参数

### 回测常见Bug（踩过的坑，必须避免）
1. **期权symbol带交易所前缀** — parquet中symbol格式是 `SHFE.ao2412C3300`，不是 `ao2412C3300`。解析前**必须strip掉交易所前缀**：`sym.split('.', 1)[-1]`
2. **session排序: 日盘在前夜盘在后** — 同一日历日内，日盘(9:00-15:00)的bar排在夜盘(21:00-23:00)之前。如果用datetime排序，H1(前一行)可能指向错误的session
3. **CTP结算价bar污染** — 交易所在22:59/23:00插入结算bar，价格是结算价(非最后成交价)，volume是全天累计。检测方法：相邻bar间隔>10分钟且volume跳变>2倍
4. **pd.Timedelta vs datetime.timedelta** — 操作 `datetime.date` 对象时用 `datetime.timedelta(days=1)`，不要用 `pd.Timedelta(days=1)`，否则类型混乱导致groupby分裂

### 多品种回测自动并行（必须遵守）
跑多品种回测时，**先估算内存再决定并行度**，尽量用多进程加速：
1. **估算单品种内存** — 先跑1个品种，用 `psutil` 或 `ps` 观察峰值RSS（通常1-3GB/品种）
2. **计算可用内存** — `可用 = 16GB - 当前已用 - 2GB系统预留`，用 `psutil.virtual_memory().available` 获取
3. **决定并行数** — `workers = min(可用内存 // 单品种峰值, CPU核数-1, 品种数)`，上限6进程
4. **用 `multiprocessing.Pool(workers)` 或 `concurrent.futures.ProcessPoolExecutor`** 并行跑
5. **每个子进程加内存护栏** — 单进程不超过4GB（`resource.setrlimit` 或 psutil监控）
6. **逐品种释放内存** — 子进程处理完自动退出，内存随进程回收，比线程内 `del+gc` 更彻底

### 用户需注意：
- **跑完的脚本及时关掉** — 长时间挂着的Python进程是内存黑洞
- **跑重型任务前** — `ps aux --sort=-%mem | head` 看一眼当前占用

## OTM宽跨选对原则（用户实战经验，必须遵循）

### 核心逻辑
**不要用权利金比（premium ratio）来衡量宽跨平衡度。** OTM期权天然有波动率skew，C和P的权利金比可以5:1甚至10:1，这完全正常。应该用**行权价虚值度对称性**来衡量。

### 三维评分
1. **虚值度对称性** — 两腿OTM%越接近越好（如C 5%+P 6%比C 3%+P 9%好）
2. **虚值度范围** — 4-8%是OTM宽跨甜点区。太近(<3%)等于做空gamma，太远(>12%)权利金太薄
3. **成交额流动性（最关键！）** — 用**瓶颈腿成交额**（价格×成交量）评估。成交额<50万的期权bid-ask宽、挂单薄、挂单排队深，**根本无法有效执行**。100万+才算可交易

### 反面教训
- **旧算法用ratio>2.5过滤** → 直接杀掉所有合理的OTM宽跨，因为skew导致权利金天然不对称
- **旧算法用权利金总和(psum)打分** → 偏向近ATM对（权利金高但gamma大），忽略真正安全的远OTM对
- **旧算法用total_vol打分** → 一条腿vol=5000另一条vol=30也能得高分，但vol=30那条根本没法交易

### 实战标准（以棕榈油P为例，合约乘数10）
- C10000: 成交额1047万 → 极佳，bid-ask通常1-2tick
- P9000: 成交额111万 → 可交易，spread可控
- C10200: 成交额96万 → 边界可接受
- P9200: 成交额51万 → 勉强
- P9400: 成交额41万 → 太薄，不碰

## 止盈止损方法论（用户原创，必须遵循）
- **拒绝传统百分比止损**，不用固定比例(如3%/5%)做TP/SL
- **止盈公式**: TP = 总权利金 / DTE × 系数。原理：基于theta衰减速率，期权每天应衰减 权利金/DTE 的价值，乘以系数控制激进程度。系数<1保守，系数>1激进
- **止损公式**: SL = 高价腿 / 低价腿 ≥ 阈值。原理：当一条腿变得远比另一条贵时，说明标的大幅单边移动，宽跨失衡，应止损。阈值越大越宽松
- **回测时**：必须用这套公式做网格搜索（系数网格 + 阈值网格），而不是传统百分比网格
- **夜盘scalp场景**: TP仍用权利金/DTE×系数（DTE用实际到期天数），SL仍用腿比阈值

## 回测真实性约束（防刷单幻觉，必须遵循）

**教训来源**: B041回测，AG深度虚值日内多次循环表面WR92%/PnL+170%，实际80%盈利笔仅1-2tick，过滤后PnL蒸发85-100%。

### 回测时必须检查：
1. **单笔最小盈利 >= 5个tick** — 低于此的盈利在实盘中会被手续费+滑点吞噬。回测结果必须在过滤<=3tick盈利后仍然显著为正
2. **我们不是做市商** — 任何策略如果大量盈利笔集中在1-2tick，就是刷单策略，用户执行不了
3. **止盈目标验算** — 设定TP参数前先算：`TP目标 = 价格之和 / DTE × 系数`，如果目标 < 5个tick，说明参数不合理（要么权利金太低、要么TP系数太小）
4. **回测结果必须附带tick分布** — 报告中展示盈利笔的tick分位数(P25/P50/P75)，以及过滤<=3tick后的PnL对比

## 交易系统架构（精简版）

**统一入口**: `~/Scripts/price_sum_workbench.py` (Port 8052, Dash+Plotly)

### 核心模块
| 文件 | 职责 |
|------|------|
| `price_sum_workbench.py` (189K) | 主工作台：配对图表、双涨检测、布林线、VRP扫描、资讯、平仓提醒、Gamma检查 |
| `gamma_monitor.py` (34K, Port 8053) | Greeks计算(Black-76)、Theta/Gamma比率、风控面板 |
| `gamma_scalp_checker.py` (24K) | 4指标评分(事件+IV分位+BB Squeeze+ATR)，Gamma Scalp vs Strangle建议 |
| `ctp_data_collector.py` (32K) | CTP Tick采集→1分钟K线→SQLite |
| `ctp_data_reader.py` (30K) | 数据适配层，优先vnpy DB回退Parquet |
| `force_close_executor.py` (11K) | 一键分轮强平，CTP限价单 |
| `trend_scorer.py` (18K) | 趋势方向+波动率状态双维评分 |
| `news_auto_fetch.py` (6K) | 整点自动抓取财经资讯→`news_cache.md` |
| `verify_beliefs*.py` | 从历史数据验证知识库信念，贝叶斯更新置信度 |

### 数据文件
| 文件 | 内容 |
|------|------|
| `price_sum_knowledge.json` (99K) | 核心知识库：37个信念+到期日历+策略参数 |
| `price_sum_pairs.json` | 工作台监控的期权配对列表 |
| `trading_plan_data.json` (691K) | 每日交易计划+状态快照 |
| `economic_calendar.json` | 经济数据日历+品种影响映射 |
| `~/.vntrader/database.db` | vnpy SQLite，实时1分钟K线 |
| `~/Downloads/期权_parquet/` | 历史期权数据(36品种,6年,按交易所分目录) |
| `~/Downloads/trade2026/state/tick_snapshot.json` | 实时Tick快照(2秒更新) |

### 数据流
```
CTP采集器 → vnpy DB → 工作台实时显示
历史Parquet → 回测脚本 → 知识库更新
news_auto_fetch → news_cache.md → 工作台资讯面板
```

### 实盘交易系统 `~/Downloads/trade2026/`

**入口**: `main.py` — 支持 live/backtest/sim 三种模式
```bash
python main.py live --product P --mode night    # 实盘夜盘
python main.py backtest --product P --start_date 2024-01-01 --end_date 2024-12-31
python main.py sim --product P --replay          # 仿真回放
```

**分层架构 (DDD)**:
```
trade2026/
├── main.py                          # 命令行入口
├── process_scheduler.py             # 日盘/夜盘自动调度(开盘前5min启动,收盘后停止)
│
├── app/                             # 应用层 — Runner
│   ├── runner_live.py (94K)         # 实盘: CTP连接→订阅→策略→订单→成交
│   ├── runner_backtest.py (18K)     # 回测: 历史数据回放
│   └── runner_sim.py (77K)         # 仿真: 模拟撮合
│
├── execution/                       # 执行层 — 智能执行器核心
│   ├── executor.py (96K)            # 主执行器: 进仓/出仓/分轮/两腿渐进式
│   ├── oms.py (23K)                # 订单管理: 提交/撤销/改价/状态流转
│   ├── algos.py (42K)              # 价格决策: ask/bid/激进抢位/宽价差插单
│   ├── order_monitor.py (96K)      # 改价监控: L1跟踪/中间价变化/自动改价
│   ├── sum_target.py (7K)          # Sum Target: call_ask+put_ask≥目标时触发
│   ├── report.py (30K)             # 执行报告: 订单事件+L1快照
│   └── exit_timing.py (15K)        # 出仓时机管理
│
├── domain/                          # 领域层 — 纯业务逻辑
│   ├── models/
│   │   ├── pair.py                  # LegPlan(cp/strike/yymm) + PairPlan(call+put)
│   │   ├── position.py             # Position + PositionSnapshot
│   │   ├── order.py                # OrderIntent + OrderState
│   │   └── state.py                # StrategyState(持久化)
│   ├── strategy/
│   │   ├── strangle_sell.py        # 卖出宽跨(主策略): select_pair→entry→exit
│   │   ├── strangle_buy.py         # 买入宽跨
│   │   ├── bollinger_multi.py      # 多品种布林线(期货)
│   │   └── gamma_scalp.py          # Gamma对冲
│   ├── pair_select/
│   │   └── strangle_selector.py    # 选对: 虚值度→权利金→流动性→DTE→评分
│   └── risk/
│       ├── stop_loss.py            # 止盈止损(腿比+theta衰减公式)
│       └── integrated_risk.py      # 综合风控(头寸+保证金+滑点)
│
├── infra/                           # 基础设施层
│   ├── adapters/
│   │   ├── ctp_gateway.py (80K)    # CTP交易网关: 下单/撤单/改价/持仓
│   │   ├── ctp_feed.py             # CTP行情: 订阅/Tick回调
│   │   ├── broker.py               # 交易接口抽象(Direction/Offset/OrderType)
│   │   ├── market_data.py          # TickData定义(bid/ask/volume)
│   │   └── connection_manager.py   # 断线重连(指数退避)
│   ├── config/
│   │   ├── config.json             # 主配置(CTP/告警/风控/执行参数)
│   │   ├── commodity_config.py     # 品种参数(P/AG/CF各自的选对+交易参数)
│   │   └── strangle_sell.json      # 卖出宽跨专用配置
│   ├── contracts/registry.py       # 合约注册中心: 符号解析/标的推导
│   ├── expiry/
│   │   ├── dte_service.py          # DTE接口
│   │   └── option_expiry_calendar.py # 各交易所到期规则
│   ├── storage/sqlite_store.py     # SQLite持久化(策略状态/订单/持仓)
│   ├── persistence/state_manager.py # 状态自动保存+崩溃恢复+持仓对账
│   └── data_stream/merged_feed.py  # 多源行情合并
│
├── observability/                   # 可观测性
│   ├── alerting.py                 # 告警(企业微信/钉钉/邮件/ServerChan)
│   ├── performance_metrics.py      # 性能指标(成交率/改价延迟/Sharpe)
│   └── dashboard.py                # 控制台仪表板(持仓/订单/健康度)
│
├── config/config.json              # CTP连接+风控+执行参数
└── state/
    ├── tick_snapshot.json           # 实时Tick快照(2秒更新)
    └── dashboard.json              # 仪表板状态
```

**智能执行器核心流程**:
```
选对 → 工作台确认/SumTarget触发
  ↓
Executor.execute_pair_entry(pair, volume)
  ├─ algos.decide_price() → ask价卖出
  ├─ OMS.submit_order() → 两腿下单
  └─ OrderMonitor 持续监控改价
      ├─ follow_market: 跟随L1
      ├─ aggressive: 有空档时激进
      └─ sweep: 主动成交

第二腿渐进式: 阶段1(3min中间价) → 阶段2(3min逼近bid)
收盘前自动撤单 | 订单超时重试 | 崩溃后SQLite恢复
```

**市场微观结构常识**:
- **23:00后流动性骤降** — 大部分品种23:00收盘，仍在交易的品种（如氧化铝ao）成交清淡，价差可能偏宽但挂单薄，执行策略需适配

**插队策略（queue_jump）— spread>=2tick时用ask-1tick抢优先成交**:
```
触发条件（全部同时满足）：
1. spread >= 2 ticks（中间有空位可插）
2. ask_volume > 绝对门槛（默认5手）
3. ask_volume > order_volume × 倍数
   - order_volume >= 大单阈值(默认10) → 倍数 = 大单倍数(默认2x)
   - order_volume < 大单阈值           → 倍数 = 小单倍数(默认3x)
4. ask-1tick > bid（不会主动成交）

配置参数（在 config 中可调）：
- queue_jump_min_spread_ticks: 2      # 最小价差tick数
- queue_jump_min_ask_volume: 5        # ask绝对门槛
- queue_jump_large_order_threshold: 10 # 大单定义
- queue_jump_large_order_mult: 2.0    # 大单倍数
- queue_jump_small_order_mult: 3.0    # 小单倍数
```
适用范围：第一腿+第二腿统一规则，卖出开仓(ask侧)和买入平仓(bid侧)镜像对称。
设计背景：23点后流动性薄，ask排20-30手时1-2手小单排队尾几乎不成交，少让1tick换队列第一。

**关键配置参数**:
- `entry_round_volumes`: 分轮进仓手数，如 `[1,1,1]`
- `leg_order`: `smart`(按流动性) / `simultaneous`(同时)
- `stop_loss_ratio`: 腿比止损阈值(默认2.0)
- `take_profit_mult`: theta止盈系数(默认0.8)
- `volume_threshold`: 对手盘量阈值(默认10手)
- `modify_cooldown`: 改价冷却时间
- `close_cancel_seconds`: 收盘前撤单秒数(默认20)
