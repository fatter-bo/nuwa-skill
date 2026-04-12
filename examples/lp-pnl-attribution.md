---
name: lp-pnl-attribution
description: |
  LP收益归因分析框架。精确分解LP总P&L为手续费收入、无常损失/LVR、
  有毒流量损失、Gas成本、机会成本等组成部分。
  基于Paradigm LVR论文、Uniswap V3合约源码、Dune Analytics数据、
  Gauntlet/Panoptic量化研究深度调研。
  核心理念：LP的P&L不是一个数字——它是多个独立驱动因素的叠加，只有分解才能优化。
  触发词：「LP赚了多少」「LP收益分解」「无常损失多少」「fee收入分析」「LP归因」。
---

# LP收益归因分析 · 精确分解框架

> 说"LP亏了5%"没有意义。你需要知道：是fee不够多？是IL太大？是被套利bot吃了？还是Gas太贵？
> 只有精确分解每个P&L组件，才能找到优化方向。

## Skill规则

**此Skill激活后，以LP P&L归因分析师身份回应。**

- 所有计算基于链上数据，可复现
- P&L分解公式给出完整推导
- 附带Python代码和Dune SQL
- 区分事前（预测）和事后（回测）分析
- 不同市场环境（趋势/震荡）给出差异化分析

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「分解这个LP Position的P&L」 | -> 获取链上数据 → 逐项计算 → 输出完整归因表 |
| 「fee收入多少」 | -> 从feeGrowthGlobal计算精确fee |
| 「无常损失多少」 | -> V3 concentrated IL精确计算 |
| 「LVR怎么算」 | -> LVR数学推导 + 链上估算 |
| 「有毒流量损失占多少」 | -> 结合markout分析分解toxic flow component |
| 「这个策略Sharpe多少」 | -> 计算风险调整后收益 |
| 「趋势行情LP表现怎样」 | -> 分环境分析 + 历史回测 |

---

## 一、P&L分解模型

### 1.1 总P&L公式

```
LP_Total_PnL = V(T) - V(0)

其中：
  V(0) = 初始投入价值（包含token0 + token1的USD价值）
  V(T) = 终止时的总价值 = Position价值 + 累积fee（已claim + 未claim）

分解为：

LP_Total_PnL = Fee_Income 
             + Impermanent_Loss (负值)
             + Toxic_Flow_Loss (负值，IL的子集/补充)
             - Gas_Cost
             - Opportunity_Cost
```

### 1.2 各组件定义

```
1. Fee Income (手续费收入):
   LP从swap交易中获得的手续费
   来源: feeGrowthGlobal在Position范围内的累积
   性质: 始终为正

2. Impermanent Loss / LVR (无常损失):
   由于AMM价格更新滞后于市场价格，LP被动承受的损失
   = Position当前价值 - HODL同等初始token的价值
   性质: 始终为负或零

3. Toxic Flow Loss (有毒流量损失):
   被套利bot等知情交易者造成的额外损失
   = IL的"已实现"部分中由知情交易造成的
   注意: Toxic Flow Loss ⊂ IL/LVR, 不是独立组件
   但区分它有助于判断损失的结构性vs临时性

4. Gas Cost (Gas成本):
   - 主动管理LP: mint/burn/collect/rebalance的gas费用
   - 被动LP: 只有初始mint和最终burn
   - 在L2上可忽略，在L1上可能显著

5. Opportunity Cost (机会成本):
   - vs HODL: 持有同等token组合的收益
   - vs 无风险收益: 同期ETH staking / USDC lending yield
   - vs 指数: 同期BTC/ETH benchmark收益
```

### 1.3 P&L组件关系图

```
            LP Total Return
                  |
    ┌─────────────┴──────────────┐
    |                            |
  Position Value Change      Fee Income
    |                            |
    ├── Price Appreciation    ┌──┴──┐
    |   (token price change)  | Fee  |
    |                         └─────┘
    └── Impermanent Loss
        |
        ├── LVR (continuous)
        |   |
        |   ├── Toxic Flow (informed arb)
        |   └── Normal Rebalancing
        |
        └── Discrete IL
            (large price jumps)
```

---

## 二、精确计算方法

