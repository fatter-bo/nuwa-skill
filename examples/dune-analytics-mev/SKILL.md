---
name: dune-analytics-mev
description: |
  链上数据分析专家技能。使用Dune Analytics和SQL分析MEV活动与LP表现。
  覆盖Dune数据表结构、DuneSQL语法、MEV分析SQL模板、LP分析SQL模板、
  协议健康监控、Dashboard设计、替代工具对比。
  触发词：「MEV分析」「LP表现」「Dune查询」「TVL监控」「Dashboard设计」「SQL模板」。
---

# Dune Analytics MEV 与 LP 分析 · 操作手册

> 链上数据不会说谎，但会隐藏。
> 好的 SQL 查询是把隐藏的信息挖出来的铲子。

---

## 使用规则

**此 Skill 激活后，以链上数据分析专家身份回应。**

### 核心原则
- 所有 SQL 查询必须可在 Dune 上直接运行
- 优先使用 decoded tables（已解码表），性能远优于 raw tables
- 查询必须考虑 gas 成本（避免全表扫描）
- 分析结论必须有数据支撑

### 触发式工作流

| 用户指令 | 行动 |
|----------|------|
| 「MEV 分析」 | → 输出 sandwich/backrun/JIT 检测 SQL |
| 「LP 表现」 | → 输出 LP PnL/fee/IL 计算 SQL |
| 「Dune 查询」 | → 根据需求编写优化的 DuneSQL |
| 「TVL 监控」 | → 输出协议 TVL 异常检测查询 |
| 「Dashboard 设计」 | → 输出 Dashboard 布局 + 查询集合 |
| 「SQL 模板」 | → 输出可复用的参数化查询模板 |

---

## 一、Dune 基础

### 1.1 数据表结构

Dune 的数据分为三层：

```
Layer 1: Raw Tables（原始数据）
├── ethereum.transactions     — 所有交易
├── ethereum.logs             — 所有 event logs
├── ethereum.traces           — 所有 internal transactions
├── ethereum.blocks           — 区块信息
└── ethereum.creation_traces  — 合约创建记录

Layer 2: Decoded Tables（解码数据）
├── uniswap_v3_ethereum.Pair_evt_Swap
├── uniswap_v2_ethereum.Pair_evt_Swap
├── aave_v3_ethereum.Pool_evt_Supply
├── erc20_ethereum.evt_Transfer
└── ... (每个已验证合约的每个 event 都有对应表)

Layer 3: Spellbook（社区维护的聚合表）
├── dex.trades              — 所有 DEX 交易汇总
├── tokens.erc20            — ERC20 token 信息
├── prices.usd              — Token 价格（分钟级）
├── labels.all              — 地址标签
└── balances.erc20_latest   — 最新 ERC20 余额
```

### 1.2 DuneSQL vs 标准 SQL

DuneSQL 基于 Trino（原 PrestoSQL），与标准 SQL 有以下区别：

```sql
-- 1. 地址和 hash 使用 varbinary 类型
-- 正确写法：
WHERE "from" = 0xd8da6bf26964af9d7eed9e03e53415d37aa96045
-- 错误写法（字符串）：
-- WHERE "from" = '0xd8da6bf26964af9d7eed9e03e53415d37aa96045'

-- 2. 字节操作
SELECT bytearray_substring(data, 1, 4) AS selector  -- 提取前 4 bytes
SELECT bytearray_to_uint256(bytearray_substring(data, 33, 32)) AS amount

-- 3. uint256 使用 UINT256 类型
SELECT CAST(value AS UINT256) FROM ethereum.transactions

-- 4. 时间函数
SELECT date_trunc('day', block_time) AS day
SELECT date_diff('hour', start_time, end_time)

-- 5. 数组操作
SELECT element_at(topics, 1) AS topic0  -- Trino 数组从 1 开始

-- 6. LATERAL JOIN（展开数组）
SELECT t.tx_hash, e.log
FROM transactions t
CROSS JOIN UNNEST(t.logs) AS e(log)
```

### 1.3 常用 DeFi 表速查

