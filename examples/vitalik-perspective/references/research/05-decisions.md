# Vitalik Buterin 重大决策记录

> 调研日期：2026-04-12
> 涵盖范围：2013年至2026年期间 Vitalik Buterin 在以太坊生态中的关键决策、技术选择、公开理由与实际结果。

---

## 一、2013-2014年：创建以太坊

### 1.1 为什么不在 Bitcoin 上构建？

Vitalik Buterin 最初活跃于比特币社区，曾于2011年联合创办《Bitcoin Magazine》。他向 Bitcoin Core 开发者提议：区块链技术不应仅用于货币，应支持更广泛的应用开发，如将股票、房产等现实资产上链。

**比特币的根本局限性：**
- 比特币采用故意简化的、非图灵完备的脚本语言
- 这种设计虽然安全，但几乎不可能在其上构建复杂应用
- Buterin 的提案遭到 Bitcoin Core 社区拒绝

**核心创新决策：** Buterin 决定不再修补比特币，而是从零构建全新平台。2013年11月，他发布了以太坊白皮书，提出了一个通用的、图灵完备的智能合约平台。

**白皮书的核心创新：**
- 图灵完备的智能合约语言（后来的 Solidity）
- 以太坊虚拟机（EVM）作为全球去中心化计算平台
- 不仅仅是"可编程货币"，而是"可编程世界计算机"
- 支持去中心化应用（DApps）、去中心化组织（DAOs）、金融衍生品等

### 1.2 退学与 Thiel Fellowship

Buterin 在多伦多长大，高中毕业后进入滑铁卢大学（University of Waterloo）学习计算机科学，师从密码学家 Ian Goldberg 担任研究助理。

**关键时间线：**
- 2013年：在滑铁卢大学就读期间撰写以太坊白皮书
- 2014年：获得 Peter Thiel 创立的 Thiel Fellowship，奖金10万美元
- 随即退学，全职投入以太坊开发

**Buterin 的自述：** 他坦言退学"实际上很可怕"（actually was scary），担心会在某些知识领域落后。但 Thiel Fellowship 为他已经决心追求的事业提供了经济支持。

**实际结果：** 这一决策被证明极为成功。以太坊成为仅次于比特币的第二大区块链平台，Buterin 后来获得了巴塞尔大学荣誉博士学位。

---

## 二、以太坊 ICO（2014年）

### 2.1 融资结构

**ICO 基本信息：**
- 开始日期：2014年7月22日
- 持续时间：42天（至9月2日结束）
- 筹集资金：约31,600 BTC（约1,840万美元）
- 售出代币：约6,000万 ETH

**ETH 预挖分配：**
- 公开销售：约60,000,000 ETH（约83.3%）
- 联合创始人及早期团队：占公开销售量的9.9%
- 以太坊基金会：同样占公开销售量的9.9%
- 总初始供应量推至约7,200万 ETH，团队控制至少16.7%

**Vitalik 设计的分配系统：** 基于个人加入项目的日期和贡献工时来计算分配额度。

**规则与漏洞：**
- 基金会不允许参与众筹，以避免中心化嫌疑
- 基金会在预售期间最多只能提取5,000 BTC用于加速开发
- 但捐赠接收者（联合创始人）可以在销售中购买更多 ETH，只要不超过总供应量的12.5%
- 实际上这一限制无法执行，且存在激励扭曲——联合创始人有动力在销售中大量购入

**争议：** 在以太坊之前，几乎任何带有预挖的加密货币项目都会被快速认定为骗局。以太坊的预挖规模在当时引发了广泛质疑。

### 2.2 联合创始人之间的分歧

以太坊共有8位联合创始人，其中最关键的冲突发生在 Vitalik Buterin、Charles Hoskinson 和 Gavin Wood 之间。

**核心分歧：营利性 vs 非营利性**
- **Charles Hoskinson** 主张以太坊应走商业化路线，成为类似 Google 的科技公司
- **Vitalik Buterin** 坚持以太坊应效仿 Mozilla，成为非营利组织
- **Gavin Wood** 在此问题上向 Vitalik 发出最后通牒："要么是 Charles，要么是我"

**决定性时刻——2014年6月7日瑞士楚格会议：**
- 联合创始人聚集在楚格，准备签署文件将以太坊转型为营利性公司
- Vitalik 在阳台独处一段时间后返回
- 宣布 Hoskinson 和 Amir Chetrit 被移除，以太坊将成为非营利基金会

**后续发展：**
- Charles Hoskinson 离开后创立了 Cardano（2017年启动）
- Gavin Wood 后来也离开，创立了 Polkadot 和 Web3 Foundation
- 讽刺的是，Hoskinson 的 Cardano 基金会本身也是非营利组织

