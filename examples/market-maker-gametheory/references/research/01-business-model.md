# 加密货币做市商商业模型与激励结构

> 调研日期: 2026-04-12
> 用途: 为"加密货币做市商行为识别与博弈框架"Skill建立做市商商业模型的系统性理解

---

## 一、做市商收入来源全景

加密货币做市商的收入来源可分为五大类:

### 1.1 买卖价差 (Bid-Ask Spread)

做市商的核心盈利模式。做市商通过在买方和卖方之间设置价差来获利——以较低价格买入资产,以较高价格卖出同一资产。Wintermute CEO Evgeny Gaevoy 明确表示:"like all other proprietary trading firms, [we] aim to make money through trading, we are not a charity." (来源: ChainCatcher)

- **主流交易对价差**: 0.01%-0.05% (BTC/USDT等)
- **长尾代币价差**: 0.1%-0.5% 甚至更高
- **Kaiko研究发现**: FTX倒闭后,做市商越来越倾向于在极窄价格区间(0.1%以内)提供流动性

### 1.2 交易所返佣 (Exchange Rebates)

交易所采用 maker-taker 费用模型激励流动性提供:

- **标准maker费率**: 通常为0.00%-0.02%
- **高级做市商计划返佣**: 最高可达 **-0.015%**(交易所倒贴给做市商,如Bybit的Market Maker Incentive Program)
- **阶梯制度**: 交易量越大,费率越低甚至获得负费率返佣
- **准入门槛**: 通常要求月交易量达到 100万-500万 USD 以上

### 1.3 项目方服务费

做市商为代币项目提供流动性服务收取的费用,根据合作模式不同而异(详见第三节):

- **固定费用模式**: 每月 $1,000-$7,000/交易对/交易所
- **Hybrid模式**: 固定月费 + 15%-20% 绩效分成(carry)
- **贷款+期权模式**: 不收取现金费用,而是获取代币期权作为补偿

### 1.4 代币借贷收益

做市商在贷款模式下获得大量代币库存,可以通过以下方式获得额外收益:

- 在DeFi协议中提供流动性获取LP收益
- 跨交易所套利——Wintermute业务涵盖50+交易所的CeFi做市、DeFi套利(Uniswap/Aave等)、以及OTC交易
- 利用代币库存进行delta对冲策略

### 1.5 Call Option (看涨期权) 收益

在最常见的贷款+期权模式中,做市商获得以预设价格购买代币的权利(详见3.2节)。当代币升值时,期权本身就是重要利润来源。

---

## 二、做市商的客户关系与利益动态

### 2.1 项目方-做市商关系本质

项目方付费请做市商维护流动性,双方存在利益绑定但不完全一致:

**利益一致的方面:**
- 双方都希望代币有良好的流动性和健康的市场结构
- 代币上市后良好的流动性有助于吸引更多投资者
- 做市商声誉与项目成功相关

**利益冲突的方面:**
- 做市商追求短期交易利润,项目方追求长期价值
- 信息不对称——项目方无法验证做市商是否真正部署了承诺的资本 (来源: blocmates)
- 做市商可能在市场下行时减少资本部署以"保存资本",而非履行流动性义务
- 在贷款+期权模式下,做市商可能有动力在期权到期前操纵价格

### 2.2 Wintermute 的公开立场

Gaevoy 对价格下跌的解释包括三个因素 (来源: ChainCatcher):

1. **宏观因素**: 加密市场与美股和宏观经济趋势相关
2. **供应增加**: 上市后流动性增强,早期持有者更容易退出
3. **嵌入式看涨期权**: 部分项目授予做市商预设行权价的卖出权利

Gaevoy强调Wintermute所有操作保持"delta中性"——头寸在多平台间对冲以消除方向性风险。但他也直言:"adults should take responsibility for their choices"。

### 2.3 DWF Labs争议——利益冲突的极端案例

**2024年Binance内部调查事件:**

- 华尔街日报报道:Binance内部调查指控DWF Labs操纵多个代币市场,涉及**3亿美元**的自成交(wash trade)
- 调查发现DWF Labs创始人推广代币后抛售,导致价格暴跌——符合"拉高出货"(pump-and-dump)模式
- 受影响代币包括 YGG、DODO、C98等
- **Binance的争议性处理**: Binance解雇了揭发此事的内部调查员,保留DWF Labs为客户,声称自成交"可能是意外的"

