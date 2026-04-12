---
name: jit-liquidity-attack
description: |
  JIT（Just-In-Time）流动性策略分析师。Uniswap V3集中流动性特有的MEV形式。
  涵盖JIT原理（在大额swap前添加窄范围流动性、捕获手续费、之后移除）、
  交易构造（从mempool识别"值得JIT"的swap、最优tick范围、流动性数量、
  bundle: mint→[target swap]→burn）、对被动LP的影响（手续费稀释量化）、
  检测与防御（链上检测、时间加权流动性、V4 Hook可能性）、
  市场数据（主要JIT bot、活动占比、利润趋势）。
  包含tick范围优化数学和Dune SQL JIT检测查询。
  当用户问「JIT流动性是什么」「Uniswap V3 MEV」「LP被稀释」时使用。
---

# JIT (Just-In-Time) 流动性策略分析

> JIT流动性是Uniswap V3引入集中流动性后出现的独特MEV形式——通过在大额swap前精准添加流动性来捕获手续费。

## 使用说明

**本Skill用于分析JIT流动性策略的机制、影响和防御。** 当用户询问JIT MEV、Uniswap V3 LP被稀释、集中流动性MEV时，按对应章节提供分析。

---

## 一、JIT流动性基本原理

### 1.1 什么是JIT流动性

```
JIT流动性的完整流程：

1. JIT bot监听mempool中的大额swap交易
2. 识别出该swap将经过哪个Uniswap V3池、哪个价格范围
3. 在该swap之前，bot向池中添加一笔极窄范围的集中流动性
   → 这笔流动性只覆盖几个tick，但金额很大
4. 大额swap执行，产生手续费
   → 由于JIT流动性占该tick范围的绝大部分，bot获得大部分手续费
5. swap执行后，bot立即移除所有流动性
   → 取回本金 + 赚取的手续费

Bundle结构：
TX1 (JIT bot):  mint() → 添加窄范围集中流动性
TX2 (Victim):   swap() → 大额交易执行，手续费分配给LP
TX3 (JIT bot):  burn() + collect() → 移除流动性，收取手续费
```

### 1.2 为什么JIT只在Uniswap V3上可行

```
Uniswap V2 vs V3的关键区别：

V2（均匀流动性）：
- 流动性均匀分布在0到∞的价格范围
- 无法选择特定价格范围
- 手续费按流动性份额均匀分配
- JIT不可行：你需要提供巨额流动性才能获得显著份额

V3（集中流动性）：
- LP可以选择在特定价格范围（tick range）提供流动性
- 范围越窄，资本效率越高
- 手续费只在活跃tick范围内分配
- JIT可行：在极窄范围提供流动性 → 极高的资本效率
         → 用较少资金占据该tick范围的大部分份额
         → 获取大部分手续费
```

### 1.3 JIT vs 三明治攻击的区别

| 维度 | JIT流动性 | 三明治攻击 |
|------|----------|-----------|
| **机制** | 添加/移除流动性 | 买入/卖出代币 |
| **对用户影响** | 间接（减少常驻LP收入） | 直接（用户获得更差价格） |
| **受害者** | 被动LP | swap用户 |
| **伦理评价** | 灰色地带 | 明确掠夺性 |
| **利润来源** | 手续费收入 | 价格操纵收入 |
| **资金需求** | 较大（需要提供流动性） | 较小 |
| **风险** | 无常损失（极小，因为只持续1个区块） | 几乎无风险（原子性） |

---

## 二、Uniswap V3 Tick数学基础

### 2.1 Tick与价格的关系

```
Uniswap V3中，价格以tick索引表示：

price(i) = 1.0001^i

其中 i 是tick索引（整数）。

相邻tick之间的价格变化 ≈ 0.01%（1个basis point）。

tick spacing（不同费率池）：
- 0.01% fee pool: tick spacing = 1
- 0.05% fee pool: tick spacing = 10
- 0.30% fee pool: tick spacing = 60
- 1.00% fee pool: tick spacing = 200

LP只能在 tick spacing 的整数倍位置设置范围边界。
```

### 2.2 集中流动性的资本效率

