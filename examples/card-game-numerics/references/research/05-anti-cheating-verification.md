# 维度五：防作弊数值校验

## 调研时间
2026-04-12

## 核心发现

### 异常行为检测

#### 共谋检测
- 机器学习扫描海量手牌历史，检测协作模式
- 检测指标：固定对手频率、同时进出桌频率、让利行为模式
- IP追踪+设备指纹：关联同一物理位置或设备的多个账户
- 下注相关性分析：两个账户间的下注模式统计相关度
- PokerStars 2025年引入生物识别定期验证（高额桌）

来源：[PokerStars Anti-Cheating 2025](https://www.deucescracked.com/pokerstars-introduces-revolutionary-anti-cheating-measures-for-2025/)、[EvenBet Anti-Cheating](https://evenbetgaming.com/blog/articles/how-to-prevent-cheating-in-online-poker/)

#### 刷分检测
- 胜率异常偏离：与同段位玩家胜率的标准差比较
- 收益曲线异常：短时间内收益增长不符合正态分布
- 对局时长异常：过短对局（速输协议）
- 固定时间段活跃模式（自动化脚本特征）

#### 外挂/RTA检测
- Real-Time Assistance(RTA)工具是2025年最大威胁
- 检测策略：操作时间统计分析（人类决策时间有自然分布，Bot趋于一致）
- 进程检测：客户端扫描可疑进程（隐私争议大）
- 行为一致性：对比GTO求解器输出与玩家实际操作的吻合度

来源：[PokerNews Cheating Technology 2025](https://www.pokernews.com/news/2025/12/top-stories-of-2025-cheating-technology-fears-rise-50115.htm)、[Creatiosoft Anti-Cheating](https://creatiosoft.com/blogs/prevent-cheating-in-online-poker/)

### RNG公平性保证

#### Fisher-Yates洗牌算法
- 标准实现：从后向前遍历，每个位置与前方随机位置交换
- 时间复杂度O(n)，空间复杂度O(1)
- 关键：随机源必须是CSPRNG，不能用Math.random()
- 常见错误：Sattolo算法（只产生环排列，不等概率）

#### CSPRNG要求
- AES-256-CTR 或 ChaCha20
- 种子来源：硬件真随机数发生器(HRNG)
- 定期重播种（每1000次或每小时）

#### 第三方认证
- eCOGRA、iTech Labs、GLI 独立测试
- NIST SP 800-22 全部15项统计测试
- 认证确认RNG输出不可预测、不可重复、均匀分布

来源：[Gambling911 RNG Fair Play](https://www.gambling911.com/gambling/random-number-generators-are-silent-guardians-fair-play-online-poker-19-02-2026.html)、[Stake Provably Fair](https://stake.com/provably-fair/conversions)

### Provably Fair（可证公平）

#### 传统认证模式
- 第三方机构审计RNG代码和输出
- 玩家信任基于机构信誉
- 缺点：玩家无法自行验证

#### 密码学可证公平模式
- 发牌前：服务器生成牌序 → 计算Hash → 发送Hash给玩家
- 对局中：正常进行
- 对局后：公开种子(seed) → 玩家可验证Hash一致性
- Commit-Reveal方案：Hash(server_seed + client_seed + nonce) → 牌序

#### CoinPoker模式
- 多方输入随机性：服务器种子 + 每位玩家客户端种子
- 区块链记录种子哈希
- 任何人可在链上验证

来源：[CoinPoker RNG](https://coinpoker.com/rng/)、[Webopedia RNG vs Provably Fair](https://www.webopedia.com/crypto-gambling/casinos/guides/rng-vs-provably-fair-explained/)

### 结果可审计性

- 不可篡改的审计日志：每手牌的完整记录（种子、牌序、操作序列、时间戳）
- HSM签名：硬件安全模块对关键操作签名
- 数据留存：监管要求通常为5-10年
- 玩家可查询：个人历史手牌+验证接口

来源：[PokerDiscover Provably Fair vs RNG Certification](https://pokerdiscover.com/blog/provably-fair-vs-rng-certification)

## 信息源
- PokerStars / PokerNews (2025 anti-cheating measures)
- EvenBet Gaming (anti-cheating best practices)
- CoinPoker (provably fair RNG)
- Gambling911 / Webopedia (RNG fairness)
- Sigma World (provably fair analysis)
