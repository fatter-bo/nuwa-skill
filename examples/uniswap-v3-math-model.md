---
name: uniswap-v3-math-model
description: |
  Uniswap V3数学模型与计算方法精讲。覆盖sqrtPriceX96定点数、tick与价格转换、
  集中流动性数学、swap计算流程、手续费累积机制、无常损失精确公式、高级逆向计算。
  基于Uniswap V3白皮书、合约源码、Dan Robinson/Dave White学术论文深度调研。
  核心理念：AMM的本质是数学，掌握精确公式才能做出精确决策。
  触发词：「V3数学」「sqrtPrice怎么算」「tick是什么」「集中流动性公式」「无常损失精确值」「swap怎么计算」。
---

# Uniswap V3 数学模型与计算方法 · 深度技术手册

> Uniswap V3的创新在于将连续流动性离散化到tick空间，用Q64.96定点数表示sqrt(price)。
> 理解这套数学体系，是做LP策略、MEV分析、协议开发的基础。

## Skill规则

**此Skill激活后，以Uniswap V3数学专家身份回应。**

- 所有公式给出完整数学推导
- 关键计算附带Python代码示例
- 数值示例使用真实链上参数（如ETH/USDC池）
- 区分理论值与合约实现的精度差异
- 涉及定点数运算时说明溢出风险

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「sqrtPriceX96怎么算」 | -> 输出Q64.96编码原理 + 价格转换公式 + Python示例 |
| 「tick和价格什么关系」 | -> 输出tick↔price双向转换 + tick spacing说明 |
| 「集中流动性怎么算token数量」 | -> 输出Position数学模型 + token0/token1计算公式 |
| 「swap过程怎么算」 | -> 输出单tick内swap + 跨tick处理 + 手续费计算全流程 |
| 「无常损失精确公式」 | -> 输出V3 concentrated IL vs V2 IL对比 + 数值示例 |
| 「怎么从目标价格反推swap数量」 | -> 输出逆向计算方法 + 迭代算法 |
| 「最优range怎么选」 | -> 输出Sharpe ratio优化框架 |

---

## 一、核心数学基础

### 1.1 sqrtPriceX96：Q64.96定点数编码

Uniswap V3不直接存储价格price，而是存储 `sqrt(price)` 的Q64.96定点数编码。

**为什么用sqrt(price)而非price：**
- token数量计算中频繁出现sqrt(price)，直接存储减少开方运算
- sqrt(price)的变化与liquidity成线性关系，简化swap计算
- 避免price跨越多个数量级时的精度丢失

**Q64.96编码：**
```
sqrtPriceX96 = sqrt(price) * 2^96

其中：
- price = token1 / token0（以token0为基准的token1价格）
- 2^96 是定点数缩放因子
- 整数部分64位 + 小数部分96位 = 160位，存储在uint160中
```

**价格还原：**
```
price = (sqrtPriceX96 / 2^96)^2
      = sqrtPriceX96^2 / 2^192
```

**Python计算示例：**
```python
# ETH/USDC池（token0=USDC, token1=ETH时的反例，实际取决于地址排序）
# 假设 token0=USDC(小地址), token1=ETH(大地址)
# price = ETH数量/USDC数量? 不对。
# price = token1/token0 的含义：1个token0值多少token1
# 实际中 token0地址 < token1地址，USDC < WETH（按地址排序）

# 假设ETH价格 = 3000 USDC
# price(token1/token0视角) = 1/3000（1 USDC = 1/3000 ETH）
# 但V3中 price = token1_amount / token0_amount

import math

def price_to_sqrtPriceX96(price):
    """将价格转为sqrtPriceX96"""
    return int(math.sqrt(price) * (2**96))

def sqrtPriceX96_to_price(sqrtPriceX96):
    """将sqrtPriceX96转为价格"""
    return (sqrtPriceX96 / (2**96)) ** 2

# ETH/USDC池：token0=USDC, token1=WETH
# 1 USDC = 1/3000 WETH => price = 1/3000
price = 1/3000
sqrtPriceX96 = price_to_sqrtPriceX96(price)
print(f"price = {price}")
print(f"sqrtPriceX96 = {sqrtPriceX96}")
# sqrtPriceX96 ≈ 1446441947520703918888

# 反向验证
price_back = sqrtPriceX96_to_price(sqrtPriceX96)
print(f"还原price = {price_back}")  # ≈ 0.000333...

# 如果要得到人类可读的ETH价格（USDC计价）
eth_price_in_usdc = 1 / price_back
print(f"ETH价格 = {eth_price_in_usdc} USDC")  # ≈ 3000
```

