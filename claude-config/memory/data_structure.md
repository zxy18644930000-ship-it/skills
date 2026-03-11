# 历史数据结构

## 通用期权 `~/Downloads/期权_parquet/{交易所}/{品种}.parquet`
- **格式**: 1分钟K线, 列=`datetime, open, high, low, close, volume, open_oi, close_oi, symbol`
- **symbol示例**: `CZCE.CF011C13000` = 郑商所棉花2021年1月Call行权价13000
- **覆盖**: CZCE(20品种), DCE, SHFE, INE, GFEX, CFFEX
- **CF**: 9200万行, 2019-12~2025-08, 2166个合约, 1383交易日

## 通用期货 `~/Downloads/期货数据_parquet/{交易所}/{品种}.parquet`
- **格式**: 同上(1分钟K线)
- **注意**: CZCE缺少CF/SA/TA/FG/SR等主力品种！只有AP/MA/OI/RM等
- **可用替代**: 从期权数据用put-call parity推算期货价格

## 棕榈油 `~/Downloads/p_parquet/{YYYY-MM}/`
- 期货: `p2409.parquet` — 列=`datetime, open, high, low, close, volume, money, open_interest`
- 期权: `p2409-C-8600.parquet` — tick数据, 列=`TradingDay, InstrumentID, UpdateTime, UpdateMillisec, LastPrice, Volume, Amount, OpenInterest, BidPrice1, BidVolume1, AskPrice1, AskVolume1`
- 新格式(2025-09起): `DCE.p2510-C-10000.parquet`

## 白银 `~/Downloads/ag_parquet/` — 同棕榈油结构

## CTP实时数据 `~/.vntrader/database.db`
- 表: `dbbardata`, 列=`symbol, datetime, close_price` (1分钟K线)
- 覆盖: 所有在交易的品种(期货+期权), 实时采集
