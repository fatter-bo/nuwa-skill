# 03-live-betting: Live Betting链上实现分析

## 市场规模
- Live投注占总投注量62.35%（2025）
- 重大赛事期间70%+来自现场（DraftKings UEFA冠军联赛数据）
- 行业标准延迟：<500ms
- 传统博彩市场$1190亿（2025），Live占~$740亿

## 链上技术挑战

### 延迟
- 以太坊L1：12秒出块
- Optimism/Arbitrum：~2秒
- Azuro（Gnosis）：5秒
- WINR L3：1秒
- 中心化系统：<100ms
- **差距**：链上最快1秒 vs 中心化<100ms——10倍差距

### MEV
- 抢跑者在mempool看到投注交易→三明治攻击
- 以太坊2025年$12亿+ MEV提取
- 三明治攻击占MEV量51%
- 体育博彩MEV特殊性：知道比分变化前抢先投注

### 吞吐量
- 一场足球赛90分钟内可能有数百次赔率更新
- 同时数十场比赛进行中
- 每次赔率更新需链上反映

### 数据一致性
- 进球/得分事件发生在毫秒级
- 所有节点需同时知道当前比赛状态
- 不同数据源可能有微秒级差异

## 各协议Live Betting方案

### Azuro V3
- 链下签名（EIP-712）→ Relayer执行
- HostCore集中管理比赛状态
- WebSocket API近即时推送
- 快照保存法处理延迟投注

### Overtime V2
- Merkle树架构：赔率作为Merkle根推送上链
- OpticOdds流式传输实时赔率
- Merkle树设计防止toxic flow利用

### SX Bet
- 自有L1（SX Network）
- 完全链上CLOB
- <0.5% vig
- P2P串关（首创）

### Purebet
- 跨协议聚合
- 自定义撮合引擎，数千TPS
- 链下处理→Solana链上结算

## FastEvent WebSocket在Live Betting中的价值

### 为什么<500ms很重要
1. **实时赔率重定价**：进球后赔率需立即更新
2. **微型投注**：下一个角球、下一个犯规——秒级事件
3. **即时市场暂停**：红牌/受伤时暂停投注
4. **动态风险管理**：LP池需实时调整暴露

### 推送频率建议
| 数据 | 频率 | 延迟要求 |
|------|------|---------|
| 比分事件（进球） | 事件驱动 | <5秒 |
| 状态变更 | 事件驱动 | <10秒 |
| 时钟数据 | 每10秒 | <20秒 |
| 统计数据 | 每30秒 | <60秒 |

### 与Azuro Relayer的集成点
- FastEvent WebSocket → Azuro HostCore → LiveCore赔率更新
- 数据格式需匹配Azuro的Condition/Outcome结构
- matchId使用keccak256确保确定性