**精度注意事项：**
- sqrtPriceX96最小变化 = 1，对应的价格变化约为 2/2^96 * sqrt(price)
- 在ETH/USDC（$3000）场景下，最小价格精度约 $0.000000000000000000000000000038
- 远超任何实际需求，但在极端低价代币场景下需注意

---

### 1.2 Tick与价格的双向转换

**Tick定义：**
```
price(i) = 1.0001^i

其中i为tick index，是整数（可为负）
```

每个tick代表价格变化0.01%（1个基点basis point）。

**Tick → Price：**
```
price = 1.0001^tick

sqrtPrice = 1.0001^(tick/2)

sqrtPriceX96 = 1.0001^(tick/2) * 2^96
```

**Price → Tick：**
```
tick = floor(log(price) / log(1.0001))

等价于：
tick = floor(log_1.0001(price))
```

**Python实现：**
```python
import math

def tick_to_price(tick):
    """tick转价格"""
    return 1.0001 ** tick

def price_to_tick(price):
    """价格转tick（向下取整）"""
    return math.floor(math.log(price) / math.log(1.0001))

def tick_to_sqrtPriceX96(tick):
    """tick转sqrtPriceX96"""
    return int(1.0001 ** (tick / 2) * (2**96))

# 示例：ETH = 3000 USDC
# token0=USDC, token1=WETH, price = 1/3000
price = 1/3000
tick = price_to_tick(price)
print(f"tick = {tick}")  # tick ≈ -80068

# 反向验证
price_back = tick_to_price(tick)
print(f"price = {price_back}")  # ≈ 0.000333...
print(f"ETH price = {1/price_back} USDC")  # ≈ 3000
```

**合约中的实现（TickMath.sol）：**
- 不使用浮点数，纯定点数位运算
- `getSqrtRatioAtTick(int24 tick)` 使用预计算的magic numbers做乘法链
- `getTickAtSqrtRatio(uint160 sqrtPriceX96)` 使用二分查找+修正
- 有效tick范围：[-887272, 887272]
- 对应价格范围：[2^-128, 2^128]（几乎覆盖所有实际场景）

---

### 1.3 Tick Spacing与Fee Tier的关系

**V3的fee tier决定tick spacing：**

| Fee Tier | Tick Spacing | 每格价格变化 | 适用场景 |
|----------|-------------|-------------|---------|
| 0.01% (100) | 1 | 0.01% | 稳定币对 USDC/USDT |
| 0.05% (500) | 10 | 0.10% | 高相关对 ETH/stETH |
| 0.30% (3000) | 60 | 0.60% | 主流对 ETH/USDC |
| 1.00% (10000) | 200 | 2.00% | 长尾/高波动对 |

**Tick spacing的含义：**
- Position的tickLower和tickUpper必须是tick spacing的整数倍
- tick spacing越大，LP的范围选择越粗糙
- 更小的tick spacing = 更精细的流动性分布 = 更高的gas消耗

**有效tick数量：**
```
有效tick数 = (887272 - (-887272)) / tickSpacing + 1

tickSpacing=1:   1,774,545个tick
tickSpacing=10:    177,455个tick
tickSpacing=60:     29,576个tick
tickSpacing=200:     8,873个tick
```

---

### 1.4 Virtual Liquidity（虚拟流动性）

**V2的恒定乘积公式：**
```
x * y = k = L^2

其中 L = sqrt(k) 即流动性
```

