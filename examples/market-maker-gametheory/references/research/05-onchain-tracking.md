# 做市商链上行为追踪方法论

> 系统性追踪框架：从钱包识别到库存监控，从工具实操到案例分析

---

## 一、识别做市商钱包的方法

### 1.1 高频交易模式识别

做市商钱包在链上呈现出明显区别于普通用户的行为特征：

- **交易频率异常**：专业做市商钱包每分钟可发起数十笔交易，远超普通用户。在以太坊上，可通过 nonce 序列模式（nonce sequence patterns）分析钱包的管理实践，高频递增的 nonce 是做市商钱包的典型标志。
- **时间模式分析**：不同类型的用户呈现出不同的时间行为特征——机构交易者在工作时间活跃，散户投资者则对新闻事件做出反应。做市商的特征是7x24小时不间断、高度规律的交易模式。
- **Gas 费策略**：做市商倾向于使用高于平均水平的 Gas 费以确保交易优先执行，尤其在套利窗口期。Gas 费的使用策略可以反映用户的紧迫程度和专业程度。
- **大宗交易执行**：做市商的签名特征包括在多个时间段内执行大宗交易（large block execution），并呈现出与交易量峰值相关的系统性规律。

### 1.2 资金流转模式

- **多交易所资金流转**：做市商在多个中心化和去中心化交易平台之间持续移动大额资金，形成独特的跨平台资金流转网络。
- **跨链桥使用频率**：跨链桥已成为加密套利工具箱的核心组成部分。做市商频繁使用跨链桥在不同链上的 DEX 之间套利，其跨链交易频率和金额远超普通用户。
- **与已知交易所热钱包的高频互动**：做市商钱包与 Binance、Coinbase 等主流交易所存取款地址之间存在大量、规律性的交互。

### 1.3 钱包聚类与实体识别

**地址聚类（Address Clustering）** 是链上做市商识别的基础方法论：

- **共同输入所有权启发式（Common Input Ownership Heuristic）**：如果多个地址在同一笔交易中作为输入使用，则假定它们由同一实体控制。这是最基本的聚类方法。
- **找零地址检测（Change Address Detection）**：追踪交易中的找零地址可以将更多地址关联到同一实体。
- **钱包软件指纹识别**：不同钱包软件生成交易的方式存在细微差异，可用于辅助聚类。
- **行为模式匹配**：通过关联多个钱包之间交易的时间和金额来推断实体关系。

通用聚类启发式可以应用于区块链上的多种服务或钱包，但特定服务型启发式则是在通用方法无法全面捕捉交易行为时所需要的补充。

### 1.4 Arkham Intelligence 已标记的做市商实体

Arkham Intelligence 是专门设计用于揭示钱包所有者并精确追踪链上活动的区块链分析平台。其核心能力包括：

