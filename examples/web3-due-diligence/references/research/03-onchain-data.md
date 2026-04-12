# 链上数据真实性 -- Web3尽调方法论

> 调研日期：2026-04-12
> 本文建立链上数据验证的系统性方法论，覆盖TVL真实性、用户数甄别、协议收入可持续性、工具使用及历史案例。

---

## 目录

1. [TVL真实性判断](#1-tvl真实性判断)
2. [真实用户 vs 刷交互地址（Sybil检测）](#2-真实用户-vs-刷交互地址sybil检测)
3. [协议收入来源与可持续性分析](#3-协议收入来源与可持续性分析)
4. [链上数据分析工具](#4-链上数据分析工具)
5. [核心指标体系](#5-核心指标体系)
6. [历史案例：虚假链上数据](#6-历史案例虚假链上数据)
7. [实操检查清单](#7-实操检查清单)

---

## 1. TVL真实性判断

### 1.1 DeFiLlama的TVL计算方法论

DeFiLlama是行业最广泛使用的TVL追踪平台，其核心方法论如下：

- **TVL定义**：锁定在协议智能合约中的代币总价值，借出的代币不计入TVL（仅计实际锁定的抵押品）
- **代币定价**：几乎所有代币通过CoinGecko API定价；无法获取价格的代币通过链上方法（如Uniswap V2池权重）推算
- **排除规则**：
  - 不计未流通/未发行代币（如团队锁仓的vesting合约）
  - 同一协议内不重复计算（存入获receipt token再存入只计一次）
  - 不计原生代币质押（如ATOM质押保护Cosmos Hub）
  - 跨链桥TVL单独计算，不归入任何链的TVL
  - 不计智能合约钱包（如Gnosis Safe）中的资金

**来源**: [DefiLlama Methodology](https://docs.llama.fi/)

### 1.2 TVL虚高的常见手法

#### a) 积分/空投激励驱动的TVL

**典型模式**：项目通过Points System吸引资金存入，承诺未来空投代币作为回报。

**案例**：
- **Manta Pacific（2023-2024）**：TVL在2023年12月暴涨1,250%，吸引$3.45亿资金。但其canonical bridge仅吸引$2,800万存款，绝大部分资金由空投猎人贡献。一旦空投结束，TVL大幅回落。
- **Blast（2023-2024）**：上线一周即吸引超$6.5亿存款，采用积分系统+邀请制（金字塔式结构），顶层用户获得巨大的代币分配优势。本质上是用未来代币预期换取当前TVL。

**识别方法**：
1. 对比TVL增长曲线与积分/空投活动时间线
2. 检查空投活动结束后TVL是否急剧下降
3. 分析canonical bridge实际存入量 vs 总TVL
4. 观察TVL来源的地址集中度

#### b) 双重计算与杠杆循环

**核心问题**：同一笔资金在多个协议间被重复计算。

- **Restaking循环**：ETH -> Lido(stETH) -> Aave(抵押stETH借出ETH) -> 再次存入 -> 同一笔ETH被计算多次
- **学术研究发现**：2021年12月2日DeFi高峰期，TVL与TVR（Total Value Resolved，去重后真实值）之间的差距高达$1,398.7亿，TVL/TVR比率约为2倍
- **跨链重复计算**：资产桥接、包装和跨协议复用在多链环境中导致同一资本被多次计入

**识别方法**：
1. 使用DeFiLlama的 "Exclude double-counted TVL" 选项
2. 检查协议TVL中LST（液态质押代币）和LRT（液态再质押代币）的占比
3. 计算协议的独立资金来源数量

#### c) 自充值/关联方资金

**手法**：项目方或关联基金将自有资金存入协议以虚增TVL。

**识别方法**：
1. 通过Arkham/Nansen追踪大额存入地址的关联关系
2. 检查前10大存款地址是否与项目方钱包有资金往来
3. 分析TVL的地址集中度（如前10地址占比超60%则高风险）

### 1.3 TVL健康度评估框架

| 指标 | 健康 | 警告 | 危险 |
|------|------|------|------|
| 空投结束后TVL留存率 | >70% | 30-70% | <30% |
| 前10地址TVL占比 | <30% | 30-60% | >60% |
| TVL/TVR比率 | <1.5 | 1.5-2.5 | >2.5 |
| 有机增长（无激励期） | 持续增长 | 持平 | 依赖激励才有增长 |
| Revenue/TVL比率（年化） | >5% | 1-5% | <1% |

---

## 2. 真实用户 vs 刷交互地址（Sybil检测）

### 2.1 定义"活跃用户"

在DeFi中，活跃用户（Active User）指在特定时间窗口内与协议智能合约进行过至少一次有意义链上交互的独立钱包地址。

**三层用户分类**：
1. **Connected Wallets（连接钱包）**：仅连接但未执行操作的被动访客 -- 属于虚荣指标
2. **Active Wallets（活跃钱包）**：执行过特定价值操作（如swap、mint、投票）的地址
3. **Power Users（核心用户）**：高频交互且长期留存的钱包

### 2.2 Sybil攻击检测方法

#### a) 链上行为分析

- **资金来源追踪**：大量地址从同一交易所钱包在短时间内获得初始资金
- **行为模式聚类**：多个钱包在相同时间执行相同操作序列
- **交易时间分析**：交互时间呈现机械化规律（如精确的固定间隔）
- **金额模式**：多个地址使用相似的交易金额

#### b) 图分析与机器学习

- **资金流图谱**：使用Nansen/Arkham追踪钱包间的资金流动关系，识别"扇入-操作-扇出"模式
- **TrustaLabs框架**：开源的[Sybil识别机器学习框架](https://github.com/TrustaLabs/Airdrop-Sybil-Identification)，用于空投反女巫检测
- **AI异常检测**：2025-2026年期间，机器学习检测方法快速发展，结合经济成本与社交图谱的混合方法成为主流

#### c) 身份验证系统（Proof of Personhood）

| 系统 | 方法 | 规模 | 特点 |
|------|------|------|------|
| **Worldcoin** | 虹膜生物识别 | 200万+验证用户 | 高安全性，但隐私争议 |
| **Human Passport**（原Gitcoin Passport） | 多凭证聚合评分 | 保护$4.3亿+资金 | 灵活可组合，多协议集成 |
| **BrightID** | 去中心化社交图谱 | 社区级验证 | 无需生物识别，但规模有限 |
| **Proof of Humanity** | 视频+社交担保 | 链上注册 | 去中心化但用户体验较差 |

### 2.3 实操判断真实用户数

1. **DAU/MAU比率**：>20%表示强用户粘性和产品市场契合度
2. **新地址留存**：追踪首次交互后7天/30天内再次交互的地址比例
3. **交互深度**：单个地址的交互种类数（仅swap vs swap+LP+借贷+治理投票）
4. **Gas费支出分布**：真实用户的Gas费支出呈长尾分布；批量操作的Sybil地址Gas费集中在极低值
5. **时间分布**：真实用户活动呈现24小时周期性波动；机器人活动更均匀

---

## 3. 协议收入来源与可持续性分析

### 3.1 收入分类框架

```
总手续费（Fees）
  |
  +-- 供应方收入（Supply-side Revenue）: 支付给流动性提供者/验证者
  |
  +-- 协议收入（Protocol Revenue）: 协议自身捕获的费用
        |
        +-- 持有人收入（Holders Revenue）: 通过回购/销毁/分红分配给代币持有者
        |
        +-- 金库收入（Treasury Revenue）: 存入协议金库
        |
        +-- 收入成本（Cost of Revenue）: 支付给第三方服务商（如节点运营商、DA费用）
              |
              +-- 净收益（Earnings）= 协议收入 - 收入成本 - 代币激励
```

**来源**: [Token Terminal Metrics](https://tokenterminal.com/explorer/metrics)

### 3.2 可持续 vs 不可持续收入

| 维度 | 可持续收入 | 不可持续收入 |
|------|-----------|-------------|
| **来源** | 交易手续费、借贷利息、清算罚金 | 代币激励、积分奖励 |
| **驱动力** | 用户需求驱动 | 代币排放驱动 |
| **周期性** | 与市场活跃度正相关但可持续 | 激励结束即消失 |
| **典型协议** | Uniswap、Aave、MakerDAO | 早期流动性挖矿项目 |
| **指标** | Fee Revenue > Token Incentives | Token Incentives >> Fee Revenue |

### 3.3 关键判断指标

- **Earnings = Revenue - Cost of Revenue - Token Incentives**：若Earnings为负，说明协议在"花钱买用户"
- **Token Incentives / Revenue 比率**：>100%意味着协议支付的代币激励超过其收入（不可持续）
- **Revenue / TVL 比率（年化）**：
  - DEX：5-20%（Uniswap约8-15%）
  - 借贷：1-5%（Aave约2-4%）
  - 收益聚合器：<1%
  - 若Revenue/TVL极低（<0.5%），TVL可能是激励驱动而非有机需求

### 3.4 收入验证步骤

1. **Token Terminal查看Earnings**：若持续为负 -> 不可持续
2. **DeFiLlama对比Fees vs Incentives**：观察代币激励占总费用的比例变化趋势
3. **追踪激励削减后的TVL变化**：激励减少时TVL同步下降 -> 资本不具粘性
4. **分析用户付费意愿**：即使无激励，用户是否仍会使用（有替代方案成本更高的场景）

---

## 4. 链上数据分析工具

### 4.1 DeFiLlama

**定位**：DeFi协议TVL、费用和收入的免费聚合仪表盘

**核心功能**：
- **TVL追踪**：支持所有主流链和协议的TVL追踪，带"排除双重计算"选项
- **Fees & Revenue**：按协议和链维度查看手续费和收入
- **Yields**：追踪DeFi收益率，辅助判断收益来源（有机 vs 激励）
- **Stablecoins**：稳定币供应量和流通监控
- **DEX Volume**：去中心化交易所交易量
- **Raises**：融资记录追踪

**尽调用法**：
1. 查看协议TVL历史曲线，对比重大事件时间点
2. 使用"Exclude double-counted" 获取更真实的TVL
3. 比较同赛道协议的 Revenue/TVL 比率
4. 追踪TVL按链和资产的分布

**来源**: [DeFiLlama](https://defillama.com/), [How to Use DeFiLlama (2026)](https://www.dextools.io/tutorials/how-to-use-defillama-defi-tvl-stablecoin-revenue-tutorial-2026)

### 4.2 Token Terminal

**定位**：加密协议基本面分析平台（类似Bloomberg Terminal for crypto）

**核心指标**：
- **Revenue / Fees / Earnings**：区分总费用、协议收入和净收益
- **Cost of Revenue**：协议外包核心业务的成本（如LST协议的节点运营商费用、L2的DA费用）
- **P/E & P/S Ratios**：市盈率和市销率估值
- **Active Users**：活跃用户数追踪
- **FDV（完全稀释估值）**：结合收入数据计算估值倍数

**尽调用法**：
1. 检查Earnings是否为正（可持续性核心指标）
2. 比较同赛道P/S倍数（过高可能高估，过低需查原因）
3. 追踪Revenue增长趋势 vs 代币激励趋势
4. 使用Token Incentives指标量化"补贴率"

**来源**: [Token Terminal](https://tokenterminal.com/), [Token Terminal Tutorial (2026)](https://www.dextools.io/tutorials/how-to-use-token-terminal-fees-revenue-metrics-tutorial-2026)

### 4.3 Dune Analytics

**定位**：社区驱动的链上数据查询和可视化平台

**核心特点**：
- **SQL查询**：直接对区块链数据运行SQL查询
- **社区仪表盘**：海量社区贡献的预制仪表盘
- **自定义分析**：可构建自定义查询以验证特定假设
- **原始数据级别**：从区块链底层出发，不依赖预处理数据

**尽调用法**：
1. 搜索目标协议的社区仪表盘获取快速概览
2. 自定义查询验证TVL数据的真实性（如独立计算合约内余额）
3. 分析用户地址的交互模式（识别Sybil行为）
4. 追踪大户资金流向

**来源**: [Dune Analytics](https://dune.com/)

### 4.4 Nansen

**定位**：AI驱动的钱包标签和Smart Money追踪平台

**核心能力**：
- **钱包标签**：标记超过5亿个加密钱包，覆盖18+条链
- **Smart Money分类**：基于历史盈利表现标记顶级交易者、LP、空投农民等
- **实时提醒**：追踪标记钱包的实时链上行为
- **持仓分析**：分析已标记地址的代币持仓和DeFi仓位

**尽调用法**：
1. 查看协议TVL中Smart Money占比及其持仓变化
2. 追踪VC/基金钱包是否在卖出项目代币
3. 分析新项目的早期用户画像（是否为空投农民主导）
4. 监控大户的DeFi仓位变化

**定价**：$999/月起（专业版），功能全面但成本高

**来源**: [Nansen](https://www.nansen.ai/), [Smart Money Tracking Guide](https://www.nansen.ai/post/how-to-analyze-blockchain-data-for-smart-money-movements)

### 4.5 Arkham Intelligence

**定位**：免费实体标签和链上情报平台

**核心能力**：
- **Profiler**：展示钱包组合、历史表现、交易对手和标记的交易历史
- **Visualizer**：可视化钱包间资金流动关系图
- **Intel Exchange**：使用ARKM代币买卖链上情报的市场
- **Ultra AI引擎**：用于实体匹配的AI系统

**尽调用法**：
1. 使用Profiler分析协议合约的资金来源
2. 使用Visualizer追踪可疑资金流向
3. 检查项目方钱包的代币持有和销售行为
4. 交叉验证大额存款地址与项目方的关系

**定价**：免费（含实体标签），是Nansen的低成本替代方案

**来源**: [Arkham Intelligence](https://www.arkham.com/), [Coin Bureau Review](https://coinbureau.com/review/arkham-intelligence-review)

### 4.6 工具对比矩阵

| 特性 | DeFiLlama | Token Terminal | Dune | Nansen | Arkham |
|------|-----------|---------------|------|--------|--------|
| **价格** | 免费 | 免费+付费 | 免费+付费 | $999/月 | 免费 |
| **TVL数据** | 核心功能 | 有 | 自定义 | 有 | 无 |
| **收入/费用** | 有 | 核心功能 | 自定义 | 有限 | 无 |
| **钱包标签** | 无 | 无 | 社区 | 核心功能 | 核心功能 |
| **自定义查询** | 无 | 有限 | 核心功能 | 有限 | 无 |
| **Smart Money** | 无 | 无 | 社区 | 核心功能 | 有 |
| **实时提醒** | 无 | 有 | 有 | 有 | 有 |
| **适用场景** | TVL/收入概览 | 基本面估值 | 深度自定义分析 | 机构级钱包追踪 | 免费实体分析 |

---

## 5. 核心指标体系

### 5.1 DAU / MAU（日活/月活）

**定义**：在指定时间窗口内与协议智能合约进行过至少一次成功链上交易的唯一地址数。

**注意事项**：
- 一个用户可能控制多个钱包 -> DAU可能高估真实用户数
- Sybil地址可能大幅膨胀DAU -> 需结合Sybil检测
- DeFi的DAU/MAU比率>20%被认为是优秀的用户粘性

**数据来源**：Token Terminal、Dune Analytics自定义查询

### 5.2 交易量（Volume）

**定义**：协议处理的交易总额。

**关键区分**：
- **有机交易量**：真实交易需求产生
- **Wash Trading量**：同一实体自买自卖制造的虚假交易量
- Chainalysis 2024年报告估计ERC20和BEP20代币的刷量交易高达$25.7亿

**验证方法**：
- 对比链上交易量 vs 链下报告交易量
- 分析交易量的时间分布（真实交易跟随市场周期，刷量更均匀）
- 检查大额交易是否在短时间内反复往返于相同地址对
- 使用Chainalysis的wash trading启发式规则：交易时间、金额、频率、地址关联

### 5.3 Revenue / TVL 比率

**含义**：每单位锁定资本产生的收入，衡量资本效率和真实需求。

**基准值**（年化）：

| 协议类型 | 健康范围 | 说明 |
|----------|---------|------|
| DEX | 5-20% | Uniswap V3 由于集中流动性效率更高 |
| 借贷 | 1-5% | 取决于利用率和利差 |
| 永续合约DEX | 10-50% | 杠杆交易产生更多费用 |
| 收益聚合器 | 0.1-1% | 本质是管理费模式 |
| 跨链桥 | 1-5% | 与桥接量正相关 |

**解读**：
- Revenue/TVL极低（<0.5%年化）：TVL可能靠激励堆叠而非有机需求
- Revenue/TVL异常高（>50%年化）：可能存在wash trading虚增交易量

### 5.4 用户留存率

**定义**：首次交互后在后续时间窗口再次交互的地址比例。

**衡量方法**：
- **D7留存**：首次交互后7天内再次交互的地址占比
- **D30留存**：首次交互后30天内再次交互的地址占比
- **长期粘性**：连续3个月每月至少交互一次的地址占比

**数据获取**：Dune Analytics自定义SQL查询是最灵活的方式

### 5.5 综合评分框架

| 指标 | 权重 | 优秀 | 合格 | 危险 |
|------|------|------|------|------|
| DAU/MAU比率 | 15% | >25% | 10-25% | <10% |
| Revenue/TVL（年化） | 20% | >5% | 1-5% | <1% |
| Earnings正负 | 20% | 持续为正 | 偶尔为正 | 持续为负 |
| D30用户留存 | 15% | >30% | 10-30% | <10% |
| TVL地址集中度 | 15% | 前10<30% | 30-50% | >50% |
| 激励结束后TVL留存 | 15% | >70% | 40-70% | <40% |

---

## 6. 历史案例：虚假链上数据

### 6.1 Wash Trading案例

#### Chainalysis 2025报告：$25.7亿刷量

Chainalysis通过两套独立启发式规则，识别出2024年DEX上ERC20和BEP20代币的刷量交易总额高达$25.7亿（上限估计，两套规则可能有重叠）。

**关键发现**：
- 启发式规则1：$7.04亿
- 启发式规则2：$18.7亿（覆盖Ethereum、BNB Chain、Base）
- 2024年发行的300万+代币中，仅1.7%在报告时仍活跃交易
- Pump-and-dump占2024年市场活动的4.52%，高于2023年的3.59%

**来源**: [Chainalysis: Crypto Market Manipulation 2025](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)

#### NFT/代币市场刷量

AMM（自动做市商）上的刷量更加隐蔽，研究表明自动化方法可以将刷量行为伪装成正常市场参与者的交易模式，使简单启发式规则无法检测。

### 6.2 虚假/膨胀TVL案例

#### Manta Pacific -- 空投驱动的TVL海市蜃楼

- TVL在2023年12月暴涨1,250%，达到$3.45亿
- 实际通过canonical bridge存入仅$2,800万
- 绝大部分资金来自空投猎人，而非有机用户
- 空投结束后TVL大幅回落

**教训**：canonical bridge存款量 vs 总TVL的巨大差异是关键红旗。

#### Blast -- 积分金字塔

- 上线一周吸引$6.5亿+存款
- 采用金字塔式邀请积分系统
- 早期/顶层邀请者获得不成比例的代币分配
- 资金本质上是被代币预期锁定，而非协议使用需求

**教训**：积分系统的邀请层级结构是判断TVL有机性的关键信号。

#### Multichain Bridge -- 内部人员挪用

- 2023年7月，$1.26亿资金异常流出
- CEO赵军被中国警方逮捕，团队失去MPC密钥
- Chainalysis分析确认为内部人员操作（可能的rug pull）
- 跨链桥因集中式资产存储成为高风险目标

**教训**：跨链桥的中心化密钥管理是系统性风险；需审查密钥管理架构。

**来源**: [Chainalysis Multichain Exploit Analysis](https://www.chainalysis.com/blog/multichain-exploit-july-2023/)

#### TVL双重计算 -- 行业系统性问题

- 2021年12月DeFi高峰期，TVL与去重后的TVR差距达$1,398.7亿
- TVL/TVR比率约为2，意味着行业TVL整体虚高约一倍
- restaking叙事进一步加剧了这一问题（ETH -> stETH -> eETH -> 多次计入）

**来源**: [Piercing the Veil of TVL: DeFi Reappraised (Springer)](https://link.springer.com/chapter/10.1007/978-3-032-07035-7_1)

### 6.3 指标造假的警告信号总结

Lex Sokolin（2025年11月）公开指出加密行业广泛存在的指标造假现象，包括：粉丝数、用户数、交易量、账户数、稳定币供应量、互动量和TVL。他将这种行为称为"blitzscaling vaporware"。

**结构性链上信号用于识别欺诈项目**：
- **代币集中度**：诈骗代币在前1%持有者中的代币集中度和持有方差显著高于正常项目
- **流动性池锁定**：开发者控制的未锁定流动性池是rug pull的前提条件
- **合约代码**：通过验证的源代码掩盖恶意逻辑，将漏洞藏在复杂性或条件逻辑中

---

## 7. 实操检查清单

### Phase 1: 快速筛查（30分钟）

- [ ] **DeFiLlama**：查看协议TVL历史曲线，是否有与空投/激励活动同步的异常跳升
- [ ] **DeFiLlama**：使用"Exclude double-counted"选项获取更真实TVL
- [ ] **Token Terminal**：检查Earnings是否为正、Revenue趋势
- [ ] **Token Terminal**：查看Token Incentives / Revenue比率
- [ ] **DeFiLlama**：对比Revenue/TVL与同赛道基准值

### Phase 2: 深度分析（2-4小时）

- [ ] **Arkham/Nansen**：追踪协议前10大存款地址，检查与项目方的关联
- [ ] **Dune**：查询协议的DAU/MAU数据和留存率
- [ ] **Dune**：分析用户地址交互模式，识别Sybil行为特征
- [ ] **Arkham Visualizer**：可视化大额资金流向，追踪可疑循环
- [ ] 对比激励削减前后的TVL和用户数变化

### Phase 3: 交叉验证（1-2小时）

- [ ] 独立计算合约余额 vs DeFiLlama报告TVL
- [ ] 检查交易量时间分布和交易对手分析（wash trading检测）
- [ ] 验证代币持有者分布的健康度（Gini系数/前10集中度）
- [ ] 检查流动性池是否锁定以及锁定期限
- [ ] 审查密钥管理架构（多签 vs 单点控制）

---

## 参考来源

- [DefiLlama Methodology](https://docs.llama.fi/)
- [How to Use DeFiLlama (2026)](https://www.dextools.io/tutorials/how-to-use-defillama-defi-tvl-stablecoin-revenue-tutorial-2026)
- [Token Terminal Metrics](https://tokenterminal.com/explorer/metrics)
- [Token Terminal Tutorial (2026)](https://www.dextools.io/tutorials/how-to-use-token-terminal-fees-revenue-metrics-tutorial-2026)
- [Nansen Smart Money Guide](https://www.nansen.ai/post/how-to-analyze-blockchain-data-for-smart-money-movements)
- [Nansen Wallet Tracking Guide](https://www.nansen.ai/post/how-to-monitor-wallet-activity-track-smart-money-in-crypto)
- [Arkham Intelligence Review (Coin Bureau)](https://coinbureau.com/review/arkham-intelligence-review)
- [Chainalysis: Crypto Market Manipulation 2025](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)
- [Chainalysis: Multichain Exploit Analysis](https://www.chainalysis.com/blog/multichain-exploit-july-2023/)
- [Chainalysis: 2025 Crypto Crime Report](https://www.chainalysis.com/blog/2025-crypto-crime-report-introduction/)
- [Piercing the Veil of TVL: DeFi Reappraised (Springer)](https://link.springer.com/chapter/10.1007/978-3-032-07035-7_1)
- [DeFi's Trust Crisis (AInvest)](https://www.ainvest.com/news/defi-deepening-trust-crisis-tvl-security-risks-signal-reckoning-crypto-investors-2512/)
- [Sybil Attacks in Crypto (Formo)](https://formo.so/blog/what-are-sybil-attacks-in-crypto-and-how-to-prevent-them)
- [TrustaLabs Sybil Identification Framework (GitHub)](https://github.com/TrustaLabs/Airdrop-Sybil-Identification)
- [Human Passport (Gitcoin Passport)](https://passport.human.tech/)
- [Web3 Active Users: DAU, WAU, MAU (Formo)](https://formo.so/blog/web3-active-users-dau-wau-mau)
- [User Lifecycle Analysis for DeFi (Formo)](https://formo.so/blog/user-lifecycle-analysis-for-web3-and-defi-apps)
- [Multi-Chain TVL Problem (Lampros Tech)](https://lampros.tech/blogs/multi-chain-tvl-problem)
- [Inflated Bitcoin DeFi TVL (Bitcoin News)](https://news.bitcoin.com/inflated-tvl-figures-could-pop-a-30b-bitcoin-defi-ecosystem/)
- [Lex Sokolin on Fake Crypto Metrics (Blockchain News)](https://blockchain.news/flashnews/lex-sokolin-warns-of-fake-crypto-metrics-volume-tvl-stablecoin-supply-trader-risk-checklist-and-data-sources)