| 协议 | 表名 | 关键字段 |
|------|------|----------|
| **Uniswap V3** | `uniswap_v3_ethereum.Pair_evt_Swap` | amount0, amount1, sqrtPriceX96, tick, liquidity |
| **Uniswap V3** | `uniswap_v3_ethereum.NonfungiblePositionManager_evt_IncreaseLiquidity` | tokenId, liquidity, amount0, amount1 |
| **Uniswap V2** | `uniswap_v2_ethereum.Pair_evt_Swap` | amount0In, amount1In, amount0Out, amount1Out |
| **Aave V3** | `aave_v3_ethereum.Pool_evt_Supply` | reserve, user, amount |
| **Aave V3** | `aave_v3_ethereum.Pool_evt_Borrow` | reserve, user, amount, interestRateMode |
| **ERC20** | `erc20_ethereum.evt_Transfer` | "from", "to", value, contract_address |
| **DEX 聚合** | `dex.trades` | project, token_bought_address, amount_usd, tx_hash |
| **价格** | `prices.usd` | contract_address, minute, price |

---

## 二、MEV 分析 SQL 模板

### 2.1 Sandwich Attack 检测

```sql
-- Sandwich Attack 检测：在同一区块中，同一 MEV bot 在受害者交易前后
-- 对同一 pool 进行方向相反的 swap
WITH block_swaps AS (
    SELECT
        s.evt_block_number,
        s.evt_tx_hash,
        s.evt_index,
        s.contract_address AS pool,
        t.index AS tx_index,
        t."from" AS tx_sender,
        s.amount0,
        s.amount1,
        s.sender,
        s.recipient
    FROM uniswap_v3_ethereum.Pair_evt_Swap s
    JOIN ethereum.transactions t
        ON s.evt_tx_hash = t.hash
        AND s.evt_block_number = t.block_number
    WHERE s.evt_block_time >= NOW() - INTERVAL '24' HOUR
),
-- 找到同一区块同一 pool 的连续交易
sandwich_candidates AS (
    SELECT
        a.evt_block_number AS block_number,
        a.pool,
        a.evt_tx_hash AS frontrun_tx,
        b.evt_tx_hash AS victim_tx,
        c.evt_tx_hash AS backrun_tx,
        a.tx_sender AS mev_bot,
        b.tx_sender AS victim,
        a.tx_index AS front_index,
        b.tx_index AS victim_index,
        c.tx_index AS back_index,
        a.amount0 AS front_amount0,
        c.amount0 AS back_amount0
    FROM block_swaps a
    JOIN block_swaps b ON a.evt_block_number = b.evt_block_number
        AND a.pool = b.pool
        AND a.tx_index < b.tx_index
    JOIN block_swaps c ON b.evt_block_number = c.evt_block_number
        AND b.pool = c.pool
        AND b.tx_index < c.tx_index
    WHERE a.tx_sender = c.tx_sender           -- 前后交易同一 sender
        AND a.tx_sender != b.tx_sender        -- 与受害者不同
        AND c.tx_index = b.tx_index + 1       -- backrun 紧跟 victim
        AND a.tx_index = b.tx_index - 1       -- frontrun 紧贴 victim
        -- 方向相反：frontrun 买入，backrun 卖出
        AND SIGN(CAST(a.amount0 AS DOUBLE)) != SIGN(CAST(c.amount0 AS DOUBLE))
)
SELECT
    block_number,
    mev_bot,
    victim,
    frontrun_tx,
    victim_tx,
    backrun_tx,
    pool
FROM sandwich_candidates
ORDER BY block_number DESC
LIMIT 100
```

### 2.2 Backrun Arbitrage 识别与利润计算

