# Web3项目融资与估值尽调框架

> 调研日期：2026-04-12
> 覆盖周期：2024-2026年加密市场
> 用途：Web3项目尽调中的融资和估值维度评估

---

## 一、融资轮次评估——典型估值范围（2024-2026）

### 1.1 各轮次融资规模基准

| 融资阶段 | 平均融资额（2024） | 趋势变化 | 典型估值范围 |
|----------|-------------------|----------|-------------|
| Pre-seed | $2.5M | YoY +28% | $5M-$15M |
| Seed | $5.6M | YoY +12%（从$5M增长） | $15M-$40M |
| Series A | $17.6M | YoY -3%（从$18.1M下降） | $50M-$150M |
| Series B | $40M-$50M | 案例：Blackbird Labs $50M（a16z等领投） | $150M-$500M |

### 1.2 市场总体趋势

- **2024年**：全年约$6.9B流入Web3，共827轮融资；Q3单季超$2B（300笔交易），YoY +43%
- **2025年**：全年$5.74B，仅379轮——**单笔更大、总量更少**，反映VC更挑剔
- **2025年广义统计**（含代币融资/Treasury等）：约$49.75B流入区块链项目，YoY +433%（从2024年$9.31B），但传统VC口径为$18.9B（+44% YoY），交易数从2,900+降至约1,200笔
- **2026展望**：多数投资者预期早期融资小幅改善，但远低于2021-2022牛市水平

### 1.3 热门赛道融资分布（2025）

| 赛道 | 融资占比/金额 | 说明 |
|------|-------------|------|
| DeFi | 30.4%份额 | 最大单一赛道 |
| AI + Crypto | $3.5B / 40%流入 | 282个项目获投；a16z、Polychain、Coinbase重仓 |
| 基础设施 | $2.2B | L1/L2、跨链、中间件 |
| DePIN | $744M+（至2025.07） | 165+项目；去中心化物理基础设施 |
| RWA | 快速增长 | Pantera预测2026翻倍 |

### 1.4 估值红旗

- Seed轮估值超过$50M但无产品/用户——过度炒作
- Series A估值与Seed轮差距<2x——增长故事不成立
- 估值大幅依赖"叙事溢价"（如纯AI概念、纯meme）而非链上数据

---

## 二、投资方质量判断——VC信号分析

### 2.1 Tier 1 顶级基金画像

| 基金 | AUM/基金规模 | 核心能力 | 信号强度 |
|------|-------------|---------|---------|
| **a16z Crypto** | $7.6B+ AUM | 零售ROI 60,704%；深度投后支持（法务、招聘、GTM） | 极强 |
| **Paradigm** | $850M（第三期基金） | 内部研究组40+技术论文/年；工程团队验证协议性能 | 极强 |
| **Polychain Capital** | $200M+（第四期基金首关） | 分析70+链的安全假设、验证者经济学 | 强 |
| **Pantera Capital** | 多只基金累计数十亿 | 4家组合公司2025年上市，合计市值$33B | 强 |
| **Galaxy Digital** | 大型综合平台 | 研究+交易+资管全链条 | 强 |

**关键数据**：顶级5家VC（a16z、Paradigm、Pantera、Galaxy、Sequoia）占据约40%最高估值轮次。

### 2.2 VC质量评估框架

| 评估维度 | Tier 1 基金 | 不知名/小基金 |
|----------|-----------|-------------|
| 尽调深度 | 数月技术审计+经济模型验证 | 可能仅看叙事和团队背景 |
| 投后支持 | 法务、合规、招聘、交易所上币 | 基本无 |
| 品牌背书 | 自带后续融资引力 | 无引力效应 |
| 退出纪律 | 长期持有+锁仓承诺 | 可能TGE后快速OTC退出 |
| 技术参与 | 代码审计、协议设计参与 | 纯财务投资 |
| 信息价值 | 公开研究报告影响市场 | 无市场影响力 |

### 2.3 警示信号