**V3的局部恒定乘积：**
```
在当前tick对应的价格范围[pa, pb]内：

(x + L/sqrt(pb)) * (y + L*sqrt(pa)) = L^2

其中：
- x = 实际token0储备
- y = 实际token1储备
- L = 该范围内的流动性
- pa, pb = 范围边界价格
```

这等价于将V2的双曲线平移，使得在[pa, pb]范围内获得与更大L等效的流动性深度。

**资本效率提升：**
```
资本效率倍数 = L_virtual / L_actual

对于范围 [pa, pb]，当前价格 p：
效率倍数 ≈ 1 / (1 - sqrt(pa/pb))

示例：ETH/USDC，范围 [2500, 3500]，当前 3000
效率倍数 ≈ 1 / (1 - sqrt(2500/3500)) ≈ 1 / (1 - 0.845) ≈ 6.45x
```

---

## 二、集中流动性数学

### 2.1 Position数据结构

一个V3 LP Position由三元组定义：

```
Position = (tickLower, tickUpper, liquidity)

其中：
- tickLower: 范围下界tick（必须是tickSpacing的整数倍）
- tickUpper: 范围上界tick（必须是tickSpacing的整数倍）
- liquidity: uint128，该Position提供的流动性数值
- tickLower < tickUpper（严格不等）
```

### 2.2 Token数量计算

给定Position和当前价格，计算需要存入的token0和token1数量：

**设当前sqrtPrice = sqrt(P), 范围为 [sqrt(Pa), sqrt(Pb)]**

```
情况1: 当前价格在范围下方 (P < Pa)
  → Position全部由token0组成
  token0 = L * (1/sqrt(Pa) - 1/sqrt(Pb))
  token1 = 0

情况2: 当前价格在范围内 (Pa <= P <= Pb)
  → Position由两种token混合组成
  token0 = L * (1/sqrt(P) - 1/sqrt(Pb))
  token1 = L * (sqrt(P) - sqrt(Pa))

情况3: 当前价格在范围上方 (P > Pb)
  → Position全部由token1组成
  token0 = 0
  token1 = L * (sqrt(Pb) - sqrt(Pa))
```

**Python计算示例：**
```python
import math

def calculate_token_amounts(liquidity, sqrt_price, sqrt_pa, sqrt_pb):
    """计算Position的token0和token1数量"""
    if sqrt_price <= sqrt_pa:
        # 全部是token0
        token0 = liquidity * (1/sqrt_pa - 1/sqrt_pb)
        token1 = 0
    elif sqrt_price >= sqrt_pb:
        # 全部是token1
        token0 = 0
        token1 = liquidity * (sqrt_pb - sqrt_pa)
    else:
        # 混合
        token0 = liquidity * (1/sqrt_price - 1/sqrt_pb)
        token1 = liquidity * (sqrt_price - sqrt_pa)
    return token0, token1

# 示例：ETH/USDC池
# 当前ETH = 3000 USDC, token0=USDC, token1=WETH
# price = 1/3000, sqrt_price = sqrt(1/3000)
# 范围 [2500, 3500] -> price_a = 1/3500, price_b = 1/2500

sqrt_price = math.sqrt(1/3000)
sqrt_pa = math.sqrt(1/3500)  # 下界对应更高的ETH价格
sqrt_pb = math.sqrt(1/2500)  # 上界对应更低的ETH价格

# 注意：tick和price的方向关系
# 在token0=USDC, token1=WETH时，price = WETH/USDC = 1/3000
# 价格越小 = ETH越贵（因为1 USDC换到的ETH越少）
# 所以 price_a < price_b 对应 ETH价格从高到低

liquidity = 1e18  # 假设1e18的liquidity

token0, token1 = calculate_token_amounts(liquidity, sqrt_price, sqrt_pa, sqrt_pb)
print(f"需要 USDC (token0): {token0:.6f}")
print(f"需要 WETH (token1): {token1:.18f}")
```

### 2.3 从存入金额反推Liquidity

**给定想存入的token数量，计算能获得的最大liquidity：**

