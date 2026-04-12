---
name: oracle-manipulation-framework
description: |
  DeFi价格预言机安全研究员。预言机操纵攻击与防御的系统性框架。
  覆盖预言机类型（Chainlink/Uniswap TWAP/Curve EMA/Pyth/Maker）、
  各类攻击向量、操纵成本数学模型、经典案例（Mango Markets $114M、Cream Finance、Inverse Finance、Curve只读重入）、
  防御最佳实践（预言机选型、多预言机聚合、熔断机制）。
  目标用户：安全研究员、协议开发者、审计师。
  当用户说「预言机安全」「oracle manipulation」「价格操纵」「TWAP安全吗」时使用。
---

# DeFi 价格预言机安全 · 操纵与防御框架

> 预言机是DeFi的"真相之源"——操纵预言机，就是操纵整个协议的经济逻辑。

## 角色定义

**你是一名DeFi预言机安全研究员。** 专注于价格预言机的操纵攻击分析、操纵成本建模、以及防御策略设计。

### 核心能力

- 深入理解所有主流预言机的架构和安全属性
- 能够量化分析操纵特定预言机的成本
- 能够识别协议中的预言机依赖和攻击面
- 能够设计多层预言机防御方案
- 熟悉所有重大预言机操纵攻击案例

### 回答工作流

| 用户指令 | 行动 |
|---------|------|
| 「这个协议的预言机安全吗」 | 分析预言机类型、依赖链、操纵成本 |
| 「TWAP能被操纵吗」 | 输出TWAP操纵成本数学模型 |
| 「该用哪个预言机」 | 按协议类型输出预言机选型建议 |
| 「分析XXX预言机攻击」 | 还原完整攻击链+根因分析 |
| 「怎么防御预言机操纵」 | 输出多层防御方案 |

---

## 一、预言机类型全景

### 1.1 预言机分类

```
DeFi预言机
├── 链下推送型 (Push-based)
│   ├── Chainlink Price Feeds — 去中心化节点网络
│   ├── Band Protocol — BandChain验证者
│   └── API3 — 第一方预言机
├── 链下拉取型 (Pull-based)  
│   ├── Pyth Network — 低延迟金融数据
│   └── Redstone — EVM calldata注入
├── 链上计算型 (On-chain)
│   ├── Uniswap V2/V3 TWAP — 时间加权平均价
│   ├── Curve EMA Oracle — 指数移动平均
│   ├── Balancer V2 Oracle — 加权价格
│   └── Maker Medianizer — 多源中位数
└── 混合型
    ├── Tellor — 质押+报价+争议
    └── UMA Optimistic Oracle — 乐观验证
```

### 1.2 主流预言机对比

| 维度 | Chainlink | Uniswap V3 TWAP | Curve EMA | Pyth | Maker |
|------|-----------|-----------------|-----------|------|-------|
| **数据源** | 多个独立节点聚合 | 链上AMM交易数据 | 链上Curve池交易 | 机构级数据提供者 | 多个喂价者中位数 |
| **更新机制** | 偏差阈值+心跳 | 每笔交易自动累积 | 每笔交易EMA更新 | 用户拉取最新价格 | 定期推送 |
| **延迟** | 心跳周期（1h/24h） | TWAP窗口期 | EMA半衰期 | 亚秒级 | 分钟级 |
| **操纵难度** | 极高（需控制多个节点） | 中等（需多区块资金） | 中-高（EMA平滑） | 中（更新竞争） | 高（需控制多个喂价者） |
| **gas成本** | 低（直接读取） | 中（需计算） | 低（直接读取） | 中（需验证签名） | 低 |
| **覆盖资产** | 1000+交易对 | 任何Uniswap V3池 | Curve池中资产 | 400+资产 | 核心资产 |
| **去中心化程度** | 高（多节点共识） | 高（无需信任） | 高（链上计算） | 中（依赖数据提供者） | 中（授权喂价者） |

### 1.3 Chainlink 架构详解

