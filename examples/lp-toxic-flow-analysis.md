---
name: lp-toxic-flow-analysis
description: |
  LP有毒流量分析与量化框架。覆盖有毒流量定义、Markout分析方法论、
  套利机器人识别、CEX-DEX套利检测、跨交易对/fee tier差异、LP防御策略。
  基于Thierry Foucault学术论文、Flashbots研究、Paradigm LVR论文、
  CrocSwap/Ambient动态费率实践深度调研。
  核心理念：LP的对手方不是随机的——识别谁在对面交易，决定了LP的生死。
  触发词：「有毒流量」「toxic flow」「markout分析」「LP为什么亏钱」「套利bot识别」。
---

# LP有毒流量分析与量化框架

> LP亏损的根源不是无常损失的数学公式，而是信息不对称。
> 当你的对手方知道价格要往哪走，你提供的流动性就是在补贴他的利润。
> 量化"谁在和你交易"比"价格怎么变"更重要。

## Skill规则

**此Skill激活后，以LP有毒流量分析专家身份回应。**

- 所有分析基于链上数据，给出SQL查询示例
- Markout使用多个时间窗口交叉验证
- 区分结构性有毒流量（套利）和临时性有毒流量（知情交易）
- 防御策略按可行性和效果排序
- 引用最新学术研究和行业数据

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「什么是有毒流量」 | -> 输出定义 + 信息不对称框架 + 直观示例 |
| 「做markout分析」 | -> 输出方法论 + SQL查询 + 解读框架 |
| 「识别套利bot」 | -> 输出识别方法 + 链上特征 + 工具推荐 |
| 「这个池子有毒吗」 | -> 按分析框架评估 + 输出有毒流量比例 |
| 「LP怎么防御」 | -> 输出防御策略优先级列表 + 可行性评估 |
| 「不同fee tier对比」 | -> 输出跨tier有毒流量比例差异 + 选择建议 |

---

## 一、有毒流量定义与理论框架

### 1.1 信息不对称与逆向选择

**传统做市商理论（Glosten-Milgrom模型）：**
```
做市商面临两类交易者：
1. 噪声交易者（Uninformed）: 出于流动性需求交易，不知道未来价格
   → 做市商从他们身上赚取bid-ask spread
   
2. 知情交易者（Informed）: 拥有价格即将变动的信息
   → 做市商在他们身上亏损（逆向选择成本）

做市商利润 = 噪声交易者利润 - 知情交易者损失
```

**AMM的特殊性：**
```
传统做市商可以：
  - 拒绝某些订单
  - 调整报价speed（变慢）
  - 识别知情交易者并加宽spread

AMM不能：
  - 不能拒绝任何交易（permissionless）
  - 不能区分交易者身份
  - 价格更新速度受限于区块时间（12秒）
  
结果：AMM的LP承受了不成比例的逆向选择成本
```

### 1.2 有毒流量的操作性定义

```
有毒流量（Toxic Flow）= 交易执行后，价格在"正确"方向持续移动的流量

具体指标：
- Markout为正的交易方（taker）= 知情/有毒
- Markout为负的交易方（taker）= 噪声/无毒

流量有毒程度 = Σ(正markout交易) / Σ(所有交易)
```

### 1.3 有毒流量来源分类

```
类型1: CEX-DEX套利（最主要，占50-80%有毒流量）
  → CEX价格先变动 → 套利bot在DEX上按旧价格交易
  → 本质：CEX价格信息 vs AMM陈旧价格
  → 12秒区块时间 = 12秒的信息优势

类型2: 三角/多路径套利
  → DEX间价格不一致 → 跨DEX套利
  → 一个DEX的LP损失 = 另一个路径的LP损失
  → 本质：DEX之间的价格同步延迟

类型3: 三明治攻击中的有毒部分
  → 三明治的前跑交易 = 造成LP临时损失
  → 三明治的回跑交易 = LP部分恢复
  → 但在净效果上，LP仍然受损

类型4: 基于非公开信息的知情交易
  → 内幕消息（项目方/VC提前知道利好/利空）
  → 链上信息优势（MEV搜索者的bundle分析）
  → 占比较小但单笔影响大
```