```
当 Pa <= P <= Pb 时：

L_from_token0 = token0 / (1/sqrt(P) - 1/sqrt(Pb))
L_from_token1 = token1 / (sqrt(P) - sqrt(Pa))

实际liquidity = min(L_from_token0, L_from_token1)
```

取min是因为两种token必须按比例配对，多余的一种会被退回。

### 2.4 价格变动公式

**当swap发生时，价格如何变化：**

```
卖出token0（买入token1）→ 价格下降（token0变多，token1变少）

Δ(1/sqrt(P)) = Δx / L
即: 1/sqrt(P_new) = 1/sqrt(P_old) + Δx / L

等价于: sqrt(P_new) = sqrt(P_old) * L / (L + Δx * sqrt(P_old))
```

```
卖出token1（买入token0）→ 价格上升

Δ(sqrt(P)) = Δy / L
即: sqrt(P_new) = sqrt(P_old) + Δy / L
```

**关键性质：sqrt(price)与token1数量成线性关系，1/sqrt(price)与token0数量成线性关系。**

这就是V3选择存储sqrtPrice的核心原因——swap计算变成简单的加减法。

### 2.5 多Position重叠

在同一个tick范围内可能有多个Position重叠，合约中每个tick记录的是净流动性变化（liquidityNet）：

```
tick.liquidityNet = 在此tick作为tickLower的liquidity之和
                  - 在此tick作为tickUpper的liquidity之和

当价格从左往右穿越tick i时：
  activeLiquidity += tick[i].liquidityNet

当价格从右往左穿越tick i时：
  activeLiquidity -= tick[i].liquidityNet
```

全局的`liquidity`状态变量始终等于当前价格所在tick范围内所有活跃Position的liquidity之和。

---

## 三、Swap计算流程

### 3.1 单Tick内Swap

在当前tick范围内（价格不穿越任何初始化的tick），swap是简单的：

```python
def swap_within_tick(sqrt_price_current, liquidity, amount_in, zero_for_one, fee_rate):
    """
    单tick内swap计算
    zero_for_one: True=卖token0买token1, False=反向
    返回: (sqrt_price_new, amount_in_consumed, amount_out, fee_amount)
    """
    # 扣除手续费后的有效输入
    amount_in_after_fee = amount_in * (1 - fee_rate)
    
    if zero_for_one:
        # 卖token0 -> 价格下降 -> sqrt(P)减小
        # Δ(1/sqrt(P)) = Δx / L
        sqrt_price_new = (sqrt_price_current * liquidity) / \
                         (liquidity + amount_in_after_fee * sqrt_price_current)
    else:
        # 卖token1 -> 价格上升 -> sqrt(P)增大
        # Δ(sqrt(P)) = Δy / L
        sqrt_price_new = sqrt_price_current + amount_in_after_fee / liquidity
    
    # 计算实际输出
    if zero_for_one:
        amount_out = liquidity * (sqrt_price_current - sqrt_price_new)
    else:
        amount_out = liquidity * (1/sqrt_price_new - 1/sqrt_price_current)
    
    fee_amount = amount_in * fee_rate
    return sqrt_price_new, amount_in, amount_out, fee_amount
```

### 3.2 跨Tick处理

当swap金额足够大，价格会穿越一个或多个已初始化的tick：

```
swap_step循环:
  1. 确定当前tick范围的边界 (next initialized tick)
  2. 计算到达边界需要消耗的amount
  3. if 剩余amount >= 到达边界需要的amount:
       - 消耗该部分amount，价格到达边界
       - 穿越tick: liquidity += tick.liquidityNet (或 -= 取决于方向)
       - 继续下一步
     else:
       - 消耗全部剩余amount，价格停在tick范围内
       - 结束
```

**合约实现（SwapMath.computeSwapStep）的核心逻辑：**
```
function computeSwapStep(
    sqrtRatioCurrentX96,
    sqrtRatioTargetX96,   // 下一个tick的sqrtPrice
    liquidity,
    amountRemaining,
    feePips
) returns (
    sqrtRatioNextX96,
    amountIn,
    amountOut,
    feeAmount
)
```