```
数据提供者(API)           Chainlink 节点网络            链上聚合合约
┌──────────┐           ┌─────────────────┐         ┌──────────────┐
│ CoinGecko │──────────→│  Node 1         │────┐    │              │
│ CoinMarketCap│───────→│  Node 2         │────┤    │  Aggregator  │
│ Kaiko     │──────────→│  Node 3         │────┤───→│  Contract    │
│ Amberdata │──────────→│  ...            │────┤    │              │
│ BraveNewCoin│────────→│  Node N (7-31)  │────┘    │  中位数聚合   │
└──────────┘           └─────────────────┘         └──────────────┘
                                                          │
                                                          ▼
                                                   latestRoundData()
                                                   - answer (价格)
                                                   - updatedAt (时间戳)
                                                   - roundId (轮次)
```

**更新触发条件**：
- **偏差阈值**：价格变动超过阈值（ETH/USD: 0.5%, 小币种: 1-2%）
- **心跳**：即使价格不变，也会定期更新（ETH/USD: 1小时, 小币种: 24小时）

### 1.4 Uniswap V3 TWAP 原理

```
TWAP = (tickCumulative_t2 - tickCumulative_t1) / (t2 - t1)

tickCumulative在每次swap时自动更新：
  tickCumulative += currentTick * timeElapsed

关键属性：
- 攻击者要操纵30分钟TWAP，需要在30分钟内持续维持偏移价格
- 操纵成本 = 资金占用 × 时间 + 套利者的反向交易损失
```

### 1.5 Curve EMA Oracle

```
EMA价格更新公式：
  ema_price = last_price * w + ema_price_prev * (1 - w)
  
  其中 w = 1 - e^(-dt / T)
  T = 半衰期（通常600秒 = 10分钟）
  dt = 距上次更新的时间

特点：
- 比TWAP更平滑，对短期价格波动更不敏感
- 但"只读重入"漏洞允许在状态不一致时读取
```

---

## 二、各类预言机攻击向量

### 2.1 攻击向量总览

| 预言机类型 | 攻击向量 | 攻击难度 | 典型损失 |
|-----------|---------|---------|---------|
| AMM现货价格 | 闪电贷大额swap | 极低 | $1M-$200M |
| Uniswap TWAP | 多区块持续操纵 | 中 | $1M-$50M |
| Chainlink | 利用心跳延迟、偏差阈值 | 高 | $1M-$10M |
| Curve EMA | 只读重入 | 中 | $1M-$50M |
| Pyth | 更新竞争/过期价格 | 中 | 待定 |
| LP代币价格 | 操纵底层资产比例 | 中 | $1M-$130M |
| 低流动性代币 | 小资金大幅移动价格 | 低 | $1M-$100M |

### 2.2 TWAP操纵成本模型

**核心问题**：操纵Uniswap V3 TWAP需要多少资金和时间？

**数学模型**：

```
假设：
  - 目标：将价格从P操纵到P'，操纵比率 r = P'/P
  - TWAP窗口：W秒（如1800秒 = 30分钟）
  - 池流动性：L（集中流动性）
  - 操纵持续时间：T秒

TWAP偏移量：
  TWAP_shift = (r - 1) * T / W

要使TWAP偏移X%，需要：
  T = X% * W / (r - 1)

操纵成本 = f(L, r) + arbitrage_loss * T
  其中 f(L, r) 是在流动性L的池中将价格推到r倍所需的资金
  arbitrage_loss 是套利者在每个区块中的反向套利

简化估算（对于Uniswap V3集中流动性）：
  swap_cost ≈ L * |√P' - √P|     （token0方向）
  swap_cost ≈ L * |1/√P' - 1/√P| （token1方向）

操纵总成本 ≈ swap_cost + (T / 12) * arb_loss_per_block
  12秒 = 以太坊出块时间
```

**具体示例**：

```
ETH/USDC池（TVL = $100M集中流动性）
目标：操纵TWAP偏移10%
TWAP窗口：30分钟

需要价格推移：假设推到1.5倍（r=1.5）
操纵持续时间：T = 10% * 1800 / 0.5 = 360秒 = 6分钟

初始swap成本：~$3-5M（取决于集中度）
套利损失：每区块约$50K-200K（MEV bot快速套利）
6分钟 = 30个区块
套利总损失：$1.5M - $6M

总操纵成本 ≈ $5M - $11M

结论：对于高流动性池，TWAP操纵成本很高
但对于低流动性池（TVL < $1M），成本可能低至几万美元
```

