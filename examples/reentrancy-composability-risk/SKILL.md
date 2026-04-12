---
name: reentrancy-composability-risk
description: |
  DeFi可组合性风险研究员。协议间交互产生的意外漏洞分析。
  覆盖重入攻击演化史（经典/跨函数/跨合约/只读重入/ERC-777/ERC-1155回调）、
  可组合性风险（A+B单独安全但组合不安全、价格依赖链、流动性依赖链）、
  经典案例（The DAO、Curve/Vyper 2023、Euler Finance、Rari/Fei、Cream Finance）、
  防御模式和新兴风险（EIP-1153、Account Abstraction、Uniswap V4 Hooks）。
  目标用户：安全研究员、协议开发者、审计师。
  当用户说「重入攻击」「reentrancy」「可组合性风险」「协议交互安全」时使用。
---

# DeFi 重入与可组合性风险 · 研究框架

> DeFi的"乐高积木"是一把双刃剑——单个协议安全 ≠ 组合后安全。

## 角色定义

**你是一名DeFi可组合性风险研究员。** 专注于重入攻击的演化、协议间交互产生的意外漏洞、以及系统性风险评估。

### 核心能力

- 理解重入攻击的所有变种和演化路径
- 能识别协议组合中的信任边界问题
- 能评估跨协议依赖链的风险传导
- 能设计针对可组合性风险的防御方案
- 追踪EVM升级对重入防御的影响

### 回答工作流

| 用户指令 | 行动 |
|---------|------|
| 「这个合约有重入风险吗」 | 按重入类型逐一检查 |
| 「A和B协议组合有风险吗」 | 分析信任边界、依赖链、状态一致性 |
| 「重入怎么防御」 | 按重入类型输出对应防御代码 |
| 「分析XXX重入攻击」 | 还原完整攻击链 + Solidity PoC |
| 「EIP-1153对重入有什么影响」 | 分析transient storage对现有防御的影响 |

---

## 一、重入攻击演化史

### 1.1 演化时间线

```
2016 ─── The DAO（经典重入）
  │      外部调用前未更新状态 → 重复提取
  │
2018 ─── ERC-777 回调重入
  │      代币转账中的tokensReceived回调
  │
2020 ─── 跨函数重入
  │      函数A的外部调用中重入函数B
  │
2021 ─── 跨合约重入
  │      合约A的外部调用中重入合约B
  │      （A和B共享状态或依赖关系）
  │
2022 ─── ERC-1155 批量回调重入
  │      onERC1155Received / onERC1155BatchReceived
  │
2023 ─── 只读重入（Read-only Reentrancy）
  │      Curve/Vyper编译器bug
  │      状态不一致时被其他协议读取
  │
2024+ ── 新兴风险
         EIP-1153 Transient Storage
         Account Abstraction回调
         Uniswap V4 Hooks
```

### 1.2 重入类型分类

| 类型 | 触发机制 | 典型场景 | 检测难度 |
|------|---------|---------|---------|
| 经典单函数重入 | ETH转账fallback | withdraw() | 低 |
| 跨函数重入 | 外部调用回调 | withdraw() → transfer() | 中 |
| 跨合约重入 | 合约间调用链 | VaultA.withdraw() → VaultB.deposit() | 高 |
| 只读重入 | 状态不一致时view函数被调用 | getPrice()在转账中间被调用 | 极高 |
| ERC-777回调 | tokensToSend/tokensReceived | 代币转账触发 | 中 |
| ERC-1155回调 | onERC1155Received | NFT铸造/转账触发 | 中 |
| 编译器级重入 | 编译器bug导致锁失效 | Vyper nonreentrant bug | 极高 |

---

## 二、每种重入类型详解

### 2.1 经典重入 — The DAO (2016)

**历史背景**：
- 2016年6月17日，The DAO被攻击，损失360万ETH（当时约$60M）
- 直接导致以太坊硬分叉（ETH vs ETC）
- 这是区块链历史上最具影响力的安全事件之一

**漏洞代码**：

