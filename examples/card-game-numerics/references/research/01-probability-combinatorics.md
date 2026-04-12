# 维度一：概率与组合学基础

## 调研时间
2026-04-12

## 核心发现

### 扑克牌组合数学
- 52张牌中取5张的组合数：C(52,5) = 2,598,960
- 皇家同花顺概率：4/2,598,960 = 1/649,740 ≈ 0.000154%
- 同花顺：36/2,598,960 ≈ 0.00139%
- 四条：624/2,598,960 ≈ 0.024%
- 葫芦：3,744/2,598,960 ≈ 0.144%
- 同花：5,108/2,598,960 ≈ 0.197%
- 顺子：10,200/2,598,960 ≈ 0.392%
- 三条：54,912/2,598,960 ≈ 2.11%
- 两对：123,552/2,598,960 ≈ 4.75%
- 一对：1,098,240/2,598,960 ≈ 42.26%
- 高牌：1,302,540/2,598,960 ≈ 50.12%

来源：[Poker probability - Wikipedia](https://en.wikipedia.org/wiki/Poker_probability)、[McGill MATH 323](https://www.math.mcgill.ca/dstephens/323/knitr/knit-01-Combinatorics.pdf)

### 德州扑克7选5
- 7张牌选5张组合：C(7,5) = 21种子集
- 总可能的7张手牌组合：C(52,7) = 133,784,560
- 不重复手牌等级：7,462种（Cactus Kev分类）

来源：[Math Hawaii Poker Hands](https://www.math.hawaii.edu/~ramsey/Probability/PokerHands.html)

### 麻将组合数学
- 标准麻将136张牌（34种各4张），起手13张
- 胡牌基本结构：4组面子(顺子/刻子) + 1对雀头
- 向听数(deficiency number)：计算从当前手牌到胡牌所需最少换牌数
- 九莲宝灯概率：约0.000113（13张手牌条件下）
- 清一色概率：约1/32
- 大四喜概率：约1/57,856

来源：[Mathematical aspects of Mahjong - arXiv](https://arxiv.org/pdf/1707.07345)、[Mahjong Wiki Combinatorics](http://mahjong.wikidot.com/analysis:combinatorics)

### 条件概率与贝叶斯推理
- 德州扑克中翻牌后计算对手范围的条件概率
- Pot Odds = 跟注金额 / (底池+跟注金额)，与胜率比较决定是否+EV
- EV = P(win) × Amount Won - P(lose) × Amount Lost

来源：[GTO Wizard EV](https://blog.gtowizard.com/what-is-expected-value-in-poker/)、[SplitSuit EV Formula](https://www.splitsuit.com/simple-poker-expected-value-formula)

### 蒙特卡洛模拟验证
- 通过100万+次模拟验证理论概率
- Java/Python模拟结果与理论概率吻合度极高
- 适用于复杂条件概率场景（如已知部分公共牌时的胜率计算）
- Monte Carlo Tree Search (MCTS) 用于评估决策期望收益

来源：[Monte Carlo Poker Simulation](https://petrosdemetrakopoulos.medium.com/estimating-the-outcome-of-a-texas-holdem-game-using-monte-carlo-simulation-1be35be29036)、[IEOMSOCIETY Poker Simulation](http://www.ieomsociety.org/paris2018/papers/31.pdf)

## 信息源
- Wikipedia: Poker probability
- McGill University MATH 323 课程材料
- arXiv: Mathematical aspects of Mahjong (Li, 2017)
- GTO Wizard Blog
- Mahjong Wiki (wikidot)
