---
name: sandwich-attack-mechanics
description: |
  MEV三明治攻击技术分析师。涵盖攻击原理（frontrun→target→backrun流程、利润模型、最优参数）、
  目标识别（滑点检测、金额/深度比、Router交易解析）、攻击交易构造（最优frontrun金额、gas竞价、
  bundle原子性、多池路由）、防御机制（MEV Blocker、Flashbots Protect、私有池、最优滑点设置）、
  L2三明治攻击（中心化sequencer、Base/Optimism、Arbitrum FCFS）、链上分析（历史检测、利润统计、
  主要bot地址）。包含数学利润公式推导和Dune SQL检测查询。
  用于安全研究与防御目的。
  当用户问「三明治攻击怎么工作」「如何防御sandwich attack」「MEV三明治利润分析」时使用。
---

# MEV 三明治攻击技术分析

> 三明治攻击是MEV中最常见的掠夺性策略——理解其机制是防御的第一步。本文档用于安全研究与防御目的。

## 使用说明

**本Skill用于MEV安全研究与防御。** 当用户询问三明治攻击的原理、检测或防御时，按对应章节提供系统性分析。

---

## 一、三明治攻击基本原理

### 1.1 攻击流程

三明治攻击由三笔交易组成，打包在同一个区块中，顺序严格保证：

```
交易1 (Frontrun):  攻击者买入目标代币 → 推高价格
交易2 (Victim):    受害者按照更差价格执行swap → 进一步推高价格
交易3 (Backrun):   攻击者卖出代币 → 在更高价格获利
```

### 1.2 核心前提条件

| 条件 | 说明 |
|------|------|
| **Mempool可见性** | 攻击者必须能看到受害者的pending交易 |
| **交易排序控制** | 通过gas竞价或Bundle提交控制三笔交易的顺序 |
| **原子性保证** | 三笔交易必须在同一区块，且frontrun在victim之前，backrun在victim之后 |
| **足够的滑点容忍** | 受害者设置的slippage tolerance必须足够大以容纳攻击 |

### 1.3 AMM价格影响机制

以Uniswap V2恒定乘积公式为例：

```
x * y = k

价格 P = y / x （其中x为目标代币储备，y为报价代币储备）

买入Δx后的新价格：
P_new = (y - Δy) / (x + Δx)
     = y / (x + Δx) * x / x  （近似）

价格影响 = Δx / x （对于小额交易的近似）
```

---

## 二、利润模型数学推导

### 2.1 Uniswap V2 三明治利润公式

设定初始池状态 `(x, y)`，恒定乘积 `k = x * y`，交易手续费率 `f = 0.3%`（即乘以 `γ = 1 - f = 0.997`）。

**Step 1: 攻击者Frontrun买入**

攻击者投入 `Δ_a` 数量的token Y（报价代币），买入token X（目标代币）。

```
收到的 token X 数量：
dx_a = (x * γ * Δ_a) / (y + γ * Δ_a)

新池状态：
x' = x - dx_a
y' = y + Δ_a
```

**Step 2: 受害者交易执行**

受害者投入 `Δ_v` 数量的token Y：

```
收到的 token X 数量：
dx_v = (x' * γ * Δ_v) / (y' + γ * Δ_v)

新池状态：
x'' = x' - dx_v
y'' = y' + Δ_v
```

**Step 3: 攻击者Backrun卖出**

攻击者卖出 `dx_a` 数量的token X：

```
收到的 token Y 数量：
dy_a = (y'' * γ * dx_a) / (x'' + γ * dx_a)
```

**攻击者利润：**

```
Profit = dy_a - Δ_a - gas_cost

其中 gas_cost 包括：
- frontrun交易的gas费
- backrun交易的gas费
- bundle提交的tip（给builder的小费）
```

### 2.2 最优Frontrun金额

对利润公式求导并令 `dProfit/dΔ_a = 0`，可得最优frontrun金额：

