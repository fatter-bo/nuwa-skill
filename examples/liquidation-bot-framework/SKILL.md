---
name: liquidation-bot-framework
description: |
  借贷协议清算bot专家。涵盖Aave V3、Compound V3、MakerDAO的清算机制对比、
  仓位监控（Health Factor实时计算、预言机监控、大额仓位追踪、临近清算告警）、
  清算策略（闪电贷 vs 自有资金、抵押品处置路径、多协议清算、gas竞价和bundle）、
  级联清算与系统性风险分析、竞争格局。
  包含Health Factor公式、Aave V3清算Solidity接口示例、协议参数对比表。
  当用户问「清算bot怎么做」「Health Factor是什么」「Aave清算机制」时使用。
---

# 借贷协议清算Bot框架

> 清算是DeFi借贷协议的免疫系统——清算bot既是利润追逐者，也是协议安全的守护者。

## 使用说明

**本Skill聚焦DeFi借贷协议的清算机制和Bot构建。** 当用户询问清算策略、Health Factor、借贷协议风险时，按对应章节提供分析。

---

## 一、清算机制基础

### 1.1 为什么需要清算

```
DeFi借贷的基本模型：
1. 用户存入抵押品（如ETH）
2. 用户借出资产（如USDC）
3. 借贷比率（LTV）必须低于清算阈值
4. 当抵押品价值下跌（或借出资产升值）时，LTV上升
5. LTV超过清算阈值 → 仓位可被清算

清算的作用：
- 保护协议免受坏账
- 保护存款人的资金安全
- 维护协议的偿付能力

清算激励：
- 清算人可以以折扣价购买被清算仓位的抵押品
- 折扣通常为5-15%（取决于协议和资产）
```

### 1.2 主要协议清算机制对比

| 特性 | Aave V3 | Compound V3 (Comet) | MakerDAO (Liquidations 2.0) |
|------|---------|--------------------|-----------------------------|
| **清算模式** | 即时折扣清算 | 即时折扣清算 | 荷兰拍卖清算 |
| **清算比例** | 单次最多50%的债务 | 单次可清算全部（absorb） | 拍卖完整仓位 |
| **清算折扣** | 4-10%（取决于资产） | 5-15% | 拍卖起始价为市价，递减 |
| **清算触发** | Health Factor < 1 | borrowBalance > maxBorrow | CR < liquidation ratio |
| **闪电贷支持** | 原生支持（同协议） | 需要外部闪电贷 | 需要外部资金或闪电贷 |
| **Gas消耗** | ~400K-700K gas | ~300K-500K gas | ~500K-1M gas（拍卖模式） |
| **竞争模式** | gas竞价/Bundle | gas竞价/Bundle | 拍卖出价竞争 |

---

## 二、Health Factor深入分析

### 2.1 Aave V3 Health Factor公式

```
Health Factor (HF) = Σ(Collateral_i * Price_i * LiquidationThreshold_i) / Σ(Debt_j * Price_j)

其中：
- Collateral_i: 第i种抵押品的数量
- Price_i: 第i种抵押品的价格（来自Chainlink预言机）
- LiquidationThreshold_i: 第i种抵押品的清算阈值
- Debt_j: 第j种借款的数量（含累计利息）
- Price_j: 第j种借款的价格

当 HF < 1 时，仓位可被清算。

详细计算示例：
用户仓位：
- 抵押品: 10 ETH, 价格 $2,000, LiquidationThreshold = 82.5%
- 借款: 12,000 USDC, 价格 $1.00

HF = (10 * 2000 * 0.825) / (12000 * 1.0)
   = 16,500 / 12,000
   = 1.375

当ETH跌至 $1,454.55 时：
HF = (10 * 1454.55 * 0.825) / 12000
   = 12,000 / 12,000
   = 1.00 → 可以被清算
```

### 2.2 Compound V3 (Comet) 清算条件