```sql
-- Backrun Arbitrage 识别：
-- 特征：交易起始和结束 token 相同，涉及多个 DEX pool，净赚 ETH/token
WITH arb_txs AS (
    SELECT
        t.hash AS tx_hash,
        t.block_number,
        t.block_time,
        t."from" AS arb_bot,
        t.gas_used * t.gas_price / 1e18 AS gas_cost_eth,
        -- 计算 ETH 净变化
        (CAST(t.value AS DOUBLE) / 1e18) AS eth_sent
    FROM ethereum.transactions t
    WHERE t.block_time >= NOW() - INTERVAL '24' HOUR
        AND t.success = true
        AND t.gas_used > 100000  -- 过滤简单交易
),
-- 获取每笔交易涉及的 DEX swap 数量
swap_counts AS (
    SELECT
        d.tx_hash,
        COUNT(*) AS swap_count,
        COUNT(DISTINCT d.project) AS dex_count
    FROM dex.trades d
    WHERE d.block_time >= NOW() - INTERVAL '24' HOUR
    GROUP BY d.tx_hash
    HAVING COUNT(*) >= 2  -- 至少 2 次 swap
),
-- 计算每笔交易的 WETH 净变化
weth_flows AS (
    SELECT
        evt_tx_hash AS tx_hash,
        SUM(CASE WHEN "to" = t."from" THEN CAST(value AS DOUBLE) ELSE 0 END) / 1e18
        - SUM(CASE WHEN "from" = t."from" THEN CAST(value AS DOUBLE) ELSE 0 END) / 1e18
            AS weth_net
    FROM erc20_ethereum.evt_Transfer e
    JOIN ethereum.transactions t ON e.evt_tx_hash = t.hash
    WHERE e.contract_address = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 -- WETH
        AND e.evt_block_time >= NOW() - INTERVAL '24' HOUR
    GROUP BY evt_tx_hash, t."from"
)
SELECT
    a.tx_hash,
    a.block_time,
    a.arb_bot,
    s.swap_count,
    s.dex_count,
    COALESCE(w.weth_net, 0) AS weth_profit_eth,
    a.gas_cost_eth,
    COALESCE(w.weth_net, 0) - a.gas_cost_eth AS net_profit_eth
FROM arb_txs a
JOIN swap_counts s ON a.tx_hash = s.tx_hash
LEFT JOIN weth_flows w ON a.tx_hash = w.tx_hash
WHERE COALESCE(w.weth_net, 0) > a.gas_cost_eth  -- 只看盈利交易
ORDER BY net_profit_eth DESC
LIMIT 100
```

### 2.3 JIT Liquidity 检测

```sql
-- JIT Liquidity 检测：同一区块内 Mint → Swap → Burn 序列
WITH v3_events AS (
    SELECT
        evt_block_number,
        evt_tx_hash,
        evt_index,
        contract_address AS pool,
        'mint' AS event_type,
        owner,
        amount AS liquidity
    FROM uniswap_v3_ethereum.Pair_evt_Mint
    WHERE evt_block_time >= NOW() - INTERVAL '7' DAY

    UNION ALL

    SELECT
        evt_block_number,
        evt_tx_hash,
        evt_index,
        contract_address AS pool,
        'burn' AS event_type,
        owner,
        amount AS liquidity
    FROM uniswap_v3_ethereum.Pair_evt_Burn
    WHERE evt_block_time >= NOW() - INTERVAL '7' DAY
),
-- 同一区块同一 pool 的 mint-burn 配对
mint_burn_pairs AS (
    SELECT
        m.evt_block_number,
        m.pool,
        m.owner AS jit_provider,
        m.evt_tx_hash AS mint_tx,
        b.evt_tx_hash AS burn_tx,
        m.liquidity AS minted_liquidity,
        b.liquidity AS burned_liquidity
    FROM v3_events m
    JOIN v3_events b ON m.evt_block_number = b.evt_block_number
        AND m.pool = b.pool
        AND m.owner = b.owner
    WHERE m.event_type = 'mint'
        AND b.event_type = 'burn'
        AND m.evt_tx_hash != b.evt_tx_hash  -- 不同交易
),
-- 检查中间是否有大额 swap
sandwiched_swaps AS (
    SELECT
        p.*,
        s.evt_tx_hash AS swap_tx,
        s.amount0,
        s.amount1
    FROM mint_burn_pairs p
    JOIN uniswap_v3_ethereum.Pair_evt_Swap s
        ON p.evt_block_number = s.evt_block_number
        AND p.pool = s.contract_address
    WHERE s.evt_tx_hash != p.mint_tx
        AND s.evt_tx_hash != p.burn_tx
)
SELECT
    evt_block_number AS block_number,
    jit_provider,
    pool,
    mint_tx,
    swap_tx,
    burn_tx,
    minted_liquidity,
    CAST(amount0 AS DOUBLE) / 1e18 AS swap_amount0,
    CAST(amount1 AS DOUBLE) / 1e6 AS swap_amount1
FROM sandwiched_swaps
ORDER BY evt_block_number DESC
LIMIT 50
```

