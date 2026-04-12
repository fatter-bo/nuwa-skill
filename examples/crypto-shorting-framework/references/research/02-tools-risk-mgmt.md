# 做空工具选择与风险管理框架

> 调研日期：2026-04-12
> 状态：深度调研完成

---

## A. 做空工具选择

### A1. 永续合约 vs 交割合约 vs 期权(Put) 适用场景对比

| 维度 | 永续合约 (Perpetual Swap) | 交割合约 (Delivery Futures) | 看跌期权 (Put Option) |
|------|--------------------------|---------------------------|---------------------|
| **到期日** | 无到期日，可无限持仓 | 固定到期日（周/月/季度） | 固定到期日 |
| **结算方式** | 资金费率机制维持锚定现货 | 到期时按结算价清算 | 到期时按行权价结算或放弃 |
| **最大亏损** | 理论无限（无止损时） | 理论无限（无止损时） | 仅限已付权利金 |
| **资金效率** | 高（支持高杠杆） | 中等 | 低（需全额支付权利金） |
| **持仓成本** | 资金费率（可正可负） | 基差（contango/backwardation） | 时间价值衰减 (Theta Decay) |
| **流动性** | 最优（占衍生品市场约75%） | 中等 | 较低（Deribit集中） |
| **适用场景** | 短中期趋势做空、日内交易 | 事件驱动（如升级前后）、跨期套利 | 黑天鹅对冲、小币做空、风险可控的方向性押注 |

**核心决策逻辑**：

1. **短期趋势做空（1-30天）** --> 永续合约。流动性最好，进出方便，资金费率在熊市中通常为负值（即做空方收取费用）
2. **事件驱动做空（锁定特定日期）** --> 交割合约。无资金费率风险，到期自动结算，适合围绕已知事件布局
3. **尾部风险对冲 / 小币做空** --> Put期权。亏损上限明确，不怕被"定点爆破"，适合不确定性极高的场景
4. **长期结构性做空（>3个月）** --> Put期权或滚动交割合约。永续合约长期持仓的资金费率成本不可预测

### A2. 交易所做空成本差异

#### 资金费率对比

| 交易所 | 结算频率 | BTC永续典型费率 | 山寨币费率特征 | 备注 |
|--------|---------|----------------|--------------|------|
| **Binance** | 8小时（部分4小时） | +0.01%~+0.03%/8h（牛市） | 波动大，极端时可达 +/- 0.5%/8h | 市占率29.3%，流动性最佳 |
| **OKX** | 8小时 | 与Binance接近 | 山寨币种类丰富 | 市占率约21%，支持组合保证金 |
| **Bybit** | 8小时 | 与Binance接近 | 新币上线速度快 | 市占率约21%，统一交易账户 |
| **Deribit** | 8小时 | 略低于CEX均值 | 仅BTC/ETH/SOL等主流币 | 期权市场占85%份额，专业衍生品 |

**关键数据**（截至2026年初）：
- BTC永续平均资金费率：+0.51%/8h（约70.2% APR），反映持续的多头偏好
- 熊市/恐慌期：资金费率可转为-0.01%至-0.1%/8h，此时做空需支付费用
- 山寨币资金费率波动远大于BTC，极端行情下单次结算可达 +/- 1%

#### 借币利率（用于现货做空/杠杆交易）

- Binance/OKX/Bybit 的借币利率通常为年化 5%-30%，取决于币种供需
- 热门做空标的借币利率可能飙升至年化 50%-200%（类似传统市场的"hard to borrow"）
- 借币利率飙升本身即为轧空前兆信号

#### 做空成本优化建议