```
Compound V3使用不同的模型：

isBorrowCollateralized(account):
  borrowBalance = 用户的借款总额（含利息）
  maxBorrow = Σ(collateral_i * price_i * borrowCollateralFactor_i)
  
  if borrowBalance > maxBorrow:
    该仓位可被清算

isLiquidatable(account):
  borrowBalance = 用户的借款总额
  liquidationCollateral = Σ(collateral_i * price_i * liquidateCollateralFactor_i)
  
  if borrowBalance > liquidationCollateral:
    该仓位可被清算

关键区别：
- Compound V3每个市场只有一种借款资产（通常是USDC）
- 使用两个不同的factor: borrowCollateralFactor 和 liquidateCollateralFactor
- liquidateCollateralFactor > borrowCollateralFactor（清算阈值更高）
```

### 2.3 MakerDAO清算比率

```
MakerDAO使用Collateralization Ratio (CR):

CR = (Collateral_Value) / (Debt_Value + Stability_Fee)
   = (Collateral * Oracle_Price) / (DAI_Debt + Accumulated_Fee)

当 CR < Liquidation_Ratio 时，仓位（Vault）可被清算。

不同Vault类型的Liquidation Ratio:
- ETH-A: 145%
- ETH-B: 130%（更高杠杆，更容易被清算）
- ETH-C: 170%（更保守）
- WBTC-A: 145%
- WSTETH-A: 160%

MakerDAO Liquidations 2.0（荷兰拍卖）：
1. 清算触发后，抵押品进入拍卖
2. 起始价 = 预言机价格 * buf（通常1.2倍）
3. 价格按指数函数递减: price(t) = top * (cut^t)
4. 任何人可以在任意价格点出价购买
5. 拍卖有最长时间限制（tail参数）
```

### 2.4 实时Health Factor监控

```
监控架构：

方案1: 事件驱动监控
┌──────────────────────────────┐
│ Chainlink Price Feed Events  │
│ (价格变动事件)                │
└──────────┬───────────────────┘
           │ 触发
           ▼
┌──────────────────────────────┐
│ Health Factor重新计算引擎     │
│ - 维护所有仓位的本地缓存      │
│ - 增量更新受影响仓位的HF      │
└──────────┬───────────────────┘
           │ HF < 阈值
           ▼
┌──────────────────────────────┐
│ 清算执行引擎                  │
│ - 利润计算                    │
│ - 交易构造和提交               │
└──────────────────────────────┘

方案2: 轮询监控
- 每个区块查询目标仓位的HF
- 使用multicall批量查询减少RPC调用
- 适合监控少量大额仓位

方案3: 混合模式
- 大额仓位: 事件驱动 + 轮询双重监控
- 小额仓位: 仅在价格大幅波动时检查
```

---

## 三、清算策略详解

### 3.1 Aave V3清算接口

```solidity
// Aave V3 Pool清算接口
interface IPool {
    /**
     * @notice 清算一个不健康的仓位
     * @param collateralAsset 要接收的抵押品地址
     * @param debtAsset 要偿还的债务资产地址
     * @param user 被清算的用户地址
     * @param debtToCover 要偿还的债务金额
     * @param receiveAToken 是否接收aToken而非底层资产
     */
    function liquidationCall(
        address collateralAsset,
        address debtAsset,
        address user,
        uint256 debtToCover,
        bool receiveAToken
    ) external;
}

// 清算参数计算
function calculateLiquidationParams(
    address user,
    address collateralAsset,
    address debtAsset
) {
    // 1. 获取用户仓位数据
    (totalCollateralBase, totalDebtBase, , , , healthFactor) = 
        pool.getUserAccountData(user);
    
    require(healthFactor < 1e18, "Position not liquidatable");
    
    // 2. 计算最大可清算金额
    // Aave V3: 当HF < 0.95时可以清算100%，否则最多50%
    uint256 maxLiquidatable;
    if (healthFactor < 0.95e18) {
        maxLiquidatable = userDebtBalance; // 100% close factor
    } else {
        maxLiquidatable = userDebtBalance / 2; // 50% close factor
    }
    
    // 3. 计算获得的抵押品
    // liquidationBonus 通常为 105%-110%
    uint256 collateralReceived = debtToCover * debtPrice / collateralPrice 
                                  * liquidationBonus / 10000;
    
    // 4. 利润 = collateralReceived的市场价值 - debtToCover - gas
    uint256 profit = collateralReceived * collateralPrice 
                     - debtToCover * debtPrice 
                     - gasCost;
}
```

