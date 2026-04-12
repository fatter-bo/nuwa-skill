# Arthur Hayes 重大决策记录与关键行动

> 调研日期：2026-04-12
> 调研对象：Arthur Hayes（BitMEX联合创始人、Maelstrom基金CIO）

---

## 一、2014年：创立BitMEX

### 1.1 从花旗银行交易员到创业

**背景**：Arthur Hayes毕业于沃顿商学院，先后在德意志银行（Deutsche Bank）和花旗银行（Citibank）担任股票衍生品交易员，驻香港。2013年被花旗银行裁员，此后发现了加密货币市场中的套利机会。

**决策逻辑**：
- Hayes在传统金融（TradFi）中积累了深厚的衍生品设计经验，尤其是Delta One交易策略
- 2013年被裁后，他注意到不同比特币交易所之间存在显著价差，开始进行跨交易所套利
- 他认识到加密市场缺乏专业级的衍生品交易工具，而这正是他的专长所在

**实际行动**：2014年11月，Hayes与Benjamin Delo和Samuel Reed共同创立了BitMEX（Bitcoin Mercantile Exchange）。

**结果**：BitMEX成长为全球最大的加密货币衍生品交易所之一，累计交易量达数万亿美元。

### 1.2 为什么选择香港和塞舌尔？

**决策**：运营总部设在香港，但母公司HDR Global Trading Limited注册在塞舌尔共和国。

**决策逻辑**：
- **香港**：Hayes在香港有深厚的人脉网络和工作经验；香港是亚洲金融中心，人才密集。BitMEX租下了长江中心（Cheung Kong Center）第45层——这是香港最昂贵的办公楼之一，高盛、巴克莱、彭博和美国银行都在此办公
- **塞舌尔**：Hayes明确表示选择塞舌尔是因为该国"对比特币没有明确的监管规定"。这是一种典型的监管套利策略——在信息披露要求最低的离岸司法管辖区注册，同时在金融发达地区运营

**Hayes的原话**：
- 被问及为何在塞舌尔注册时，Hayes说他不需要"向美国政府低头接受监管"
- 被问美国和塞舌尔有什么区别时，Hayes的回答是"美国的贿赂更贵，在塞舌尔只需要一个椰子"

**结果**：塞舌尔注册并未保护公司免受美国执法，2020年Hayes和联合创始人仍遭美国起诉。

---

## 二、永续合约（Perpetual Swap）的设计

### 2.1 产品创新背景

**时间**：2016年5月，BitMEX推出XBTUSD永续合约。

**决策逻辑**：
- 传统期货有到期日，交易者需要不断滚仓（roll positions），产生额外费用和摩擦成本
- Hayes借鉴了外汇市场中的概念，设计出无到期日的合约产品
- Ben Delo提供了关键的技术解决方案

**产品特点**：
- 无到期日——消除了滚仓需求
- 以比特币结算（cash-settled in BTC）
- 允许高达100倍杠杆交易

### 2.2 Funding Rate机制

**设计逻辑**：
- 每8小时结算一次资金费率
- 费率取决于永续合约价格与现货指数价格之间的价差
- 当合约价格高于现货价格（多头过多）时，多头向空头支付资金费率；反之亦然
- 这一机制巧妙地将合约价格锚定在现货价格附近，无需传统的到期交割

**创新意义**：使投资者能享受期货交易的杠杆优势，同时免去合约到期和滚仓的麻烦。

### 2.3 100倍杠杆的决策

**演进过程**：
- BitMEX最初采用保证金担保结算系统（guaranteed settlement margin system），只能提供3倍杠杆
- 为了提升竞争力，Hayes将系统切换为社会化损失机制（socialized loss system）
- 结合BitMEX优越的交易引擎技术，到2015年10月实现了100倍杠杆

**决策逻辑**：高杠杆是吸引专业交易者的核心卖点，社会化损失机制将极端行情下的风险分散到全体盈利交易者身上，而非由交易所承担。

### 2.4 市场影响

- 到2020年，所有主要衍生品交易所都复制了永续合约设计
- 永续合约已成为有史以来交易量最大的加密货币工具
- 累计交易量达数万亿美元

---

