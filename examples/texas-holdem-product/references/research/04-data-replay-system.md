# 维度四：数据与回放系统

## 1. Hand History 数据格式

### PokerStars Hand History 格式
PokerStars的手牌记录是行业事实标准，使用纯文本格式：

```
PokerStars Hand #234567890: Hold'em No Limit ($1/$2 USD) - 2025/03/15 14:23:45 ET
Table 'Andromeda IV' 6-max Seat #3 is the button
Seat 1: Player1 ($200 in chips)
Seat 2: Player2 ($185.50 in chips)
Seat 3: Hero ($250 in chips)
Seat 4: Player4 ($312 in chips)
Seat 5: Player5 ($198 in chips)
Seat 6: Player6 ($175.25 in chips)
Player4: posts small blind $1
Player5: posts big blind $2
*** HOLE CARDS ***
Dealt to Hero [Ah Kd]
Player6: folds
Player1: raises $6 to $8
Player2: folds
Hero: raises $18 to $26
Player4: folds
Player5: folds
Player1: calls $18
*** FLOP *** [Ks 7h 2c]
Player1: checks
Hero: bets $30
Player1: calls $30
*** TURN *** [Ks 7h 2c] [9d]
Player1: checks
Hero: bets $65
Player1: folds
Uncalled bet ($65) returned to Hero
Hero collected $115 from pot
*** SUMMARY ***
Total pot $115 | Rake $3
Board [Ks 7h 2c 9d]
```