### 3.2 闪电贷清算流程

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@aave/v3-core/contracts/flashloan/base/FlashLoanSimpleReceiverBase.sol";
import "@aave/v3-core/contracts/interfaces/IPool.sol";
import "@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol";

contract AaveLiquidator is FlashLoanSimpleReceiverBase {
    ISwapRouter public immutable swapRouter;
    address public owner;

    struct LiquidationParams {
        address collateralAsset;
        address debtAsset;
        address user;
        uint256 debtToCover;
        bool receiveAToken;
    }

    constructor(
        address _poolProvider,
        address _swapRouter
    ) FlashLoanSimpleReceiverBase(IPoolAddressesProvider(_poolProvider)) {
        swapRouter = ISwapRouter(_swapRouter);
        owner = msg.sender;
    }

    function executeLiquidation(
        LiquidationParams calldata params
    ) external {
        require(msg.sender == owner, "Not owner");

        // 步骤1: 借入债务资产（闪电贷）
        bytes memory data = abi.encode(params);
        POOL.flashLoanSimple(
            address(this),
            params.debtAsset,
            params.debtToCover,
            data,
            0
        );
    }

    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external override returns (bool) {
        require(msg.sender == address(POOL), "Not pool");

        LiquidationParams memory liqParams = abi.decode(
            params, (LiquidationParams)
        );

        // 步骤2: 执行清算
        IERC20(asset).approve(address(POOL), amount);
        POOL.liquidationCall(
            liqParams.collateralAsset,
            liqParams.debtAsset,
            liqParams.user,
            liqParams.debtToCover,
            liqParams.receiveAToken
        );

        // 步骤3: 将获得的抵押品swap为债务资产
        uint256 collateralBalance = IERC20(liqParams.collateralAsset)
            .balanceOf(address(this));

        IERC20(liqParams.collateralAsset).approve(
            address(swapRouter), collateralBalance
        );

        uint256 amountOut = swapRouter.exactInputSingle(
            ISwapRouter.ExactInputSingleParams({
                tokenIn: liqParams.collateralAsset,
                tokenOut: asset,
                fee: 3000, // 0.3%
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: collateralBalance,
                amountOutMinimum: amount + premium, // 至少覆盖闪电贷
                sqrtPriceLimitX96: 0
            })
        );

        // 步骤4: 偿还闪电贷
        uint256 totalOwed = amount + premium;
        IERC20(asset).approve(address(POOL), totalOwed);

        // 剩余利润留在合约中
        // profit = amountOut - totalOwed
        return true;
    }

    function withdraw(address token) external {
        require(msg.sender == owner, "Not owner");
        uint256 balance = IERC20(token).balanceOf(address(this));
        IERC20(token).transfer(owner, balance);
    }
}
```

### 3.3 闪电贷 vs 自有资金对比

| 维度 | 闪电贷清算 | 自有资金清算 |
|------|-----------|------------|
| **资金需求** | 几乎为零（只需gas费） | 需要持有足够的债务资产 |
| **Gas消耗** | 更高（闪电贷+swap额外gas） | 更低（直接清算） |
| **速度** | 稍慢（更多步骤） | 更快（步骤更少） |
| **复杂度** | 高（需要处理闪电贷回调+swap） | 低（直接调用liquidationCall） |
| **利润率** | 较低（闪电贷费用+swap滑点） | 较高 |
| **适用场景** | 大额清算、无需预先资金 | 小额高频清算、时间敏感 |
| **风险** | 低（原子性保证） | 持有资产的价格风险 |

### 3.4 抵押品处置路径优化

```
清算后获得抵押品，需要选择最优的售出路径：

路径选择策略：
1. 直接DEX swap（最简单）
   - Uniswap V3最优路径
   - 适用于主流资产（ETH, WBTC, stETH）