### 2.4 MEV Bot 利润排行榜

```sql
-- MEV Bot 利润排行榜（过去 30 天）
WITH mev_profits AS (
    SELECT
        t."from" AS mev_bot,
        t.hash AS tx_hash,
        t.block_time,
        t.gas_used * t.gas_price / 1e18 AS gas_cost_eth,
        -- 通过 coinbase transfer 判断 builder payment
        COALESCE(
            (SELECT CAST(tr.value AS DOUBLE) / 1e18
             FROM ethereum.traces tr
             WHERE tr.tx_hash = t.hash
                AND tr."to" = b.miner  -- 支付给 builder
                AND tr.type = 'call'
                AND tr.call_type = 'call'
            ), 0
        ) AS builder_payment_eth
    FROM ethereum.transactions t
    JOIN ethereum.blocks b ON t.block_number = b.number
    WHERE t.block_time >= NOW() - INTERVAL '30' DAY
        AND t."from" IN (
            -- 已知 MEV bot 地址列表
            0x56178a0d5F301bAf6CF3e1Cd53d9863437345Bf9,  -- jaredfromsubway.eth
            0x6b75d8AF000000e20B7a7DDf000Ba900b4009A80,  -- Sandwich bot
            0x00000000003b3cc22aF3aE1EAc0440BcEe416B40   -- Flashbots searcher
        )
        AND t.success = true
)
SELECT
    mev_bot,
    COUNT(*) AS total_txs,
    SUM(gas_cost_eth) AS total_gas_cost_eth,
    SUM(builder_payment_eth) AS total_builder_payment_eth,
    -- 估算利润需要更精确的计算（这里用简化版）
    SUM(gas_cost_eth + builder_payment_eth) AS total_cost_eth
FROM mev_profits
GROUP BY mev_bot
ORDER BY total_txs DESC
```

---

## 三、LP 分析 SQL 模板

### 3.1 Uniswap V3 Position 生命周期

```sql
-- 追踪指定 LP position 的完整生命周期
-- 参数化查询：{{token_id}} 为 NFT position ID
WITH position_events AS (
    -- Mint（创建/增加流动性）
    SELECT
        evt_block_time AS event_time,
        evt_tx_hash AS tx_hash,
        'increase' AS event_type,
        liquidity,
        amount0,
        amount1,
        tokenId
    FROM uniswap_v3_ethereum.NonfungiblePositionManager_evt_IncreaseLiquidity
    WHERE tokenId = {{token_id}}

    UNION ALL

    -- Burn（减少流动性）
    SELECT
        evt_block_time AS event_time,
        evt_tx_hash AS tx_hash,
        'decrease' AS event_type,
        liquidity,
        amount0,
        amount1,
        tokenId
    FROM uniswap_v3_ethereum.NonfungiblePositionManager_evt_DecreaseLiquidity
    WHERE tokenId = {{token_id}}

    UNION ALL

    -- Collect（收取手续费）
    SELECT
        evt_block_time AS event_time,
        evt_tx_hash AS tx_hash,
        'collect' AS event_type,
        CAST(0 AS UINT256) AS liquidity,
        amount0,
        amount1,
        tokenId
    FROM uniswap_v3_ethereum.NonfungiblePositionManager_evt_Collect
    WHERE tokenId = {{token_id}}
)
SELECT
    event_time,
    tx_hash,
    event_type,
    CAST(liquidity AS DOUBLE) AS liquidity,
    CAST(amount0 AS DOUBLE) / 1e18 AS amount0,
    CAST(amount1 AS DOUBLE) / 1e6 AS amount1
FROM position_events
ORDER BY event_time ASC
```

### 3.2 LP 手续费收入计算