**决策评价：** Buterin 选择非营利路线被普遍认为是正确的。这一决策让以太坊保持了"公共基础设施"的定位，避免了利益冲突，奠定了社区治理的基调。

---

## 三、2016年：The DAO Hack 和硬分叉决策

### 3.1 事件背景

The DAO（去中心化自治组织）于2016年在以太坊上线，通过代币销售筹集了约1.5亿美元的 ETH，是当时最大的众筹项目。

**黑客攻击：** 上线不到三个月，The DAO 因代码中的重入漏洞（reentrancy vulnerability）被黑客攻击，约6,000万美元的 ETH 被盗。

### 3.2 Vitalik 的角色与决策过程

Vitalik Buterin 本人并未直接参与 The DAO 项目的开发。

**解决方案演进：**
1. **软分叉方案（第一选择）：** Buterin 最初建议通过软分叉添加代码，将黑客地址列入黑名单并冻结资金。但黑客（或冒充黑客的人）在 DAO Slack 中警告将奖励不配合软分叉的矿工，且技术上发现软分叉方案存在漏洞。
2. **硬分叉方案（最终选择）：** 社区投票决定实施硬分叉，将网络状态回滚至攻击前，将 ETH 归还投资者。

**2016年7月20日，第192,000区块：** 以太坊硬分叉正式执行。

### 3.3 "Code is Law" vs 实际操作

**这是以太坊历史上最具争议的决策。**

**支持硬分叉的理由：**
- 保护投资者，挽回损失
- The DAO 占当时以太坊总市值的约15%，不救可能致命
- 社区投票多数支持

**反对硬分叉的理由：**
- 违背区块链不可变性（immutability）的核心原则
- 破坏"代码即法律"的社会契约
- 开创了人为干预区块链历史的危险先例
- 如果大到不能倒，那和传统金融有何区别？

**实际结果：**
- 大多数利益相关者接受了硬分叉
- 拒绝接受的一方继续运行原链，命名为 **Ethereum Classic（ETC）**
- 当前的以太坊（ETH）是实施了硬分叉的版本
- 此后以太坊再未进行过类似的"回滚式"硬分叉

**长期影响：** 这一决策虽然在当时极具争议，但从实际效果看巩固了社区信心，以太坊此后蓬勃发展。而 Ethereum Classic 虽然存活至今，但市场影响力远不及 ETH。这也成为区块链治理领域最经典的案例研究之一。

---

## 四、2020-2022年：以太坊 2.0 / The Merge（PoW 转 PoS）

### 4.1 技术路线选择

从 Proof of Work（PoW）转向 Proof of Stake（PoS）实际上是 Buterin **从一开始就计划好的**，但实际实现花了近七年的规划和测试。

**为什么选择 PoS 而非继续 PoW？**

1. **能源效率：** PoW 以太坊单笔交易消耗的能源约等于美国普通家庭一周的用电量。转为 PoS 后，能源消耗降低约99.95%。
2. **可扩展性：** PoS 为后续的分片（sharding）等扩容方案提供了基础架构
3. **安全性改进：** PoS 通过经济惩罚（slashing）机制约束验证者行为
4. **去中心化改善：** 降低了参与网络验证的硬件门槛

### 4.2 The Merge 实施

**关键时间线：**
- 2020年12月1日：Beacon Chain（信标链）启动，PoS 链开始独立运行
- 2022年8月：Buterin 预估 Merge 将在当年8月完成，如遇复杂情况推迟至10月
- **2022年9月15日：The Merge 正式完成**

**技术方案：** Vitalik 提出将 Ethereum 1.0（执行层）与 Ethereum 2.0（共识层/Beacon Chain）"合并"，原 PoW 链作为"shard 0"并入新的 PoS 架构。

**实际结果：**
- Merge 在技术上极为成功，零宕机时间完成
- 以太坊能源消耗骤降99.95%
- ETH 发行量大幅减少（"超声波货币"叙事）
- 未出现重大安全事件
- 但 PoS 后以太坊的中心化风险（如 Lido 等流动性质押协议的主导地位）成为新的担忧

---

## 五、以太坊路线图演变

### 5.1 Rollup-Centric 路线图

2020年10月，Buterin 发布了"以 Rollup 为中心"的路线图，标志着以太坊扩容策略的根本性转变：

- **核心理念：** 以太坊 L1 专注于安全性和数据可用性，将执行和计算卸载至 L2 Rollups
- **关键技术：** EIP-4844（Proto-Danksharding）为 Rollups 提供廉价的数据可用性层
- Buterin 在2023年强调：Rollups、零知识证明、账户抽象和二代隐私解决方案已经变得更加主流