- 项目仅有不知名基金参与，但宣称高估值——可能是关联方虚假融资
- VC列表中出现已倒闭或声誉不佳的基金（如Three Arrows Capital关联方）
- 大量"战略投资者"但无财务VC——可能是做市商/交易所的包装
- 同一VC在同赛道投资过多竞品——分散风险但信号减弱

---

## 三、估值合理性——核心指标方法论

### 3.1 关键估值指标

#### FDV（完全稀释估值）

```
FDV = 当前代币价格 x 最大代币供应量
```

- **用途**：评估项目"终态"市值，识别通胀风险
- **陷阱**：大量代币未解锁时，FDV严重高估当前真实市值
- **基准**：若流通率<20%而FDV已达数十亿，需高度警惕

#### MC/TVL（市值/锁仓价值比）

```
MC/TVL = 流通市值 / TVL
```

- 类似传统金融的P/B（市净率）
- **MC/TVL < 1**：可能低估（每$1锁仓价值市场仅定价<$1）
- **MC/TVL > 5**：可能高估或TVL质量差（循环质押、激励驱动）

#### FDV/TVL

```
FDV/TVL = 完全稀释估值 / TVL
```

- 衡量"每$1锁仓资金的市场溢价"
- **FDV/TVL > 10**：投机驱动成分大
- **FDV/TVL < 3**：相对合理（需结合协议类型判断）

#### P/S（价格/销售比）

```
P/S = 市值 / 年化协议收入
```

- Token Terminal标准化定义：Revenue = 协议保留的费用（非全部用户费用）
- **DeFi协议**：P/S 10-30x为合理区间
- **L1/L2**：P/S 50-200x常见（平台属性溢价）
- **P/S > 500x**：除非极早期高增长，否则红旗

#### P/E（价格/收益比）

```
P/E = 市值 / 年化净收入
```

- 仅适用于有明确收入分配机制的协议
- 多数DeFi协议2025年P/E在50-150x之间

#### P/F（价格/费用比）

```
P/F = 市值 / 年化总费用
```

- Fees = 用户支付的全部费用（含LP分成、国库收入等）
- 比P/S更宽口径，适合比较不同费用分配模型的协议

### 3.2 Messari Disruption Factor（DF）框架

Messari在2025-2026年提出Disruption Factor（破坏性因子）框架，承认传统P/E、P/S等指标无法完全适用于加密项目：

- **核心理念**：衡量加密项目多深地嵌入现实世界和主流用户行为
- **四大原则**：透明性、可定制性、长期主义、开源演进
- **评估维度**：不仅看链上活跃度，更看是否有效替代传统系统、吸引非加密原生用户

### 3.3 估值对比快速参考（2025-2026）

| 指标 | 低估信号 | 合理范围 | 高估信号 |
|------|---------|---------|---------|
| MC/TVL | < 0.5 | 0.5 - 3 | > 5 |
| FDV/TVL | < 2 | 2 - 8 | > 15 |
| P/S (DeFi) | < 10 | 10 - 50 | > 100 |
| P/S (L1/L2) | < 30 | 30 - 150 | > 300 |
| 流通率 | > 50% | 20%-50% | < 10% |

---

## 四、做市协议——结构、影响与操控识别

### 4.1 三大做市模型

#### 模型一：Token借贷 + 看涨期权（Loan & Call Option）

- **结构**：项目方将代币借给做市商，做市商提供流动性；做市商获得在预定行权价购买部分借出代币的看涨期权
- **合同期限**：通常1-2年
- **风险**：做市商有动机先拉高价格再行权，或在合约到期时大量抛售归还已贬值的代币
- **典型条款**：借贷5%-10%流通供应量，行权价为上市价的100%-150%

#### 模型二：做市即服务/月费制（MMaaS / Retainer）

- **结构**：项目方提供代币和稳定币作为交易库存，按月支付固定费用
- **合同到期**：做市商归还全部借贷资产
- **优势**：利益冲突较小
- **费用**：月费$10K-$50K不等