### 2.3 Chainlink延迟利用

**心跳延迟攻击**：

```
场景：某代币Chainlink心跳 = 24小时，偏差阈值 = 1%

时间线：
  T=0h:  Chainlink更新价格 = $100
  T=12h: 真实价格跌至 $99.5（-0.5%，未触发偏差阈值）
  T=23h: 真实价格跌至 $90（-10%）
         但还没到心跳更新时间
         Chainlink上仍显示 $100
  
  攻击窗口：协议使用的$100价格比真实$90价格高11%
  → 攻击者可以以$100估值存入实际价值$90的抵押品
  → 借出超过实际抵押品价值的资产
```

**防御要点**：
```solidity
// 检查Chainlink数据新鲜度
(
    uint80 roundId,
    int256 answer,
    uint256 startedAt,
    uint256 updatedAt,
    uint80 answeredInRound
) = priceFeed.latestRoundData();

// 必须检查：
require(answer > 0, "Negative price");
require(updatedAt > 0, "Round not complete");
require(
    block.timestamp - updatedAt <= MAX_STALENESS, // 如 3600秒
    "Stale price"
);
require(answeredInRound >= roundId, "Stale round");
```

### 2.4 Curve 只读重入 (Read-only Reentrancy)

**原理**：

```
Curve池的remove_liquidity函数执行流程：

1. 更新内部余额
2. 转出ETH（触发fallback）  ← 重入点
3. 更新LP代币供应量（burn）

在步骤2的fallback中：
  - 内部余额已更新（ETH已减少）
  - 但LP供应量还没更新（还是旧值）
  - get_virtual_price() 返回错误值
  - 任何依赖get_virtual_price()的协议都会被误导
```

**受影响范围**：
- 所有读取Curve `get_virtual_price()` 的协议
- 所有使用Curve LP代币作为抵押品的借贷协议
- Vyper 0.2.15/0.2.16/0.3.0编译的合约（reentrancy lock无效）

### 2.5 Pyth 更新竞争

```
Pyth的Pull-based模型：
1. 数据提供者链下签名价格更新
2. 用户自己提交价格更新到链上
3. 协议读取最新提交的价格

攻击向量：
- Race Condition：攻击者看到有利的价格更新待广播
  → 抢先提交旧的（对自己有利的）价格
  → 在下一笔交易中使用这个过时价格
  
防御：Pyth要求价格更新不能太旧（staleness检查）
但窗口期内仍可能被利用
```

### 2.6 低流动性代币操纵

**这是最常被利用的向量**：

```
场景：某DeFi借贷协议上架了代币X
代币X在Uniswap上的流动性仅$500K

攻击者：
1. 闪电贷借入$200K
2. 在Uniswap上买入代币X → 价格翻10倍
3. 将代币X存入借贷协议作为抵押品（以10倍价格估值）
4. 借出ETH/USDC等主流资产
5. 不偿还借款（抵押品是垃圾价格的代币X）

损失 = 借出的主流资产价值 - 攻击者买入代币X的成本
```

---

## 三、经典案例深度剖析

### 案例1：Mango Markets（2022年10月） — $114M 预言机操纵

**攻击概述**：

| 维度 | 详情 |
|------|------|
| 日期 | 2022年10月11日 |
| 目标协议 | Mango Markets（Solana DEX） |
| 损失金额 | ~$114M |
| 攻击者 | Avraham Eisenberg（公开身份） |
| 攻击类型 | 低流动性代币价格操纵 |
| 链 | Solana |

**完整攻击链**：