```sql
-- 计算指定 pool 中 LP 的手续费收入（过去 30 天）
WITH pool_swaps AS (
    SELECT
        date_trunc('day', evt_block_time) AS day,
        contract_address AS pool,
        ABS(CAST(amount0 AS DOUBLE)) / 1e18 AS volume_token0,
        ABS(CAST(amount1 AS DOUBLE)) / 1e6 AS volume_token1
    FROM uniswap_v3_ethereum.Pair_evt_Swap
    WHERE contract_address = {{pool_address}}
        AND evt_block_time >= NOW() - INTERVAL '30' DAY
),
daily_volume AS (
    SELECT
        day,
        SUM(volume_token0) AS total_volume_token0,
        SUM(volume_token1) AS total_volume_token1
    FROM pool_swaps
    GROUP BY day
)
SELECT
    day,
    total_volume_token0,
    total_volume_token1,
    -- Uniswap V3 fee tier: 0.05%, 0.3%, 1%
    -- 假设 0.3% fee tier
    total_volume_token0 * 0.003 AS fee_token0,
    total_volume_token1 * 0.003 AS fee_token1
FROM daily_volume
ORDER BY day DESC
```

### 3.3 Impermanent Loss 计算

```sql
-- IL 计算：比较 LP position 与单纯持有的价值差异
-- 简化版本：基于 V2 constant product 公式
-- IL = 2 * sqrt(price_ratio) / (1 + price_ratio) - 1
WITH price_data AS (
    SELECT
        date_trunc('hour', minute) AS hour,
        AVG(price) AS avg_price
    FROM prices.usd
    WHERE contract_address = {{token_address}}
        AND minute >= NOW() - INTERVAL '30' DAY
    GROUP BY 1
),
il_calculation AS (
    SELECT
        hour,
        avg_price,
        FIRST_VALUE(avg_price) OVER (ORDER BY hour ASC) AS entry_price,
        avg_price / FIRST_VALUE(avg_price) OVER (ORDER BY hour ASC) AS price_ratio
    FROM price_data
)
SELECT
    hour,
    avg_price,
    price_ratio,
    -- IL 公式
    2 * SQRT(price_ratio) / (1 + price_ratio) - 1 AS impermanent_loss,
    -- 百分比形式
    (2 * SQRT(price_ratio) / (1 + price_ratio) - 1) * 100 AS il_percentage
FROM il_calculation
ORDER BY hour ASC
```

### 3.4 Markout 分析（LP 毒性流量）

```sql
-- Markout Analysis: 衡量 swap 后价格变动方向
-- 正 markout = LP 亏损（swap 方向与后续价格趋势一致）
-- 负 markout = LP 获利（swap 方向与后续价格趋势相反）
WITH swaps_with_price AS (
    SELECT
        s.evt_block_time AS swap_time,
        s.evt_tx_hash,
        s.contract_address AS pool,
        -- sqrtPriceX96 转换为实际价格
        POWER(CAST(s.sqrtPriceX96 AS DOUBLE) / POWER(2, 96), 2) AS price_at_swap,
        -- swap 方向：amount0 > 0 表示 token0 流入 pool（买 token1）
        CASE WHEN CAST(s.amount0 AS DOUBLE) > 0 THEN 'buy_token1' ELSE 'sell_token1' END AS direction,
        ABS(CAST(s.amount0 AS DOUBLE)) / 1e18 AS abs_amount0
    FROM uniswap_v3_ethereum.Pair_evt_Swap s
    WHERE s.contract_address = {{pool_address}}
        AND s.evt_block_time >= NOW() - INTERVAL '7' DAY
),
-- 获取 swap 后 5 分钟的价格
future_prices AS (
    SELECT
        s.*,
        (SELECT AVG(POWER(CAST(s2.sqrtPriceX96 AS DOUBLE) / POWER(2, 96), 2))
         FROM uniswap_v3_ethereum.Pair_evt_Swap s2
         WHERE s2.contract_address = s.pool
           AND s2.evt_block_time BETWEEN s.swap_time + INTERVAL '4' MINUTE
                                      AND s.swap_time + INTERVAL '6' MINUTE
        ) AS price_5min_later
    FROM swaps_with_price s
)
SELECT
    date_trunc('hour', swap_time) AS hour,
    COUNT(*) AS num_swaps,
    AVG(
        CASE
            WHEN direction = 'buy_token1'
            THEN (price_5min_later - price_at_swap) / price_at_swap
            ELSE (price_at_swap - price_5min_later) / price_at_swap
        END
    ) * 10000 AS avg_markout_bps,  -- 以 basis points 表示
    -- 正 markout = LP 被逆向选择（亏损）
    SUM(CASE
        WHEN direction = 'buy_token1' AND price_5min_later > price_at_swap THEN 1
        WHEN direction = 'sell_token1' AND price_5min_later < price_at_swap THEN 1
        ELSE 0
    END) * 100.0 / COUNT(*) AS toxic_flow_pct
FROM future_prices
WHERE price_5min_later IS NOT NULL
GROUP BY 1
ORDER BY hour DESC
```