### 2.1 Fee Income计算

**从feeGrowthGlobal到Position fee：**

```python
def calculate_position_fees(
    fee_growth_global_0,      # 当前feeGrowthGlobal0X128
    fee_growth_global_1,      # 当前feeGrowthGlobal1X128
    fee_growth_outside_lower_0,  # tickLower的feeGrowthOutside0
    fee_growth_outside_lower_1,
    fee_growth_outside_upper_0,  # tickUpper的feeGrowthOutside0
    fee_growth_outside_upper_1,
    fee_growth_inside_last_0,    # Position上次记录的feeGrowthInside
    fee_growth_inside_last_1,
    liquidity,                   # Position的liquidity
    current_tick,
    tick_lower,
    tick_upper
):
    """
    精确计算Position的未收取手续费
    基于Uniswap V3合约的fee accounting逻辑
    """
    Q128 = 2**128
    
    # Step 1: 计算范围下方累积的fee
    if current_tick >= tick_lower:
        fee_below_0 = fee_growth_outside_lower_0
        fee_below_1 = fee_growth_outside_lower_1
    else:
        fee_below_0 = fee_growth_global_0 - fee_growth_outside_lower_0
        fee_below_1 = fee_growth_global_1 - fee_growth_outside_lower_1
    
    # Step 2: 计算范围上方累积的fee
    if current_tick < tick_upper:
        fee_above_0 = fee_growth_outside_upper_0
        fee_above_1 = fee_growth_outside_upper_1
    else:
        fee_above_0 = fee_growth_global_0 - fee_growth_outside_upper_0
        fee_above_1 = fee_growth_global_1 - fee_growth_outside_upper_1
    
    # Step 3: 计算范围内累积的fee
    fee_inside_0 = fee_growth_global_0 - fee_below_0 - fee_above_0
    fee_inside_1 = fee_growth_global_1 - fee_below_1 - fee_above_1
    
    # Step 4: 减去上次记录值，得到增量
    uncollected_0 = (fee_inside_0 - fee_growth_inside_last_0) * liquidity // Q128
    uncollected_1 = (fee_inside_1 - fee_growth_inside_last_1) * liquidity // Q128
    
    return uncollected_0, uncollected_1


# 使用web3从链上获取数据
def get_position_fees_onchain(w3, pool_address, token_id, position_manager_address):
    """
    从链上获取Position fee数据
    """
    # 通过NonfungiblePositionManager获取Position信息
    npm = w3.eth.contract(address=position_manager_address, abi=NPM_ABI)
    
    position = npm.functions.positions(token_id).call()
    tick_lower = position[5]
    tick_upper = position[6]
    liquidity = position[7]
    fee_growth_inside_last_0 = position[8]
    fee_growth_inside_last_1 = position[9]
    tokens_owed_0 = position[10]  # 已经claim但还没withdraw的
    tokens_owed_1 = position[11]
    
    # 获取Pool状态
    pool = w3.eth.contract(address=pool_address, abi=POOL_ABI)
    slot0 = pool.functions.slot0().call()
    current_tick = slot0[1]
    
    fee_growth_global_0 = pool.functions.feeGrowthGlobal0X128().call()
    fee_growth_global_1 = pool.functions.feeGrowthGlobal1X128().call()
    
    # 获取tick数据
    tick_lower_data = pool.functions.ticks(tick_lower).call()
    tick_upper_data = pool.functions.ticks(tick_upper).call()
    
    fee_growth_outside_lower_0 = tick_lower_data[2]
    fee_growth_outside_lower_1 = tick_lower_data[3]
    fee_growth_outside_upper_0 = tick_upper_data[2]
    fee_growth_outside_upper_1 = tick_upper_data[3]
    
    uncollected_0, uncollected_1 = calculate_position_fees(
        fee_growth_global_0, fee_growth_global_1,
        fee_growth_outside_lower_0, fee_growth_outside_lower_1,
        fee_growth_outside_upper_0, fee_growth_outside_upper_1,
        fee_growth_inside_last_0, fee_growth_inside_last_1,
        liquidity, current_tick, tick_lower, tick_upper
    )
    
    # 加上已claim未withdraw的
    total_0 = uncollected_0 + tokens_owed_0
    total_1 = uncollected_1 + tokens_owed_1
    
    return total_0, total_1
```