---

## 二、Markout分析方法论

### 2.1 Markout定义

```
Markout(T) = 交易执行后T时间的价格 - 交易执行价格

对于买入交易（taker买入token1）:
  Markout(T) = P(t+T) - P(t)
  正Markout = 买对了（taker赚钱，LP亏钱）
  负Markout = 买错了（taker亏钱，LP赚钱）

对于卖出交易（taker卖出token1）:
  Markout(T) = P(t) - P(t+T)
  正Markout = 卖对了
  负Markout = 卖错了

LP Markout = -Taker Markout（零和博弈）
```

### 2.2 时间窗口选择

| 时间窗口 | 含义 | 适用分析 |
|----------|------|---------|
| 5秒 (next block) | 即时价格反应 | 三明治攻击、同区块MEV |
| 1分钟 | 短期价格趋势 | CEX-DEX套利 |
| 5分钟 | 中短期 | 多数套利类型 |
| 1小时 | 中期 | 知情交易 |
| 24小时 | 长期 | 基本面信息优势 |

**最佳实践：使用5分钟markout作为主要指标，配合1分钟和1小时做交叉验证。**

### 2.3 Dune Analytics Markout SQL查询

```sql
-- Uniswap V3 ETH/USDC 0.05% Pool Markout分析
-- 基于Dune Analytics

WITH swaps AS (
    SELECT
        s.evt_block_time AS swap_time,
        s.evt_block_number AS block_number,
        s.evt_tx_hash AS tx_hash,
        -- 计算执行价格
        CASE 
            WHEN s.amount0 > 0 THEN ABS(CAST(s.amount1 AS DOUBLE)) / ABS(CAST(s.amount0 AS DOUBLE))
            ELSE ABS(CAST(s.amount0 AS DOUBLE)) / ABS(CAST(s.amount1 AS DOUBLE))
        END AS execution_price,
        -- 交易方向: token0->token1 (zeroForOne=true) 
        CASE WHEN s.amount0 > 0 THEN 'sell_token1' ELSE 'buy_token1' END AS direction,
        -- 交易金额（以token1即WETH计）
        ABS(CAST(s.amount1 AS DOUBLE)) / 1e18 AS amount_eth,
        s.sender
    FROM uniswap_v3_ethereum.Pair_evt_Swap s
    WHERE s.contract_address = 0x88e6a0c2ddd26feeb64f039a2c41296fcb3f5640  -- ETH/USDC 0.05%
    AND s.evt_block_time >= NOW() - INTERVAL '7' DAY
),

-- 获取每笔swap后5分钟的价格
price_after AS (
    SELECT 
        s1.tx_hash,
        s1.execution_price,
        s1.direction,
        s1.amount_eth,
        s1.sender,
        -- 找5分钟后最近的swap价格
        (
            SELECT s2.execution_price
            FROM swaps s2
            WHERE s2.swap_time > s1.swap_time
            AND s2.swap_time <= s1.swap_time + INTERVAL '5' MINUTE
            ORDER BY s2.swap_time DESC
            LIMIT 1
        ) AS price_5min_later
    FROM swaps s1
)

SELECT
    sender,
    COUNT(*) AS trade_count,
    AVG(amount_eth) AS avg_size_eth,
    -- 5分钟Markout
    AVG(
        CASE 
            WHEN direction = 'buy_token1' THEN (price_5min_later - execution_price) / execution_price
            ELSE (execution_price - price_5min_later) / execution_price
        END
    ) * 10000 AS avg_markout_bps,
    -- 正Markout比例
    AVG(
        CASE 
            WHEN direction = 'buy_token1' AND price_5min_later > execution_price THEN 1.0
            WHEN direction = 'sell_token1' AND price_5min_later < execution_price THEN 1.0
            ELSE 0.0
        END
    ) AS positive_markout_ratio,
    -- 总Markout金额（ETH）
    SUM(
        amount_eth * 
        CASE 
            WHEN direction = 'buy_token1' THEN (price_5min_later - execution_price) / execution_price
            ELSE (execution_price - price_5min_later) / execution_price
        END
    ) AS total_markout_eth
FROM price_after
WHERE price_5min_later IS NOT NULL
GROUP BY sender
ORDER BY total_markout_eth DESC
LIMIT 50
```