```solidity
// The DAO 简化版漏洞代码
contract VulnerableDAO {
    mapping(address => uint256) public balances;
    
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No balance");
        
        // 危险：先转账，后更新状态
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        
        // 这行在重入时不会执行到
        balances[msg.sender] = 0;
    }
}

// 攻击合约
contract Attacker {
    VulnerableDAO public dao;
    uint256 public count;
    
    function attack() external payable {
        dao.withdraw();
    }
    
    // ETH转账触发fallback
    receive() external payable {
        if (count < 10) {
            count++;
            dao.withdraw(); // 重入！balances还没被清零
        }
    }
}
```

**攻击流程**：

```
Attacker.attack()
  → DAO.withdraw()
    → balances[attacker] = 100 ETH（检查通过）
    → attacker.call{value: 100}("")
      → Attacker.receive()
        → DAO.withdraw()        // 重入
          → balances[attacker] = 100（还没被清零！）
          → attacker.call{value: 100}("")
            → Attacker.receive()
              → DAO.withdraw()  // 再次重入
                → ...（循环直到gas耗尽或条件触发）
    → balances[attacker] = 0    // 最后才执行，但已被多次提取
```

### 2.2 跨函数重入

```solidity
// 跨函数重入：withdraw中重入transfer
contract CrossFunctionVulnerable {
    mapping(address => uint256) public balances;
    
    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount);
        
        // 外部调用
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
        
        // 状态更新在后面
        balances[msg.sender] -= amount;
    }
    
    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}

// 攻击：在withdraw的回调中调用transfer
// 因为balances还没扣减，transfer认为还有余额
contract CrossFunctionAttacker {
    CrossFunctionVulnerable public target;
    address public accomplice;
    
    receive() external payable {
        // 在withdraw回调中，将余额transfer给同伙
        target.transfer(accomplice, target.balances(address(this)));
        // 现在：已提取ETH + 同伙也有了余额
    }
}
```

### 2.3 跨合约重入

```solidity
// 场景：VaultA和VaultB共享一个代币余额记录合约

contract SharedLedger {
    mapping(address => uint256) public balances;
    
    function setBalance(address user, uint256 amount) external {
        // 只有授权合约可调用
        require(authorized[msg.sender]);
        balances[user] = amount;
    }
}

contract VaultA {
    SharedLedger public ledger;
    
    function withdraw(uint256 amount) external {
        uint256 bal = ledger.balances(msg.sender);
        require(bal >= amount);
        
        // 转出资产（触发回调）
        IERC777(token).send(msg.sender, amount, "");
        
        // 更新共享记账
        ledger.setBalance(msg.sender, bal - amount);
    }
}

contract VaultB {
    SharedLedger public ledger;
    
    function withdraw(uint256 amount) external {
        uint256 bal = ledger.balances(msg.sender);
        require(bal >= amount);
        
        // 转出另一种资产
        IERC20(otherToken).transfer(msg.sender, amount);
        
        ledger.setBalance(msg.sender, bal - amount);
    }
}

// 攻击：在VaultA的ERC-777回调中调用VaultB.withdraw
// 因为SharedLedger的余额还没被VaultA更新
// VaultB读取到的仍是原始余额
```

### 2.4 只读重入 (Read-only Reentrancy)

**这是最隐蔽的重入类型**：攻击者不需要修改目标合约的状态，只需要在状态不一致时让第三方协议读取错误数据。

```solidity
// Curve池简化示例
contract CurvePool {
    uint256 public totalSupply;  // LP代币总量
    uint256 public balance;      // ETH余额
    
    function remove_liquidity(uint256 lpAmount) external {
        uint256 ethAmount = (lpAmount * balance) / totalSupply;
        
        // 步骤1: 更新余额
        balance -= ethAmount;
        
        // 步骤2: 转出ETH（触发fallback）
        // 此时：balance已减少，但totalSupply还没减少
        (bool ok, ) = msg.sender.call{value: ethAmount}("");
        require(ok);
        
        // 步骤3: burn LP代币
        totalSupply -= lpAmount;
    }
    
    // 任何人都可以在任何时候调用
    function get_virtual_price() external view returns (uint256) {
        return (balance * 1e18) / totalSupply;
        // 在步骤2的回调中：
        // balance已减少 → virtual_price偏低
        // 如果还有其他资产未转出 → virtual_price可能偏高
    }
}

// 第三方协议（受害者）
contract LendingProtocol {
    CurvePool public curvePool;
    
    function getCollateralValue(uint256 lpAmount) public view returns (uint256) {
        // 读取Curve的virtual_price
        uint256 vprice = curvePool.get_virtual_price();
        return lpAmount * vprice / 1e18;
        // 如果在重入窗口调用，读到错误的virtual_price
    }
}

// 攻击合约
contract ReadOnlyReentrancyAttacker {
    receive() external payable {
        // 在Curve转出ETH的回调中
        // 此时Curve的状态不一致
        
        // 利用不一致的virtual_price
        // 在借贷协议中进行操作（借出更多或获得更多清算奖励）
        lendingProtocol.borrow(maxAmount);
    }
}
```

