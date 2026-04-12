# 维度二：竞赛系统

## 1. SNG（Sit & Go）产品设计

### 核心机制
- 固定人数（通常6人或9人单桌），坐满即开
- 无预定开始时间，随到随打
- 买入固定，起始筹码相同
- 盲注按时间递增（标准10-15分钟/级，Turbo 5分钟/级，Hyper-Turbo 2-3分钟/级）

来源: [CoinPoker - Online Sit and Go Poker](https://coinpoker.com/online-poker/sit-and-go/)

### 赔付结构
标准9人SNG赔付：
- 第1名：50%
- 第2名：30%
- 第3名：20%

6人SNG赔付：
- 第1名：65%
- 第2名：35%

来源: [Beasts of Poker - Poker Tournament Payout Structure](https://beastsofpoker.com/poker-tournament-payout-structure/)

### 产品变体
- **Turbo SNG**：盲注加速，缩短游戏时间至20-30分钟
- **Hyper-Turbo SNG**：极速盲注，5-15分钟一局
- **Heads-Up SNG**：1v1对决
- **Multi-Table SNG (MT-SNG)**：多桌SNG，18人/27人/45人/90人/180人

来源: [PokerStrategy - MT-SnGs Overview](https://www.pokerstrategy.com/strategy/mtt/1820/)

## 2. MTT（Multi-Table Tournament）产品设计

### 核心机制
- 预定开始时间，注册窗口（通常开赛后仍可迟到注册/重买一段时间）
- 几十到上万人参与
- 盲注按级别递增
- 桌子合并（Table Balancing）算法保证各桌人数均衡

来源: [Poker Game Developers - Building MTT System](https://pokergamedevelopers.com/build-multi-table-tournament-system-poker-platform/)

### 盲注结构设计
WSOP标准盲注结构特点：
- 起始盲注与起始筹码比 = 1:150-300（深筹码）
- 每级15-20分钟（现场），8-15分钟（线上）
- 盲注增幅约25-33%/级（温和递增）
- 引入Ante的时机：通常在第4-6级

来源: [WSOP Structure Sheets 2025](https://www.pokernews.com/tours/wsop/2025-wsop/wsop-structure-sheets.htm)
来源: [PokerSoup Blind Structure Calculator](https://pokersoup.com/tool/blindStructureCalculator)

### 赔付结构
常见MTT赔付比例：
- 奖金覆盖前10-15%的玩家
- 第1名获得总奖池的15-25%（取决于参赛人数）
- Top-heavy赔付（前3名拿走大部分）vs Flat赔付（更均匀分配）
- Final Table Deal（最终桌协商分奖）是常见功能需求

来源: [Beasts of Poker - Poker Tournament Payout Structure](https://beastsofpoker.com/poker-tournament-payout-structure/)

### MTT产品功能
- **迟到注册 (Late Registration)**：开赛后一段时间内可注册
- **重买 (Rebuy)**：筹码归零或低于初始值时可重新买入
- **增购 (Add-on)**：重买期结束时的最后一次加筹码机会
- **暂停与休息 (Breaks)**：每小时5-10分钟休息
- **最终桌 (Final Table)**：最后一桌的特殊展示模式
- **泡沫保护 (Bubble)**：即将进钱圈时的特殊提示与倒计时
- **ICM计算器**：帮助玩家理解锦标赛筹码价值

## 3. Spin & Go 产品设计

### 核心创新
Spin & Go是PokerStars 2014年推出的革命性产品，核心是**随机奖池倍增器**：
- 3人制超级Turbo SNG
- 开局前随机决定奖池倍数（2x到240,000x买入）
- 起始筹码500，盲注10/20，每2-3分钟升级

来源: [PokerListings - Spin & Go Complete Guide](https://www.pokerlistings.com/poker-guides/spin-go-a-complete-guide-to-pokerstars-jackpot-sit-go-tournaments)
来源: [World Poker Deals - Spin and Go Multipliers](https://worldpokerdeals.com/blog/pokerstars-new-spin-go-multipliers-and-1000-spins)

### 倍增器概率分布（PokerStars参考）
| 倍增器 | 概率 | 赔付方式 |
|--------|------|---------|
| 2x | ~72% | 赢者通吃 |
| 3x | ~15% | 赢者通吃 |
| 5x | ~8% | 赢者通吃 |
| 10x | ~3.5% | 80/20 |
| 25x | ~1% | 70/20/10 |
| 120x | ~0.02% | 三人分奖 |
| 240x | 极低 | 三人分奖 |
| 更高 | 百万分之一级 | 三人分奖 |

来源: [PokerStars NJ - Spin & Go Prize Multipliers](https://www.pokerstarsnj.com/help/articles/trn-sag-prizes/40226/)

### 盲注级别与倍增器关联
不同倍增器对应不同的盲注递增速度：
- 2x倍增器：盲注每2分钟升级（极速结束）
- 10x-25x：盲注每3分钟升级
- 100x+：盲注每5分钟升级（给予更多博弈空间）

### 各平台变体对比
| 平台 | 产品名 | 特色 |
|------|--------|------|
| PokerStars | Spin & Go | Flash Hold'em（更快）、Omaha Spins |
| GGPoker | Spin & Gold | 2x保险选项、ELO评级、6人制可选 |
| partypoker | Spins Overdrive | 小数倍增器（如2.65x）、动态结构 |
| Unibet | HexaPro | 低方差赔付、含1x倍增器（退还买入）|
| 888poker | Blast | 根据倍增器调整起始筹码(300-1000) |
| Winamax | Expresso | Nitro变体、8/10买入含百万奖池可能 |

来源: [Pokerfuse - Jackpot SNGs Ultimate Guide](https://pokerfuse.com/online-poker/guides/jackpot-sngs/)

### 产品设计洞察
Spin & Go的成功核心是**彩票心理**：
- 低买入 + 高倍奖池的心理诱惑
- 极短游戏时长（3-7分钟）= 高频决策 = 高黏性
- 开局动画（转盘/揭奖）制造期待感
- 本质上是将SNG与刮刮乐/老虎机的即时反馈结合

## 4. 竞赛系统通用产品需求

### 赛事大厅设计
- **筛选维度**：游戏类型 / 买入范围 / 速度 / 开赛时间 / 奖池保底
- **排序方式**：开赛时间 / 买入 / 参赛人数 / 奖池
- **收藏/提醒**：标记感兴趣的赛事，开赛前推送提醒
- **赛事日历**：周/月视图展示赛事安排

### 赛中信息展示
- 当前盲注级别 + 下一级预告 + 升级倒计时
- 参赛人数 / 剩余人数 / 平均筹码 / 你的排名
- 奖金结构（点击可查看完整赔付表）
- 休息倒计时