### 2.2 V3 Concentrated IL精确计算

```python
import math

def concentrated_il(p0, p1, pa, pb):
    """
    V3集中流动性无常损失精确计算
    
    p0: 初始价格 (token1/token0)
    p1: 当前价格
    pa: 范围下界价格
    pb: 范围上界价格
    
    返回: IL百分比 (负值表示损失)
    """
    assert pa < pb, "pa must be less than pb"
    assert pa <= p0 <= pb, "Initial price must be in range"
    
    sqrt_p0 = math.sqrt(p0)
    sqrt_p1 = math.sqrt(p1)
    sqrt_pa = math.sqrt(pa)
    sqrt_pb = math.sqrt(pb)
    
    # 初始token数量 (以L=1为单位)
    x0 = 1/sqrt_p0 - 1/sqrt_pb    # token0
    y0 = sqrt_p0 - sqrt_pa         # token1
    
    # 初始价值 (以token0计)
    v0 = x0 + y0 / p0  # 或以token1计: v0 = x0 * p0 + y0
    v0_in_token1 = x0 * p0 + y0
    
    # 当前token数量
    if p1 <= pa:
        # 全部变为token0
        x1 = 1/sqrt_pa - 1/sqrt_pb
        y1 = 0
    elif p1 >= pb:
        # 全部变为token1
        x1 = 0
        y1 = sqrt_pb - sqrt_pa
    else:
        x1 = 1/sqrt_p1 - 1/sqrt_pb
        y1 = sqrt_p1 - sqrt_pa
    
    # 当前LP价值 (以token1计)
    v_lp = x1 * p1 + y1
    
    # HODL价值 (以token1计)
    v_hodl = x0 * p1 + y0
    
    # IL = LP价值 / HODL价值 - 1
    il = v_lp / v_hodl - 1
    
    return {
        'il_pct': il,
        'v_lp': v_lp,
        'v_hodl': v_hodl,
        'il_absolute': v_lp - v_hodl,  # 以token1计的绝对损失
        'current_token0': x1,
        'current_token1': y1
    }


# 数值示例
result = concentrated_il(
    p0=3000,   # 初始ETH=3000 USDC
    p1=3500,   # 当前ETH=3500 USDC
    pa=2500,   # 范围下界
    pb=3500    # 范围上界
)
print(f"IL = {result['il_pct']:.4%}")
print(f"LP价值 = {result['v_lp']:.2f} USDC (per unit liquidity)")
print(f"HODL价值 = {result['v_hodl']:.2f} USDC")
print(f"绝对损失 = {result['il_absolute']:.2f} USDC")
```

### 2.3 LVR (Loss Versus Rebalancing) 数学推导

**LVR定义（Milionis et al. 2023）：**

```
LVR是LP相对于"连续再平衡"策略的损失。

直觉：如果你能以市场价格连续再平衡portfolio（无交易成本），
你的组合将始终和HODL一样。但AMM只在有人交易时才"再平衡"，
而且是以旧价格（而非市场价格）再平衡。
这个差距就是LVR。

数学：
对于恒定乘积AMM（V2），在连续时间模型下：

dLVR = (σ^2 / 2) * V * dt

其中：
  σ = 资产价格的瞬时波动率
  V = Pool中的总价值
  dt = 时间增量

离散形式（每个区块）：
LVR_block ≈ (ΔP/P)^2 * V / 2

总LVR = Σ LVR_block
```

**V3 Concentrated的LVR调整：**
```
V3的LVR = V2的LVR * capital_efficiency_multiplier

对于范围 [Pa, Pb]：
LVR_V3 = (σ^2 / 2) * V_virtual * dt

其中 V_virtual = V_actual * efficiency_multiplier
efficiency ≈ 1 / (1 - sqrt(Pa/Pb))

关键见解：集中流动性放大了fee收入，但同样放大了LVR
```

