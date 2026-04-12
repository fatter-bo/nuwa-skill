# 01-writings: Hart Lambur 的系统性著述与思想

## 核心理论体系

| 理论/概念 | 首次提出 | 核心主张 |
|-----------|---------|---------|
| Cost of Corruption (CoC) | UMA白皮书 | 最小可能的成功贿赂金额——操纵系统的最低成本 |
| Profit from Corruption (PfC) | UMA白皮书 | 通过操纵预言机可获得的最大利润 |
| CoC > PfC不等式 | UMA白皮书 | 只要腐败成本>腐败利润，系统在经济上就是安全的 |
| Optimistic Oracle | 2020 Medium | 乐观假设数据正确，只在争议时验证——lazy evaluation的预言机版本 |
| Oracle as Last Resort | 多次访谈 | 好的预言机应该尽可能少被使用——类似L1对L2的角色 |
| Oracle Extractable Value (OEV) | 2024 Oval发布 | 预言机更新创造的可提取价值——类似MEV但发生在预言机层 |

## 关键技术文章

### "Introducing UMA's Optimistic Oracle"（Medium）
- 核心论点：价格预言机为每个数据对设置链上feed太贵，且限于常见加密货币对
- 乐观预言机的优势：可处理任意数据类型，gas成本低（不争议就不投票）
- 类比：lazy evaluation——如果不需要做额外工作来证明结果，就不做

### "UMA's Data Verification Mechanism"（Medium）
- DVM的技术细节：commit-reveal投票、Schelling Point收敛、质押/罚没机制
- 安全证明：只要CoC > PfC，攻击预言机不经济
- 与51%攻击的类比：需要半数投票者非理性行为才能失败

### Oval公告文章（Medium, 2024.01）
- 与Flashbots合作推出
- 包装Chainlink价格预言机来捕获OEV
- 关键洞察：以太坊领先DeFi协议每年产生数亿美元的OEV
- 务实定位：不替代Chainlink，增值Chainlink

## 思想特征

### 经济学第一性原理
- 所有安全论证最终回到经济激励
- 不依赖「信任好人」——依赖「作恶不经济」
- Goldman交易员的思维训练：理解对手方，理解激励

### 机制设计师的精确性
- 每个概念都有精确定义（CoC = 最小成功贿赂, PfC = 最大腐败利润）
- 用数学不等式而非自然语言表达安全属性
- 类比总是技术性的（lazy evaluation, L1/L2）

### 务实的竞品态度
- 不替代Chainlink → 包装它（Oval）
- 不批评Chainlink → 指出价格预言机的通用局限
- 承认自己的领域边界——不做价格数据