```
对于小额交易的近似解：
Δ_a_optimal ≈ sqrt(Δ_v * y * γ) - y

精确解需要求解三次方程，实际bot使用二分搜索或数值优化：

function findOptimalAmount(reserve_x, reserve_y, victim_amount, fee):
    low = 0
    high = victim_amount * 10  // 搜索上界
    while high - low > precision:
        mid = (low + high) / 2
        profit_mid = simulateProfit(reserve_x, reserve_y, mid, victim_amount, fee)
        profit_mid_plus = simulateProfit(reserve_x, reserve_y, mid + epsilon, victim_amount, fee)
        if profit_mid_plus > profit_mid:
            low = mid
        else:
            high = mid
    return low
```

### 2.3 利润约束条件

```
约束1: Profit > gas_cost （净利润为正）
约束2: victim_slippage_tolerance > price_impact(Δ_a) （受害者交易不能revert）
约束3: Δ_a < available_capital （资金约束）
约束4: 总gas < block_gas_limit （区块gas限制）
```

### 2.4 受害者滑点与攻击者利润的关系

```
受害者可被榨取的最大价值 ≈ slippage_tolerance * Δ_v

例：受害者swap $10,000，设置5%滑点
→ 攻击者最大可提取约 $500（扣除gas后的上限）

实际提取率通常为滑点容忍的30-70%，取决于：
- 池深度
- gas成本
- 竞争者数量
```

---

## 三、目标识别与筛选

### 3.1 Mempool监听与交易解析

攻击者需要识别以下类型的交易：

```
目标Router合约：
- Uniswap V2 Router: swapExactTokensForTokens, swapTokensForExactTokens
- Uniswap V3 Router: exactInputSingle, exactInput
- Uniswap Universal Router: execute (command 0x00, 0x01)
- SushiSwap Router: 类似V2接口
- 1inch Router: swap, unoswap
- Paraswap: multiSwap, megaSwap

解析关键参数：
- amountIn: 输入金额
- amountOutMin: 最小输出金额（反推slippage tolerance）
- path: swap路径（确定目标池）
- deadline: 交易截止时间
```

### 3.2 目标筛选条件

| 筛选维度 | 条件 | 原因 |
|---------|------|------|
| **金额/深度比** | `amountIn / pool_reserve > 0.1%` | 太小的交易利润不足以覆盖gas |
| **滑点容忍** | `slippage > 0.5%` | 滑点太低无法插入frontrun |
| **池流动性** | 中等流动性池 | 深池利润低，浅池可能导致交易失败 |
| **gas价格** | 当前gas不能过高 | gas成本吞噬利润 |
| **代币类型** | 排除fee-on-transfer代币 | 这类代币的利润计算复杂且不可靠 |
| **Router类型** | 已知的DEX Router | 未知合约可能有反三明治逻辑 |

### 3.3 利润预估流程

```
1. 解析victim交易 → 获取 (tokenIn, tokenOut, amountIn, amountOutMin, path)
2. 查询链上池状态 → 获取 (reserve0, reserve1, fee)
3. 计算victim的滑点容忍 = (expected_output - amountOutMin) / expected_output
4. 计算最优frontrun金额 Δ_a
5. 模拟三笔交易序列 → 计算净利润
6. 利润 > min_threshold → 提交Bundle
```

---

## 四、攻击交易构造

### 4.1 Bundle构造

```javascript
// Flashbots Bundle结构
const bundle = {
  txs: [
    frontrunTx,    // 攻击者买入
    victimTxHash,  // 受害者交易（通过hash引用pending tx）
    backrunTx      // 攻击者卖出
  ],
  blockNumber: targetBlock,
  minTimestamp: undefined,
  maxTimestamp: undefined
};

// 通过 eth_sendBundle 提交给Builder
```

### 4.2 Gas竞价策略

```
三明治攻击的gas策略：
- Frontrun tx: 必须排在victim之前 → 需要更高的priority fee
- Backrun tx: 必须紧跟victim → priority fee可以较低

竞价模型：
max_gas_bid = expected_profit * bid_percentage
bid_percentage 通常为 60-90%（视竞争程度而定）

EIP-1559下的gas参数：
- maxFeePerGas: 当前 baseFee * 2 + maxPriorityFeePerGas
- maxPriorityFeePerGas: 根据利润比例动态调整
```

### 4.3 多池路由三明治

当受害者的swap经过多个池时：