```
资本效率公式：

设当前价格 P_c，LP设置的价格范围为 [P_a, P_b]：

资本效率倍数 = sqrt(P_c) / (sqrt(P_c) - sqrt(P_a))
              ≈ 1 / (1 - sqrt(P_a/P_c))

示例：
- 全范围LP（V2等价）: 1x 资本效率
- ±10% 范围: ~10x 资本效率
- ±1% 范围: ~100x 资本效率
- ±0.1% 范围: ~1000x 资本效率

JIT的策略：
- 使用极窄范围（±0.01% 到 ±0.1%）
- 资本效率 100x-10000x
- 用 $100K 流动性就能在该tick范围占据 >90% 的份额
```

### 2.3 JIT最优Tick范围计算

```
JIT bot需要确定最优的tick范围 [tick_lower, tick_upper]：

输入：
- current_tick: 当前价格对应的tick
- swap_amount: 受害者swap的金额
- swap_direction: 买入还是卖出
- pool_liquidity: 当前tick的流动性总量
- tick_spacing: 池的tick间距

计算swap将移动到的tick：
// 简化版（单tick范围内）
price_impact = swap_amount / pool_liquidity
tick_after = current_tick + price_impact_to_ticks(price_impact, swap_direction)

最优tick范围：
if swap_direction == BUY_TOKEN0:  // 价格上升
    tick_lower = current_tick  // 对齐到tick_spacing
    tick_upper = tick_after    // 覆盖swap的整个范围
else:  // 价格下降
    tick_lower = tick_after
    tick_upper = current_tick

// 对齐到tick_spacing
tick_lower = floor(tick_lower / tick_spacing) * tick_spacing
tick_upper = ceil(tick_upper / tick_spacing) * tick_spacing

// 确保至少覆盖1个tick spacing
if tick_upper == tick_lower:
    tick_upper += tick_spacing
```

### 2.4 手续费收入计算

```
JIT bot的手续费收入：

fee_earned = swap_amount * fee_rate * (jit_liquidity / total_liquidity_in_range)

其中：
- swap_amount: 受害者swap的金额
- fee_rate: 池的手续费率（0.01%, 0.05%, 0.3%, 1%）
- jit_liquidity: JIT bot提供的流动性
- total_liquidity_in_range: 该tick范围内的总流动性
  = existing_liquidity + jit_liquidity

JIT利润：
profit = fee_earned - impermanent_loss - gas_cost

由于JIT只持续1个区块（~12秒），无常损失极小：
IL ≈ (price_change)^2 / 8 * liquidity_value
对于0.1%的价格变化: IL ≈ 0.0000125 ≈ 可忽略

实际利润主要取决于：
profit ≈ swap_amount * fee_rate * share - gas_cost
```

---

## 三、JIT交易构造

### 3.1 识别"值得JIT"的交易

```
JIT筛选标准：

1. 手续费收入阈值：
   min_fee = swap_amount * fee_rate * expected_share
   min_fee > gas_cost * safety_margin  （通常 safety_margin = 2x）

2. 金额阈值（经验值）：
   - 0.3% fee pool: swap_amount > $50,000
   - 0.05% fee pool: swap_amount > $200,000
   - 0.01% fee pool: swap_amount > $1,000,000

3. 池的现有流动性：
   - 现有流动性太高 → 需要太多JIT资金才能占据显著份额
   - 现有流动性太低 → 可能是小池，不值得

4. 排除条件：
   - 已知的MEV保护交易（通过Flashbots Protect提交）
   - 多跳swap（需要精确预测路径）
   - 异常代币（rebase、fee-on-transfer等）
```

### 3.2 Bundle构造详解

```
JIT Bundle的三笔交易：

TX1: Mint Position（添加流动性）
  合约: NonfungiblePositionManager
  函数: mint(MintParams)
  参数:
    - token0, token1: 池的两个代币
    - fee: 费率等级
    - tickLower, tickUpper: 计算得到的最优范围
    - amount0Desired, amount1Desired: 提供的代币数量
    - amount0Min, amount1Min: 最小数量（防止被抢先）
    - recipient: bot地址
    - deadline: block.timestamp

TX2: 受害者的Swap交易
  通过hash引用pending交易

TX3: Burn + Collect（移除流动性并收取手续费）
  操作1: burn(tokenId, liquidity)
  操作2: collect(tokenId, recipient, amount0Max, amount1Max)

  注意：需要先获取TX1 mint的tokenId
  策略：使用合约预计算tokenId，或使用回调模式

// 优化：使用自定义合约将mint和burn整合
contract JITLiquidity {
    function mintAndScheduleBurn(
        INonfungiblePositionManager.MintParams calldata params
    ) external returns (uint256 tokenId) {
        // mint流动性
        (tokenId,,,) = positionManager.mint(params);
        // 记录tokenId供burn使用
        pendingBurns[msg.sender] = tokenId;
    }

    function burnAndCollect() external {
        uint256 tokenId = pendingBurns[msg.sender];
        // 获取position信息
        (,,,,,,, uint128 liquidity,,,,) = positionManager.positions(tokenId);
        // burn
        positionManager.decreaseLiquidity(
            INonfungiblePositionManager.DecreaseLiquidityParams({
                tokenId: tokenId,
                liquidity: liquidity,
                amount0Min: 0,
                amount1Min: 0,
                deadline: block.timestamp
            })
        );
        // collect
        positionManager.collect(
            INonfungiblePositionManager.CollectParams({
                tokenId: tokenId,
                recipient: msg.sender,
                amount0Max: type(uint128).max,
                amount1Max: type(uint128).max
            })
        );
    }
}
```