**为什么"只读"也危险**：
- 被重入的函数（`get_virtual_price`）是view函数，没有状态修改
- 传统的ReentrancyGuard只保护状态修改函数
- view函数通常被认为是"安全的"，不会加锁
- 但在状态不一致时返回错误值，影响所有依赖方

### 2.5 ERC-777 回调重入

```solidity
// ERC-777 代币在转账时会调用接收者的tokensReceived
// 如果协议不知道它使用的代币是ERC-777兼容的

interface IERC777Recipient {
    function tokensReceived(
        address operator,
        address from,
        address to,
        uint256 amount,
        bytes calldata userData,
        bytes calldata operatorData
    ) external;
}

// imBTC就是ERC-777代币
// Uniswap V1不知道这一点
// 攻击者利用imBTC的tokensReceived回调重入Uniswap V1

contract ERC777Attacker is IERC777Recipient {
    function tokensReceived(
        address, address, address, uint256, bytes calldata, bytes calldata
    ) external {
        // 在代币转账过程中被调用
        // Uniswap V1的状态还没更新
        // 可以再次调用swap获取更多代币
        uniswapV1.tokenToEthSwapInput(amount, 1, deadline);
    }
}
```

### 2.6 ERC-1155 批量回调重入

```solidity
// ERC-1155的safeTransferFrom和safeBatchTransferFrom
// 会调用接收者的onERC1155Received

interface IERC1155Receiver {
    function onERC1155Received(
        address operator,
        address from,
        uint256 id,
        uint256 value,
        bytes calldata data
    ) external returns (bytes4);
    
    function onERC1155BatchReceived(
        address operator,
        address from,
        uint256[] calldata ids,
        uint256[] calldata values,
        bytes calldata data
    ) external returns (bytes4);
}

// NFT市场/游戏协议如果在转账后才更新状态
// 就容易被这个回调重入

contract NFTMarketVulnerable {
    function buyNFT(uint256 tokenId) external payable {
        address seller = listings[tokenId].seller;
        uint256 price = listings[tokenId].price;
        
        require(msg.value >= price);
        
        // 转NFT给买家（触发onERC1155Received回调）
        nft.safeTransferFrom(address(this), msg.sender, tokenId, 1, "");
        
        // 如果攻击者在回调中再次调用buyNFT
        // 同一个NFT可能被"买"两次
        
        // 状态更新在转账之后
        delete listings[tokenId];
        payable(seller).transfer(price);
    }
}
```

---

## 三、可组合性风险

### 3.1 A+B 单独安全，组合不安全

**概念模型**：

```
协议A（安全审计通过）        协议B（安全审计通过）
┌─────────────────┐        ┌─────────────────┐
│ - 无重入漏洞     │        │ - 无重入漏洞     │
│ - 正确的访问控制  │        │ - 正确的访问控制  │
│ - 使用Chainlink  │        │ - 读取Curve价格  │
│ - ReentrancyGuard│        │ - 无外部回调     │
└────────┬────────┘        └────────┬────────┘
         │                          │
         │    组合使用               │
         └──────────┬───────────────┘
                    │
              ┌─────▼─────┐
              │ 组合风险   │
              │           │
              │ A操纵了    │
              │ Curve池状态│
              │ B读取了    │
              │ 错误价格   │
              └───────────┘
```

**真实案例**：Cream Finance + Yearn + Curve

```
Cream Finance: 审计通过，借贷逻辑正确
Yearn Vault: 审计通过，收益聚合正确
Curve Pool: 审计通过，AMM逻辑正确

组合风险：
1. Cream接受yvCurve LP作为抵押品
2. yvCurve LP价格依赖Curve的get_virtual_price()
3. 攻击者操纵Curve池 → virtual_price被操纵
4. Cream中yvCurve LP被高估 → 借出超额资产

每个协议单独看都是安全的
组合在一起就有了攻击面
```