2. 多跳路径
   - wstETH → ETH → USDC
   - 某些代币没有直接交易对

3. 聚合器（1inch, Paraswap）
   - 自动找最优路径和分拆
   - 适用于流动性分散的资产

4. 不swap，直接持有
   - 接收aToken（Aave）可以继续获得存款收益
   - 适合预期价格会反弹的场景

5. OTC处置
   - 极大额清算时DEX流动性不足
   - 通过Cowswap等RFQ系统获得更好价格

// 路径优化算法
function findBestDisposalPath(
    collateralAsset,
    targetAsset,
    amount
) {
    paths = [
        directSwap(collateralAsset, targetAsset, amount),
        multiHopSwap(collateralAsset, WETH, targetAsset, amount),
        aggregatorQuote("1inch", collateralAsset, targetAsset, amount),
        aggregatorQuote("paraswap", collateralAsset, targetAsset, amount),
    ];
    
    return max(paths, key=netReturn);
}
```

---

## 四、大额仓位监控

### 4.1 目标仓位发现

```
发现高价值清算目标的方法：

1. 链上事件扫描
   - 监听Borrow, Supply, Repay, Withdraw事件
   - 维护所有活跃仓位的状态

2. 按HF排序
   - 定期计算所有仓位的HF
   - 维护一个按HF排序的优先队列
   - 重点关注 HF < 1.5 的仓位

3. 大额仓位追踪
   - 借款金额 > $100K 的仓位
   - 这些仓位的清算利润最高

4. 使用SubGraph/Dune
   - Aave SubGraph: 实时查询仓位数据
   - Dune: 历史分析和趋势
```

### 4.2 预言机监控

```
价格预言机是清算的触发器：

Chainlink价格更新机制：
1. 偏差触发: 价格偏离 > 0.5%（主流资产）或 > 1%（其他）时更新
2. 心跳触发: 即使价格未变，每3600秒（1小时）也会更新
3. 多节点聚合: 多个节点报价取中位数

监控策略：
- 监听Chainlink Aggregator的AnswerUpdated事件
- 在价格更新前预判（监控链下价格变动）
- 提前计算哪些仓位将在下次价格更新时变为可清算

// 预判清算的逻辑
function predictLiquidations(priceChangePercent, asset) {
    // 获取所有使用该资产作为抵押品的仓位
    positions = getPositionsByCollateral(asset);
    
    for (pos in positions) {
        newPrice = pos.collateralPrice * (1 + priceChangePercent);
        newHF = calculateHF(pos, newPrice);
        
        if (newHF < 1.0) {
            // 该仓位将在价格更新后可清算
            addToPendingLiquidations(pos);
        }
    }
}
```

### 4.3 临近清算告警系统

```
多级告警体系：

Level 1 (Watch): HF < 1.3
  → 添加到重点监控列表
  → 增加轮询频率（每区块检查）

Level 2 (Alert): HF < 1.1
  → 预先准备清算交易
  → 预计算最优清算参数
  → 预先approve代币

Level 3 (Ready): HF < 1.05
  → 持续提交Bundle（ETH L1）或交易（L2）
  → 动态调整gas价格

Level 4 (Execute): HF < 1.0
  → 立即执行清算
  → 最高优先级gas竞价

// 告警配置示例
const alertConfig = {
    levels: [
        { name: "WATCH",   hfThreshold: 1.3,  action: "monitor" },
        { name: "ALERT",   hfThreshold: 1.1,  action: "prepare" },
        { name: "READY",   hfThreshold: 1.05, action: "submit" },
        { name: "EXECUTE", hfThreshold: 1.0,  action: "liquidate" }
    ],
    minProfitUSD: 50,        // 最小利润阈值
    maxGasGwei: 200,         // 最大gas价格
    pollIntervalMs: 1000,    // 轮询间隔
    notificationChannels: ["telegram", "discord", "pagerduty"]
};
```

---

## 五、协议参数对比

### 5.1 Aave V3主要资产参数

| 资产 | LTV | 清算阈值 | 清算Bonus | E-Mode LTV | E-Mode清算阈值 |
|------|-----|---------|-----------|-----------|---------------|
| ETH | 80% | 82.5% | 5% | 90% (ETH correlated) | 93% |
| WBTC | 73% | 78% | 6.25% | - | - |
| USDC | 77% | 80% | 4.5% | 97% (Stablecoin) | 97.5% |
| USDT | 75% | 78% | 4.5% | 97% (Stablecoin) | 97.5% |
| wstETH | 79% | 81% | 7% | 90% (ETH correlated) | 93% |
| DAI | 67% | 77% | 4% | 97% (Stablecoin) | 97.5% |
| LINK | 68% | 73% | 7% | - | - |

### 5.2 E-Mode（效率模式）的影响

```
Aave V3 E-Mode允许相关资产之间的高杠杆借贷：