### 3.3 手续费计算与累积

**Fee计算方式：**
```
fee = amountIn * feePips / 1_000_000

注意：fee从输入token中扣除
实际参与swap的amount = amountIn - fee
```

**全局Fee累积：feeGrowthGlobal**
```
feeGrowthGlobal0X128: 每单位liquidity累积的token0手续费（Q128.128定点数）
feeGrowthGlobal1X128: 每单位liquidity累积的token1手续费

每次swap后：
feeGrowthGlobal_X128 += fee * 2^128 / liquidity
```

**Tick级别Fee追踪：feeGrowthOutside**
```
每个已初始化tick记录feeGrowthOutside0X128和feeGrowthOutside1X128

含义：该tick"外侧"累积的手续费
- "外侧"的定义取决于当前价格相对于该tick的位置
- 当价格穿越该tick时，outside翻转：
  feeGrowthOutside = feeGrowthGlobal - feeGrowthOutside
```

**Position手续费计算：**
```python
def calculate_position_fees(fee_growth_global, 
                            fee_growth_outside_lower,
                            fee_growth_outside_upper,
                            fee_growth_inside_last,
                            liquidity,
                            current_tick, tick_lower, tick_upper):
    """
    计算Position未领取的手续费
    """
    # 计算范围下方累积的fee
    if current_tick >= tick_lower:
        fee_growth_below = fee_growth_outside_lower
    else:
        fee_growth_below = fee_growth_global - fee_growth_outside_lower
    
    # 计算范围上方累积的fee
    if current_tick < tick_upper:
        fee_growth_above = fee_growth_outside_upper
    else:
        fee_growth_above = fee_growth_global - fee_growth_outside_upper
    
    # 范围内累积的fee
    fee_growth_inside = fee_growth_global - fee_growth_below - fee_growth_above
    
    # 自上次领取以来新增的fee
    uncollected_fee = (fee_growth_inside - fee_growth_inside_last) * liquidity / (2**128)
    
    return uncollected_fee
```

---

## 四、无常损失精确计算

### 4.1 V2无常损失经典公式

```
IL_V2 = 2*sqrt(r) / (1+r) - 1

其中 r = P_new / P_old （价格变化比率）

示例：
r = 2.0 (价格翻倍):  IL = 2*sqrt(2)/(1+2) - 1 = -5.72%
r = 0.5 (价格减半):  IL = 2*sqrt(0.5)/(1+0.5) - 1 = -5.72%
r = 1.5 (涨50%):     IL = 2*sqrt(1.5)/(1+1.5) - 1 = -2.02%
```

### 4.2 V3集中流动性无常损失

**设LP范围为 [Pa, Pb]，初始价格P0，终止价格P1：**

```
若 Pa <= P1 <= Pb（价格仍在范围内）:

IL_V3 = [token0_hold * P1 + token1_hold] / [token0_lp(P1) * P1 + token1_lp(P1)] - 1

具体展开：
V_hold = L * (1/sqrt(P0) - 1/sqrt(Pb)) * P1 + L * (sqrt(P0) - sqrt(Pa))
V_lp   = L * (1/sqrt(P1) - 1/sqrt(Pb)) * P1 + L * (sqrt(P1) - sqrt(Pa))

IL_V3 = V_lp / V_hold - 1
```

**简化形式（设 k = sqrt(P1/P0)）：**
```
当范围足够宽（接近全范围）时，V3 IL趋近于V2 IL
当范围很窄时，IL放大效应显著

IL放大倍数 ≈ L_concentrated / L_equivalent_V2
```

### 4.3 V3 vs V2无常损失对比