### 3.3 资金管理

```
JIT流动性的资金需求：

假设要在tick范围 [tick_a, tick_b] 上占据 90% 的份额：
required_liquidity = existing_liquidity * 9  （因为 9/(9+1) = 90%）

对于典型的ETH/USDC 0.3% 池：
- 活跃tick范围的现有流动性: ~$5M-$50M（取决于价格范围宽度）
- 窄范围（±0.1%）的现有流动性: ~$100K-$2M

JIT需要的资金：
- 低竞争场景: $100K-$500K
- 高竞争场景: $1M-$5M

资金优化策略：
1. 选择现有流动性较低的tick范围
2. 不追求100%份额，80%也足够
3. 使用闪电贷获取临时资金
4. 只JIT高价值的swap（手续费 > 资金成本）
```

---

## 四、对被动LP的影响

### 4.1 手续费稀释量化

```
被动LP损失计算：

没有JIT时：
被动LP的手续费 = swap_amount * fee_rate * (passive_liquidity / total_liquidity)

有JIT时：
被动LP的手续费 = swap_amount * fee_rate * (passive_liquidity / (total_liquidity + jit_liquidity))

手续费损失率：
dilution = jit_liquidity / (total_liquidity + jit_liquidity)

示例：
- tick范围内现有流动性: $1M
- JIT添加的流动性: $9M
- 稀释率: 9M / (1M + 9M) = 90%
- 被动LP只获得原来10%的手续费

年化影响估算：
- 假设高价值swap占总交易量的20%
- 这20%的交易中，50%被JIT
- 被JIT的交易平均稀释率80%
- 被动LP收入损失: 20% * 50% * 80% = 8%
（实际数据因池而异，主要池的JIT影响约5-15%）
```

### 4.2 JIT对市场效率的影响

```
正面影响：
1. 提高了大额swap的执行质量（更多流动性=更小滑点）
2. 提高了资本效率（流动性只在需要时出现）
3. 降低了大额交易者的执行成本

负面影响：
1. 被动LP收入减少 → 可能导致被动LP退出 → 长期流动性下降
2. 只有技术强的参与者能做JIT → 不公平的竞争环境
3. 可能导致"流动性萎缩螺旋"：
   JIT稀释 → 被动LP退出 → 常规流动性下降 → 更依赖JIT → ...

学术争论：
- 支持方：JIT是"专业化LP"的自然演进
- 反对方：JIT是寄生行为，长期损害生态健康
```

---

## 五、检测与分析

### 5.1 JIT检测逻辑

```
JIT检测算法：

1. 在同一区块中找到 mint → swap → burn 模式：
   - TX_i: mint() 到Uniswap V3池（添加流动性）
   - TX_j: swap() 经过同一个池 (j > i)
   - TX_k: burn() + collect() 从同一池移除流动性 (k > j)
   - TX_i和TX_k的sender相同

2. 验证条件：
   - mint和burn的position覆盖swap的价格范围
   - mint和burn在同一区块（或相邻区块）
   - burn的流动性量 ≈ mint的流动性量
   - 时间间隔极短（同一区块=0秒）

3. 量化指标：
   - JIT流动性占比 = jit_liquidity / total_active_liquidity
   - 手续费份额 = fee_earned_by_jit / total_fee_in_range
   - 净利润 = fee_earned - IL - gas_cost
```

### 5.2 Dune SQL JIT检测查询

