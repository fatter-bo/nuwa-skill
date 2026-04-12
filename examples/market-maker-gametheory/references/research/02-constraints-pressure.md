# 做市商资源约束、压力点与恐惧

> 调研目标：理解加密货币做市商"做不到什么"以及"什么情况下他们会被迫行动"。

---

## A. 资源约束

### A1. 资金池有限——多项目资金分散

**核心矛盾：做市商的资金是有限的，同时服务数十个项目时资金被严重稀释。**

- 做市商通常将资金分散在多个交易所，**单一交易所的资金敞口限制在总资金的 10-30%**。这种分散策略意味着任何单个项目能调动的资金有天花板。
- **资金效率陷阱**：一个在 5 家交易所做套利的做市商，必须在每家交易所存入全额保证金。行业分析显示，这种模式**锁死了 60-70% 的工作资金**，无法产生额外收益（[TheStreet](https://www.thestreet.com/crypto/innovation/why-institutional-crypto-still-traps-60-billion-in-pre-funded-accounts-)）。
- **项目方代币分配**：大多数项目将总供应量的 **5-20%** 分配给流动性做市。如果在多交易所上市，需要更多流动性分摊到各平台（[Empirica](https://empirica.io/crypto-market-making/)）。
- **资本缓冲要求**：专业做市商通常保持 **20-50% 的资金作为缓冲**，用于应对不利价格波动而不被迫减仓。这进一步压缩了实际可部署的做市资金（[Bitget Academy](https://www.bitget.com/amp/academy/crypto-market-makers)）。

**实战案例——Alameda Gap**：

Alameda Research 倒闭前是行业最大的做市商，为比特币、以太坊和山寨币提供数十亿美元的流动性。2022 年 11 月崩溃后，BTC 2% 价差内的流动性从 11,800 BTC 骤降到 7,000 BTC。直到 2023 年底，市场深度虽恢复至约 3.5 亿美元（对比 Q3 的 2.5 亿），但仍远低于 FTX 崩溃前的水平——**一个大型做市商的消失留下了持续一年以上的流动性缺口**（[CoinDesk](https://www.coindesk.com/markets/2023/11/03/bitcoin-is-up-70-a-year-after-ftx-debacle-but-alameda-gap-in-liquidity-persists)；[CoinTelegraph](https://cointelegraph.com/explained/the-alameda-gap-and-crypto-liquidity-crisis-explained)）。

---

### A2. 筹码集中度困境——散户"钻石手"推高收筹成本

**当散户持仓分散且坚定不卖时，做市商收集筹码的成本升高，操作难度加大。**

- 本轮周期山寨币数量超过 **20,000 个**（不含低市值 meme），是 2021 年的 2-3 倍。Token 品种激增导致注意力和资金分散。
- "钻石手"策略在上一轮周期有效，但本轮效果递减——市场目标的涨跌节奏加快，参与者容易被深套。这意味着散户行为更难预测，做市商需要为收筹支付更高成本或使用更长的洗盘周期。
- **治理影响风险**：做市商通过费用或库存条款积累大量 Token，可能在链上治理投票中获得不成比例的影响力，引发项目方和社区警惕（[Kairon Labs](https://kaironlabs.com/blog/understanding-market-making-models-in-crypto)）。

---

### A3. 库存风险——做市商也会亏钱

**做市商不是神，持有大量代币库存时价格下跌他们同样承受损失。**

- 做市商在正常交易中不断积累多头或空头头寸。例如卖单占主导时，做市商被迫吃进大量 BTC 库存。价格下跌时直接亏损。
- 复杂对冲策略（使用永续合约、期权、跨交易所对冲）可以中和方向性敞口，但**对冲本身有成本，且在极端行情下可能失效**。
- **每对交易对的敞口限制、总组合 Delta、最大回撤阈值**是做市商的核心风控指标，超过阈值会触发强制减仓。

**实战案例——Wintermute 2022 年被黑**：

2022 年 9 月 20 日，Wintermute 因热钱包私钥泄露（Profanity 虚荣地址漏洞）被黑客窃取 **1.6 亿美元**。CEO Evgeny Gaevoy 声称公司仍有偿付能力，CeFi 和 OTC 业务未受影响。但作为在 50+ 交易所提供流动性的大型做市商，**1.6 亿的损失意味着库存和资金缓冲被严重削弱**，直接影响了其在多个平台上的做市能力（[CoinDesk](https://www.coindesk.com/business/2022/09/20/crypto-market-maker-wintermute-hacked-for-160m-says-ceo)；[TechCrunch](https://techcrunch.com/2022/09/20/crypto-market-maker-wintermute-loses-160-million-in-defi-hack/)）。

---

### A4. 资金成本——借来的钱有利息

**做市周期拖长时，资金利息成本吃掉利润。**

- **比特币抵押贷款利率**：大额贷款（100 万美元以上）年利率约 **9.99-11.9%**。短期贷款利率更低，因为价格波动和市场风险暴露更少。
- **做市合约模式与成本**：
  - **Retainer 模式**：项目方借出 Token + 稳定币给做市商，做市商按月收费，合约到期归还全部借款。成本固定可预测。
  - **Loan/Call Option 模式**：项目方借出 Token（通常为总供应量的一定比例）+ 授予做市商看涨期权。做市商可选择归还 Token 或在市场价高于行权价时行使期权。做市商通过期权和 Gamma 交易获利（[Flowdesk](https://flowdesk.co/updates/blogs/67215e228e4bf46d9bd3f247/)；[Blocmates](https://www.blocmates.com/articles/how-market-making-deals-work-in-crypto)）。
- **最大风险**：在 Loan/Call Option 模式下，如果行权价处于亏损状态（out of the money），做市商可能停止积极做市——尽管合约仍要求达到 KPI，但实际动力大幅下降。

---

### A5. 多项目并行——精力分散

**市场剧烈波动时，做市商顾不过来所有项目。**

- 大型做市商如 Wintermute 同时在 50+ 交易所提供流动性，覆盖数百个交易对。极端行情下无法同时维护所有项目的订单簿。
- **2025 年 10 月闪崩案例**：BTC 从约 $122,000 跌至约 $105,000，SOL 暴跌 40%+，TON 一度跌 80%。Wintermute 和 LO:TECH **暂时停止做市**，因为其"预定义交易规则被打破，无法以安全的 Delta 中性方式做市"。多个交易所买盘侧订单簿几乎清空，出现极端插针行情（[Decrypt](https://decrypt.co/344796/why-wintermute-market-makers-stopped-trading-bitcoin-crash-liquidations)）。
- 清算级联效应：该次事件中 **超过 190 亿美元杠杆头寸在数小时内被清算**，是此前单日最大清算额的 9 倍（[insights4vc](https://insights4vc.substack.com/p/inside-the-19b-flash-crash)）。

---

## B. 恐惧与被迫行动

### B1. 怕被反杀/轧空（Short Squeeze / Gamma Squeeze）

**大买单或利好突袭时，做市商可能被空头套牢，被迫反向买入推高价格。**

- **Gamma Squeeze 机制**：当大量看涨期权被买入，做市商作为卖方为了对冲风险被迫买入标的资产。价格上涨 → 更多期权进入实值 → 做市商被迫买入更多 → 形成正反馈循环。**最大输家是做市商本身**。
- **加密市场的特殊性**：
  - 缺乏严格监管，杠杆可达 10x-20x 甚至更高
  - 市场参与者基数小，价格更容易被操纵
  - Gamma Squeeze 通常**持续几小时到几天**，极为短暂但杀伤力巨大
- **对比**：Short Squeeze 的最大输家是空头交易者；Gamma Squeeze 的最大输家是作为期权卖方的做市商，因为他们持有的是**十亿美元级别的强制对冲头寸**（[Phemex Academy](https://phemex.com/academy/what-is-gamma-squeeze)；[TradingSim](https://www.tradingsim.com/blog/gamma-squeezes-explained)）。

---

### B2. 怕流动性枯竭——极端行情下撤单保命

**做市商在极端行情下会集体撤离，保护自身资金安全。**

**案例一：Luna/UST 崩盘（2022 年 5 月）**

- Terra 生态在 3 天内蒸发 **500 亿美元**市值。
- Luna Foundation Guard (LFG) 将数十亿美元储备资产"借给"专业做市商以防御 UST 脱锚，但储备耗尽后防线崩溃。
- 2022 年 5 月 9 日晚 Anchor 提款加速，LUNA 市值跌至与 UST 流通量持平时，UST 暴跌至 $0.75。
- **做市商行为**：在储备金耗尽后集体撤离，不再为 UST 提供买盘支撑（[Harvard Law](https://corpgov.law.harvard.edu/2023/05/22/anatomy-of-a-run-the-terra-luna-crash/)；[MIT Sloan](https://mitsloan.mit.edu/cfi/anatomy-a-run-terra-luna-crash)）。

**案例二：2025 年 10 月闪崩**

- 做市商 Wintermute 公开表示"无法以安全的 Delta 中性方式做市"而暂停交易。
- 多家交易所出现 API 延迟、界面冻结（Binance）、暂停交易（Robinhood）等情况。
- **订单簿买盘侧几乎为空白**，导致极端插针——这就是做市商撤离时的真实面貌。

**Kaiko 研究发现**：在市场压力期间，恐慌导致的交易量飙升暴露了 BTC 订单簿的巨大流动性缺口，订单簿在数分钟内几乎清空——这是活跃做市商撤退所放大的价格冲击和波动（[Kaiko Research](https://research.kaiko.com/insights/market-makers-continue-to-reduce-liquidity)）。

---

### B3. 怕监管——执法案例的威慑效应

**监管打击正在改变做市商的行为模式和地理分布。**

#### SEC 执法行动

- **2024 年 10 月"Operation Token Mirrors"**：SEC 对 **ZM Quant、Gotbit、CLS Global** 三家做市商及 9 名个人提起欺诈指控。FBI 创建了虚假代币 **NexFundAI** 用作诱饵，三家公司被指控通过算法进行 Wash Trading，每天生成"数万亿笔交易和数十亿美元虚假交易量"（[SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-166)）。
- **Gotbit CEO 后续判决**：Gotbit 的 2,300 万美元 Wash Trade 方案导致 CEO 面临刑事定罪（[CryptoNews](https://cryptonews.com/news/gotbit-collapse-23m-wash-trading-scheme-nets-ceo-prison-sec-civil-suit-imminent/)）。
- **2024 年 SEC 罚款总额激增 3,018%**，对 Terraform Labs 和 Do Kwon 处以创纪录的 **46.8 亿美元**罚款。
- **Jump Trading（Tai Mo Shan）**：2024 年 12 月 SEC 对 Jump 的离岸实体 Tai Mo Shan 提起诉讼，指控其在 UST 稳定性上误导投资者并从事未注册证券交易，最终以 **1.23 亿美元**和解。Terraform Labs 还对 Jump Trading 提起 **40 亿美元**索赔，指控其与 Do Kwon 的"秘密协议"触发了 500 亿美元的 Terra 崩盘（[SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-212)；[DL News](https://www.dlnews.com/articles/regulation/jump-trading-crypto-subsidiary-pays-123-million-for-terrausd/)）。

#### CFTC 执法行动

- 2024 财年 CFTC 追回创纪录的 **171 亿美元**，其中 **FTX 案贡献了 87 亿美元赔偿 + 40 亿美元追缴**。
- CFTC 正在调查 Jump Trading 的加密业务，Jump Crypto 负责人 Kanav Kariya 于 2024 年 6 月辞职。
- **2025-2026 年趋势**：CFTC 与 SEC 联合声明，现货加密交易可在注册交易所进行，但企业必须展示严格的反操纵保障措施——**合规成本预计增加 15%**。

#### 大型做市商撤退

- **2023 年 5 月**：Jane Street 和 Jump Trading 宣布缩减美国加密交易业务，原因是"监管不确定性导致无法按照内部标准运营"。两家撤退后，Coinbase 和 Binance 的流动性明显下降，市场变得"更不流动、更不主流、对机构投资者吸引力更低"（[CoinDesk](https://www.coindesk.com/business/2023/05/09/market-maker-jane-street-jump-retreating-from-us-crypto-trading-bloomberg)；[Bloomberg](https://www.bloomberg.com/news/articles/2023-05-19/crypto-trading-less-institutional-after-jane-street-jump-pull-back)）。

#### DWF Labs 争议

- DWF Labs 被内部 Binance 调查指控进行超过 **3 亿美元 Wash Trade**，操纵 YGG 等至少 6 个 Token 的价格。**揭露此事的 Binance 调查员被解雇**。
- DWF Labs 否认指控，称其为"竞争对手驱动的 FUD"。Binance 后续调查声称"证据不足"（[CoinDesk](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)）。

#### 交易所层面的新规

- **2026 年 3 月**：Binance 发布新规，要求项目方必须**披露做市商身份、法律实体和合约条款**。禁止利润分成和保底收益安排，要求借贷合约明确规定 Token 用途。违规做市商将被列入黑名单。预计 Coinbase、Kraken、OKX 等将在未来数月跟进类似规则（[CoinDesk](https://www.coindesk.com/business/2026/03/25/binance-tightens-market-maker-rules-tells-token-issuers-they-must-disclose-partners)）。

---

### B4. 被迫平仓——库存超风控阈值

**做市商有严格的风控体系，超过阈值时不管价格被迫减仓。**

- **阶梯式清算机制**：当保证金余额低于维持保证金时，系统执行渐进式清算，以最小化市场冲击。根据用户头寸风险限额进行分层清算，防止全仓被一次性清算。
- **做市商的内部风控**：
  - 单交易对最大敞口限制
  - 总组合 Delta 限制（控制方向性风险）
  - 最大回撤阈值（自动触发减仓）
  - 每日/每周 VaR（Value at Risk）限制
- **波动性飙升时**：价差扩大可能触发止损或强制清算。**2025 年 10 月**，190 亿美元的清算级联就是做市商和杠杆交易者集体触发风控阈值的结果。
- **关键认知**：做市商不是散户的"对手方"——他们是有风控约束的理性参与者。当库存风险超过阈值时，他们和散户一样会"割肉出场"。

---

### B5. 合约到期压力——协议周期的行为变化

**做市协议通常 3-12 个月，到期前后做市商行为会发生显著变化。**

#### 合约模式决定到期行为

**Retainer 模式**（月费制）：
- 合约到期时做市商**归还全部借来的 Token**
- 到期前无明显加速出货动机，但做市商可能降低投入精力
- 到期后如不续约，**流动性会突然消失**

**Loan/Call Option 模式**（期权制）：
- **行权价格决定一切**：如果市场价格远高于行权价，做市商会行使期权（低价买入、高价卖出）
- 如果期权处于"out of the money"状态，做市商可能在合约期内逐步消极做市
- **到期前的危险信号**：做市商可能加速将库存 Token 卖出套现（尤其当行权价低于市场价时）
- Binance 新规已标记"与 Token 释放时间表冲突的抛售行为"和"不自然的单向交易"为违规行为

#### 到期后的流动性悬崖

- 做市商在合约到期后**归还流动性给项目方**，如果没有新的做市商接手，交易对的买卖价差会急剧扩大
- 项目方面临"做市商真空期"——这是做市商协议中最被忽视的风险之一
- **Binance 2026 新规要求**：Token 借贷合约必须明确规定被借 Token 的使用方式，减少再抵押、直接出售或转质押的模糊空间

---

## C. 关键洞察总结

### 做市商"做不到什么"

| 约束维度 | 具体限制 |
|---------|---------|
| 资金总量 | 60-70% 资金被锁定在各交易所保证金中，实际可调用资金有限 |
| 单项目资金 | 受限于总资金的分散配置，单个项目通常只能调用总资金的一小部分 |
| 极端行情 | 无法在闪崩中维持做市（如 2025.10 月 Wintermute 暂停案例） |
| 收筹效率 | 散户分散持仓时，收筹成本高、时间长 |
| 风控硬约束 | 超过 Delta/VaR/回撤阈值时必须减仓，无法"扛单" |
| 合规限制 | 监管加码后做市商被迫缩减业务范围（Jane Street、Jump 撤离美国） |

### 做市商"什么时候被迫行动"

| 触发条件 | 被迫行为 |
|---------|---------|
| Gamma Squeeze / 大量看涨期权被买入 | 被迫买入标的资产对冲，推高价格 |
| 库存超过风控阈值 | 被迫减仓，不管当前价格 |
| 极端行情 / 流动性枯竭 | 撤单逃跑，保护自有资金 |
| 做市合约到期 | 归还 Token，可能在到期前加速出货 |
| 监管执法 / 交易所调查 | 缩减业务、退出特定市场或代币 |
| 被黑客攻击 | 库存和资金缓冲削弱，被迫调整全平台做市策略 |
| 交易对手风险爆发 | 减少对关联平台/项目的敞口（如 FTX 倒闭后的连锁反应） |

---

## D. 参考来源

### Alameda Research 与 FTX 崩溃
- [The Alameda gap and crypto liquidity crisis explained - CoinTelegraph](https://cointelegraph.com/explained/the-alameda-gap-and-crypto-liquidity-crisis-explained)
- [Bitcoin Is Up 70% a Year After FTX Debacle, but 'Alameda Gap' in Liquidity Persists - CoinDesk](https://www.coindesk.com/markets/2023/11/03/bitcoin-is-up-70-a-year-after-ftx-debacle-but-alameda-gap-in-liquidity-persists)
- [Market Makers Were Wary of FTX Before Collapse - CoinDesk](https://www.coindesk.com/markets/2022/11/16/market-makers-were-wary-of-ftx-before-collapse)
- [Crypto market liquidity dries up following Alameda and FTX collapse - The Block](https://www.theblock.co/post/188845/crypto-market-liquidity-dries-up-following-alameda-and-ftx-collapse)

### Luna/Terra 崩盘
- [Anatomy of a Run: The Terra Luna Crash - Harvard Law](https://corpgov.law.harvard.edu/2023/05/22/anatomy-of-a-run-the-terra-luna-crash/)
- [Anatomy of a Run: The Terra Luna Crash - MIT Sloan](https://mitsloan.mit.edu/cfi/anatomy-a-run-terra-luna-crash)
- [NBER Working Paper: Anatomy of a Run](https://www.nber.org/system/files/working_papers/w31160/w31160.pdf)

### Wintermute 被黑事件
- [Crypto Market Maker Wintermute Hacked for $160M - CoinDesk](https://www.coindesk.com/business/2022/09/20/crypto-market-maker-wintermute-hacked-for-160m-says-ceo)
- [Wintermute Hacked for About $160 Million - Bloomberg](https://www.bloomberg.com/news/articles/2022-09-20/wintermute-hacked-for-about-160-million-in-its-defi-operations)
- [Explained: The Wintermute Hack - Halborn](https://www.halborn.com/blog/post/explained-the-wintermute-hack-september-2022)

### Kaiko 流动性研究
- [Market Makers Continue to Reduce Liquidity - Kaiko Research](https://research.kaiko.com/insights/market-makers-continue-to-reduce-liquidity)
- [The Crypto Liquidity Concentration Report - Kaiko Research](https://research.kaiko.com/insights/the-crypto-liquidity-concentration-report)
- [When October Surprise Meets Crypto Liquidity Drought - Kaiko Research](https://research.kaiko.com/insights/when-october-surprise-meets-crypto-liquidity-drought)

### SEC/CFTC 执法
- [SEC Charges Three Market Makers in Crackdown - SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-166)
- [SEC Charges Tai Mo Shan (Jump Trading) $123M - SEC.gov](https://www.sec.gov/newsroom/press-releases/2024-212)
- [Jump Trading faces $4B lawsuit over Terra collapse - CoinTelegraph](https://cointelegraph.com/news/jump-trading-4b-lawsuit-tied-to-50b-terra-crash)
- [CFTC Releases FY 2024 Enforcement Results - CFTC.gov](https://www.cftc.gov/PressRoom/PressReleases/9011-24)
- [Gotbit Collapse: $23M Wash-Trade Conviction - CryptoNews](https://cryptonews.com/news/gotbit-collapse-23m-wash-trading-scheme-nets-ceo-prison-sec-civil-suit-imminent/)

### 做市商撤退与监管压力
- [Market Makers Jane Street, Jump Retreating From U.S. Crypto Trading - CoinDesk](https://www.coindesk.com/business/2023/05/09/market-maker-jane-street-jump-retreating-from-us-crypto-trading-bloomberg)
- [Crypto Trading Less Institutional After Jane Street, Jump Pull Back - Bloomberg](https://www.bloomberg.com/news/articles/2023-05-19/crypto-trading-less-institutional-after-jane-street-jump-pull-back)

### DWF Labs 争议
- [Binance Fired Investigator Who Uncovered Market Manipulation at DWF Labs - CoinDesk/WSJ](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [DWF Labs calls manipulation claims 'competitor-driven FUD' - DL News](https://www.dlnews.com/articles/people-culture/dwf-denies-market-manipulation-wash-trading-on-binance/)

### 做市商合约与商业模式
- [Crypto Market Making: Retainer vs. Loan/Call Model - Flowdesk](https://flowdesk.co/updates/blogs/67215e228e4bf46d9bd3f247/)
- [How Market-Making Deals Work In Crypto - Blocmates](https://www.blocmates.com/articles/how-market-making-deals-work-in-crypto)
- [Retainer vs Options Model for Market-Making - Caladan](https://caladan.xyz/retainer-vs-options/)
- [Market Maker Red Flags and Guidelines - Binance](https://www.binance.com/en/blog/education/3378047125698186214)

### 闪崩与做市商行为
- [Why Wintermute and Other Market Makers Stopped Trading During Bitcoin Crash - Decrypt](https://decrypt.co/344796/why-wintermute-market-makers-stopped-trading-bitcoin-crash-liquidations)
- [Inside the $19B Flash Crash - insights4vc](https://insights4vc.substack.com/p/inside-the-19b-flash-crash)

### 交易所新规
- [Binance tightens market maker rules and warns token issuers to disclose partners - CoinDesk](https://www.coindesk.com/business/2026/03/25/binance-tightens-market-maker-rules-tells-token-issuers-they-must-disclose-partners)

### Gamma Squeeze 与轧空
- [What Is A Gamma Squeeze - Phemex Academy](https://phemex.com/academy/what-is-gamma-squeeze)
- [Gamma Squeeze vs Short Squeeze Explained - TradingSim](https://www.tradingsim.com/blog/gamma-squeezes-explained)
- [Gamma Squeeze in Crypto - Tradelink](https://tradelink.pro/blog/gamma-squeeze-in-crypto/)
