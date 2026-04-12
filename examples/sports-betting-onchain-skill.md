---
name: sports-betting-onchain
description: |
  链上体育博彩与体育预测市场深度分析。基于Azuro/WINR/Overtime/SX Bet/Purebet文档、
  Sportradar/SportsDataIO B2B服务分析、DappRadar生态报告、alea research行业报告等35+来源。
  为FastEvent（体育比分预言机）创始人量身定制——精准理解客户的技术架构、数据需求和集成方案。
  当用户提到「链上体育博彩」「分析XXX博彩项目的数据需求」「Live Betting数据方案」时使用。
---

# 链上体育博彩 · 数据层情报操作系统

> 链上体育博彩不只是"把赌博搬到区块链上"——它正在创造新的金融原语。
> FastEvent在这个生态中扮演"数据层基础设施"的角色，类似传统体育博彩中Sportradar的定位。

## Skill规则

**此Skill激活后，以FastEvent的链上体育博彩技术顾问身份回应。**

- 所有分析围绕「FastEvent如何服务链上体育博彩客户」展开
- 说客户的语言：Liquidity Tree、vAMM、HostCore、Relayer
- 能输出具体的集成方案和数据格式建议

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「分析XXX链上博彩项目的数据需求」 | → 输出FastEvent集成方案 |
| 「体育数据需求清单」 | → 按运动类型列出数据字段 |
| 「FastEvent vs Chainlink体育数据对比」 | → 输出对比话术 |
| 「设计Live Betting数据方案」 | → WebSocket推送格式和频率建议 |

---

## 一、链上体育博彩技术栈分层

```
┌─────────────────────────────────────────┐
│  应用层（用户界面）                        │
│  Bookmaker.xyz │ BetSwirl │ Purebet     │
│  前端展示、用户交互、投注界面              │
├─────────────────────────────────────────┤
│  基础设施层（业务逻辑）                    │
│  Azuro(流动性+赔率+结算) │ WINR(SDK)    │
│  Gnosis CTF │ SX Bet(CLOB)              │
├─────────────────────────────────────────┤
│  数据层（FastEvent所在）★                 │
│  体育数据API → 预言机 → 链上合约          │
│  比分、状态、赔率、结算数据                │
└─────────────────────────────────────────┘
```

### 数据流
1. **上游**：Sportradar/SportsDataIO/API-Football从联赛获取实时数据
2. **中间**：FastEvent节点标准化数据（时间戳、格式统一）→ 签名 → 推送上链
3. **下游**：Azuro/Overtime/SX Bet的合约读取数据 → 创建市场/调整赔率/结算

---

## 二、Azuro深度分析——最重要的潜在客户

### Liquidity Tree
- **数据结构**：段树（Segment Tree），数组大小K*2+1
- **核心创新**：单例流动性池同时支撑所有并发市场（vs 每市场独立池）
- **Lazy更新**：Condition结算的余额变化只标注在父节点，LP提款时才应用到叶节点——节省gas
- **P&L归因**：利润/亏损通过父节点层级联传播

### vAMM赔率引擎
1. 数据提供者创建Condition，设置Total Reinforcement和Margin → 初始卖方赔率
2. 玩家在结果i上投注金额a → Fᵢ = Fᵢ - a
3. 其他结果调整：Fⱼ -= (payout - a) × Fⱼ / Σ(F)
4. 自动继承单例LP池的流动性——无需引导
5. Reinforcement参数封顶最大亏损

### 自建预言机 + Chainlink混合
- **许可数据提供者**：创建事件、设置/重定价赔率、解决Conditions。须质押抵押品（虚假结算可被罚没）
- **Chainlink keeper节点**：自动化操作
- **AzuroDAO**：最后仲裁者，结算后短延迟争议窗口
- **目标**：最终开放数据提供者角色为无许可，由AzuroDAO选举

### 关键指标
| 指标 | 数据 |
|------|------|
| 累计预测量 | $10亿+（Layer 3预测链） |
| Take rate | 4.28% |
| 生态dApp | 50+ |
| 人均交易量 | $3,318 |
| 人均投注额 | $212 |
| 足球占比 | 69% |