### 2.4 Markout结果解读框架

```
Taker Markout分布（典型ETH/USDC 0.05%池）:

高度正Markout（>20bps, 5min）:
  → 几乎肯定是套利bot
  → 检查是否是已知MEV bot地址
  → 占交易笔数10-20%，但占交易量50-70%

微正Markout（0-20bps）:
  → 可能是次优执行的正常交易
  → 或者温和的信息优势
  → 需要配合1h markout进一步分类

负Markout（<0bps）:
  → 噪声交易者 / 流动性需求驱动
  → LP从这些交易中获利
  → 但利润可能不够覆盖其他损失
  
整池有毒度评估:
  Volume-weighted Markout > 5bps → 高毒（LP大概率亏损）
  Volume-weighted Markout 2-5bps → 中毒（fee可能勉强覆盖）
  Volume-weighted Markout < 2bps → 低毒（LP友好）
```

---

## 三、有毒流量来源识别

### 3.1 CEX-DEX套利Bot识别

**链上特征：**
```
1. 交易模式:
   - 高频（每分钟多次交易）
   - 金额精确到非整数（计算最优量）
   - 方向与CEX价格变动一致
   - 通常在区块较早位置（支付priority fee）

2. 地址特征:
   - 合约地址（非EOA）
   - 复杂的internal transactions
   - 与Flashbots/MEV-Boost builder交互
   - 高nonce（大量历史交易）

3. 盈利特征:
   - 每笔交易利润小但稳定
   - 5min markout高度正相关
   - 几乎没有亏损交易
```

**识别SQL查询：**
```sql
-- 识别高频正Markout地址（疑似套利Bot）
WITH sender_stats AS (
    -- ... (使用上面的markout查询作为基础)
    SELECT
        sender,
        COUNT(*) AS trade_count,
        AVG(markout_5min_bps) AS avg_markout,
        STDDEV(markout_5min_bps) AS markout_stddev,
        SUM(CASE WHEN markout_5min_bps > 0 THEN 1 ELSE 0 END)::FLOAT / COUNT(*) AS win_rate,
        SUM(volume_usd) AS total_volume
    FROM markout_data
    GROUP BY sender
    HAVING COUNT(*) >= 10  -- 至少10笔交易
)

SELECT 
    sender,
    trade_count,
    avg_markout,
    win_rate,
    total_volume,
    CASE 
        WHEN avg_markout > 20 AND win_rate > 0.8 AND trade_count > 100 THEN 'HIGH_CONFIDENCE_ARB_BOT'
        WHEN avg_markout > 10 AND win_rate > 0.65 THEN 'LIKELY_ARB_BOT'
        WHEN avg_markout > 5 AND win_rate > 0.55 THEN 'POSSIBLE_INFORMED'
        ELSE 'LIKELY_UNINFORMED'
    END AS classification
FROM sender_stats
ORDER BY total_volume DESC
```

### 3.2 CEX-DEX套利检测方法

