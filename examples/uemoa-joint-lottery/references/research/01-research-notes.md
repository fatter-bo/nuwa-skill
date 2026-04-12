# UEMOA区域联合乐透 — 调研笔记

> 调研日期：2026-04-12
> 调研方法：Web深度检索，覆盖EuroMillions/Powerball/Viking Lotto运营模式、UEMOA现有彩票生态、法律框架、技术架构等

---

## 1. EuroMillions经验调研

### 1.1 基本信息
- **构想时间**：1994年，由法国Francaise des Jeux（FDJ）高管提出
- **首期开奖**：2004年2月13日（巴黎）
- **初始参与国**：3国（法国、西班牙、英国）
- **2004年10月扩展**：新增奥地利、比利时、爱尔兰、卢森堡、葡萄牙、瑞士（共9国）
- **筹备到首期**：~10年（1994构想 -> 2004首期）
- **首期头奖**：EUR 1500万（法国玩家赢得）

### 1.2 治理架构
- 不设独立跨国法人实体
- 由9家国家彩票运营商组成联合体协作运营
- 各运营商负责本国票务、营销、兑奖
- 开奖统一在巴黎进行，各国代表监督
- 重大规则变更需所有参与运营商同意

### 1.3 资金管理——Trust账户机制
- 各参与运营商设立EuroMillions Trust账户
- 用于跨国奖金清算和未来奖金储备
- Trust机制保护各方免受单一运营商违约风险
- 玩家利益通过Trust架构获得保障

### 1.4 奖池与收入
- 票面收入的**50%**进入奖池
- 13个奖级，Pari-mutuel分配
- 头奖分配：前5期50%，第6期起42%
- 储备金：前5期10%，第6期起18%
- 头奖上限EUR 2.5亿（历史最高EUR 2.4亿，2023年）
- 估计年总销售额EUR 80-100亿（9国合计，无官方公开统一数据）
- 法国、西班牙、英国、葡萄牙各约占20%销售份额

### 1.5 产品设计
- **号码池**：5/50（主号）+ 2/12（Lucky Stars）
- **头奖概率**：1:139,838,160
- **开奖频率**：每周二+每周五
- **奖级**：13级（从5+2到2+0）

### 1.6 附加产品
- UK: Millionaire Maker（百万英镑Raffle抽奖，自动附加，无额外费用）
- 西班牙: El Millon（百万欧元Raffle）
- 法国: My Million（百万欧元Raffle）
- 各国可设计本地附加产品，增加本地吸引力