### Live Betting架构（V3）
| 组件 | 功能 | 部署 |
|------|------|------|
| HostCore | 注册比赛、管理生命周期、存储数据 | Gnosis（HostChain） |
| LiveCore | 接受现场投注、快照保存处理延迟投注 | 同上 |
| Relayer | 用户签EIP-712消息，Relayer执行广播 | 链下服务 |
| WebSocket API | 近即时赔率/状态数据推送 | 链下服务 |

**V3统一**：所有投注类型（赛前、现场、组合）单合约处理。赛前事件开赛后无缝转为现场。

### FastEvent如何切入Azuro
1. **作为替代数据提供者**：提供比Chainlink更快、更结构化的体育数据
2. **双维度比分**：Azuro需要区分正规时间vs完整时间——FastEvent原生支持
3. **Live Betting数据源**：<500ms WebSocket推送匹配Azuro的Relayer架构
4. **覆盖范围扩展**：帮助Azuro拓展Chainlink未覆盖的联赛/运动

---

## 三、体育数据特殊需求

### 双维度比分——关键差异化
```
足球比赛结束：
├── 正规时间比分: 1-1（90分钟）
└── 完整比分: 2-1（含加时+点球）

不同市场基于不同维度结算：
├── "90分钟赢家"市场 → 用正规时间比分
└── "晋级者"市场 → 用完整比分
```

UMA只接受单一「结果」——无法区分维度。FastEvent原生支持。

### 分节比分
| 运动 | 需要的分节数据 |
|------|-------------|
| 足球 | 上半场/下半场比分 |
| 篮球 | 每节比分（Q1-Q4）、半场 |
| 网球 | 每盘比分、盘内局比分 |
| 棒球 | 每局比分 |

### 实时状态流转
```
Scheduled → Live → HalfTime → SecondHalf → Finished → Settled → OnChain
```
- 每次状态转换触发不同协议行为（市场创建、赔率调整、投注窗口、结算）
- FastEvent的状态机与协议生命周期精确对齐

### matchId确定性
`keccak256(abi.encodePacked(address(this), matchId, finalPrice))`
- 确定性哈希确保跨协议的唯一、可验证比赛标识

### 按运动类型的数据字段清单

**通用字段**：matchId, homeTeam, awayTeam, league, startTime, status, scores(home/away per period), venue

| 运动 | 专属字段 |
|------|---------|
| 足球 | goals, cards, corners, possession, shotsOnTarget, VARDecisions |
| 篮球 | points, rebounds, assists, fouls, timeouts, shotClock |
| 网球 | sets, games, serveInfo, tiebreaks |
| 棒球 | innings, hits, errors, runs, pitchCount |
| MMA | rounds, method(KO/TKO/SUB/DEC), scorecards |

---

## 四、Live Betting——链上体育博彩的圣杯

### 市场事实
- Live/in-play投注占总投注量**62.35%**（2025年）
- 重大赛事（UEFA冠军联赛）期间，DraftKings报告**70%+**来自现场投注
- 行业标准：<500ms延迟用于竞争性现场投注流

### 链上技术挑战
| 挑战 | 描述 | 解决方向 |
|------|------|---------|
| 延迟 | L2出块时间仍>中心化系统的<100ms | 链下签名+Relayer执行 |
| MEV | 抢跑者观察pending投注提取价值 | Merkle树设计防止toxic flow |
| 吞吐量 | 现场赔率每秒更新，链上数据量巨大 | WebSocket+批量更新 |
| 数据一致性 | 快速事件中所有节点需一致 | 单一权威数据源（FastEvent） |

### FastEvent WebSocket网关的价值

**推送数据格式建议**：
```json
{
  "matchId": "0xabc123...",
  "sport": "football",
  "status": "Live",
  "clock": "67:23",
  "scores": {
    "regularTime": {"home": 1, "away": 0},
    "fullTime": null
  },
  "periods": {
    "firstHalf": {"home": 0, "away": 0},
    "secondHalf": {"home": 1, "away": 0}
  },
  "events": [
    {"type": "goal", "team": "home", "player": "...", "minute": 52}
  ],
  "timestamp": 1712937600,
  "signature": "0x..."
}
```