**背景**: DWF Labs由Andrei Grachev控制,此人此前因策划拉高出货计划被定罪(来源: The Nation/Wikipedia)

---

## 三、三种核心合作模式深度分析

### 3.1 固定费用模式 (Retainer / MMaaS)

**运作机制:**
- 项目方每月支付固定费用(每个交易对每个交易所 $1,000-$7,000)
- 项目方自行提供治理代币和基础资产(如USDT)作为做市库存
- 做市商使用自有交易基础设施和算法
- 合同期限通常12-24个月,最短3个月

**激励分析:**
- 做市商无方向性激励——费用与代币价格无关
- 不存在因期权到期带来的价格操纵动机
- 项目方保留流动性的所有权
- **缺点**: 做市商缺乏超额表现的动力,可能仅满足最低KPI

**适用场景:**
- 项目方资金充裕,希望保持对流动性的完全控制
- 不愿稀释代币给做市商的团队
- 需要透明且可预测的成本结构

**监管考量:**
- Caladan指出,固定费用模式也可能面临证券法审查
- 费用结构不随市场条件调整是潜在风险

### 3.2 贷款 + 看涨期权模式 (Loan + Call Option)

**运作机制:**
- 项目方借出代币给做市商(通常为总FDV的1%-2%)
- 做市商获得看涨期权——有权在约定期限内以预设行权价(strike price)购买代币
- 合同期限: 12-24个月
- 项目方无需支付现金费用

**具体案例 (来源: blocmates):**
> 假设"Mickey Mouse"协议借出100,000枚MICKEY代币,行权价$1/枚,期限12个月。
> - 若代币涨至$10: 做市商可选择支付$100,000现金(而非归还价值$1,000,000的代币),净赚$900,000
> - 若代币跌至$0.50: 归还代币比支付现金更划算,做市商依靠价差利润弥补

**核心博弈问题: 做市商有动力砸盘还是拉盘?**

这是一个复杂的多层次问题:

**拉盘激励 (表面逻辑):**
- 期权价值与代币价格正相关——价格越高,call option越有价值
- Presto Labs指出:"the value of the call option correlates directly with the token's price, providing market makers with safeguards against early pump-and-dump schemes."

**砸盘激励 (隐藏逻辑):**
- 做市商可以先拉盘使期权进入深度实值(deep in-the-money),然后在行权前大量抛售
- 做市商持有大量借入代币,可以在行权前通过抛售获利,最后以现金结算期权
- 低行权价 + 无KPI/SLA约束 = 掠夺性条款(predatory terms)
- SEC和DOJ 2024年10月起诉了ZM Quant、Gorbit、CLS Global、MyTrade四家做市商,指控其利用交易机器人制造虚假交易量

**关键风险因素 (来源: Caladan):**

| 因素 | 影响 |
|------|------|
| 时间周期 | 更长期限增加行权价被触及的概率,建议最少一年 |
| 贷款规模 | 与FDV相关;FDV越高,百分比越小 |
| 波动率 | 波动率越高,期权进入实值的可能性越大 |
| 分批行权(Tranches) | 阶梯式行权价(季度/半年)保护项目方 |
| 行权价 | 行权价越低,做市商盈利概率越高 |

**Caladan的警告:**
> "If an MM promises a token price target or trading volume (aka wash trading), run for the hills. This is market manipulation, not market-making."

### 3.3 绩效费模式 (Performance Fee)

**运作机制:**
- 做市商根据达成的交易量、流动性指标或其他KPI收取费用
- 可以是纯绩效费(按利润抽成)或混合模式(固定费+绩效奖金)
- 典型的绩效抽成比例: 15%-20% carry

**激励分析:**
- 按交易量收费可能激励wash trading(虚假交易量)
- 按流动性深度收费更能对齐项目方利益
- 需要精心设计的KPI框架防止游戏化(gaming)

**适用场景:**
- 项目方希望做市商有更强的表现动力
- 可接受更高的变动成本
- 有能力监控和验证KPI达成情况

---

## 四、KPI体系与合规压力

### 4.1 常见KPI指标