### 5.2 六大阶段

Buterin 将以太坊的长期路线图划分为六个阶段（2024年更新版本与2023年"变化相对较少"）：

| 阶段 | 名称 | 核心目标 |
|------|------|----------|
| 1 | **The Merge** | 稳健的 PoS 共识机制（已完成） |
| 2 | **The Surge** | 实现 L1+L2 合计 100,000 TPS |
| 3 | **The Scourge** | 缓解 MEV（最大可提取价值）风险和流动性质押集中化问题 |
| 4 | **The Verge** | 更轻量的区块验证（Verkle Trees、SNARK 验证） |
| 5 | **The Purge** | 简化协议、消除技术债务、降低参与成本 |
| 6 | **The Splurge** | 修复其余所有事项（账户抽象、EVM 改进等） |

**The Verge 的技术亮点：** Verkle Trees 几乎准备就绪，未来验证区块只需下载少量字节数据、执行几项基本计算并验证一个 SNARK 证明。

---

## 六、近期决策（2024-2026年）

### 6.1 d/acc（防御性加速主义）理念的推广

**核心理念：** d/acc 代表 Defensive/Decentralized/Democratic/Differential Acceleration——主张加速技术发展，但差异化地聚焦于提升防御能力而非攻击能力的技术，以及分散而非集中权力的技术。

**四类防御性技术：**
1. 抵御大型物理威胁（如坦克）
2. 抵御小型物理威胁（如疾病）
3. 抵御明确敌意信息（如欺诈）
4. 抵御模糊敌意信息（如可能的虚假信息）

**历史论据：** Buterin 以瑞士为例——在防御占优的环境中，被认为"乌托邦式"的治理体系更有可能繁荣。

**对 AI 的立场：** Buterin 提出通过责任规则和全球工业硬件"软暂停按钮"来缓解超级智能 AI 的灾难性风险。

**2025年1月回顾：** Buterin 撰文《d/acc: one year later》，重申并深化了这一理念。

### 6.2 对 L2 生态的态度变化

Buterin 在 L2 生态治理问题上做出了明确表态：

- **不再投资 L2 项目：** 明确宣布在"可预见的未来"不会投资任何 L2
- **清仓 L2 代币：** 将出售持有的所有 L2 代币，并将全部收益捐赠给慈善机构
- **动机说明：** "这不是关于利润，而是关于采取更明确的立场和树立标准——我不是某个阴谋的一部分，不会把以太坊协议扭曲到有利于我持有的某些基础设施/L2 代币的方向。"

### 6.3 个人 ETH 处置决策

**持仓变化：**
- 2026年初持有超过240,000 ETH
- 截至调研时降至约224,000 ETH
- 2026年已出售约17,196 ETH
- 2026年1月30日宣布出售16,384 ETH

**出售理由：**
- 称之为他在加密市场低迷期间的"个人份额的紧缩"（own share of the austerity）
- 收益用于以太坊基金会优先推进长期可持续性和核心协议开发
- 自2018年以来未出售并保留过收益，历次出售均用于资助其认为有价值的以太坊项目或更广泛的慈善事业

### 6.4 以太坊基金会领导层变革

**2025年1月：** Buterin 宣布以太坊基金会将经历"重大领导层变革"，强调需要增强 EF 领导层的技术能力。

**关键人事变动：**
- Hsiao-Wei Wang 加入 EF 领导团队（自2017年以来一直在 EF Research 团队）
- 2026年2月：联合执行董事 Tomasz K. Stanczak 宣布月底卸任

**明确边界（2026年3月）：**
- EF 发布官方使命宣言（mandate）
- 核心理念：以太坊存在的意义是作为"逃生舱"（escape hatch）
- EF **不会**从事政治游说、意识形态转向，或在生态中扮演更核心的角色
- Buterin 反对 EF 激进游说监管者的呼声，认为这可能"损害以太坊作为全球中立平台的地位"

---

## 决策风格总结

纵观 Vitalik Buterin 十余年的重大决策，可以提炼出以下模式：

1. **原则驱动但务实：** 坚持去中心化和开放性原则（如选择非营利、不投资 L2），但在必要时愿意做出务实妥协（如 DAO 硬分叉）
2. **长期主义：** PoS 转型从构想到实现花了7年；路线图以十年为尺度规划
3. **技术优先：** 领导层改革强调技术能力；决策基于技术可行性而非市场压力
4. **个人以身作则：** 出售 ETH 用于公益而非个人获利；清仓 L2 代币避免利益冲突
5. **社区治理观：** 既不放弃影响力，也不独断专行；DAO 硬分叉经过社区投票，EF 改革公开讨论
6. **渐进式激进：** 目标大胆（世界计算机、10万 TPS），但执行路径渐进（分阶段路线图、先测试网后主网）