```
路径: A → B → C（经过池AB和池BC）

策略选择：
1. 单池三明治：只在利润最高的单个池上做三明治
2. 多池三明治：在多个池上同时做三明治（更复杂，需要更多资金）

多池利润计算：
Profit_total = Σ Profit_i - Σ gas_i
需要确保每个池的操作不互相干扰
```

### 4.4 Uniswap V3三明治的特殊考量

```
V3 集中流动性的影响：
- 价格影响不再是简单的恒定乘积公式
- 需要模拟tick-by-tick的价格变动
- 当价格跨越tick边界时，liquidity会变化

V3三明治难度更高：
1. 需要精确模拟SqrtPriceX96的变化
2. 跨tick的计算复杂度显著增加
3. 利润预估准确度要求更高（否则交易可能revert）

// V3价格计算核心
sqrtPriceX96_after = computeSwapStep(
    sqrtPriceX96_current,
    sqrtPriceX96_target,  // 当前tick的边界价格
    liquidity,
    amountRemaining,
    fee
)
```

---

## 五、防御机制

### 5.1 用户层面防御

| 防御措施 | 原理 | 效果 | 缺点 |
|---------|------|------|------|
| **降低滑点设置** | 减小攻击者的利润空间 | 高 | 可能导致交易失败率增加 |
| **使用Flashbots Protect RPC** | 交易直接发给builder，不进公共mempool | 非常高 | 交易确认可能稍慢 |
| **使用MEV Blocker** | 由Cowswap团队开发，backrun利润返还用户 | 高 | 依赖builder合作 |
| **私有交易池** | 各种private RPC endpoint | 高 | 信任假设转移给RPC提供者 |
| **小额分批交易** | 减少单笔交易的金额/深度比 | 中 | 增加gas成本和时间 |
| **使用限价单DEX** | CoW Protocol/1inch Fusion等 | 高 | 执行延迟较大 |

### 5.2 最优滑点设置指南

```
推荐滑点设置策略：

稳定币对 (USDC/USDT): 0.1% - 0.3%
主流币对 (ETH/USDC):   0.3% - 1.0%
小币对 (低流动性):      1.0% - 3.0%

原则：
- 滑点 = 预期价格影响 + 安全余量
- 预期价格影响 = amountIn / pool_liquidity * fee_adjustment
- 安全余量 = 0.1% - 0.5%（考虑区块间价格波动）

永远不要设置 >5% 的滑点（除非你清楚知道为什么需要）
```

### 5.3 Flashbots Protect详解

```
使用方式：将RPC endpoint切换为 Flashbots Protect

MetaMask设置：
- 网络名称: Flashbots Protect
- RPC URL: https://rpc.flashbots.net
- Chain ID: 1
- 货币符号: ETH
- 区块浏览器: https://etherscan.io

工作原理：
1. 用户交易发送到Flashbots Protect RPC
2. 交易不进入公共mempool
3. Flashbots将交易直接发给合作builder
4. Builder将交易打包进区块
5. 如果交易在25个区块内未被包含，自动取消

高级选项（hints参数）：
- fast: 发送给所有builder（更快但隐私性降低）
- none: 只发送给Flashbots builder（最高隐私）
```

### 5.4 MEV Blocker (by CoW Protocol)

```
原理：
1. 用户交易发送到MEV Blocker RPC
2. Searcher竞价获取backrun权（不允许frontrun和sandwich）
3. Backrun产生的利润的90%返还给用户
4. 用户既免受三明治攻击，又能获得backrun分成

RPC URL: https://rpc.mevblocker.io

统计数据（截至2025年）：
- 已为用户节省超过 $3亿 MEV费用
- 日均处理 >50,000 笔交易
- 与多个主要builder合作
```

### 5.5 协议层面防御

| 防御机制 | 实现方 | 原理 |
|---------|-------|------|
| **TWAP Oracle** | Uniswap V3 | 时间加权平均价格，减少单区块操纵影响 |
| **Commit-Reveal** | 部分DEX | 先提交加密的交易意图，后揭示执行 |
| **Batch Auction** | CoW Protocol | 所有交易批量匹配，无ordering优势 |
| **Encrypted Mempool** | Shutter Network | 交易加密直到被打包 |
| **Uniswap V4 Hooks** | Uniswap V4 | 可自定义anti-MEV逻辑（如验证交易来源） |