1. 使用 [CoinGlass](https://www.coinglass.com/FundingRate) 实时对比各交易所费率，选择费率最低的交易所开仓
2. 在资金费率极正时开空（做空方收取费用），避免在极负时开空
3. 考虑交易所间资金费率套利：在费率高的交易所做空，在费率低的交易所做多

### A3. 杠杆倍数选择公式

**核心公式**：

```
最大杠杆倍数 = 可承受亏损比例 / 止损百分比
```

**示例计算**：

| 场景 | 可承受亏损（占总资金） | 止损百分比 | 最大杠杆 |
|------|----------------------|-----------|---------|
| 保守做空 BTC | 2% | 5% | 0.4x（实际不需杠杆） |
| 标准做空 BTC | 2% | 3% | 0.67x |
| 激进做空山寨币 | 5% | 10% | 0.5x |
| 事件驱动短线 | 3% | 2% | 1.5x |

**爆仓价格计算**（做空）：

```
爆仓价 = 开仓价 x (1 + 1/杠杆倍数 - 维持保证金率)
```

例：10x杠杆做空BTC于$60,000，维持保证金率1%
- 爆仓价 = $60,000 x (1 + 0.1 - 0.01) = $65,400（仅需涨9%即爆仓）

**实操原则**：
1. **永远不要满杠杆**：最大可用杠杆 =/= 应使用杠杆
2. **做空杠杆应低于做多**：因做空亏损理论无限，而做多最多归零
3. **山寨币杠杆 <= 3x**：波动率远高于BTC，滑点和插针风险大
4. **BTC做空杠杆 <= 5x**：即便是最成熟的标的也应保守
5. 始终将止损距离纳入杠杆计算，而非反过来

### A4. 小币做空：为什么优先用期权而非合约

**合约做空小币的三大致命风险**：

#### 1. 滑点风险
- 小币永续合约的订单簿深度极浅，大单进出可能造成 5%-15% 的滑点
- 做空时滑点使实际开仓价低于预期，止损时滑点使实际平仓价高于预期，双向吃亏
- 在恐慌或FOMO行情中，滑点可能扩大至 20%+

#### 2. 定点爆破（Pin Attack / Stop Hunting）
- 做市商/项目方可以在流动性极薄时拉盘插针，精准触发清算
- 小币的做空持仓数据对项目方和做市商是可见的（通过交易所API或链上数据）
- 只需少量资金即可将价格推高 20-50%，触发大量空单清算，形成级联清算
- 清算后价格迅速回落，但你的仓位已经被强平

#### 3. 流动性突然消失
- 小币做市商可能突然撤出流动性，导致价格剧烈波动
- 新做市商接手时往往伴随剧烈的价格发现过程

**为什么期权更安全**：

| 对比维度 | 合约做空小币 | 期权做空小币 |
|----------|------------|------------|
| 最大亏损 | 无限 | 权利金（已知、固定） |
| 插针爆仓 | 会被强制清算 | 不会，持有至到期 |
| 滑点影响 | 开仓+平仓双重影响 | 仅开仓时一次 |
| 做市商反杀 | 极易被针对 | 结构性保护 |
| 时间成本 | 资金费率不确定 | Theta确定衰减 |

**实操策略**：
- 买入虚值（OTM）Put期权，行权价设在预期下跌目标位
- 权利金视为"做空保险费"，即便判断错误也不会超额亏损
- 对于Deribit未上线的小币，考虑通过场外期权（OTC，如Paradigm撮合）获取敞口

### A5. 做空ETF vs 直接做空

#### ProShares BITI（Short Bitcoin Strategy ETF）

| 维度 | BITI做空ETF | 直接合约做空 |
|------|-----------|------------|
| **机制** | 持有CME BTC期货空头，目标-1x日收益 | 直接在交易所开空仓 |
| **费用** | 年化0.95%管理费 + 期货展期成本 | 资金费率（可正可负） |
| **杠杆** | 无杠杆（-1x） | 可选1x-125x |
| **准入门槛** | 证券账户即可，无需加密交易所 | 需KYC注册加密交易所 |
| **交易时间** | 美股交易时间 | 7x24小时 |
| **长期持有** | 展期损耗 + 波动率拖累，长期偏离严重 | 持续费率支出但无展期损耗 |
| **适用人群** | 传统投资者、合规要求高的机构 | 加密原生交易者 |

**关键缺陷**：
- BITI仅追踪BTC日收益的-1x，持有超过一天会产生**波动率拖累（Volatility Drag）**
- 期货展期每月滚动，contango结构下做空方额外承担正向展期成本
- 无法做空山寨币，仅限BTC

**结论**：对于活跃的加密交易者，直接在交易所做空几乎在所有维度都优于做空ETF。BITI仅适合不愿/不能开设加密交易所账户的传统投资者作为**短期战术性工具**使用。

---

## B. 风险管理

### B1. 做空的非对称风险与对冲

#### 风险本质

```
做多：最大亏损 = 100%（跌至零）
做空：最大亏损 = 无限（理论上价格可无限上涨）

做多 $50 -> 跌至 $0 = 亏损 $50（100%）
做空 $50 -> 涨至 $500 = 亏损 $450（900%）
```

这种**非对称性**是做空最核心的风险特征：收益有限（最多100%），亏损无限。

#### 虚值Call对冲策略（Protective Call）

**策略构建**：
```
做空仓位：永续合约/交割合约空单
+ 对冲仓位：买入虚值（OTM）Call期权

做空 1 BTC @ $60,000
+ 买入 1 BTC Call @ 行权价 $66,000（10% OTM）
  权利金：$1,200（约2%）
```

**效果**：
- BTC跌至$50,000：空单盈利$10,000，Call期权过期作废，净盈利 = $10,000 - $1,200 = $8,800
- BTC涨至$80,000：空单亏损$20,000，Call期权盈利$14,000，净亏损 = $20,000 - $14,000 + $1,200 = $7,200
- **最大亏损被锁定**：开仓价到行权价的距离 + 权利金 = $6,000 + $1,200 = $7,200

**对冲比例选择**：

| 风险偏好 | Call行权价 | OTM程度 | 权利金成本 | 保护程度 |
|---------|-----------|--------|-----------|---------|
| 保守 | +5% | 浅虚值 | 高（3-5%） | 强 |
| 标准 | +10% | 中度虚值 | 中（1.5-3%） | 中等 |
| 激进 | +20% | 深虚值 | 低（0.5-1.5%） | 仅防黑天鹅 |

**最佳实践**：
- 每笔做空仓位都应配套Call对冲，视权利金为"保险费"
- 期权到期日应覆盖预计持仓周期+缓冲（建议多加1-2周）
- Deribit占据85%的加密期权市场份额，是首选平台
- 对冲成本计入交易计划的盈亏平衡计算

### B2. 轧空（Short Squeeze）前兆信号

轧空是做空者最大的系统性威胁。以下信号应持续监控：

#### 信号清单（按紧急度排序）

| 信号 | 数据来源 | 危险阈值 | 紧急度 |
|------|---------|---------|--------|
| **资金费率极负** | CoinGlass / CoinAnk | < -0.05%/8h 持续超过24h | !!!! |
| **借币利率飙升** | 交易所借贷市场 | 年化 > 50% 或 24h内翻倍 | !!!! |
| **清算地图空单堆积** | CoinGlass Liquidation Heatmap | 当前价上方5-10%处有密集清算带 | !!! |
| **突然大额现货买入** | 链上数据 / 交易所大单监控 | 单笔 > 日均成交量5% | !!! |
| **OI（未平仓合约）快速上升 + 价格横盘/下跌** | CoinGlass | OI增速 > 10%/24h | !! |
| **交易所BTC净流出** | Glassnode / CryptoQuant | 大额净流出 > 10,000 BTC/日 | !! |
| **社交媒体做空情绪极端一致** | 定性判断 | "全员看空"即危险 | ! |

#### 轧空的机械过程

```
资金费率极负（空头拥挤）
    --> 价格小幅上涨触及第一层空单清算区
    --> 清算产生强制买入
    --> 价格进一步上涨触及更多清算区
    --> 级联清算形成正反馈循环
    --> 空头恐慌平仓加剧上涨
    --> 短时间内价格暴涨 20%-50%+
```

#### 历史案例

**GameStop (2021.01)**：
- 空头持仓达流通股的140%
- 散户协调买入 --> 级联清算
- 价格从$17涨至$500+（30倍），多家对冲基金巨亏

**BTC 2026.02**：
- 资金费率跌至-6%以下（6个月低点）
- 市场极度看空共识
- 随后出现显著的轧空反弹

**教训**：当"所有人都在做空"时，就是最危险的做空时刻。

### B3. 做市商反杀风险评估

做市商（Market Maker）拥有信息优势和流动性控制能力，在以下6个条件同时或多数满足时，做空面临被"反杀"的极高风险：

#### 6大危险条件

| # | 条件 | 如何检测 | 风险等级 |
|---|------|---------|---------|
| 1 | **做市商正在吸筹** | 链上大额转入做市商关联地址；交易所挂单模式异常（冰山单） | 极高 |
| 2 | **资金费率极负** | CoinGlass显示连续负费率，且负值在加深 | 极高 |
| 3 | **项目方有利好待发** | 即将上线主网/重大合作/代币销毁/回购；关注项目路线图和社交媒体 | 高 |
| 4 | **流动性集中在少数地址** | 10-20个钱包持有60%+流通量；可通过链上分析工具验证 | 高 |
| 5 | **新做市商刚接手** | 项目方宣布更换做市商；订单簿风格突变（如从薄变厚） | 中高 |
| 6 | **订单簿买盘异常厚** | 买方深度远超卖方深度；出现大量隐藏的冰山买单 | 中高 |

#### 风险评分模型

```
反杀风险得分 = 满足条件数量

0-1个：低风险，正常做空
2-3个：中高风险，降低仓位至标准的50%，加紧止损
4-5个：极高风险，建议不做空或仅用Put期权
6个全满足：禁止做空，这是教科书级的反杀布局
```

#### 做市商反杀的典型模式

1. **前期**：做市商悄悄在低价吸筹，同时维持弱势盘面诱空
2. **中期**：空头持仓不断增加，资金费率转负
3. **触发**：做市商利用手中筹码+配合利好消息，一次性拉盘
4. **清算**：价格快速突破空单密集清算区，形成级联清算
5. **收割**：做市商在高位出货给被迫平仓的空头

### B4. 仓位管理

#### 单笔仓位上限

| 账户规模 | 单笔做空上限 | 总做空敞口上限 | 备注 |
|---------|------------|-------------|------|
| < $10K | 2-3% | 5% | 小账户更需保守 |
| $10K-$100K | 2-5% | 10-15% | Jim Chanos标准：0.5%-5% |
| $100K-$1M | 1-3% | 10% | 分散化更重要 |
| > $1M | 0.5-2% | 8% | 流动性冲击需考虑 |

#### 分批建仓 vs 一次性建仓

**分批建仓（推荐）**：

```
第1批：30%仓位 -- 初始确认信号时入场
第2批：30%仓位 -- 价格反弹至阻力位确认失败时加仓
第3批：40%仓位 -- 趋势确认、破关键支撑时完成建仓

每批之间间隔：至少4-8小时，观察市场反应
```

**一次性建仓的适用场景**：
- 事件驱动型交易，窗口期极短（如利空消息刚出时）
- 已通过期权对冲了下行风险
- 超高确信度 + 明确的止损位

#### 浮盈加仓条件

**严格的三重确认**：
1. **趋势确认**：价格已跌破关键支撑位且未反弹
2. **盈利缓冲**：现有仓位浮盈 >= 新加仓位的止损空间（即加仓后最坏情况仍盈利或持平）
3. **信号未弱化**：做空的基本面/技术面逻辑仍成立，未出现轧空信号

**浮盈加仓的金字塔法则**：
```
每次加仓量 <= 前一次的50%
例：初始 100单位 -> 加仓 50单位 -> 再加仓 25单位
绝不倒金字塔（加仓越来越大）
```

### B5. 止损纪律

#### 硬止损 vs 移动止损

| 类型 | 机制 | 适用场景 | 优缺点 |
|------|------|---------|--------|
| **硬止损** | 固定价格触发平仓 | 所有做空仓位（必须） | 简单明确，但可能被插针触发 |
| **移动止损（Trailing Stop）** | 跟随价格下跌自动调整 | 趋势做空已有浮盈后 | 锁定利润，但震荡市容易被震出 |
| **时间止损** | 持仓超过预设时间自动平仓 | 事件驱动型交易 | 避免无效持仓消耗资金费率 |
| **波动率止损** | 基于ATR（平均真实波幅）设定 | 高波动币种 | 更贴合市场节奏 |

**移动止损实操（做空）**：
```
做空入场价：$100
初始硬止损：$108（-8%，必须设定）

价格跌至 $90 --> 移动止损调整至 $99（10%回撤）
价格跌至 $80 --> 移动止损调整至 $88（10%回撤）
价格跌至 $70 --> 移动止损调整至 $77（10%回撤）

移动止损只能向下调（对做空而言），永远不能向上调
```

#### "再等等"心理的对抗

这是做空亏损的头号心理陷阱。价格触及止损位时，交易者倾向于：
- "再给它一点空间"
- "应该马上就会跌了"
- "刚才只是假突破"
- "我不想在最高点被止损"

**对抗机制**：

1. **止损自动化**：开仓时立即挂好止损单（条件单），消除手动执行时的犹豫
2. **事前承诺**：写下交易计划（包括止损位），触发即执行，不二次决策
3. **损失预接受**：开仓前就将止损金额"心理记账"为已花费的成本
4. **认知重构**：止损不是"亏损"，是"买保险"——保护了账户其余95%的资金
5. **记录违规**：每次违反止损纪律都记录，定期复盘。违规次数过多则强制停止交易

#### 冷静期规则

| 触发条件 | 冷静期时长 | 恢复条件 |
|---------|-----------|---------|
| 单笔亏损 > 账户3% | 24小时 | 书面复盘完成 |
| 连续2笔做空止损 | 48小时 | 书面复盘 + 重新评估做空论点 |
| 连续3笔做空止损 | 1周 | 全面策略审查 |
| 单日亏损 > 账户5% | 3天 | 书面复盘 + 减半仓位上限 |
| 违反止损纪律 | 48小时 | 写检讨 + 审查自动化止损设置 |

### B6. "禁止做空"场景清单

以下场景出现任一条，应**绝对禁止做空**或**立即平仓退出**：

#### 市场结构层面

- [ ] **牛市初期/中期确认**：BTC突破前高且站稳，周线级别上升趋势确立
- [ ] **ETF大规模净流入**：连续5天+净流入，机构资金入场信号
- [ ] **减半后6-12个月**：历史上每次减半后都有显著上涨
- [ ] **宏观利好共振**：美联储降息周期 + 美元走弱 + 风险偏好回升

#### 标的层面

- [ ] **刚经历暴跌（>30%）**：技术性反弹概率极高，此时做空是在"接飞刀"
- [ ] **做市商反杀信号满4+**：参见B3的6条件评估
- [ ] **轧空前兆信号满3+**：参见B2的信号清单
- [ ] **项目方即将公布重大利好**：主网上线、重大合作、代币回购
- [ ] **代币流通量极低**：低流通 + 高FDV = 极易被操纵
- [ ] **刚上线新交易所**：上线效应带来短期买盘
- [ ] **没有可用的期权对冲**：小币无法买Call保护时，不应用合约做空

#### 个人状态层面

- [ ] **处于冷静期内**
- [ ] **情绪化交易**：因FOMO或报复心理想做空
- [ ] **无明确交易计划**：没有写下入场理由、止损位、目标位
- [ ] **已达到总做空敞口上限**
- [ ] **睡前/无法盯盘时**：不开新的高杠杆做空仓位
- [ ] **连续亏损后急于回本**：这是最危险的心理状态

---

## 附录

### 工具与数据源

| 用途 | 工具 | 链接 |
|------|------|------|
| 资金费率对比 | CoinGlass | https://www.coinglass.com/FundingRate |
| 资金费率历史 | CoinAnk | https://coinank.com/fundingRate/current |
| 清算地图/热力图 | CoinGlass Pro | https://www.coinglass.com/pro/futures/LiquidationHeatMap |
| 期权交易 | Deribit | https://www.deribit.com |
| 场外期权 | Paradigm | https://www.paradigm.co |
| 链上分析 | Glassnode / CryptoQuant | https://glassnode.com |
| 费率套利扫描 | ArbitrageScanner | https://arbitragescanner.io/funding-rates |
| 做空ETF | ProShares BITI | https://www.proshares.com/our-etfs/leveraged-and-inverse/biti |

### 关键公式速查

```
最大杠杆 = 可承受亏损比例 / 止损百分比
爆仓价（空） = 开仓价 x (1 + 1/杠杆 - 维持保证金率)
对冲后最大亏损 = (Call行权价 - 开仓价) + Call权利金
浮盈加仓安全线 = 现有浮盈 >= 新仓位止损金额
年化资金费率 = 8h费率 x 3 x 365
```

### Jim Chanos 做空方法论要点

1. **仓位控制**：单一空头仓位限制在资金的0.5%-5%
2. **分批建仓**：从小仓位起步，随更多证据确认逐步加仓
3. **多元分散**：跨行业分散做空，单一错误不致命
4. **耐心持有**：愿意持仓数月甚至数年等待论点兑现
5. **止损纪律**：仓位过大时主动减仓，论点证伪时果断止损
6. **期权保护**：使用期权对冲组合层面的尾部风险
7. **寻找结构性缺陷**：关注会计造假、行业颠覆、消费趋势衰退、技术过时四大主题

---

## 参考来源

- [Perpetual Futures vs. Traditional Futures - Sei Network](https://blog.sei.io/trading/perps/perpetual-futures-vs-traditional-futures/)
- [Perpetual Futures vs. Options - Drift](https://www.drift.trade/learn/perpetual-futures-vs-options)
- [Options vs Perpetual Contracts vs Futures Contracts - OSL](https://www.osl.com/hk-en/academy/article/options-vs-perpetual-contracts-vs-futures-contracts)
- [Expert Guide to Perpetual Swap Contracts Trading 2026 - DroomDroom](https://droomdroom.com/guide-to-perpetual-swap-contracts-trading/)
- [CoinGlass Funding Rate](https://www.coinglass.com/FundingRate)
- [How to Analyze Funding Rates in Crypto 2026 - Zipmex](https://zipmex.com/blog/how-to-analyze-funding-rates-in-crypto/)
- [Best Crypto Exchange For Shorting 2026 - Coin Bureau](https://coinbureau.com/analysis/best-crypto-exchange-for-shorting)
- [How to Short Cryptocurrency - ECOS](https://ecos.am/en/blog/how-to-short-cryptocurrency-strategies-tools-risks-and-expert-tips-2)
- [Short Squeeze in Crypto - BingX](https://bingx.com/en/learn/article/what-is-short-squeeze-in-crypto-how-liquidations-trigger-price-surge)
- [How to Detect Short Squeeze - Bitunix](https://blog.bitunix.com/en/crypto-short-squeeze-detect-defend-profit/)
- [Bitcoin Funding Rate Short Squeeze Warning - BitcoinWorld](https://bitcoinworld.co.in/bitcoin-funding-rate-short-squeeze-warning/)
- [Short Squeeze Dynamics and Negative Funding - Amberdata](https://blog.amberdata.io/short-squeeze-dynamics-negative-funding-and-btc/eth-decoupling-signal-a-fragile-market)
- [Jim Chanos Investment Insights - Analyzing Alpha](https://analyzingalpha.com/jim-chanos)
- [Jim Chanos Short Selling Prophet - Verified Investing](https://verifiedinvesting.com/blogs/education/jim-chanos-the-short-selling-prophet-who-profited-from-corporate-catastrophes)
- [Jim Chanos Position Sizing - MSN/Zer0es TV](https://www.msn.com/en-us/money/smallbusiness/jim-chanos-teaches-you-how-to-size-your-short-positions-short-selling-101-zer0es-tv/vi-AA1FzVHT)
- [Hedging USD Value By Shorting 1x - Deribit Insights](https://insights.deribit.com/education/hedging-usd-value-by-shorting-1x/)
- [Cryptocurrency Put Options - Deribit Insights](https://insights.deribit.com/education/cryptocurrency-put-options/)
- [Bitcoin Options Open Interest $50B on Deribit - CoinDesk](https://www.coindesk.com/markets/2025/10/23/bitcoin-options-open-interest-surges-to-record-usd50b-on-deribit-as-traders-actively-hedge-downside-risks)
- [How to Hedge with Options - OKX](https://www.okx.com/learn/how-to-hedge-crypto-options)
- [Bitcoin 2025 Short Squeeze Market Maker Manipulation - AInvest](https://www.ainvest.com/news/bitcoin-2025-short-squeeze-market-maker-manipulation-bear-trap-dynamics-2509/)
- [Crypto Market Manipulation 2025 - Chainalysis](https://www.chainalysis.com/blog/crypto-market-manipulation-wash-trading-pump-and-dump-2025/)
- [Critical Crypto Short Squeeze Risks 2025 - Stockpil](https://www.stockpil.com/crypto-short-squeeze-2025-analysis/)
- [CoinGlass Liquidation Heatmap Guide](https://www.coinglass.com/learn/how-to-use-liqmap-to-assist-trading-en)
- [How to Read Coinglass Heatmap - Bittime](https://www.bittime.com/en/blog/cara-baca-coinglass-heatmap)
- [ProShares BITI Short Bitcoin ETF](https://www.proshares.com/our-etfs/leveraged-and-inverse/biti)
- [Short Bitcoin ETFs Explained - Capital.com](https://capital.com/en-int/analysis/short-bitcoin-etf-explained)
- [Leverage Crypto Trading - CoinGecko](https://www.coingecko.com/learn/leverage-crypto-trading-how-does-it-work)
- [GameStop Short Squeeze - Wikipedia](https://en.wikipedia.org/wiki/GameStop_short_squeeze)
- [Anatomy of Terra-Luna Failure - ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S1544612322005359)
- [Trailing Stop Loss Tips - Altrady](https://www.altrady.com/crypto-trading/technical-analysis/trailing-stop-loss-tips-crypto-trading)
- [Stop-Loss Secrets - Coinmonks/Medium](https://medium.com/coinmonks/stop-loss-secrets-placing-them-like-a-pro-fce5de4f5c50)
- [Hedging Cryptocurrency Options - NIH/PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC9911343/)
- [Market Maker Hedging Strategies - DWF Labs](https://www.dwf-labs.com/news/understanding-market-maker-hedging)