**Python LVR估算：**
```python
import numpy as np

def estimate_lvr(price_series, pool_value, dt_seconds=12):
    """
    从价格时间序列估算LVR
    
    price_series: 按区块的价格数组
    pool_value: Pool总价值 (USD)
    dt_seconds: 区块间隔（ETH约12秒）
    """
    log_returns = np.diff(np.log(price_series))
    
    # 每个区块的LVR
    block_lvr = (log_returns ** 2) * pool_value / 2
    
    # 总LVR
    total_lvr = np.sum(block_lvr)
    
    # 年化LVR率
    blocks_per_year = 365.25 * 24 * 3600 / dt_seconds
    total_blocks = len(log_returns)
    annualized_lvr_rate = total_lvr / pool_value * (blocks_per_year / total_blocks)
    
    # 实现波动率
    realized_vol = np.std(log_returns) * np.sqrt(blocks_per_year)
    
    return {
        'total_lvr_usd': total_lvr,
        'annualized_lvr_rate': annualized_lvr_rate,
        'realized_volatility': realized_vol,
        'theoretical_lvr_rate': realized_vol**2 / 2,  # 理论LVR = σ^2/2
        'blocks_analyzed': total_blocks
    }

# 示例
prices = np.array([3000, 3005, 2998, 3010, 3002, ...])  # 区块级价格
result = estimate_lvr(prices, pool_value=100_000_000)  # $1亿TVL
print(f"总LVR: ${result['total_lvr_usd']:,.2f}")
print(f"年化LVR率: {result['annualized_lvr_rate']:.2%}")
print(f"实现波动率: {result['realized_volatility']:.2%}")
```

### 2.4 区分IL与Toxic Flow Loss

```
IL和Toxic Flow Loss的关系：

IL（传统定义）= Position价值 - HODL价值
  这是一个"结果"指标，不区分原因

LVR = IL的"驱动力"分解
  LVR = Σ 每笔swap对LP造成的损失
  = Σ (交易量 * |价格偏离|)

Toxic Flow Loss = LVR中由知情交易造成的部分
  = Σ (知情交易的量 * |价格偏离|)

计算方法：
  1. 计算总LVR
  2. 用Markout分析分类每笔swap
  3. Toxic Flow Loss = 正markout交易造成的LVR子集

Python实现：
```

```python
def decompose_il_toxic(swaps_df, markout_threshold_bps=5):
    """
    将IL分解为有毒部分和非有毒部分
    
    swaps_df需包含：
    - execution_price: 执行价格
    - markout_5min_bps: 5分钟markout (bps)
    - amount_usd: 交易金额
    - price_impact_bps: 对池子价格的影响
    """
    # 分类
    swaps_df['is_toxic'] = swaps_df['markout_5min_bps'] > markout_threshold_bps
    
    # 每笔swap的LVR贡献
    # LVR ≈ |price_impact| * amount / 2
    swaps_df['lvr_contribution'] = (
        abs(swaps_df['price_impact_bps']) / 10000 * 
        swaps_df['amount_usd'] / 2
    )
    
    total_lvr = swaps_df['lvr_contribution'].sum()
    toxic_lvr = swaps_df[swaps_df['is_toxic']]['lvr_contribution'].sum()
    non_toxic_lvr = total_lvr - toxic_lvr
    
    # 有毒流量的额外信息
    toxic_volume = swaps_df[swaps_df['is_toxic']]['amount_usd'].sum()
    total_volume = swaps_df['amount_usd'].sum()
    
    return {
        'total_lvr_usd': total_lvr,
        'toxic_lvr_usd': toxic_lvr,
        'non_toxic_lvr_usd': non_toxic_lvr,
        'toxic_lvr_share': toxic_lvr / total_lvr if total_lvr > 0 else 0,
        'toxic_volume_share': toxic_volume / total_volume if total_volume > 0 else 0,
    }
```

---

## 三、时间序列归因分析

### 3.1 日/周/月级别P&L分解

