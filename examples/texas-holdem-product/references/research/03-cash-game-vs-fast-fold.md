# 维度三：常规桌 vs 快速桌

## 1. 常规桌（Regular Cash Game）

### 核心特征
- 玩家选择特定牌桌入座，持续对抗相同对手
- 可随时买入/离座，筹码即真实货币价值
- 标准速度：6人桌约75-100手/小时，9人桌约60-80手/小时
- 支持选桌（Table Selection）：玩家可挑选对手阵容有利的桌

### 产品优势
- **读牌深度**：长时间对抗同一对手，可积累对手信息
- **社交体验**：固定对手带来社交互动和竞争关系
- **策略深度**：位置、对手调整、元博弈（metagame）充分展现
- **数据积累**：HUD工具在常规桌效果最佳

### 用户画像
- 偏好策略深度的中高级玩家
- 多桌职业/半职业玩家
- 注重读牌和心理博弈的玩家

## 2. 快速桌（Fast-Fold / Zoom Poker）

### 核心机制
Zoom Poker由PokerStars于2012年推出，核心创新：
- 玩家加入的是**玩家池（Player Pool）**，而非特定牌桌
- 每手牌结束后（或弃牌后），立即被分配到新牌桌与新对手
- 可在轮到自己行动前点击**快速弃牌（Fast Fold）**按钮，立即进入下一手
- 软件自动平衡盲注频率，确保每位玩家公平地缴纳盲注

来源: [PokerStars - Introduction to Zoom Poker](https://www.pokerstars.com/poker/learn/strategies/introduction-to-zoom-poker/)

### 各平台命名
| 平台 | 产品名 |
|------|--------|
| PokerStars | Zoom Poker |
| GGPoker | Rush & Cash |
| partypoker | fastforward |
| 888poker | SNAP Poker |
| Full Tilt (已关闭) | Rush Poker（最早发明者） |
| Winning Poker Network | Blitz Poker |

### 技术实现要点

#### 玩家池管理
- 维护活跃玩家池（Player Pool），所有同级别同人数配置的玩家在同一池中
- 每手牌从池中随机选取N位玩家组成临时牌桌
- 盲注位置追踪：系统记录每位玩家的盲注状态，确保公平分配

来源: [VIP Grinders - Fast Fold Poker Strategy Guide](https://www.vip-grinders.com/fast-fold-poker-strategy-guide/)

#### 快速弃牌技术
- 玩家点击Fast Fold后，客户端立即将其从当前牌桌移除
- 服务端在该玩家轮次时自动执行弃牌
- 对其他玩家而言，弃牌动作在正常流程中执行，无感知差异
- 已快速弃牌的玩家同时被分配到新牌桌

#### 速度数据
- Zoom 6人桌：约200-250手/小时（常规桌的2.5-3倍）
- Zoom 9人桌：约150-200手/小时
- 4桌Zoom全桌：可达1500手/小时

来源: [BlackRain79 - Zoom Poker Strategy Essential Guide](https://www.blackrain79.com/2015/07/zoom-poker-strategy-essential-guide.html)

### 用户画像差异

| 维度 | 常规桌玩家 | 快速桌玩家 |
|------|-----------|-----------|
| 技术水平 | 全范围 | 偏中级以上 |
| 游戏风格 | 多样化 | 整体更紧（VPIP更低） |
| Session时长 | 1-4小时 | 30分钟-2小时 |
| 核心诉求 | 策略深度、对手博弈 | 效率、手数量、快速反馈 |
| 多桌行为 | 常见（4-12桌） | 较少（1-4桌，已够快） |
| HUD依赖 | 高（读牌核心工具） | 中（样本量难积累） |
| 社交需求 | 中等 | 低 |

来源: [Upswing Poker - Regular vs Fast-Fold Strategy](https://upswingpoker.com/fast-fold-zoom-zone-rush-poker-vs-regular-strategy/)

### 快速桌对游戏生态的影响

#### 正面影响
- **降低等待时间**：新手不需忍受漫长的弃牌等待
- **提升手数体验**：时间有限的玩家可在短session内获得足够手数
- **简化选桌**：不需要table selection技能，降低入门门槛
- **减少挂机/占座**：弃牌即离开，无占位问题

#### 负面影响
- **游戏变紧**：平均VPIP从常规桌的25%降到20%以下
- **读牌难度增大**：对手轮换太快，HUD需更大样本才有效
- **减少创意打法**：紧缩环境抑制了诈唬和非常规策略
- **常规桌流失**：部分玩家迁移到快速桌，导致常规桌流动性下降

来源: [PokerListings - Crash Course in Zoom Poker](https://www.pokerlistings.com/poker-strategies/cash-game-nl-holdem/crash-course-in-zoom-poker-on-pokerstars)

## 3. 创新变体

### GGPoker Rush & Cash
- 快速桌基础上增加**全下保险（All-in Insurance）**
- 当你领先全下时，可购买保险锁定部分EV
- 产品创新：将保险精算融入快速桌体验

### PokerStars Zoom 特殊功能
- **Ctrl+Fold**：按住Ctrl点弃牌可留下观看本手结果
- **Zoom Fast Deal**：每手间过渡动画最小化
- **Zoom 锦标赛**：将快速桌机制引入SNG格式

## 4. 产品设计关键决策点

### 要不要做快速桌？
- **做的条件**：有足够活跃用户（至少同时200-500人在线同级别）组成健康玩家池
- **不做的理由**：用户基数小会导致池内对手重复率过高，丧失快速桌核心价值
- **折中方案**：先做常规桌，达到一定DAU后再开放快速桌

### 快速桌的最小可玩人数
- 6人桌Zoom需至少50-100人同时在线
- 低于此阈值，玩家会频繁遇到相同对手，体验退化