---

## 六、L2上的三明治攻击

### 6.1 L2 Sequencer与MEV

| L2 | Sequencer模式 | 三明治可能性 | 说明 |
|----|-------------|------------|------|
| **Arbitrum** | 中心化FCFS | 低 | 先到先处理，难以插队；但sequencer本身有排序权 |
| **Optimism/Base** | 中心化Priority Fee排序 | 中 | 类似L1的gas竞价，但sequencer可以选择不配合 |
| **zkSync Era** | 中心化 | 低 | 自定义排序逻辑，不完全按gas价格 |
| **Polygon zkEVM** | 中心化 | 低-中 | 取决于sequencer实现 |

### 6.2 Arbitrum的FCFS模型

```
Arbitrum的First-Come-First-Served排序：
- 交易按到达sequencer的时间排序
- 没有priority fee竞价
- 三明治攻击者需要：
  1. 比victim更早提交frontrun → 需要更低延迟
  2. 在victim之后立即提交backrun → 需要精确时间控制

实际上Arbitrum上仍有MEV：
- Sequencer可以看到所有pending交易
- 低延迟连接到sequencer的节点有优势
- "延迟游戏"替代了"gas竞价游戏"
```

### 6.3 Base/Optimism的Priority Fee排序

```
OP Stack L2（Base, Optimism等）：
- 使用类似L1的EIP-1559 gas模型
- Sequencer按priority fee排序交易
- 理论上可以通过gas竞价实现三明治

但实际限制：
1. Sequencer由单一实体运营（Coinbase for Base, Optimism Foundation for OP）
2. Sequencer有声誉激励不配合MEV
3. 区块时间2秒，竞争窗口更短
4. 目前主要builder不积极在L2上做三明治（利润太低）

趋势：
- 随着L2 TVL增长，L2 MEV将成为新战场
- Shared Sequencer方案可能改变格局
```

---

## 七、链上检测与分析

### 7.1 三明治攻击检测逻辑

```
检测算法：
1. 在同一区块中，找到三笔交易满足：
   - tx1和tx3的sender相同（或属于同一合约/EOA组合）
   - tx2的sender不同（受害者）
   - tx1在tx2之前，tx3在tx2之后（按tx_index排序）
   - tx1是买入操作，tx3是卖出操作
   - tx1、tx2、tx3操作同一个交易对

2. 验证：
   - tx3的卖出金额 ≈ tx1的买入金额（同一代币）
   - tx3获得的报价代币 > tx1投入的报价代币（正利润）
```

### 7.2 Dune SQL检测查询

```sql
-- 三明治攻击检测：Uniswap V2/V3
-- 检测同一区块中的frontrun-victim-backrun模式

WITH swap_events AS (
    SELECT
        block_number,
        block_time,
        tx_hash,
        tx_index,
        tx_from,
        contract_address AS pool_address,
        -- Uniswap V2 Swap event解析
        CASE
            WHEN topic0 = 0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822
            THEN 'uniswap_v2'
            ELSE 'uniswap_v3'
        END AS dex_type,
        -- 提取swap方向和金额
        varbinary_to_uint256(substr(data, 1, 32)) AS amount0In,
        varbinary_to_uint256(substr(data, 33, 32)) AS amount1In,
        varbinary_to_uint256(substr(data, 65, 32)) AS amount0Out,
        varbinary_to_uint256(substr(data, 97, 32)) AS amount1Out
    FROM ethereum.logs
    WHERE block_time >= NOW() - INTERVAL '7' DAY
        AND topic0 IN (
            0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822,  -- V2 Swap
            0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67   -- V3 Swap
        )
),

potential_sandwiches AS (
    SELECT
        s1.block_number,
        s1.block_time,
        s1.tx_hash AS frontrun_tx,
        s2.tx_hash AS victim_tx,
        s3.tx_hash AS backrun_tx,
        s1.tx_from AS attacker,
        s2.tx_from AS victim,
        s1.pool_address,
        s1.tx_index AS frontrun_index,
        s2.tx_index AS victim_index,
        s3.tx_index AS backrun_index,
        -- 利润计算（简化：token1进出差额）
        CAST(s3.amount1Out AS double) - CAST(s1.amount1In AS double) AS raw_profit_token1
    FROM swap_events s1
    JOIN swap_events s2
        ON s1.block_number = s2.block_number
        AND s1.pool_address = s2.pool_address
        AND s1.tx_index < s2.tx_index
        AND s1.tx_from != s2.tx_from
    JOIN swap_events s3
        ON s2.block_number = s3.block_number
        AND s2.pool_address = s3.pool_address
        AND s2.tx_index < s3.tx_index
        AND s1.tx_from = s3.tx_from  -- 同一攻击者
    WHERE s1.amount0In > 0 AND s1.amount1Out > 0   -- frontrun: 买入token0
        AND s3.amount0Out > 0 AND s3.amount1In > 0   -- backrun: 卖出token0（反向）
        -- 确保不是正常交易
        AND s3.tx_index - s1.tx_index <= 5  -- 间隔不超过5笔交易
)

SELECT
    block_number,
    block_time,
    attacker,
    victim,
    pool_address,
    frontrun_tx,
    victim_tx,
    backrun_tx,
    raw_profit_token1 / 1e18 AS profit_eth_approx,
    frontrun_index,
    victim_index,
    backrun_index
FROM potential_sandwiches
WHERE raw_profit_token1 > 0
ORDER BY block_time DESC
LIMIT 100
```