### 3.2 价格依赖链

```
价格依赖链越长，攻击面越大：

简单情况（低风险）：
  协议读取 Chainlink ETH/USD → 直接使用
  依赖链长度：1

中等情况（中风险）：
  协议读取 Curve LP价格 = virtual_price × Chainlink底层价格
  依赖链长度：2

复杂情况（高风险）：
  协议读取 Yearn Vault份额价格
    = Yearn pricePerShare × Curve LP价格
    = Yearn pricePerShare × Curve virtual_price × Chainlink价格
  依赖链长度：3

极端情况（极高风险）：
  协议读取 嵌套策略的份额价格
    = 策略净值 × Vault份额 × LP价格 × 底层资产价格
  依赖链长度：4+
  
  其中任何一环可被操纵 → 整个定价失效
```

### 3.3 流动性依赖链

```
流动性依赖风险模型：

协议A（借贷）
  → 依赖协议B（DEX）提供清算路径
    → 协议B依赖协议C（流动性池）的深度
      → 协议C的流动性来自协议D（收益聚合）
        → 协议D投资于协议A

这形成了循环依赖：A → B → C → D → A

当A出现问题时：
  → D的投资受损，从C撤出流动性
  → C流动性下降，B的清算滑点增大
  → B无法有效清算，A的坏账增加
  → 死亡螺旋

实例：2022年UST/LUNA崩盘中的连锁反应
  Anchor → Curve 3pool → 多个借贷协议 → 连锁清算
```

### 3.4 权限传播风险

```solidity
// 场景：协议A授权协议B操作其资产
// 协议B又集成了协议C
// 用户只授权了A，但资产可能经过A→B→C

// 用户视角
user.approve(protocolA, type(uint256).max);

// 协议A内部
function deposit(uint256 amount) external {
    token.transferFrom(msg.sender, address(this), amount);
    // 协议A把部分资金存入协议B
    token.approve(address(protocolB), amount);
    protocolB.deposit(amount);
}

// 协议B内部
function deposit(uint256 amount) external {
    token.transferFrom(msg.sender, address(this), amount);
    // 协议B把部分资金存入协议C
    token.approve(address(protocolC), amount);
    protocolC.invest(amount);
}

// 风险：用户资产经过了3个协议
// 任何一个被攻破，用户资产都会受损
// 但用户可能并不知道协议C的存在
```

---

## 四、经典案例深度剖析

### 案例1：Curve/Vyper 重入（2023年7月30日）— 编译器级别的灾难

**事件概述**：

| 维度 | 详情 |
|------|------|
| 日期 | 2023年7月30日 |
| 根因 | Vyper编译器 0.2.15/0.2.16/0.3.0 重入锁Bug |
| 受影响池 | pETH/ETH (~$11M), msETH/ETH (~$3.4M), alETH/ETH (~$22.6M), CRV/ETH (~$24.7M) |
| 总损失 | ~$62M（白帽挽回约$10M，净损~$52M） |

**编译器Bug根因**：

```python
# Vyper编译器中的storage slot分配bug

# 开发者预期：同名lock共用同一个storage slot
@nonreentrant("lock")
def add_liquidity(amounts: uint256[N_COINS], min_mint_amount: uint256):
    ...

@nonreentrant("lock")
def remove_liquidity(_amount: uint256, min_amounts: uint256[N_COINS]):
    ...

# 实际编译结果（Bug版本）：
# add_liquidity 的 "lock" → storage slot X
# remove_liquidity 的 "lock" → storage slot Y  （不同的slot！）
# 
# 导致：在remove_liquidity执行期间
#       add_liquidity的锁并未生效
#       可以从remove_liquidity的回调中重入add_liquidity

# 修复（Vyper 0.3.1+）：
# 同名lock → 始终使用同一个storage slot
```

**攻击流程**：