## 三、无KYC/AML的早期运营策略

### 3.1 决策内容

**时间跨度**：2015年9月至2020年9月，BitMEX故意未建立充分的KYC（了解你的客户）和AML（反洗钱）程序。

**Hayes的公开立场**：
- 在BitMEX网站、博客和媒体采访中公开宣传公司不做KYC合规
- 2017年2月博客文章中写道："KYC和AML违规被全世界政府用来骚扰公司"
- Hayes认为2014年时，对于比特币交易"没有政府监管覆盖KYC或AML"

### 3.2 决策逻辑

- **意识形态**：与加密社区早期的去中心化、抗审查理念一致
- **商业考量**：无KYC降低了用户注册摩擦，加速用户增长
- **监管套利**：在塞舌尔注册，认为不受美国法律管辖

### 3.3 后果

- 检察官指控无KYC要求使BitMEX成为犯罪活动的温床，包括洗钱和逃避制裁
- 直接导致2020年的DOJ/CFTC起诉
- BitMEX被罚款1亿美元
- 三位联合创始人各被罚1000万美元民事罚款

---

## 四、2020年DOJ/CFTC起诉后的应对

### 4.1 起诉经过

**时间**：2020年10月，美国司法部（DOJ）和商品期货交易委员会（CFTC）同时对BitMEX及其运营者提起诉讼。

**指控**：Hayes、Reed、Delo以及BitMEX首任员工Gregory Dwyer各面临一项违反《银行保密法》（BSA）的指控，以及一项共谋违反BSA的指控。

### 4.2 Hayes的应对策略

**决策**：
1. **离开日常运营**：Hayes卸任BitMEX CEO职务
2. **谈判认罪协议**：2022年2月，Hayes认罪一项故意未实施反洗钱程序的指控
3. **接受处罚**：认罪协议规定6-12个月监禁的量刑指南

### 4.3 判决结果

- 2022年5月20日，Hayes被判处两年缓刑，其中前六个月须居家监禁并佩戴电子脚环
- 法官Koeltl拒绝将Hayes作为离岸交易所的"典型"来严厉判罚，主张应将其作为个人对待
- 向CFTC支付1000万美元民事罚款
- BitMEX作为公司在2024年7月认罪，被罚1亿美元

### 4.4 值得注意的细节

- Hayes并未完全离开BitMEX——他、Delo和Reed至今仍共同担任BitMEX母公司的董事会成员
- 判决结果远轻于检察官建议的监禁刑期，被加密社区视为相对温和的处罚

---

## 五、转型为投资者/作家

### 5.1 Maelstrom基金

**定位**：Hayes的家族办公室（Family Office），专注于加密生态系统的长期投资。

**投资领域**：
- **风险投资（Venture）**：已投资39家公司，过去12个月新增9笔投资
- **流动性代币（Liquid）**：积极交易BTC、ETH、ENA、PENDLE、HYPE、ZEC等
- **私募股权（Private Equity）**：2026年筹集2.5亿美元新基金，计划每年投资4000-7500万美元收购4-6家中型加密基础设施和数据公司
- **公开市场（Public Markets）**：包括黄金、石油、国防股票

**2025年投资表现**：
- 盈利但不均衡
- 成功：BTC、HYPE、PENDLE带来强劲回报
- 失败：PUMP等代币造成损失
- 2025年4月比特币跌破85,000美元时"全力做多"（maximum long）

**2025-2026年组合调整**：
- 大幅减持ETH（两周内抛售超550万美元ETH）
- 增持DeFi代币：ENA、PENDLE、ETHFI、LDO
- 2026年初进入"几乎最大风险"状态，押注山寨币
- 增加隐私币（ZEC等）头寸

**部分代币表现（截至2025年12月）**：
- ENA：年内下跌78%
- PENDLE：年内下跌65%
- ETHFI：年内下跌68%
- 这些下跌反映了Hayes部分中短期投资的困难

### 5.2 博客写作作为市场影响力工具

**平台**：Crypto Trader Digest（Substack）及Medium，拥有数万名订阅者。