### 来源
- [EuroMillions Wikipedia](https://en.wikipedia.org/wiki/EuroMillions)
- [EuroMillions Prizes & Distribution](https://www.euro-millions.com/prizes)
- [EuroMillions Countries](https://www.rincondefortuna.es/blog/Euromillions-Countries.htm)
- [Who Owns EuroMillions](https://www.worldlotteryhubs.com/archives/6768)
- [UK Millionaire Maker](https://www.national-lottery.com/euromillions/millionaire-maker)
- [EuroMillions History](https://euromillionswins.com/history.html)
- [Springer: EuroMillions Market & Fund Transfer](https://link.springer.com/article/10.1007/s40888-023-00316-9)
- [UK Gambling Commission - EuroMillions Regulation](https://assets.ctfassets.net/j16ev64qyf6l/5qYF3E2oywmw82mWdvozqO/6fbc6fb9128bf0d9f41cecb834bbfe4c/23_05_04_-_Section_6_EuroMillions_and_UK_Millionaire_Maker_Issue3_v6.pdf)

---

## 2. US Powerball / MUSL调研

### 2.1 MUSL治理模式
- **全称**：Multi-State Lottery Association（多州彩票协会）
- **性质**：非营利、政府利益协会，由成员彩票机构拥有和运营
- **覆盖**：45州 + DC + 波多黎各 + 维尔京群岛
- **Powerball推出**：1992年（MUSL成立于1987年）

### 2.2 协调机制
- MUSL提供：游戏设计、财务管理、开奖制作、IT/安全标准、供应商审查
- MUSL集中职能：开奖执行、游戏开发研究、中央会计、奖金基金持有与转移、政府证券/年金购买
- 利润由各州彩票保留，用于州立法机构批准的项目
- 重大变更需所有成员同意

### 2.3 Powerball产品
- **号码池**：5/69（白球）+ 1/26（Powerball）
- **头奖概率**：1:292,201,338
- **附加产品**：Power Play倍增器（2x-10x非头奖奖金）
- **历史最高头奖**：USD 20.4亿（2022年）

### 2.4 对UEMOA的启示
- MUSL的非营利协会模式适合UEMOA语境
- 各成员保留利润自主权是政治可行性的关键
- 中央化的游戏设计+分散化的运营+利润留各方 = 成功公式

### 来源
- [MUSL Wikipedia](https://en.wikipedia.org/wiki/Multi-State_Lottery_Association)
- [MUSL Official](https://www.musl.com/)
- [Powerball Wikipedia](https://en.wikipedia.org/wiki/Powerball)
- [MUSL Legal Overview](https://legal-resources.uslegalforms.com/m/multi-state-lottery-association-musl)

---

## 3. Viking Lotto调研

### 3.1 基本信息
- **首期**：1993年3月17日
- **首创意义**：欧洲第一个跨国彩票
- **创始国**：5国（挪威、丹麦、芬兰、瑞典、冰岛），人口~2700万
- **扩展历程**：
  - 2000年：爱沙尼亚加入
  - 2011年：拉脱维亚、立陶宛加入
  - 2017年：斯洛文尼亚加入
  - 2020年：比利时加入
  - 现有10国参与

### 3.2 治理
- Viking Lotto Block：创始组织
- 首任主席：Reidar Nordby（Norsk Tipping）
- 开奖地点：挪威Hamar（Norsk Tipping总部）
- 开奖时间：每周三20:00 CET

### 3.3 产品设计
- **号码池**：6/48（主号）+ 1/5（Viking Number，从独立池抽取）
- **头奖概率**：1:98,172,096（部分来源引用1:61,357,560，可能因规则变更）
- **头奖上限**：EUR 3500万
- **最低保证头奖**：EUR 300万

### 3.4 奖金分层机制
- **国际层**：头奖 + 二等奖 = 跨国Pari-mutuel共享
- **国家层**：三等奖及以下 = 各国用剩余资金自行决定
- 这种分层大幅降低协调复杂度
- 比利时甚至增加了匹配1号+Viking Number也能获奖的本地奖级

### 3.5 对UEMOA的启示
- **最佳可比案例**：初始5国~2700万人口，与UEMOA单国或3-4国组合规模相当
- 证明小市场能通过联合制造EUR 300-3500万的吸引力头奖
- Block协议模式（非独立法人）降低启动门槛
- 分层奖金设计（国际+本地）是降低协调成本的关键创新
- 渐进式扩展（5->10国，历时27年）证明"小起步、稳扩展"策略可行

### 来源
- [Vikinglotto Wikipedia](https://en.wikipedia.org/wiki/Vikinglotto)
- [Viking Lotto History](https://www.vikinglottery.com/History.php)
- [Vikinglotto Odds](https://viking-lotto.net/en/odds)
- [Vikinglotto How to Play](https://viking-lotto.net/en/how-to-play)
- [Vikinglotto Participating Countries](https://viking-lotto.net/en/participating-countries)

---

## 4. Masse Commune UEMOA调研

### 4.1 概况
- 全称：Masse Commune UEMOA
- 性质：UEMOA 8国的年度联合赛马投注活动
- 产品：联合PMU Quinté（五重彩赛马投注）
- 2025年为第22届（即始于约2003年）
- 22届由科特迪瓦LONACI主办（2025年5月5-9日，阿比让）
- 下一届（2026年）计划在贝宁科托努举办

### 4.2 运作机制
- 各参与国的彩票/PMU公司将选定赛事的净营业额的**60%**汇集为共享奖池
- 统一选定一场赛马支撑Quinté投注
- 所有UEMOA区域的投注者有相同的赢大奖机会
- 100百万XOF特别奖池（Quinté Ordre）

### 4.3 参与国
- 2025年22届参与：科特迪瓦、塞内加尔、贝宁、马里、布基纳法索、尼日尔
- 注意：并非每届都有全部8国参与（几内亚比绍和多哥参与度不稳定）

### 4.4 轮值主办制
- 每年由不同成员国轮流主办
- 主办国负责活动组织、赛事选择、现场协调
- 具有促进区域一体化和团结的使命

### 4.5 局限性（从联合乐透角度）
- 年度1次 -> 无法积累品牌认知和玩家习惯
- 赛马投注而非数字乐透 -> 产品类型不同
- 手工协调 -> 无法支撑高频运营
- 无常设机构 -> 连续性和专业性不足
- 但证明了UEMOA各国有联合投注的意愿和操作经验

### 来源
- [LONACI 22e Masse Commune](https://news.abidjan.net/articles/741462/loterie-masse-commune-uemoa-22e-edition-une-cagnotte-speciale-de-100-millions-sur-lordre-du-quinte-ce-jeudi)
- [22nd UEMOA Common Mass Abidjan](https://focusgn.com/africa/uemoa-common-mass-kicks-off-in-abidjan-with-record-jackpot)
- [Masse Commune cap sur Cotonou 2026](https://www.maliweb.net/societe/masse-commune-uemoa-apres-abidjan-cap-sur-cotonou-en-mai-2026-3105020.html)
- [AIP LONACI 22e edition](https://www.aip.ci/189188/cote-divoire-aip-la-lonaci-accueille-la-22e-edition-de-la-masse-commune-uemoa-a-abidjan/)
- [Seneweb Masse Commune](https://www.seneweb.com/news/Economie/masse-commune-uemoa-les-societes-de-lote_n_305438.html)

---

## 5. UEMOA国家彩票运营商调研

### 5.1 各国运营商概况

| 国家 | 运营商 | 所有权 | 年收入（估） | 产品 |
|------|--------|--------|------------|------|
| 科特迪瓦 | LONACI | 国家80%+社保15%+员工5% | XOF 6576亿（EUR 10亿，2025） | 乐透、即开票、PMU、Sport Cash |
| 塞内加尔 | LONASE | 国家100% | 数据有限 | 乐透、即开票、PMU |
| 布基纳法索 | LONAB | 国家100% | 数据有限 | 乐透、即开票、PMU |
| 马里 | PMU-Mali | 国家控股 | 数据有限 | 主要PMU |
| 贝宁 | LNB | 国家控股 | 数据有限 | 乐透、PMU |
| 多哥 | LONATO | 国家控股 | 数据有限 | 乐透、PMU |
| 尼日尔 | LONANI | 国家控股 | 数据有限 | 乐透、PMU |
| 几内亚比绍 | 待确认 | -- | 数据有限 | -- |

### 5.2 科特迪瓦LONACI——区域领军者
- 2025年营业额：XOF 6576亿（约EUR 10亿）
- 2023年净利润：XOF 40亿+
- 2026年营收目标：XOF 7443亿（约EUR 11.4亿）
- 向玩家返还：XOF 4139亿（约EUR 6.32亿，占比~63%）
- 2020年新博彩法通过，更新了50年前的法律框架
- 2023年新的行政令进一步打击非法博彩
- 80%国有确保政府对博彩业的控制

### 5.3 塞内加尔LONASE
- 100%国有
- 覆盖全国1200+销售网点，14个代理机构+6个办事处
- 法国PMU在2022年将赛马投注产品引入塞内加尔市场

### 5.4 布基纳法索LONAB
- 100%国有
- 2025年预算统一了博彩税率
- 曾派代表团到LONACI学习法规改革经验
- 2026年战略规划中

### 来源
- [LONACI Net Income 2023](https://gamblingtalk.net/news/ivory-coasts-lonaci-achieves-over-4-billion-fcfa-in-net-income-for-fiscal-year-2023)
- [LONACI 2026 Target](https://focusgn.com/africa/cote-divoires-national-lottery-targets-fcfa5bn-capital-boost-as-lonaci-sets-2026-course)
- [LONASE-LONACI Cooperation](https://gamblingtalk.net/news/ivory-coast-and-senegal-strengthen-lottery-ties-lonase-delegation-visits-abidjan)
- [LONAB Strategic 2026](https://focusgn.com/africa/burkina-fasos-national-lottery-gears-up-for-a-strategic-2026)
- [Burkina Faso Tax Rate](https://gamblingtalk.net/news/burkina-faso-unifies-gambling-tax-rate-in-2025-budget-approval)
- [PMU Senegal](https://www.gamingintelligence.com/products/racing/154820-pmu-brings-horse-race-betting-offering-to-senegal/)
- [LONAB Réforme](https://news.abidjan.net/articles/709186/reforme-du-secteur-des-jeux-de-hasard-la-lonaci-recoit-une-delegation-burkinabe-en-mission-detude)

---

## 6. 法律与监管框架调研

### 6.1 UEMOA层面
- **无统一博彩/彩票指令**：UEMOA尚未发布专门的博彩业统一指令
- **财税协调**：UEMOA有多项税收协调指令（1998年起），博彩税被归入"特定服务税"类别，但未专门立法
- **OHADA**：非洲商法统一组织提供区域性商法框架，可用于争议解决
- **BCEAO**：西非央行管辖资金跨境流动，Trust账户需其配合

### 6.2 科特迪瓦法律框架（区域最完善）
- 原法律：1970年两部法律（Loi n.70-208 创设国家彩票 + Loi n.70-575 禁止私人彩票）
- 2020年：全新博彩法框架（Loi n.2020-480）
- 2023年：新行政令强化非法博彩打击
- LONACI受财政部监管

### 6.3 布基纳法索
- 2025年预算中统一了博彩税率
- 正在参考LONACI经验进行法规改革
- LONAB受财政部+内政部双重监管

### 6.4 冈比亚（非UEMOA）
- 2015年全面禁止博彩，2017年重新合法化
- 冈比亚税务局（GRA）负责监管
- 冈比亚国民被禁止进入赌场（特殊限制）
- 线上博彩无具体法规，处于监管灰色地带
- 使用GMD（冈比亚达拉西），非XOF

### 6.5 法律框架建设的关键挑战
1. 8国法律差异显著（从科特迪瓦的现代法律到几内亚比绍的薄弱框架）
2. UEMOA无博彩领域的"acquis communautaire"（共同体法律体系）
3. 各国对博彩的社会/宗教态度不同（尤其马里、尼日尔的穆斯林人口占比高）
4. 跨国奖金支付涉及BCEAO外汇管制
5. 反洗钱合规需跨国协调

### 来源
- [Cote d'Ivoire Loi Jeux de Hasard](http://www.droit-afrique.com/uploads/RCI-Loi-2020-480-regime-juridique-jeux-hasard.pdf)
- [LONACI Nouveau Decret](https://www.koaci.com/article/2023/12/11/cote-divoire/societe/cote-divoire-nouveau-decret-sur-les-jeux-de-hasard-le-dg-de-la-lonaci-salue-une-decision-qui-ouvre-la-voie-a-des-actions-plus-efficaces-contre-les-jeux-illegaux_174316.html)
- [7info Nouveau Cadre Juridique](https://www.7info.ci/un-nouveau-cadre-juridique-pour-les-jeux-de-hasard/)
- [Burkina Faso Tax Unification](https://gamblingtalk.net/news/burkina-faso-unifies-gambling-tax-rate-in-2025-budget-approval)
- [CMS Gambling Laws Africa](https://cms.law/en/int/expert-guides/cms-expert-guide-to-gambling-laws-in-africa)
- [Gambia Gaming Regulation](https://www.igamingtoday.com/gaming-regulation-in-gambia/)
- [Gambia Gambling Sites](https://www.gamingzion.com/gambia/gambling/gambling-sites/)
- [UEMOA Fiscal Coordination IMF](https://www.imf.org/external/french/np/seminars/2014/waemu/pdf/traores2.pdf)
- [Cairn: Jeux d'Argent Afrique de l'Ouest](https://shs.cairn.info/revue-afrique-contemporaine1-2019-1-page-323?lang=fr)

---

## 7. 号码池与概率计算

### 7.1 计算方法
组合数公式：C(n,k) = n! / (k! x (n-k)!)

### 7.2 各方案概率

**方案A: 5/45+1/10**
- C(45,5) = 1,221,759
- 附加池: 10
- 总组合: 12,217,590
- 头奖概率: 1:12,217,590

**方案B: 5/50+1/12**
- C(50,5) = 2,118,760
- 附加池: 12
- 总组合: 25,425,120
- 头奖概率: 1:25,425,120

**方案C: 6/49**
- C(49,6) = 13,983,816
- 头奖概率: 1:13,983,816

### 7.3 参考对比

| 彩票 | 头奖概率 | 覆盖人口 | 备注 |
|------|---------|---------|------|
| EuroMillions 5/50+2/12 | 1:139,838,160 | ~3.5亿 | 双附加号大幅提高难度 |
| Powerball 5/69+1/26 | 1:292,201,338 | ~3.3亿 | 最难的主流彩票 |
| Viking Lotto 6/48+1/5 | 1:98,172,096 | ~2700万(初始) | 对小市场偏难 |
| Canada Lotto 6/49 | 1:13,983,816 | ~3800万 | 经典参考 |
| 方案A 5/45+1/10 | 1:12,217,590 | 适配3000万-8000万 | UEMOA Phase 1 |
| 方案B 5/50+1/12 | 1:25,425,120 | 适配8000万-3亿 | UEMOA Phase 2-3 |

### 7.4 滚存周期估算

假设UEMOA 1.3亿人口，2%初始参与率 = 260万张/期：
- 方案A: 260万/1222万 = 0.213 -> 每4.7期出一次头奖
- 方案B: 260万/2543万 = 0.102 -> 每9.8期出一次头奖
- 方案C: 260万/1398万 = 0.186 -> 每5.4期出一次头奖

如果参与率仅1%（130万张/期）:
- 方案A: 130万/1222万 = 0.106 -> 每9.4期
- 方案B: 130万/2543万 = 0.051 -> 每19.6期
- 方案C: 130万/1398万 = 0.093 -> 每10.8期

### 来源
- [Lottery Mathematics Wikipedia](https://en.wikipedia.org/wiki/Lottery_mathematics)
- [Lotterycodex Formula](https://lotterycodex.com/lottery-formula/)
- [Omnicalculator Lottery](https://www.omnicalculator.com/statistics/lottery)
- [BFI Equilibrium Model Rollover Lotteries](https://bfi.uchicago.edu/wp-content/uploads/2024/03/BFI_WP_2024-34.pdf)

---

## 8. 技术架构调研

### 8.1 跨国彩票技术要求
- 中央服务器作为权威数据源
- 实时投注数据聚合
- 各参与节点通过安全数据链路连接中央系统
- 实时奖池计算和显示更新
- 每次投注自动按预设比例汇入中央奖池
- 跨币种自动换算（对UEMOA来说XOF统一，简化了这一需求）
- 时区处理（UEMOA 8国跨越GMT+0到GMT+1，时差很小）

### 8.2 技术商评估
- **IGT**：全球最大彩票技术商，总部伦敦，在非洲有部署
- **Intralot**：希腊公司，在非洲多国运营，了解本地环境
- **Scientific Games**：即开票+系统整合，全面解决方案
- **GammaStack**：彩票管理软件提供商，可扩展方案
- **GENLOT**：全渠道彩票解决方案

### 8.3 UEMOA特有技术挑战
- 网络基础设施不均衡（尼日尔、几内亚比绍较弱）
- 电力供应不稳定 -> 需UPS和离线模式
- 移动渗透率高但智能手机普及率不均 -> USSD购票需求
- 移动货币（Orange Money, MTN Money等）作为支付渠道

### 来源
- [GammaStack Lottery Software](http://www.gammastack.com/lottery-management-software)
- [GENLOT Omnichannel](https://www.genlot.com/lottery-omnichannel-solution/)
- [IGT Official](https://www.igt.com/)
- [Intralot Official](https://www.intralot.com/)

---

## 9. UEMOA区域背景数据

### 9.1 成员国
贝宁、布基纳法索、科特迪瓦、几内亚比绍、马里、尼日尔、塞内加尔、多哥

### 9.2 人口与经济
- 总人口：约1.37-1.52亿（来源不同，取~1.3亿保守估计）
- 总面积：351万平方公里
- GDP：约USD 1750亿
- 货币：XOF（西非法郎），与EUR固定汇率：1 EUR = 655.957 XOF
- 央行：BCEAO（西非国家中央银行）

### 9.3 GDP per capita差异
- 科特迪瓦：USD 1,950（最高）
- 尼日尔：USD 484（最低）
- 差距约4倍，对联合乐透的票价设计和参与率有重要影响

### 9.4 冈比亚
- 非UEMOA成员
- ECOWAS成员
- 人口：~250万
- 货币：GMD（冈比亚达拉西）
- 地理上被塞内加尔完全环绕

### 来源
- [UEMOA Wikipedia](https://en.wikipedia.org/wiki/West_African_Economic_and_Monetary_Union)
- [UEMOA Official Presentation](https://www.uemoa.int/en/presentation-of-uemoa)
- [UEMOA Worlddata](https://www.worlddata.info/trade-agreements/uemoa-west-africa-economy.php)
- [UNCTAD WAEMU Growth](https://unctad.org/news/unctad-charts-path-inclusive-growth-west-african-economic-and-monetary-union)
- [WAEMU Housing Finance Africa](https://housingfinanceafrica.org/wp-content/uploads/2025/01/WAEMU.pdf)

---

## 10. 关键洞察与结论

### 10.1 核心发现

1. **EuroMillions花了10年从构想到首期**，但UEMOA有Masse Commune基础，可以加速。目标18个月内Phase 1启动是激进但可行的。

2. **Viking Lotto是最佳参照**：初始5国~2700万人口的成功案例证明小市场联合可行。其"上层共享+下层自主"的分层奖金设计是降低协调成本的核心创新。

3. **号码池不宜照搬EuroMillions**（5/50+2/12的1:1.4亿概率对UEMOA太难），建议从5/45+1/10起步（1:1200万），随参与率增长逐步扩大。

4. **LONACI是区域领军者**（年营收EUR 10亿），自然应成为联合乐透的推动者。科特迪瓦+塞内加尔的双核驱动是Phase 1的最佳起点。

5. **法律框架是最大障碍**：UEMOA无统一博彩指令，需要"商业协议先行+政府间协议跟进+UEMOA指令收尾"的三阶段法律策略。

6. **共同货币XOF是天然优势**：相比EuroMillions需处理3种货币、Viking Lotto需处理多种北欧货币，UEMOA的XOF统一大幅简化了奖池管理和跨国清算。

7. **Trust账户机制是必须的**：EuroMillions的经验表明，Trust账户是跨国彩票信任基础的关键，UEMOA联合乐透必须建立类似机制。

8. **移动优先策略**：鉴于UEMOA的移动渗透率远高于传统零售网络覆盖，USSD购票+移动货币支付应作为核心渠道。

9. **Masse Commune不是障碍而是资产**：22年的联合经验、运营商之间的关系网络、轮值主办传统——这些都是联合乐透的"启动资本"。

10. **冈比亚应推迟到Phase 3**：非UEMOA成员、不同货币、不同法律框架——引入冈比亚的收益（+250万人口）不值得Phase 1-2的额外复杂度。