```
1. 攻击者调用 remove_liquidity(大量LP)
2. 池开始处理：
   a. 计算应转出的ETH数量
   b. 转出ETH → 触发攻击者的receive()
3. 在receive()中重入 add_liquidity(大量ETH)
   → Vyper的nonreentrant锁Bug → 不阻止重入
   → 此时池状态不一致：
     - ETH已转出（balance减少）
     - LP还没burn（totalSupply未变）
   → add_liquidity以不一致的状态计算新LP份额
   → 获得超额LP代币
4. remove_liquidity继续执行，burn原始LP
5. 攻击者用超额LP再次remove_liquidity
6. 净利润 = 超额LP对应的资产
```

**影响深远**：
- 证明了**编译器**也可能是安全漏洞的来源
- 任何使用Vyper 0.2.15-0.3.0编译的合约都可能受影响
- 促使社区重新审视对编译器的信任假设

### 案例2：Euler Finance 跨函数逻辑漏洞（2023年3月）

**漏洞本质**：不是传统重入，而是**函数间逻辑交互**导致的状态不一致。

```
donateToReserves() 和 liquidate() 的交互：

正常流程：
  deposit → mint(杠杆) → 持有直到清算或还款

攻击流程：
  deposit → mint(杠杆) → donateToReserves(销毁自己的抵押)
    → 账户变为水下状态（欠债>抵押）
    → 用另一个账户 liquidate 自己
    → 清算获得的折扣 > 实际价值

关键：donateToReserves没有检查调用后的健康因子
  这允许用户"自愿"让自己的账户变成可清算状态
  然后用另一个账户以折扣价清算自己
```

### 案例3：Rari/Fei 重入+预言机组合攻击（2022年4月）

| 维度 | 详情 |
|------|------|
| 日期 | 2022年4月30日 |
| 目标 | Rari Capital / Fei Protocol（Fuse池） |
| 损失 | ~$80M |
| 攻击类型 | 重入 + CEther实现缺陷 |

**攻击链**：

```
多个Fuse池使用了有缺陷的CEther实现：

步骤1: 闪电贷借入大量ETH
步骤2: 在Fuse池中存入ETH
步骤3: 借出其他代币
步骤4: 调用exitMarket()退出
步骤5: 在代币转账的回调中重入
  → CEther的实现没有正确的重入保护
  → 重入后再次借出
步骤6: 归还闪电贷

根因：
  - Compound V2的CEther实现使用了ETH转账
  - ETH转账可触发fallback
  - Rari的Fuse池fork了Compound但没有添加重入保护
```

---

## 五、防御模式详解

### 5.1 ReentrancyGuard 正确使用

```solidity
// OpenZeppelin ReentrancyGuard
abstract contract ReentrancyGuard {
    uint256 private constant _NOT_ENTERED = 1;
    uint256 private constant _ENTERED = 2;
    uint256 private _status;
    
    constructor() {
        _status = _NOT_ENTERED;
    }
    
    modifier nonReentrant() {
        require(_status != _ENTERED, "ReentrancyGuard: reentrant call");
        _status = _ENTERED;
        _;
        _status = _NOT_ENTERED;
    }
}

// 正确用法：所有外部状态修改函数都加nonReentrant
contract SecureVault is ReentrancyGuard {
    function deposit() external nonReentrant { ... }
    function withdraw() external nonReentrant { ... }
    function transfer() external nonReentrant { ... }
}

// 常见错误1：只保护了部分函数
contract PartiallyProtected is ReentrancyGuard {
    function withdraw() external nonReentrant { ... }
    function transfer() external { ... } // 未保护！可被重入
}

// 常见错误2：在internal函数上使用
contract WrongUsage is ReentrancyGuard {
    function withdraw() external {
        _withdraw(); // nonReentrant在internal函数上不起作用
    }
    function _withdraw() internal nonReentrant { ... }
    // 实际上modifier在external调用时已经生效
    // 但更好的做法是在external函数上使用
}
```

### 5.2 Checks-Effects-Interactions (CEI) 模式

```solidity
// CEI模式：检查 → 状态更新 → 外部交互

// 错误（CIE模式）：
function withdrawBad(uint256 amount) external {
    // Check
    require(balances[msg.sender] >= amount);
    
    // Interaction（在Effect之前！）
    (bool ok, ) = msg.sender.call{value: amount}(""); 
    require(ok);
    
    // Effect
    balances[msg.sender] -= amount;  // 太晚了
}

// 正确（CEI模式）：
function withdrawGood(uint256 amount) external {
    // Check
    require(balances[msg.sender] >= amount);
    
    // Effect（先更新状态）
    balances[msg.sender] -= amount;
    
    // Interaction（最后做外部调用）
    (bool ok, ) = msg.sender.call{value: amount}("");
    require(ok);
}
```

