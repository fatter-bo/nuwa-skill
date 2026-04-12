# 维度二：各玩法数值体系

## 调研时间
2026-04-12

## 核心发现

### 德州扑克数值体系

#### 盲注结构
- SNG标准：起始筹码T1500，盲注从10/20开始，每轮递增（10min/轮常见）
- MTT标准：起始筹码更深（100-300BB），盲注递增更缓慢（15-20min/轮）
- 关键参数：起始筹码/大盲比（越大越偏技术，越小越偏运气）
- Turbo结构：5-8min/轮，Hyper-Turbo：3-5min/轮

来源：[PokerSoup Blind Calculator](https://pokersoup.com/tool/blindStructureCalculator)、[Home Poker Tourney Blinds](https://homepokertourney.org/blinds_samples.htm)

#### ICM模型
- ICM (Independent Chip Model)：将锦标赛筹码转换为美元期望值
- 由Mason Malmuth于1987年引入
- 核心原理：在锦标赛中赢到的筹码的边际价值递减
- Bubble Factor：泡沫期筹码损失的价值 > 筹码获取的价值
- ICM压力下短码倾向保守，大码利用此策略施压

来源：[GGPoker ICM Guide](https://ggpoker.com/blog/intermediate-strategy/understanding-icm-an-essential-component-of-poker-strategy/)、[888poker ICM Guide](https://www.888poker.com/magazine/strategy/poker-tournament/icm-in-poker-tournaments)

#### GTO与EV
- GTO (Game Theory Optimal)：不可被剥削的策略
- EV公式：EV = P(win) × Win Amount - P(lose) × Lose Amount
- Pot Odds：跟注所需胜率 = 跟注额 / (底池+跟注额)
- Implied Odds：考虑后续街可能赢得的额外价值

来源：[GTO Wizard](https://blog.gtowizard.com/what-is-expected-value-in-poker/)

### 麻将番型倍率体系

#### 日本立直麻将
- 基于翻(Han)和符(Fu)双重计分
- 每+1翻，手牌价值约翻倍，直到满贯(Mangan)上限
- 满贯(5翻)=2000基本点，跳满(6-7翻)=3000，倍满(8-10翻)=4000
- 三倍满(11-12翻)=6000，役满(13翻+)=8000基本点
- 闭门手加成：庄家×6，闲家×4（自摸）；庄家×6，闲家×4（放铳）
- Dora(宝牌)机制：每张dora +1翻

来源：[riichi.wiki scoring rules](https://riichi.wiki/Japanese_mahjong_scoring_rules)、[Mahjong Wiki Japanese Modern Scoring](http://mahjong.wikidot.com/rules:japanese-modern-scoring)

#### 中国国标麻将(MCR)
- 最低胡牌要求8番
- 136张牌，无花牌/季节牌
- 番型种类丰富（约81种番型）
- 番数累加制而非指数翻倍

#### 中国地方麻将差异
- 四川麻将：缺门制，血战到底
- 广东麻将：鸡胡/推倒胡，门槛低
- 武汉麻将：开口翻/赖子玩法
- 各地方玩法的概率差异主要来自规则限制（如缺门降低胡牌概率）

来源：[MJ Mall Regional Comparison](https://mahjong-mall.com/blogs/mahjong-blog/riichi-mahjong-vs-other-versions)

### 斗地主数值体系

#### 叫分系统
- 三级叫分：1分/2分/3分
- 叫分即为本局基础倍率
- 玩家可选择叫分或不叫，最高叫分者成为地主

#### 炸弹倍率体系
- 每出一个炸弹或火箭，全局倍率×2
- 火箭(双王)也算炸弹效果
- 春天/反春天：额外×2
- 最终得分 = 基础分 × 叫分倍率 × 2^(炸弹数) × 春天倍率
- 地主赢/输双倍，农民各承担一半

来源：[Wikipedia Dou dizhu](https://en.wikipedia.org/wiki/Dou_dizhu)、[Pagat Doudizhu Rules](https://www.pagat.com/climbing/doudizhu.html)

### 21点(Blackjack)
- 基础庄家优势：约0.5%（完美基本策略）
- 无策略玩家庄家优势：约2-5%
- 6副牌比单副牌庄家优势更高
- 算牌可将优势反转至玩家（+0.5%~+1.5%）

来源：[SHS Conferences Blackjack Paper](https://www.shs-conferences.org/articles/shsconf/pdf/2022/18/shsconf_icprss2022_03038.pdf)、[UNLV Casino Math Guide](https://www.trendfollowing.com/pdfs/casino_math.pdf)

### 百家乐(Baccarat)
- 庄家赢概率：45.8%，庄家优势1.06%
- 闲家赢概率：44.6%，闲家优势1.24%
- 和局概率：9.6%，和局优势14.36%
- 算牌对百家乐几乎无效（与21点不同）

来源：[PlaySmart Baccarat Odds](https://www.playsmart.ca/table-games/baccarat/odds/)、[Cache Creek Baccarat](https://www.cachecreek.com/baccart-odds-probabilities)

## 信息源
- PokerSoup / Home Poker Tourney
- GGPoker / 888Poker 官方策略文档
- riichi.wiki / Mahjong Wiki
- Wikipedia / Pagat.com 规则文档
- UNLV Gaming Studies Research Center
