# 散户反做市商博弈框架

> 调研主题：散户如何识别和利用做市商弱点  
> 更新日期：2026-04-12

---

## 目录

1. [识别做市商力不从心的信号](#1-识别做市商力不从心的信号)
2. [识别做市商被迫行动的信号](#2-识别做市商被迫行动的信号)
3. [做市商换届期的机会](#3-做市商换届期的机会)
4. [做市商内斗的机会](#4-做市商内斗的机会)
5. [散户"被收割预警信号"清单](#5-散户被收割预警信号清单)
6. [散户"做市商露出破绽"机会信号清单](#6-散户做市商露出破绽机会信号清单)
7. [实际案例分析](#7-实际案例分析)
8. [工具与数据源](#8-工具与数据源)
9. [参考来源](#9-参考来源)

---

## 1. 识别做市商力不从心的信号

做市商并非永远坚不可摧。当市场条件超出其风控边界、或其自身资金/库存出现问题时，会暴露出明显的"疲态"。

### 1.1 深度突然变薄、价差扩大

**核心逻辑**：做市商在宏观不确定性增大或方向性信心不足时，会主动缩减挂单规模、扩大价差，以降低库存风险。

**具体信号**：

- **Order book 深度骤降**：2025年10月加密市场流动性危机中，订单簿深度远低于10月初水平，表明做市商大规模撤退而非临时性错位。现在推动现货市场上下所需的资本量大幅减少。
- **Bid-Ask Spread 异常扩大**：价差扩大是流动性提供者正在撤退的实时信号。做市商在高波动期主动扩大价差，在稳定期收窄价差（动态价差调整）。
- **"空心市场"（Hollow Market）**：ETF资金外流、美联储利率预期变化和方向性信心不足共同抑制做市意愿，使市场变薄、脆弱，更容易出现夸张的价格反应。

**散户应对**：
- 监控 top-of-book 深度变化，使用 CoinGlass 等工具追踪订单簿深度历史数据
- 当深度骤降超过50%时，减少仓位或暂停交易
- 薄市场中避免使用市价单，改用限价单

### 1.2 大单频繁撤单又挂单——犹豫不决的做市商

**核心逻辑**：与spoofing（故意制造虚假深度）不同，犹豫不决的做市商表现为反复挂单-撤单，反映其对方向的不确定性和库存管理的困难。

**具体信号**：

- **高撤单率（Order-to-Fill Ratio 异常）**：合法做市商通常有较高的成交率；当某账户挂单-撤单比率超过基线95百分位以上2个标准差时，属于异常行为。
- **100毫秒内撤单**：下单后100毫秒内撤单是分层操纵（layering）的典型特征。
- **"幽灵墙"（Ghost Walls）**：在订单簿热力图上，大额流动性区域看似持续存在，但当价格接近时立即消失。这些"幽灵墙"是做市商犹豫或操纵的标志。

**散户应对**：
- 使用 Bookmap 等订单流可视化工具监控订单簿热力图
- 当看到反复出现又消失的大单墙时，不要将其作为支撑/阻力参考
- 关注实际成交量而非挂单量

### 1.3 Funding Rate 持续异常但价格不动

**核心逻辑**：永续合约 funding rate 与价格之间的背离，是市场结构失衡的重要信号。当 funding rate 持续极端而价格纹丝不动时，说明有人（通常是做市商/套利者）在刻意维持价格。

**具体信号**：

- **Funding rate 与价格的背离**：价格上涨但未平仓合约下降，可能暗示空头平仓（short squeeze）而非真正的买入需求。
- **极端 funding rate 预警**：2025年10月的$200-400亿美元清算事件中，BTC/ETH funding rate 飙升至年化6-10%，成为事后被证实的早期预警信号。
- **做市商撤退导致的 funding rate 飙升**：当做市商撤回永续合约市场的流动性时，期现价差扩大，funding rate 大幅偏离，机制本身需要更努力地纠正基差。

**散户应对**：
- 使用 CryptoQuant、Coinglass 追踪 funding rate
- Funding rate 年化超过 50% 持续24小时以上时，视为高风险信号
- 最可靠的信号来自极端背离，小幅背离通常是噪音

### 1.4 借币利率飙升

**核心逻辑**：借币利率飙升反映做空需求激增或做市商在对冲过程中面临库存压力。

**具体信号**：

- **借币费率突然上升**：负 funding rate、价格下跌期间未平仓合约上升、高借币需求，这三者同时出现表明空头仓位过度拥挤。
- **Short squeeze 前兆**：当清算发生时，被迫平仓的买单冲击订单簿，推动价格进一步上涨，每次清算都将价格推高，迫使更多交易者退出，形成级联效应。

**散户应对**：
- 监控主要借贷平台（Aave、Compound）的借币利率
- 借币利率飙升 + 负 funding rate = 潜在的空头清算机会
- 注意清算热力图（Coinglass），观察清算墙的位置

---

## 2. 识别做市商被迫行动的信号

做市商有时不得不执行特定任务——清仓、对冲、或按合约义务操作——这些"被迫行为"往往留下可识别的痕迹。

### 2.1 非交易时段的异常大单

**核心逻辑**：虽然加密市场24/7运行，但不同时区仍有明显的活跃/低活跃窗口。在低活跃期出现的大额交易，往往是机构/做市商的程序化操作。

**具体信号**：

- **亚洲凌晨/欧美假日的大额市价单**：流动性最薄时段的大额执行，通常意味着紧急性高于价格敏感性。
- **定时定量的模式化交易**：做市商机器人通过交易所API持续监控实时数据，按预编程的规则自动下单。当观察到连续多日在相同时间出现相似规模的卖单/买单时，这很可能是程序化清仓指令。
- **TWAP/VWAP 执行痕迹**：机构大额交易常通过时间加权平均价格（TWAP）或成交量加权平均价格（VWAP）算法分批执行，留下均匀分布的交易痕迹。

**散户应对**：
- 使用 Whale Alert 等工具监控大额链上转账
- 关注非高峰时段的异常成交量
- 如果连续多日在相近时间出现定量卖单，考虑是否有程序化出货

### 2.2 连续多日定时定量买入/卖出

**核心逻辑**：做市商执行清仓/建仓指令时，为减少市场冲击，通常使用算法将大单拆分为小单在数天内执行。

**具体信号**：

- **链上规律性转账**：每天固定时间向交易所转入相似数量的代币（如Mantra OM崩盘前，17个钱包向交易所转入了4360万OM，价值2.27亿美元）。
- **交易所API数据的规律性**：同一价格区间持续出现相同规模的卖单/买单。
- **成交量分布异常**：正常市场的成交量呈钟形分布，做市商程序化操作导致成交量在特定时间段异常均匀。

### 2.3 合约到期前的加速出货模式

**核心逻辑**：做市商与项目方的合约通常为6-12个月。合约到期前，做市商可能加速处理手中的库存代币。

**具体信号**：

- **Loan+Option 模型的末期行为**：做市商在合约结束前，有动力将手中借来的代币全部卖出获利，然后在低价买回归还。这通常导致合约最后1-2个月内出现持续的卖出压力。
- **卖出节奏加快**：观察到每日卖出量逐渐递增，且伴随价格下行趋势。
- **流动性逐步撤出**：做市商减少挂单深度，为最终退出做准备。

**散户应对**：
- 追踪项目方公开的做市商合约信息
- 关注 Binance 新规要求的做市商披露信息
- 在已知合约到期日前1-2个月提高警惕

---

## 3. 做市商换届期的机会

### 3.1 老做市商合约到期、新做市商未接上的真空期

**核心逻辑**：当一个项目的做市商合约到期但新做市商尚未就位时，会出现"流动性真空"，此时代币的交易条件急剧恶化，但也可能带来机会。

**具体信号**：

- **价差突然扩大数倍**：正常价差0.1%的代币，突然扩大到1-5%。
- **深度急剧下降**：订单簿两侧的挂单量大幅减少。
- **价格波动性骤增**：没有做市商维护的代币，小额买卖即可引起巨大价格波动。
- **成交量可能下降或反常上升**：取决于市场对这一变化的认知和反应。

**历史参考**：

为防止合约到期时的突然中断，行业最佳实践要求：
- 合约中包含有序退出条款
- 维持与替代流动性提供者的关系
- 在关键事件期间考虑临时聘用多个做市商

**散户策略**：
- 流动性真空期价格通常被低估——如果基本面未变，可能是低价买入机会
- 但必须用限价单操作，避免在极薄市场中使用市价单
- 做好持仓时间较长的准备，因为流动性恢复需要时间

### 3.2 交易所下架引发的做市商撤退

**核心逻辑**：当代币被主流交易所下架时，做市商通常第一时间撤退，引发价格崩盘。

**实际案例**：
- 2026年4月9日 Binance 下架6个代币公告后，FUN 在数分钟内暴跌27.93%
- 2025年 VIB 被 Binance 下架后数日内价格下跌29.7%
- VOXEL 在被移除后7天内下跌41.90%

**散户策略**：
- Binance 通常提供30天通知期——利用此窗口退出或迁移资产
- 如果代币在其他DEX或CEX上仍有流动性，可考虑跨平台套利
- 下架后的极端低价有时提供短期反弹机会（但风险极高）

---

## 4. 做市商内斗的机会

### 4.1 同一代币多做市商时的博弈

**核心逻辑**：当一个代币有多个做市商时，它们之间的竞争和冲突会在价格行为中留下痕迹。

**竞争的表现**：

- **价差压缩竞赛**：多个做市商竞相设置更窄的价差以捕获订单流。如果一个做市商设置稍窄的价差，其他做市商通常会跟进。出价通常每100毫秒变化多次。
- **订单存活时间缩短**：竞争加剧时，"订单在出价位置停留的时间更短"，因为竞争者持续互相超越。
- **最终市场退出**：交易量下降 + 价差收窄最终使运营无利可图，做市商被迫退出。

**冲突的表现**：

- **Loan+Option 模型冲突**：如果一个做市商行使看涨期权卖出代币，另一个做市商可能正在试图支撑价格，导致价格行为出现矛盾。
- **算法协调风险**：基于强化学习的交易代理可能在没有沟通或协议的情况下自行创造出超越正常竞争的共谋结果，损害竞争和价格效率。
- **透明度缺失**：多个做市商运营需要明确定义运营边界和职责的透明协议，但实际中这种协调往往缺失。

### 4.2 价格行为的矛盾信号

**如何识别做市商内斗**：

- **同一时间框架内的方向矛盾**：价格在窄幅区间内频繁往返，且伴随大额挂单在买卖两侧快速变化。
- **深度两侧不对称波动**：买方深度和卖方深度交替出现异常变化，表明不同做市商在不同方向上操作。
- **价格发现失效**：当做市商之间无法协调时，价格可能在一个区间内"震荡"而没有明确的趋势。

**散户策略**：
- 做市商竞争 = 更窄价差 + 更好执行价格，这对散户有利
- 做市商冲突 = 高波动 + 价格发现失效，此时使用区间交易策略
- 当一方做市商退出时，价格通常会朝另一方做市商希望的方向移动

---

## 5. 散户"被收割预警信号"清单

以下信号出现时，散户应立刻提高警惕或离场：

### 5.1 即刻离场信号（红色警报）

| 信号 | 描述 | 风险等级 |
|------|------|----------|
| **大额代币转入交易所** | 17个钱包在短时间内向交易所转入大量代币（如OM案例中的2.27亿美元） | 极高 |
| **做市商已知地址活跃卖出** | 通过Nansen、Arkham等工具追踪到做市商地址在持续转出代币 | 极高 |
| **项目方/VC解锁前的异常活动** | Token unlock前1-2周出现异常卖压，每周超过6亿美元的代币解锁，约90%会导致价格下跌 | 极高 |
| **交易所下架公告** | 主要交易所宣布下架，做市商将第一时间撤退 | 极高 |
| **Funding rate极端偏离** | 年化超过100%持续超过12小时 | 高 |

### 5.2 高度警惕信号（橙色警报）

| 信号 | 描述 | 风险等级 |
|------|------|----------|
| **订单簿深度骤降 > 50%** | 做市商正在撤退 | 高 |
| **Bid-ask spread 扩大3倍以上** | 流动性提供者信心不足 | 高 |
| **成交量暴增但价格不涨** | 有人在大量出货，买方被用作退出流动性 | 高 |
| **BTC dominance 跌破50%** | 散户过度涌入山寨币，通常是周期顶部信号 | 高 |
| **Fear & Greed 指数 > 75** | 极度贪婪，应开始减仓 | 高 |
| **OTC市场异常活跃** | 锁仓代币在场外以折价交易，暗示大户急于退出 | 高 |

### 5.3 需关注信号（黄色警报）

| 信号 | 描述 | 风险等级 |
|------|------|----------|
| **价格上涨但成交量下降** | 上涨动力减弱，潜在反转 | 中 |
| **做市商合约即将到期** | 关注公开信息和链上行为变化 | 中 |
| **借币利率持续攀升** | 做空需求增加或做市商对冲压力增大 | 中 |
| **项目方更换做市商** | 过渡期可能出现流动性真空 | 中 |
| **AI生成的FUD大量扩散** | 通过自动化账户、评论农场和合成新闻叙事制造恐慌 | 中 |

### 5.4 退出流动性陷阱的识别

退出流动性陷阱（Exit Liquidity Trap）是散户被收割的最常见模式：

1. **代币上线激发兴趣和炒作**
2. **散户涌入买入，推高价格**
3. **内部人员以低价获得的代币在高位卖出**
4. **大量抛售冲击市场，价格暴跌**
5. **散户成为内部人员的"退出流动性"**

**防御策略**：
- 检查代币分配：团队/VC持有超过50%供应量是高风险信号
- 检查解锁日历：使用 Tokenomist.ai、CryptoRank、DropsTab 追踪
- 分批建仓/减仓，使用 DCA-out 策略而非一次性操作

---

## 6. 散户"做市商露出破绽"机会信号清单

以下信号出现时，可能是有利的入场/出场时机：

### 6.1 高概率入场机会

| 机会信号 | 描述 | 逻辑 |
|----------|------|------|
| **空头挤压前兆** | 负funding rate + 高借币利率 + 价格企稳 | 空头过度拥挤，清算级联即将触发 |
| **流动性真空后的价格稳定** | 做市商撤退导致暴跌后，价格在低位企稳且成交量回升 | 新做市商可能正在入场，或自然需求回归 |
| **做市商竞争加剧** | 价差收窄、深度增加、执行改善 | 多做市商竞争环境下，散户获得更好的交易条件 |
| **恐慌清算后的超卖** | 大规模清算导致价格远低于基本面支撑 | 做市商和套利者将修复定价偏差 |
| **极端负funding rate** | 年化 < -30% 持续超过24小时 | 空头过度拥挤，资金费率套利机会 |

### 6.2 高概率出场/做空机会

| 机会信号 | 描述 | 逻辑 |
|----------|------|------|
| **做市商加速出货** | 链上可见的规律性大额卖出，节奏加快 | 做市商正在清理库存 |
| **Token unlock前的价格支撑** | 大额解锁前价格被异常维持在高位 | 做市商/项目方在为解锁后的卖出创造更好的退出价格 |
| **极端正funding rate** | 年化 > 100% | 多头过度杠杆，清算风险极高 |
| **订单簿单侧异常增厚** | 卖方深度突然大幅增加 | 做市商正在为大量卖出做准备 |

### 6.3 DEX 上的 MEV 保护策略

MEV（最大可提取价值）三明治攻击是做市商行为的极端形态，散户需要主动防御：

**攻击规模**：
- 2025年末到2026年初，仅以太坊上，搜索者在30天内提取了约2400万美元
- 2025年3月，一名交易者在Uniswap V3上进行稳定币兑换时损失了超过21.5万美元

**防御策略**：

1. **使用受保护的RPC端点**：将交易通过受保护的RPC路由，直接发送给可信的区块构建者，隐藏你的交易意图直到被包含。
2. **使用抗MEV协议**：CoW Swap使用批量拍卖，批次内的交易无法被三明治攻击，求解器竞争最佳价格。
3. **严格管理滑点**：散户设置更高的滑点容忍度以保证成交，实际上为机器人创造了完美的攻击机会。将滑点设为最低可接受值。
4. **选择Layer 2**：2026年战场已从以太坊L1转向Layer 2和高吞吐量链。
5. **拆分大额交易**：将大额交易拆分为多笔小交易执行。

---

## 7. 实际案例分析

### 7.1 案例一：Mantra (OM) 崩盘事件（2025年4月）

**事件经过**：
- 2025年4月13日，OM代币在数小时内暴跌90%，从约$6.32跌至$0.49，蒸发超过50亿美元市值
- 崩盘前，17个钱包向交易所转入4360万OM（价值2.27亿美元）
- 团队持有代币总循环供应量的90%

**各方说法**：
- 项目方：指责交易所"鲁莽的强制清算"
- OKX：否认责任，指出是多个串通账户的协调市场操纵
- 链上数据：显示大量代币在崩盘前被系统性转入交易所

**散户教训**：
- 团队持有超高比例代币是极端风险信号
- 链上大额转入交易所是崩盘的前兆信号
- 当项目基本面存疑（Mantra此前曾被揭露虚假声称FTX投资），结合链上异常，应果断离场

### 7.2 案例二：Movement Labs (MOVE) 做市商丑闻（2025年）

**事件经过**：
- Movement Labs高管与其做市商串通，在公开市场抛售了价值3800万美元的MOVE代币
- MOVE代币随后暴跌近20%
- 散户持有者被套

**行业影响**：
- 这两个丑闻正在重塑加密做市行业
- 做市商表示"信任不再被假设——而是需要被设计"
- Binance随后推出严格的做市商披露新规

### 7.3 案例三：DWF Labs 争议（2023-2025）

**事件经过**：
- Binance调查团队发现DWF Labs在2023年进行了超过3亿美元的洗盘交易
- 调查发现DWF操纵了YGG和至少其他6个代币的价格
- DWF Labs的商业模式引发争议——将代币交易协议包装为"风险投资"
- Wintermute CEO公开批评DWF Labs的行为

**散户教训**：
- 当一个代币的"投资者"实际上是做市商时，要质疑真实的买入需求
- DWF Labs投资公告后的代币价格上涨可能是人为制造的
- 关注做市商之间的公开冲突——这通常揭示了行业的真实运作方式

### 7.4 案例四：二级OTC市场——隐藏的供应炸弹

**运作机制**：
- 早期投资者在锁仓期到期前通过OTC渠道以折价出售锁仓代币
- 买家在代币解锁后立即抛售
- 做市商基于"官方流通供应量"校准流动性，但实际上隐藏供应远超预期
- 做市商在不知情的情况下成为内部人员的"退出流动性"

**涉及代币**：$LAYER、$OM、$MOVE 均被发现有可疑的OTC市场活动

**散户教训**：
- 官方流通供应量可能严重低估实际卖压
- 关注链上是否有未被计入的代币流动
- 要求项目方实施链上锁仓、不可转让锁定、时间锁定智能合约

### 7.5 案例五：Alameda Research 的教训

**核心问题**：
- Alameda Research作为顶级做市商，因姐妹公司FTX的流动性危机而崩溃
- FTX挪用了100亿美元客户资金借给Alameda进行高风险交易
- 做市商和交易所之间的利益冲突可能是灾难性的

---

## 8. 工具与数据源

### 8.1 链上追踪工具

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| **Whale Alert** | 实时监控大额区块链转账 | 追踪做市商地址的大额转账 |
| **Nansen** | 钱包分类（VC、做市商、巨鲸）、组合追踪 | 识别做市商地址行为模式 |
| **Arkham Intelligence** | 地址标签、资金流向分析 | 追踪做市商资金流动 |
| **CryptoQuant** | Funding rate、交易所流入/流出 | 监控异常资金流动 |

### 8.2 订单簿与流动性工具

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| **CoinGlass** | 大额订单、清算数据、历史订单簿 | 分析做市商挂单行为 |
| **Bookmap** | 订单流可视化、热力图 | 识别spoofing和ghost walls |
| **Kaiko** | 机构级市场数据 | 深度分析流动性变化 |

### 8.3 代币解锁追踪

| 工具 | 功能 |
|------|------|
| **Tokenomist.ai** | 代币解锁日历和锁仓数据 |
| **DropsTab** | 代币锁仓和分配追踪 |
| **CryptoRank** | 代币解锁时间表 |

### 8.4 MEV 防护工具

| 工具 | 功能 |
|------|------|
| **CoW Swap** | 批量拍卖防三明治攻击 |
| **Flashbots Protect** | 受保护的RPC端点 |
| **MEV Blocker** | 隐藏交易意图 |

---

## 9. 参考来源

### 市场微观结构与操纵

- [5 Ways to Spot Crypto Market Manipulation Before It Costs You](https://financefeeds.com/5-ways-to-spot-crypto-market-manipulation/)
- [Crypto Market Manipulation: Algorithms, Spoofing & Social Signals (2026)](https://www.outlookindia.com/amp/story/xhub/blockchain-insights/crypto-market-manipulation-in-the-algorithmic-age-how-price-influence-has-evolved-beyond-wash-trading)
- [Crypto Market Manipulation 2025 - Chainalysis](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)
- [Unmasking Crypto Market Manipulation - CertiK](https://www.certik.com/resources/blog/unmasking-crypto-market-manipulation-wash-trading-spoofing-and-more)
- [$20 Billion Crypto Crash: What Liquidations Reveal - Solidus Labs](https://www.soliduslabs.com/post/when-whales-whisper-inside-the-20-billion-crypto-meltdown)

### 订单簿与流动性分析

- [Structural Shift: Can Crypto Trading Recover After October's Liquidity Crash? - CoinDesk](https://www.coindesk.com/markets/2025/11/15/crypto-liquidity-still-hollow-after-october-crash-risking-sharp-price-swings)
- [Optimizing Order Book Depth with Crypto Market Making - Yellow Capital](https://www.yellowcapital.com/blog/optimizing-order-book-depth-with-crypto-market-making/)
- [Order Book Depth & Heatmaps Explained - WhalePortal](https://whaleportal.com/blog/order-book-depth-explained/)
- [Order Book Liquidity on Crypto Exchanges - MDPI](https://www.mdpi.com/1911-8074/18/3/124)

### Funding Rate 与衍生品信号

- [Funding Rates in Crypto: The Hidden Cost, Sentiment Signal, and Strategy Trigger](https://quantjourney.substack.com/p/funding-rates-in-crypto-the-hidden)
- [How to Analyze Funding Rates in Crypto: Complete Guide 2026](https://zipmex.com/blog/how-to-analyze-funding-rates-in-crypto/)
- [The Bearish Crypto Funding Rate Divergence - AInvest](https://www.ainvest.com/news/bearish-crypto-funding-rate-divergence-implications-risk-management-2512/)
- [Funding Rates Structure: Floor & Ceiling - BitMEX](https://www.bitmex.com/blog/2025q3-derivatives-report)

### Spoofing 与订单操纵

- [Crypto Market Spoofing: Identifying Fake Orders - CoinTelegraph](https://cointelegraph.com/news/crypto-market-spoofing-identifying-fake-orders-and-their-impact)
- [Liquidity Spoofing: How Fake Orders Manipulate Crypto Prices (2026)](https://www.outlookindia.com/xhub/blockchain-insights/how-does-liquidity-spoofing-influence-price-without-executing-trades)
- [How to Detect Spoofing in Trading - Bookmap](https://bookmap.com/blog/unmasking-spoofing-detecting-and-navigating-deceptive-trading-practices)
- [Learning the Spoofability of Limit Order Books - arXiv](https://arxiv.org/html/2504.15908v1)

### 做市商丑闻与案例

- [Mantra (OM) and Movement Labs (MOVE) Token Scandals - CoinDesk](https://www.coindesk.com/markets/2025/05/17/movement-labs-and-mantra-scandal-are-shaking-up-crypto-market-making)
- [Mantra's OM Crashes 90% - CoinDesk](https://www.coindesk.com/markets/2025/04/14/mantra-s-om-crashes-90-in-bizarre-selloff-as-team-alleges-forced-liquidations)
- [Market Maker Deals Are Quietly Killing Crypto Projects - CoinTelegraph](https://cointelegraph.com/news/market-maker-deals-quietly-killing-crypto-projects)
- [DWF Labs Controversy - Wikipedia](https://en.wikipedia.org/wiki/DWF_Labs)
- [The Secretive Dealer: Rise and Fall of Crypto Market Makers - ChainCatcher](https://www.chaincatcher.com/en/article/2201766)

### Token Unlock 与 OTC 市场

- [Secondary Crypto OTC Market Turns Market Makers into Exit Liquidity](https://medium.com/coinmonks/secondary-crypto-otc-market-turns-market-makers-into-exit-liquidity-fe5c615d2000)
- [How VCs Trade Token Unlocks - CoinTelegraph](https://cointelegraph.com/news/how-vcs-trade-token-unlocks)
- [Crypto Market Making: Retainer vs. Loan/Call Model - Flowdesk](https://flowdesk.co/updates/blogs/67215e228e4bf46d9bd3f247/)
- [Broken Market-Making Deals Are Derailing Promising Projects](https://crypto.news/broken-market-making-deals-derail-promising-projects/)

### MEV 与 DEX 保护

- [What Is MEV in Crypto and How Can Retail Investors Avoid Getting Exploited? - KuCoin](https://www.kucoin.com/blog/bd-what-is-mev-in-crypto-and-how-can-retail-investors-avoid-getting-exploited)
- [Implementing Effective MEV Protection in 2025](https://medium.com/@ancilartech/implementing-effective-mev-protection-in-2025-c8a65570be3a)
- [Understanding MEV Sandwich Attacks - Carbon DeFi](https://www.carbondefi.xyz/blog/understanding-mev-sandwich-attacks-frequently-asked-questions)
- [Sandwich Attacks & MEV Bots - Outlook India](https://www.outlookindia.com/xhub/blockchain-insights/what-are-sandwich-attacks-how-mev-bots-drain-millions-from-crypto-traders)

### 散户策略与退出信号

- [Why Retail Strategies Fail in 2025 - Coinmonks](https://medium.com/coinmonks/why-retail-strategies-fail-in-2025-decoding-the-institutional-code-for-consistent-profits-d2cebe7ae8d7)
- [Battle of the Bots: How Market Makers Fight It Out](https://mackgrenfell.com/blog/how-market-makers-fight-it-out-on-cryptocurrency-exchanges)
- [Binance Market Maker Disclosure Requirements - CryptoRank](https://cryptorank.io/news/feed/91ebf-binance-market-maker-disclosure-requirements)
- [Binance Delisting Triggers Double-Digit Losses](https://en.cryptonomist.ch/2026/04/09/binance-delisting-six-altcoins/)
- [Market Making: Predatory or Essential? - Presto Research](https://www.prestolabs.io/research/market-making-predatory-or-essential)

### 巨鲸追踪与链上分析

- [How to Track Crypto Whale Movements - Ledger](https://www.ledger.com/academy/topics/crypto/how-to-track-crypto-whale-movements)
- [Whale Tracker Crypto Guide - Stoic.ai](https://stoic.ai/blog/crypto-whale-tracker-expert-guide-to-monitoring-market-movers/)
- [CoinGlass Large Order & Trade Statistics](https://www.coinglass.com/large-orderbook-statistics)
- [DOJ Charges 10 in Massive Crypto Wash Trading Scheme - CCN](https://www.ccn.com/news/crypto/doj-indicts-10-crypto-wash-trading-bots-market-manipulation/)
