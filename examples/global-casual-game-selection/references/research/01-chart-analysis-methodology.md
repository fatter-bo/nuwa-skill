# 维度一：榜单分析方法论

## 核心工具与数据源

### Sensor Tower（含原data.ai）
- 2024年收购data.ai后，合并双方用户面板，2025年推出全新数据管线，估算准确度大幅提升
- **Game IQ分类体系**：覆盖70+子品类（Swap/Blast/Merge/Sort/Block/Word/Hidden Object等），支持按子品类、题材、美术风格、变现机制等维度分析
- 提供历史下载量和收入数据，可按国家/设备拆分
- Puzzle品类内已细分出19个子品类，Swap Puzzle以8B+下载和$25B+收入领先

### 其他主要工具
- **AppMagic**：偏重西方Tier-1市场休闲游戏分析，定期发布品类报告
- **GameRefinery**：偏重游戏设计特征分析（feature-level），提供竞品功能对标
- **Naavik**：深度行业分析，擅长Match-3/Merge等子品类拆解

## App Store榜单排名算法

### 苹果App Store
- **下载速度（Download Velocity）**：核心因素，短时间大量下载可快速拉升排名
- **收入速度**：畅销榜由IAP收入而非下载量决定
- **评分与评论量**：次要因素，影响推荐权重
- **留存率与卸载率**：卸载率高会负面影响排名
- **算法窗口**：使用4-7天移动平均值评估各指标

### Google Play
- 除上述因素外，更重视关键词相关性、崩溃率/ANR率
- 用户参与度指标（会话时长、打开频率）权重更高
- 编辑推荐和Collections对曝光影响显著

## 排名与实际表现的关系模型

### 关键认知
1. **免费榜排名 ≈ 下载速度**，非累计下载量
2. **畅销榜排名 ≈ 收入速度**，非累计收入
3. 同一排名在不同国家代表完全不同的绝对值（美国Top 100 vs 越南Top 100差距可达100倍）
4. 榜单排名存在"惯性效应"——高排名带来自然流量，进一步巩固排名

### 实用分析框架
- **排名稳定度**：连续30天在Top 200内 → 产品已找到PMF
- **排名波动模式**：周末冲高/工作日回落 → 依赖买量；平稳 → 自然流量健康
- **新品上榜速度**：首发3天内进Top 50 → 有强推广/预注册；缓慢爬升 → 口碑驱动

## 参考来源
- Sensor Tower官方产品页与博客
- SplitMetrics: App Store Ranking Factors (2025)
- MobileAction: App Store / Google Play ranking factors (2026)
- AppTweak: App Store ranking factors
- Apptopia: App store rank explained