#### 模型三：固定费用 / 绩效分成

- **Flat fee**：项目方自己提供资金，做市商收取固定年费
- **Performance fee**：做市商用项目方资金交易，盈利时收取分成（通常15%-30%）

### 4.2 主要做市商画像

| 做市商 | 特点 | 争议/风险 |
|--------|------|----------|
| **Wintermute** | 算法交易+流动性提供 | 2023年被指控与Celsius联合操控CEL代币；拒绝签署2025年市场诚信联盟框架 |
| **DWF Labs** | 2025年扩展美国市场 | 创始人被指关联OneCoin骗局；否认操纵市场指控 |
| **GSR Markets** | 机构级做市 | 相对低调 |
| **Jump Crypto** | 传统金融出身 | 2023年后大幅收缩加密业务 |
| **Gotbit/CLS Global** | 小型做市商 | 2025年被SEC/DOJ起诉虚假交易和操纵 |

### 4.3 如何识别做市商操控

| 操控信号 | 识别方法 |
|----------|---------|
| **Wash Trading**（对敲交易） | 大量成交但价差不变；买卖双方地址关联 |
| **Pump and Dump** | 短时间内价格暴涨+巨量，随后崩盘 |
| **合约到期抛售** | 做市合同到期日前后出现异常抛压 |
| **流动性假象** | 订单簿深度看似充足，但撤单率极高（spoofing） |
| **Movement Labs式丑闻** | 合同条款包含激励做市商人为拉高价格的条款 |

### 4.4 重大执法案例（2025-2026）

- **DOJ起诉10人**（2026.03）：涉及Gotbit、Vortex、Contrarian、Antier Solutions，罪名为Wash Trading和价格操纵
- **SEC起诉4家做市商**：ZM Quant、Gotbit、CLS Global、MyTrade——制造虚假交易量
- **Movement Labs丑闻**（2025.04）：秘密合同授权做市商大量抛售MOVE代币
- **Cointelegraph调查**：做市商贷款模型被描述为"正在悄悄杀死加密项目"

---

## 五、OTC交易与二级市场转让

### 5.1 OTC市场规模与增长

- **2024年**：加密OTC市场年增长率106%
- **2025年**：市场规模再次翻倍
- **驱动力**：稳定币需求激增+crypto-to-crypto交易增长+机构入场

### 5.2 VC代币折扣结构

| 代币类型 | OTC折扣率（vs交易所价格） | 锁仓条件 |
|----------|------------------------|---------|
| Top 20代币 | ~50%折扣 | 1年锁仓期 |
| Top 20-100代币 | 50%-65%折扣 | 1年锁仓+1-2年线性释放 |
| 100名以外代币 | 最高70%折扣 | 1年锁仓+2年月度释放 |
| 资金费率套利策略用 | 60%-65%折扣 | Delta-neutral对冲 |

**具体示例**：交易所价$1的代币，在STIX等OTC平台可能以$0.30成交，附带1年锁仓+2年月度释放。

### 5.3 SAFT/SAFE合同转让

- **TGE前**：从早期投资者手中购买SAFT/SAFE合同，通常**按面值或溢价25%-30%**定价
- **TGE后**：根据代币市场价折价交易，折扣取决于剩余锁仓期和项目质量

### 5.4 OTC市场参与者

| 角色 | 动机 |
|------|------|
| **卖方：VC/基金** | 锁仓期内提前获利了结、管理DPI/回报 |
| **卖方：项目方/基金会** | 运营资金需求、国库管理 |
| **买方：Hodler** | 相信长期价值，愿意接受锁仓换取大幅折扣 |
| **买方：套利者** | 利用资金费率进行Delta-neutral套利 |

### 5.5 OTC交易风险与尽调要点

- 确认代币锁仓/释放时间表是否可在链上验证
- 评估卖方身份——VC大量OTC出售可能是看空信号
- 核实SAFT合同的法律效力和司法管辖
- 关注做市商是否成为OTC交易的"退出流动性"
- 2025年趋势：VC影响力下降，公平发行（Fair Launch）模式兴起

