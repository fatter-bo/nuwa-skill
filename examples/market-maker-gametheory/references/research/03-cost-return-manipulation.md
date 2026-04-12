# 做市商成本回报计算与操纵模式识别

> 调研日期：2026-04-12
> 主题：理解做市商拉盘/砸盘的成本结构，以及散户可识别的操纵模式

---

## A. 成本回报计算

### A1. 拉盘成本结构

#### 核心变量：订单簿深度决定拉盘成本

拉盘的资金需求并非固定值，而是由以下变量动态决定：

**1) 卖盘深度 (Ask-side Depth)**

订单簿深度决定了拉升一定幅度需要"吃掉"多少卖单。根据 Kaiko Research 的数据，2024年比特币的 1% 市场深度（即将价格推动 1% 所需的资金量）恢复至 FTX 崩溃前的约 1.2 亿美元水平。这意味着：
- **BTC 拉升 1%**：需要约 1.2 亿美元的买入压力（在主流交易所聚合深度下）
- **小市值代币**：由于订单簿极薄，可能仅需数万至数十万美元即可拉升 10-50%
- 机构级交易所 LMAX Digital 的 BTC 市场深度达到创纪录的 2700 万美元

**2) 散户抛压与卖出意愿**

- 在恐慌阶段拉盘成本更低（因为散户已抛售殆尽，卖盘稀薄）
- 在贪婪阶段拉盘成本更高（获利盘会持续抛售）
- 高 Funding Rate 意味着多头拥挤，继续拉盘的边际效益递减

**3) 滑点与价格冲击**

滑点公式：`滑点 = ((实际成交价 - 预期价格) / 预期价格) x 100%`

根据 Amberdata 的订单簿分析研究：
- 价格冲击在高波动率、宽买卖价差、低订单簿深度环境下放大
- 更激进的交易策略（如更短的 TWAP/VWAP 时间窗口）会增加价格冲击
- 机构交易者在流动性"黄金时段"与"危险时段"之间的滑点差异可达 20-30 个基点

**4) 成本估算框架**

| 代币类型 | 日均交易量 | 拉升10%估算成本 | 风险等级 |
|---------|-----------|---------------|---------|
| BTC/ETH | >10亿美元 | 数亿至数十亿美元 | 极高（几乎不可能单独操纵）|
| 中等市值 | 1000万-1亿美元 | 数百万至数千万美元 | 高 |
| 小市值 | 10万-1000万美元 | 数万至数百万美元 | 中 |
| 微型代币 | <10万美元 | 数千至数万美元 | 低（最常被操纵）|

> **关键洞察**：DOJ 2024年10月起诉案表明，被操纵的代币通常是"历史交易量低的小市值代币"，因为"即使小额买单也能引发大幅价格波动"。

#### 做市商合约结构与成本分摊

最常见的做市商合约结构是**代币借贷+看涨期权**：
- 做市商从项目方借入做市所需的代币，通常期限 1-2 年
- 作为回报，获得在借贷到期时以约定价格行权的看涨期权
- 这意味着做市商的拉盘动力与期权行权价直接挂钩——如果当前价格远低于行权价，做市商有强烈的拉盘动力

---

### A2. 砸盘的矛盾

砸盘是做市商面临的核心博弈困境，存在天然矛盾：

#### 砸盘收筹码的逻辑
- **目标**：通过压低价格制造恐慌，诱导散户割肉，以更低价格收集筹码
- **手段**：集中大额卖出、配合负面消息、在薄流动性时段操作
- **成本**：砸盘本身需要卖出持仓，导致自身库存价值下降

#### 库存贬值的矛盾
- 做市商持有大量代币库存，砸盘 10% 意味着库存价值也缩水 10%
- 如果持有 1000 万美元库存，砸盘 10% = 账面亏损 100 万美元
- 只有当预期低价收集的筹码在未来拉升后的利润 > 库存贬值损失时，砸盘才有经济意义

#### 库存风险管理