**写作策略**：
- Hayes自称"首席故事官"（Chief Story Officer）
- 写作风格独特、非正式、直率，用口语化语言和个人轶事解释复杂金融概念
- 他不在乎读者对其叙事的态度是正面还是负面——只要叙事能传播即可
- 他认为"因为做多赚钱比做空多，乐观主义在周期中将战胜悲观主义"

**市场影响力**：
- 他的文章发布经常引发市场波动
- 已成为塑造加密交易者叙事和市场情绪的重要声音
- 超过十年的加密市场交易经验和三轮牛市周期的亲身经历为其分析提供了基础

---

## 六、关键投资决策和市场预测

### 6.1 宏观框架

**核心理论——"末日与繁荣"（Doom-and-Boom）框架**：

1. **美元流动性主导一切**：比特币本质上是"流动性警报器"，反映市场中美元流动性是否充足
2. **央行印钞驱动牛市**：COVID-19期间美国印钞超4万亿美元，比特币从3,800美元涨至近70,000美元
3. **财政部才是真正的操控者**：Hayes认为真正牵动全球流动性的不是美联储，而是美国财政部（尤其是财政部长Scott Bessent通过债务管理策略操控全球资金流）
4. **战争支出+AI驱动的经济冲击+隐性流动性注入**：这些因素正在积累系统性压力，美联储最终不得不重启大规模印钞

**价格预测**：
- 比特币到2028年达100万美元
- 比特币到2027年达75万美元
- 认为美联储的"储备管理购买"（RMP）政策将替代传统QE，注入流动性推动风险资产

### 6.2 预测准确度分析

**Protos分析：20次市场预测的回顾**（2024-2025）

| 指标 | 数据 |
|------|------|
| 总预测数 | 20 |
| 正确 | 2 |
| 错误 | 16 |
| 待验证 | 2 |
| 成功率 | ~10% |

**典型失败案例**：
1. **2024年9月**：预测"BTC将在本周末跌破5万美元"——实际最低仅到52,546美元，此后再未触及更低价格
2. **2025年3月**：预测"BTC将先触达11万美元再重新测试76,500美元"——实际BTC在4月7日重新测试76,500美元，但未能在此期间达到11万美元
3. **2025年1月**：看涨7个代币（BIO、VITA、ATH、GROW、PSY、CRYO、NEURON）——结论是"七次失败"

**Hayes的自我评估**：
- 承认过去一年8次预测中只有2次正确
- 自评命中率为25%
- 坦言自己的短期预测准确率"对普通人来说相当糟糕"（"pretty shit to the common man"）

### 6.3 预测框架的优劣势

**优势**：
- 宏观方向上的判断有时非常精准（如对央行印钞推动加密市场的长期判断）
- 对加密市场周期的结构性理解深刻
- 产品创新能力卓越（永续合约已被验证为改变行业的发明）

**劣势**：
- 短期价格预测准确率极低
- 具体代币推荐的表现经常不佳
- 时间节点的判断经常出错（方向对但时间不对）

### 6.4 2026年展望

- Hayes预测2026年将是加密市场的关键年
- 重点看好隐私币和零知识证明（ZK）技术
- Maelstrom基金正积极布局AI和隐私相关的加密资产
- 同时关注黄金和国防股票作为多元化配置

---

## 七、决策模式总结

### Hayes的决策特征

1. **反叛精神**：从选择塞舌尔注册到公开嘲讽监管机构，Hayes始终展现出对传统金融体制的挑战态度
2. **产品天才，预测平庸**：永续合约是加密行业最成功的金融创新之一，但短期市场预测极不准确
3. **叙事大师**：擅长通过写作构建和传播市场叙事，将自己从交易所创始人转型为加密思想领袖
4. **风险偏好极高**：无论是100倍杠杆的产品设计、无KYC的运营策略，还是"几乎最大风险"的投资仓位，Hayes始终倾向于激进策略
5. **长期正确，短期失误**：宏观判断的方向性通常正确，但具体时间点和标的选择经常出错
6. **韧性强**：经历DOJ起诉、认罪、缓刑后成功转型，影响力不降反升

---

## 参考来源