```sql
-- JIT流动性检测查询（Uniswap V3）
-- 检测同一区块中的mint→swap→burn模式

WITH v3_mints AS (
    SELECT
        block_number,
        block_time,
        tx_hash,
        tx_index,
        tx_from,
        contract_address AS pool_address,
        -- IncreaseLiquidity event
        varbinary_to_uint256(substr(data, 1, 32)) AS tokenId,
        varbinary_to_uint256(substr(data, 33, 32)) AS liquidity,
        varbinary_to_uint256(substr(data, 65, 32)) AS amount0,
        varbinary_to_uint256(substr(data, 97, 32)) AS amount1
    FROM ethereum.logs
    WHERE block_time >= NOW() - INTERVAL '7' DAY
        AND topic0 = 0x3067048beee31b25b2f1681f88dac838c8bba36af25bfb2b7cf7473a5847e35f
        -- IncreaseLiquidity event signature
),

v3_burns AS (
    SELECT
        block_number,
        block_time,
        tx_hash,
        tx_index,
        tx_from,
        contract_address AS pool_address,
        varbinary_to_uint256(substr(data, 1, 32)) AS tokenId,
        varbinary_to_uint256(substr(data, 33, 32)) AS liquidity
    FROM ethereum.logs
    WHERE block_time >= NOW() - INTERVAL '7' DAY
        AND topic0 = 0x26f6a048ee9138f2c0ce266f322cb99228e8d619ae2bff30c67f8dcf9d2377b4
        -- DecreaseLiquidity event signature
),

v3_swaps AS (
    SELECT
        block_number,
        tx_hash,
        tx_index,
        tx_from,
        contract_address AS pool_address
    FROM ethereum.logs
    WHERE block_time >= NOW() - INTERVAL '7' DAY
        AND topic0 = 0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67
        -- V3 Swap event
),

jit_candidates AS (
    SELECT
        m.block_number,
        m.block_time,
        m.tx_hash AS mint_tx,
        s.tx_hash AS swap_tx,
        b.tx_hash AS burn_tx,
        m.tx_from AS jit_provider,
        s.tx_from AS swapper,
        m.pool_address,
        m.tx_index AS mint_index,
        s.tx_index AS swap_index,
        b.tx_index AS burn_index,
        CAST(m.liquidity AS double) AS mint_liquidity,
        CAST(m.amount0 AS double) / 1e18 AS amount0_provided,
        CAST(m.amount1 AS double) / 1e6 AS amount1_provided
    FROM v3_mints m
    JOIN v3_swaps s
        ON m.block_number = s.block_number
        AND m.pool_address = s.pool_address
        AND m.tx_index < s.tx_index
    JOIN v3_burns b
        ON s.block_number = b.block_number
        AND s.pool_address = b.pool_address
        AND s.tx_index < b.tx_index
        AND m.tx_from = b.tx_from  -- 同一JIT provider
        AND m.tokenId = b.tokenId  -- 同一position
    WHERE m.tx_from != s.tx_from   -- JIT provider不是swapper
        AND b.tx_index - m.tx_index <= 5  -- 间隔不超过5笔交易
)

SELECT
    block_number,
    block_time,
    jit_provider,
    swapper,
    pool_address,
    mint_tx,
    swap_tx,
    burn_tx,
    mint_liquidity,
    amount0_provided,
    amount1_provided,
    swap_index - mint_index AS gap_mint_to_swap,
    burn_index - swap_index AS gap_swap_to_burn
FROM jit_candidates
ORDER BY block_time DESC
LIMIT 100
```

### 5.3 JIT活动统计查询

```sql
-- JIT活动趋势统计
SELECT
    DATE_TRUNC('day', block_time) AS day,
    COUNT(*) AS jit_count,
    COUNT(DISTINCT jit_provider) AS unique_jit_providers,
    COUNT(DISTINCT pool_address) AS unique_pools,
    SUM(amount0_provided) AS total_amount0,
    SUM(amount1_provided) AS total_amount1,
    AVG(gap_mint_to_swap) AS avg_gap_to_swap
FROM jit_candidates
GROUP BY DATE_TRUNC('day', block_time)
ORDER BY day DESC
LIMIT 30
```

---

## 六、主要JIT Bot分析

### 6.1 市场格局