```python
import pandas as pd

class LPPnLAttribution:
    def __init__(self, position_data, market_data, gas_costs):
        """
        position_data: Position的链上状态快照序列
        market_data: token价格时间序列
        gas_costs: 所有相关交易的gas费用
        """
        self.position = position_data
        self.market = market_data
        self.gas = gas_costs
    
    def daily_attribution(self):
        """计算每日P&L归因"""
        results = []
        
        for date in self.position['date'].unique():
            day_start = self.position[self.position['date'] == date].iloc[0]
            day_end = self.position[self.position['date'] == date].iloc[-1]
            
            # Fee收入
            fee_income = self._calculate_daily_fees(date)
            
            # IL
            il = self._calculate_daily_il(day_start, day_end)
            
            # Gas
            gas_cost = self.gas[self.gas['date'] == date]['cost_usd'].sum()
            
            # HODL基准
            hodl_return = self._calculate_hodl_return(day_start, day_end)
            
            # 机会成本 (vs ETH staking ~3.5% APR)
            daily_staking_yield = day_start['position_value_usd'] * 0.035 / 365
            opportunity_cost = daily_staking_yield
            
            # 总P&L
            total_pnl = fee_income + il - gas_cost
            
            # vs HODL
            excess_return = total_pnl - hodl_return
            
            results.append({
                'date': date,
                'fee_income_usd': fee_income,
                'il_usd': il,
                'gas_cost_usd': gas_cost,
                'opportunity_cost_usd': opportunity_cost,
                'total_pnl_usd': total_pnl,
                'hodl_return_usd': hodl_return,
                'excess_vs_hodl_usd': excess_return,
                'position_value_usd': day_end['position_value_usd']
            })
        
        return pd.DataFrame(results)
    
    def period_summary(self, daily_df, period='M'):
        """周/月汇总"""
        daily_df['period'] = pd.to_datetime(daily_df['date']).dt.to_period(period)
        
        summary = daily_df.groupby('period').agg({
            'fee_income_usd': 'sum',
            'il_usd': 'sum',
            'gas_cost_usd': 'sum',
            'opportunity_cost_usd': 'sum',
            'total_pnl_usd': 'sum',
            'excess_vs_hodl_usd': 'sum',
            'position_value_usd': 'last'
        })
        
        # 计算期间收益率
        summary['total_return_pct'] = summary['total_pnl_usd'] / summary['position_value_usd']
        summary['fee_yield_pct'] = summary['fee_income_usd'] / summary['position_value_usd']
        summary['il_pct'] = summary['il_usd'] / summary['position_value_usd']
        
        return summary
```

### 3.2 市场环境特征化

```
趋势行情 (Trending):
  特征: 连续多日单方向价格变动，波动率上升
  LP表现:
    - Fee收入增加（交易量通常增大）
    - IL显著增加（价格持续偏离）
    - 有毒流量占比上升（CEX-DEX价差持续存在）
    - 净效果: 通常亏损，尤其是窄range

震荡行情 (Range-Bound):
  特征: 价格在一定范围内波动
  LP表现:
    - Fee收入稳定
    - IL较小（价格回归）
    - 有毒流量占比较低
    - 净效果: 通常盈利，窄range更好

高波动行情 (High Volatility):
  特征: 大幅双向波动
  LP表现:
    - Fee收入很高（交易量暴增）
    - IL很高（大幅价格变动）
    - LVR极高（每个区块的价格变化大）
    - 净效果: 不确定，取决于fee vs IL平衡
```

```python
def classify_market_regime(prices, window=24*3600//12):
    """
    基于价格序列分类市场环境
    window: 区块数（默认约1天）
    """
    import numpy as np
    
    returns = np.diff(np.log(prices))
    
    # 滚动计算
    realized_vol = np.std(returns[-window:]) * np.sqrt(window)
    drift = np.mean(returns[-window:]) * window
    mean_reversion = np.corrcoef(returns[:-1], returns[1:])[0, 1]  # 自相关
    
    # 分类逻辑
    if abs(drift) > 2 * realized_vol:
        regime = 'TRENDING'
    elif mean_reversion < -0.3:
        regime = 'MEAN_REVERTING'  # 震荡/均值回归
    elif realized_vol > 0.03:  # 日波动率>3%
        regime = 'HIGH_VOLATILITY'
    else:
        regime = 'LOW_VOLATILITY'  # 低波动震荡
    
    return {
        'regime': regime,
        'realized_vol': realized_vol,
        'drift': drift,
        'mean_reversion_coefficient': mean_reversion
    }
```

---

## 四、量化评估指标

### 4.1 Sharpe Ratio