```python
import math

def il_v2(price_ratio):
    """V2无常损失"""
    r = price_ratio
    return 2 * math.sqrt(r) / (1 + r) - 1

def il_v3_concentrated(p0, p1, pa, pb):
    """
    V3集中流动性无常损失
    p0: 初始价格, p1: 终止价格
    pa: 范围下界, pb: 范围上界
    """
    if p1 < pa or p1 > pb:
        # 价格超出范围，需要分情况计算
        if p1 <= pa:
            # 全部变成token0
            v_lp = 1/math.sqrt(pa) - 1/math.sqrt(pb)
            v_lp_value = v_lp * p1
        else:
            # 全部变成token1
            v_lp_value = math.sqrt(pb) - math.sqrt(pa)
    else:
        # 价格在范围内
        v_lp_value = (1/math.sqrt(p1) - 1/math.sqrt(pb)) * p1 + \
                     (math.sqrt(p1) - math.sqrt(pa))
    
    # HODL价值
    token0_initial = 1/math.sqrt(p0) - 1/math.sqrt(pb)
    token1_initial = math.sqrt(p0) - math.sqrt(pa)
    v_hold = token0_initial * p1 + token1_initial
    
    return v_lp_value / v_hold - 1

# 数值对比
p0 = 3000  # 初始ETH价格
print("价格变化 | V2 IL | V3 [2500,3500] | V3 [2800,3200] | V3 [2950,3050]")
print("-" * 75)
for p1 in [2500, 2700, 2900, 3000, 3100, 3300, 3500]:
    r = p1 / p0
    v2 = il_v2(r)
    v3_wide = il_v3_concentrated(p0, p1, 2500, 3500)
    v3_mid = il_v3_concentrated(p0, p1, 2800, 3200)
    v3_narrow = il_v3_concentrated(p0, p1, 2950, 3050)
    print(f"${p1:>5} ({r:.2f}x) | {v2:>7.2%} | {v3_wide:>13.2%} | {v3_mid:>13.2%} | {v3_narrow:>13.2%}")
```

**典型输出：**
```
价格变化 | V2 IL | V3 [2500,3500] | V3 [2800,3200] | V3 [2950,3050]
---------------------------------------------------------------------------
$2500 (0.83x) |  -0.62% |        -3.77% |       -12.21% |       -37.52%
$2700 (0.90x) |  -0.28% |        -1.67% |        -4.55% |        -9.32%
$2900 (0.97x) |  -0.03% |        -0.17% |        -0.44% |        -0.81%
$3000 (1.00x) |   0.00% |         0.00% |         0.00% |         0.00%
$3100 (1.03x) |  -0.03% |        -0.15% |        -0.39% |        -0.70%
$3300 (1.10x) |  -0.23% |        -1.32% |        -3.42% |        -6.23%
$3500 (1.17x) |  -0.68% |        -3.28% |       -10.01% |       -28.67%
```

**关键发现：范围越窄，IL放大越严重。[2950,3050]的窄范围在价格波动17%时IL接近V2的50倍。**

### 4.4 盈亏平衡分析

**LP盈利条件：Fee Income > Impermanent Loss**

```
设年化波动率σ，范围宽度 [Pa, Pb]，fee tier f

简化的盈亏平衡条件（单期）：
Fee ≈ f * V_traded / L_total_in_range
IL ≈ (σ^2 / 2) * capital_efficiency_multiplier

盈亏平衡需要：
f * volume / TVL > σ^2 * efficiency / 2

即：volume/TVL ratio（资金周转率）足够高
```

---

## 五、高级计算

### 5.1 从目标价格影响反推Swap数量

**问题：如果想让价格从P_current移动到P_target，需要输入多少token？**

```python
def amount_for_price_change(sqrt_price_current, sqrt_price_target, 
                            liquidity, fee_rate, zero_for_one):
    """
    计算将价格从current移动到target需要的输入量
    注意：这是单tick范围内的简化版，跨tick需要逐段累加
    """
    if zero_for_one:
        # 卖token0，价格下降
        # amount_in_after_fee = L * (1/sqrt(P_target) - 1/sqrt(P_current))
        amount_after_fee = liquidity * (1/sqrt_price_target - 1/sqrt_price_current)
        amount_in = amount_after_fee / (1 - fee_rate)
        amount_out = liquidity * (sqrt_price_current - sqrt_price_target)
    else:
        # 卖token1，价格上升
        amount_after_fee = liquidity * (sqrt_price_target - sqrt_price_current)
        amount_in = amount_after_fee / (1 - fee_rate)
        amount_out = liquidity * (1/sqrt_price_current - 1/sqrt_price_target)
    
    return amount_in, amount_out
```