根据多家做市商的公开资料：
- 库存风险是做市商最核心的风险——价格方向性运动导致持仓中贬值资产累积，而升值资产被卖出
- 做市商通过 delta 对冲管理库存风险：若现货积累多头仓位，则通过永续合约或期权建立空头对冲
- 高级做市商采用 delta 中性策略，使利润完全来自买卖价差而非价格投机

#### 砸盘的决策条件
砸盘在以下条件下经济合理：
1. 做市商已通过衍生品（做空永续/期货）对冲了库存下跌风险
2. 砸盘后能以更低价格回购足够多的筹码
3. 有能力在后续将价格拉回并超过砸盘前水平
4. 市场情绪允许——不会触发不可控的恐慌性抛售

---

### A3. 时机选择逻辑：为什么在流动性薄的时段操作

#### 流动性的时间节律

根据 Amberdata 的流动性节律研究：

**"危险时段"（21:00 UTC）**
- 机构桌关闭、亚洲市场尚未完全激活
- 市场深度比平均值下降约 25%
- 买卖价差扩大，大额订单的价格冲击放大

**周末流动性枯竭**
- 机构做市商通常在周末减少运营
- 订单簿变薄，买卖价差扩大
- 同样的资金量能产生更大的价格波动
- 止损更容易被触发，滑点增加

**亚洲早盘特征**
- 亚洲时段交易量和流动性通常低于欧美时段
- 小市值代币在亚洲低流动性时段更容易出现异常波动
- 做市商可利用此窗口以更低成本移动价格

**美国假期**
- 美国交易者和机构缺席，流动性显著下降
- 合规监控团队可能不在岗
- 为操纵行为提供了更大的操作空间

#### 时机选择的经济逻辑

| 时段 | 流动性水平 | 拉盘成本（相对） | 操纵成功率 |
|-----|-----------|---------------|-----------|
| 美欧重叠（13:00-17:00 UTC）| 最高 | 100% | 最低 |
| 美国交易时段 | 高 | 80-90% | 低 |
| 亚洲交易时段 | 中等 | 60-70% | 中 |
| 21:00 UTC 空窗期 | 低 | 40-50% | 高 |
| 周末 | 最低 | 20-40% | 最高 |

> **关键洞察**：机构交易者在"黄金时段"与"危险时段"之间的滑点差异达 20-30 个基点，这直接反映了操纵者在薄流动性时段操作的成本优势。

---

### A4. 风险回报评估：什么时候放弃方向性操作

做市商在以下情况下选择"躺平吃价差"而非做方向：

#### 放弃方向性操作的条件

**1) 高波动率环境**
- 价格剧烈波动时，方向性押注的风险回报比恶化
- 做市商通过扩大买卖价差来补偿风险，价差收入反而增加
- 波动率越高，做市价差利润越丰厚

**2) 流动性充裕的市场**
- BTC/ETH 等主流资产的深厚流动性使方向性操纵几乎不可能
- 最理想场景是价格在区间内波动——买卖单频繁成交，价差利润最大化

**3) 多方博弈均衡**
- 多个大型做市商同时操作同一标的时，单边操纵的成本急剧上升
- 博弈均衡下，各方选择安全的价差策略而非高风险的方向性操作

**4) 监管风险上升**
- DOJ 2024 年起诉案后，做市商对操纵行为的法律风险评估大幅提升
- 合规成本增加可能促使部分做市商转向纯粹的做市策略

**5) 衍生品对冲困难**
- 如果无法有效对冲库存风险（如缺乏相应的永续合约或期权市场），方向性操作的风险太大
- 做市商追求的理想状态是 delta 中性——通过对冲使利润完全来自价差

---

## B. 操纵模式识别

### B1. 拉高出货 (Pump & Dump) 的链上和订单簿特征

#### 链上特征

根据 Chainalysis 2025 年报告和 DOJ 2024 年起诉案：

**发起阶段**：
- 代币创建者或大户购买大量供应量（通常是历史交易量低的代币）
- 链上可见：单一地址或关联地址集群在短时间内积累大量代币
- 流动性池操作：向 DEX 注入初始流动性，控制池子参数