```python
import requests
import time

def detect_cex_dex_arb(dex_swap, cex_price_feed):
    """
    检测一笔DEX swap是否是CEX-DEX套利
    
    逻辑：
    1. 获取swap发生时的CEX中间价
    2. 比较DEX执行价格与CEX价格
    3. 如果DEX价格显著偏离CEX价格，且swap方向"纠正"了偏离 → 套利
    """
    swap_time = dex_swap['timestamp']
    dex_price = dex_swap['execution_price']
    swap_direction = dex_swap['direction']  # 'buy' or 'sell'
    
    # CEX价格（使用swap发生前1秒的最新价格）
    cex_price = cex_price_feed.get_price_at(swap_time - 1)
    
    # 价格偏离（bps）
    deviation_bps = (dex_price - cex_price) / cex_price * 10000
    
    # 套利判断
    is_arb = False
    if swap_direction == 'buy' and deviation_bps < -5:
        # DEX价格低于CEX → 在DEX买入 → CEX-DEX套利
        is_arb = True
        arb_profit_bps = abs(deviation_bps) - dex_swap['fee_bps']  # 减去手续费
    elif swap_direction == 'sell' and deviation_bps > 5:
        # DEX价格高于CEX → 在DEX卖出 → CEX-DEX套利
        is_arb = True
        arb_profit_bps = abs(deviation_bps) - dex_swap['fee_bps']
    
    return {
        'is_arb': is_arb,
        'deviation_bps': deviation_bps,
        'arb_profit_bps': arb_profit_bps if is_arb else 0,
        'cex_price': cex_price,
        'dex_price': dex_price
    }
```

### 3.3 三明治攻击的有毒成分分析

```
三明治攻击结构：
  tx1: 前跑（attacker买入）→ 推高价格
  tx2: 受害者交易 → 以更差价格执行
  tx3: 回跑（attacker卖出）→ 获利

对LP的影响分解：
  tx1: LP以低价卖出token → LP损失 = (市场价 - 执行价) * 数量
  tx2: 受害者交易 → LP正常做市（但在被推高的价格基础上）
  tx3: LP以高价买回token → LP部分恢复

净效果：
  LP净损失 ≈ attacker利润 * (1 - fee收入占比)
  
  但LP同时赚了三笔交易的手续费
  在0.3%池中，三笔交易的fee可能覆盖大部分损失
  在0.05%池中，fee不够覆盖 → 净亏损
```

---

## 四、跨交易对与Fee Tier差异

### 4.1 ETH/USDC不同Fee Tier对比

```
ETH/USDC 0.05%池（高流动性，低费率）:
  - 有毒流量占比: 60-75%（交易量加权）
  - 主要有毒来源: CEX-DEX套利（Binance/Coinbase → Uniswap）
  - LP 5min markout: -8 to -15 bps
  - 年化fee yield: ~5-8%
  - 年化IL/LVR: ~8-12%
  - 净结果: 大多数LP亏损

ETH/USDC 0.3%池（中等流动性，中费率）:
  - 有毒流量占比: 30-45%
  - 套利bot较少（0.3% fee使大部分套利无利可图）
  - LP 5min markout: -3 to -7 bps
  - 年化fee yield: ~10-15%
  - 年化IL/LVR: ~5-8%
  - 净结果: 部分LP盈利

结论：0.3%池通过更高fee"过滤"掉了大部分小额套利
```

### 4.2 稳定币对的特殊性

```
USDC/USDT 0.01%池:
  - 有毒流量占比: 20-30%
  - 价格波动极小（1:1附近）
  - 有毒来源: depeg事件期间的知情交易
  - 正常情况: fee足够覆盖，LP稳定盈利
  - 风险情况: 突然depeg时，LP承受巨大损失

DAI/USDC 0.01%池:
  - 类似USDC/USDT
  - 额外风险: DAI的抵押品清算事件造成临时偏离
```

### 4.3 小盘币极端情况

```
SHIB/WETH 1%池:
  - 有毒流量占比: 40-60%
  - 但绝对金额较小
  - 主要有毒来源: 内幕信息（上线/退市消息）
  - fee率高(1%)但波动更高
  - 风险: 价格归零时100% IL

低流动性新币:
  - 有毒流量可能>90%（几乎只有套利者交易）
  - LP本质上是在做空波动率
  - 极不建议做LP
```