---

## 六、历史案例——估值脱离基本面

### 6.1 ICP（Internet Computer Protocol）—— FDV虚高典型

- **上市时间**：2021年5月
- **上市FDV**：>$120B
- **流通率**：<25%
- **问题**：未经验证的协议，几乎无用户，FDV直接对标以太坊
- **结果**：价格从$700跌至<$5（-99.3%）
- **教训**：低流通+高FDV = 估值幻觉

### 6.2 LUNA/UST（Terra）—— 机制缺陷下的估值泡沫

- **巅峰FDV**：>$40B（2022年初）
- **核心问题**：Anchor协议用20% APY吸引75%的UST供应量，收益不可持续
- **崩盘过程**：2022年5月，3天蒸发$50B
- **教训**：高APY激励驱动的TVL不等于真实需求；算法稳定币机制脆弱

### 6.3 FTT（FTX Token）—— 自我引用的估值循环

- **核心问题**：Alameda Research资产负债表中持有$6B的FTT，用作抵押品获取贷款
- **估值脱离**：FTT没有独立价值支撑，完全依赖FTX生态内部循环
- **结果**：FTX破产后FTT从$25跌至<$1
- **教训**：代币作为抵押品的"价值"是自我引用的——当信心崩塌，价值归零

### 6.4 更多警示模式

| 项目/模式 | 估值脱离特征 | 结果 |
|----------|------------|------|
| 2021年高FDV低流通项目群 | 流通率<10%，FDV数十亿 | 大多跌90%+ |
| Axie Infinity式GameFi | TVL/估值基于代币激励而非真实经济 | Play-to-Earn经济崩塌 |
| 2024-2025 Meme币 | 零基本面，纯社区/叙事驱动 | 极端波动，多数归零 |
| 高估AI+Crypto概念项目 | 无实际AI产品，仅包装叙事 | 泡沫风险极高 |

---

## 七、评估框架总结——融资与估值尽调清单

### 7.1 融资尽调

- [ ] 融资轮次和估值是否与同阶段项目可比？
- [ ] 领投方是否为Tier 1基金？是否有真实投入而非顾问代币？
- [ ] 融资时间线是否合理？（如6个月内从Seed到B轮——异常）
- [ ] 代币经济学中VC/团队分配比例是否过高？（>30%警示）
- [ ] 有无关联方虚假融资的迹象？

### 7.2 估值尽调

- [ ] FDV vs 流通市值差距是否过大？（流通率<15%需重点关注）
- [ ] P/S、P/E是否在赛道合理范围内？
- [ ] TVL是否由真实需求驱动而非激励/循环质押？
- [ ] 估值增长是否有链上数据（活跃用户、交易量、收入）支撑？
- [ ] 与同赛道竞品的估值倍数对比是否合理？

### 7.3 做市与流动性尽调

- [ ] 项目是否披露了做市商合作关系？
- [ ] 做市协议采用什么模型？（Loan+Option需额外警惕）
- [ ] 是否存在异常交易量/价格模式暗示操纵？
- [ ] 代币上币后的流动性深度是否真实？

### 7.4 OTC/退出尽调

- [ ] VC锁仓/释放时间表是否透明且链上可验证？
- [ ] 是否有大量OTC折价交易信号（VC批量出售）？
- [ ] 代币解锁时间表是否会造成集中抛压？
- [ ] 项目方/基金会是否在OTC市场大量出售？

---

## 参考来源