ETH correlated E-Mode:
- 可用资产: ETH, wstETH, rETH, cbETH
- LTV: 90% (vs 普通模式 ~80%)
- 清算阈值: 93%
- 清算bonus: 1% (vs 普通模式 5%)

对清算bot的影响：
1. E-Mode仓位的清算bonus很低（1%）
   → 单笔利润很小
   → 需要更大金额才能覆盖gas
2. E-Mode仓位更容易接近清算线
   → 清算机会更频繁
3. 相关资产价格联动
   → 当ETH大跌时，wstETH/ETH的比率变化较小
   → E-Mode仓位实际被清算的概率低于预期
```

### 5.3 Compound V3 (Comet) 参数

```
Compound V3每个市场一种基础资产：

USDC Market (Ethereum):
| 抵押品 | Supply Cap | Borrow CF | Liquidate CF | Liquidation Factor |
|--------|-----------|-----------|-------------|-------------------|
| ETH    | 400K ETH  | 82.5%     | 85%         | 95%               |
| WBTC   | 25K WBTC  | 70%       | 77%         | 93%               |
| wstETH | 200K      | 82%       | 85%         | 95%               |
| COMP   | 500K      | 65%       | 70%         | 93%               |

Compound V3清算特性：
- absorb(): 协议直接接管抵押品（不是清算人购买）
- 清算人获得 absorb 的gas补偿
- 抵押品由协议以折扣价出售
- buyCollateral(): 任何人可以购买协议持有的折扣抵押品
```

---

## 六、级联清算与系统性风险

### 6.1 级联清算机制

```
级联清算发生过程：

1. 市场下跌 → 部分仓位被清算
2. 清算者卖出获得的抵押品 → 进一步压低价格
3. 更多仓位跌破清算线 → 更多清算发生
4. 形成正反馈循环: 清算 → 抛压 → 价格下跌 → 更多清算

触发条件：
- 高杠杆仓位集中在相近的清算价格
- 市场流动性不足以吸收清算抛压
- 短时间内大量仓位同时触发清算

历史案例：
- 2020年3月12日"黑色星期四": ETH从$200跌至$90
  → Maker清算系统因gas爆炸和竞价不足导致$800万坏账
  → 多个协议同时级联清算

- 2022年5月Luna崩盘:
  → stETH脱锚引发级联清算
  → 多个大户在Aave上被清算
  → 清算抛压进一步压低ETH价格
```

### 6.2 级联清算分析

```
识别级联清算风险：

1. 清算密度图
   → 统计不同价格水平上的可清算仓位金额
   → 密集区 = 级联清算高风险区

2. 清算量预估
   清算量 at price P = Σ(仓位债务) where HF(P) < 1

3. 市场冲击预估
   价格影响 = 清算抛压 / 市场深度
   若价格影响 > 下一级清算价格差 → 级联风险高

// 级联清算模拟
function simulateCascade(positions, initialPriceDrop, marketDepth) {
    price = currentPrice * (1 - initialPriceDrop);
    totalLiquidated = 0;
    round = 0;
    
    while (true) {
        liquidatable = positions.filter(p => calculateHF(p, price) < 1);
        if (liquidatable.isEmpty()) break;
        
        round++;
        sellPressure = sum(liquidatable.map(p => p.collateralValue));
        priceImpact = sellPressure / marketDepth;
        price = price * (1 - priceImpact);
        totalLiquidated += sellPressure;
        
        // 移除已清算仓位
        positions = positions.filter(p => !liquidatable.includes(p));
    }
    
    return { totalLiquidated, finalPrice: price, rounds: round };
}
```

### 6.3 对清算bot的机会

```
级联清算是清算bot的最大机会：