```
主要JIT流动性提供者（2024-2025年数据）：

Tier 1（专业做市商的JIT部门）：
- 专业MEV团队（匿名地址）
- 占以太坊JIT活动的约50-60%
- 通常同时运营多种MEV策略

Tier 2（专注JIT的中型bot）：
- 数量约10-20个
- 各自占5-15%的份额
- 通常在特定池上有优势

活动数据：
- 以太坊主网日均JIT事件: ~200-1000次
- JIT占大额swap手续费的比例: ~10-30%
- 主要集中在高交易量池:
  * ETH/USDC 0.05%
  * ETH/USDC 0.3%
  * WBTC/ETH 0.3%
  * 热门新代币池
```

### 6.2 JIT利润趋势

```
JIT利润演变：

2022年（JIT策略早期）：
- 日均利润: $5K-$20K（少数bot）
- 竞争者少，利润率高
- 技术门槛是主要壁垒

2023年（竞争加剧）：
- 日均利润: $2K-$10K/bot
- 更多bot进入
- 利润率下降约50%

2024-2025年（成熟期）：
- 日均利润: $1K-$5K/bot（头部）
- 长尾bot微利或亏损
- 差异化成为关键：
  * 更快的mempool数据
  * 更精确的tick计算
  * 更低的gas消耗
```

---

## 七、防御机制

### 7.1 协议层面防御

| 防御机制 | 原理 | 实现状态 |
|---------|------|---------|
| **时间加权流动性** | 手续费按流动性持续时间分配，不按瞬时份额 | 部分协议已实现 |
| **最小流动性持续时间** | 流动性必须持续N个区块才能获得手续费 | Uniswap V4可通过Hook实现 |
| **渐进式手续费分配** | 新增流动性的手续费份额随时间线性增长 | 提案阶段 |
| **JIT检测Hook** | 自动检测并惩罚同区块mint-burn行为 | V4 Hook可实现 |

### 7.2 Uniswap V4 Hook防御可能性

```solidity
// 概念性的Anti-JIT Hook（Uniswap V4）
contract AntiJITHook is BaseHook {
    // 记录流动性添加的区块
    mapping(uint256 => uint256) public positionMintBlock;
    // 最小持续区块数
    uint256 public constant MIN_BLOCKS = 10;

    function beforeAddLiquidity(
        address sender,
        PoolKey calldata key,
        IPoolManager.ModifyLiquidityParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4) {
        // 记录添加流动性的区块
        uint256 positionId = getPositionId(sender, params);
        positionMintBlock[positionId] = block.number;
        return BaseHook.beforeAddLiquidity.selector;
    }

    function beforeRemoveLiquidity(
        address sender,
        PoolKey calldata key,
        IPoolManager.ModifyLiquidityParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4) {
        uint256 positionId = getPositionId(sender, params);
        uint256 mintBlock = positionMintBlock[positionId];

        // 如果流动性持续时间不足，收取惩罚费
        if (block.number - mintBlock < MIN_BLOCKS) {
            uint256 penalty = calculatePenalty(
                params.liquidityDelta,
                MIN_BLOCKS - (block.number - mintBlock)
            );
            // 将惩罚费分配给长期LP
            distributePenalty(key, penalty);
        }

        return BaseHook.beforeRemoveLiquidity.selector;
    }
}
```

### 7.3 被动LP的自我保护

```
被动LP减少JIT影响的策略：

1. 选择低JIT活动的池
   - 1% fee pool: JIT活动较少（大额swap也少）
   - 新池/长尾池: JIT bot覆盖较少

2. 使用更宽的价格范围
   - 宽范围LP受JIT影响较小
   - 因为JIT只针对窄tick范围

3. 使用LP管理协议
   - Arrakis/Gamma等协议会动态调整范围
   - 部分协议有anti-JIT逻辑

4. 关注时间加权手续费协议
   - 选择使用时间加权分配手续费的V3 fork
   - 如Ambient/CrocSwap等

5. 迁移到V4池（可用后）
   - 选择配置了anti-JIT Hook的池
```

---

## 八、高级技术分析

### 8.1 跨池JIT

```
当大额swap经过多个池时，JIT bot可以在多个池上同时操作：

用户路径: USDC → ETH → WBTC
池1: USDC/ETH
池2: ETH/WBTC

多池JIT Bundle:
TX1: mint() 到池1（USDC/ETH）
TX2: mint() 到池2（ETH/WBTC）
TX3: 用户的swap交易（经过两个池）
TX4: burn() 从池1
TX5: burn() 从池2

难点：
- 需要同时准备两种代币对的资金
- 两个池的tick计算需要联动
- Gas成本更高
- Bundle更复杂，失败概率更高
```

