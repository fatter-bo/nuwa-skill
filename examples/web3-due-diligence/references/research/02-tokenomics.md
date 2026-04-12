# 代币经济模型 (Tokenomics) 尽调研究报告

> 调研日期: 2026-04-12
> 维度: Web3项目尽调 -- 代币经济模型评估框架

---

## 目录

1. [通胀率评估](#1-通胀率评估)
2. [解锁节奏 (Vesting Schedule)](#2-解锁节奏-vesting-schedule)
3. [VC 持仓比例](#3-vc-持仓比例)
4. [价值捕获机制](#4-价值捕获机制)
5. [隐性增发机制](#5-隐性增发机制)
6. [FDV vs 市值](#6-fdv-vs-市值)
7. [历史崩溃案例](#7-历史崩溃案例)
8. [代币经济模型尽调检查清单](#8-代币经济模型尽调检查清单)
9. [参考来源](#9-参考来源)

---

## 1. 通胀率评估

### 1.1 什么是健康的年化通胀率?

| 通胀率区间 | 风险评级 | 说明 |
|---|---|---|
| < 1% | 低风险 | 接近通缩或轻微通胀, 如 Bitcoin (~0%), Ethereum (~0.5%) |
| 1% - 3% | 可接受 | 行业长期目标共识区间, 如 Solana 长期目标 1.5% |
| 3% - 7% | 中等风险 | 需评估通胀是否有对应的价值创造, 如 Solana 当前 ~3.9% |
| 7% - 15% | 高风险 | 除非处于早期引导阶段, 否则稀释严重 |
| > 15% | 极高风险 | 类似庞氏结构, 需高度警惕 |

### 1.2 主流项目通胀率对比

- **Bitcoin**: 年化通胀率约 0.8%(2024 减半后), 固定上限 2100 万枚, 每四年减半
- **Ethereum**: 年化通胀率约 0.5%, EIP-1559 引入的 fee burn 机制使 ETH 在高网络活动时期可实现净通缩
- **Solana**: 当前年化通胀率约 3.9%, 每年递减 15%, 长期目标 1.5%; SIMD-228 提案获 70% 投票支持, 目标将通胀降至约 0.92%
- **Cosmos (ATOM)**: 动态通胀率 7%-20%, 根据质押比例调整

### 1.3 关键评估要点

- 通胀率本身不能孤立看待, 需结合 **实际网络使用率** 和 **费用销毁机制** 综合评估
- Coinbase 机构研究指出: 解读代币通胀时, 不仅要看 vesting schedule, 还需考虑挖矿和质押带来的隐性排放
- **净通胀率 = 新增发行量 - 销毁量**, 这是比名义通胀率更有意义的指标

---

## 2. 解锁节奏 (Vesting Schedule)

### 2.1 典型 VC 解锁结构

| 参数 | 行业常见范围 | 健康标准 |
|---|---|---|
| Cliff 期 | 6-12 个月 | >= 12 个月为佳 |
| 线性释放期 | 12-36 个月 (cliff 后) | >= 24 个月为佳 |
| TGE 解锁比例 | 0%-25% | < 10% 为佳 |
| 总 Vesting 周期 | 18-48 个月 | >= 36 个月为佳 |

### 2.2 解锁类型

- **Cliff Unlock (悬崖解锁)**: 在锁定期结束后一次性释放大量代币, 对价格冲击最大
- **Linear Vesting (线性释放)**: 代币在一段时间内均匀释放, 卖压分散, 相对温和
- **Non-linear Vesting (非线性释放)**: 代币释放速率不均, 需仔细分析具体节奏

### 2.3 如何用 Token Unlocks 查询

目前主要的代币解锁数据追踪平台:

| 平台 | 网址 | 特点 |
|---|---|---|
| Tokenomist (原 Token Unlocks) | https://tokenomist.ai | 最全面的解锁日历, 提供单项目详情页 |
| CryptoRank | https://cryptorank.io/token-unlock | 支持多项目对比, 有分类筛选 |
| DropsTab | https://dropstab.com/vesting | 可视化解锁时间线 |
| CoinMarketCap | https://coinmarketcap.com/token-unlocks/ | 集成在主平台, 方便快速查看 |

**查询步骤 (以 Tokenomist 为例)**:
1. 访问 tokenomist.ai, 搜索目标项目
2. 查看 Unlock Schedule 图表, 关注大额解锁事件 (>2% 流通量)
3. 查看分配明细: Team / Investors / Ecosystem / Treasury 各占比多少
4. 对比 Cliff 到期日期与当前日期, 判断近期卖压

### 2.4 风险信号

- 单次解锁量超过流通量的 5%: **高风险**
- 多个大额解锁事件集中在同一时间段: **高风险**
- 解锁加速条款 (如 AltLayer 2024 年宣布暂停 6 个月 vesting, 这虽是利好但反映出项目方可操控解锁节奏): **需关注治理透明度**
- 2025 年全年预计释放代币价值 **$974.3 亿**, 其中 insider 解锁 $187.7 亿, 非 insider 分配 $786.6 亿

---

## 3. VC 持仓比例

### 3.1 Insider 分配基准

| 分配比例 (团队+投资人) | 风险评级 | 说明 |
|---|---|---|
| < 20% | 低风险 | 社区驱动型项目, 如早期 Bitcoin / 部分 DeFi 协议 |
| 20% - 35% | 可接受 | 合理的激励与社区平衡 |
| 35% - 50% | 中等风险 | 常见于 VC-backed L1/L2, 如 DOT (33%), SOL (48%) |
| > 50% | 高风险 | 如 FLOW (58%), 社区话语权弱, 存在大量抛压 |

### 3.2 "VC 币" 争议

BERA 代币的推出重新点燃了 "VC 币" 的争论, 即早期风投机构持有大量代币分配的项目。类似争议也出现在 Aptos、Sei Network、Starknet 等项目中。

**核心问题**: 以牺牲社区利益为代价, 将代币分配给内部人 (团队、创始人、VC) 的项目, 从长期来看处于劣势。

### 3.3 如何查询 VC 持仓

1. **链上追踪**: 使用 Bitquery, Arkham Intelligence 或 Nansen 追踪已知 VC 钱包地址
2. **公开数据**: CryptoRank (cryptorank.io/funds) 追踪主要 crypto fund 的投资组合
3. **解锁数据**: Tokenomist 的 Standard Allocation 分析, 区分 insider vs non-insider 分配
4. **项目文档**: 审查项目白皮书和代币分配文档中的分配比例

### 3.4 红旗信号

根据 Token Unlocks 研究:
- MC/FDV < 20%: 流通量过低, 高稀释风险
- Insider 控制超过总发行量 50%: 提取价值的结构
- 下一次解锁超过流通量 5%: 短期卖压巨大
- 团队和投资者解锁比生态系统分配对价格的破坏性更大, 因为 insider 面临更少的协调成本

---

## 4. 价值捕获机制

### 4.1 核心问题: 协议收入是否流向代币持有者?

大量代币是 "治理代币", 除投票权外不捕获任何经济价值。真正健康的代币经济模型应建立 **协议收入 -> 代币持有者** 的价值传导通道。

### 4.2 三大价值捕获机制

#### (A) Fee Switch (费用开关)

**定义**: 协议将部分交易费用分配给代币持有者的机制。

**典型案例 -- Uniswap**:
- Uniswap DAO 投票激活 fee switch, 将部分 LP 费用导向 UNI 代币
- 同时引入程序化 UNI 销毁机制, 包括从国库追溯销毁 1 亿枚 UNI
- 形成正反馈循环: 交易量增加 -> 更多费用 -> 更大销毁量 -> 供给减少 -> 潜在需求增加

**评估要点**: fee switch 是否已激活? 如未激活, 是否有明确的治理提案和时间表?

#### (B) Buyback & Burn (回购销毁)

**定义**: 协议用收入在公开市场回购代币并永久销毁。

**典型案例**:
- **Aave**: DAO 启动结构化回购计划, 每周分配 $100 万回购 AAVE, 6 个月试点期, 年化回购超过 $5000 万, 回购的 AAVE 分配给质押者
- **Jito**: 基金会完成首轮 $100 万 JTO 回购, 分 4 次执行, 使用 TWAP 策略最小化市场冲击
- **Hyperliquid**: 协议也在实施 buyback 计划

**评估要点**: 回购资金来源是否可持续? 回购规模相对于流通市值是否有意义?

#### (C) 质押收益 (Staking Yield)

**定义**: 持有者通过质押代币获取协议收入的一部分。

**分类**:
- **真实收益 (Real Yield)**: 来自实际协议收入 (如交易费), 可持续
- **排放收益 (Emission Yield)**: 来自新代币增发, 本质是稀释, 不可持续

**评估要点**: 质押 APY 来源是什么? 高 APY (>20%) 通常来自排放而非真实收益, 需警惕。

### 4.3 Token Terminal 估值框架

Token Terminal 将区块链活动转化为类似传统金融的业务指标:

| 指标 | 定义 | 用途 |
|---|---|---|
| Fees | 协议产生的总费用 | 衡量需求侧活跃度 |
| Revenue | 流向代币持有者的费用 | 衡量价值捕获效率 |
| Earnings | 收入减去所有支出后的净值 | 衡量协议盈利能力 |
| Cost of Revenue | 外包核心业务的支出 | 如 L2 的 DA 费用、LST 的节点运营商支出 |
| P/F Ratio | 价格/费用比 | 类似传统 P/E ratio |
| P/S Ratio | 价格/收入比 | 估值合理性判断 |

**使用方法**: 在 tokenterminal.com 上对比同赛道项目的 P/F、P/S ratio, 判断代币是否被高估或低估。

---

## 5. 隐性增发机制

### 5.1 常见隐性增发形式

| 类型 | 说明 | 风险等级 |
|---|---|---|
| 治理投票增发 | 通过治理提案修改最大供应量或增发率 | 高 |
| 智能合约 Mint 权限 | 合约保留 mint 函数且未放弃管理员权限 | 极高 |
| 生态激励排放 | 以 "生态建设" 名义持续增发奖励代币 | 中 |
| 跨链桥接铸造 | 跨链桥可在目标链无限铸造包装代币 | 高 |
| Treasury 解锁 | 基金会/Treasury 持有大量未计入流通量的代币 | 中高 |

### 5.2 无限铸造攻击 (Infinite Mint)

- 无限铸造攻击占加密货币黑客事件的 **30% 以上**, 造成超过 **$5 亿** 损失
- 典型案例: PAID Network 遭攻击, 代币价格因供应量暴增而下跌 85%
- 根本原因: 智能合约代码中的 bug 为攻击者提供了后门

### 5.3 尽调检查项

1. **合约审计**: 是否存在未受限的 mint 函数? 管理员是否有权任意铸造?
2. **权限检查**: mint authority 是否已销毁或转移给多签/DAO?
3. **Supply Cap**: 合约层面是否有硬编码的最大供应量?
4. **治理风险**: 治理是否可以投票修改供应量参数? 投票门槛是否合理?
5. **透明度**: 代币排放计划是否可预测且透明? Coinbase 研究强调, 可预测的铸造计划能增强市场信心, 而临时性或无限制铸造会导致信任崩塌

---

## 6. FDV vs 市值

### 6.1 核心概念

- **Market Cap (市值)** = 当前价格 x 流通量
- **FDV (完全稀释估值)** = 当前价格 x 总供应量
- **MC/FDV 比率** = 流通量占总供应量的比例, 反映稀释程度

### 6.2 什么比例算合理?

| MC/FDV 比率 | 评估 | 说明 |
|---|---|---|
| > 80% | 低稀释风险 | 大部分代币已流通, 如 BTC, ETH |
| 60% - 80% | 可接受 | 适度的未来稀释, 长期投资者可接受 |
| 40% - 60% | 中等风险 | 需仔细评估解锁节奏 |
| 20% - 40% | 高风险 | 大量代币待解锁, 卖压显著 |
| < 20% | 极高风险 | "低流通高 FDV" 陷阱, 零售投资者极易成为退出流动性 |

### 6.3 "低流通高 FDV" 骗局 -- Cobie 的分析

加密领域知名分析师 Cobie 在其 Substack 文章中深入分析了这一问题:

**核心论点**:
1. **私有价值捕获 (Private Capture)**: 大部分代币上涨空间在公开市场之前就已被私人市场消化。VC 以极低估值进入, 代币上市时已是高 FDV, 公开市场投资者几乎无利可图。
2. **幻影定价 (Phantom Pricing)**: 持有锁仓代币的投资者通过 OTC 市场以大幅折扣出售其锁仓头寸, 形成 "影子市场", 公开价格并不反映真实供需。
3. **新发行变得不可投资**: Cobie 直言 "new launches have become uninvestable", 呼吁投资者拒绝购买膨胀 FDV 的代币。

**数据佐证**:
- 低流通代币占市值排名前 300 加密货币的 **21.3%**
- 2024-2025 周期中, L2、GameFi、NFT 等叙事驱动赛道因 "低流通高 FDV" 结构大面积崩塌
- Haseeb (Dragonfly) 也公开讨论了为何这些低流通高 FDV 代币表现不佳

**投资者保护策略**:
- 优先关注 MC/FDV > 60% 的项目
- 对 FDV > 5x Market Cap 且 12-18 个月内有激进解锁的项目视为重大风险因素
- 要求透明的 OTC 市场信息, 了解实际的二级市场折扣

---

## 7. 历史崩溃案例

### 7.1 UST/LUNA 崩溃 (2022年5月)

**项目概述**: Terra 是一条 L1 区块链, UST 是其算法稳定币, 通过与 LUNA 的铸造/销毁机制维持美元锚定。

**死亡螺旋机制**:
1. UST 跌破 $1, 触发 LUNA 铸造以吸收卖压
2. 大量 LUNA 被铸造 -> LUNA 价格暴跌
3. LUNA 暴跌 -> 市场对机制失去信心 -> 更多 UST 被抛售
4. 更多 UST 抛售 -> 更多 LUNA 铸造 -> 正反馈循环失控

**灾难数据**:
- LUNA 供应量从 3.43 亿爆增至 **6.53 万亿** (数日内)
- LUNA 价格下跌 99.99999%
- 市值蒸发超过 **$450 亿**
- 创始人 Do Kwon 被判处 15 年监禁

**教训**: 算法稳定币的 mint/burn 机制完全依赖市场信心; 一旦信心崩塌, 反身性 (reflexivity) 会导致不可逆的死亡螺旋。任何双代币 mint/burn 模型都需要极端情况下的熔断机制。

### 7.2 OlympusDAO (OHM) 及其分叉

**项目概述**: OlympusDAO 是一个 DeFi 协议, 核心是通过 bonding 机制构建 "协议拥有的流动性 (Protocol-Owned Liquidity)", 并通过极高 APY 吸引质押。

**崩溃过程**:
- 巅峰时期 APY 高达 **7,000%**, 靠新增 OHM 铸造支撑
- 博弈论模型 (3,3) 依赖新买家不断进入支撑价格
- OHM 从高点下跌 **93-97%**
- 大量投资者将其称为庞氏骗局: 早期参与者通过流动性协议以后来者为代价获利

**OHM Fork 连锁效应**:
- Klima: 使用 OHM 模型购买碳信用额度
- Wonderland (TIME): 跨链 OHM 分叉, 因资金管理丑闻崩塌
- 大量仿盘在数周内归零

**教训**: 天文数字级 APY 不可能来自真实收益, 只能来自稀释; (3,3) 博弈论假设所有人合作, 但实际上先跑者获利, 留下者亏损。

### 7.3 Bancor

**项目概述**: Bancor V2.1 引入 "无常损失保护" (Impermanent Loss Protection), 承诺 LP 不会遭受无常损失。

**崩溃原因**:
- 无常损失保护实质上通过铸造新 BNT 代币来补偿
- 市场下行时, 铸造量激增, BNT 价格持续下跌
- 最终被迫暂停无常损失保护, 大量 LP 遭受巨额亏损

**教训**: 用通胀来覆盖风险的模型, 在市场压力下会加速崩溃。

### 7.4 共同模式总结

| 特征 | UST/LUNA | OHM Forks | Bancor |
|---|---|---|---|
| 不可持续的承诺 | 算法锚定 | 7000% APY | 无常损失保护 |
| 底层机制 | Mint/Burn 螺旋 | 稀释性排放 | 通胀补偿 |
| 信心依赖 | 极高 | 极高 | 高 |
| 反身性风险 | 致命 | 严重 | 严重 |
| 最终跌幅 | >99.99% | >90% | >80% |

---

## 8. 代币经济模型尽调检查清单

### 通胀与供应

- [ ] 年化通胀率是否 < 5%? 净通胀率 (减去销毁) 是多少?
- [ ] 是否有最大供应量硬上限?
- [ ] 通胀率是固定的还是可通过治理修改?
- [ ] 智能合约是否保留未受限的 mint 权限?

### 解锁与分配

- [ ] MC/FDV 比率是否 > 40%?
- [ ] Insider (团队+VC) 分配比例是否 < 35%?
- [ ] Cliff 期是否 >= 12 个月?
- [ ] 总 vesting 周期是否 >= 36 个月?
- [ ] 未来 6 个月是否有超过流通量 5% 的单次解锁事件?

### 价值捕获

- [ ] 代币是否捕获协议收入 (fee switch / buyback / staking)?
- [ ] 质押收益是否来自真实协议收入而非排放?
- [ ] P/F 和 P/S ratio 是否与同赛道项目可比?
- [ ] 协议是否有持续正现金流?

### 风险模式

- [ ] 是否存在双代币 mint/burn 机制 (UST/LUNA 模式)?
- [ ] 质押 APY 是否超过 20% (可能为稀释性排放)?
- [ ] 是否有 "保护" 或 "保险" 承诺靠通胀支撑 (Bancor 模式)?
- [ ] 是否存在 OTC 锁仓代币折价交易的影子市场?

---

## 9. 参考来源

### a16z Crypto 代币设计

- [Token Design Tag Page - a16z crypto](https://a16zcrypto.com/posts/tags/token-design/)
- [Tokenology: Moving beyond 'tokenomics' - a16z crypto](https://a16zcrypto.com/posts/article/beyond-tokenomics-tokenology/)
- [Your guide to tokens: Types, design, uses, more - a16z crypto](https://a16zcrypto.com/posts/podcast/token-guide-types-design-uses/)
- [VC Giant A16z Stands Against 'Predatory' Crypto Token Agreements](https://sdlccorp.com/post/vc-giant-a16z-stands-against-predatory-crypto-token-agreements/)

### Token Terminal 估值框架

- [Token Terminal - Fundamentals for crypto](https://tokenterminal.com/)
- [How to Use Token Terminal: Fees, Revenue and Crypto Fundamentals (2026)](https://www.dextools.io/tutorials/how-to-use-token-terminal-fees-revenue-metrics-tutorial-2026)
- [Token Terminal Revenue Metrics](https://tokenterminal.com/explorer/metrics/revenue)

### Messari 代币经济分析

- [DePIN Tokenomics Part 1 - Messari](https://messari.io/report/depin-tokenomics-part-1-token-distribution-models-incentive-mechanisms-token-trends-and-more)
- [Messari Methodology and Glossary](https://docs.messari.io/glossary/classification-system)
- [Berachain Tokenomics - Messari](https://messari.io/report/berachain-tokenomics)

### Token Unlocks 数据

- [Tokenomist (原 Token Unlocks)](https://tokenomist.ai)
- [CryptoRank Token Unlock Calendar](https://cryptorank.io/token-unlock)
- [DropsTab Vesting Schedules](https://dropstab.com/vesting)
- [CoinMarketCap Token Unlocks](https://coinmarketcap.com/token-unlocks/)
- [Standard Allocation Trends - Unlock Insights](https://insights.unlocks.app/tokenunlocks-standard-allocation/)
- [The Hidden Token Inflation No One Talks About - Unlock Insights](https://insights.unlocks.app/the-hidden-token-inflation-no-one-talks-about/)

### Cobie 低流通高 FDV 分析

- [New launches (part 1) - private capture, phantom pricing - Cobie](https://cobie.substack.com/p/new-launches-part-1-private-capture)
- [On the meme of market caps & unlocks - Cobie](https://cobie.substack.com/p/on-the-meme-of-market-caps-and-unlocks)

### FDV vs 市值

- [Market Cap vs FDV in Crypto - Mudrex](https://mudrex.com/learn/market-cap-vs-fdv-in-crypto-key-differences/)
- [What is FDV? - CoinGecko](https://www.coingecko.com/learn/what-is-fully-diluted-valuation-fdv-in-crypto)
- [FDV - The Great Dilution Dilemma - CoinMarketCap](https://coinmarketcap.com/academy/article/fully-diluted-valuation-fdv-the-great-dilution-dilemma)
- [Low Float High FDV Tokens Exposed - HackerNoon](https://hackernoon.com/the-crypto-scam-youre-falling-for-low-float-high-fdv-tokens-exposed)

### 价值捕获机制

- [Uniswap finally turns the fee switch - Blockworks](https://blockworks.co/news/uniswap-fee-switch)
- [Token Buybacks in Web3 - DWF Labs](https://www.dwf-labs.com/research/547-token-buybacks-in-web3)
- [Token Value Accrual - Vader Research](https://defivader.medium.com/token-value-accrual-8b27dd019551)
- [Token Trends: Blockchain Buybacks - WisdomTree](https://www.wisdomtreeprime.com/blog/token-trends-blockchain-buybacks-how-defi-is-adapting-tradfis-playbook/)

### 通胀率与货币政策

- [Interpreting Token Inflation - Coinbase Institutional](https://www.coinbase.com/institutional/research-insights/research/market-intelligence/interpreting-token-inflation)
- [Solana Tokenomics - Crypto.com](https://crypto.com/us/crypto/learn/solana-tokenomics)
- [Crypto's Monetary Policy: What is Tokenomics - CoinTracker](https://www.cointracker.io/blog/tokenomics)

### 崩溃案例

- [Tokenomics Lessons from Terra, Bancor, and the Death Spiral](https://medium.com/@defi_dm/tokenomics-lessons-from-terra-bancor-and-the-death-spiral-81bc6024b998)
- [Anatomy of a Run: The Terra Luna Crash - UNC Finance](https://jhfinance.web.unc.edu/wp-content/uploads/sites/12369/2023/11/Anatomy-of-a-Run.pdf)
- [Tokenomics 101: Olympus Protocol - Forgd](https://content.forgd.com/p/tokenomics-101-olympus-protocol)
- [Top Crypto Infinite Mint Hacks - Pontem](https://pontem.network/posts/top-crypto-infinite-mint-hacks)

### VC 持仓与分配

- [The "VC Coin" Controversy - ChainCatcher](https://www.chaincatcher.com/en/article/2166652)
- [VC Token Holdings - Bitquery](https://bitquery.io/blog/venture-capital-crypto-holdings)
- [Crypto Insider Ownership Trends - Tomasz Tunguz](https://tomtunguz.com/l1-ownership-over-time/)
- [Token Unlock Strategies Used by Top Crypto VCs - CoinTelegraph](https://cointelegraph.com/news/how-vcs-trade-token-unlocks)