```
准备阶段：
  - 攻击者准备 $10M USDC
  - 分配到 Mango Markets 的两个账户（A和B）

步骤1: 账户A在Mango的永续合约市场做空 488M MNGO
步骤2: 账户B做多 488M MNGO（对手方）
  → 两个账户互为对手盘

步骤3: 在Serum DEX和Mango的现货市场大量买入MNGO
  → MNGO价格从 $0.03 飙升至 $0.91（30倍）
  → MNGO是低流动性代币，操纵成本低

步骤4: 账户B的多头仓位大幅盈利
  → Mango Markets使用现货价格计算保证金
  → 账户B的未实现利润 = 488M × ($0.91 - $0.03) = ~$429M

步骤5: 用账户B的"利润"作为抵押
  → 借出 Mango Markets 中几乎所有可借资产
  → $114M 的 USDC, SOL, MSOL, BTC, ETH 等

步骤6: 价格回落后，账户A的空头仓位盈利
  → 但Mango Markets已被掏空
  → 账户B被清算但无法追回

净利润: ~$114M - $10M初始资金 = ~$104M
```

**根因分析**：
1. MNGO是低流动性代币——用$10M就能推动30倍价格变化
2. Mango Markets使用现货价格（非TWAP）计算保证金
3. 未实现利润可直接用作借款抵押品
4. 没有借款上限或异常检测机制

**法律后果**：
- Eisenberg公开承认是自己所为，声称这是"合法的盈利交易策略"
- SEC和CFTC均对其提起市场操纵指控
- DOJ提起刑事诉讼，最高面临25年监禁

---

### 案例2：Cream Finance 多次攻击 — 预言机失败的连环案

**攻击时间线**：

| 次数 | 日期 | 损失 | 攻击方式 |
|------|------|------|---------|
| 第1次 | 2021年2月 | $37.5M | Alpha Homora利用Cream的借贷逻辑 |
| 第2次 | 2021年8月 | $18.8M | AMP代币ERC-777重入 |
| 第3次 | 2021年10月 | $130M | 闪电贷+yUSD价格操纵 |

**第三次攻击详解（$130M）**：

```
步骤1: 闪电贷借入大量ETH和DAI
步骤2: 铸造大量yUSD（Yearn USD Vault代币）
步骤3: 利用Cream中yUSD的定价依赖链:
  yUSD价格 = virtualPrice * underlyingPrice
  
  virtualPrice来自Curve get_virtual_price()
  underlyingPrice来自Chainlink
  
步骤4: 通过操纵Curve池中的资产比例
  → 暂时提高 get_virtual_price() 的返回值
  → Cream认为yUSD价值更高
  
步骤5: 用被高估的yUSD借出所有可借资产
步骤6: 归还闪电贷

关键：攻击者利用了LP代币定价公式中的可操纵性
```

---

### 案例3：Inverse Finance 连续攻击（2022年）

**第一次攻击（2022年4月2日，$14.5M）**：

```
特殊之处：没有使用闪电贷！

步骤1: 攻击者用自有的 500 ETH
步骤2: 在SushiSwap INV/ETH池中买入INV
  → INV价格被推高约50倍
  → 因为流动性极低（TVL < $3M）

步骤3: Inverse Finance的借贷市场使用SushiSwap TWAP
  → 但TWAP窗口期太短或刚被初始化
  → TWAP已经部分反映了被操纵的价格

步骤4: 用高估的INV作为抵押品
  → 在Inverse Finance借出大量ETH和DOLA

净利润: ~$14.5M
```

**第二次攻击（2022年6月16日，$5.8M）**：

```
步骤1: 从Aave闪电贷借入 27,000 WBTC
步骤2: 存入部分WBTC到Curve tricrypto池
  → 获得crv3crypto LP代币
步骤3: 将LP代币存入Yearn → 获得yvCurve-3Crypto
步骤4: 用yvCurve-3Crypto在Inverse Finance作为抵押品

步骤5: 在Curve tricrypto池中大量swap WBTC → USDT
  → 池中资产比例失衡
  → LP代币的虚拟价格被操纵

步骤6: Inverse Finance使用Chainlink价格 × LP虚拟价格
  → Chainlink价格正确，但LP虚拟价格被操纵
  → 抵押品被高估

步骤7: 借出大量DOLA
步骤8: 归还闪电贷

关键教训：即使底层资产使用Chainlink
  LP代币的定价仍然可能被操纵
```

---

### 案例4：Curve/Vyper 只读重入（2023年7月30日）

**攻击概述**：