### 7.3 主要三明治Bot统计

```sql
-- 统计主要三明治攻击bot的利润
-- 已知主要bot合约地址（截至2025年数据）

WITH known_sandwich_bots AS (
    SELECT address, label FROM (VALUES
        (0x6b75d8AF000000e20B7a7DDf000Ba900b4009A80, 'jaredfromsubway.eth'),
        (0xae2Fc483527B8EF99EB5D9B44875F005ba1FaE13, 'sandwich_bot_2'),
        (0x00000000003b3cc22aF3aE1EAc0440BcEe416B40, 'bloxroute_bot')
        -- 更多bot地址可通过链上检测发现
    ) AS t(address, label)
)

SELECT
    b.label,
    COUNT(DISTINCT ps.block_number) AS sandwich_count,
    SUM(ps.raw_profit_token1) / 1e18 AS total_profit_eth,
    AVG(ps.raw_profit_token1) / 1e18 AS avg_profit_eth,
    MIN(ps.block_time) AS first_seen,
    MAX(ps.block_time) AS last_seen
FROM potential_sandwiches ps
JOIN known_sandwich_bots b ON ps.attacker = b.address
GROUP BY b.label
ORDER BY total_profit_eth DESC
```

### 7.4 历史数据与趋势

```
三明治攻击规模（以太坊主网，2024-2025年数据）：

2024年统计：
- 日均三明治攻击数量: ~15,000-30,000次
- 日均受害者损失: $1M-$5M
- 主要攻击者（jaredfromsubway.eth）累计利润: >$50M

趋势变化：
- 2023年是三明治攻击的高峰期
- 2024年后随着Flashbots Protect和MEV Blocker普及，攻击量略降
- L2上的三明治攻击开始出现但规模较小
- 攻击者利润率压缩（竞争加剧，防御工具普及）

jaredfromsubway.eth分析：
- 最著名的三明治bot
- 合约地址: 0x6b75d8AF000000e20B7a7DDf000Ba900b4009A80
- 巅峰期消耗以太坊总gas的5-8%
- 使用复杂的多层代理合约结构
```

---

## 八、高级主题

### 8.1 多区块三明治（Cross-block Sandwich）

```
传统三明治：三笔交易在同一区块
多区块三明治：frontrun和backrun可能跨区块

场景：
- Block N: 攻击者买入（frontrun）
- Block N: 受害者交易执行
- Block N+1: 攻击者卖出（backrun）

风险：跨区块意味着失去原子性保证
- 在Block N和N+1之间，其他交易可能改变池状态
- 攻击者承担库存风险

使用场景：
- 当单区块内无法完成套利时
- 大额交易需要分散在多个区块时
```

### 8.2 反三明治策略（Anti-Sandwich）