```python
def lp_sharpe_ratio(daily_returns, risk_free_rate_annual=0.035):
    """
    计算LP策略的Sharpe Ratio
    
    daily_returns: 每日P&L / Position价值
    risk_free_rate: 无风险利率（如ETH staking yield）
    """
    import numpy as np
    
    daily_rf = risk_free_rate_annual / 365
    excess_returns = daily_returns - daily_rf
    
    sharpe = np.mean(excess_returns) / np.std(excess_returns) * np.sqrt(365)
    
    return sharpe

# 典型LP策略Sharpe Ratio参考值
# ETH/USDC 0.3% 全范围被动LP: Sharpe ≈ 0.3-0.8
# ETH/USDC 0.3% ±10%范围被动LP: Sharpe ≈ 0.5-1.2
# ETH/USDC 0.05% 主动管理LP: Sharpe ≈ -0.5 to 0.5
# USDC/USDT 0.01% LP: Sharpe ≈ 1.0-3.0（正常市场）
```

### 4.2 Maximum Drawdown

```python
def lp_max_drawdown(cumulative_pnl_series):
    """
    计算LP策略的最大回撤
    """
    import numpy as np
    
    peak = np.maximum.accumulate(cumulative_pnl_series)
    drawdown = (cumulative_pnl_series - peak) / peak
    max_dd = np.min(drawdown)
    
    # 找到最大回撤的时间区间
    end_idx = np.argmin(drawdown)
    start_idx = np.argmax(cumulative_pnl_series[:end_idx])
    
    return {
        'max_drawdown': max_dd,
        'drawdown_start_idx': start_idx,
        'drawdown_end_idx': end_idx,
        'duration_days': (end_idx - start_idx)  # 需要转换为实际天数
    }
```

### 4.3 Benchmark比较

```python
def benchmark_comparison(lp_daily_pnl, position_value, token0_prices, token1_prices):
    """
    LP策略vs多个Benchmark比较
    """
    import numpy as np
    
    n_days = len(lp_daily_pnl)
    
    # Benchmark 1: HODL 50/50
    hodl_return = (token0_prices[-1]/token0_prices[0] + token1_prices[-1]/token1_prices[0]) / 2 - 1
    
    # Benchmark 2: 100% token1 (ETH)
    eth_return = token1_prices[-1] / token1_prices[0] - 1
    
    # Benchmark 3: 100% token0 (stablecoin)
    stable_return = 0.035 * n_days / 365  # 3.5% APR lending
    
    # Benchmark 4: ETH staking
    staking_return = eth_return + 0.035 * n_days / 365  # ETH price + staking yield
    
    lp_return = np.sum(lp_daily_pnl) / position_value[0]
    
    return {
        'lp_return': lp_return,
        'hodl_50_50_return': hodl_return,
        'eth_only_return': eth_return,
        'stable_lending_return': stable_return,
        'eth_staking_return': staking_return,
        'excess_vs_hodl': lp_return - hodl_return,
        'excess_vs_staking': lp_return - staking_return,
    }
```

### 4.4 Range宽度策略比较

```python
def compare_range_strategies(p0, price_series, fee_rate, volume_series, tvl):
    """
    比较不同range宽度的LP策略表现
    """
    ranges = [
        ('Full Range', 0, float('inf')),
        ('±50%', p0*0.5, p0*1.5),
        ('±20%', p0*0.8, p0*1.2),
        ('±10%', p0*0.9, p0*1.1),
        ('±5%', p0*0.95, p0*1.05),
        ('±2%', p0*0.98, p0*1.02),
    ]
    
    results = []
    for name, pa, pb in ranges:
        # 模拟该策略
        total_fee = 0
        total_il = 0
        in_range_pct = 0
        
        for i, (price, volume) in enumerate(zip(price_series, volume_series)):
            in_range = pa <= price <= pb
            if in_range:
                in_range_pct += 1
                # Fee (简化: 按流动性份额分配)
                cap_eff = 1 / (2 * (pb/pa - 1)) if pb != float('inf') else 1
                fee = fee_rate * volume * cap_eff / tvl
                total_fee += fee
        
        in_range_pct /= len(price_series)
        
        # IL at final price
        p_final = price_series[-1]
        il_result = concentrated_il(p0, p_final, pa, pb)
        total_il = il_result['il_pct']
        
        net_return = total_fee + total_il  # IL是负值
        
        results.append({
            'strategy': name,
            'range': f'[{pa:.0f}, {pb:.0f}]',
            'total_fee_pct': total_fee,
            'il_pct': total_il,
            'net_return_pct': net_return,
            'in_range_pct': in_range_pct,
        })
    
    return pd.DataFrame(results)
```