| KPI类别 | 典型要求 | 说明 |
|---------|---------|------|
| 最大价差 (Max Spread) | 主流币对 < 0.05%, 长尾 < 2% | 买卖报价之间的最大允许差距 |
| 最低流动性深度 (Min Depth) | $10,000+ 双边 | 在指定价差范围内维持的最低订单量 |
| 挂单时间 (Uptime) | >95% | 做市商必须在规定时间内在线报价 |
| 价格区间维护 | 价格偏离基准 < X% | 维持价格在合理区间内 |
| 交易量目标 | 日均交易量达标 | 需警惕wash trading风险 |
| 波动性响应时间 | 极端波动时的调整速度 | 黑天鹅事件中的表现 |

### 4.2 Binance 2026年3月新规要点

Binance于2026年3月发布了重大做市商规则更新 (来源: CoinDesk/Binance Blog):

**六大红旗信号:**
1. **抛售与代币释放时间表冲突**: 做市商在锁仓期结束前大量抛售
2. **单边交易**: 持续卖单而无对应买单
3. **多交易所协调抛售**: 大额代币同时转入多个平台并抛售
4. **交易量-价格不匹配**: 高交易量但价格变动极小(wash trading信号)
5. **薄流动性下的价格剧烈波动**: 浅薄订单簿中小额交易导致超大价格影响
6. **交易量与流动性不平衡**: 大量交易活动但订单簿深度极浅

**核心禁令:**
- 禁止利润分成模式(profit-sharing)和保证收益模式(guaranteed profit)
- 代币贷款协议必须明确说明用途和条款
- 项目方必须披露做市商身份、法律实体和合同条款
- Binance保留将不合规做市商列入黑名单的权利

### 4.3 Amberdata流动性研究发现

Amberdata的时间维度流动性研究揭示了做市商行为的可预测模式 (来源: Amberdata Blog):

- **UTC 11:00** Bitcoin/FDUSD在Binance: $386万流动性(10bp以内),价差紧密
- **UTC 21:00** 同一交易对: 仅$271万深度——**流动性下降42%**
- 市场从紧密价差的买单主导 转向 宽价差的卖单主导,是电子化订单簿的结构性特征

**启示**: 做市商的流动性提供是动态的、有规律的,这为识别做市商行为提供了量化基础。

---

## 五、做市商之间的竞争博弈

### 5.1 同一项目多做市商的博弈动态

**价差竞争:**
- 多个做市商在同一代币上竞争时,会竞相缩小价差
- 如果一个做市商设置略窄的价差,其他做市商通常会跟进
- NASDAQ的多做市商竞争模型证明了这一机制的有效性

**合谋风险:**
- 多个做市商在信息不对称环境中可能形成隐性合谋
- DWF Labs事件显示,做市商可能与交易所形成利益共同体

### 5.2 主要做市商竞争格局

**Tier 1 (头部做市商):**
- **Wintermute**: 日交易量$50亿+, 覆盖50+交易所, CeFi+DeFi+OTC全覆盖
- **GSR Markets**: 9年加密做市经验, 专注新代币和异质资产做市
- **Cumberland/DRW**: 传统金融巨头DRW的加密子公司, OTC大宗交易领导者
- **Jump Trading/Jump Crypto**: 传统高频交易巨头

**Tier 2 (争议型/新兴做市商):**
- **DWF Labs**: 争议缠身但业务量巨大, 模式更偏投资+做市混合
- **Flowdesk**: 提供详细的retainer vs option模型对比分析
- **Caladan**: 公开教育项目方如何评估做市商

**传统金融进入者:**
- **Citadel Securities**: 2025年2月正式进军加密做市
- **Virtu Financial**: 与Citadel联合建设EDX Markets加密交易生态
- **目标**: 复制传统金融的市场结构,将交易所、托管和交易功能分离

### 5.3 传统金融做市商模型的可迁移框架

**Citadel/Virtu的关键特征:**
- 纯技术驱动的算法做市——不持有方向性头寸
- 极低延迟(<1微秒)的报价更新
- 多资产、多市场的风险对冲体系
- 上市公司透明度(Virtu是公开上市公司)
- 严格的监管合规框架(SEC/FINRA)

**迁移到加密市场的挑战:**
- 加密市场24/7运作 vs 传统市场有固定交易时段
- 加密市场缺乏统一的监管框架
- 链上透明性与做市商策略保密性的矛盾
- DeFi做市与CeFi做市的技术差异巨大