**炒作阶段**：
- 通过社交媒体和聊天室（特别是 Telegram 群组）制造 FOMO
- 链上可见：大量新钱包地址开始买入，交易笔数激增
- 订单簿：买单逐步加大，卖盘被快速消化

**出货阶段**：
- 价格在数分钟内达到峰值后快速反转
- 链上可见：创始人/早期积累者的地址开始大量转出到交易所
- 订单簿：大额卖单出现，买盘深度急剧萎缩

#### 订单簿特征

根据 Solidus Labs 的监控研究：
- 交易量异常飙升（通常达到正常水平的 10-100 倍）
- 价格在数分钟到数小时内上涨 50%-1000%
- 买卖价差在拉升阶段异常收窄（人为买盘支撑），在崩溃阶段急剧扩大
- 订单簿呈现单边倾斜——拉升时几乎只有买单，崩溃时几乎只有卖单

#### DOJ 2024 年起诉案关键细节

2024 年 10 月，美国司法部在波士顿起诉 18 名个人和实体，这是**加密行业首次对做市商提出市场操纵和洗盘交易的刑事指控**：

- **涉案做市商**：ZM Quant、CLS Global、MyTrade、Gotbit
- **涉案代币**：Saitama、Robo Inu、VZZN、Lillian Finance
- **操作模式**：代币公司付费雇佣做市商执行洗盘交易，人为膨胀交易量，诱导散户买入后高位抛售
- **FBI 钓鱼执法**：FBI 创建了自己的代币 "NexFundAI Token"，探员伪装成项目方在视频会议和 Telegram 中与做市商沟通，收集关键证据
- **处罚**：已有两名被告认罪并判刑，当局查获超过 100 万美元加密货币

---

### B2. 洗盘交易 (Wash Trading) 的识别方法

#### 规模

根据 Chainalysis 2025 年报告：
- 2024 年在 Ethereum、BNB Smart Chain、Base 上的洗盘交易量约 25.7 亿美元
- 第一种启发式方法识别 7.04 亿美元，第二种识别 18.7 亿美元
- 识别出 23,436 个参与洗盘交易的独立地址
- 平均每个地址参与 2 个 DEX 池、执行 129 笔可疑交易

根据 Solidus Labs 的研究：
- 自 2020 年以来，以太坊 DEX 上至少 20 亿美元的洗盘交易
- 分析的约 30,000 个 DEX 流动性池中，67% 被洗盘交易者操纵

#### 识别方法

**启发式方法一：地址行为分析**
- 同一地址或关联地址之间的循环交易
- 短时间内在同一交易对上大量买入后立即卖出
- 交易量激增但价格几乎不变（买卖自我对消）

**启发式方法二：代币多发器分析**
- 通过代币批量发送工具（token multi-senders）同时向多个地址转移代币
- 这些地址随后在 DEX 上执行看似独立但实际协调的交易

**通用检测信号**：
- 交易量与价格变动不成比例（高交易量但低波动）
- 流动性池操作异常（频繁添加/移除流动性）
- 交易对手方分析：买卖双方地址具有资金关联
- 交易时间模式：高度规律的交易间隔暗示机器人操作
- 每月约 1,000-1,800 个 DEX 池涉及洗盘交易（占活跃池的 0.2-0.3%），表明活动高度集中

---

### B3. 假突破/假跌破 (Spoofing/Layering) 的订单簿信号

#### Spoofing（幌骗）定义与机制

Spoofing 是通过在订单簿一侧放置大额订单来欺骗市场参与者对真实供需的判断，然后在对面一侧触发成交。

**Layering（分层挂单）** 是 Spoofing 的变体：在不同价格级别放置多个订单，制造流动性增加的假象，一旦达到预期市场反应即取消或修改订单。

#### 可识别信号