---

## 五、链上数据获取

### 5.1 Position生命周期数据

```
关键链上事件/数据源：

1. Mint事件 (Position创建):
   - NonfungiblePositionManager.IncreaseLiquidity
   - 获取: tokenId, liquidity, amount0, amount1
   - 时间戳即Position开始时间

2. Burn/Decrease事件 (Position缩减/关闭):
   - NonfungiblePositionManager.DecreaseLiquidity
   - 获取: tokenId, liquidity, amount0, amount1

3. Collect事件 (Fee领取):
   - NonfungiblePositionManager.Collect
   - 获取: tokenId, amount0, amount1
   - 累积所有Collect = 总已领取fee

4. 当前状态:
   - positions(tokenId) → 获取当前liquidity和feeGrowthInside
   - Pool.slot0() → 当前价格和tick
   - Pool.feeGrowthGlobal() → 全局fee累积
```

### 5.2 Subgraph查询

```graphql
# Uniswap V3 Subgraph查询Position历史
{
  position(id: "token_id_here") {
    id
    owner
    pool {
      id
      token0 { symbol decimals }
      token1 { symbol decimals }
      feeTier
    }
    tickLower { tickIdx }
    tickUpper { tickIdx }
    liquidity
    depositedToken0
    depositedToken1
    withdrawnToken0
    withdrawnToken1
    collectedFeesToken0
    collectedFeesToken1
    
    # 交易历史
    transaction {
      id
      timestamp
      blockNumber
    }
  }
}

# 查询Pool的swap历史（用于markout分析）
{
  swaps(
    where: { pool: "pool_address" }
    orderBy: timestamp
    orderDirection: desc
    first: 1000
  ) {
    id
    timestamp
    sender
    recipient
    amount0
    amount1
    sqrtPriceX96
    tick
    logIndex
  }
}
```

### 5.3 Dune SQL完整归因查询

```sql
-- 完整LP P&L归因查询（Uniswap V3, Dune Analytics）
-- 以特定tokenId为例

WITH position_info AS (
    -- 获取Position基本信息
    SELECT
        tokenId,
        pool,
        tickLower,
        tickUpper,
        block_time AS mint_time
    FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_IncreaseLiquidity
    WHERE tokenId = {{token_id}}
    ORDER BY block_time
    LIMIT 1
),

fee_claims AS (
    -- 所有fee领取
    SELECT
        SUM(amount0) AS total_fee_token0,
        SUM(amount1) AS total_fee_token1
    FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_Collect
    WHERE tokenId = {{token_id}}
),

deposits AS (
    -- 所有存入
    SELECT
        SUM(amount0) AS total_deposit_token0,
        SUM(amount1) AS total_deposit_token1
    FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_IncreaseLiquidity
    WHERE tokenId = {{token_id}}
),

withdrawals AS (
    -- 所有提取
    SELECT
        SUM(amount0) AS total_withdraw_token0,
        SUM(amount1) AS total_withdraw_token1
    FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_DecreaseLiquidity
    WHERE tokenId = {{token_id}}
),

gas_costs AS (
    -- 所有相关交易的Gas费用
    SELECT
        SUM(t.gas_used * t.gas_price / 1e18 * p.price) AS total_gas_usd
    FROM ethereum.transactions t
    JOIN prices.usd p ON p.symbol = 'ETH' AND p.minute = date_trunc('minute', t.block_time)
    WHERE t.hash IN (
        SELECT evt_tx_hash FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_IncreaseLiquidity WHERE tokenId = {{token_id}}
        UNION ALL
        SELECT evt_tx_hash FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_DecreaseLiquidity WHERE tokenId = {{token_id}}
        UNION ALL
        SELECT evt_tx_hash FROM uniswap_v3_ethereum.NonfungibleTokenPositionManager_evt_Collect WHERE tokenId = {{token_id}}
    )
)

SELECT
    d.total_deposit_token0,
    d.total_deposit_token1,
    w.total_withdraw_token0,
    w.total_withdraw_token1,
    f.total_fee_token0,
    f.total_fee_token1,
    g.total_gas_usd,
    -- P&L组件需要用价格转换为USD后计算
    -- IL = (当前Position价值 + 已提取 + 已领fee) - 初始存入在HODL下的当前价值
    -- 具体USD转换需要结合prices.usd表
    'See Python script for complete attribution' AS note
FROM deposits d, withdrawals w, fee_claims f, gas_costs g
```

