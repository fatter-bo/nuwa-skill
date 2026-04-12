---
name: betting-terminology-mapping
description: |
  体育博彩术语、盘口类型与数据需求映射。基于Bet365/Pinnacle/William Hill/Betfair结算规则、
  Sportradar数据标准、Azuro盘口文档、Asian Handicap完整规则等30+权威来源。
  为FastEvent创始人量身定制——听懂客户每一个术语，立刻知道需要什么数据。
  当用户提到任何博彩术语、盘口类型、结算规则时使用。
  当用户说「这个客户需要支持XXX盘口」时，能立刻告诉FastEvent能否满足。
---

# 体育博彩术语 · 数据需求映射系统

> 每一种盘口类型本质上就是对数据的一种"查询方式"。
> 客户说"我需要支持Asian Handicap"，翻译成技术语言就是"我需要你的API返回精确比分"。

## Skill规则

**此Skill激活后，作为FastEvent的博彩术语翻译器运行。**

- 客户说任何博彩术语 → 秒懂并翻译成数据需求
- 客户说需要某个盘口 → 立刻判断FastEvent能否满足
- 不教赌博策略，只做"术语→数据"的映射

---

## 一、基础术语速查（中英对照）

### 赔率相关

| 英文 | 中文 | 一句话 | 例子 |
|------|------|--------|------|
| Decimal Odds | 欧赔 | 投注$1总返还多少 | 2.50 = 投$1返$2.50（含本金） |
| Fractional Odds | 英赔 | 利润与本金的比例 | 3/2 = 投$2赚$3 |
| Moneyline / American Odds | 美赔 | 正数=赚多少/负数=需投多少 | +150 = 投$100赚$150; -200 = 投$200赚$100 |
| Implied Probability | 隐含概率 | 赔率反映的概率 | 欧赔2.50 → 1/2.50 = 40% |
| Overround / Vig / Juice | 庄家利润率 | 所有结果隐含概率之和超过100%的部分 | 三个结果概率加起来104.8% → 庄家抽4.8% |
| True Odds | 真实赔率 | 去除庄家利润后的赔率 | 永远比显示赔率更高 |

**赔率转换公式**：
```
欧赔 → 美赔: 如果≥2.0: (欧赔-1)×100 = +XX; 如果<2.0: -100/(欧赔-1) = -XX
欧赔 → 隐含概率: 1/欧赔
美赔 → 欧赔: 正数: (美赔/100)+1; 负数: (100/|美赔|)+1
```

### 投注相关

| 英文 | 中文 | 一句话 |
|------|------|--------|
| Stake | 投注金额 | 下注的钱 |
| Payout | 派奖金额 | 赢了拿到的总额（含本金） |
| Bankroll | 资金池 | 投注总资本 |
| Unit | 投注单位 | 标准化下注额，通常=资金池1-2% |
| ROI / Yield | 投资回报率 | 总利润÷总投注×100% |

### 市场相关

| 英文 | 中文 | 一句话 | FastEvent关联 |
|------|------|--------|-------------|
| Pre-match | 赛前盘 | 比赛开始前的投注 | REST API即可 |
| In-play / Live | 走地盘/实时盘 | 比赛进行中的投注 | 需要WebSocket <500ms |
| Ante-post / Futures | 期货盘 | 赛季冠军等长期投注 | 赛季结束后结算 |
| Outright | 直接投注 | 投注某个最终结果 | 赛事完成后结算 |

### 状态相关

| 英文 | 中文 | 一句话 | FastEvent需要报告 |
|------|------|--------|-----------------|
| Void / Push | 作废/退注 | 投注取消，退还本金 | 状态: `voided` |
| Cash Out | 提前结算 | 比赛结束前锁定利润/止损 | 需要实时比分 |
| Dead Heat | 并列 | 多个结果并列 | 报告并列状态 |
| Walkover | 弃权胜 | 对手退出，直接获胜 | 状态: `walkover` |
| Postponed | 延期 | 比赛推迟 | 状态: `postponed` |
| Abandoned | 中止 | 比赛开始后被取消 | 状态: `abandoned` + 已进行时间 |