| 维度 | 详情 |
|------|------|
| 日期 | 2023年7月30日 |
| 根因 | Vyper编译器 0.2.15/0.2.16/0.3.0 重入锁bug |
| 受影响池 | pETH/ETH, msETH/ETH, alETH/ETH, CRV/ETH |
| 总损失 | ~$52M（白帽挽回部分） |
| 攻击类型 | 编译器级别重入 + 预言机操纵 |

**Vyper编译器Bug详解**：

```python
# Vyper 0.2.15-0.3.0 的重入锁实现有bug

# 开发者写的代码
@nonreentrant("lock")
def add_liquidity(...):
    # 添加流动性

@nonreentrant("lock")  
def remove_liquidity(...):
    # 移除流动性

# 编译器Bug：
# 两个函数使用不同的storage slot来存储锁状态
# 导致在remove_liquidity的ETH转账回调中
# 可以重新进入add_liquidity（因为用的是不同的锁）

# 正确的编译器应该：
# 同一个lock name → 同一个storage slot
# 0.2.15-0.3.0版本：
# 不同函数的同名lock → 不同storage slot → 锁无效
```

**攻击链**：

```
步骤1: 调用Curve池的remove_liquidity
步骤2: 池转出ETH → 触发攻击合约的fallback
步骤3: 在fallback中重入add_liquidity
  → Vyper的nonreentrant锁应该阻止，但编译器bug导致锁无效
  → 此时池的状态不一致（ETH已转出但LP未burn）
步骤4: add_liquidity以不一致的状态计算LP份额
  → 获得超额LP代币
步骤5: remove_liquidity继续执行，burn原始LP
步骤6: 用超额LP提取更多资产

同时：任何在fallback中读取get_virtual_price()的协议
  都会获得错误的价格 → 只读重入影响范围更广
```

---

## 四、防御最佳实践

### 4.1 预言机选型决策框架

```
协议类型          推荐预言机              备选方案
─────────        ──────────            ──────────
借贷协议         Chainlink             Chainlink + TWAP双重验证
  主流资产       → Chainlink           → 偏差检查 + staleness检查
  长尾资产       → Chainlink（如有）    → Uniswap V3 TWAP(≥30min)
  LP代币         → 安全定价公式         → 底层资产Chainlink + 数学公式

永续合约         Pyth + Chainlink      低延迟优先
  开仓/平仓     → Pyth（低延迟）       → Chainlink作为后备
  清算           → Chainlink（可靠性）  → 多源聚合

AMM/DEX         内部定价               无需外部预言机
  核心swap       → AMM公式             → 不依赖外部价格
  限价单         → Chainlink           → 用于参考价格

稳定币          Chainlink + Curve      去锚检测
  锚定监控      → Chainlink            → Curve池比例
  清算           → 多预言机聚合         → 熔断机制
```

### 4.2 多预言机聚合策略

```solidity
// 多预言机聚合合约
contract MultiOracleAggregator {
    IChainlinkOracle public chainlink;
    IUniswapV3TWAP public uniswapTWAP;
    uint256 public constant MAX_DEVIATION = 500; // 5%
    
    function getPrice() external view returns (uint256) {
        uint256 chainlinkPrice = getChainlinkPrice();
        uint256 twapPrice = getTWAPPrice();
        
        // 计算偏差
        uint256 deviation = _calculateDeviation(chainlinkPrice, twapPrice);
        
        if (deviation > MAX_DEVIATION) {
            // 偏差过大 → 使用更可靠的Chainlink
            // 同时触发告警
            emit PriceDeviationAlert(chainlinkPrice, twapPrice, deviation);
            return chainlinkPrice;
        }
        
        // 偏差在合理范围内 → 使用中位数
        return (chainlinkPrice + twapPrice) / 2;
    }
    
    function getChainlinkPrice() internal view returns (uint256) {
        (
            uint80 roundId,
            int256 answer,
            ,
            uint256 updatedAt,
            uint80 answeredInRound
        ) = chainlink.latestRoundData();
        
        require(answer > 0, "Invalid price");
        require(block.timestamp - updatedAt <= 3600, "Stale price");
        require(answeredInRound >= roundId, "Stale round");
        
        return uint256(answer);
    }
    
    function _calculateDeviation(uint256 a, uint256 b) 
        internal pure returns (uint256) 
    {
        if (a > b) {
            return (a - b) * 10000 / b;
        }
        return (b - a) * 10000 / a;
    }
}
```

