# 维度三：欢乐豆经济系统

## 调研时间
2026-04-12

## 核心发现

### 水龙头-排水口(Faucet-Drain)模型

经济系统的核心平衡公式：**产出速率 ≈ 消耗速率**

#### 水龙头(货币产出渠道)
- 每日登录奖励（基础产出）
- 完成对局奖励（核心产出）
- 成就/任务奖励（里程碑产出）
- 好友赠送（社交产出）
- 广告观看奖励（变现联动产出）
- 购买充值（付费产出）
- 新手礼包（初始产出）

#### 排水口(货币消耗池)
- 对局入场费/台费抽水（核心消耗，通常3-5%）
- 高等级房间门槛（分层消耗）
- 道具/表情购买（社交消耗）
- 锦标赛报名费（竞技消耗）
- VIP特权订阅（付费消耗）
- 物品过期/衰减（时间消耗）

来源：[1kxnetwork Sinks & Faucets](https://medium.com/1kxnetwork/sinks-faucets-lessons-on-designing-effective-virtual-game-economies-c8daf6b88d05)、[Lost Garden Value Chains](https://lostgarden.com/2021/12/12/value-chains/)

### 通胀控制模型

#### 通胀产生原因
- 高技能玩家持续从低技能玩家处净转移货币
- 系统产出（登录/任务等）持续注入无对应消耗
- 作弊/刷分行为加速货币产出

#### 控制机制
- **动态抽水率**：根据服务器总货币量调整台费比例
- **指数级消耗设计**：道具/特权价格指数增长（F2P经典模式）
- **货币上限**：单账户持有上限，溢出部分转化为不可交易荣誉点
- **赛季重置**：定期部分重置货币（保留基础量+比例衰减）
- **交易税**：玩家间转移收取税率（如Axie的4.25%市场税）
- **实时监控**：跟踪M2货币总量、流通速度、基尼系数

来源：[Machinations Game Economy Inflation](https://machinations.io/articles/what-is-game-economy-inflation-how-to-foresee-it-and-how-to-overcome-it-in-your-game-design)、[Gamedeveloper Inflation Domination](https://www.gamedeveloper.com/business/managing-virtual-economies-inflation-domination)

### 防刷机制

- **频率限制**：单位时间内对局次数上限
- **异常检测**：连续快速输赢模式检测（协议刷分）
- **设备指纹**：同设备/IP多账户关联检测
- **行为分析**：对局时长过短、固定对手反复对局等模式
- **验证码**：可疑行为触发人机验证
- **金额追踪**：大额异常变动自动标记人工审核

来源：[ResearchGate Virtual Currency Economy Design](https://www.researchgate.net/publication/389818173)、[Dev.to Virtual Economy Strategies](https://dev.to/okoye_ndidiamaka_5e3b7d30/building-a-thriving-virtual-economy-in-games-strategies-towards-balance-and-engagement-bn8)

### 棋牌特有经济设计要点

- 零和博弈特性：玩家间货币转移为主，系统产出为辅
- 抽水是核心Sink：每局从赢家收取的台费是主要货币消耗
- 破产保护：零货币玩家需要"救济金"机制维持活跃
- 分层房间：不同货币量级的房间隔离（初级场/中级场/高级场）
- 付费转化节点：破产时的充值引导是核心变现时机

## 信息源
- 1kxnetwork Medium (Sinks & Faucets)
- Lost Garden (Value Chains)
- Machinations.io (Game Economy articles)
- Gamedeveloper.com (Managing Virtual Economies)
- ResearchGate (Virtual Currency Economy Design, 2024)