```
合约层面的反三明治：
1. 检查同区块是否有来自已知bot的交易
2. 使用 block.timestamp 和 tx.origin 进行验证
3. 在回调函数中检查价格偏移

// Solidity 示例：反三明治检查
contract AntiSandwich {
    mapping(address => uint256) lastSwapBlock;

    modifier antiSandwich() {
        require(
            lastSwapBlock[tx.origin] != block.number,
            "Same block swap not allowed"
        );
        lastSwapBlock[tx.origin] = block.number;
        _;
    }
}

注意：这种简单方法可被绕过（攻击者使用不同EOA）
更复杂的方案需要结合oracle检查价格偏移
```

### 8.3 MEV-Share与三明治攻击

```
MEV-Share（Flashbots推出）改变了三明治攻击的经济模型：

传统模式：
- 攻击者获取100%利润（减去gas和builder tip）
- 受害者损失100%

MEV-Share模式：
- 用户通过MEV-Share提交交易
- 用户可以选择性披露交易信息（hints）
- Searcher只能看到部分信息（如交易涉及的合约，但看不到金额）
- Searcher提交Bundle竞价
- 利润按预设比例分配给用户（通常>90%）

影响：
- 三明治攻击利润被大幅压缩
- Backrun仍然可行（对用户无害）
- Frontrun利润需要与用户分享
```

---

## 九、实战分析模板

### 分析一笔疑似三明治攻击

```
Step 1: 识别交易组
  → 在区块浏览器中查看同一区块的相邻交易
  → 检查是否存在 buy-swap-sell 模式

Step 2: 验证攻击者身份
  → frontrun和backrun的sender/合约是否相同
  → 检查该地址的历史交易模式

Step 3: 计算利润
  → 攻击者投入的token数量（frontrun）
  → 攻击者收回的token数量（backrun）
  → 差额 = 毛利润
  → 毛利润 - gas费 = 净利润

Step 4: 评估受害者损失
  → 受害者实际收到的token数量
  → 如果没有三明治攻击，受害者应该收到的数量
  → 差额 = 受害者损失（≈ 攻击者利润 + AMM费用）

Step 5: 输出报告
  → 攻击者地址
  → 受害者地址
  → 攻击池
  → 净利润
  → 受害者损失
  → 使用的gas
```

---

## 十、工具速查表

| 用途 | 工具 | 说明 |
|------|------|------|
| Mempool监听 | Bloxroute / Chainbound Fiber | 低延迟mempool数据流 |
| 交易模拟 | Tenderly / Alchemy Simulate | 模拟Bundle执行结果 |
| Bundle提交 | Flashbots / MEV-Share | 向Builder提交Bundle |
| 链上检测 | Dune Analytics | SQL查询历史三明治数据 |
| 实时监控 | EigenPhi | MEV实时监控仪表盘 |
| Bot识别 | Arkham / Nansen | 标记已知MEV bot地址 |
| 防御工具 | Flashbots Protect / MEV Blocker | 用户端防护RPC |
| 价格模拟 | Uniswap SDK | 精确模拟swap价格影响 |

---

## 诚实边界

- **本文档用于安全研究与防御目的**：理解攻击机制是构建防御的前提
- **利润公式是简化模型**：实际利润受gas波动、竞争、MEV-Share分成等因素影响
- **链上检测不完美**：部分三明治攻击使用复杂合约结构，难以通过简单模式匹配检测
- **防御不是绝对的**：Flashbots Protect等工具显著降低但不能完全消除三明治风险
- **L2 MEV是快速演变的领域**：随着L2架构变化，三明治攻击的可行性也在变化
- **数据时效性**：bot地址、利润数据、市场份额等统计会快速过时，需要定期更新

---

## 关联Skills

- [mempool-monitoring](../mempool-monitoring/) — Mempool监听基础设施
- [mev-bundle-construction](../mev-bundle-construction/) — Bundle构造与提交
- [backrun-arbitrage](../backrun-arbitrage/) — Backrun套利策略（三明治的backrun部分）

---

## 附录：调研来源

### 核心参考
- Flashbots官方文档: https://docs.flashbots.net/
- MEV-Explore: https://explore.flashbots.net/
- EigenPhi MEV数据: https://eigenphi.io/
- Dune Analytics MEV Dashboard
- "Flash Boys 2.0" 论文 (Daian et al., 2019)
- Paradigm MEV研究系列
- Uniswap V2/V3白皮书

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