- [Arthur Hayes - Wikipedia](https://en.wikipedia.org/wiki/Arthur_Hayes_(banker))
- [Who is Arthur Hayes? Crypto Derivatives Pioneer & BitMEX Co-Founder](https://whales.market/blog/who-is-arthur-hayes-crypto-derivatives-pioneer-bitmex-co-founder/)
- [Arthur Hayes: Early Life and Net Worth - Medium/Coinmonks](https://medium.com/coinmonks/arthur-hayes-early-life-and-net-worth-the-vision-behind-bitmex-and-the-evolution-of-crypto-1416d686cddf)
- [The Token Dispatch: Arthur Hayes The Derivatives Genius](https://www.thetokendispatch.com/p/arthur-hayes-the-derivatives-genius)
- [Adapt or Die - Arthur Hayes Substack](https://cryptohayes.substack.com/p/adapt-or-die)
- [BitMEX Founders Plead Guilty - CoinDesk](https://www.coindesk.com/policy/2022/02/24/bitmex-founders-arthur-hayes-ben-delo-plead-guilty-to-violating-us-law)
- [Former BitMEX CEO Sentenced to 2 Years Probation - CoinDesk](https://www.coindesk.com/policy/2022/05/20/former-bitmex-ceo-arthur-hayes-sentenced-to-2-years-probation)
- [CFTC Press Release: BitMEX Founders Ordered to Pay $30 Million](https://www.cftc.gov/PressRoom/PressReleases/8522-22)
- [DOJ: BitMEX Fined $100 Million for Violating Bank Secrecy Act](https://www.justice.gov/usao-sdny/pr/global-cryptocurrency-exchange-bitmex-fined-100-million-violating-bank-secrecy-act)
- [BitMEX Pleads Guilty to BSA Offense - DOJ](https://www.justice.gov/usao-sdny/pr/global-cryptocurrency-exchange-bitmex-pleads-guilty-bank-secrecy-act-offense)
- [We Check the Accuracy of 20 Arthur Hayes Market Calls - Protos](https://protos.com/we-calculated-the-accuracy-of-20-market-calls-by-arthur-hayes/)
- [Arthur Hayes Rates His Market Prediction Accuracy as 'Pretty Shit' - Decrypt](https://decrypt.co/283282/bitcoin-billionaire-arthur-hayes-rates-his-market-prediction-accuracy-as-pretty-shit)
- [Arthur Hayes' Maelstrom Enters 2026 at 'Almost Maximum Risk' - CoinDesk](https://www.coindesk.com/markets/2026/01/06/arthur-hayes-maelstrom-enters-2026-at-almost-maximum-risk-betting-on-altcoins)
- [Arthur Hayes' Maelstrom Targets $250M PE Fund - Yahoo Finance](https://finance.yahoo.com/news/arthur-hayes-maelstrom-targets-250-170656391.html)
- [Arthur Hayes Net Worth and On-Chain Holdings - Arkham Intelligence](https://info.arkm.com/research/arthur-hayes-net-worth-crypto-holdings-2025)
- [Maelstrom Capital Portfolio - RootData](https://www.rootdata.com/Investors/detail/Maelstrom%20Capital?k=MTE4NTM%3D)
- [Arthur Hayes Bitcoin $1M Prediction - The Block](https://www.theblock.co/post/352482/arthur-hayes-repeats-1-million-bitcoin-price-prediction-by-2028)
- [Crypto Trader Digest - Arthur Hayes Substack](https://cryptohayes.substack.com/)
- [Chief Story Officer - Arthur Hayes Substack](https://cryptohayes.substack.com/p/chief-story-officer)
- [Wharton FinTech Interview with Arthur Hayes](https://medium.com/wharton-fintech/wharton-fintech-interview-with-arthur-hayes-ceo-and-co-founder-of-bitmex-55a0821b41d7)
- [Arthur Hayes BitMEX Dominance - Market Insiders](https://marketinsiders.in/2025/09/17/arthur-hayes-bitmex-dominance/)
- [From Food Delivery Arbitrage to Multi-Billion Dollar Exchange - Odaily](https://www.odaily.news/en/post/5205860)
- [Arthur Hayes: Crypto Markets, Geopolitics, and Web3 Predictions - Validation Cloud](https://blog.validationcloud.io/high-stakes-arthur-hayes)
