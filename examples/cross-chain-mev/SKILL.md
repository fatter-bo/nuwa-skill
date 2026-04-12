---
name: cross-chain-mev
description: |
  跨链MEV研究者。专注L1与L2之间、L2与L2之间的信息不对称套利。
  涵盖信息不对称来源（区块时间差12s vs 2s、桥延迟、终局性差异、预言机更新时机）、
  L1↔L2套利（ETH价格领先、sequencer feed优势、桥成本/延迟）、
  L2↔L2套利（同token价格差、跨L2桥比较、原子性风险）、
  技术挑战（多链状态同步、执行风险、资金预部署）、
  前沿研究（Shared Sequencer影响、Intent架构、SUAVE跨链）。
  包含L2区块时间/终局性对比表、跨链监控架构设计。
  当用户问「跨链MEV怎么做」「L2套利机会」「信息不对称」时使用。
---

# 跨链MEV研究

> 跨链MEV是MEV研究的前沿领域——L1、L2、甚至跨不同L2之间的信息不对称创造了新一代套利机会。

## 使用说明

**本Skill聚焦跨链MEV的机制、策略和技术挑战。** 当用户询问L1-L2套利、跨L2价格差、跨链信息不对称时，按对应章节提供分析。

---

## 一、信息不对称来源

### 1.1 跨链信息不对称的本质

```
信息不对称 = 同一信息在不同链上的确认时间差

核心洞察：
所有链都在观察同一个外部世界（ETH价格、BTC价格等），
但每条链"感知"到价格变化的时间不同。

信息传播链：
真实世界事件 → CEX价格变动 → L1价格变动 → L2价格变动
                                         → 另一条L2价格变动

每一步都有延迟，延迟 = 套利机会
```

### 1.2 L2区块时间与终局性对比

| L2 | 区块时间 | 软终局性 | 硬终局性(L1确认) | Sequencer类型 | MEV特征 |
|----|---------|---------|----------------|-------------|---------|
| **Arbitrum One** | 0.25s | ~0.25s | ~7天(挑战期) | 中心化(Offchain Labs) | FCFS排序，延迟竞赛 |
| **Optimism** | 2s | ~2s | ~7天(挑战期) | 中心化(OP Foundation) | Priority fee排序 |
| **Base** | 2s | ~2s | ~7天(挑战期) | 中心化(Coinbase) | 类似OP |
| **zkSync Era** | ~1s | ~1s | ~1-3小时(ZK证明) | 中心化(Matter Labs) | 自定义排序 |
| **Polygon zkEVM** | ~2s | ~2s | ~30分钟(ZK证明) | 中心化 | 自定义排序 |
| **Scroll** | ~3s | ~3s | ~1-4小时(ZK证明) | 中心化 | 自定义排序 |
| **Linea** | ~2s | ~2s | ~1-2小时(ZK证明) | 中心化(Consensys) | 自定义排序 |
| **Ethereum L1** | 12s | 12s | ~15分钟(32 slots) | 去中心化 | Builder竞价 |

### 1.3 信息不对称的四种类型

```
类型1: 区块时间差（Block Time Asymmetry）
- L1 12s vs Arbitrum 0.25s
- 同一事件在L1上确认需要更长时间
- Arbitrum可以"先看到"L1上即将发生的交易

类型2: 预言机更新延迟（Oracle Update Lag）
- Chainlink在不同链上的更新频率不同
- L1上的价格更新可能比L2早几秒到几分钟
- 例：Chainlink在L1更新ETH价格 → L2上的Aave仍使用旧价格

类型3: 桥传输延迟（Bridge Delay）
- 资金/消息通过桥传输需要时间
- 不同桥的延迟差异巨大（几分钟到几天）
- 在桥消息到达前，两端的状态不一致

类型4: 终局性差异（Finality Gap）
- Optimistic Rollup: 7天挑战期
- ZK Rollup: 证明生成时间（分钟到小时）
- 终局性前的交易存在被revert的风险
```

---

## 二、L1↔L2套利

### 2.1 ETH价格领先套利