### 3.5 LP 表现排名

```sql
-- 按 LP 地址排名：综合考虑手续费收入和 IL
-- 基于 Collect 事件统计手续费收入
WITH lp_fees AS (
    SELECT
        recipient AS lp_address,
        SUM(CAST(amount0 AS DOUBLE)) / 1e18 AS total_fee_token0,
        SUM(CAST(amount1 AS DOUBLE)) / 1e6 AS total_fee_token1
    FROM uniswap_v3_ethereum.NonfungiblePositionManager_evt_Collect
    WHERE evt_block_time >= NOW() - INTERVAL '30' DAY
    GROUP BY recipient
),
lp_fee_usd AS (
    SELECT
        lp_address,
        total_fee_token0,
        total_fee_token1,
        -- 简化：使用当前价格估算
        total_fee_token0 * (SELECT price FROM prices.usd WHERE contract_address = {{token0_address}} ORDER BY minute DESC LIMIT 1)
        + total_fee_token1 * (SELECT price FROM prices.usd WHERE contract_address = {{token1_address}} ORDER BY minute DESC LIMIT 1)
            AS total_fee_usd
    FROM lp_fees
)
SELECT
    lp_address,
    total_fee_token0,
    total_fee_token1,
    total_fee_usd,
    RANK() OVER (ORDER BY total_fee_usd DESC) AS fee_rank
FROM lp_fee_usd
WHERE total_fee_usd > 100  -- 过滤小额
ORDER BY total_fee_usd DESC
LIMIT 50
```

---

## 四、协议健康监控

### 4.1 TVL 异常检测

```sql
-- TVL 日变化检测：超过 10% 的变化标记为异常
WITH daily_tvl AS (
    SELECT
        date_trunc('day', evt_block_time) AS day,
        SUM(CASE WHEN "to" = {{protocol_address}} THEN CAST(value AS DOUBLE) / 1e18 ELSE 0 END)
        - SUM(CASE WHEN "from" = {{protocol_address}} THEN CAST(value AS DOUBLE) / 1e18 ELSE 0 END)
            AS daily_net_flow
    FROM erc20_ethereum.evt_Transfer
    WHERE (
        "to" = {{protocol_address}}
        OR "from" = {{protocol_address}}
    )
    AND contract_address = {{token_address}}
    AND evt_block_time >= NOW() - INTERVAL '90' DAY
    GROUP BY 1
),
cumulative_tvl AS (
    SELECT
        day,
        daily_net_flow,
        SUM(daily_net_flow) OVER (ORDER BY day ASC) AS tvl,
        LAG(SUM(daily_net_flow) OVER (ORDER BY day ASC)) OVER (ORDER BY day ASC) AS prev_tvl
    FROM daily_tvl
)
SELECT
    day,
    tvl,
    daily_net_flow,
    CASE
        WHEN prev_tvl > 0 THEN (tvl - prev_tvl) / prev_tvl * 100
        ELSE 0
    END AS tvl_change_pct,
    CASE
        WHEN prev_tvl > 0 AND ABS((tvl - prev_tvl) / prev_tvl) > 0.1
        THEN 'ALERT'
        ELSE 'NORMAL'
    END AS status
FROM cumulative_tvl
ORDER BY day DESC
```

### 4.2 大额交易监控

```sql
-- 过去 24 小时大额 DEX 交易（> $100K）
SELECT
    block_time,
    project,
    tx_hash,
    tx_from AS trader,
    token_bought_symbol,
    token_sold_symbol,
    amount_usd,
    CASE
        WHEN amount_usd >= 1000000 THEN 'WHALE'
        WHEN amount_usd >= 100000 THEN 'LARGE'
        ELSE 'MEDIUM'
    END AS size_category
FROM dex.trades
WHERE block_time >= NOW() - INTERVAL '24' HOUR
    AND amount_usd >= 100000
ORDER BY amount_usd DESC
LIMIT 200
```

### 4.3 流动性分布可视化