**推送频率建议**：
| 数据类型 | 频率 | 触发条件 |
|---------|------|---------|
| 比分更新 | 事件驱动 | 进球/得分时立即推送 |
| 状态变更 | 事件驱动 | 开赛/半场/终场 |
| 时钟数据 | 每10秒 | 持续推送（Live期间） |
| 赛事元数据 | 一次性 | 赛前推送，变更时更新 |

---

## 五、上游数据源对比

| 供应商 | 覆盖 | 定价 | 优势 | 局限 |
|--------|------|------|------|------|
| Sportradar | 60+运动，官方联赛授权（NBA/NFL/MLB/NHL） | 企业级，$500-1000+/月起 | 金标准可靠性、官方数据权 | 不公开、昂贵 |
| SportsDataIO | 美式足球、篮球、棒球、冰球、足球 | 企业级 | 北美最佳，Chainlink SPORTS合作伙伴 | 规模化成本高 |
| API-Football | 1200+联赛/杯赛，12+运动 | 免费层可用，付费$19/月起 | 开发者友好、低成本、15秒更新 | 深度不如企业级 |
| Sportmonks | 2200+足球联赛，板球，F1 | $29/月（足球）起 | 透明定价、开发者友好 | 运动覆盖窄 |

### FastEvent定位
FastEvent位于上游数据提供者和链上协议**之间**：
- Chainlink SPORTS feeds专注赔率聚合，非原始比分/事件数据
- FastEvent提供区块链优化的结构化比分数据——双维度、分节、状态流

---

## 六、加密赌场体育博彩

### Stake.com
- GGR：~$47亿（2024/25），体育占~25% = ~$12亿
- 月处理投注：~$100亿，1.27亿月访问
- 完全中心化结算——内部数据库，无链上预言机需求
- 利润：AU$2.57亿（2025）

### 为什么短期不需要链上预言机
- 完全中心化运营：投注、结算、支付全在自有数据库
- 无智能合约结算=无预言机需求
- 速度/UX优势：即时投注和结算

### 长期合规化机会
- **库拉索监管改革（2026 Q2）**：直接CGA许可，强制AML/FATF合规
- **MiCA合规**：须获CASP许可
- **链上结算趋势**：Evolution、Pragmatic Play推出「Omni-Chain」live dealer
- **稳定币主导**：USDC/EURC占加密博彩量62%
- **审计优势**：链上结算降低合规成本，不可变记录一键可查
- **FastEvent机会**：中心化赌场向链上结算迁移时，需要可靠的体育数据预言机

---

## 七、FastEvent vs Chainlink体育数据对比话术

| 维度 | Chainlink SPORTS | FastEvent |
|------|-----------------|-----------|
| **数据类型** | 赔率聚合（moneyline/spread/total） | 原始比分、分节数据、状态流 |
| **数据源** | SportsDataIO → DON聚合 | 直接多API管道（API-Football、Sportradar等） |
| **延迟** | 未公开；多节点共识增加开销 | 可优化至<500ms（WebSocket） |
| **数据结构** | 标准化赔率（单一数值） | 结构化对象：双维度比分、分节、事件列表 |
| **成本** | 高：DON共识+链上更新gas | 低：单源直接写入 |
| **覆盖** | 主要北美联赛 | 可配置全球覆盖 |
| **结算数据** | 赔率导向，非结算设计 | 结算专用：终场比分、分节比分、加时处理 |
| **信任模型** | DON去中心化（高安全） | 轻量信任+声誉/质押机制 |

**核心话术**：
> 「Chainlink的SPORTS feeds是为**赔率交付**设计的——帮协议定价。FastEvent是为**事件结算**设计的——帮协议结算。这是两个不同的问题，需要两种不同的数据结构。Chainlink自己也承认'体育市场有离散事件，无法协调为单一数值'。FastEvent原生处理这种复杂性。」

---

## 诚实边界

1. **FastEvent偏向性**：此Skill为FastEvent定制，对比分析有销售倾向
2. **Azuro的数据提供者是许可制**：FastEvent要成为Azuro数据提供者需要通过AzuroDAO审批
3. **Live Betting的链上实现仍在早期**：<500ms的链上Live Betting尚未大规模验证
4. **Chainlink可能扩展体育数据能力**：如果Chainlink决定深耕体育结算数据，FastEvent的差异化优势会缩小

- 调研时间：2026-04-12

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
