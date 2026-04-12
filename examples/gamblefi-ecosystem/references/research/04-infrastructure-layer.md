# 04-infrastructure-layer: GambleFi基础设施层分析

## 基础设施栈概览

GambleFi技术栈可分为五层：

```
┌─────────────────────────────────────────────┐
│  前端应用层：Stake, Rollbit, BC.Game, Bluff  │
├─────────────────────────────────────────────┤
│  流动性层：Azuro (LP池), WINR (银行卷)       │
├─────────────────────────────────────────────┤
│  赔率/市场层：Azuro (vAMM), 传统oddsmakers   │
├─────────────────────────────────────────────┤
│  数据/预言机层：FastEvent ← 定位在这里        │
│  Chainlink, Pyth, SportsDataIO              │
├─────────────────────────────────────────────┤
│  随机数层：VRF (Chainlink), Bitcoin哈希       │
├─────────────────────────────────────────────┤
│  结算层：L1/L2链 (Polygon, Base, Solana)     │
└─────────────────────────────────────────────┘
```

## Azuro Protocol — 流动性 + 赔率 + 结算

### 定位
去中心化预测市场/体育博彩基础设施，提供：流动性池 + 赔率引擎 + 链上结算

### 核心架构

#### LiquidityTree（流动性树）
- 基于Segment Tree的新型数据结构
- 允许多个投注条件从统一的单例流动性池获取资金
- 公平分配LP的利润和损失

#### vAMM（虚拟自动做市商）
- 赔率根据池内可用资金和虚拟增强值（virtual reinforcements）动态计算
- 投注者通过下注移动卖方赔率
- 最终链上赔率反映Azuro对事件概率的看法

#### 参与者角色
| 角色 | 功能 |
|------|------|
| **Liquidity Provider** | 提供USDC到池中，赚取庄家P&L |
| **Data Provider** | 创建投注条件、设置赔率、更新概率、结算结果 |
| **前端运营商** | 基于Azuro构建面向用户的投注应用 |
| **投注者** | 通过前端应用下注 |

### 关键数据
| 指标 | 数值 |
|------|------|
| 总投注量 | $300M+（历史累计） |
| 支持链 | Solana, Polygon, Base |
| 已建应用 | 42个 |
| LP正收益率 | >95%（持仓>1月） |
| AZUR FDV | ~$20M |
| AZUR总供应 | 1B |

### 合作伙伴
- deBridge（跨链互操作）
- 多条EVM兼容链

---

## WINR Protocol — 开发者SDK

### 定位
100%链上、无Gas的赌场基础设施套件，为开发者提供构建去中心化博彩应用的工具

### 核心架构

#### Layer 3网络
- 专为去中心化博彩打造的L3网络
- 无Gas交易
- 每个游戏直接运行在智能合约上

#### 模块化基础设施
```
传统：垂直整合（每个平台自建一切）
WINR：模块化共享基础设施
  → 银行卷层（Bankroll）：共享流动性池
  → RNG层：链上可验证随机数
  → 激励框架：开发者从用户活动中获利
```

#### WINR SDK
- 开发者可快速构建链上游戏
- 无需自行解决流动性问题
- 新游戏可提交WINR DAO审核后上线
- 提供智能合约工具 + 流动性引擎 + 激励框架

### 关键优势
- 每笔投注可链上验证（trustless, proof-of-design）
- 开发者专注游戏设计，流动性/RNG/结算由协议处理
- 无Gas设计降低用户门槛

---

## Gnosis Conditional Token Framework (CTF)

### 定位
开源的预测结果代币化框架，被Polymarket等主要预测市场采用

### 核心机制
```
$1 USDC → 锁入CTF合约 → 铸造 1 YES token + 1 NO token
  → 事件结束 → 正确结果token可赎回$1
  → 错误结果token归零
```

### 关键应用
| 项目 | 使用方式 |
|------|---------|
| Polymarket | 核心结算框架，$2B+选举相关交易量 |
| Presagio/Omen 2.0 | AI驱动的预测市场 |
| OLAS Predict | 361+日活AI代理，8.2M+交易 |

### 与GambleFi的关系
- CTF适合二元结果预测市场（是/否）
- 体育博彩可建模为条件代币（球队A赢/球队B赢/平局）
- Polymarket模式验证了CTF在大规模交易中的可靠性

---

## FastEvent的定位：纯数据层

### FastEvent解决的问题
```
链上体育博彩智能合约
  ↓ 需要
"谁赢了？比分是多少？是否进球？"
  ↓ 但是
区块链无法获取链下数据
  ↓ 需要
预言机 / 数据提供者
  ↓
FastEvent = 快速、准确的体育事件数据预言机
```

### 与其他基础设施的分工

| 基础设施 | 提供什么 | FastEvent重叠？ |
|---------|---------|---------------|
| Azuro | 流动性 + 赔率 + 结算 | **否** — FastEvent可为Azuro提供数据 |
| WINR | 开发者SDK + 银行卷 + RNG | **否** — FastEvent补充数据层 |
| Gnosis CTF | 结果代币化框架 | **否** — CTF需要数据触发结算 |
| Chainlink | 通用预言机网络 | **部分** — Chainlink覆盖价格,FastEvent专精体育事件 |
| Pyth | 高频价格数据 | **否** — Pyth专注金融价格 |
| SportsDataIO | 传统体育数据API | **直接竞争** — 但FastEvent更快/更链上原生 |

### FastEvent的差异化优势
1. **纯数据层**：不涉及流动性、随机数、前端——专注做好一件事
2. **不直接受博彩监管**：数据提供者 ≠ 博彩运营商
3. **B2B模式**：服务Azuro、WINR等协议及其上42+个dApp
4. **体育专精**：vs Chainlink的通用性
5. **速度**：体育博彩需要秒级结算数据（尤其live betting），FastEvent满足<20秒结算需求

### 体育博彩对数据的具体需求

| 数据类型 | 延迟要求 | 用途 |
|---------|---------|------|
| 赛前赔率 | 分钟级 | 开盘/调整赔率 |
| 实时比分 | 秒级（<5s） | Live betting赔率调整 |
| 事件结果 | <20秒 | 自动结算 |
| 球员统计 | 分钟级 | 球员道具盘 |
| 伤病/阵容 | 小时级 | 赔率调整 |

Sources:
- [Azuro Protocol Introduction](https://gem.azuro.org/knowledge-hub/introduction/what-is-azuro)
- [Azuro LiquidityTree](https://gem.azuro.org/knowledge-hub/how-azuro-works/liquidity-tree)
- [Azuro Liquidity Providers](https://gem.azuro.org/knowledge-hub/how-azuro-works/protocol-actors/liquidity-providers)
- [WINR Protocol Docs](https://docs.winr.games/gambling-infrastructure-is-moving-onchain)
- [WINR Infrastructure Analysis](https://cryptonary.com/research/winr-infrastructural-leader-risky-bet-gamblefi/)
- [Gnosis CTF Overview](https://hackernoon.com/gnosis-conditional-token-framework-ctf-tokenizing-potential-outcomes-in-prediction-markets)
- [Chainlink Sports Markets](https://blog.chain.link/bringing-sports-markets-to-blockchains-using-chainlink/)
- [Oracle Role in Betting](https://www.webopedia.com/crypto/learn/blockchain-oracles-decentralized-betting-markets/)
- [SportsDataIO × Chainlink](https://sportsdata.io/blockchains-via-chainlink-network-partnership)