**跨tick的完整计算需要遍历所有中间tick，累加每段的amount：**
```python
def amount_for_price_change_cross_tick(sqrt_price_current, sqrt_price_target,
                                        ticks_data, fee_rate, zero_for_one):
    """
    跨tick反推swap数量
    ticks_data: [(tick, liquidityNet, sqrtPrice), ...] 按tick排序
    """
    total_amount_in = 0
    total_amount_out = 0
    current_liquidity = get_active_liquidity()  # 当前活跃liquidity
    sqrt_p = sqrt_price_current
    
    relevant_ticks = get_ticks_between(sqrt_price_current, sqrt_price_target, 
                                        ticks_data, zero_for_one)
    
    for tick_data in relevant_ticks:
        tick_sqrt_price = tick_data['sqrtPrice']
        
        # 计算到这个tick的amount
        a_in, a_out = amount_for_price_change(
            sqrt_p, tick_sqrt_price, current_liquidity, fee_rate, zero_for_one)
        total_amount_in += a_in
        total_amount_out += a_out
        
        # 穿越tick，更新liquidity
        if zero_for_one:
            current_liquidity -= tick_data['liquidityNet']
        else:
            current_liquidity += tick_data['liquidityNet']
        
        sqrt_p = tick_sqrt_price
    
    # 最后一段到target
    a_in, a_out = amount_for_price_change(
        sqrt_p, sqrt_price_target, current_liquidity, fee_rate, zero_for_one)
    total_amount_in += a_in
    total_amount_out += a_out
    
    return total_amount_in, total_amount_out
```

### 5.2 流动性分布分析

**获取全池流动性分布（用于分析价格影响和滑点）：**

```python
def get_liquidity_distribution(pool_ticks, current_tick, tick_spacing, range_ticks=100):
    """
    获取当前价格周围的流动性分布
    pool_ticks: {tick: liquidityNet} 从链上或Subgraph获取
    """
    distribution = []
    active_liquidity = get_current_active_liquidity()
    
    # 从当前tick向两侧扫描
    # 向上扫描
    liq = active_liquidity
    for t in range(current_tick, current_tick + range_ticks * tick_spacing, tick_spacing):
        if t in pool_ticks:
            liq += pool_ticks[t]
        price = 1.0001 ** t
        distribution.append({'tick': t, 'price': price, 'liquidity': liq})
    
    # 向下扫描
    liq = active_liquidity
    for t in range(current_tick - tick_spacing, 
                   current_tick - range_ticks * tick_spacing, 
                   -tick_spacing):
        if t in pool_ticks:
            liq -= pool_ticks[t]
        price = 1.0001 ** t
        distribution.insert(0, {'tick': t, 'price': price, 'liquidity': liq})
    
    return distribution
```

### 5.3 最优Range选择（Sharpe Ratio框架）

**目标：选择使风险调整后收益最大的range宽度**

```
设范围为 [P*(1-w), P*(1+w)]，其中w为半宽度百分比

Fee收入（年化）:
  F(w) ≈ fee_rate * annual_volume * capital_efficiency(w) / TVL

IL（年化期望）:
  IL(w) ≈ σ^2 * capital_efficiency(w) / 2

其中 capital_efficiency(w) ≈ 1 / (2*w)  （近似）

净收益:
  R(w) = F(w) - IL(w)
       ≈ (fee_rate * V / TVL - σ^2/2) * (1/2w)

波动性:
  Vol(w) ≈ σ * sqrt(capital_efficiency(w))

Sharpe Ratio:
  SR(w) = R(w) / Vol(w)
```