1. 量大: 短时间内大量仓位可被清算
2. 利润高: 高波动期清算bonus更有价值
3. gas竞争激烈: 需要最优的Bundle策略

级联清算策略：
- 提前识别级联风险区间
- 预先构造多笔清算交易
- 使用Bundle一次性提交多笔清算
- 优先清算大额仓位（利润最高）
- 抵押品不立即卖出（避免加剧下跌），等待价格稳定后处置
```

---

## 七、竞争格局

### 7.1 清算bot生态

```
清算bot的层次：

Tier 1 (专业团队):
- 全职开发团队
- 自建全节点+低延迟基础设施
- 多协议同时覆盖
- 使用Flashbots Bundle
- 占据主要清算份额的 ~60-70%

Tier 2 (半专业):
- 1-3人团队
- 使用RPC服务（Alchemy/Infura）
- 专注1-2个协议
- 混合使用Bundle和普通交易
- 占据 ~20-30%

Tier 3 (个人/业余):
- 使用开源bot框架
- 主要靠运气和长尾机会
- 占据 ~5-10%

协议自身的清算机制:
- Aave: 无协议级清算bot，完全依赖外部
- Compound V3: absorb()机制允许任何人触发
- MakerDAO: Keeper网络有协议级支持
```

### 7.2 竞争策略

| 策略 | 描述 | 适用场景 |
|------|------|---------|
| **速度优先** | 低延迟节点+快速Bundle提交 | 主流大额清算 |
| **长尾覆盖** | 监控小型/新兴协议 | 竞争较少的协议 |
| **多链部署** | 同时覆盖多条链 | L2上的清算竞争较L1低 |
| **专有数据** | 预测价格更新时机 | 预判Chainlink更新 |
| **Gas优化** | 合约和执行层面优化gas | 降低盈亏平衡点 |
| **风险承担** | 清算后持有抵押品不立即卖出 | 在预期反弹时获取额外利润 |

---

## 八、多协议清算实战

### 8.1 协议发现与参数获取

```javascript
// 多协议清算bot的配置框架 (TypeScript)
interface ProtocolConfig {
    name: string;
    chain: string;
    poolAddress: string;
    oracleAddresses: Map<string, string>;
    liquidationFunction: string;
    healthFactorFunction: string;
    flashLoanAvailable: boolean;
    minProfitUSD: number;
}

const protocols: ProtocolConfig[] = [
    {
        name: "Aave V3 Ethereum",
        chain: "ethereum",
        poolAddress: "0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2",
        liquidationFunction: "liquidationCall",
        healthFactorFunction: "getUserAccountData",
        flashLoanAvailable: true,
        minProfitUSD: 50
    },
    {
        name: "Aave V3 Arbitrum",
        chain: "arbitrum",
        poolAddress: "0x794a61358D6845594F94dc1DB02A252b5b4814aD",
        liquidationFunction: "liquidationCall",
        healthFactorFunction: "getUserAccountData",
        flashLoanAvailable: true,
        minProfitUSD: 20  // L2 gas低，阈值可以更低
    },
    {
        name: "Compound V3 USDC",
        chain: "ethereum",
        poolAddress: "0xc3d688B66703497DAA19211EEdff47f25384cdc3",
        liquidationFunction: "absorb",
        healthFactorFunction: "isLiquidatable",
        flashLoanAvailable: false,  // 需要外部闪电贷
        minProfitUSD: 50
    }
];
```

### 8.2 清算利润计算

```
利润计算公式（通用）：

Gross Profit = Collateral Received * Market Price - Debt Repaid
Net Profit = Gross Profit - Gas Cost - Flash Loan Fee - Swap Slippage

各协议的Gross Profit计算：