**EDX Markets的设计原则:**
- 复制传统金融市场的结构,确保交易所、托管和交易功能的清晰分离
- 集中更多流动性,使投资者订单以比传统加密交易所更紧的价差执行

---

## 六、Chainalysis 2025年市场操纵数据

Chainalysis 2025年加密犯罪报告的关键数据 (来源: Chainalysis Blog):

- **2024年illicit交易量**: 约$25.7亿由虚假交易人为制造
- **Pump-and-dump占比**: 占市场活动的4.52%(2023年为3.59%,同比上升)
- **代币发行量**: 2024年超过300万枚新代币发行,其中约129万枚(42.54%)在DEX上市
- **SEC/DOJ执法行动**: 2024年10月,SEC指控ZM Quant、Gorbit、CLS Global、MyTrade四家做市商制造虚假交易量;DOJ同期起诉18个个人和实体

---

## 七、综合分析: 做市商行为识别的关键维度

### 7.1 合法做市 vs 操纵行为的判断框架

| 维度 | 合法做市 | 操纵行为 |
|------|---------|---------|
| 报价行为 | 双边持续报价,价差稳定 | 单边报价,价差异常波动 |
| 交易量 | 与市场深度匹配 | 虚假放量,深度不匹配 |
| 价格影响 | 平滑价格波动 | 制造人为价格波动 |
| 代币流动 | 符合释放时间表 | 提前大量转移到交易所 |
| 跨平台行为 | 套利维持价格一致性 | 协调多平台抛售 |
| 信息披露 | 透明的合同条款 | 隐藏关联方交易 |

### 7.2 做市商激励结构的博弈论总结

**在Retainer模式下:**
- 纳什均衡: 做市商提供最低合规水平的流动性
- 博弈类型: 委托-代理博弈(principal-agent)
- 关键问题: 道德风险(moral hazard)——做市商可能"出工不出力"

**在Loan+Call Option模式下:**
- 纳什均衡: 取决于期权条款——合理的行权价和分批行权机制下,均衡倾向于做市商维护价格稳定
- 博弈类型: 带有期权博弈的动态博弈
- 关键问题: 在行权日临近时的非合作行为激励——做市商可能先拉盘再砸盘
- 对策: 分批行权(tranches)、KPI约束、事后审计

**在多做市商竞争下:**
- 纳什均衡: 价差趋向于最低可持续水平(类似伯特兰竞争)
- 博弈类型: 重复博弈中的价差竞争
- 关键问题: 合谋可能性——特别是在信息不对称的加密市场

---

## 参考来源

