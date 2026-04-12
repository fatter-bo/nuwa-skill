# 01-tech-stack: 链上体育博彩技术栈

## 三层架构

### 数据层（FastEvent所在）
- 上游：Sportradar、SportsDataIO、API-Football、Sportmonks
- 中间：预言机网络（Chainlink、Azuro自建、FastEvent）
- 下游：链上合约读取数据

### 基础设施层
- **Azuro**：Polygon/Gnosis/Chiliz，LiquidityTree+vAMM+BettingEngines(PrematchCore/LiveCore/BetExpress)，50+前端
- **WINR**：Arbitrum Orbit L3，1秒终局，GMX启发的WLP vault，Bankroll-as-a-Service
- **Gnosis CTF**：ERC-1155条件代币，Polymarket核心
- **SX Network**：自有L1，完全链上CLOB

### 应用层
| 项目 | 架构 | 特点 |
|------|------|------|
| Bookmaker.xyz | Azuro前端，Polygon | 首个Azuro前端，无KYC |
| BetSwirl | Azuro前端/affiliate | 透明非托管 |
| Overtime Markets | Merkle树，Optimism/Arbitrum/Base | Chainlink预言机，$OVER代币 |
| SX Bet | P2P交易所，自有L1 | <0.5% vig，$6.75亿已投注 |
| Purebet | 聚合器，Solana | 8协议×6链，零费率 |

## Azuro V3 Live Betting详解

### HostCore（Gnosis链上）
- 注册新比赛、管理生命周期
- 存储所有现场比赛和市场数据
- 处理赔率提供、向链下接口提供比赛信息

### LiveCore
- 动态层，接受投注（结果/赔率/金额参数）
- 快照保存法：结算后提交的投注被时间戳记录，自动取消/退款
- 使用LP合约处理投注金额和支付准备金

### Relayer
- 执行物流处理
- 用户不直接提交交易——签EIP-712消息
- Relayer执行广播，收取用户设定的奖励

### V3统一
- 单合约处理所有投注类型（赛前+现场+组合）
- 赛前事件开赛后无缝转为现场事件
- 组合投注可混合赛前和现场