来源: [Wikipedia - Hand History](https://en.wikipedia.org/wiki/Hand_history)

### PHH（Poker Hand History）标准格式
学术界提出的标准化手牌记录格式：
- 机器友好 + 人类可读
- 支持所有扑克变体（Hold'em/Omaha/Stud/Draw等）
- 开源规范，有Python解析器
- 完整记录：玩家动作、筹码量、位置、公共牌、摊牌

来源: [PHH Standard - GitHub](https://github.com/uoftcprg/phh-std)
来源: [arXiv - Poker Hand History File Format Specification](https://arxiv.org/html/2312.11753v2)

### 格式设计要点
一个好的手牌记录格式需包含：
- 时间戳与唯一手牌ID
- 牌桌信息（名称、人数、盲注级别、赛事类型）
- 所有玩家的座位、筹码量
- 按钮位置（庄家位）
- 每条街（preflop/flop/turn/river）的完整动作序列
- 公共牌
- 摊牌的底牌
- 底池分配与抽水（rake）

## 2. 手牌回放器（Hand Replayer）

### 核心功能
- **逐步回放**：按动作步骤前进/后退
- **快速跳转**：直接跳到Flop/Turn/River
- **播放速度**：可调节自动播放速度
- **连续回放**：批量加载手牌，顺序播放

来源: [Holdem Manager - Replayer & Hand History Viewer](https://kb.holdemmanager.com/knowledge-base/article/replayer-hand-history-viewer)

### 视觉呈现
- 完整牌桌还原：座位、筹码、动作标注
- 底牌展示/隐藏切换
- 底池变化实时更新
- 动作标注：Raise $20 → Call $20 等
- 筹码变化高亮

### 高级功能
- **HUD数据叠加**：在回放中显示对手的统计数据（截至该手时）
- **盈亏图表**：批量手牌的盈亏走势图
- **玩家匿名**：可隐藏玩家名称用于分享/教学
- **导出分享**：生成图片/GIF/短视频分享到社交平台
- **标注系统**：在特定节点添加文字注释（用于复盘学习）

来源: [ICMIZER Replayer Features](https://www.icmizer.com/icmizer/replayer-features/)
来源: [Poker Copilot - Hand Replayer](https://pokercopilot.com/poker-hand-replayer)

### 技术实现
- 支持多平台手牌格式解析（PokerStars/GGPoker/888poker等）
- 前端方案：HTML5 Canvas + JavaScript 或桌面端 Qt/Electron
- 开源参考：[GitHub - Poker Hand Replayer (PyQt5)](https://github.com/caJoey/Poker_Hand_Replayer)

## 3. 个人数据统计系统

### 核心统计指标

#### 行动频率指标
| 指标 | 全称 | 说明 | 健康范围(6max) |
|------|------|------|----------------|
| VPIP | Voluntarily Put $ In Pot | 自愿入池比例 | 22-28% |
| PFR | Pre-Flop Raise | 翻前加注比例 | 18-24% |
| 3Bet | Three-Bet | 翻前再加注比例 | 7-10% |
| ATS | Attempt to Steal | 偷盲尝试比例 | 30-40% |
| F3B | Fold to 3-Bet | 面对3bet弃牌比例 | 55-65% |

来源: [Poker Copilot - VPIP and PFR Statistics](https://pokercopilot.com/poker-statistics/vpip-pfr)

#### 翻后指标
| 指标 | 全称 | 说明 |
|------|------|------|
| AF | Aggression Factor | 攻击因子 = (Bet+Raise) / Call |
| AFq | Aggression Frequency | 攻击频率 = (Bet+Raise) / (Bet+Raise+Call+Check+Fold) |
| CBet | Continuation Bet | 持续下注比例 |
| FCB | Fold to C-Bet | 面对持续下注弃牌比例 |
| WTSD | Went to Showdown | 看到摊牌的比例 |
| W$SD | Won $ at Showdown | 摊牌后赢钱比例 |

来源: [Poker Copilot - Essential Poker Statistics](https://pokercopilot.com/essential-poker-statistics)

#### 盈利指标
| 指标 | 说明 |
|------|------|
| bb/100 | 每100手赢取的大盲数（核心盈利指标） |
| $/hr | 每小时盈利 |
| ROI | 锦标赛投资回报率 |
| ITM% | 锦标赛进钱圈比例 |

### 样本量要求
统计数据的可靠性取决于样本量：
- VPIP/PFR：200手以上开始有参考价值
- 3Bet/F3B：500手以上
- CBet/WTSD：1000手以上
- bb/100盈利率：至少10,000手才有统计意义，50,000手才较可靠

来源: [PlayGreatPoker - Basic HUD Stats](https://www.playgreatpoker.com/ArticleBasicHUDStats.html)

## 4. HUD（Heads-Up Display）

### 传统外挂式HUD
代表产品：PokerTracker 4、Holdem Manager 3、Hand2Note
- 独立运行的桌面软件
- 读取平台生成的手牌记录文件
- 在牌桌上覆盖显示对手统计数据
- 支持高度自定义的统计组合
- 支持弹出窗口查看详细分析

来源: [Hand2Note](https://hand2note.com/)
来源: [Pokerology - Poker HUD Software Explained](https://www.pokerology.com/poker/strategy/hud/)

### 内置式HUD（GGPoker Smart HUD）
GGPoker开创性地内置HUD，设计哲学：
- **公平竞争**：所有玩家看到同样的数据，消除付费工具优势
- **禁止外部追踪器**：只能使用内置Smart HUD
- **仅限当前Session**：不积累历史数据，防止职业玩家的长期数据优势
- **显示内容**：VPIP、PFR、ATS、3Bet、总手数 + 街级别详细数据
- **视觉标识**：赢家头像火焰特效、输家冰冻特效
- **笔记系统**：最多1000字符的彩色笔记

来源: [World Poker Deals - GGPoker Smart HUD](https://worldpokerdeals.com/blog/the-good-game-network-launched-a-built-in-hud)
来源: [RakeRace - How to Use GGPoker Smart HUD](https://rakerace.com/news/poker-rooms/2025/05/14/how-to-use-ggpoker-s-smart-hud-effectively-for-player-profiling)

### PokerCraft（GGPoker内置分析工具）
- 完整的个人游戏历史数据库
- Session图表和手牌图表
- 按游戏类型/级别/时间段筛选
- 盈亏追踪与趋势分析
- 无需第三方工具即可复盘

来源: [GGPoker - PokerCraft Statistics](https://ggpoker.com/blog/pokercraft-analyze-your-ggpoker-statistics/)

### 产品设计决策：外挂HUD vs 内置HUD
| 维度 | 外挂HUD | 内置HUD |
|------|---------|---------|
| 公平性 | 付费用户优势 | 所有人平等 |
| 数据深度 | 极深（数万手历史） | 较浅（仅当前Session） |
| 对休闲玩家 | 不友好（被追踪） | 友好（保护隐私） |
| 对职业玩家 | 核心工具 | 限制明显 |
| 平台控制力 | 低（难以完全禁止） | 高 |
| 用户体验 | 需单独安装配置 | 零配置开箱即用 |

**趋势判断**：行业正在向内置HUD + 限制外挂方向发展，核心驱动力是保护休闲玩家的游戏体验。