### 8.2 JIT与三明治的组合

```
理论上JIT可以与三明治攻击结合：

组合策略：
TX1: 攻击者买入目标代币（frontrun）
TX2: 攻击者添加JIT流动性（在推高后的价格范围）
TX3: 受害者swap执行（更差的价格 + 手续费被JIT捕获）
TX4: 攻击者移除JIT流动性
TX5: 攻击者卖出代币（backrun）

攻击者收益 = 三明治利润 + JIT手续费收入 - gas成本

这种组合极为恶劣，但技术上可行。
检测方法：检查同一区块中是否有来自同一地址的buy+mint+burn+sell模式。
```

### 8.3 JIT的时间敏感性

```
JIT对时间/延迟的要求：

关键时间窗口：
1. 发现pending swap → 计算最优参数: <50ms
2. 构造Bundle → 提交给Builder: <100ms
3. Builder收到 → 被包含进区块: 取决于Builder

延迟预算分配：
- Mempool数据到达: 10-50ms
- Tx解析+池状态查询: 5-20ms
- Tick计算+金额优化: 5-15ms
- 签名+Bundle构造: 5-10ms
- 网络传输到Builder: 10-30ms
- 总计: 35-125ms

竞争优势来源：
- 更快的Mempool数据源（Bloxroute BDN, Chainbound Fiber）
- 预计算常见池的状态
- 优化的数学计算库（Rust/C++实现）
- 地理位置靠近Builder
```

---

## 九、实战检测模板

### 检测特定池的JIT活动

```
Step 1: 确定目标池地址
  → 通过DeFiLlama或Uniswap Info获取

Step 2: 查询该池的Mint/Burn事件
  → 筛选同一区块内mint+burn的地址

Step 3: 统计JIT指标
  → JIT频率（次/天）
  → JIT流动性占比
  → JIT bot地址列表
  → 被JIT的swap金额分布

Step 4: 评估对被动LP的影响
  → 计算无JIT时的手续费收入
  → 计算有JIT时的手续费收入
  → 手续费损失率

Step 5: 输出报告
  → 池地址和代币对
  → JIT活动水平（低/中/高）
  → 主要JIT bot列表
  → 被动LP收入影响估算
  → 防御建议
```

---

## 十、工具速查表

| 用途 | 工具 | 说明 |
|------|------|------|
| JIT检测 | Dune Analytics | SQL查询mint-swap-burn模式 |
| 实时监控 | EigenPhi | 可视化JIT事件 |
| 池数据 | Uniswap Info / Revert Finance | 流动性分布、手续费数据 |
| LP分析 | Revert Finance | LP收益和JIT影响分析 |
| Tick计算 | Uniswap V3 SDK | 精确的tick数学计算 |
| 合约交互 | Etherscan | 查看JIT bot合约和交易 |
| Mempool | Bloxroute / Chainbound | 实时pending交易数据 |

---

## 诚实边界

- **JIT的伦理边界尚有争议**：它不直接伤害swap用户，但间接损害被动LP的利益
- **检测不完美**：复杂的JIT bot可能使用多个地址和合约来规避检测
- **数据有限**：精确的JIT利润难以计算，因为需要知道每个tick的完整流动性分布
- **V4 Hook防御是理论性的**：Uniswap V4的anti-JIT Hook尚未在生产环境验证
- **市场在变化**：随着Uniswap V4和新型DEX的出现，JIT的可行性和策略可能发生根本变化
- **被动LP的"损失"是相对的**：JIT同时提高了swap用户的执行质量

---

## 关联Skills

- [uniswap-v3-math-model](../uniswap-v3-math-model/) — Uniswap V3集中流动性数学模型
- [lp-toxic-flow-analysis](../lp-toxic-flow-analysis/) — LP有毒流分析
- [mempool-monitoring](../mempool-monitoring/) — Mempool监听基础设施

---

## 附录：调研来源

### 核心参考
- Uniswap V3白皮书: https://uniswap.org/whitepaper-v3.pdf
- "Just-in-Time Liquidity on the Uniswap Protocol" (Uniswap Labs blog)
- EigenPhi JIT数据: https://eigenphi.io/
- Revert Finance LP分析: https://revert.finance/
- Paradigm: "MEV and Me" 系列
- Uniswap V4文档和Hook规范
- Dune Analytics JIT Dashboard

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