### 融资数据与市场趋势
- [2024 in Review: Fundraising in Web3 - Outlier Ventures](https://outlierventures.io/article/2024-in-review-fundraising-in-web3/)
- [Crypto VC State 2026: Where $49.75 Billion Flowed - BlockEden](https://blockeden.xyz/blog/2026/01/13/crypto-vc-state-2026-smart-money-narratives/)
- [Top crypto VCs share 2026 funding outlook - The Block](https://www.theblock.co/post/384209/top-crypto-vcs-share-2026-funding-and-token-sales-outlook)
- [Web 3.0 - 2026 Market & Investments Trends - Tracxn](https://tracxn.com/d/sectors/web-3.0/__62L1Vk6M1bK6B3fOhMAfFexVGJoQ1nBGLMoQCGYKzM0)

### 估值方法论
- [The Crypto Theses 2026 - Messari](https://messari.io/report/the-crypto-theses-2026)
- [Crypto Asset Valuation - CoinMarketCap](https://coinmarketcap.com/academy/article/crypto-asset-valuation-how-to-value-crypto-protocols-and-blockchains)
- [Crypto Valuation Metrics - Eqvista](https://eqvista.com/company-valuation/valuation-crypto-assets/crypto-valuation-metrics/)
- [Market Cap to TVL Ratio - Phemex](https://phemex.com/academy/what-is-market-cap-to-tvl-ratio)
- [FDV in Cryptocurrency - Gate.com](https://www.gate.com/crypto-wiki/article/what-is-fully-diluted-valuation-fdv-in-cryptocurrency-20260119)

### 投资方框架
- [Navigating Crypto in 2026 - Pantera Capital](https://panteracapital.com/blockchain-letter/navigating-crypto-in-2026/)
- [Pantera and Galaxy Digital Forecast Utility Shift - FinanceFeeds](https://financefeeds.com/pantera-and-galaxy-digital-forecast-a-pivotal-shift-toward-utility-in-2026/)
- [Overview of a16z Ecosystem Projects - Gate Learn](https://www.gate.com/learn/articles/overview-of-a16z-ecosystem-projects/5876)
- [Crypto VC List 2025 - CryptoResearch.Report](https://cryptoresearch.report/crypto-research/the-definitive-crypto-vc-list-top-firms-investing-in-blockchain-in-2025/)
- [Delphi's Year Ahead 2026](https://delphidigital.io/year-ahead-2026)

### 做市商分析
- [The State of Crypto Market Making 2025 - Lo.tech](https://lo.tech/stateofmarketmaking)
- [Crypto Market Making: Retainer vs Loan/Call - Flowdesk](https://flowdesk.co/updates/blogs/67215e228e4bf46d9bd3f247/)
- [Market maker deals are quietly killing crypto projects - Cointelegraph](https://cointelegraph.com/news/market-maker-deals-quietly-killing-crypto-projects)
- [Market Making: Predatory or Essential? - Presto Research](https://www.prestolabs.io/research/market-making-predatory-or-essential)
- [Inside Movement's Token-Dump Scandal - CoinDesk](https://www.coindesk.com/tech/2025/04/30/inside-movement-s-token-dump-scandal-secret-contracts-shadow-advisors-and-hidden-middlemen)
- [Crypto Market Manipulation 2025 - Chainalysis](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)

### OTC市场
- [State of The Secondary OTC Market - Presto Research](https://www.prestolabs.io/research/state-of-the-secondary-otc-market)
- [Crypto VC's Shift: OTC Markets - Gate Learn](https://www.gate.com/learn/articles/crypto-vcs-shift-otc-markets-and-investment-changes/4887)
- [Crypto OTC Review: 2024 Results & 2025 Outlook - Finery Markets](https://finerymarkets.com/blog/crypto-otc-review-2024-results-2025-outlook)
- [Crypto investors cash in on locked assets - Mitrade](https://www.mitrade.com/insights/news/live-news/article-3-690257-20250312)

### 历史案例
- [Anatomy of a Run: The Terra Luna Crash - MIT Sloan](https://mitsloan.mit.edu/cfi/anatomy-a-run-terra-luna-crash)
- [FTX's downfall and Binance's consolidation - arXiv](https://arxiv.org/html/2302.11371v3)
- [FDV in Crypto and Overvaluation Risks - Tangem](https://tangem.com/en/blog/post/fdv-in-crypto/)