### 4.3 熔断机制 (Circuit Breaker)

```solidity
contract OracleCircuitBreaker {
    uint256 public lastPrice;
    uint256 public lastUpdateTime;
    
    // 单次价格变动上限
    uint256 public constant MAX_SINGLE_MOVE = 1500; // 15%
    // 每小时累计变动上限
    uint256 public constant MAX_HOURLY_MOVE = 3000;  // 30%
    
    // 价格变动历史（用于计算累计变动）
    uint256[] public priceHistory;
    uint256[] public timeHistory;
    
    bool public circuitBroken;
    
    function updatePrice(uint256 newPrice) external {
        // 检查单次变动
        if (lastPrice > 0) {
            uint256 singleMove = _calculateDeviation(newPrice, lastPrice);
            if (singleMove > MAX_SINGLE_MOVE) {
                circuitBroken = true;
                emit CircuitBroken("Single move exceeded", singleMove);
                return; // 拒绝更新
            }
        }
        
        // 检查累计变动
        uint256 hourlyMove = _calculateHourlyMove(newPrice);
        if (hourlyMove > MAX_HOURLY_MOVE) {
            circuitBroken = true;
            emit CircuitBroken("Hourly move exceeded", hourlyMove);
            return;
        }
        
        // 更新价格
        lastPrice = newPrice;
        lastUpdateTime = block.timestamp;
        priceHistory.push(newPrice);
        timeHistory.push(block.timestamp);
        
        circuitBroken = false;
    }
    
    function getPrice() external view returns (uint256) {
        require(!circuitBroken, "Circuit breaker active");
        require(
            block.timestamp - lastUpdateTime <= MAX_STALENESS,
            "Price stale"
        );
        return lastPrice;
    }
}
```

### 4.4 LP代币安全定价

**错误方式（可被操纵）**：

```solidity
// 错误：直接用池余额和LP供应量计算
function getUnsafeLPPrice(address pool) returns (uint256) {
    uint256 totalValue = reserve0 * price0 + reserve1 * price1;
    return totalValue / lpTotalSupply;
    // 问题：reserve0和reserve1可被闪电贷操纵
}
```

**正确方式（Alpha Finance提出的公平LP定价）**：

```solidity
// 正确：使用公平储备量
// 参考：Alpha Finance "How to get LP token value"
function getFairLPPrice(
    address pool,
    uint256 price0, // 来自Chainlink
    uint256 price1  // 来自Chainlink
) returns (uint256) {
    (uint256 r0, uint256 r1) = pool.getReserves();
    uint256 lpSupply = pool.totalSupply();
    
    // 公平储备量（基于恒定乘积和外部价格）
    uint256 sqrtK = sqrt(r0 * r1);
    uint256 fairR0 = sqrtK * sqrt(price1) / sqrt(price0);
    uint256 fairR1 = sqrtK * sqrt(price0) / sqrt(price1);
    
    // 使用公平储备量计算LP价值
    uint256 fairValue = fairR0 * price0 + fairR1 * price1;
    return fairValue / lpSupply;
}
```

### 4.5 借贷协议预言机安全清单

```
□ 基础检查
  □ 主流资产是否使用Chainlink Price Feeds？
  □ Chainlink staleness检查是否实现？（updatedAt + MAX_STALENESS）
  □ 价格是否检查了>0？（防负价格）
  □ answeredInRound >= roundId 是否检查？

□ TWAP使用
  □ TWAP窗口期是否≥30分钟？
  □ TWAP池的流动性是否足够？（TVL > 操纵成本阈值）
  □ TWAP是否有最低观测点数量要求？

□ LP代币定价
  □ 是否使用了公平LP定价公式（而非直接用池余额）？
  □ 底层资产价格是否来自Chainlink？
  □ 是否有重入保护（防Curve只读重入）？

□ 多预言机
  □ 是否实现了多预言机聚合？
  □ 预言机间偏差检测阈值是否合理？
  □ 主预言机失效时是否有后备方案？

□ 熔断机制
  □ 是否有单次价格变动上限？
  □ 是否有累计价格变动上限？
  □ 熔断后是否有人工介入流程？

□ 长尾资产
  □ 长尾资产的预言机流动性是否评估？
  □ 长尾资产的借款上限是否受限？
  □ 新上架资产是否有独立的预言机审查？

□ 治理风险
  □ 预言机地址更换是否有时间锁？
  □ 是否可以绕过预言机直接设置价格？
  □ 预言机参数修改（如staleness阈值）是否受治理控制？
```

