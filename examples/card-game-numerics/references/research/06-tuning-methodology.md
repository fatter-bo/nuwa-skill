# 维度六：数值调优方法论

## 调研时间
2026-04-12

## 核心发现

### A/B测试框架

#### 基本流程
1. 定义假设（如"降低初级场入场费10%将提升D7留存"）
2. 确定样本量（基于预期效果量和统计功效）
3. 随机分组（控制组/实验组）
4. 运行测试（通常7-14天）
5. 统计检验（t-test for ARPU, Chi-squared for conversion）
6. 决策与上线

#### 棋牌游戏特有考量
- ARPU分布极度偏斜（95%玩家不付费），需要大样本
- D7 cumulative ARPU 是特别好的综合指标（间接包含留存+变现）
- 样本量计算：方差越大需要越多用户，期望检测的变化越小需要指数级更多用户
- 网络效应干扰：改变匹配规则会影响对照组（需要集群随机化）

来源：[devtodev A/B Testing in LiveOps](https://www.devtodev.com/education/articles/en/351/a-b-testing-in-liveops)、[Superscale A/B Testing](https://superscale.com/a-b-testing-in-mobile-games/)

### 关键指标定义

#### 用户指标
- **DAU/MAU**：日活/月活，MAU比率反映粘性
- **D1/D7/D30留存**：次日/7日/30日留存率
- **场次/人/日**：平均每人每天对局次数
- **平均对局时长**：衡量沉浸度
- **Session Length**：单次会话时长

#### 变现指标
- **ARPU** = 总收入 / 总用户数（含非付费）
- **ARPPU** = 总收入 / 付费用户数
- **付费率** = 付费用户 / 总用户
- **LTV** = ARPU × 用户平均生命周期

#### 数值健康指标
- **胜率分布**：理想状态接近正态分布，中位数50%
- **基尼系数**：衡量货币分布不均匀度（过高=贫富分化严重）
- **破产率**：零货币玩家比例（过高=流失风险）
- **通胀率**：服务器总货币量的月增长率
- **匹配等待时间**：中位数和P95

来源：[GameAnalytics ARPU Guide](https://www.gameanalytics.com/blog/how-to-calculate-arpu-arppu-arpdau-and-more)、[Kevurugames Game Metrics](https://kevurugames.com/blog/what-are-game-metrics-and-why-do-they-matter/)

### 新手保护机制

#### 目标
- 降低新手前N局的负面体验
- 延长新手到达"Aha Moment"(首次有意义胜利)的存活时间

#### 常见策略
- **新手匹配池**：前20-50局仅与新手或Bot对局
- **保底机制**：连续输局后降低匹配对手强度
- **免费资源倾斜**：新手期每日登录奖励更丰厚
- **教学对局**：引导型Bot对局（故意展示核心玩法并让新手赢）
- **输保护**：新手期输局扣减货币减半
- **段位保护**：最低段位不降级

#### 风险
- 过度保护导致新手进入正常匹配后落差过大
- 保护期结束的"断崖"效应需要平滑过渡

来源：[Elevatix A/B Testing Monetization](https://www.elevatix.io/post/the-importance-of-a-b-testing-in-optimizing-mobile-game-monetization-strategies)、[Gamedeveloper Re-think A/B Testing](https://www.gamedeveloper.com/business/it-s-time-to-re-think-a-b-testing)

### 数值调优实践流程

1. **基线建立**：上线初始数值配置，采集2-4周数据
2. **异常诊断**：识别指标异常（如D7留存骤降、破产率过高）
3. **归因分析**：漏斗分析定位流失节点
4. **假设生成**：基于数据提出调整假设
5. **A/B验证**：小流量测试假设
6. **全量上线**：验证通过后推全量
7. **持续监控**：上线后持续跟踪防回退

## 信息源
- devtodev (A/B Testing in LiveOps)
- Superscale (A/B Testing in Mobile Games)
- GameAnalytics (ARPU calculation guide)
- Kevurugames (Game Metrics guide)
- Gamedeveloper.com (A/B Testing, Monetization)
- Elevatix (Monetization A/B Testing)