```sql
-- Uniswap V3 流动性分布（按 tick range）
WITH current_positions AS (
    SELECT
        tickLower,
        tickUpper,
        SUM(CAST(amount AS DOUBLE)) AS total_liquidity
    FROM uniswap_v3_ethereum.Pair_evt_Mint
    WHERE contract_address = {{pool_address}}
        AND evt_block_time >= NOW() - INTERVAL '30' DAY
    GROUP BY tickLower, tickUpper
),
burned AS (
    SELECT
        tickLower,
        tickUpper,
        SUM(CAST(amount AS DOUBLE)) AS burned_liquidity
    FROM uniswap_v3_ethereum.Pair_evt_Burn
    WHERE contract_address = {{pool_address}}
        AND evt_block_time >= NOW() - INTERVAL '30' DAY
    GROUP BY tickLower, tickUpper
)
SELECT
    m.tickLower,
    m.tickUpper,
    m.total_liquidity - COALESCE(b.burned_liquidity, 0) AS active_liquidity,
    -- 将 tick 转换为价格近似值
    POWER(1.0001, m.tickLower) AS price_lower,
    POWER(1.0001, m.tickUpper) AS price_upper
FROM current_positions m
LEFT JOIN burned b ON m.tickLower = b.tickLower AND m.tickUpper = b.tickUpper
WHERE m.total_liquidity - COALESCE(b.burned_liquidity, 0) > 0
ORDER BY m.tickLower ASC
```

### 4.4 用户增长与活跃度

```sql
-- 协议日活跃用户和新用户统计
WITH user_first_interaction AS (
    SELECT
        "from" AS user_address,
        MIN(block_time) AS first_interaction
    FROM ethereum.transactions
    WHERE "to" = {{protocol_address}}
        AND success = true
    GROUP BY "from"
),
daily_activity AS (
    SELECT
        date_trunc('day', block_time) AS day,
        COUNT(DISTINCT "from") AS daily_active_users,
        COUNT(*) AS daily_transactions
    FROM ethereum.transactions
    WHERE "to" = {{protocol_address}}
        AND success = true
        AND block_time >= NOW() - INTERVAL '90' DAY
    GROUP BY 1
),
daily_new_users AS (
    SELECT
        date_trunc('day', first_interaction) AS day,
        COUNT(*) AS new_users
    FROM user_first_interaction
    WHERE first_interaction >= NOW() - INTERVAL '90' DAY
    GROUP BY 1
)
SELECT
    a.day,
    a.daily_active_users,
    a.daily_transactions,
    COALESCE(n.new_users, 0) AS new_users,
    a.daily_active_users - COALESCE(n.new_users, 0) AS returning_users
FROM daily_activity a
LEFT JOIN daily_new_users n ON a.day = n.day
ORDER BY a.day DESC
```

---

## 五、Dashboard 设计

### 5.1 MEV Activity Dashboard 布局

```
┌─────────────────────────────────────────────────────┐
│  MEV Activity Dashboard                              │
├──────────────────────┬──────────────────────────────┤
│ [Counter]            │ [Counter]                     │
│ 24h Sandwich Count   │ 24h Total MEV Profit (ETH)   │
├──────────────────────┴──────────────────────────────┤
│ [Line Chart] Daily MEV Profit Trend (30d)           │
│ - Sandwich profit                                    │
│ - Arbitrage profit                                   │
│ - Liquidation profit                                 │
├─────────────────────────────────────────────────────┤
│ [Bar Chart] MEV Type Distribution                    │
├──────────────────────┬──────────────────────────────┤
│ [Table]              │ [Pie Chart]                   │
│ Top MEV Bots         │ MEV by Protocol               │
│ - Address            │                               │
│ - Tx Count           │                               │
│ - Total Profit       │                               │
├──────────────────────┴──────────────────────────────┤
│ [Table] Latest Sandwich Attacks (auto-refresh)       │
│ - Block | Frontrun Tx | Victim Tx | Backrun Tx |    │
│   Profit | Pool                                      │
└─────────────────────────────────────────────────────┘
```

### 5.2 LP Performance Tracker 布局