Aave V3:
  Collateral Received = debtToCover * debtPrice / collateralPrice * (1 + liquidationBonus)
  Gross Profit = debtToCover * liquidationBonus  (以债务资产计价)
  
  例: 清算 $10,000 USDC债务, liquidationBonus = 5%
  Gross Profit = $10,000 * 5% = $500

Compound V3:
  协议absorb后以折扣价出售抵押品
  Discount = storeFrontPriceFactor (通常 ~92%)
  Gross Profit = Collateral Value * (1 - storeFrontPriceFactor)
  
MakerDAO:
  荷兰拍卖中的折扣随时间增加
  Gross Profit = Collateral Value * (1 - currentAuctionPrice/marketPrice)
  取决于出价时机
```

---

## 九、L2清算机会

### 9.1 L2上的清算特点

| 特性 | Ethereum L1 | Arbitrum | Base/Optimism |
|------|------------|----------|---------------|
| **Gas成本** | $5-$50 | $0.1-$1 | $0.01-$0.5 |
| **区块时间** | 12s | 0.25s | 2s |
| **清算竞争** | 极激烈 | 中等 | 较低 |
| **最小利润阈值** | ~$50 | ~$5 | ~$1 |
| **Bundle支持** | Flashbots | 有限 | 有限 |
| **Aave V3部署** | 是 | 是 | 是 |

### 9.2 L2清算策略差异

```
L2清算的独特考量：

1. Gas成本极低 → 可以清算更小的仓位
   - L1: 只清算 > $10K 的仓位才划算
   - L2: > $100 的仓位就可能有利可图

2. 区块时间更短 → 反应需要更快
   - Arbitrum 0.25s区块 → 毫秒级决策
   - 需要更低延迟的基础设施

3. Bundle支持有限 → 需要不同的提交策略
   - 更依赖普通交易的gas竞价
   - 或直接连接sequencer

4. 竞争较低 → 适合新手入场
   - 头部清算bot主要集中在L1
   - L2上的长尾机会更多
```

---

## 十、工具速查表

| 用途 | 工具 | 说明 |
|------|------|------|
| 仓位数据 | Aave SubGraph / Compound API | 获取所有仓位及HF |
| 价格监控 | Chainlink Data Feeds | 获取预言机价格 |
| 交易模拟 | Tenderly / Alchemy Simulate | 模拟清算交易 |
| 清算利润 | DeFiLlama Liquidations | 清算数据仪表盘 |
| Bot框架 | Aave Liquidation Bot (开源) | 参考实现 |
| Bundle提交 | Flashbots | L1 Bundle提交 |
| 多链RPC | Alchemy / QuickNode | 多链节点服务 |
| 仓位分析 | Dune Analytics | 大额仓位和清算历史 |

---

## 诚实边界

- **清算是高度竞争的市场**：除非有显著的技术优势，否则很难在L1上持续盈利
- **协议参数会变化**：治理投票可以随时修改LTV、清算bonus等参数
- **预言机风险**：预言机延迟或故障可能导致清算时机错误
- **闪电贷并非零风险**：虽然原子性保证不亏本金，但gas费是确定的成本
- **级联清算难以预测**：历史模拟不能准确预测未来的级联情况
- **L2清算是新兴领域**：基础设施和策略还在快速演变

---

## 关联Skills

- [oracle-manipulation-framework](../oracle-manipulation-framework/) — 预言机操纵与防御框架
- [flash-loan-attack-patterns](../flash-loan-attack-patterns/) — 闪电贷攻击模式
- [mempool-monitoring](../mempool-monitoring/) — Mempool监听基础设施

---

## 附录：调研来源

### 核心参考
- Aave V3技术文档: https://docs.aave.com/
- Compound V3文档: https://docs.compound.finance/
- MakerDAO技术文档: https://docs.makerdao.com/
- DeFiLlama Liquidation Data: https://defillama.com/liquidations
- Chainlink Data Feeds: https://data.chain.link/
- "An Empirical Study of DeFi Liquidations" (Qin et al., 2021)
- Aave Governance Forum: 参数讨论

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