### 平台相关

| 英文 | 中文 | 一句话 |
|------|------|--------|
| Bookmaker / Sportsbook | 博彩公司 | 设定赔率、接受投注、承担风险 |
| Exchange | 博彩交易所 | 玩家对玩家，平台收佣金（Betfair） |
| Sharp | 专业赌客 | 使用模型/数据的职业投注者 |
| Square | 业余赌客 | 凭感觉/情感投注的休闲玩家 |
| Closing Line | 收盘线 | 赛前最后赔率——被认为最准确 |

---

## 二、足球盘口类型（Phase 1核心）

**总规则：除非明确说明，所有足球盘口只按正规时间（90分钟+伤停补时）结算。加时赛和点球不算。**

### 终极映射表

| 盘口 | 怎么玩 | 结算数据 | 时间边界 | FastEvent接口 | 实时性 |
|------|--------|---------|---------|-------------|--------|
| **1X2** | 猜主胜/平/客胜 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **Asian Handicap** | 让球消除平局 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **Over/Under** | 总进球大/小 | totalGoals | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **BTTS** | 双方都进球？ | homeScore≥1, awayScore≥1 | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **Correct Score** | 猜精确比分 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **HT/FT** | 半场+全场结果 | htScore + ftScore | 半场+全场 | `getHalfTimeScore()` + `getRegulationScore()` | **中场时推送** |
| **Goalscorer** | 谁进球 | goalscorers[] | 正规时间 | `getGoalEvents()` | 事件驱动 |
| **Double Chance** | 三选二 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **Draw No Bet** | 平局退款 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **European Handicap** | 三路让球 | homeScore, awayScore | 正规时间 | `getRegulationScore()` | 赛后即可 |
| **Corners** | 角球数大小 | homeCrns, awayCrns | 正规时间 | `getMatchStats()` | 赛后即可 |
| **Cards** | 红黄牌数 | cards[] | 正规时间 | `getMatchStats()` | 赛后即可 |
| **To Qualify** | 谁晋级 | fullTimeScore+penalties | **含加时+点球** | `getFullResult()` | 赛后即可 |
| **Accumulator** | 多场串关 | 每注每条腿数据 | 各注各规则 | 全部接口 | **最高** |
| **Live O/U** | 走地大小盘 | currentScore | 实时 | WebSocket推送 | **秒级** |

### Asian Handicap详解（走盘/走水）

**走盘（Push）**：整数让球线，让球后比分恰好持平 → 退还本金。
- 例：主队-1，主队赢2-1 → 让球后1-1 → Push，退注

**四分之一让球（Quarter Lines）**：投注额拆成两半：
| 线 | 拆分 | 例子（$100投注，主队赢2-1） |
|----|------|--------------------------|
| -0.25 | $50在0 + $50在-0.5 | 0线赢 + -0.5线赢 = 全赢 |
| -0.75 | $50在-0.5 + $50在-1.0 | -0.5线赢 + -1.0线Push = 半赢半退 |
| -1.25 | $50在-1.0 + $50在-1.5 | -1.0线Push + -1.5线输 = 半退半输 |
| -1.75 | $50在-1.5 + $50在-2.0 | -1.5线输 + -2.0线输 = 全输 |

**FastEvent需要**：只需精确比分（homeScore, awayScore），让球计算由下游协议处理。

---

## 三、篮球盘口（Phase 2）

**总规则：除非标注"Regulation Only"，篮球全场盘口包含加时赛。**

| 盘口 | 加时算不算 | 数据需求 | FastEvent接口 |
|------|-----------|---------|-------------|
| Moneyline | ✅ 算 | finalScore（含OT） | `getFinalScore()` |
| Point Spread | ✅ 算 | finalScore（含OT） | `getFinalScore()` |
| Over/Under | ✅ 算 | totalPoints（含OT） | `getFinalScore()` |
| Q1/Q2/Q3 | ❌ 不算 | 各节比分 | `getQuarterScores()` |
| Q4 | ❌ 不算 | 第四节比分 | `getQuarterScores()` |
| 2nd Half | ✅ 算OT | Q3+Q4+OT | `getQuarterScores()` |
| Player Props | ✅ 通常算 | 球员统计 | `getPlayerStats()` |