```
最基本的L1-L2套利形式：

场景：
1. CEX上ETH价格突然从$2000涨到$2050
2. L1 Uniswap上的价格在12秒内被套利到$2050
3. Arbitrum上的Uniswap价格仍是$2000（延迟0.25-2s）
4. 套利者在Arbitrum上以$2000买入ETH

执行方式：
方案A（纯DEX）：
- 在Arbitrum上买入ETH
- 等待价格收敛后在Arbitrum上卖出
- 风险：价格可能不收敛或反向移动

方案B（CEX对冲）：
- 在Arbitrum上买入ETH
- 同时在CEX做空ETH
- 等待价格收敛后两端平仓
- 更低风险但需要CEX账户和保证金

方案C（原子跨链）：
- 目前不可行（跨链无法原子执行）
- 未来可能通过Shared Sequencer或Intent实现
```

### 2.2 Sequencer Feed优势

```
Arbitrum Sequencer Feed:

Arbitrum的sequencer运行独立的feed，可以直接连接获取：
- 最新的L2交易
- Sequencer排序的交易顺序
- 比公共RPC更快的数据

连接方式：
- 直接WebSocket连接到sequencer endpoint
- 比通过RPC节点快 ~100-500ms

利用方式：
1. 通过sequencer feed看到大额swap
2. 在同一个或下一个L2区块中提交backrun交易
3. 利用L2内部的价格偏差套利

Arbitrum Sequencer地址和端点：
- Sequencer Inbox: 发布到L1的入口
- Delayed Inbox: 延迟消息入口（绕过sequencer）
- Sequencer Feed: wss://arb1.arbitrum.io/feed
  （注：实际端点可能变化，需查阅最新文档）
```

### 2.3 L1→L2消息套利

```
利用L1→L2消息传递延迟的套利：

场景：L1上的Chainlink预言机更新ETH价格
1. L1预言机更新价格（区块N）
2. L2的预言机需要等待L1→L2消息传递
3. 传递延迟约 5-15分钟（取决于L2类型）
4. 在L2价格更新前，利用旧价格在L2 DeFi协议中操作

具体策略：
- 在L2上借入（使用旧的、较高的抵押品价格）
- 等L2价格更新后，抵押品价格下跌
- 但你的借款已经完成

风险：
- 协议可能有额外的安全机制（如L2 sequencer uptime feed检查）
- Aave V3在L2上有Sequencer Uptime Feed检查
- 如果sequencer不可用，协议会暂停清算（grace period）
```

### 2.4 桥成本与延迟对比

| 桥类型 | 示例 | L1→L2延迟 | L2→L1延迟 | 成本 | 安全模型 |
|--------|------|----------|----------|------|---------|
| **原生桥** | Arbitrum Bridge | ~10分钟 | ~7天 | Gas only | 最高（Rollup安全性） |
| **原生桥** | OP Bridge | ~15分钟 | ~7天 | Gas only | 最高 |
| **快速桥** | Across Protocol | ~2分钟 | ~2分钟 | 0.04-0.12% | 中（流动性网络） |
| **快速桥** | Stargate | ~5分钟 | ~5分钟 | 0.06% | 中（LayerZero消息） |
| **快速桥** | Hop Protocol | ~5-15分钟 | ~5-15分钟 | 0.04-0.3% | 中（Bonder网络） |
| **意图桥** | deBridge | ~1-5分钟 | ~1-5分钟 | 变动 | 中（Solver网络） |

---

## 三、L2↔L2套利

### 3.1 同Token跨L2价格差

```
常见的L2间价格差来源：

1. 流动性差异
   - Arbitrum上的ETH/USDC池可能有$500M TVL
   - Base上的同一池可能只有$50M TVL
   - 同样的买入量在Base上造成更大的价格影响

2. 用户活动差异
   - 某个大户在Arbitrum上大量买入ETH
   - Base上的价格没有同步变动
   - 产生短暂的价格差

3. 预言机更新不同步
   - 不同L2的Chainlink更新频率不同
   - 价格偏差可能持续数分钟

监控策略：
for each token in watchlist:
    prices = {}
    for each l2 in [arbitrum, optimism, base, zksync]:
        prices[l2] = getDEXPrice(token, l2)
    
    for each pair in combinations(l2s, 2):
        spread = abs(prices[pair[0]] - prices[pair[1]]) / min(prices)
        if spread > threshold:
            evaluateArbitrage(token, pair, spread)
```

### 3.2 跨L2套利执行