- **实体标签系统**：将钱包地址与个人、机构或组织进行关联。Arkham 维护了主要做市商的标记档案页面，包括：
  - [Wintermute](https://intel.arkm.com/explorer/entity/wintermute)——已标记多个关联钱包
  - [Jump Trading](https://intel.arkm.com/explorer/entity/jump-trading)——完整的实体档案
  - DWF Labs、GSR、Cumberland 等主要做市商
- **资金流可视化**：支持发现持仓、钱包、流入、流出和交易对手方。
- **交叉分析**：支持跨多个区块链的资金追踪和实体分析。

---

## 二、CEX-DEX 搬砖行为模式

### 2.1 套利机制

CEX 和 DEX 之间的价格差异为做市商提供了低风险套利机会。这些临时价格差异源自以下因素：

- **流动性差异**：不同交易所的订单簿深度和 AMM 流动性池规模不同
- **区域需求差异**：不同市场的供需关系存在时间差
- **价格发现速度差异**：CEX 的订单簿模型和 DEX 的 AMM 模型在价格发现速度上存在天然差异

**典型套利流程**：
1. 做市商在 DEX（如 Uniswap）上发现代币价格低于 CEX（如 Binance）
2. 在 DEX 上买入代币
3. 将代币转入 CEX 卖出
4. 或反向操作：CEX 买入 -> 提币到链上 -> DEX 卖出

**原子交易套利**：专门设计的套利智能合约可以在单笔交易中在不同市场执行多笔交易，原子交易保证要么所有交易都通过，要么全部回滚，消除了执行风险。

### 2.2 对价格的影响

- **价格收敛效应**：做市商的套利活动客观上促进了 CEX 和 DEX 之间的价格趋同，提高了市场效率。
- **流动性虹吸**：大规模搬砖活动可能暂时性地耗尽 DEX 流动性池中某一侧的资产，导致滑点增大。
- **DEX 交易量占比上升**：截至2025年底，DEX 相对于 CEX 的交易量占比已超过 20%，并在2026年持续增长。这意味着最大的加密做市商将是那些能够在集中式和去中心化交易之间无缝切换的机构。

### 2.3 链上可观察的搬砖信号

- **CEX 存取款模式**：大额代币从链上地址转入 CEX 存款地址，或从 CEX 提款到链上地址，随后在 DEX 上出现对应方向的交易。
- **时间关联性**：DEX 交易与 CEX 存取款之间呈现出高度时间关联，通常在数分钟内完成闭环。
- **Gas 费异常**：套利交易往往使用较高的 Gas 费来确保执行速度。

---

## 三、做市商库存变化的监控

### 3.1 库存管理的链上信号

做市商的库存管理是其核心运营的关键环节。链上可观察的库存变化包含重要信号：

**持仓增加（吸筹信号）**：
- 链上指标如活跃地址数、交易所流出量（exchange outflows）和币龄（coin age）显示供应正在整合
- 现货交易量保持低迷的同时，特定地址持续买入
- 聪明的参与者使用分阶段买入策略，包括定投（DCA）和在支撑位附近设置限价单

**持仓减少（出货信号）**：
- 大量代币从做市商标记钱包流向交易所存款地址
- 做市商开始在多个交易所分散出货以减少市场冲击
- 链上余额持续下降同时交易所余额上升

### 3.2 区分做市活动与方向性交易

做市活动和方向性投机在链上表现不同：

| 特征 | 做市活动 | 方向性交易 |
|------|----------|------------|
| 持仓变化速度 | 缓慢、渐进 | 快速、集中 |
| 双向性 | 买卖并行，保持平衡 | 单一方向持续操作 |
| 与价格的关系 | 在价差两侧提供流动性 | 追随或预判价格趋势 |
| 库存回归 | 倾向于回归目标水平 | 持续偏离 |
| 跨平台行为 | 多平台同时报价 | 可能集中在单一平台 |

**关键区分指标**：
- **订单簿倾斜**：当订单簿出现持续不平衡——买单远多于卖单或反之——通常反映做市商意图。买方不平衡可能表示积极吸筹或意图推高价格；卖方不平衡则可能信号出货或打压价格意图。
- **库存回归速度**：真正的做市活动中，如果市场突然趋势化导致库存偏离，算法会通过在替代场所执行对冲交易来防止危险的头寸积累。
- **阈值触发机制**：如果敞口超过预定阈值，实时监控软件会通过一系列卖单触发自动响应。

### 3.3 库存变化速度的信号意义

- **缓慢库存积累**：通常是正常做市活动中的被动库存积累，或有计划的战略建仓
- **突然的库存激增**：可能表明做市商获取了信息优势（如即将上线新交易所），或正在为特定事件做准备
- **快速清仓**：高度警惕信号，尤其当伴随着做市商创始人在社交媒体上推广该代币时（参见 DWF Labs 案例）
- **系统性资本轮转**：做市商会根据收益率差异进行有计划的资本轮转，这是正常的做市行为

---

## 四、工具实操指南

### 4.1 Arkham Intelligence

**核心功能**：
- **实体追踪（Entity Tracking）**：输入做市商名称（如 "Wintermute"、"Jump Trading"），查看该实体所有已标记钱包的持仓、交易和资金流向
- **地址分析（Address Explorer）**：输入具体钱包地址，查看完整的交易历史、持仓变化和交易对手方
- **警报系统（Alerts）**：设置特定做市商钱包的大额交易警报
- **资金流图谱**：可视化做市商与其他实体之间的资金流动关系

**实操步骤**：
1. 访问 [intel.arkm.com](https://intel.arkm.com)
2. 在搜索栏输入做市商名称或已知钱包地址
3. 查看 "Portfolio" 标签了解当前持仓
4. 查看 "Transactions" 标签追踪最新交易
5. 使用 "Counterparties" 功能发现关联实体
6. 设置 Alert 监控大额异动

**已知做市商实体页面**：
- Wintermute: `intel.arkm.com/explorer/entity/wintermute`
- Jump Trading: `intel.arkm.com/explorer/entity/jump-trading`

### 4.2 Nansen 标签系统

**标签体系**：
Nansen 汇聚了来自 20+ 条链、超过 5 亿个已标记地址的数据。关键标签类型包括：

- **Smart Money**：基于 PnL 表现自动识别的顶级链上参与者
- **Fund**：公开的投资管理实体
- **First Mover LP**：早期参与流动性池的地址
- **First Mover Staking**：早期参与质押的地址
- **Emerging Smart Trader**：PnL 可比 Smart Money 但交易规模较小的地址

**做市商追踪实操**：
1. 使用 **Smart Money Dashboard**：过滤 "Fund" 或 "Market Maker" 标签，查看过去24小时或7天内净买入或净卖出的情况
2. 使用 **Token God Mode**：输入特定代币，查看 Smart Money 持有者、近期交易和余额变化
3. 使用 **Wallet Profiler**：对已知做市商钱包进行详细分析，包括完整投资组合、盈亏和交易历史
4. 使用 **Smart Segments**：设定自定义条件（如"过去7天交易量 > $10M 且与已知 DEX 交互 > 100 次"），自动识别符合条件的地址
5. 设置 **Smart Alerts**：对大额买入、卖出、转账和复杂 DeFi 交互设置警报

### 4.3 Dune Analytics

**平台能力**：
Dune 是链上数据分析平台，支持 100+ 条区块链、3+ PB 数据，通过 SQL 查询实现自定义分析和可视化。

**做市商相关 Dashboard**：
- **Exchange Flows**（`dune.com/soispoke/Exchange-Flows`）：追踪主要交易所的资金流入流出
- 自建查询可追踪特定做市商钱包的 DEX 交易行为

**自定义查询示例**：
```sql
-- 追踪特定地址在 Uniswap 上的交易活动
SELECT
  block_time,
  token_bought_symbol,
  token_bought_amount,
  token_sold_symbol,
  token_sold_amount,
  amount_usd
FROM dex.trades
WHERE taker = 0x... -- 做市商钱包地址
ORDER BY block_time DESC
LIMIT 100;
```

**使用建议**：
- Dune 主要专注于链上数据，不直接提供 CEX 订单簿数据
- 适合追踪做市商在 DEX 上的交易行为、LP 操作和跨协议交互
- 通过 API 和 DataShare 功能可以将数据导出做进一步分析

### 4.4 交易所 API 获取订单簿快照

**Kaiko——机构级市场数据**：
- 提供 Level 1（市场交易）和 Level 2（交易所订单簿）数据
- **订单簿快照**：每30秒生成一次，覆盖最佳买卖价 10% 深度范围内的原始快照
- **市场深度指标**：在最佳买卖价 0-10% 范围内聚合买卖单量，更高的量意味着更多流动性
- **买卖价差（Bid-Ask Spread）**：最高买价与最低卖价之差，更小的价差意味着更多流动性
- **滑点分析**：基于假设订单规模（如 $100K 买入或卖出）提供滑点百分比数据

**其他数据来源**：
- **CoinAPI**：提供跨交易所的标准化市场数据 API
- **交易所原生 API**：
  - Binance API: `GET /api/v3/depth` 获取订单簿快照
  - Coinbase API: `GET /products/{product_id}/book` 获取 Level 2 订单簿
  - OKX API: `GET /api/v5/market/books` 获取订单簿数据

### 4.5 其他关键工具

- **Chainalysis**：企业级区块链情报平台，使用高级启发式、聚类算法和专有归因方法追踪跨链资金流
- **TRM Labs**：提供深度交易映射和实体追踪功能
- **MetaSleuth**：用于跨链资金流动分析
- **Etherscan / Blockscan**：基础链上浏览器，查看具体交易和地址详情
- **Hypernative**：实时风险监控平台，Wintermute 等做市商使用其来扩展 DeFi 操作

---

## 五、具体案例分析

### 5.1 ZachXBT 的链上侦查方法论

**背景**：ZachXBT 是自学成才的区块链调查员，在2018年因加密骗局损失 $15K 后，将挫折转化为使用链上分析追踪欺诈者的使命。四年间发布了 200+ 份报告，追回了数亿美元被盗资金。

**核心方法论**：
1. **多层调查方法**：将链上追踪与传统 OSINT 结合——抓取 X、Discord、Telegram、Instagram 和公共记录
2. **工具组合**：
   - TRM Labs / Arkham：深度交易映射和实体追踪
   - Pulsy / Socketscan：分析跨链资金移动
   - Dune Analytics：查询区块链数据集以检测趋势和异常
   - Wayback Machine：历史数据存档
3. **追踪技术**：
   - 分析链上交易痕迹
   - 追踪 Railgun 隐私协议流
   - 分析 Avalanche 桥接记录
   - 通过不寻常的 NFT 清洗交易追溯到可疑地址
4. **典型案例**：
   - 暴露 $70M Pixelmon 骗局
   - 追踪朝鲜 Lazarus Group 通过 Tornado Cash 洗钱 $200M
   - 在 Movement Labs 市场做市商不当行为案中指出与 Web3Port 的潜在关联

### 5.2 DWF Labs 链上行为被追踪暴露

**事件概述**：

DWF Labs 是2023-2024年最具争议的加密做市商之一，其链上行为经多方调查被暴露。

**链上证据**：
- **Wash Trading**：Binance 内部调查员的报告指控 DWF 在2023年操纵了多个代币价格，涉及超过 $3 亿的清洗交易。
- **YGG 代币操纵**：调查发现 DWF 操纵了 YGG 和至少其他六个代币的价格。
- **Pump and Dump 模式**：调查还发现了 DWF Labs 出售其创始人先前推广的代币导致价格崩溃的案例，符合拉高出货（pump and dump）模式。
- **链上资金流向**：通过追踪 DWF Labs 标记钱包的链上行为，可以观察到其在投资项目代币后，快速将代币从锁仓地址转移并在市场上出售的模式。

**后续影响**：
- Binance 在报告提交一周后解雇了调查团队负责人
- DWF Labs 否认操纵指控，创始合伙人 Heng Yu Lee 称这些指控是"竞争对手驱动的 FUD"
- 该事件引发了行业对做市商行为监督的广泛讨论

**来源**：
- [CoinDesk: Binance Fired Investigator Who Uncovered Market Manipulation at Client DWF Labs](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [CoinDesk: Crypto Market Maker DWF Labs' $200M Deals Blur What 'Investing' Means](https://www.coindesk.com/business/2023/04/14/market-maker-dwf-labs-more-than-200m-in-deals-blur-what-investing-means)

### 5.3 Alameda Research 的链上行为模式

**Nansen 的区块链分析揭露了 FTX/Alameda 的核心问题**：

**早期关联（2019年）**：
- 链上数据显示 Alameda 是 FTX 成立前唯一可明确识别的交易对手方
- Alameda 在 FTT 上线前两天从 FTX 部署合约接收了 500万 FTT 代币

**FTT 代币集中度异常**：
- 链上数据显示约 2.8 亿（占总供应量 3.5 亿的约 80%）FTT 由 FTX 单独持有
- 大量 FTT 交易量在 FTX 和 Alameda 的多个钱包之间流转
- 代币供应的大部分被锁定在一个 3 年期归属合约中，唯一受益人是 Alameda 控制的钱包

**循环资金流**：
- 历史链上数据显示 FTX、Alameda 和 Genesis Trading 钱包之间存在定期的大额资金流入流出，转账量高达 $17 亿（2021年12月数据）
- CoinDesk 报告发布前（2022年9月28日至11月1日），出现异常交易：$41 亿 FTT 从 Alameda 转入 FTX，以及总计 $3.88 亿稳定币从不同来源流入 Alameda 钱包

**中间人钱包混淆**：
- 链上分析发现不寻常的钱包模式表明存在混淆企图："FTX 经常将 FTT 代币发送到 Fund: 0xf155，然后再发送到 Alameda Research"，之后再返回 FTX
- 这种循环转账模式是做市商和关联实体之间隐藏资金流向的典型手法

**方法论启示**：
- Nansen 研究团队采用"扎根理论方法"（grounded theory approach），以钱包余额和交易量为中心进行分析
- 通过追踪四个关键时间框架的交易数据重构事件链
- 钱包标签启发式、投资组合仪表板分析和交易模式匹配的组合使用是还原真相的关键

**来源**：
- [Nansen: Blockchain Analysis: The Collapse of Alameda and FTX](https://www.nansen.ai/research/blockchain-analysis-the-collapse-of-alameda-and-ftx)
- [CoinDesk: On-Chain Data Shows Close Ties Between FTX and Alameda](https://www.coindesk.com/business/2022/11/22/on-chain-data-shows-close-ties-between-ftx-and-alameda-were-there-from-the-start-nansen)

### 5.4 Movement Labs / MOVE 代币做市商丑闻（2025年）

**事件概述**：
- Movement Labs 正在调查其是否被误导签署了一份做市协议，该协议赋予了一个名不见经传的中间商（Rentech）对 6600 万 MOVE 代币的控制权
- 这些代币在上市后被迅速抛售，触发了 $3800 万的大抛售

**链上追踪要点**：
- 链上调查员 ZachXBT 指出与 Web3Port 的潜在关联
- Binance 封禁了涉嫌操纵的做市商 Web3Port，MOVE 代币价值随后下跌超过 20%
- Coinbase 暂停了 MOVE 代币交易

**自我交易（Self-Dealing）的红旗**：
- 内部合同显示 Rentech 同时以 Web3Port 子公司和 Movement Foundation 代理人的身份出现在交易双方
- 基金会官员最初将 Rentech 交易标记为"可能是他们见过的最糟糕的协议"
- 该协议创造了在向散户投资者抛售代币前拉高 MOVE 价格的激励

**来源**：
- [CoinDesk: Inside Movement's Token-Dump Scandal](https://www.coindesk.com/tech/2025/04/30/inside-movement-s-token-dump-scandal-secret-contracts-shadow-advisors-and-hidden-middlemen)
- [Blockworks: Movement Labs Market Maker Investigation](https://blockworks.com/news/movement-market-maker-investigation)

---

## 六、系统性追踪框架总结

### 6.1 追踪流程

```
Step 1: 实体识别
  ├── Arkham Intelligence 查找已标记做市商实体
  ├── Nansen 标签系统筛选 Smart Money / Fund 标签
  └── 通过地址聚类启发式关联未标记钱包

Step 2: 行为监控
  ├── 设置 Arkham / Nansen 大额交易警报
  ├── Dune Analytics 自定义查询追踪 DEX 交易
  └── 监控 CEX 存取款模式（交易所流入/流出）

Step 3: 库存分析
  ├── 追踪做市商钱包余额变化趋势
  ├── 区分做市活动 vs 方向性交易
  └── 关注库存变化速度的异常信号

Step 4: 市场影响评估
  ├── Kaiko 订单簿数据分析流动性变化
  ├── 交易所 API 获取实时买卖价差和市场深度
  └── 关联链上行为与价格变动的时间序列
```

### 6.2 关键红旗信号

1. **做市商钱包快速清仓某代币**——可能信号出货或对项目失去信心
2. **大量代币从锁仓地址异常转出**——可能违反锁仓协议
3. **做市商创始人公开推广某代币后钱包出现大额卖出**——Pump and Dump 信号
4. **同一实体的多个钱包同时向 CEX 转入代币**——协调出货
5. **与匿名中间人钱包的异常大额交互**——可能的混淆行为
6. **循环转账模式**（如 A -> B -> C -> A）——可能的清洗交易

### 6.3 工具矩阵

| 工具 | 核心用途 | 数据范围 | 适用场景 |
|------|----------|----------|----------|
| Arkham Intelligence | 实体标签、资金流可视化 | 多链链上数据 | 做市商实体识别与追踪 |
| Nansen | Smart Money 标签、钱包分析 | 20+ 链、5 亿+ 标记地址 | 做市商钱包分类与行为分析 |
| Dune Analytics | SQL 自定义查询、Dashboard | 100+ 链链上数据 | DEX 交易追踪、自定义分析 |
| Kaiko | 订单簿数据、流动性指标 | CEX Level 1/2 数据 | 市场深度与做市质量评估 |
| Chainalysis | 企业级合规追踪 | 跨链全覆盖 | 资金流追踪与聚类分析 |
| TRM Labs | 交易映射、实体追踪 | 多链覆盖 | 深度调查与合规 |
| Etherscan | 基础交易查询 | 以太坊及 EVM 链 | 具体交易验证 |

---

## 参考来源

### 平台与工具
- [Arkham Intelligence - Wintermute Entity](https://intel.arkm.com/explorer/entity/wintermute)
- [Arkham Intelligence - Jump Trading Entity](https://intel.arkm.com/explorer/entity/jump-trading)
- [Nansen: How to Track Crypto Wallets](https://www.nansen.ai/post/how-to-track-crypto-wallets-across-multiple-blockchains-with-nansen)
- [Nansen: Understanding Smart Money Labels](https://www.nansen.ai/research/following-the-nerds-understanding-smart-money-labels-and-how-to-use-them)
- [Nansen: Wallet Labels Guide](https://www.nansen.ai/guides/wallet-labels-emojis-what-do-they-mean)
- [Dune Analytics - Exchange Flows](https://dune.com/soispoke/Exchange-Flows)
- [Kaiko: Level 1 and Level 2 Market Data](https://www.kaiko.com/products/l1-l2-data)
- [Kaiko: Raw Order Book Snapshot API](https://docs.kaiko.com/rest-api/data-feeds/level-1-and-level-2-data/level-2-aggregations/raw-order-book-snapshot)

### 调查报道与案例
- [CoinDesk: Binance Fired Investigator Who Uncovered Manipulation at DWF Labs (WSJ)](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [CoinDesk: DWF Labs' $200M Deals Blur What 'Investing' Means](https://www.coindesk.com/business/2023/04/14/market-maker-dwf-labs-more-than-200m-in-deals-blur-what-investing-means)
- [CoinDesk: Inside Movement's Token-Dump Scandal](https://www.coindesk.com/tech/2025/04/30/inside-movement-s-token-dump-scandal-secret-contracts-shadow-advisors-and-hidden-middlemen)
- [CoinDesk: On-Chain Data Shows Close Ties Between FTX and Alameda](https://www.coindesk.com/business/2022/11/22/on-chain-data-shows-close-ties-between-ftx-and-alameda-were-there-from-the-start-nansen)
- [Nansen: Blockchain Analysis - The Collapse of Alameda and FTX](https://www.nansen.ai/research/blockchain-analysis-the-collapse-of-alameda-and-ftx)
- [Nansen: Arbitrage on Decentralised Exchanges](https://www.nansen.ai/research/arbitrage-on-decentralised-exchanges)

### 方法论与技术
- [Cryptowisser: Advanced On-Chain Analysis - Wallet Tracking & Smart Money Flows](https://www.cryptowisser.com/guides/advanced-on-chain-analysis/)
- [Elementus: Decoding the Chain - Data Science-Based Heuristics](https://www.elementus.io/blog-post/decoding-the-chain-how-data-science-based-heuristics-reveal-blockchain-networks)
- [Nansen: Transaction Clustering in Crypto](https://www.nansen.ai/post/what-is-transaction-clustering-in-crypto-address-analysis)
- [AMLBot: Wallet Clustering and Entity Identification](https://blog.amlbot.com/wallet-and-entity-identification-in-blockchain-analytics/)
- [TRM Labs: Fundamentals of Cryptocurrency Transaction Tracing](https://www.trmlabs.com/resources/blog/the-fundamentals-of-cryptocurrency-transaction-tracing)
- [BitPinas: ZachXBT's Complete Toolkit](https://bitpinas.com/learn-how-to-guides/zachxbt-complete-toolkit-top-blockchain-investigation/)
- [PANews: ZachXBT - From Victim to On-Chain Detective](https://www.panewslab.com/en/articles/83055ef5-cb39-46a9-a64f-92a59b6cd6e2)

### 做市商策略
- [DWF Labs: 4 Core Crypto Market Making Strategies](https://www.dwf-labs.com/news/4-common-strategies-that-crypto-market-makers-use)
- [Cryptowisser: Arbitrage in 2025 - DEXs, CEXs, and Cross-Chain Bridges](https://www.cryptowisser.com/guides/arbitrage-dexs-cexs-cross-chain-bridges/)
- [Amberdata: Developing and Backtesting DEX/CEX Arbitrage Strategies](https://blog.amberdata.io/developing-and-backtesting-dex-cex-arbitrage-trading-strategies)
- [Kaiko Research: Understanding Centralized Exchange Liquidity](https://research.kaiko.com/insights/centralized-exchange-liquidity)