### 4.4 跨池有毒流量比例数据（基于2024-2025研究）

| 交易对 | Fee Tier | 有毒流量占比(volume) | LP 5min Markout | LP 净收益（年化） |
|--------|----------|--------------------|-----------------|--------------------|
| ETH/USDC | 0.05% | 65-75% | -10 to -15 bps | -3% to -7% |
| ETH/USDC | 0.30% | 35-45% | -4 to -8 bps | +2% to +8% |
| ETH/USDC | 1.00% | 15-25% | -2 to -5 bps | +5% to +15% |
| USDC/USDT | 0.01% | 20-30% | -1 to -3 bps | +1% to +3% |
| WBTC/ETH | 0.05% | 55-65% | -8 to -12 bps | -2% to -5% |
| WBTC/ETH | 0.30% | 30-40% | -3 to -6 bps | +3% to +10% |

*数据来源：Paradigm Research, CrocSwap分析, Dune community dashboards*

---

## 五、LP防御策略

### 5.1 策略1：动态费率 / MEV Tax

**原理：** 对知情/有毒交易收取更高费率，对无毒交易保持低费率。

```
实现方式（V4 Hook）:

beforeSwap中判断交易是否可能有毒:
  - 检查priority fee（高priority → 可能是MEV）
  - 检查交易size（大额 → 可能是套利）
  - 检查历史markout（已知bot地址）

动态费率:
  base_fee = 5 bps
  mev_tax = priority_fee * coefficient  （Diamandis et al. 2024）
  total_fee = base_fee + mev_tax
```

**效果评估：**
```
优点：
  - 理论上最优解——让套利者为信息优势付费
  - 不影响正常用户（低priority fee）
  - V4 Hook原生支持

缺点：
  - priority fee不完全等于"有毒程度"
  - 需要精确校准参数
  - 可能被绕过（通过builder直接交易）
  
实际项目：
  - CrocSwap/Ambient: 实现了基于价格波动的动态费率
  - Sorella Labs: 研究MEV-aware AMM fee
```

### 5.2 策略2：动态Range管理

```python
class DynamicRangeLP:
    """
    根据市场状况动态调整LP range
    核心逻辑：波动率高时加宽range，波动率低时收窄range
    """
    
    def __init__(self, base_range_pct=0.05, rebalance_threshold=0.8):
        self.base_range_pct = base_range_pct
        self.rebalance_threshold = rebalance_threshold
    
    def calculate_optimal_range(self, current_price, realized_vol_24h, 
                                 fee_rate, volume_tvl_ratio):
        """
        基于当前市场参数计算最优range
        """
        # 高波动率 → 宽range（减少IL和rebalance频率）
        # 高volume/TVL → 窄range（更多fee）
        
        # 简化的最优range公式（基于Sharpe ratio优化）
        # optimal_width ≈ σ * sqrt(2 / (fee_rate * V/TVL))
        if volume_tvl_ratio > 0 and fee_rate > 0:
            optimal_half_width = realized_vol_24h * (2 / (fee_rate * volume_tvl_ratio)) ** 0.5
        else:
            optimal_half_width = self.base_range_pct
        
        # 限制range在合理范围内
        optimal_half_width = max(0.01, min(0.50, optimal_half_width))
        
        lower = current_price * (1 - optimal_half_width)
        upper = current_price * (1 + optimal_half_width)
        
        return lower, upper, optimal_half_width
    
    def should_rebalance(self, current_price, range_lower, range_upper):
        """判断是否需要rebalance"""
        range_width = range_upper - range_lower
        distance_to_lower = current_price - range_lower
        distance_to_upper = range_upper - current_price
        
        # 如果价格接近边界（距离<总宽度的20%），触发rebalance
        if min(distance_to_lower, distance_to_upper) / range_width < (1 - self.rebalance_threshold) / 2:
            return True
        return False
```