### Wintermute / Evgeny Gaevoy
- [Wintermute CEO Discusses Future of Crypto Trading - CoinDesk](https://www.coindesk.com/consensus-hong-kong-2025-coverage/2025/01/31/wintermute-ceo-evgeny-gaevoy-discusses-the-future-of-crypto-trading)
- [Wintermute Founder Responds: We Are Not a Charity - ChainCatcher](https://www.chaincatcher.com/en/article/2165791)
- [Gaevoy on Business Expansion Plans - Bloomberg](https://www.bloomberg.com/news/videos/2025-02-19/wintermute-s-gaevoy-on-business-expansion-plans-video)
- [Gaevoy's Unique Liquidity Model - Bitcoin Ethereum News](https://bitcoinethereumnews.com/tech/evgeny-gaevoy-wintermutes-unique-liquidity-model-drives-market-making-retail-investors-struggle-with-late-entries-and-the-rise-of-derivatives-enhances-institutional-safety/)
- [Wintermute CEO on Pump-and-Dump Accusations - DL News](https://www.dlnews.com/articles/people-culture/wintermute-ceo-says-pump-and-dump-accusations-are-flattering/)

### DWF Labs 争议
- [Binance Fired Investigator Who Uncovered DWF Labs Manipulation - CoinDesk](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [DWF Labs Calls Claims 'Competitor-Driven FUD' - DL News](https://www.dlnews.com/articles/people-culture/dwf-denies-market-manipulation-wash-trading-on-binance/)
- [DWF Labs Fraud Schemes - Crypto Shaman / Medium](https://cryptoshaman.medium.com/en-dwf-labs-fraud-schemes-pump-and-dumps-and-scams-e4d96a407432)
- [DWF Labs - Wikipedia](https://en.wikipedia.org/wiki/DWF_Labs)

### 做市商模型对比分析
- [Retainer vs Loan/Call Model - Flowdesk](https://flowdesk.co/updates/blogs/67215e228e4bf46d9bd3f247/)
- [Retainer vs Options Model - Caladan](https://caladan.xyz/retainer-vs-options/)
- [Liquidity is the Product - Fabric Ventures / Medium](https://medium.com/fabric-ventures/liquidity-is-the-product-how-to-launch-tokens-and-work-with-market-makers-251cd7d7eb01)
- [How Market-Making Deals Work - blocmates](https://www.blocmates.com/articles/how-market-making-deals-work-in-crypto)
- [Market Making 101 - Presto Labs / Medium](https://medium.com/prestolabs/market-making-101-en-9f260823f501)
- [Market-Making Models in Web3 - TechAccountingPro](https://blog.techaccountingpro.com/p/accounting-for-market-making-arrangements)

### Binance 2026新规
- [Binance Tightens Market Maker Rules - CoinDesk](https://www.coindesk.com/business/2026/03/25/binance-tightens-market-maker-rules-tells-token-issuers-they-must-disclose-partners)
- [Binance Market Maker Red Flags - Binance Blog](https://www.binance.com/en/blog/education/3378047125698186214)
- [Binance Cracks Down on Market Makers - Disruption Banking](https://www.disruptionbanking.com/2026/03/31/binance-cracks-down-on-market-makers-except-the-ones-it-depends-on-most/)
- [Binance New Market Maker Guidelines - The Block](https://www.theblock.co/post/395111/binance-points-market-maker-token-launch-red-flags-updated-trading-rules)

### Kaiko / Amberdata 研究
- [Crypto Liquidity Concentration Report - Kaiko Research](https://research.kaiko.com/insights/the-crypto-liquidity-concentration-report)
- [Temporal Patterns in Market Depth - Amberdata Blog](https://blog.amberdata.io/the-rhythm-of-liquidity-temporal-patterns-in-market-depth)
- [How Liquidity Really Works - Amberdata Blog](https://blog.amberdata.io/how-liquidity-really-works-in-crypto-markets)
- [Understanding Centralized Exchange Liquidity - Kaiko Research](https://research.kaiko.com/insights/centralized-exchange-liquidity)

### Chainalysis 市场操纵报告
- [Crypto Market Manipulation 2025 - Chainalysis](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)
- [2025 Crypto Crime Report Introduction - Chainalysis](https://www.chainalysis.com/blog/2025-crypto-crime-report-introduction/)

### 传统金融做市商 (Citadel / Virtu)
- [Citadel Securities Building Crypto Trading Platform - Blockworks](https://blockworks.co/news/citadel-securities-virtu-financial-building-crypto-trading-platform)
- [Citadel Securities Ventures into Crypto Market-Making - Blockhead](https://www.blockhead.co/2025/02/25/citadel-securities-ventures-into-cryptocurrency-market-making-2/)
- [Citadel Securities - Wikipedia](https://en.wikipedia.org/wiki/Citadel_Securities)

### SEC/DOJ 执法行动
- [FBI Caught Pump-and-Dump Schemers - TaxPage](https://taxpage.com/articles-and-tips/how-the-fbi-went-fishing-with-nexfundai-a-crypto-currency-and-caught-pump-and-dump-market-maker-schemers/)
- [Cryptocurrency Companies Charged for Wash Trading - Arnold & Porter](https://www.arnoldporter.com/en/perspectives/blogs/enforcement-edge/2024/10/remaking-the-classics)
- [Crypto Market Makers Under the Spotlight - Solidus Labs](https://www.soliduslabs.com/post/enforcement-around-the-corner-crypto-market-makers-under-the-spotlight)

### 其他
- [Broken Market-Making Deals Derailing Projects - Crypto.news](https://crypto.news/broken-market-making-deals-derail-promising-projects/)
- [DWF Labs: Roles and Models Explained](https://www.dwf-labs.com/news/roles-and-models-of-market-making-in-crypto-explained-insights-from-dwf-labs)
- [Crypto Market Maker Incentives - Paybis Blog](https://paybis.com/blog/crypto-market-maker-incentives/)