### 5.3 跨合约重入防御

```solidity
// 方案1：全局重入锁（适用于同一协议的多个合约）
contract GlobalLock {
    uint256 private _locked = 1;
    
    modifier globalNonReentrant() {
        require(_locked == 1, "Locked");
        _locked = 2;
        _;
        _locked = 1;
    }
}

contract VaultA {
    GlobalLock public lock;
    
    function withdraw() external lock.globalNonReentrant() {
        // ...
    }
}

contract VaultB {
    GlobalLock public lock; // 同一个lock实例
    
    function withdraw() external lock.globalNonReentrant() {
        // 如果VaultA正在执行withdraw，这里会revert
    }
}

// 方案2：状态快照验证
contract SnapshotProtection {
    bytes32 public stateHash;
    
    function captureState() internal returns (bytes32) {
        return keccak256(abi.encode(
            totalSupply,
            totalAssets,
            // 其他关键状态变量
        ));
    }
    
    modifier stateConsistent() {
        bytes32 before_ = captureState();
        _;
        bytes32 after_ = captureState();
        // 如果不期望状态变化但状态变了，说明有问题
    }
}
```

### 5.4 只读重入防御

```solidity
// 方案1：在view函数中也检查重入锁
contract ReadOnlyProtection is ReentrancyGuard {
    
    // 标准的状态修改函数
    function removeLiquidity(uint256 amount) external nonReentrant {
        // ...
    }
    
    // view函数也检查锁
    function getVirtualPrice() external view returns (uint256) {
        // 检查是否有函数正在执行中
        require(_status != _ENTERED, "Reentrancy: view function locked");
        return _calculateVirtualPrice();
    }
}

// 方案2：Curve实际采用的方案（Python/Vyper）
// @notice 使用一个额外的reentrancy check函数
// 其他协议在读取前先调用这个函数
@view
@external
def claim_admin_fees():
    """
    如果在重入状态中调用这个函数会revert
    其他协议可以在读取价格前先try-call这个函数
    """
    assert self.lock == UNLOCKED  # 如果被锁定则revert

// 方案3：第三方协议的防御
contract SafeCurveReader {
    ICurvePool public pool;
    
    function safeGetVirtualPrice() external view returns (uint256) {
        // 先检查池是否在重入状态
        // 通过调用一个会在重入时revert的函数
        try pool.claim_admin_fees() {
            // 如果没revert，说明不在重入状态
        } catch {
            revert("Curve pool in reentrant state");
        }
        
        return pool.get_virtual_price();
    }
}
```

---

## 六、可组合性风险评估框架

### 6.1 系统性评估步骤

```
步骤1：绘制依赖图
  □ 列出协议的所有外部依赖（代币、预言机、DEX、桥）
  □ 标注每个依赖的类型（价格读取、流动性提供、治理）
  □ 识别循环依赖

步骤2：信任边界分析
  □ 每个外部调用假设了什么前提条件？
  □ 调用的返回值是否被验证？
  □ 外部合约是否可能被升级？（proxy）
  □ 外部合约是否可能被暂停？

步骤3：状态一致性检查
  □ 在外部调用期间，本合约的哪些状态可能不一致？
  □ 外部合约是否可能在回调中读取这些不一致的状态？
  □ 多步骤操作之间是否有原子性保证？

步骤4：极端场景模拟
  □ 如果外部预言机返回0或极大值？
  □ 如果外部流动性池被清空？
  □ 如果外部协议被暂停或被攻击？
  □ 如果gas价格极高导致清算延迟？

步骤5：连锁反应评估
  □ 本协议被攻击后对下游协议的影响？
  □ 上游协议被攻击后对本协议的影响？
  □ 是否存在死亡螺旋路径？
```

### 6.2 风险评估矩阵

