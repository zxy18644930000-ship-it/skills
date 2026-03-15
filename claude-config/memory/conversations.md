# 对话摘要日志

## 2026-03-12
- 讨论记忆机制优化：确定分主题文件存储 + 事件驱动写入（不等对话结束）
- 重构 MEMORY.md 为索引，详细内容搬入主题文件（trading/health/construction/data_structure/remote_server）
- 搭建远程回测服务器：Windows WSL2 (32核/128G) + Tailscale SSH 组网 → `ssh win`
- 远程数据整理：Tick zip→parquet 转换（10品种453文件30G），清理旧版本 v1~v11
- 创建 `/远程主机` 技能，包含完整数据地图和使用流程
- 确定回测分级原则：本地小验证 → 远程大规模跑
