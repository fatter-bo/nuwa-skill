# 维度四：匹配与房间系统

## 调研时间
2026-04-12

## 核心发现

### ELO评分系统

#### 基本原理
- 由Arpad Elo于1960年代为国际象棋设计
- 新玩家初始分：通常1500
- 预期胜率公式：E_A = 1 / (1 + 10^((R_B - R_A)/400))
- 分数更新：R'_A = R_A + K × (S_A - E_A)，K为调节系数
- K值选择：新玩家K=32（快速定位），稳定玩家K=16（缓慢调整）

#### ELO的局限
- 不考虑评分不确定性（长期不活跃玩家的分数可靠性下降）
- 对多人游戏（非1v1）适配不佳
- 可被"降分刷新手"策略利用

来源：[arXiv Is Elo Rating Reliable?](https://arxiv.org/pdf/2502.10985)

### Glicko/Glicko-2系统

#### Glicko改进
- 增加Rating Deviation (RD)参数：衡量评分的不确定性
- RD大=不确定性高（新玩家或久未活跃），匹配时权重调整更大
- 置信区间：约95%概率真实实力在 [Rating - 2×RD, Rating + 2×RD] 内

#### Glicko-2改进
- 新增Volatility(σ)参数：衡量玩家表现的波动性
- 已知问题："波动性农场"(Volatility Farming)——故意输棋提高σ再连胜快速升分
- 棋牌游戏比象棋更需要Glicko-2（玩家活跃度差异大，技能波动明显）

来源：[ELO vs Glicko Poker Ranking](https://pokergamedevelopers.com/elo-vs-glicko-poker-ranking-system/)、[Glicko-2 Enhancement Paper](https://www.cimachinelearning.com/assets/article/vol6-iss1/glicko-2-algorithm.pdf)

### Elo-MMR系统
- 专为大规模多人竞赛设计（如Codeforces竞赛）
- 处理N人同时比赛的排名（而非仅1v1）
- 适合锦标赛场景：一局多人参与，最终排名作为结果

来源：[Stanford Elo-MMR Paper](https://cs.stanford.edu/people/paulliu/files/www-2021-elor.pdf)、[arXiv Elo-MMR](https://arxiv.org/pdf/2101.00400)

### 等级场分层设计

#### 分层原则
- 按货币持有量分层（初级场/中级场/高级场/VIP场）
- 每层设置入场下限和携带上限
- 示例：初级场(1K-10K)、中级场(10K-100K)、高级场(100K-1M)
- 上限防止大鱼屠幼鱼，下限保证对局质量

#### 匹配队列设计
- 匹配等待时间目标：<30秒（休闲棋牌）
- 匹配池扩大策略：等待超时后逐步放宽Rating范围（±50 → ±100 → ±200）
- 高峰/低谷时段动态调整匹配严格度

### Bot填充策略

#### 使用场景
- 低在线时段匹配池不足
- 新手保护期（确保新玩家有对手）
- 特定房间玩家不足时维持开局

#### Bot难度设计
- 多级难度：初级Bot（随机+基本规则）、中级Bot（简单策略）、高级Bot（接近真人水平）
- Bot的Rating需纳入匹配系统，按对应段位匹配
- 透明度考量：是否告知玩家对手为Bot（行业实践不一）

#### Bot行为拟人化
- 模拟人类思考时间（非即时响应）
- 模拟人类犯错率（非最优策略）
- 模拟人类操作节奏变化

来源：[Gamedeveloper AI Scaling](https://gamedeveloper.com/blogs/building-ai-for-games-that-scales)、[SBMM Research Paper](http://www.cemyuksel.com/research/matchmaking/i3d2024-matchmaking.pdf)

## 信息源
- arXiv (Elo reliability, Elo-MMR)
- Stanford CS (Elo-MMR paper)
- pokergamedevelopers.com (ELO vs Glicko)
- CI Machine Learning (Glicko-2 Enhancement)
- Game Developer (AI Scaling)
- ACM/IEEE (Skill-Based Matchmaking paper)