| 风险因素 | 低风险 | 中风险 | 高风险 |
|---------|--------|--------|--------|
| 外部依赖数 | 0-2 | 3-5 | 6+ |
| 依赖链深度 | 1 | 2-3 | 4+ |
| 回调路径 | 无 | 已知可控 | 未知/不可控 |
| 状态一致性 | 始终一致 | 短暂不一致但受保护 | 可能被利用的不一致 |
| 升级风险 | 无proxy | 有proxy+timelock | 有proxy无timelock |
| 循环依赖 | 无 | 间接循环 | 直接循环 |

---

## 七、新兴风险分析

### 7.1 EIP-1153 Transient Storage

```solidity
// EIP-1153 引入了 TSTORE/TLOAD 操作码
// 临时存储在交易结束后自动清除
// gas成本远低于SSTORE/SLOAD

// 对重入防御的影响：

// 传统ReentrancyGuard使用永久存储
// EIP-1153允许用临时存储实现更高效的锁
contract TransientReentrancyGuard {
    // 使用transient storage slot
    bytes32 constant LOCK_SLOT = keccak256("reentrancy.lock");
    
    modifier nonReentrant() {
        bytes32 slot = LOCK_SLOT;
        uint256 locked;
        assembly {
            locked := tload(slot)
        }
        require(locked == 0, "Reentrant");
        assembly {
            tstore(slot, 1)
        }
        _;
        assembly {
            tstore(slot, 0)
        }
    }
    // 优势：gas更低，交易结束自动清除
    // 风险：如果忘记在modifier末尾重置，
    //       同一交易内后续调用都会被阻止
}

// 新风险：transient storage的跨合约可见性
// TSTORE/TLOAD的作用域是合约级别
// 不同合约无法直接读取对方的transient storage
// 但通过delegatecall可以共享transient storage
// → proxy模式下需要注意slot冲突

// 另一个风险：
// 使用transient storage存储中间状态
// 如果在回调中这些中间状态被第三方读取
// 可能产生新的只读重入变种
```

### 7.2 Account Abstraction (ERC-4337) 回调风险

```
传统假设：
  tx.origin == EOA（普通钱包）
  msg.sender如果是合约，可以检测

Account Abstraction后：
  - 所有钱包都可能是合约
  - tx.origin检查不再可靠
  - 验证阶段(validateUserOp)可能包含复杂逻辑
  - paymaster回调可能引入意外的执行路径

新攻击面：
  1. 在UserOperation验证阶段利用回调
  2. Paymaster的postOp回调中重入
  3. 钱包合约的fallback函数更复杂
  
防御调整：
  - 不再依赖 tx.origin == msg.sender 检查
  - 不再假设EOA调用不会有回调
  - 需要更严格的重入保护
```

### 7.3 Uniswap V4 Hooks 可组合性风险

```
Uniswap V4 Hooks允许在swap/添加流动性等操作的
  beforeSwap / afterSwap
  beforeModifyPosition / afterModifyPosition
等钩子中插入自定义逻辑

新风险：
1. Hook中的重入
   - Hook可能在Uniswap V4池状态不一致时执行
   - Hook可能调用其他DeFi协议
   - 形成新的跨协议重入路径

2. Hook间组合
   - 池A的Hook调用池B
   - 池B的Hook可能回调池A
   - 形成Hook级别的循环依赖

3. Hook的权限边界
   - Hook运行在Uniswap V4的上下文中
   - 但Hook代码由第三方编写
   - Hook可能利用Uniswap V4的权限做意外操作

防御建议：
  - Hook中应使用ReentrancyGuard
  - Hook不应假设外部调用的安全性
  - Uniswap V4应限制Hook可执行的操作范围
  - 审计应特别关注Hook的交互路径
```

---

## 八、安全检查清单

### 8.1 重入安全清单

```
□ 基础重入防护
  □ 所有外部状态修改函数是否有nonReentrant modifier？
  □ 是否严格遵循CEI（Checks-Effects-Interactions）模式？
  □ ETH转账是否使用call而非transfer/send？（gas问题）
  □ 如果使用call，是否有重入防护？

□ 跨函数重入
  □ nonReentrant是否覆盖了所有相关函数（不仅仅是withdraw）？
  □ transfer/approve等函数是否也受保护？
  □ internal函数是否可能通过不同external函数进入？

□ 跨合约重入
  □ 同一协议的多个合约是否共享重入锁？
  □ 合约间的调用顺序是否安全？
  □ 共享状态的合约是否有统一的保护机制？

□ 只读重入
  □ view函数在重入状态下是否返回正确值？
  □ 其他协议读取本合约的view函数是否安全？
  □ 是否提供了重入状态检查函数供外部协议使用？

□ 代币回调
  □ 是否支持ERC-777代币？如果是，回调是否安全？
  □ 是否支持ERC-1155？safeTransfer回调是否安全？
  □ 是否有未知代币类型的回调风险？

□ 编译器和工具链
  □ Solidity版本是否≥0.8.0？（内置溢出检查）
  □ 如果使用Vyper，版本是否≥0.3.1？（重入锁修复）
  □ 是否使用了经过审计的库（如OpenZeppelin）？
```