**1) "幽灵墙"（Ghost Walls）**
- 订单簿热力图上出现大额流动性区域，看似持久存在
- 当价格接近这些区域时，订单突然消失
- 信号：大额订单在关键价位出现又消失

**2) 异常挂撤单模式**
- 频繁放置和取消订单，且从不实际成交
- 订单规模显著大于该市场的典型订单大小
- 订单存在时间极短（秒级或毫秒级）

**3) 被动区域挂单**
- 幌骗订单通常放置在订单簿的被动区域（远离当前价格）
- 目的：制造方向性压力，同时最小化意外成交的风险
- 由于被动区域流动性较薄，动量变化更加明显

**4) 流动性突然消失**
- 如果在价格波动期间或之前，一波流动性突然消失，可能存在操纵
- 这种模式常伴随着随后的剧烈价格运动

#### 检测工具和方法

- **Bookmap**：提供订单簿热力图可视化，可直观识别幌骗模式
- **Solidus Labs HALO**：使用 AI 代理检测幌骗、分层、洗盘交易等行为
- **Amberdata**：提供订单簿快照监控，分析市场深度变化

#### 检测挑战
- 在实时环境中检测幌骗极其困难，因为问题具有多维性
- 与股票交易所不同，许多加密平台缺乏像 FINRA/SEC 这样的监管框架
- 合法的大额订单取消与幌骗之间的界限模糊

---

### B4. 薄流动性时段的异常波动

#### 特征模式

**"Bart Simpson 形态"**
- 价格在一个方向突然大幅运动（"凸起"），短暂横盘后，又在相反方向突然运动回到起始价格附近
- 图形类似 Bart Simpson 的头部轮廓
- 常见于数分钟内出现数百枚 BTC 的大额订单
- 目标：清除多头或空头的保证金交易者（触发止损/强制平仓）

**识别要素**：
- 发生在低流动性时段（21:00 UTC 空窗期、周末、假期）
- 价格波动幅度与基本面消息不匹配
- 波动前后交易量分布异常（波动开始时突然放量，结束后迅速缩量）
- 订单簿在波动方向上的深度在事前已被人为削薄

**周末异常**：
- 机构做市商减少运营 -> 价格对情绪、技术形态、杠杆驱动的清算更加敏感
- 同等规模的订单在周末的价格冲击可能是工作日的 2-5 倍
- 去中心化交易所和小市值代币受影响最大

---

### B5. Funding Rate 操控

#### 机制原理

Funding Rate 是永续合约中多头和空头之间的定期支付，用于将合约价格锚定到现货价格：
- **正 Funding Rate**：多头付给空头 -> 做空有收益
- **负 Funding Rate**：空头付给多头 -> 做多有收益

#### 做市商操控策略

**步骤一：推高 Funding Rate**
- 在永续合约市场以相对少量资金大量做多，推高 Funding Rate
- 在薄流动性时段操作，所需资金更少
- 持续的正 Funding Rate 向市场释放"多头拥挤"的信号

**步骤二：引诱散户开空**
- 高 Funding Rate 使做空看起来是"稳赚不赔"的交易（收取 funding 费用）
- 散户和量化策略大量涌入做空
- 空头仓位集中到一定程度形成"挤压风险"

**步骤三：拉爆空头**
- 在现货市场发力拉升价格
- 空头被迫平仓（止损或强制清算），产生级联买入压力
- 短挤压 (Short Squeeze) 效应放大价格上涨

#### 近期案例规模

- 2025-2026 年多次发生大规模短挤压事件
- 单次清算事件规模达到 2.94 亿至 5.4 亿美元
- 2025 年全年约 1500 亿美元的加密清算量
- 清算级联的特征：极端偏向空头清算，确认短挤压

#### 散户识别信号
- 持续升高的 Funding Rate + 不断增加的未平仓合约 = 挤压风险累积
- Funding Rate 突然转负（跌至 3-6 个月低点）= 市场过度看空，可能反弹
- 关注清算热力图，识别大量止损集中的价格区域

---

### B6. 做市商"画线"——创造技术图形诱导散户

