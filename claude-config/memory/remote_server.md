# 远程回测服务器 (Windows WSL2)

## 连接信息
- **SSH 快捷方式**: `ssh win`
- **Tailscale IP**: 100.80.19.31
- **Mac Tailscale IP**: 100.93.235.28
- **WSL2 用户**: jesussos
- **SSH 配置**: ~/.ssh/config 已配置 Host win

## 硬件配置
- **CPU**: 32核
- **内存**: 128G 物理（WSL2 当前分配 61G，可调大）
- **GPU**: RTX 5080
- **磁盘**: 954G 可用
- **系统**: Ubuntu 24.04 on WSL2

## 软件环境
- Python 3.12.3
- pandas 3.0.1, pyarrow 23.0.1, numpy 2.4.3

## 使用方式
```bash
# 连接
ssh win

# 远程执行命令
ssh win "python3 /home/jesussos/backtest/script.py"

# 同步脚本到远程
scp script.py win:/home/jesussos/backtest/

# 同步数据到远程
rsync -avz --progress ~/Downloads/期权_parquet/ win:/home/jesussos/data/期权_parquet/

# 拉取结果
scp win:/home/jesussos/backtest/result.json /tmp/
```

## 注意事项
- SSH 和 Tailscale 已配置 systemctl enable，WSL2 启动后自动运行，无需手动操作
- 数据路径映射: Mac `~/Downloads/期权_parquet/` → 远程 `/home/jesussos/data/期权_parquet/`
- Windows 文件可通过 `/mnt/c/` 访问（百度网盘下载的文件在这里）
- Tailscale 需要两台机器都在线才能连通

## 远程数据已有（不需要从Mac同步！）
- 期权1分钟K线: `/mnt/d/backtest_data/Options_parquet/` — 6交易所58品种，与Mac一致
- 期货1分钟K线: `/mnt/d/backtest_data/Futures_parquet/` — 6交易所92品种
- Tick数据(zip): `/mnt/d/BaiduNetdiskDownload/` — ag/p/ao/SA/CF/FG/SH/i/m 等
- SA tick(parquet): `/mnt/d/backtest_data/sa-tick-parquet/` — 2023-10起

## 路径映射
| Mac | 远程 |
|-----|------|
| `~/Downloads/期权_parquet/` | `/mnt/d/backtest_data/Options_parquet/` |
| `~/Downloads/期货数据_parquet/` | `/mnt/d/backtest_data/Futures_parquet/` |

## 搭建历史 (2026-03-12)
- WSL2 Ubuntu 24.04 安装
- Tailscale 组网（绕过了 OpenSSH 密钥配置问题，直接用 Tailscale SSH）
- Python 环境配置
- 发现D盘已有完整回测数据，无需从Mac同步