### 5.3 策略3：选择低毒交易对

```
低毒交易对特征：
1. 稳定币对（USDC/USDT, DAI/USDC）
   - 价格波动极小 → 套利机会少
   - 但fee也低（0.01%）→ 需要大资金

2. 高度相关资产对（stETH/ETH, cbETH/ETH）
   - 价格比率稳定（围绕1:1）
   - 套利空间有限
   - fee适中（0.05%）

3. 高fee tier（1%）的长尾对
   - fee高到大部分套利无利可图
   - 但流动性可能很低，rebalance困难

不建议做LP的高毒对：
  - ETH/USDC 0.05%（套利bot天堂）
  - 低市值新币（信息不对称极端）
  - 即将有重大事件的代币（知情交易风险）
```

### 5.4 策略4：V4 Hook级别防御

```solidity
// 示例：Anti-Toxic Flow Hook
contract AntiToxicHook is BaseHook {
    // 使用TWAP偏离度作为有毒指标
    uint256 public constant TWAP_WINDOW = 30 minutes;
    uint256 public constant MAX_DEVIATION_BPS = 50;
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        // 获取当前价格与TWAP的偏离度
        uint256 currentPrice = getCurrentPrice(key);
        uint256 twapPrice = getTWAP(key, TWAP_WINDOW);
        uint256 deviationBps = abs(currentPrice - twapPrice) * 10000 / twapPrice;
        
        // 偏离度越大 → 费率越高
        uint24 dynamicFee;
        if (deviationBps > MAX_DEVIATION_BPS) {
            // 价格显著偏离TWAP → 可能刚发生大波动 → 提高费率
            dynamicFee = uint24(key.fee + deviationBps * 10); // 线性增加
        } else {
            dynamicFee = uint24(key.fee);
        }
        
        return (BaseHook.beforeSwap.selector,
                BeforeSwapDeltaLibrary.ZERO_DELTA,
                dynamicFee | uint24(1 << 23));
    }
}
```

---

## 六、分析工具与实时监控

### 6.1 Dune Analytics Dashboard推荐

```
推荐Dashboard:
1. @thiccythot - "Uniswap V3 LP Profitability"
   - 按pool/fee tier的LP PnL追踪
   - Markout计算

2. @0xpizzza - "MEV on Uniswap"
   - 套利交易量统计
   - Bot地址识别

3. @springzhang - "DEX Toxic Flow Monitor"
   - 实时有毒流量比例
   - 跨池对比
```

### 6.2 实时监控架构

```
数据流:
  Ethereum Node (Archive) 
    → Event Indexer (Swap events)
    → Price Calculator
    → Markout Engine (rolling window)
    → Alert System

组件:
1. Event Indexer:
   - 监听Uniswap V3 Pool Swap events
   - 同步获取block timestamp
   - 并行获取CEX价格feed

2. Markout Engine:
   - 维护5s/1min/5min/1h markout sliding window
   - 按sender地址分组统计
   - 实时更新有毒流量比例

3. Alert Rules:
   - 有毒流量比例突然升高 → 可能有大事件
   - 特定bot地址activity spike → 可能有套利机会
   - LP markout持续恶化 → 建议调整range或撤出
```

### 6.3 Python Markout计算框架