#### Wyckoff 操纵模式

最经典的做市商"画线"框架是 **Wyckoff 四阶段模型**：

1. **吸筹 (Accumulation)**：在底部区间反复震荡，逐步收集散户恐慌抛出的筹码
2. **拉升 (Markup)**：持续推高价格，吸引追涨资金
3. **派发 (Distribution)**：在顶部区间反复震荡，逐步将筹码卖给追涨的散户
4. **下跌 (Markdown)**：撤去买盘支撑，价格自由落体

#### 具体"画线"手法

**双底/头肩底**：
- 在支撑位附近精确放置买单，两次或三次"接住"价格
- 每次触底后小幅拉起，制造"底部已确认"的技术信号
- 散户根据技术分析入场做多后，完成出货

**假突破**：
- 短暂将价格推过关键阻力位，触发追涨买单和空头止损
- 在散户跟进后迅速反转，在高位完成出货
- 追涨者被套在高位

**假跌破**：
- 短暂将价格打破关键支撑位，触发多头止损
- 收集恐慌抛盘后迅速反弹
- 散户在底部割肉，做市商收到廉价筹码

#### 比特币 vs 山寨币的差异
- 由于更好的流动性和更低的操纵风险，比特币的技术形态成功率比山寨币高 3-5%
- 山寨币的技术形态更容易被"画出来"，因此可靠性更低
- 流动性越低的代币，技术形态被人为制造的概率越高

#### 散户防御策略
- 不要在单一技术信号上重仓
- 结合链上数据验证技术形态的真实性（大户是否在积累/分发？）
- 警惕在薄流动性时段形成的"完美"技术形态
- 关注订单簿深度变化——如果关键支撑/阻力位的挂单在价格接近时消失，很可能是假信号

---

## C. 关键案例研究

### C1. DWF Labs 操纵指控

**背景**：DWF Labs 是加密行业最活跃的做市商之一。

**WSJ 报道（2024年5月）**：
- Binance 内部市场监控团队调查 DWF Labs 后结论：该公司在 2023 年进行了大规模洗盘交易和价格操纵
- 调查发现 DWF 操纵了 YGG 及至少 6 个其他代币的价格
- 洗盘交易总额超过 3 亿美元

**Binance 的争议性处理**：
- 提交报告的调查团队负责人在报告提交一周后被解雇
- Binance 随后认定"证据不足以证明 DWF 存在市场滥用行为"
- DWF Labs 创始合伙人称指控是"竞争对手驱动的 FUD"

**2025年后续**：
- 2025年12月，新的指控揭示 DWF Labs 通过匿名钱包秘密购买数亿美元的 USD1 稳定币
- 分析识别出三层流动性结构，通过 WLF 的 TokenGovernor 合约注入超过 3.01 亿美元的工程化流动性

### C2. Alameda Research / FTX

**操纵结构**：
- Alameda 同时担任 FTX 的做市商、投资者和交易者——三重角色造成严重利益冲突
- 在 2021 年初至 2022 年 3 月间，Alameda 在 FTX 上线代币前提前购入约 18 个代币，价值约 6000 万美元（内幕交易）
- 作为 FTX 的主要做市商，Alameda 有时接受交易亏损以吸引客户到交易所

**"Alameda 缺口"**：
- FTX/Alameda 崩溃后，加密市场流动性出现显著缺口
- 多个交易所的市场深度大幅下降，价差扩大
- 这一缺口持续影响市场至 2024 年才基本恢复

### C3. FBI "NexFundAI" 钓鱼行动（2024年10月）

这是加密行业首次由执法机构主动设局揭露做市商操纵：
- FBI 创建 NexFundAI 代币，探员伪装为项目推广者
- 通过视频会议和 Telegram 群组与做市商沟通
- 成功收集 ZM Quant、CLS Global、MyTrade、Gotbit 等做市商的操纵证据
- 起诉 18 名个人和实体，已有被告认罪判刑

---

## D. 散户实用识别清单

### 高概率操纵信号（出现 3 个以上需高度警惕）