---

## 六、决策模板

### 模板A：完整P&L归因报告

```
═══════════════════════════════════════════
     LP P&L Attribution Report
═══════════════════════════════════════════
Position: ETH/USDC 0.3% | Token ID: #12345
Period:   2025-01-01 to 2025-03-31 (90 days)
Range:    [2500, 3500] | Current: 3200

─── P&L Components ────────────────────
Fee Income:          +$1,250.00  (+2.50%)
Impermanent Loss:      -$850.00  (-1.70%)
  of which LVR:        -$720.00  (-1.44%)
  of which Toxic:      -$540.00  (-1.08%)  [75% of LVR]
Gas Cost:               -$45.00  (-0.09%)
─────────────────────────────────────────
Net P&L:              +$355.00  (+0.71%)
Annualized:            +2.88%

─── Benchmarks ────────────────────────
HODL 50/50:            +$480.00  (+0.96%)
ETH Only:              +$780.00  (+1.56%)
ETH Staking:           +$830.00  (+1.66%)
USDC Lending:          +$170.00  (+0.34%)

Excess vs HODL:        -$125.00  (-0.25%)
Excess vs Staking:     -$475.00  (-0.95%)

─── Risk Metrics ──────────────────────
Sharpe Ratio:          0.65
Max Drawdown:          -3.2%
In-Range %:            87.3%
═══════════════════════════════════════════
```

### 模板B：策略优化建议

```
基于归因分析的优化建议：

1. Fee收入充足，但IL > Fee:
   → 建议加宽range（减少IL放大效应）
   → 或切换到更高fee tier

2. 有毒流量占LVR > 70%:
   → 考虑使用动态费率Hook
   → 或切换到低毒交易对

3. Gas成本 > Fee收入的10%:
   → 减少rebalance频率
   → 使用L2（Arbitrum/Optimism）

4. In-Range < 70%:
   → 范围太窄，大量时间不赚fee
   → 建议扩大range 1.5-2倍

5. Sharpe < 0.5:
   → 风险调整后收益不佳
   → 考虑将资金转移到更优策略
```

---

## 七、常见误区

```
误区1: "我的LP显示+20%收益"
  → 可能只算了token数量变化而没算价格影响
  → 需要以USD计价对比HODL基准

误区2: "手续费APR有50%，肯定赚"
  → APR只是fee维度，没考虑IL
  → 高APR往往伴随高波动率 → 高IL

误区3: "无常损失在价格回来后就消失了"
  → 如果没有rebalance，理论上是的
  → 但套利bot已经"实现"了你的损失（LVR是持续的）
  → 价格回来后，你的本金已经被套利侵蚀

误区4: "窄range收益更高"
  → 窄range fee yield更高，但IL也放大
  → 而且需要更频繁rebalance → Gas成本
  → 必须做完整归因才能判断

误区5: "V3比V2对LP更有利"
  → V3给了LP更多工具，但也增加了复杂度
  → 被动全范围V3 ≈ V2
  → 窄range V3 = 高杠杆做市
```

---

## 相关技能

- [uniswap-v3-math-model](../uniswap-v3-math-model/) - V3数学模型（fee/IL公式推导）
- [lp-toxic-flow-analysis](../lp-toxic-flow-analysis/) - 有毒流量分析（toxic flow量化）
- [dan-robinson-perspective](../dan-robinson-perspective/) - Dan Robinson（LVR概念提出者之一）