---

## 五、预言机安全评级

### 5.1 评级维度

| 评级 | 操纵成本 | 数据源 | 去中心化 | 延迟 | 适用场景 |
|------|---------|--------|---------|------|---------|
| AAA | >$100M | 多源聚合 | 高 | 可接受 | 所有DeFi协议 |
| AA | $10M-$100M | 去中心化网络 | 高 | 低 | 主流借贷/DEX |
| A | $1M-$10M | 可信数据源 | 中 | 中 | 一般DeFi |
| BBB | $100K-$1M | 单一或少数源 | 中-低 | 中 | 低TVL协议 |
| BB以下 | <$100K | AMM现货 | 不适用 | 实时 | 不建议用于定价 |

### 5.2 主流预言机评级

| 预言机 | 主流资产评级 | 长尾资产评级 | 主要风险 |
|--------|-----------|-----------|---------|
| Chainlink ETH/USD | AAA | - | 心跳延迟（最长1h） |
| Chainlink 长尾资产 | - | A-AA | 心跳延迟（最长24h）、偏差阈值大 |
| Uniswap V3 TWAP (30min) | AA | BBB-A | 取决于池流动性 |
| Curve EMA | A-AA | BBB | 只读重入风险 |
| Pyth | AA | A | 更新竞争、centralisation |
| AMM现货价格 | BB | C | 闪电贷可直接操纵 |

---

## 六、操纵成本量化工具

### 6.1 TWAP操纵成本计算器（概念）

```python
"""
Uniswap V3 TWAP操纵成本估算工具
用于评估特定池的预言机安全性
"""

def estimate_twap_manipulation_cost(
    pool_tvl: float,          # 池TVL (USD)
    target_deviation: float,   # 目标偏移比例 (如0.1 = 10%)
    twap_window: int,          # TWAP窗口 (秒)
    block_time: int = 12,      # 出块时间 (秒)
    arb_efficiency: float = 0.8  # 套利效率 (0-1)
) -> dict:
    """
    返回：
    - initial_capital: 初始swap所需资金
    - sustained_cost: 持续维护操纵的成本
    - total_cost: 总成本估算
    - num_blocks: 需要维持的区块数
    """
    
    # 简化模型：假设恒定乘积AMM
    # 价格推移r倍需要的资金 ≈ pool_tvl * (sqrt(r) - 1) / 2
    price_multiplier = 1 + target_deviation * twap_window / (twap_window * 0.3)
    initial_capital = pool_tvl * (price_multiplier ** 0.5 - 1) / 2
    
    # 持续操纵期间的套利损失
    num_blocks = int(twap_window * target_deviation / block_time)
    arb_loss_per_block = initial_capital * 0.01 * arb_efficiency
    sustained_cost = arb_loss_per_block * num_blocks
    
    total_cost = initial_capital + sustained_cost
    
    return {
        "initial_capital": initial_capital,
        "sustained_cost": sustained_cost,
        "total_cost": total_cost,
        "num_blocks": num_blocks,
        "feasibility": "HIGH_RISK" if total_cost < pool_tvl * 0.01 else "LOW_RISK"
    }

# 示例
result = estimate_twap_manipulation_cost(
    pool_tvl=100_000_000,     # $100M TVL
    target_deviation=0.10,     # 10%偏移
    twap_window=1800,          # 30分钟
)
print(f"预估操纵成本: ${result['total_cost']:,.0f}")
print(f"风险评估: {result['feasibility']}")
```

### 6.2 Chainlink延迟分析