---

## 参考来源

- [Vitalik Buterin - Wikipedia](https://en.wikipedia.org/wiki/Vitalik_Buterin)
- [CoinDesk Turns 10: 2015 - Vitalik Buterin and the Birth of Ethereum](https://www.coindesk.com/consensus-magazine/2023/06/02/coindesk-turns-10-2015-vitalik-buterin-and-the-birth-of-ethereum)
- [Vitalik Buterin at Waterloo | University of Waterloo](https://cs.uwaterloo.ca/news/vitalik-buterin-waterloo)
- [Sale of the Century: The Inside Story of Ethereum's 2014 Premine](https://www.coindesk.com/markets/2020/07/11/sale-of-the-century-the-inside-story-of-ethereums-2014-premine)
- [Decade after Ethereum ICO: Blockchain forensics end double-spending debate - Cointelegraph](https://cointelegraph.com/magazine/ethereum-ico-blockchain-forensics-double-spending-debate/)
- [Launching the Ether Sale | Ethereum Foundation Blog](https://blog.ethereum.org/2014/07/22/launching-the-ether-sale)
- [Who are Ethereum's co-founders and where are they now? - Decrypt](https://decrypt.co/36641/who-are-ethereums-co-founders-and-where-are-they-now)
- [Joe Lubin: The truth about ETH founders split and 'Crypto Google'](https://cointelegraph.com/magazine/joe-lubin-the-truth-about-eth-founders-split-and-crypto-google/)
- [DAO Hack Explained | Gemini](https://www.gemini.com/cryptopedia/the-dao-hack-makerdao)
- [CoinDesk Turns 10: 2016 - How The DAO Hack Changed Ethereum and Crypto](https://www.coindesk.com/consensus-magazine/2023/05/09/coindesk-turns-10-how-the-dao-hack-changed-ethereum-and-crypto)
- [What is Ethereum 2.0? The Merge and transition to PoS explained - Cointelegraph](https://cointelegraph.com/learn/articles/ethereum-upgrades-a-beginners-guide-to-eth-2-0)
- [Ethereum Roadmap Guide: The Surge, Purge, Verge & More](https://www.blocknative.com/blog/ethereum-roadmap-guide)
- [Vitalik Buterin reveals Ethereum game plan for 2024 - Cointelegraph](https://cointelegraph.com/news/vitalik-buterin-ethereum-roadmap-2024)
- [Vitalik Buterin shares updated 2024 roadmap for Ethereum | The Block](https://www.theblock.co/amp/post/269799/vitalik-buterin-shares-updated-2024-roadmap-for-ethereum)
- [d/acc: one year later - Vitalik's Blog](https://vitalik.eth.limo/general/2025/01/05/dacc2.html)
- [Vitalik Buterin on defensive acceleration - 80,000 Hours Podcast](https://80000hours.org/podcast/episodes/vitalik-buterin-techno-optimism/)
- [Vitalik Buterin Proposes 'd/acc' Philosophy - The Defiant](https://thedefiant.io/news/people/vitalik-buterin-proposes-d-acc-philosophy-in-post-on-techno-optimism)
- [Why Is Vitalik Buterin Selling ETH? - Arkham](https://info.arkm.com/research/why-is-vitalik-buterin-selling-eth-35-million-sold-this-month)
- [Vitalik Buterin Says He Will Not Invest in Layer-2 Projects - CCN](https://www.ccn.com/news/crypto/vitalik-buterin-not-invest-l2-projects/)
- [Vitalik Buterin announces leadership changes for Ethereum Foundation - Cointelegraph](https://cointelegraph.com/news/buterin-announces-leadership-changes-ethereum-foundation)
- [Ethereum's Vitalik Buterin Goes on Offense Amid Major Leadership Shake-up - CoinDesk](https://www.coindesk.com/tech/2025/01/21/ethereum-s-vitalik-buterin-goes-on-offense-amid-major-leadership-shake-up)
- [Vitalik Buterin Reveals What the Ethereum Foundation Will and Won't Do](https://news.edaface.com/2026/03/13/vitalik-buterin-reveals-what-the-ethereum-foundation-will-and-wont-do/)
- [Ethereum's Vitalik Buterin Is Worried About Crypto's Future | TIME](https://time.com/6158182/vitalik-buterin-ethereum-profile/)