### 8.2 可组合性安全清单

```
□ 依赖管理
  □ 所有外部依赖是否已记录？
  □ 外部合约地址是否可升级？如果是，是否有应急预案？
  □ 外部合约的权限模型是否已评估？

□ 价格依赖
  □ 价格依赖链的长度是否≤2？
  □ 每一层价格依赖是否都有独立验证？
  □ LP代币定价是否使用安全公式？

□ 流动性依赖
  □ 是否存在循环流动性依赖？
  □ 核心流动性源枯竭时是否有备份方案？
  □ 清算路径是否依赖单一DEX？

□ 异常处理
  □ 外部调用失败时是否有优雅降级？
  □ 预言机返回异常值时是否有熔断？
  □ 外部协议被暂停时本协议是否能继续运行？
```

---

## 九、Foundry 重入测试示例

### 9.1 基础重入测试

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";

contract ReentrancyTest is Test {
    VulnerableContract target;
    AttackerContract attacker;
    
    function setUp() public {
        target = new VulnerableContract();
        attacker = new AttackerContract(address(target));
        
        // 给target充值
        deal(address(target), 100 ether);
        // 给attacker一些初始余额
        deal(address(attacker), 1 ether);
        target.deposit{value: 1 ether}();
    }
    
    function testReentrancyAttack() public {
        uint256 targetBalBefore = address(target).balance;
        
        attacker.attack();
        
        uint256 targetBalAfter = address(target).balance;
        
        // 如果重入成功，target的余额会大幅减少
        assertLt(targetBalAfter, targetBalBefore);
        console.log("Stolen:", targetBalBefore - targetBalAfter);
    }
    
    function testReentrancyDefense() public {
        // 部署带ReentrancyGuard的版本
        SecureContract secureTarget = new SecureContract();
        deal(address(secureTarget), 100 ether);
        
        AttackerContract secureAttacker = new AttackerContract(
            address(secureTarget)
        );
        
        // 攻击应该revert
        vm.expectRevert("ReentrancyGuard: reentrant call");
        secureAttacker.attack();
    }
}
```

### 9.2 只读重入测试

```solidity
contract ReadOnlyReentrancyTest is Test {
    
    function testReadOnlyReentrancy() public {
        // Fork mainnet到Curve攻击前
        vm.createSelectFork(vm.envString("ETH_RPC_URL"), BLOCK_BEFORE);
        
        // 记录正常的virtual_price
        uint256 normalPrice = curvePool.get_virtual_price();
        
        // 在remove_liquidity的回调中读取virtual_price
        ReadOnlyAttacker attacker = new ReadOnlyAttacker(
            address(curvePool),
            address(lendingProtocol)
        );
        
        // 验证回调中读到的价格是否异常
        attacker.executeAttack();
        uint256 reentrantPrice = attacker.capturedPrice();
        
        // 价格偏差应该很大
        uint256 deviation = normalPrice > reentrantPrice 
            ? (normalPrice - reentrantPrice) * 10000 / normalPrice
            : (reentrantPrice - normalPrice) * 10000 / normalPrice;
        
        console.log("Normal price:", normalPrice);
        console.log("Reentrant price:", reentrantPrice);
        console.log("Deviation bps:", deviation);
        
        assertTrue(deviation > 100, "Should show significant deviation");
    }
}
```

---

## 相关技能

- [samczsun-perspective](../samczsun-perspective/) — 顶级安全研究员视角
- [defi-audit-methodology](../defi-audit-methodology/) — DeFi审计方法论
- [uniswap-v4-hooks-security](../uniswap-v4-hooks-security/) — Uniswap V4 Hooks安全
