# 06 - 加密做市商历史案例研究

> 最后更新: 2026-04-12
> 本文档为做市商行为识别框架提供实证支撑

---

## 目录

1. [DWF Labs 争议](#1-dwf-labs-争议)
2. [Alameda Research 与 FTX 崩溃](#2-alameda-research-与-ftx-崩溃)
3. [Wintermute](#3-wintermute)
4. [Luna/UST 崩盘中的做市商行为](#4-lunaust-崩盘中的做市商行为)
5. [SEC/CFTC/DOJ 做市商执法案例](#5-seccftcdoj-做市商执法案例)
6. [Binance 做市商事件 (2025-2026)](#6-binance-做市商事件-2025-2026)
7. [关键发现与模式总结](#7-关键发现与模式总结)

---

## 1. DWF Labs 争议

### 1.1 背景

DWF Labs 自称为 Web3 投资公司和做市商，于 2022 年成立，迅速成为加密市场上最活跃的"投资+做市"一体化机构之一，声称投资了超过 200 个项目、部署超过 $200M 资金。

### 1.2 被指控的具体操纵行为

**Binance 内部调查 (2023)**:
- Binance 的市场监控团队在两家竞争做市商私下举报后，于 2023 年 9 月开始调查 DWF Labs
- 调查发现 DWF Labs 在 2023 年进行了超过 **$3 亿美元的洗售交易 (wash trades)**
- 具体操纵了 YGG (Yield Guild Games) 及至少 6 个其他代币的价格
- 发现 DWF Labs 创始人公开推广某些代币后随即抛售，导致代币价格暴跌 -- 符合经典的 **拉高出货 (pump and dump)** 模式

**代币价格表现模式**:
- YGG、DODO、C98 等代币在 DWF Labs 宣布"投资"后出现价格飙升
- 随后短期内价格急剧下跌
- 这种模式反复出现，强化了外界对其"投资公告与激进短期交易交织"的认知

### 1.3 媒体调查报道

**华尔街日报 (WSJ) - 2024 年 5 月**:
- 报道了 Binance 内部监控团队的调查结果
- 揭露了 DWF Labs 在 Binance 上从事拉高出货和洗售交易的详细证据
- 报道指出 Binance 在收到内部调查报告后一周内解雇了调查团队负责人

**CoinDesk - 2024 年 5 月 9 日**:
- 标题: "Binance Fired Investigator Who Uncovered Market Manipulation at Client DWF Labs"
- 详细描述了 Binance 如何解雇了发现 DWF Labs 市场操纵行为的内部调查员

### 1.4 Binance 的处理

- Binance **解雇了内部调查团队成员**，而非处罚 DWF Labs
- Binance 对外声称："指控未被充分证实"
- 辩称自交易 (self-trading) 可能是"意外的"
- 声称内部调查团队与 DWF Labs 的竞争对手走得太近
- **保留了 DWF Labs 作为客户**

### 1.5 DWF Labs 的辩护

- 创始合伙人 Andrei Grachev (Heng Yu Lee) 将指控称为**"竞争对手驱动的 FUD"**
- 否认所有市场操纵指控
- 声称其交易活动属于正常做市行为

### 1.6 案例意义

DWF Labs 案例揭示了加密做市商行业的结构性问题:
- **"投资+做市"模式的内在利益冲突**: 做市商以折扣价获取代币后，有强烈动机拉高价格再抛售
- **交易所监管的失败**: Binance 选择保护客户关系而非市场公正性
- **"调查者被解雇"效应**: 对内部合规团队产生寒蝉效应

---

## 2. Alameda Research 与 FTX 崩溃

### 2.1 三重角色冲突

Alameda Research 同时扮演三个相互冲突的角色:

| 角色 | 描述 | 利益冲突 |
|------|------|----------|
| **做市商** | FTX 的主要做市商，为交易所提供流动性 | 可获得优先执行、费率优惠 |
| **对冲基金/交易公司** | 运用套利、做市、yield farming、波动率交易等策略 | 利用做市信息进行自营交易 |
| **交易所关联方** | SBF 同时控制 FTX 和 Alameda | 可接触客户资金、内部数据 |

在传统证券市场中，做市商与交易所之间必须维持明确的法律分离，以防止固有的利益冲突。Alameda 与 FTX 的"乱伦关系"彻底违背了这一原则。

### 2.2 做市行为如何导致 FTX 崩溃

**资金挪用链条**:
1. Alameda 作为 FTX 最大的做市商/交易商，获得特殊待遇
2. 2022 年 5-6 月，Alameda 遭受一系列重大交易亏损
3. FTX 将**超过 $100 亿客户资金**借给 Alameda 填补亏损
4. Alameda 使用 FTX 原生代币 FTT 作为借款抵押品
5. FTT 价格在一天内暴跌 75%，抵押品变得不足以覆盖借款
6. 引发客户挤兑，FTX 在 2022 年 11 月崩溃

**FTT 代币的操纵**:
- 初始 3.5 亿 FTT 供应中，约 2700 万枚最终进入 Alameda 的 FTX 存款钱包
- FTX 和 Alameda **合计控制了 86% 的 FTT 供应**
- 极少的 FTT 在公开市场流通，使代币极易被操纵
- 只要 FTT 价格维持高位且客户不大规模提款，这一模式就能持续运转

**前置交易模式**:
- 2021 年初至 2022 年 3 月期间，Alameda 在以太坊链上提前积累了价值约 $6000 万的代币 -- 这些代币随后被 FTX 上线交易
- 这构成典型的**内幕交易 (insider trading)** 行为

### 2.3 链上可追踪的做市行为模式

- **"Alameda Gap"**: 2022 年 11 月 FTX 崩溃后，加密市场流动性出现显著下降，被 Kaiko 称为"Alameda Gap"
- Alameda 的已知钱包在 2022 年 6-7 月期间是 FTX 最大的稳定币存入方:
  - 占 Tether 转账的 **10%**
  - 占 USDC 转账的 **30%**
- Nansen 的链上分析清晰追踪了 Alameda 与 FTX 之间的资金流向

### 2.4 案例意义

- **做市商-交易所关联方模式**的终极反面教材
- 证明了链上分析在追踪做市商行为方面的有效性
- "Alameda Gap"证明单一大型做市商退出可导致全市场流动性危机

---

## 3. Wintermute

### 3.1 2022 年 $160M 被黑事件

**事件经过 (2022 年 9 月 20 日)**:
- Wintermute 的 DeFi 操作钱包遭黑客攻击，损失约 **$1.6 亿**
- 攻击原因: 热钱包使用了 Profanity 生成的虚荣地址 (vanity address)，存在安全缺陷
- 攻击者通过暴力破解 Profanity 地址的可能种子值，重建了私钥
- OTC 服务和借贷业务未受影响

**被黑后的库存管理和市场影响**:
- CEO Evgeny Gaevoy 声称公司仍然有偿付能力，剩余权益是损失金额的"两倍以上"
- 被黑的 90 种资产中，只有 2 种名义价值超过 $100 万（且无一超过 $250 万）
- 被黑时 Wintermute 有约 **$2 亿的未偿 DeFi 借款**
- Gaevoy 曾将此事件视为"白帽"事件，呼吁黑客联系协商

**市场影响**:
- 由于 Wintermute 为多个交易所和 DeFi 协议提供流动性，其被黑引发了对更广泛流动性收缩的担忧
- 但由于损失分散在多种资产中（无单一大额暴露），未引发系统性恐慌

### 3.2 Wintermute 的公开做市哲学

**Evgeny Gaevoy 访谈要点**:

- **技术公司定位**: Wintermute 将自身定义为"技术公司"而非金融服务或交易公司，以便专注于构建长期能力
- **全域流动性**: 目标是成为 CeFi 和 DeFi 世界中所有流动性池（包括屏幕上、OTC、现货和衍生品）的头号算法流动性提供商
- **多策略整合**: 成功的做市策略需要在内部运行多种策略和信号的组合
- **规模数据**: 日交易量超过 $50 亿，为 50 多个交易场所提供深度流动性
- **2024 年业绩**: 累计交易量达 $3 万亿，OTC 业务增长 313%

### 3.3 案例意义

- Wintermute 代表了"合规做市商"的典型形象，但其被黑事件暴露了做市商集中持有大量资产的风险
- 做市商在 DeFi 中的安全实践直接影响整个市场的稳定性
- 即使是"正规"做市商，其运营透明度仍然有限

---

## 4. Luna/UST 崩盘中的做市商行为

### 4.1 崩盘时间线与做市商角色

**前兆 -- Jump Trading 的隐秘干预 (2021 年 5 月)**:
- Jump Crypto（通过子公司 Tai Mo Shan）秘密介入，花费数千万美元防御 UST 的 $1 锚定
- 作为回报获得大幅折扣的 LUNA 代币
- 这笔 Terra 交易最终为 Jump 带来约 **$12.8 亿的利润**

**崩盘触发 (2022 年 5 月 7 日)**:
1. Terraform Labs 从 Curve-3CRV 池转移 1.5 亿 UST 到新建的 Curve-4pool，大幅减少原始池的流动性
2. 匿名交易者利用流动性失衡，执行了 8500 万 UST 换 USDC 的交易
3. UST 首次脱锚，跌破 $0.99

**做市商的撤离**:
- 大量 UST 从 Curve 和 Anchor 被提取
- LFG (Luna Foundation Guard) 将价值约 $24 亿的比特币储备"借给"专业做市商试图挽救
- 但做市商介入已无法阻止螺旋式下跌

### 4.2 撤离后的流动性真空效应

- 5 月 9 日，UST 第二次也是最后一次脱锚
- LUNA 从 $87 跌至不到 $0.00005 (99.99% 跌幅)
- UST 从 $1 跌至 $0.2
- 整个过程仅用了约 **8 天** (5 月 5 日至 5 月 13 日)
- 流动性提供者的撤离形成了**正反馈死亡螺旋**: 做市商撤离 -> 流动性下降 -> 滑点增大 -> 更多恐慌抛售 -> 更多做市商撤离

### 4.3 Jump Trading / Tai Mo Shan 的后果

- 2024 年 12 月 20 日，SEC 指控 Tai Mo Shan (Jump Trading 子公司) 误导投资者关于 TerraUSD 的稳定性
- 通过购买 $2000 万 UST 维持锚定，却未披露其经济利益
- **罚款总额: $1.23 亿** ($8600 万利润返还 + $3600 万民事罚款)
- CFTC 也在调查 Jump Crypto 的加密交易和投资活动（截至 2025 年中，调查仍在进行）
- Jump Crypto 总裁 Kanav Kariya 于 2024 年 6 月在 CFTC 调查中辞职

### 4.4 Citadel 阴谋论 -- 被否认

- 社交媒体流传阴谋论: BlackRock 和 Citadel Securities 从 Gemini 借入 100,000 BTC，将 25% 换成 UST 做空
- Citadel Securities 董事总经理 David Millar 明确否认: "Citadel Securities 不交易稳定币，包括 UST"
- **此阴谋论未获证实**

### 4.5 案例意义

- 做市商在危机中的**有义务vs无义务**问题: 传统做市商有义务在波动市场中维持报价，加密做市商则无此义务
- Jump Trading 案揭示了做市商如何通过"救助"获取巨额利润，同时为后续崩盘埋下隐患
- 流动性真空效应的速度和烈度远超传统市场

---

## 5. SEC/CFTC/DOJ 做市商执法案例

### 5.1 Operation Token Mirrors -- FBI 卧底行动 (2024 年 10 月)

**这是加密做市商执法史上最重要的案例之一。**

**行动概述**:
- FBI 采取"前所未有的步骤"，创建了一个虚假加密代币 **NexFundAI** 和一家虚假公司
- 行动代号: **"Operation Token Mirrors"**
- 目的: 引诱涉嫌市场操纵的做市商上钩

**被起诉的做市商公司**:
| 公司 | 指控 | 状态 |
|------|------|------|
| **Gotbit** | 洗售交易、市场操纵 | 认罪，没收约 $2300 万加密货币 |
| **ZM Quant** | 自交易 (洗售交易) | SEC 起诉 |
| **CLS Global** | 自交易 (洗售交易) | SEC 起诉 (被 FBI 假代币钓鱼) |
| **Vortex** | 欺诈、人为抬高代币价格和交易量 | 起诉 (2025 年 8 月) |
| **Contrarian** | 欺诈、协调交易 | 起诉 (2025 年 9 月) |
| **Antier** | 欺诈、协调交易 | 起诉 (2025 年 9 月) |

**被起诉个人**: 共 17 人涉及加密犯罪，10 名做市商高管和员工被正式起诉

### 5.2 SEC 的具体指控内容

**2024 年 10 月 9 日 SEC 公告 (Press Release 2024-166)**:

被指控方使用**算法和机器人**进行洗售交易，有时每天生成:
- **数千万亿笔交易 (quadrillions of transactions)**
- **数十亿美元的虚假交易量**

目的是制造活跃交易市场的**虚假表象**，诱导散户投资者购买代币。

**服务模式 -- "市场操纵即服务 (Market-Manipulation-as-a-Service)"**:
- 代币发行方 (如 Russell Armand, Maxwell Hernandez 等) **雇佣** ZM Quant 和 Gotbit 提供操纵服务
- 服务内容包括: 人为制造交易量、操纵代币价格
- 代币被作为未注册证券向散户销售

### 5.3 证据类型

执法机构使用的证据类型包括:
- **FBI 卧底操作**: 创建虚假代币，记录做市商如何操纵其交易量
- **链上交易数据**: 追踪洗售交易的自交易模式
- **通信记录**: 做市商与项目方之间的操纵协议
- **算法/机器人代码**: 证明操纵行为是系统性的、有预谋的

### 5.4 CFTC 执法趋势

- 2024 财年: CFTC 提起 58 项新执法行动，追回创纪录的 **$171 亿** 货币救济
- 2025 财年: 在代理主席 Caroline D. Pham 领导下，CFTC 采取"回归基础"方针，关闭约一半未决执法事项
- CFTC 正在调查 Jump Trading 的加密业务（调查仍在进行中）

### 5.5 对做市商行业的影响

- **首次刑事指控**: Operation Token Mirrors 标志着美国首次对加密做市商提起刑事指控
- **"操纵即服务"模式被定性为犯罪**: 明确了为客户提供洗售交易服务构成联邦犯罪
- **引渡先例**: 多名被告从海外被引渡至美国受审
- **行业震慑效应**: 小型做市商开始收敛操纵行为，但大型做市商面临的实际风险仍然有限

---

## 6. Binance 做市商事件 (2025-2026)

### 6.1 MOVE 代币丑闻 (2025 年 3 月)

**事件经过**:
- Movement Labs 被骗签署了一份做市协议，将 **6600 万 MOVE 代币** 的控制权交给了一个名为 Rentech 的中间商
- Rentech 在协议中同时以 Web3Port 子公司和 Movement Foundation 代理人的身份出现 -- 存在自我交易嫌疑
- 代币上线第二天 (2024 年 12 月 9 日后)，6600 万 MOVE 被抛售，套现约 **$3800 万**
- 触发 MOVE 价格暴跌

**Binance 的处理**:
- 封禁了该做市商账户，理由为"不当行为"
- Movement Labs 宣布代币回购计划
- Coinbase 随后也下架了 MOVE 代币

**核心问题**:
- Movement Foundation 官员最初就将 Rentech 协议标记为"可能是他们见过的最糟糕的协议"
- 专家警告该协议创造了"先拉高 MOVE 价格然后向散户倾销代币"的激励结构

### 6.2 GPS 代币事件 (2025 年)

- Binance 冻结了 GPS 代币的做市商账户，原因是发现**大规模代币倾销** ($500 万)
- 事件牵连到做市商 Web3Port
- 进一步调查发现 Web3Port 服务的项目不仅限于 GPS -- 还包括 SHELL、Aethir、dappOS、Movement、Puffer 等

### 6.3 SIREN 代币事件 (2026 年 3 月)

**操纵模式**:
1. SIREN 于 2025 年初发行后被遗忘
2. 2026 年 2 月起，一个神秘钱包集群开始大规模买入
3. 价格从约 $0.08 飙升至 $3.61 (涨幅超过 **45 倍**)
4. 市值一度超过 **$22 亿**，短暂进入全球前 30

**暴露**:
- Bubblemaps 发出警告: 超过 200 个地址组成的集群持有约 50% 的流通供应量 (价值约 $15 亿)
- 链上分析师 EmberCN 深挖: 前 54 个最大持仓地址中，**52 个属于同一实体**
- 72 小时内 SIREN 从峰值暴跌 **71%**，市值从 $22 亿缩水至 $7.4 亿

### 6.4 Binance 2026 年 3 月新规

**Bloomberg 报道 (2026 年 3 月 25 日)**:

Binance 发布新的做市商规则:
- **禁止项目方与做市商之间的收入分成模式**
- **禁止保底回报安排**
- **要求做市商披露身份、法律实体和合同条款**
- **要求项目方披露其做市商合作伙伴**
- Binance 保留在发现不当行为时将做市商列入黑名单的权利

**"六大红旗"指南 (2026 年 3 月 25 日)**:
Binance 发布名为"加密做市商红旗指南"的博客文章，实质含义: "我知道你们在做什么，我保留随时处理你们的权利。"

**行业反应**:
- 批评者质疑这是"透明度推进还是损害控制"
- 2026 年 4 月，Bloomberg 报道多名 Binance 高级合规人员（金融犯罪和监控岗位）离职

---

## 7. 关键发现与模式总结

### 7.1 做市商操纵行为的共同模式

| 模式 | 案例 | 识别特征 |
|------|------|----------|
| **洗售交易** | DWF Labs, Gotbit, ZM Quant, CLS Global | 同一实体控制的地址之间的自交易; 交易量与独立地址数不匹配 |
| **拉高出货** | DWF Labs, SIREN, MOVE | 投资/合作公告后价格飙升，随后创始人/关联方抛售 |
| **"操纵即服务"** | Gotbit, ZM Quant | 做市商为项目方提供付费操纵服务，使用算法机器人 |
| **做市商-交易所关联方** | Alameda/FTX | 做市商获得特殊待遇、内部信息、客户资金 |
| **流动性假象** | DWF Labs, Gotbit | 做市商创造虚假流动性深度，当真实卖压出现时流动性消失 |
| **做市商撤离引发崩盘** | Luna/UST, Alameda Gap | 做市商在危机中无义务继续做市，撤离引发流动性真空 |
| **零成本代币倾销** | MOVE (Web3Port), GPS | 做市商以零成本或极低成本获取代币，以"做市"为名抛售获利 |

### 7.2 监管执法的演进

| 时间 | 里程碑 |
|------|--------|
| 2022.11 | FTX/Alameda 崩溃，暴露做市商-交易所关联方模式 |
| 2022.05 | Luna/UST 崩盘，暴露做市商在危机中的撤离问题 |
| 2024.05 | WSJ 报道 DWF Labs 操纵行为，Binance 解雇内部调查员 |
| 2024.10 | **Operation Token Mirrors**: FBI 卧底行动，首次刑事起诉加密做市商 |
| 2024.12 | SEC 对 Jump Trading/Tai Mo Shan 罚款 $1.23 亿 |
| 2025.03 | Binance 封禁 MOVE 代币做市商，处罚 Web3Port |
| 2025-2026 | DOJ 陆续起诉 Gotbit, Vortex, Contrarian, Antier 高管 |
| 2026.03 | Binance 发布做市商新规，要求披露和禁止利益冲突安排 |

### 7.3 对做市商行为识别框架的启示

**链上识别信号**:
1. **地址集群分析**: SIREN 案中 52/54 个顶级地址属于同一实体 -- 可用 Bubblemaps 等工具检测
2. **洗售交易检测**: 同一实体控制的地址之间高频交易 -- 需要地址归属分析
3. **代币集中度**: FTT 86% 由两个关联实体控制 -- 极端集中度是操纵前提
4. **做市商钱包的资金流**: Alameda 的稳定币转账占 FTX 流入的 10-30% -- 异常集中
5. **上线前积累**: Alameda 在 FTX 上线前购入 $6000 万代币 -- 前置交易信号

**行为模式信号**:
1. **"投资公告"后价格异动**: DWF Labs 模式 -- 公告后暴涨然后暴跌
2. **做市协议中的利益冲突条款**: 收入分成、保底回报、零成本代币 -- MOVE 案的教训
3. **交易量与价格的背离**: 虚假交易量可能与真实价格发现不一致
4. **危机中做市商的行为**: 观察做市商是否在压力下撤离流动性

---

## 参考来源

### DWF Labs
- [DWF Labs - Wikipedia](https://en.wikipedia.org/wiki/DWF_Labs)
- [Binance Fired Investigator Who Uncovered Market Manipulation at Client DWF Labs: WSJ - CoinDesk](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [DWF Labs calls market manipulation claims 'competitor-driven FUD' - DL News](https://www.dlnews.com/articles/people-culture/dwf-denies-market-manipulation-wash-trading-on-binance/)
- [DWF Labs denies report that it did $300 million of wash trading on Binance last year - The Block](https://www.theblock.co/post/293429/dwf-labs-denies-report-that-it-did-300-million-of-wash-trading-on-binance-last-year)
- [Crypto Market Maker DWF Labs' More Than $200M in Deals Blur What 'Investing' Means - CoinDesk](https://www.coindesk.com/business/2023/04/14/market-maker-dwf-labs-more-than-200m-in-deals-blur-what-investing-means)
- [The DWF Labs Market Strategy: Pump and Dump of Crypto Tokens - Metaverse Post](https://mpost.io/the-dwf-labs-market-strategy-pump-and-dump-of-crypto-tokens/)

### Alameda Research / FTX
- [Alameda Research - Wikipedia](https://en.wikipedia.org/wiki/Alameda_Research)
- [The Alameda gap and crypto liquidity crisis explained - Cointelegraph](https://cointelegraph.com/explained/the-alameda-gap-and-crypto-liquidity-crisis-explained)
- [Alameda Research's market manipulation and insider trading took down FTX - Crypto.news](https://crypto.news/alameda-researchs-market-manipulation-and-insider-trading-took-down-ftx/)
- [Blockchain Analysis: The Collapse of Alameda and FTX - Nansen](https://www.nansen.ai/research/blockchain-analysis-the-collapse-of-alameda-and-ftx)
- [FTX and Alameda likely colluded from the very beginning - Cointelegraph](https://cointelegraph.com/news/ftx-and-alameda-likely-colluded-from-the-very-beginning-report)
- [Sam Bankman-Fried's Alameda quietly used FTX customer funds for trading - CNBC](https://www.cnbc.com/2022/11/13/sam-bankman-frieds-alameda-quietly-used-ftx-customer-funds-without-raising-alarm-bells-say-sources.html)

### Wintermute
- [Crypto Market Maker Wintermute Hacked for $160M - CoinDesk](https://www.coindesk.com/business/2022/09/20/crypto-market-maker-wintermute-hacked-for-160m-says-ceo)
- [Hacked Crypto Market Maker Wintermute Has $200M in Outstanding DeFi Debt - CoinDesk](https://www.coindesk.com/business/2022/09/20/hacked-crypto-market-maker-wintermute-has-200m-in-outstanding-defi-debt)
- [Explained: The Wintermute Hack - Halborn](https://www.halborn.com/blog/post/explained-the-wintermute-hack-september-2022)
- [Wintermute CEO Evgeny Gaevoy Discusses the Future of Crypto Trading - Yahoo Finance](https://finance.yahoo.com/news/wintermute-ceo-evgeny-gaevoy-discusses-165121751.html)
- [Evgeny Gaevoy: Wintermute's unique liquidity model - Unchained/CryptoBriefing](https://cryptobriefing.com/evgeny-gaevoy-wintermutes-unique-liquidity-model-drives-market-making-retail-investors-struggle-with-late-entries-and-the-rise-of-derivatives-enhances-institutional-safety-unchained/)

### Luna/UST 崩盘
- [LUNA/UST debacle and the role of liquidity providers - Flovtec](https://www.flovtec.com/post/luna-ust-debacle-and-the-role-of-liquidity-providers)
- [Anatomy of a Run: The Terra Luna Crash - Harvard Law](https://corpgov.law.harvard.edu/2023/05/22/anatomy-of-a-run-the-terra-luna-crash/)
- [Inside Jump Crypto: $1.3B Terra Trade, $321M Wormhole Rescue & More](https://insights4vc.substack.com/p/inside-jump-crypto-13b-terra-trade)
- [Jump Crypto Subsidiary Tai Mo Shan to Pay $123 Million to Settle SEC Charges - Yahoo Finance](https://finance.yahoo.com/news/jump-crypto-subsidiary-tai-mo-060333984.html)
- [Tai Mo Shan to Pay $123 Million - SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-212)

### SEC/DOJ 执法
- [SEC Charges Three So-Called Market Makers and Nine Individuals - SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-166)
- [The FBI created its own crypto token in an 'unprecedented' operation - Fortune](https://fortune.com/crypto/2024/10/09/fbi-doj-crypto-market-manipulation-fraud-charges/)
- [FBI creates bogus crypto to nab 4 companies, 14 people on fraud charges - The Block](https://www.theblock.co/post/320347/us-government-charges-four-crypto-companies-and-14-people-for-fraud-reuters)
- [DOJ Brings 10 Alleged Crypto Market Manipulators to Oakland Court - Cointelegraph](https://cointelegraph.com/news/us-brings-10-alleged-crypto-market-manipulators-oakland-court)
- [DOJ Crypto Enforcement: Key Cases and 2025 Predictions - Dynamis LLP](https://www.dynamisllp.com/white-collar-defense-crypto-criminal-regulatory)

### Binance 做市商事件 (2025-2026)
- [Binance Tightens Market Making Rules In Wake of Crash Criticism - Bloomberg](https://www.bloomberg.com/news/articles/2026-03-25/binance-tightens-market-making-rules-in-wake-of-crash-criticism)
- [Binance tightens market maker rules and warns token issuers to disclose partners - CoinDesk](https://www.coindesk.com/business/2026/03/25/binance-tightens-market-maker-rules-tells-token-issuers-they-must-disclose-partners)
- [Inside Movement's Token-Dump Scandal - CoinDesk](https://www.coindesk.com/tech/2025/04/30/inside-movement-s-token-dump-scandal-secret-contracts-shadow-advisors-and-hidden-middlemen)
- [Binance Offboards Market Maker for Movement's MOVE Token - CoinDesk](https://www.coindesk.com/markets/2025/03/25/binance-offboards-market-maker-that-it-said-made-usd38m-profit-on-move-listing)
- [Binance Freezes GPS Market Maker Over $5M Token Dump - Coin360](https://coin360.com/news/binance-freezes-gps-market-maker)
- [Siren Token Plummets 84% - BitcoinWorld](https://bitcoinworld.co.in/siren-token-crash-analysis/)
- [Transparency Push or Damage Control? - Disruption Banking](https://www.disruptionbanking.com/2026/03/31/transparency-push-or-damage-control-binance-tightens-rules-after-manipulation-concerns/)