```
跨L2套利的执行流程：

方案1: 桥接执行（慢但直接）
1. 在价格低的L2上买入代币
2. 通过桥将代币转移到价格高的L2
3. 在目标L2上卖出

问题：桥传输期间价格可能已经收敛

方案2: 预部署资金（快但需要资本）
1. 在多条L2上预先部署资金
2. 当发现价格差时，同时在两条L2上操作：
   - L2_A（便宜端）: 买入
   - L2_B（贵端）: 卖出
3. 定期通过桥rebalance资金

方案3: Intent/Solver模式（前沿）
1. 提交跨链交易意图
2. Solver网络执行跨链套利
3. 通过结算层验证和清算

// 跨L2套利bot的核心循环
async function crossL2ArbitrageLoop() {
    while (true) {
        // 获取所有L2的价格
        const prices = await Promise.all(
            L2_CONFIGS.map(l2 => getPoolPrices(l2))
        );

        // 找到价格差
        const opportunities = findArbitrageOpportunities(prices);

        for (const opp of opportunities) {
            const profit = calculateProfit(opp);
            const risk = assessRisk(opp);

            if (profit > MIN_PROFIT && risk < MAX_RISK) {
                await executeArbitrage(opp);
            }
        }

        await sleep(POLL_INTERVAL);
    }
}
```

### 3.3 原子性风险管理

```
跨链套利的最大风险：无法保证原子性

风险场景：
1. 在L2_A上成功买入ETH
2. 准备在L2_B上卖出时，价格已经变动
3. L2_B上的卖出交易失败或亏损
4. 净结果：持有不想要的ETH头寸

风险管理策略：

1. 头寸限制
   - 单笔最大头寸: 总资金的5%
   - 单链最大敞口: 总资金的20%

2. 价格偏差阈值
   - 只在偏差 > 桥成本 + 安全余量 时执行
   - 安全余量通常为预期利润的50%

3. 时间窗口控制
   - 如果2分钟内无法完成双端执行，放弃
   - 使用限价单而非市价单

4. 对冲策略
   - 在CEX上对冲方向性风险
   - 持有稳定币备用金应对紧急平仓

5. 止损机制
   - 单笔最大亏损: 预期利润的2倍
   - 日最大亏损: 总资金的2%
```

---

## 四、技术架构

### 4.1 多链状态同步架构

```
跨链MEV bot的架构设计：

┌────────────────────────────────────────────────┐
│                 价格聚合层                       │
│                                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Arbitrum │ │ Optimism │ │   Base   │ ...   │
│  │ 价格源   │ │ 价格源   │ │ 价格源   │       │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘       │
│       │            │            │              │
│       ▼            ▼            ▼              │
│  ┌─────────────────────────────────────┐       │
│  │       统一价格聚合引擎              │       │
│  │  - 标准化价格格式                   │       │
│  │  - 时间戳对齐                       │       │
│  │  - 延迟补偿                         │       │
│  └───────────────┬─────────────────────┘       │
└──────────────────┼─────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────┐
│                 策略引擎层                       │
│                                                │
│  ┌─────────────────────────────────────┐       │
│  │       跨链套利机会检测              │       │
│  │  - L1↔L2价格差                     │       │
│  │  - L2↔L2价格差                     │       │
│  │  - 预言机延迟机会                   │       │
│  └───────────────┬─────────────────────┘       │
│                  │                             │
│  ┌─────────────────────────────────────┐       │
│  │       利润计算和风险评估             │       │
│  │  - 桥成本计算                       │       │
│  │  - 滑点预估                         │       │
│  │  - 执行风险评估                     │       │
│  └───────────────┬─────────────────────┘       │
└──────────────────┼─────────────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────────────┐
│                 执行引擎层                       │
│                                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Arb执行  │ │ OP执行   │ │ Base执行 │ ...   │
│  │ 引擎     │ │ 引擎     │ │ 引擎     │       │
│  └──────────┘ └──────────┘ └──────────┘       │
│                                                │
│  ┌─────────────────────────────────────┐       │
│  │       资金管理和Rebalance            │       │
│  └─────────────────────────────────────┘       │
└────────────────────────────────────────────────┘
```

### 4.2 多链RPC管理

