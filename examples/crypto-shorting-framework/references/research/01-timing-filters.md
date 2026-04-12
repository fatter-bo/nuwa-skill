# 做空时机识别：五层过滤框架

> 调研日期：2026-04-12
> 本文系统性整理加密货币做空时机识别的五层过滤方法——宏观、叙事、技术、链上、做市商状态。

---

## 目录

- [1. 宏观层面（Macro Layer）](#1-宏观层面macro-layer)
- [2. 叙事层面（Narrative Layer）](#2-叙事层面narrative-layer)
- [3. 技术层面（Technical Layer）](#3-技术层面technical-layer)
- [4. 链上层面（On-Chain Layer）](#4-链上层面on-chain-layer)
- [5. 做市商状态层面（Market Maker Layer）](#5-做市商状态层面market-maker-layer)
- [6. 五层综合评分与实操建议](#6-五层综合评分与实操建议)
- [参考来源](#参考来源)

---

## 1. 宏观层面（Macro Layer）

### 1.1 美联储政策周期与 BTC/ETH 价格的领先滞后关系

**核心发现：**

- BTC 在美联储首次确认宽松转向后的 30-60 天内通常出现显著上涨。2024 年利率稳定化后 BTC 在六周内上涨了 42%。
- **做空窗口**：美联储从鸽派转向鹰派的拐点（加息周期开始或加息预期升温）是做空的宏观级信号。当联储开始收紧政策、加息导致美元走强时，风险资产（包括 BTC）需求减少。
- **领先指标**：关注 CME FedWatch 工具的利率预期变化、FOMC 点阵图和会议纪要中的措辞变化，这些通常领先于实际政策变动 1-3 个月。

### 1.2 美元指数（DXY）与加密市场的负相关性

**核心发现：**

- 2024 年 Q1，DXY 与 BTC 的相关系数达到 **-0.65**，显示强负相关。
- 2023-2025 年间，BTC 与 DXY 的相关性在 **-0.4 至 -0.8** 之间波动。
- **量化规律**：DXY 每下降 1 个点，BTC 在 10 日均线上平均上涨 1.2%；反之亦然。
- **做空信号**：当 DXY 突破关键阻力位进入上升通道时，系统性看空加密市场。尤其关注 DXY 突破 105、108 等关键整数关口。

### 1.3 全球 M2 供应量变化与加密资产价格的关系

**核心发现：**

- 全球 M2（21 大央行）与 BTC 存在约 **10 周（约 70 天）** 的领先-滞后关系。更精确的研究显示最佳相关窗口为 **84 天**。
- 后 ETF 时代（2024 年 1 月至 2025 年初），BTC 与 M2 的相关性约为 **0.65**，但 30 日滚动相关系数在 -1 和 +1 之间多次翻转。
- **弹性关系**：M2 更多充当"背景节奏"而非精确预测引擎。Q1 2024 的 BTC 在 M2 变化温和时仍因 ETF 批准和减半预期大幅上涨。
- **做空信号**：当全球 M2 增速转负或增速持续放缓 2-3 个月时，叠加其他宏观信号，构成中期做空依据。

### 1.4 美债收益率变化对风险资产的影响

**核心发现：**

- 2025 年 3 月以来，美国 10 年期国债收益率上升了 46 个基点至 4.42%，显著削弱了风险资产吸引力。
- **关键阈值**：市场分析师密切关注 **4.5%** 这一水平，突破后可能触发进一步金融条件收紧，从根本上改变加密市场动态。
- **情境依赖**：当收益率上升源于通胀恐慌或美联储鹰派立场时，加密通常因投资者去杠杆而抛售。但若收益率上升源于强劲 GDP 预期或 AI 驱动的生产力预期，则加密可能不跌反涨。
- BTC 和 ETH 仍表现为**高贝塔风险资产**——在投资者乐观、流动性充裕时上涨。2025 年 10 月 BTC 从高点回落部分归因于国债收益率上升、关税冲击和 ETF 资金外流。
- **做空信号**：实际收益率（名义收益率-通胀预期）持续上升 + DXY 走强 = 强做空环境。

### 1.5 Arthur Hayes 流动性框架（RRP、TGA、美联储资产负债表）

**核心发现：**

Arthur Hayes（BitMEX 前 CEO）构建了一套以美元流动性为核心的加密市场分析框架：

**净流动性公式：**
```
净流动性 = 美联储资产负债表规模 - RRP余额 - TGA余额
```

**三大核心变量：**

| 变量 | 影响 | 做空信号 |
|------|------|----------|
| **RRP（逆回购）** | RRP 下降 = 流动性释放到市场 = 利好加密 | RRP 触底后开始上升 |
| **TGA（财政部一般账户）** | TGA 消耗 = 流动性释放；TGA 补充（发债）= 流动性回收 | 财政部大规模发债补充 TGA |
| **美联储 QT** | QT 每月缩减约 600 亿美元资产负债表 | QT 加速或 QE 停止 |

**2025 年 Q1 案例：**
- Hayes 预测加密市场在 2025 年 3 月中旬见顶，随后出现大幅调整。
- 关键计算：Q1 流动性注入总量约 6120 亿美元（RRP 释放 2370 亿 + TGA 消耗），扣除 QT 的 1800 亿。
- **历史验证**：BTC 在 2022 年 Q3 触底，恰逢 RRP 达到峰值；随后随 RRP 余额下降而上涨。

**做空信号清单：**
- RRP 余额企稳或开始上升
- 财政部宣布大规模国债发行计划
- 美联储加速 QT 或暗示停止 QE
- 三者叠加时构成最强做空信号

---

## 2. 叙事层面（Narrative Layer）

### 2.1 叙事周期四阶段中的做空窗口

加密叙事遵循一个可预测的四阶段生命周期：

| 阶段 | 特征 | 持续时间 | 做空策略 |
|------|------|----------|----------|
| **萌芽期** | 少数 KOL 讨论，链上数据初现端倪 | 数周-数月 | 不做空 |
| **共识期** | 主流媒体报道增多，资金开始流入 | 数周-数月 | 不做空，可寻找结构性做多 |
| **过热期** | 圈外人开始讨论，FDV 虚高，新项目扎堆 | 数天-数周 | **核心做空窗口** |
| **退潮期** | 叙事疲劳，资金外流，新钱包增速放缓 | 数周-数月 | 持续做空直至叙事彻底破灭 |

**Cobie 的洞见：**
- 加密行业唯一稀缺的资源是**注意力**。
- 协议开发以年为单位，但牛市交易者以周为单位运作——这种时间错配是做空的结构性机会。
- 2024 年 Cobie 重点批评了低流通高 FDV 代币模式，这类代币在叙事退潮时跌幅远超大盘。
- 成功的加密老手将交易视为**概率性结果业务**——评估特定未来结果成为现实的可能性。

### 2.2 社交媒体情绪拐点的量化指标

**Fear & Greed Index（恐惧与贪婪指数）：**

| 区间 | 含义 | 做空信号强度 |
|------|------|-------------|
| 75-100（极度贪婪） | 市场可能超买，回调风险高 | **强做空信号** |
| 50-75（贪婪） | 市场偏乐观 | 观望 |
| 25-50（恐惧） | 市场偏悲观 | 不做空（可能反弹） |
| 0-25（极度恐惧） | 投资者极度避险 | 不做空（投降区域） |

**2025 年实际数据：**
- 2025 年超过 30% 的时间处于恐惧或极度恐惧区域。
- 2025 年中后期指数最低触及 11，反映投资者极度避险。
- **叙事崩塌案例**：投资者得到了 2021 年以来一直游说的宏观、政策和结构性利好，但市场在每次反弹时都遭到抛售——这是信任崩塌和叙事破裂的典型表现。

**其他社交情绪指标：**
- **社交 Volume**：CT（Crypto Twitter）讨论量激增至历史极值
- **Google Trends**："Bitcoin" 搜索量达到峰值
- **新钱包增速**：日新增地址数量开始放缓（详见下方）

### 2.3 新钱包增速放缓作为见顶信号

- 活跃地址数是衡量用户数量的不完美但广泛使用的代理指标。
- 2025 年 Q3，用户数、交易数和手续费均环比下降（Currencies 和 Smart Contract 板块）。
- **但需注意**：移动钱包用户数仍在创新高（年增长 20%），且增长集中在新兴市场（阿根廷、哥伦比亚、印度、尼日利亚）。
- **更可靠的替代指标**：交易手续费被认为是链上最有价值的基本面指标，因为最难被操纵且最具可比性。
- **做空信号**：日新增地址数连续 2-3 周下降 + 交易手续费下降 + 活跃地址数下降 = 见顶概率增大。

### 2.4 "出租车司机指标"

- 当圈外人（出租车司机、理发师、亲戚聚会）开始主动谈论加密货币时，通常标志着叙事进入过热期。
- 这是一个**定性信号**，需要与量化指标交叉验证。
- 对应的量化版本：Google Trends 搜索量突破前高、主流媒体正面报道频率激增、新注册交易所账户数暴增。

---

## 3. 技术层面（Technical Layer）

### 3.1 适合币圈做空的 K 线形态

考虑到加密市场的高波动特征，以下形态在做空时最为可靠：

**经典看空形态（适配高波动）：**

| 形态 | 可靠性 | 注意事项 |
|------|--------|----------|
| **头肩顶** | 高 | 需在高时间框架（4H/日线）确认；颈线突破后可入场 |
| **双顶/三顶** | 高 | 第二/三顶通常伴随成交量下降 |
| **看跌吞没** | 中-高 | 在关键阻力位出现时信号更强 |
| **黄昏之星** | 中 | 需配合成交量确认 |
| **乌云盖顶** | 中 | 币圈可靠性低于传统市场 |
| **下降三角形** | 高 | 水平支撑 + 下降趋势线，跌破后加速 |

**币圈特殊考量：**
- 使用 **4 小时或日线** 级别形态，1 小时以下形态噪声太大。
- 止损设置需比传统市场宽 **2-3 倍 ATR**。
- 关注与清算密集区的配合（见 3.2）。

### 3.2 清算密集区（Liquidation Clusters）的分布和利用

**核心工具：CoinGlass 清算热力图**

CoinGlass 清算热力图聚合了各大期货交易所的多空杠杆仓位，计算不同价格带的潜在清算金额。

**利用方法：**

1. **识别密集区**：在热力图上寻找颜色最深（金额最大）的价格区域。
2. **方向判断**：
   - 上方密集的空头清算区 = 价格可能被"磁吸"上去扫掉空头止损后再下跌
   - 下方密集的多头清算区 = 价格可能被"磁吸"下去触发连环清算
3. **做空策略**：等待价格扫过上方空头清算区后，在反转信号出现时入场做空。

**2025 年案例：**
- ETH 清算热力图显示在 $1,952-$2,154 之间有约 $18 亿的多空杠杆集中，5-7% 的价格波动就可能触发强制平仓的连锁反应。
- CoinGlass 提供 12 小时到 1 年的多时间框架清算数据。

### 3.3 成交量价格背离信号

**经典背离做空信号：**

- **价涨量缩**：价格创新高但成交量递减——上涨动能衰竭。
- **RSI 顶背离**：价格创新高但 RSI 走低——最经典的技术做空信号。
- **MACD 顶背离**：价格新高但 MACD 柱状图缩小或 MACD 线死叉。

**币圈特殊处理：**
- 需排除 CEX 刷量干扰，优先使用 Binance、Coinbase 等头部交易所的真实成交量。
- 结合 OI（未平仓合约量）变化：价涨 + OI 上升 + funding rate 极高 = 杠杆过度，回调风险大。

### 3.4 币圈特有的技术分析注意事项

**Scam Wick（诈骗影线）：**
- 定义：在极短时间内（通常约 1 分钟）价格急剧波动然后迅速回归，产生超长影线。
- 常见于低流动性时段（亚洲早盘、周末）。
- **做空者的风险**：做空止损可能被 scam wick 触发后价格立刻回归——需将止损设在合理位置之外，或使用限价止损。
- **做空者的机会**：向上的 scam wick 可能是入场做空的好时机（如果基本面和链上数据支持做空方向）。

**做市商画线：**
- 加密市场的价格走势常被大型做市商主导，尤其在低流动性环境下。
- 经典手法：先将价格推至关键阻力位上方触发散户 FOMO 买入和空头止损，随后反转下跌。
- **识别方法**：观察关键价位附近的订单簿变化（大量挂单突然出现或消失）、成交量分布异常。

**其他注意事项：**
- 币圈 7x24 运行，无收盘价概念——使用 UTC 0:00 作为日线收盘参考。
- 周末和节假日流动性低，技术信号可靠性下降。
- Funding rate 极端值（年化 > 100%）通常预示短期反转。

---

## 4. 链上层面（On-Chain Layer）

### 4.1 巨鲸抛售信号检测

**核心工具：Arkham Intelligence + Nansen**

| 工具 | 优势 | 做空应用 |
|------|------|----------|
| **Arkham** | 超强实体识别引擎，多链覆盖（BTC/ETH/SOL/AVAX/TRX 等），KOL 钱包标记 | 追踪已知大户地址的异常转出 |
| **Nansen** | AI 驱动的"智能钱包"分类，按行为（机构、熟练交易者、鲸鱼）和盈利能力分级 | 关注高胜率钱包群体的集体行为变化 |

**做空信号组合：**
1. Arkham 检测到已知机构/大户钱包向交易所大额转入
2. Nansen 显示"Smart Money"群体净卖出增加
3. 交叉验证：多个独立大户同时减仓而非单一大户操作

**2025 年极端案例：**
- 一个大户在特朗普宣布中国关税前几分钟开出 > 1 亿美元的 BTC 空单，最终获利约 1.92 亿美元，触发了 193 亿美元的清算。
- Glassnode 的 Accumulation Trend Score 在 2025 年末达到 0.99/1.0——表明鲸鱼在积极积累而非分发。

**重要提示：** 交易所流入不一定等于卖压。鲸鱼可能在重新调仓而非清仓。需结合宏观周期阶段判断（分发期的流入 vs 积累期的流入含义完全不同）。

### 4.2 交易所净流入激增 = 卖压信号

**核心指标：Exchange Netflow**

- **净流入**（Netflow > 0）：代币从外部钱包转入交易所，通常暗示卖出意图。
- **净流出**（Netflow < 0）：代币从交易所提取到外部钱包，通常暗示囤币意图。

**做空信号判定标准：**
- 单日净流入量超过 30 日均值的 **3 倍以上**
- 连续 3-5 天净流入
- 净流入集中在单一或少数交易所（可能暗示特定实体在准备出货）

**数据来源：** CryptoQuant 的 Exchange Netflow 指标、Glassnode 的 Exchange Balance 变化。

### 4.3 稳定币流出趋势

**核心逻辑：** 稳定币是加密市场的"弹药"。稳定币供应量增长 = 潜在购买力增加；稳定币供应量收缩 = 购买力减少。

**2025 年数据：**
- 稳定币总市值在 2025 年达到历史新高，Q1 增长 9.61%。
- USDT 市值达 1750 亿美元，USDC 增速（73%）超过 USDT（36%）。
- 交易量：2025 年稳定币总交易量达 33 万亿美元（同比增长 72%），其中 USDC 以 18.3 万亿领先。

**做空信号：**
- **USDT dominance 上升**：资金从波动资产轮换到稳定币避险——risk-off 信号。
- **USDT 流通供应量收缩**：以 FTX 崩溃以来最快速度收缩时，暗示系统性风险。
- **交易所稳定币余额下降**：购买力减少，卖压消化能力减弱。
- 大额稳定币转移高度集中在少数机构实体——关注这些实体的行为方向。

### 4.4 长期持有者开始移动代币

**核心指标（来自 CryptoQuant/Glassnode）：**

| 指标 | 做空信号 |
|------|----------|
| **MVRV（市值/已实现价值）** | MVRV 过高后开始回落 |
| **SOPR（花费产出利润率）** | SOPR 从高位跌破 1.0 |
| **LTH-SOPR** | 长期持有者 SOPR 显著上升（获利了结） |
| **长期持有者供应量** | 持续下降（分发阶段） |
| **Coin Days Destroyed** | 激增，表示长期休眠代币被移动 |

**2025 年 CryptoQuant 熊市信号：**
- Bull Score 下降
- MVRV 冷却
- SOPR 跌破 1.0
- 长期持有者已完成分发
- 矿工抛售储备
- 稳定币流动性停滞

**注意：** 这些信号也与牛市深度回调一致（尤其在杠杆率高、ETF 资金流波动大的环境中），不一定确认熊市。

### 4.5 智能钱包仓位变化

**分析方法（Nansen）：**

1. **Smart Money 分类**：Nansen 按钱包的历史表现（胜率、实现 PnL）将钱包分为不同等级。
2. **群体行为追踪**：当高胜率钱包群体集体减仓或转向稳定币时，信号强度最高。
3. **交叉验证**：
   - Arkham 用于身份归因（"这个地址属于谁"）
   - Nansen 用于绩效追踪（"这个地址的历史表现如何"）
   - Glassnode 用于宏观周期定位（"当前处于积累、分发还是投降阶段"）

**最佳实践：**
- 先用 Glassnode 确定市场周期阶段
- 在分发阶段密切关注鲸鱼动向
- 在积累阶段忽略常规卖出行为
- 投降阶段不做空（风险回报比不利）

---

## 5. 做市商状态层面（Market Maker Layer）

### 5.1 做市商库存压力评估

**核心逻辑：** 做市商持有大量库存（多头敞口）时，有内在的抛售压力；库存偏低时，可能需要从市场上买入补充。

**评估方法：**

- **订单簿不对称性**：做市商卖单显著多于买单 = 库存偏高，有出货压力。
- **Kaiko 市场微结构数据**：分析最佳买卖价附近 0-10% 范围内的挂单量（Market Depth）。
- **买卖价差变化**：做市商压力增大时，价差可能收窄（急于出货）或扩大（流动性撤离）。

**做空信号：**
- 做市商在卖方明显挂出超额订单
- 在价格上涨过程中，卖方深度持续增加（做市商在出货）

### 5.2 做市商资金成本和合约到期时间

**关键因素：**

- **资金成本**：做市商的借贷成本。当利率上升时，持有库存的成本增加，做市商更倾向于减仓。
- **合约到期**：季度期货/期权到期日前后是流动性事件。
  - 2025 年末有价值 270 亿美元的 BTC 和 ETH 期权到期。
  - 到期前做市商需要对冲调仓，可能引发价格波动。
- **Laevitas 数据**：提供完整的期权链和交易流数据，涵盖隐含波动率、Greeks 和深度历史数据。

**做空信号：**
- 大量期权到期前，Put/Call 比率上升（看跌期权需求增加）
- 做市商的 delta 对冲行为在大量期权到期时可能放大价格波动
- 期权 skew（偏斜度）在所有期限上偏向看跌 = 做市商预期下行

### 5.3 做市商控盘能力 vs 散户筹码分散度

**评估框架：**

| 维度 | 做市商强控盘 | 散户筹码分散 |
|------|-------------|-------------|
| 订单簿特征 | 少数大单主导 | 大量小单分布 |
| 价格行为 | 走势"干净"，假信号少 | 走势杂乱，scam wick 多 |
| 清算分布 | 清算集中在特定价位 | 清算均匀分布 |
| 做空难度 | 需判断做市商意图 | 技术信号相对可靠 |

**做空启示：**
- 做市商强控盘时，不要对抗做市商方向——判断做市商意图后顺势操作。
- 散户筹码分散时，技术面信号和清算热力图更有参考价值。

### 5.4 做市商流动性撤离信号

**2025 年 10 月崩盘案例分析：**

2025 年 10 月 10 日，加密市场经历史上最大规模清算事件——25 分钟内约 190 亿美元杠杆仓位被清算。BTC 从 $126,000 跌至 $105,000，ETH 跌 12%，部分山寨币单日腰斩。

**关键发现：**

- **流动性高度顺周期**：情绪乐观时交易量激增（但几乎全是单边"热钱"），恐慌到来时无人接盘。
- **做市商撤离后遗症**：崩盘后主要资产的订单簿深度远低于 10 月初水平，表明做市商系统性撤退而非临时调整。
- **对冲失效**：市中性/库存对冲策略的空头期货腿被交易所部分或完全平仓，将对冲转化为已实现亏损，剩余风险敞口裸露。
- **基础设施压力**：链上桥接、内部转账系统、法币通道均在崩盘中承压。

**流动性撤离的前兆信号：**
1. **Kaiko 数据**：0.1% 和 1% 市场深度持续下降
2. **买卖价差扩大**：尤其在非主流交易对上
3. **订单簿"闪烁"**：大量挂单反复出现和撤销
4. **跨交易所价差扩大**：同一资产在不同交易所的价差异常增大

### 5.5 做市商与做空方向一致 = 加分项

**最高置信度做空信号：**

当以下条件同时满足时，做空胜率最高：

1. **做市商在出货**：卖方订单簿深度异常增加，大单分布在关键阻力位上方
2. **衍生品市场倾斜**：
   - Put/Call 比率上升（2025 年末 BTC 为 0.38，偏看涨——反例）
   - 期权 skew 偏向看跌
   - Funding rate 极高（年化 > 50-100%）后开始下降
3. **链上数据配合**：鲸鱼向交易所转入 + 长期持有者分发
4. **宏观环境支持**：DXY 上升 + 美债收益率上升 + 流动性收缩

**2025 年实例：**
- 2025 年 10 月崩盘中，至少一个大型玩家通过激进做空 ETH 获利超 2 亿美元，利用了流动性真空并加剧了市场恐慌。
- 这展示了当做市商撤离流动性 + 大户做空方向一致时的毁灭性效果。

---

## 6. 五层综合评分与实操建议

### 6.1 综合评分表

| 层面 | 做空信号 | 权重 | 分数范围 |
|------|----------|------|----------|
| **宏观** | DXY 上升 + 美债收益率上升 + M2 收缩 + RRP 上升 | 25% | 0-25 |
| **叙事** | Fear & Greed > 75 + 社交过热 + 新钱包增速放缓 | 15% | 0-15 |
| **技术** | K 线看空形态 + 量价背离 + 清算密集区配合 | 20% | 0-20 |
| **链上** | 鲸鱼抛售 + 交易所净流入 + 稳定币流出 + LTH 分发 | 25% | 0-25 |
| **做市商** | 库存压力 + 流动性撤离 + 做市商做空方向一致 | 15% | 0-15 |
| **总分** | | 100% | **0-100** |

### 6.2 评分阈值

| 总分 | 行动 |
|------|------|
| **80-100** | 全力做空——多层信号共振，历史上极少出现但胜率极高 |
| **60-79** | 中等仓位做空——注意止损和仓位管理 |
| **40-59** | 谨慎观望——信号混杂，不宜重仓 |
| **0-39** | 不做空——做空条件不充分，可能反向被轧空 |

### 6.3 实操注意事项

1. **杠杆控制**：做空杠杆建议不超过 3x（考虑到加密市场的极端波动和 scam wick 风险）。
2. **分批入场**：不一次性建满仓位，分 2-3 批在不同价位入场。
3. **止损纪律**：基于 ATR 设置止损，一般为入场价上方 1.5-2 倍 ATR。
4. **时间框架**：宏观做空持仓数周-数月，叙事做空持仓数天-数周，技术做空持仓数小时-数天。
5. **对冲**：可使用 put 期权替代直接做空，锁定最大亏损。
6. **流动性风险**：在流动性低的时段（周末、节假日）减少敞口或不交易。
7. **黑天鹅保护**：始终预留应对极端行情的资金和方案。

---

## 参考来源

### 宏观与流动性分析
- [Arthur Hayes Predicts Crypto Peak in March 2025 Amid $612B Liquidity Surge](https://bitcoinethereumnews.com/crypto/arthur-hayes-predicts-crypto-peak-in-march-2025-amid-612b-liquidity-surge/)
- [Arthur Hayes: Bitcoin Tracks Fed's RRP, Mid-March Peak Expected](https://beincrypto.com/arthur-hayes-predicts-crypto-market-peak/)
- [Arthur Hayes Deploys Net Liquidity Strategy](https://cryptonews.com/news/arthur-hayes-net-liquidity-bitcoin-strategy/)
- [Arthur Hayes: How Will Dollar Liquidity Drive the Next Bull Market in Crypto by 2025?](https://www.theblockbeats.info/en/news/56466)
- [Bitcoin's Potential Surge Amid the Fed's 2025 Rate Cut: A Macro-Driven Playbook](https://www.ainvest.com/news/bitcoin-potential-surge-fed-2025-rate-cut-macro-driven-playbook-2509/)
- [DXY vs. Bitcoin: 2026 Correlation Shift Explained](https://www.osl.com/hk-en/academy/article/the-us-dollar-index-vs-bitcoin-why-the-inverse-correlation-matters)
- [Bitcoin analysis: dollar correlation, state reserves, and 2025 projections](https://cryptovalleyjournal.com/focus/background/bitcoin-analysis-dollar-correlation-state-reserves-and-2025-projections/)
- [How Treasury Yield Spreads Are Influencing Bitcoin Prices in 2025](https://get.ycharts.com/resources/blog/how-treasury-yields-are-influencing-crypto-and-what-advisors-need-to-know/)
- [US Treasury Yields Surge: How Soaring 4.42% Rate Crushes Risk Asset Appeal](https://bitcoinworld.co.in/treasury-yields-risk-assets-cryptocurrency-impact/)

### M2 与流动性
- [Bitcoin and Global liquidity (M2) lead 10 weeks](https://charts.bgeometrics.com/m2_global_10w.html)
- [Does M2 Money Supply Predict Bitcoin Price?](https://blog.traderspost.io/article/m2-money-supply-bitcoin-correlation-explained)
- [Global M2 money supply shifted by 90 days predicts Bitcoin price](https://cryptoslate.com/bitcoins-has-an-elastic-relationship-with-global-m2-money-supply-shifted-by-90-days/)
- [The Correlation Between Bitcoin and M2 Money Supply Growth](https://sarsonfunds.com/the-correlation-between-bitcoin-and-m2-money-supply-growth-a-deep-dive/)

### 叙事与情绪
- [Cobie Substack](https://cobie.substack.com/)
- [Cobie: Tokens in the attention economy](https://cobie.substack.com/p/tokens-in-the-attention-economy)
- [Cobie: Probabilistic thinking](https://cobie.substack.com/p/probabilistic-thinking)
- [March 2025: Crypto Fear & Greed Index](https://trustwallet.com/blog/cryptocurrency/march-2025-crypto-fear-greed-index)
- [Crypto sentiment is trapped in extreme fear](https://cryptoslate.com/bitcoin-2025-sentiment-collapse-performance-gap/)
- [Crypto Fear and Greed Index | CoinMarketCap](https://coinmarketcap.com/charts/fear-and-greed-index/)

### 技术分析与衍生品
- [CoinGlass Liquidation Heatmap](https://www.coinglass.com/pro/futures/LiquidationHeatMap)
- [CoinGlass Liquidation Map](https://www.coinglass.com/pro/futures/LiquidationMap)
- [How to use Liquidation Heatmaps to assist trading](https://www.coinglass.com/learn/how-to-use-liqmap-to-assist-trading-en)
- [ETH liquidation heatmap flags near-$2,000 trapdoor](https://crypto.news/eth-liquidation-heatmap-flags-near-2000-trapdoor-for-leveraged-longs/)
- [Scamwicks and Stop Cascades](https://www.machow.ski/posts/scamwicks-and-stop-cascades/)
- [Laevitas - Derivatives Data](https://www.laevitas.ch/)
- [Laevitas Options Skew & BF](https://app.laevitas.ch/assets/options/skew-bf/ETH/deribit)
- [How do crypto derivatives market signals predict price movements](https://www.gate.com/crypto-wiki/article/how-do-crypto-derivatives-market-signals-predict-price-movements-futures-open-interest-funding-rates-liquidation-data-long-short-ratio-and-options-explained-20260129)

### 链上分析
- [Bitcoin on-chain data flashed critical bearish signal (CryptoQuant)](https://cryptoslate.com/bitcoin-on-chain-data-just-flashed-critical-bearish-signal-that-cryptoquant-warns-marks-a-verified-cycle-top/)
- [Glassnode: The Uptober Breakout](https://insights.glassnode.com/the-week-onchain-week-40-2025/)
- [Glassnode Research](https://insights.glassnode.com/)
- [CryptoQuant Research](https://cryptoquant.com/insights/research)
- [Nansen: How to Track Smart Money Crypto Accumulation](https://www.nansen.ai/post/how-to-track-smart-money-crypto-accumulation-ultimate-guide)
- [Nansen: How to Monitor Crypto Wallet Activity](https://www.nansen.ai/post/how-to-monitor-crypto-wallet-activity-track-smart-money)
- [How To Track Whales Using On-Chain Analytics Tools (Arkham, Nansen)](https://onchainstandard.com/guides-education/track-whales-using-chain-analytics-tools/)

### 做市商与市场微结构
- [How $150 billion was liquidated from crypto market in 2025](https://cryptoslate.com/how-150-billion-was-liquidated-from-crypto-market-in-2025-driving-bitcoin-crash/)
- [Crypto Liquidity Still Hollow After October Crash](https://www.coindesk.com/markets/2025/11/15/crypto-liquidity-still-hollow-after-october-crash-risking-sharp-price-swings)
- [$20 Billion Crypto Crash: What Liquidations Reveal About Market Integrity](https://www.soliduslabs.com/post/when-whales-whisper-inside-the-20-billion-crypto-meltdown)
- [Crypto Crash Oct 2025: Leverage Meets Liquidity (FTI Consulting)](https://www.fticonsulting.com/insights/articles/crypto-crash-october-2025-leverage-met-liquidity)
- [Kaiko: Understanding Centralized Exchange Liquidity Data](https://research.kaiko.com/insights/centralized-exchange-liquidity)
- [Kaiko: Liquidity Lowdown Asset Ranking](https://research.kaiko.com/insights/liquidity-lowdown-asset-ranking)

### 稳定币数据
- [2025 Crypto Adoption and Stablecoin Usage Report (TRM Labs)](https://www.trmlabs.com/reports-and-whitepapers/2025-crypto-adoption-and-stablecoin-usage-report)
- [Stablecoin Transactions Soared 72% in 2025 (Yahoo Finance)](https://finance.yahoo.com/news/stablecoin-transactions-soared-72-2025-054951388.html)
- [USDC Growth Rate Surpasses USDT in 2025](https://www.kucoin.com/news/flash/usdc-growth-rate-surpasses-usdt-in-2025-signals-stablecoin-shift)