**最低比赛时长**：NBA 43分钟 / NCAA 35分钟 / WNBA 35分钟。不满则全部作废。

---

## 四、网球盘口（Phase 2）

| 盘口 | 退赛处理 | 数据需求 |
|------|---------|---------|
| Match Winner | 分两派：完成1盘即有效 vs 必须打完 | matchWinner, matchCompleted |
| Set Betting | 退赛=全部作废 | setScores[] |
| Game Handicap | 退赛=通常作废 | totalGames per player |
| Over/Under Games | 退赛=通常作废 | totalGames |

**退赛规则（最复杂的结算问题）**：
- **Bet365类**：打完1盘 → Match Winner有效；其他盘口作废
- **William Hill类**：没打完 → 全部作废
- **弃权（Walkover）**：比赛未开始就退出 → **所有博彩公司都作废**

**FastEvent需要报告**：`matchStatus`（completed/retired/walkover/disqualified）+ `setsCompleted`

---

## 五、电竞盘口（Phase 2可选）

| 盘口 | 数据需求 |
|------|---------|
| Series Winner | seriesWinner, mapsWon per team |
| Map Winner | mapResults[] {mapNumber, winner} |
| Map Handicap | mapsWon per team |
| Kill O/U | kills per team per map |
| First Blood | firstBlood team per map |
| First Tower | firstTower team per map (LoL/Dota2) |

**数据源挑战**：无统一权威、依赖游戏开发者API（Riot/Valve）、独家数据协议、低级别赛事缺乏监控。

---

## 六、结算边界速查卡

### 足球
```
正规时间（90min+补时）= 1X2, 让球, 大小, BTTS, 波胆, DC, DNB, 角球, 黄牌
├── 补时进球算正规时间 ✅
├── 加时赛进球不算 ❌
└── 点球大战进球不算 ❌

含加时赛 = To Qualify（晋级盘）, To Lift Trophy
├── 加时赛进球算 ✅
└── 点球大战决定晋级但不算在比分内

半场结算 = HT/FT盘口、半场比分、半场让球
└── 需要中场休息时的比分快照

Pinnacle特殊规则：比赛必须打满55分钟才有效
Bet365特殊规则：中止比赛，已决定的市场有效（如BTTS Yes双方已进球）
```

### 篮球
```
全场（含OT）= Moneyline, Spread, O/U, 2nd Half, Player Props
只算该节 = Q1, Q2, Q3, Q4（OT不算Q4）
最低时长：NBA 43min / NCAA 35min，不满全作废
```

### 网球
```
全场 = Match Winner（但退赛处理各家不同）
退赛 = Set Betting/Handicap/O/U 通常全部作废
弃权（Walkover）= 所有盘口全部作废
```

### 电竞
```
系列赛（BO3/BO5）= Series Winner, Map Handicap
单地图 = Map Winner, Kill O/U, First Blood
技术重赛（Remake）= 该地图盘口通常作废
```

---

## 七、客户对话翻译器

### 对话1：基础体育盘口
**客户**："We need to support Asian Handicap and Over/Under for all Premier League matches, settled on regulation time only."
**翻译**：他需要正规时间的精确比分（homeScore, awayScore），覆盖英超所有比赛。FastEvent的`getRegulationScore()`完全满足。让球计算和大小盘判断由客户自己做。

### 对话2：半场比分
**客户**："Can you provide half-time scores for HT/FT markets?"
**翻译**：他需要中场休息时的比分快照。FastEvent需要在HalfTime状态时推送一次数据。需要`getHalfTimeScore()`接口。

### 对话3：Live Betting
**客户**："We're building live betting. How fast can you push score updates?"
**翻译**：他需要WebSocket实时推送，延迟要求<1秒。FastEvent WS Gateway当前<500ms，满足需求。需要在进球、红牌、半场等事件发生时立即推送。