```javascript
// 多链RPC连接管理
class MultiChainProvider {
    constructor() {
        this.providers = {
            ethereum: {
                primary: new ethers.JsonRpcProvider(
                    "https://eth-mainnet.g.alchemy.com/v2/KEY"
                ),
                fallback: new ethers.JsonRpcProvider(
                    "https://mainnet.infura.io/v3/KEY"
                ),
                ws: new ethers.WebSocketProvider(
                    "wss://eth-mainnet.g.alchemy.com/v2/KEY"
                ),
                blockTime: 12000,  // ms
                chainId: 1
            },
            arbitrum: {
                primary: new ethers.JsonRpcProvider(
                    "https://arb-mainnet.g.alchemy.com/v2/KEY"
                ),
                sequencerFeed: "wss://arb1.arbitrum.io/feed",
                blockTime: 250,
                chainId: 42161
            },
            base: {
                primary: new ethers.JsonRpcProvider(
                    "https://base-mainnet.g.alchemy.com/v2/KEY"
                ),
                blockTime: 2000,
                chainId: 8453
            },
            optimism: {
                primary: new ethers.JsonRpcProvider(
                    "https://opt-mainnet.g.alchemy.com/v2/KEY"
                ),
                blockTime: 2000,
                chainId: 10
            }
        };
    }

    async getPrice(chain, pool) {
        const provider = this.providers[chain].primary;
        const slot0 = await poolContract.connect(provider).slot0();
        return sqrtPriceX96ToPrice(slot0.sqrtPriceX96);
    }

    async getAllPrices(token0, token1) {
        return Promise.all(
            Object.keys(this.providers).map(async chain => ({
                chain,
                price: await this.getPrice(chain, getPool(chain, token0, token1)),
                timestamp: Date.now()
            }))
        );
    }
}
```

### 4.3 资金预部署与Rebalance

```
资金管理策略：

1. 初始部署：
   - 总资金按交易量比例分配到各L2
   - 典型分配: Arbitrum 30%, Base 25%, Optimism 20%, Ethereum 15%, Others 10%

2. 持续Rebalance：
   触发条件：
   - 某链资金 < 初始分配的50% → 需要补充
   - 某链资金 > 初始分配的150% → 需要转移
   - 跨链套利导致资金失衡

   Rebalance策略：
   - 使用最便宜的桥（通常Across Protocol）
   - 批量rebalance（每日1-2次，而非每笔交易后）
   - 在低gas时段执行

3. 资金效率优化：
   - 空闲资金存入各链的Aave/Compound获取收益
   - 使用资金利用率监控，确保有足够的流动资金
   - 设置紧急提取阈值
```

---

## 五、前沿研究

### 5.1 Shared Sequencer对跨链MEV的影响

```
Shared Sequencer（共享排序器）概念：

传统模式：
每条L2有自己的sequencer → 独立排序 → 跨链无法原子执行

Shared Sequencer模式：
多条L2共享一个排序器 → 可以原子性地排序跨链交易

代表项目：
- Espresso Systems: 通用Shared Sequencer
- Astria: 针对Rollup的共享排序层
- Radius: 加密排序+共享sequencer

对MEV的影响：
1. 跨L2套利可能变得原子化
   → 降低风险，但也增加竞争
2. Sequencer的MEV提取能力增强
   → 可以跨L2做三明治攻击
3. MEV拍卖可能扩展到跨链
   → Builder竞价将包含多链Bundle

时间线：
- 2025-2026年: 早期测试网
- 2027+: 主网部署（乐观估计）
```

### 5.2 Intent架构与跨链MEV

```
Intent架构（意图架构）：

传统交易: 用户指定精确的执行路径
Intent: 用户只声明想要的结果，由Solver执行

示例：
传统: "在Uniswap V3 ETH/USDC池用1 ETH swap USDC"
Intent: "我想用1 ETH获得至少2000 USDC"

跨链Intent:
"我想在Base上用1 ETH获得至少2000 USDC（当前在Arbitrum上）"

Intent + 跨链MEV：
1. Solver可以选择最优的执行路径（包括跨链）
2. 多个Solver竞争 → 用户获得更好的价格
3. Solver可以捕获跨链套利利润
4. MEV从"提取"转向"竞价分享"

代表项目：
- UniswapX: 意图订单 + Filler网络
- Across Protocol: 跨链意图
- deBridge: 跨链意图 + Solver网络
- Anoma/Namada: 通用Intent架构
```