```
┌─────────────────────────────────────────────────────┐
│  LP Performance Tracker                              │
├──────────────────────┬──────────────────────────────┤
│ [Input] Pool Address │ [Dropdown] Time Range         │
├──────────────────────┴──────────────────────────────┤
│ [Area Chart] Pool TVL Over Time                      │
├──────────────────────┬──────────────────────────────┤
│ [Line Chart]         │ [Line Chart]                  │
│ Fee Revenue (daily)  │ Volume (daily)                │
├──────────────────────┴──────────────────────────────┤
│ [Heatmap] Liquidity Distribution by Tick Range       │
├──────────────────────┬──────────────────────────────┤
│ [Line Chart]         │ [Line Chart]                  │
│ IL Over Time         │ Markout (bps) by Hour         │
├──────────────────────┴──────────────────────────────┤
│ [Table] Top LPs by Fee Revenue                       │
│ - LP Address | Fees Earned | IL | Net PnL            │
└─────────────────────────────────────────────────────┘
```

### 5.3 参数化查询技巧

```sql
-- Dune 参数化查询语法
-- 在 Dune UI 中使用 {{parameter_name}} 定义参数

-- 下拉选择参数
-- {{chain}} 类型设为 Enum: ethereum, arbitrum, optimism, polygon

-- 地址参数
-- {{pool_address}} 类型设为 Text

-- 时间范围参数
-- {{time_range}} 类型设为 Enum: 1 hour, 24 hours, 7 days, 30 days

-- 在 WHERE 子句中使用
WHERE evt_block_time >= NOW() - INTERVAL '{{time_range}}'
    AND contract_address = {{pool_address}}
```

---

## 六、性能优化

### 6.1 查询优化要点

```sql
-- 1. 优先使用 decoded tables（比 raw tables 快 10-100x）
-- 好：
SELECT * FROM uniswap_v3_ethereum.Pair_evt_Swap WHERE evt_block_time >= NOW() - INTERVAL '1' DAY
-- 差：
SELECT * FROM ethereum.logs WHERE topic0 = 0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67

-- 2. 时间范围限制（必须加）
WHERE block_time >= NOW() - INTERVAL '7' DAY  -- 总是加时间范围

-- 3. 避免 SELECT * 
SELECT evt_tx_hash, amount0, amount1 FROM ...  -- 只选需要的列

-- 4. 使用 Spellbook 聚合表
SELECT * FROM dex.trades  -- 比直接查多个 DEX 表高效

-- 5. CTE 替代子查询（DuneSQL 对 CTE 有优化）
WITH filtered AS (
    SELECT * FROM table WHERE time >= NOW() - INTERVAL '1' DAY
)
SELECT * FROM filtered WHERE amount > 0

-- 6. 避免 CROSS JOIN（笛卡尔积爆炸）
-- 使用 INNER JOIN 或 LEFT JOIN

-- 7. 利用分区裁剪
-- ethereum.transactions 按 block_time 分区
-- 加 block_time 范围可以大幅减少扫描量
```

---

## 七、替代工具对比

| 工具 | 特点 | SQL 语言 | 免费额度 | 适用场景 |
|------|------|----------|----------|----------|
| **Dune Analytics** | 最大社区、Spellbook | DuneSQL (Trino) | 有（限制执行次数） | MEV 分析、Dashboard |
| **Flipside Crypto** | 免费额度大、LiveQuery | Snowflake SQL | 慷慨 | 快速验证、Cross-chain |
| **Footprint Analytics** | 无代码拖拽、GameFi 数据全 | 标准 SQL | 基础免费 | 非技术用户、GameFi |
| **自建 Subgraph** | 完全可控、实时性好 | GraphQL | 自建成本 | 特定协议深度分析 |
| **直接 RPC** | 最灵活、无限制 | 无（代码） | 取决于节点 | 实时监控、自动化 |

### 选择建议

```
需要快速验证假设 → Dune / Flipside
需要实时监控 → 直接 RPC + 自建系统
需要分享给社区 → Dune Dashboard
需要特定协议深度分析 → 自建 Subgraph
需要跨链分析 → Flipside（多链支持好）
```

---

## 关联技能

- [lp-toxic-flow-analysis](../lp-toxic-flow-analysis/SKILL.md) — LP 毒性流量深度分析
- [lp-pnl-attribution](../lp-pnl-attribution/SKILL.md) — LP 收益归因分析
- [onchain-forensics](../onchain-forensics/SKILL.md) — 链上取证分析，数据查询的底层工具链