### 对话4：晋级盘
**客户**："For Champions League knockout rounds, we need to support 'To Qualify' markets."
**翻译**：他需要完整比赛结果——含加时赛和点球大战。FastEvent需要`getFullResult()`返回`{regulationScore, extraTimeScore, penaltyResult}`三个维度。

### 对话5：角球/黄牌
**客户**："We want to add Corners and Cards markets. What additional data do you provide?"
**翻译**：他需要统计数据而非仅比分。FastEvent的`getMatchStats()`需要返回`{corners: {home, away}, cards: [{player, type, minute}]}`。这是Phase 1+的功能。

### 对话6：串关
**客户**："Parlays are our highest-margin product. We need all legs to settle within minutes of match end."
**翻译**：串关对结算速度要求最高——一条腿延迟=整个串关延迟。FastEvent需要所有覆盖比赛赛后秒级推送结果。这是FastEvent vs UMA（2h+）最大的差异化场景。

### 对话7：篮球加时
**客户**："Does your basketball feed include overtime? Our Point Spread markets need OT."
**翻译**：篮球全场盘口（Moneyline/Spread/O/U）需要含OT的最终比分。FastEvent的`getFinalScore()`必须包含所有加时赛分数。但各节盘口不含OT——需要分开报告。

### 对话8：网球退赛
**客户**："How do you handle tennis retirements? Different bookmakers have different rules."
**翻译**：FastEvent不需要替客户做结算决策——只需报告事实：`matchStatus: retired`、`setsCompleted: 1`、`retirementPoint: {set: 2, game: 3}`。客户根据自己的规则决定作废还是有效。

### 对话9：比赛中止
**客户**："What happens if a match is abandoned at 70 minutes?"
**翻译**：FastEvent报告`status: abandoned`、`minutesPlayed: 70`、当前比分。客户根据自己的规则决定——Pinnacle要求55分钟才有效，Bet365要求已决定市场有效。FastEvent提供事实，不做判断。

### 对话10：双维度比分
**客户**："We need both regulation time and full time scores separately."
**翻译**：这正是FastEvent的核心差异化。UMA只返回一个"结果"。FastEvent返回结构化对象：`{regulationTime: {home: 1, away: 1}, fullTime: {home: 2, away: 1}, penalties: {home: 4, away: 3}}`。

---

## 八、FastEvent能力评估速查

**当客户说"我需要支持XXX盘口"时：**

| 盘口 | FastEvent当前能力 | 缺什么 |
|------|-----------------|--------|
| 1X2 / AH / O/U / BTTS / CS / DC / DNB | ✅ 完全满足 | — |
| HT/FT | ✅ 需确保中场推送 | 半场比分推送时机 |
| To Qualify | ✅ 需双维度 | 确保penaltyResult字段 |
| Goalscorer | ⚠️ 需要进球事件数据 | goalscorers[]字段（Phase 1+） |
| Corners / Cards | ⚠️ 需要统计数据 | matchStats字段（Phase 1+） |
| Live Betting | ✅ WebSocket <500ms | 进球后暂停推送触发 |
| Basketball全场 | ✅ 含OT | 确保OT分节报告 |
| Basketball分节 | ✅ 需分节 | quarterScores[] |
| Tennis Match Winner | ✅ 基础 | retirementStatus |
| Tennis Set Betting | ⚠️ 需分盘 | setScores[] |
| Esports | ❌ Phase 2+ | 完全需要新数据源 |

---

## 诚实边界

1. **结算规则因博彩公司而异**：Bet365、Pinnacle、William Hill的规则有差异——FastEvent提供事实数据，结算判断由客户做
2. **Goalscorer和统计数据是Phase 1+**：当前最小可行产品可能只有比分，进球事件和统计数据需要额外数据源
3. **电竞数据复杂度高**：每个游戏API不同，Schema不统一——Phase 2需要显著投入
4. **Live Betting的"暂停投注"触发**：FastEvent报告比分变化，但"何时暂停投注"是客户的业务逻辑
5. **术语可能有地区差异**：亚洲、欧洲、北美的博彩术语习惯不完全一致

- 调研时间：2026-04-12

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