| # | 信号 | 检查方法 |
|---|------|---------|
| 1 | 交易量突然飙升 10 倍以上但无基本面消息 | 对比历史交易量 |
| 2 | 在薄流动性时段出现异常大幅波动 | 检查是否为周末/假期/21:00 UTC |
| 3 | 订单簿出现大额挂单后迅速消失 | 使用订单簿热力图工具（如 Bookmap）|
| 4 | Funding Rate 极端值（>0.1% 或 <-0.1% 每8小时）| 监控 Coinglass 等工具 |
| 5 | 链上大户地址在拉升前集中转入交易所 | 使用 Arkham/Nansen 等链上工具 |
| 6 | 技术形态"过于完美"（精确双底/精确头肩底）| 结合订单簿深度和链上数据验证 |
| 7 | 社交媒体突然出现协调性喊单 | 检查 Telegram 群、Twitter KOL |
| 8 | 代币上线后短期内交易量暴涨但持有人增长缓慢 | 对比交易量与独立地址增长 |
| 9 | 清算热力图显示大量止损集中在某价位 | 该价位极可能被"猎杀" |
| 10 | 同一代币在多个交易所间出现不同步的异常价格 | 可能是跨交易所操纵 |

---

## 参考来源

### 执法案例
- [SEC Enforcement Results FY 2024](https://www.sec.gov/newsroom/press-releases/2024-186)
- [SEC Enforcement: 2025 Year in Review (Harvard)](https://corpgov.law.harvard.edu/2026/01/21/sec-enforcement-2025-year-in-review/)
- [SEC Crypto Enforcement Fines 3018% Surge](https://glavx.org/sec-crypto-enforcement-fines-how-2024-saw-a-3-018-surge-in-penalties)
- [DOJ Charges 18 for Crypto Market Manipulation (Arnold & Porter)](https://www.arnoldporter.com/en/perspectives/blogs/enforcement-edge/2024/10/remaking-the-classics)
- [DOJ Crypto Market Manipulation Charges (PYMNTS)](https://www.pymnts.com/cryptocurrency/2024/us-charges-18-companies-and-individuals-alleging-crypto-market-manipulation)
- [FBI Creates Crypto Token to Catch Fraudsters](https://cryptobriefing.com/crypto-market-manipulation-charges/)
- [First Criminal Charges for Crypto Market Manipulation (Phillips & Cohen)](https://www.phillipsandcohen.com/first-ever-criminal-charges-filed-cryptocurrency-market/)
- [CFTC Enforcement Results FY 2024 (Davis Wright Tremaine)](https://www.dwt.com/blogs/financial-services-law-advisor/2024/12/cftc-enforcement-results-in-fy-2024)

### 市场操纵研究
- [Chainalysis: Crypto Market Manipulation 2025 - Wash Trading, Pump and Dump](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)
- [Solidus Labs: Crypto Pump-and-Dump Report 2025](https://www.soliduslabs.com/reports/crypto-pump-and-dump-report)
- [Solidus Labs: Crypto Market Makers Under the Spotlight](https://www.soliduslabs.com/post/enforcement-around-the-corner-crypto-market-makers-under-the-spotlight)
- [Solidus Labs: $2 Billion Wash Trades on DEXs](https://www.businesswire.com/news/home/20230912351705/en/Solidus-Labs-Uncovers-2-Billion-Worth-of-Wash-Trades-on-Decentralized-Exchanges)
- [arXiv: Microstructure and Manipulation - Quantifying Pump-and-Dump Dynamics](https://arxiv.org/html/2504.15790v1)
- [arXiv: Learning Spoofability of Limit Order Books](https://arxiv.org/html/2504.15908v1)

### DWF Labs 案例
- [Binance Fired Investigator Who Uncovered DWF Labs Manipulation (CoinDesk)](https://www.coindesk.com/business/2024/05/09/binance-fired-investigator-who-uncovered-market-manipulation-at-client-dwf-labs-wsj)
- [DWF Labs Denies Manipulation Claims (DL News)](https://www.dlnews.com/articles/people-culture/dwf-denies-market-manipulation-wash-trading-on-binance/)
- [DWF Labs Wikipedia](https://en.wikipedia.org/wiki/DWF_Labs)
- [USD1 Stablecoin Hidden Liquidity Engine (Disruption Banking)](https://www.disruptionbanking.com/2025/12/10/exclusive-inside-the-hidden-liquidity-engine-propping-up-world-liberty-financials-usd1-stablecoin/)

### Alameda Research / FTX
- [Alameda Research Market Manipulation (crypto.news)](https://crypto.news/alameda-researchs-market-manipulation-and-insider-trading-took-down-ftx/)
- [The Alameda Gap Explained (Cointelegraph)](https://cointelegraph.com/explained/the-alameda-gap-and-crypto-liquidity-crisis-explained)

### 市场微观结构
- [Kaiko Research: Crypto Exchange Liquidity Lowdown](https://research.kaiko.com/insights/crypto-exchange-liquidity-lowdown)
- [Kaiko: Crypto Liquidity Concentration Report](https://research.kaiko.com/insights/the-crypto-liquidity-concentration-report)
- [Amberdata: Rhythm of Liquidity - Temporal Patterns](https://blog.amberdata.io/the-rhythm-of-liquidity-temporal-patterns-in-market-depth)
- [Amberdata: Orderbook Slippage Metrics](https://blog.amberdata.io/identifying-crypto-market-trends-using-orderbook-slippage-metrics)
- [Amberdata: Monitoring Order Book Snapshots](https://blog.amberdata.io/monitoring-order-book-snapshots-to-understand-market-depth)

### Spoofing 检测
- [Cointelegraph: Crypto Market Spoofing - Identifying Fake Orders](https://cointelegraph.com/news/crypto-market-spoofing-identifying-fake-orders-and-their-impact)
- [Cointelegraph: Crypto Spoofing for Dummies](https://cointelegraph.com/explained/crypto-spoofing-for-dummies-how-traders-trick-the-market)
- [Bookmap: Detecting and Navigating Deceptive Trading Practices](https://bookmap.com/blog/unmasking-spoofing-detecting-and-navigating-deceptive-trading-practices)
- [Outlook India: Liquidity Spoofing 2026](https://www.outlookindia.com/xhub/blockchain-insights/how-does-liquidity-spoofing-influence-price-without-executing-trades)

### 做市商策略
- [Presto Labs: Market Making 101](https://medium.com/prestolabs/market-making-101-en-9f260823f501)
- [DWF Labs: Guide to Hedging Strategies of Crypto Market Makers](https://www.dwf-labs.com/news/understanding-market-maker-hedging)
- [Flovtec: Crypto Market Making Strategies](https://www.flovtec.com/post/crypto-market-making-strategies)
- [MHC Digital: Crypto Market Making Guide](https://mhcdigitalgroup.com/resources/crypto-market-making-guide)

### Funding Rate 与清算
- [CryptoSlate: $150 Billion Liquidated from Crypto in 2025](https://cryptoslate.com/how-150-billion-was-liquidated-from-crypto-market-in-2025-driving-bitcoin-crash/)
- [BingX: Short Squeeze in Crypto Explained](https://bingx.com/en/learn/article/what-is-short-squeeze-in-crypto-how-liquidations-trigger-price-surge)
- [Stockpil: $18B Leverage Time Bomb Analysis](https://www.stockpil.com/crypto-short-squeeze-2025-analysis/)

### 图形操纵
- [EBC Financial: Bart Simpson Pattern and Market Manipulation](https://www.ebc.com/forex/is-the-bart-simpson-pattern-a-sign-of-market-manipulation)
- [Clarity in Crypto: Market Manipulation](https://www.clarityincrypto.com/market-manipulation)
- [Weekend Risk in Crypto Trading (MenthorQ)](https://menthorq.com/guide/weekend-risk-in-crypto-trading/)