### 5.3 SUAVE与跨链MEV

```
SUAVE (Single Unifying Auction for Value Expression):
Flashbots推出的MEV基础设施愿景。

SUAVE的跨链能力：
1. 统一的偏好（Preference）表达
   - 用户可以表达跨链的交易偏好
   - 例："在最便宜的链上买入ETH"

2. 跨链Bundle
   - Builder可以构建跨多条链的Bundle
   - 原子性由SUAVE链保证

3. 跨链MEV拍卖
   - Searcher在SUAVE上竞价
   - 赢得的MEV在源链执行

SUAVE TEE (Trusted Execution Environment):
- 使用SGX等可信执行环境
- Searcher可以提交策略而不暴露
- 防止Builder偷取Searcher的策略

当前状态：研究阶段，部分组件在测试网
```

---

## 六、跨链监控实战

### 6.1 监控架构设计

```
实时跨链价格监控系统：

数据采集层：
- 每条链一个独立的数据采集worker
- 使用WebSocket订阅新区块事件
- 解析区块中的Swap事件获取最新价格

数据存储：
- Redis: 最新价格缓存（<1ms读取）
- TimescaleDB: 历史价格数据（时序数据库）

告警系统：
- 价格偏差 > 阈值 → Telegram/Discord通知
- 可配置多级告警（信息级、操作级、紧急级）

// 简化的监控代码框架
class CrossChainMonitor {
    constructor(chains, pools, thresholds) {
        this.chains = chains;
        this.pools = pools;
        this.thresholds = thresholds;
        this.latestPrices = {};
    }

    async start() {
        for (const chain of this.chains) {
            this.subscribeToBlocks(chain);
        }
    }

    async onNewBlock(chain, blockNumber) {
        for (const pool of this.pools[chain]) {
            const price = await this.getPoolPrice(chain, pool);
            this.latestPrices[`${chain}:${pool.pair}`] = {
                price,
                block: blockNumber,
                timestamp: Date.now()
            };
        }
        this.checkArbitrageOpportunities();
    }

    checkArbitrageOpportunities() {
        for (const pair of this.uniquePairs) {
            const chainPrices = this.getPricesForPair(pair);
            const maxPrice = Math.max(...chainPrices.map(p => p.price));
            const minPrice = Math.min(...chainPrices.map(p => p.price));
            const spread = (maxPrice - minPrice) / minPrice;

            if (spread > this.thresholds[pair]) {
                this.alertArbitrageOpportunity({
                    pair,
                    spread,
                    buyChain: chainPrices.find(p => p.price === minPrice).chain,
                    sellChain: chainPrices.find(p => p.price === maxPrice).chain,
                    estimatedProfit: this.estimateProfit(pair, spread)
                });
            }
        }
    }
}
```

### 6.2 延迟测量与优化

```
跨链延迟测量方法：

1. 同一事件在不同链上的确认时间差
   - 在L1上执行一笔交易
   - 测量该交易在各L2上被感知的时间
   - 差值 = 跨链延迟

2. 价格传播延迟
   - 记录CEX价格变动时间戳
   - 记录各DEX价格追随的时间戳
   - 差值 = 价格传播延迟

典型延迟数据（2025年实测）：
- CEX → L1 DEX: 12-36秒（1-3个L1区块）
- CEX → Arbitrum DEX: 1-5秒
- CEX → Base DEX: 2-10秒
- L1 → Arbitrum（消息传递）: 5-15分钟
- Arbitrum → Base（通过L1桥）: 20-40分钟
- Arbitrum → Base（通过快速桥）: 1-5分钟

延迟优化：
1. 使用sequencer直连（减少1-5秒）
2. 地理位置优化（靠近sequencer的数据中心）
3. 使用专用mempool数据源
4. 预计算和缓存池状态
```

---

## 七、风险分析

### 7.1 跨链MEV特有风险