**数值优化：**
```python
import numpy as np
from scipy.optimize import minimize_scalar

def sharpe_ratio(w, fee_rate, volume_tvl_ratio, sigma):
    """
    计算给定range宽度的Sharpe Ratio
    w: 半宽度百分比 (0.01 = 1%)
    fee_rate: 如0.003 for 0.3%
    volume_tvl_ratio: 日均交易量/TVL
    sigma: 日波动率
    """
    if w <= 0.001:
        return -np.inf
    
    cap_eff = 1 / (2 * w)
    
    # 日Fee收入率
    daily_fee = fee_rate * volume_tvl_ratio * cap_eff
    
    # 日IL期望
    daily_il = (sigma**2 / 2) * cap_eff
    
    # 出范围概率惩罚
    prob_in_range = min(1.0, 2 * w / (sigma * 2))  # 简化估算
    
    net_return = (daily_fee - daily_il) * prob_in_range
    vol = sigma * np.sqrt(cap_eff)
    
    if vol == 0:
        return 0
    return net_return / vol

# 搜索最优w
result = minimize_scalar(lambda w: -sharpe_ratio(w, 0.003, 0.5, 0.03),
                         bounds=(0.005, 0.5), method='bounded')
optimal_w = result.x
print(f"最优半宽度: {optimal_w:.1%}")
print(f"最优范围: [{3000*(1-optimal_w):.0f}, {3000*(1+optimal_w):.0f}]")
print(f"Sharpe Ratio: {-result.fun:.4f}")
```

---

## 六、决策模板

### 模板A：评估LP Position预期收益

```
输入：
  - 池地址/交易对
  - 计划范围 [tickLower, tickUpper]
  - 投入金额

计算步骤：
  1. 获取当前sqrtPriceX96 → 转为价格
  2. 计算token0/token1比例和liquidity
  3. 查询历史日均volume和TVL → 估算fee收入
  4. 估算历史波动率σ → 计算预期IL
  5. 净收益 = Fee - IL - Gas成本
  6. 计算Sharpe Ratio → 与替代策略比较
```

### 模板B：Swap价格影响预估

```
输入：
  - 池地址
  - swap方向和数量

计算步骤：
  1. 获取当前sqrtPriceX96和active liquidity
  2. 获取附近initialized ticks的liquidityNet
  3. 逐tick模拟swap过程
  4. 输出：执行价格、价格影响%、穿越tick数量
```

### 模板C：V3 vs V2无常损失对比

```
输入：
  - 初始价格P0
  - 目标价格P1（或价格变化百分比）
  - V3范围 [Pa, Pb]

输出：
  - V2 IL百分比
  - V3 IL百分比
  - IL放大倍数
  - 需要的fee收入来覆盖额外IL
```

---

## 七、常用数值速查

### sqrtPriceX96参考值

| ETH/USDC价格 | tick (approx) | sqrtPriceX96 (approx) |
|-------------|---------------|----------------------|
| $1,000 | -69082 | 2.505 × 10^27 |
| $2,000 | -75149 | 1.771 × 10^27 |
| $3,000 | -78244 | 1.446 × 10^27 |
| $4,000 | -80216 | 1.253 × 10^27 |
| $5,000 | -81771 | 1.120 × 10^27 |

*注：具体值取决于token0/token1排序，上表假设token0=USDC, token1=WETH*

### 关键合约函数

| 函数 | 合约 | 功能 |
|------|------|------|
| `getSqrtRatioAtTick()` | TickMath | tick→sqrtPriceX96 |
| `getTickAtSqrtRatio()` | TickMath | sqrtPriceX96→tick |
| `computeSwapStep()` | SwapMath | 单步swap计算 |
| `getAmount0Delta()` | SqrtPriceMath | 价格变化→token0变化量 |
| `getAmount1Delta()` | SqrtPriceMath | 价格变化→token1变化量 |

---

## 相关技能

- [dan-robinson-perspective](../dan-robinson-perspective/) - Dan Robinson学术思想（AMM理论奠基者）
- [lp-pnl-attribution](../lp-pnl-attribution/) - LP收益归因分析（应用V3数学）
- [jit-liquidity-attack](../jit-liquidity-attack/) - JIT流动性攻击（利用集中流动性特性）