```python
import pandas as pd
import numpy as np

class MarkoutAnalyzer:
    def __init__(self, swaps_df, windows=[5, 60, 300, 3600]):
        """
        swaps_df: DataFrame with columns [timestamp, tx_hash, sender, 
                  direction, execution_price, volume_usd]
        windows: markout时间窗口（秒）
        """
        self.swaps = swaps_df.sort_values('timestamp')
        self.windows = windows
    
    def calculate_markouts(self):
        """计算所有时间窗口的markout"""
        results = self.swaps.copy()
        
        for window in self.windows:
            markouts = []
            for idx, row in self.swaps.iterrows():
                # 找到window秒后最近的swap价格
                future_time = row['timestamp'] + window
                future_swaps = self.swaps[
                    (self.swaps['timestamp'] > row['timestamp']) &
                    (self.swaps['timestamp'] <= future_time)
                ]
                
                if len(future_swaps) > 0:
                    future_price = future_swaps.iloc[-1]['execution_price']
                    if row['direction'] == 'buy':
                        markout = (future_price - row['execution_price']) / row['execution_price']
                    else:
                        markout = (row['execution_price'] - future_price) / row['execution_price']
                    markouts.append(markout * 10000)  # 转为bps
                else:
                    markouts.append(np.nan)
            
            results[f'markout_{window}s_bps'] = markouts
        
        return results
    
    def classify_senders(self, markout_results):
        """按sender分类有毒程度"""
        sender_stats = markout_results.groupby('sender').agg(
            trade_count=('tx_hash', 'count'),
            total_volume=('volume_usd', 'sum'),
            avg_markout_5min=('markout_300s_bps', 'mean'),
            win_rate_5min=('markout_300s_bps', lambda x: (x > 0).mean()),
            avg_markout_1h=('markout_3600s_bps', 'mean'),
        ).reset_index()
        
        def classify(row):
            if row['avg_markout_5min'] > 20 and row['win_rate_5min'] > 0.8:
                return 'ARB_BOT'
            elif row['avg_markout_5min'] > 10 and row['win_rate_5min'] > 0.65:
                return 'LIKELY_INFORMED'
            elif row['avg_markout_5min'] > 0:
                return 'MILD_TOXIC'
            else:
                return 'UNINFORMED'
        
        sender_stats['classification'] = sender_stats.apply(classify, axis=1)
        return sender_stats
    
    def pool_toxicity_summary(self, markout_results):
        """整池有毒度汇总"""
        total_volume = markout_results['volume_usd'].sum()
        
        # Volume-weighted markout
        vw_markout = (markout_results['markout_300s_bps'] * 
                      markout_results['volume_usd']).sum() / total_volume
        
        # 有毒交易量占比
        toxic_volume = markout_results[
            markout_results['markout_300s_bps'] > 5
        ]['volume_usd'].sum()
        toxic_ratio = toxic_volume / total_volume
        
        return {
            'volume_weighted_markout_bps': vw_markout,
            'toxic_volume_ratio': toxic_ratio,
            'toxicity_level': 'HIGH' if vw_markout > 5 else ('MEDIUM' if vw_markout > 2 else 'LOW'),
            'total_volume_usd': total_volume,
            'total_trades': len(markout_results)
        }
```

---

## 七、学术研究参考

### 关键论文

```
1. "Automated Market Making and Loss-Versus-Rebalancing" (2023)
   - 作者: Jason Milionis, Tim Roughgarden, et al.
   - 核心: 定义LVR (Loss Versus Rebalancing)作为LP损失的精确度量
   - LVR = σ^2 * L * dt / (2 * P)

2. "An Analysis of Uniswap Markets" (2021)
   - 作者: Guillermo Angeris, Hsien-Tang Kao, et al.
   - 核心: AMM的套利等价于凸函数优化

3. "Toxic Liquidation Spirals" (2024)
   - 核心: 级联清算如何放大有毒流量

4. "Dynamic AMM Fees for MEV Mitigation" (2024)
   - 作者: Diamandis, Kulkarni, Roughgarden
   - 核心: 基于priority fee的动态费率可以最优化LP收益

5. "The Costs of DEX Liquidity Provision" (2024)
   - 核心: 实证分析V3 LP的实际盈亏分解
```

---

## 相关技能

- [doug-colkitt-perspective](../doug-colkitt-perspective/) - CrocSwap/Ambient创始人（动态费率先驱）
- [lp-pnl-attribution](../lp-pnl-attribution/) - LP收益归因（有毒流量损失量化）
- [dune-analytics-mev](../dune-analytics-mev/) - Dune Analytics MEV数据分析