| 风险类型 | 描述 | 严重程度 | 缓解措施 |
|---------|------|---------|---------|
| **执行风险** | 一端成功另一端失败 | 高 | 限制头寸、对冲、止损 |
| **桥风险** | 桥被攻击或故障 | 高 | 使用原生桥、分散桥选择 |
| **Sequencer风险** | Sequencer宕机或审查 | 中 | 强制包含机制、多链分散 |
| **终局性风险** | L2交易被revert | 低（但影响大） | 等待足够确认数 |
| **资金锁定风险** | 资金在桥中被延迟 | 中 | 预部署资金、多桥策略 |
| **预言机风险** | 不同链上预言机价格不一致 | 中 | 多源验证、偏差检测 |

### 7.2 Sequencer宕机场景

```
中心化sequencer的宕机风险：

历史事件：
- 2023年6月: Arbitrum Sequencer宕机1小时+
- 多次短暂的sequencer延迟事件

对跨链MEV的影响：
1. Sequencer宕机期间无法在该L2上执行交易
2. 已开立的跨链头寸无法平仓
3. 价格偏差可能大幅扩大（无人套利）
4. Sequencer恢复后可能出现价格跳变

应对策略：
- 使用延迟包含机制（delayed inbox）绕过sequencer
  → Arbitrum允许用户直接向L1提交交易
  → 但延迟约10-15分钟
- 限制单链敞口
- 监控sequencer健康状态
- 在sequencer恢复后优先平仓
```

---

## 八、预期收益与市场规模

### 8.1 跨链MEV市场规模估算

```
2024-2025年跨链MEV市场估算：

L1→L2套利（主要是CEX-DEX跨L2）：
- 日均机会: 数百次
- 日均利润池: $50K-$200K
- 参与者: 10-50个活跃bot

L2↔L2套利：
- 日均机会: 数十次
- 日均利润池: $10K-$50K
- 参与者: <20个活跃bot

预言机延迟套利：
- 频率低但单笔利润高
- 日均利润池: $20K-$100K
- 主要在高波动期出现

总市场规模: ~$30M-$100M/年
（远小于L1 MEV市场的~$500M-$1B/年）

增长趋势：
- L2 TVL增长 → 跨链MEV增长
- 新L2上线 → 新的套利对出现
- Shared Sequencer → 可能改变整个格局
```

---

## 九、工具速查表

| 用途 | 工具 | 说明 |
|------|------|------|
| 多链RPC | Alchemy / QuickNode / Infura | 多链节点服务 |
| 价格数据 | Chainlink / Pyth Network | 跨链预言机 |
| 桥比较 | Bungee / Li.Fi | 桥聚合器，比较延迟和成本 |
| L2数据 | L2Beat | L2 TVL、交易量、状态 |
| 跨链分析 | Dune Analytics | 跨链数据查询 |
| Sequencer状态 | Arbitrum/OP Status Pages | 实时状态监控 |
| 桥监控 | Across Protocol Dashboard | 桥的利用率和费用 |
| DEX数据 | DeFiLlama / DEX Screener | 多链DEX价格和流动性 |

---

## 诚实边界

- **跨链MEV是高风险领域**：缺乏原子性保证意味着每笔交易都有亏损可能
- **市场规模较小**：相比L1 MEV，跨链MEV的利润池目前还很小
- **基础设施要求极高**：需要在多条链上同时部署和维护bot
- **技术快速变化**：Shared Sequencer、Intent架构等新范式可能颠覆现有策略
- **数据不确定性**：跨链MEV的精确数据很难获取，本文档中的估算有较大误差范围
- **中心化sequencer风险**：目前所有主流L2都依赖中心化sequencer，这是系统性风险

---

## 关联Skills

- [l2-sequencer-feed](../l2-sequencer-feed/) — L2 Sequencer数据源
- [backrun-arbitrage](../backrun-arbitrage/) — Backrun套利策略基础
- [low-latency-infrastructure](../low-latency-infrastructure/) — 低延迟基础设施

---

## 附录：调研来源

### 核心参考
- L2Beat: https://l2beat.com/
- Flashbots SUAVE文档: https://writings.flashbots.net/
- Espresso Systems Shared Sequencer: https://docs.espressosys.com/
- Arbitrum技术文档: https://docs.arbitrum.io/
- Optimism技术文档: https://docs.optimism.io/
- "Multi-block MEV" (Flashbots研究)
- "Time, clocks, and the ordering of events in a distributed system" (Lamport)
- Across Protocol文档: https://docs.across.to/

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