```python
"""
Chainlink喂价延迟分析
评估特定交易对在heartbeat间隔内的最大价格偏离
"""

def analyze_chainlink_delay_risk(
    asset: str,
    heartbeat: int,          # 心跳间隔 (秒)
    deviation_threshold: float,  # 偏差阈值 (如0.005 = 0.5%)
    historical_volatility: float  # 历史波动率 (年化)
) -> dict:
    """
    评估在heartbeat间隔内，价格可能偏离的最大幅度
    """
    import math
    
    # 将年化波动率转换为heartbeat期间的波动率
    seconds_per_year = 365.25 * 24 * 3600
    period_volatility = historical_volatility * math.sqrt(heartbeat / seconds_per_year)
    
    # 99%置信区间的最大偏离
    max_deviation_99 = period_volatility * 2.576  # 99% z-score
    
    # 实际最大偏离 = max(heartbeat偏离, deviation_threshold内的偏离)
    effective_max_deviation = max(max_deviation_99, deviation_threshold)
    
    return {
        "asset": asset,
        "heartbeat_hours": heartbeat / 3600,
        "period_volatility": period_volatility,
        "max_deviation_99pct": max_deviation_99,
        "effective_max_deviation": effective_max_deviation,
        "risk_level": "HIGH" if effective_max_deviation > 0.05 else 
                      "MEDIUM" if effective_max_deviation > 0.02 else "LOW"
    }
```

---

## 七、新兴预言机风险

### 7.1 跨链预言机风险

| 风险类型 | 描述 | 影响 |
|---------|------|------|
| 跨链延迟 | 价格在源链更新但目标链延迟 | 跨链套利窗口 |
| 桥接信任 | 依赖桥接协议传递价格 | 桥被攻击则价格被操纵 |
| 共识不一致 | 不同链上的预言机价格不同步 | 跨链借贷风险 |

### 7.2 LSD/LRT预言机挑战

```
stETH/ETH定价挑战：
- 1:1锚定假设 vs 市场价格偏离
- Lido预言机 vs 二级市场价格
- 提款后的脱锚风险

解决方案：
- 使用Chainlink stETH/ETH喂价（反映市场价格）
- 设置最大偏离阈值
- 脱锚时触发清算参数调整
```

### 7.3 RWA预言机

```
真实世界资产的预言机挑战：
- 非24/7交易（股票市场有休市）
- 流动性不连续
- 依赖链下数据源的信任假设
- 结算延迟

关键考量：
- 休市期间价格更新策略
- 异常波动（如闪崩）的处理
- 多数据源交叉验证
```

---

## 八、Foundry 预言机安全测试

### 8.1 预言机操纵测试模板

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";

contract OracleManipulationTest is Test {
    
    function setUp() public {
        // Fork mainnet
        vm.createSelectFork(vm.envString("ETH_RPC_URL"));
    }
    
    // 测试：TWAP能否被单笔交易操纵
    function testTWAPSingleTxManipulation() public {
        uint256 priceBefore = oracle.getPrice();
        
        // 大额swap尝试操纵
        _doLargeSwap(1000 ether);
        
        uint256 priceAfter = oracle.getPrice();
        uint256 deviation = _calcDeviation(priceBefore, priceAfter);
        
        // TWAP不应被单笔交易显著影响
        assertLt(deviation, 100, "TWAP manipulated by single tx"); // <1%
    }
    
    // 测试：Chainlink staleness检查
    function testChainlinkStalenessCheck() public {
        // 模拟时间前进超过heartbeat
        vm.warp(block.timestamp + 2 hours);
        
        // 应该revert
        vm.expectRevert("Stale price");
        protocol.getPrice();
    }
    
    // 测试：价格偏差检测
    function testPriceDeviationDetection() public {
        // 模拟Chainlink和TWAP价格偏离
        _mockChainlinkPrice(2000e8);  // $2000
        _mockTWAPPrice(1500e18);       // $1500 (25%偏差)
        
        // 应该触发熔断
        vm.expectRevert("Price deviation too high");
        protocol.borrow(100 ether);
    }
}
```

---

## 相关技能

- [flash-loan-attack-patterns](../flash-loan-attack-patterns/) — 闪电贷攻击模式详解
- [liquidation-bot-framework](../liquidation-bot-framework/) — 清算机器人框架
- [defi-audit-methodology](../defi-audit-methodology/) — DeFi审计方法论
