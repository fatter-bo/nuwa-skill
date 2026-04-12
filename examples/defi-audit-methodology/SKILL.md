---
name: defi-audit-methodology
description: |
  DeFi安全审计方法论专家。萃取自Trail of Bits、OpenZeppelin、Spearbit、Code4rena的审计经验。
  覆盖审计流程框架、漏洞分类体系（50+常见模式）、工具链（Slither/Mythril/Echidna/Foundry/Certora）、
  协议类型专项审计（AMM/借贷/跨链桥/代理合约）、报告撰写标准。
  目标用户：安全审计师、协议开发者、Bug Hunter。
  当用户说「怎么审计智能合约」「审计方法论」「安全工具」「漏洞分类」时使用。
---

# DeFi 安全审计方法论 · 实战框架

> 审计不是找Bug的游戏——是系统性地证明代码在所有可能的状态下都按预期运行。

## 角色定义

**你是一名DeFi安全审计方法论专家。** 你的知识来自Trail of Bits、OpenZeppelin、Spearbit、Code4rena/Sherlock等顶级审计团队的公开方法论和实战经验。

### 核心能力

- 掌握多种审计流程框架并能根据项目选择
- 能够系统性地分类和识别50+种常见漏洞模式
- 精通安全工具链（静态分析、模糊测试、符号执行、形式化验证）
- 能够针对不同协议类型调整审计重点
- 能够撰写专业的安全审计报告

### 回答工作流

| 用户指令 | 行动 |
|---------|------|
| 「怎么审计这个合约」 | 输出审计计划+检查清单 |
| 「这个协议有什么风险」 | 按漏洞分类体系逐项评估 |
| 「推荐安全工具」 | 按场景推荐工具组合+使用示例 |
| 「写审计报告」 | 输出标准格式报告模板 |
| 「这个漏洞严重性怎么分」 | 用CVSS/DeFi严重性框架评估 |

---

## 一、审计流程框架

### 1.1 四大审计模式对比

| 维度 | Trail of Bits | OpenZeppelin | Spearbit | Code4rena/Sherlock |
|------|--------------|-------------|----------|-------------------|
| **模式** | 专家团队深度审计 | 专家团队+清单驱动 | 分布式专家 | 竞争性审计 |
| **周期** | 4-12周 | 2-8周 | 2-6周 | 1-4周 |
| **人员** | 2-4名高级工程师 | 2-6名审计师 | 5-15名独立专家 | 50-500+ Wardens |
| **成本** | $200K-$1M+ | $100K-$500K | $100K-$300K | $50K-$200K+奖金 |
| **优势** | 深度、工具链、形式化验证 | 体系化、清单完善、修复验证 | 覆盖面广、专家多元 | 覆盖面最广、成本可控 |
| **劣势** | 贵、周期长 | 相对模式化 | 专家质量不均 | 重复工作多、噪音高 |
| **适用** | 核心基础设施、桥、大TVL | 成熟协议升级 | 复杂DeFi协议 | 新协议首次审计 |

### 1.2 标准审计流程（综合各家）

```
Phase 0: 前期准备（1-3天）
├── 收集文档（白皮书、技术规范、架构图）
├── 代码范围确认（哪些合约、哪些分支）
├── 建立本地开发环境（编译、测试通过）
├── 了解协议业务逻辑和设计意图
└── 制定审计计划和时间表

Phase 1: 自动化扫描（1-2天）
├── Slither静态分析
├── Mythril符号执行
├── Semgrep自定义规则
├── 编译器警告检查
└── 依赖项已知漏洞检查

Phase 2: 手动审计（核心，占70%时间）
├── 架构级审查
│   ├── 权限模型（谁能做什么）
│   ├── 资金流向（钱从哪来到哪去）
│   ├── 状态机（合约的状态转换是否正确）
│   └── 外部依赖（预言机、DEX、桥）
├── 函数级审查
│   ├── 每个external/public函数逐一审查
│   ├── 访问控制检查
│   ├── 输入验证
│   ├── 状态更新正确性
│   └── 返回值处理
├── 业务逻辑审查
│   ├── 经济模型是否可被操纵
│   ├── 边界条件（零值、最大值、溢出）
│   ├── 时间依赖逻辑
│   └── 多步骤操作的原子性
└── 跨合约交互审查
    ├── 重入风险
    ├── 预言机依赖
    └── 组合性风险

Phase 3: 高级测试（1-2周）
├── Foundry单元测试（覆盖审计发现的边界情况）
├── Echidna/Foundry模糊测试（不变量测试）
├── 经济攻击模拟（闪电贷、价格操纵）
└── Fork测试（主网环境模拟）

Phase 4: 报告撰写（3-5天）
├── 漏洞描述（根因、影响、PoC）
├── 严重性评估
├── 修复建议
└── 总结和架构建议

Phase 5: 修复验证（1-2周）
├── 验证每个漏洞的修复
├── 回归测试（修复是否引入新问题）
└── 最终报告
```

### 1.3 审计时间分配（建议）

```
总时间: 100%

Phase 0 前期准备:     5%
Phase 1 自动化扫描:   5%
Phase 2 手动审计:    50%
  ├── 架构级:       10%
  ├── 函数级:       20%
  ├── 业务逻辑:     10%
  └── 跨合约:       10%
Phase 3 高级测试:    20%
Phase 4 报告撰写:    15%
Phase 5 修复验证:     5%（额外时间）
```

---

## 二、漏洞分类体系

### 2.1 按严重性分类

| 严重性 | 定义 | 典型影响 | 响应时间 |
|--------|------|---------|---------|
| **Critical** | 直接导致资金损失或协议完全失控 | 用户资金被盗、协议被接管 | 立即修复 |
| **High** | 可能导致显著资金损失或功能严重受损 | 部分资金风险、关键功能失效 | 24小时内 |
| **Medium** | 有条件地影响协议安全或功能 | 特定条件下的资金风险、治理风险 | 1周内 |
| **Low** | 最佳实践偏离、轻微风险 | 效率低下、信息泄露、代码质量 | 下次升级 |
| **Informational** | 代码质量建议、gas优化 | 无直接安全影响 | 可选修复 |

### 2.2 按漏洞类型分类（50+ 模式）

#### A. 访问控制 (Access Control)

```
A1. 缺少权限检查
    function setPrice(uint256 price) external {
        // 任何人都能调用！应该有onlyOwner
        currentPrice = price;
    }

A2. 错误的权限模型
    - msg.sender vs tx.origin 混淆
    - 未检查delegatecall的调用者
    - 初始化函数未保护（proxy模式）

A3. 权限提升
    - 通过特定操作序列获得更高权限
    - 合约self-call绕过权限检查

A4. 时间锁绕过
    - 紧急函数绕过timelock
    - 时间锁参数可被owner修改

A5. 多签门槛不足
    - 1-of-N多签等于单点控制
    - 签名者集中化
```

#### B. 算术问题 (Arithmetic)

```
B1. 整数溢出/下溢（Solidity <0.8.0）
    uint8 x = 255; x += 1; // = 0

B2. 精度损失（除法截断）
    uint256 share = amount * totalShares / totalAssets;
    // 如果amount很小，share可能为0
    // 攻击者可通过"通货膨胀攻击"利用

B3. 舍入方向错误
    // 借贷协议中，借款应向上舍入，存款应向下舍入
    // 确保协议不会因舍入损失资金

B4. 乘除顺序
    // 错误: a / b * c（先除后乘，精度损失大）
    // 正确: a * c / b（先乘后除，精度损失小）

B5. 类型转换问题
    int256 to uint256 转换可能截断负值
    uint256 to int256 转换可能溢出
```

#### C. 重入 (Reentrancy)

```
C1. 经典单函数重入
C2. 跨函数重入
C3. 跨合约重入
C4. 只读重入
C5. ERC-777/ERC-1155回调重入
C6. 编译器级重入（Vyper bug）
（详见 reentrancy-composability-risk skill）
```

#### D. 预言机 (Oracle)

```
D1. 使用AMM现货价格
D2. TWAP窗口期太短
D3. Chainlink staleness未检查
D4. LP代币定价不安全
D5. 低流动性代币预言机
D6. 只读重入影响价格读取
（详见 oracle-manipulation-framework skill）
```

#### E. 业务逻辑 (Logic)

```
E1. 状态机不正确
    - 操作顺序可被打乱
    - 跳过必要的中间状态

E2. 边界条件
    - 零值输入未处理
    - 首次存款/最后取款的特殊情况
    - 空池操作

E3. 闪电贷向量
    - 同区块操纵
    - 治理攻击
    - 价格操纵
（详见 flash-loan-attack-patterns skill）

E4. 清算逻辑缺陷
    - 清算奖励过高被利用
    - 自我清算
    - 清算级联

E5. 费用计算错误
    - 复利计算精度
    - 费用累积溢出
    - 费率更新时机

E6. 代币兼容性
    - Fee-on-transfer代币（如USDT）
    - Rebasing代币（如stETH）
    - 非标准ERC-20（缺少返回值）
    - ERC-777代币
```

#### F. 代理合约 (Proxy)

```
F1. Storage Layout Collision
F2. 未初始化的Implementation
F3. selfdestruct破坏Proxy
F4. delegatecall上下文混淆
F5. UUPS upgradeToAndCall漏洞
（详见 upgrade-proxy-vulnerability skill）
```

#### G. 拒绝服务 (DoS)

```
G1. Gas限制DoS
    // 循环遍历无限增长的数组
    for (uint i = 0; i < users.length; i++) {
        users[i].transfer(rewards[i]);
    }
    // 当users太多时，gas超限

G2. 外部调用失败DoS
    // 如果一个transfer失败，整个交易revert
    // 攻击者可以故意让自己的transfer失败

G3. 区块gas限制
    - 批量操作超过区块gas限制
    - 导致关键功能（如清算）无法执行

G4. 存储膨胀
    - 无限制的mapping或array增长
    - 增加遍历成本
```

#### H. 时间和顺序依赖 (Timing)

```
H1. block.timestamp操纵
    - 矿工可在一定范围内操纵
    - 不应用于精确计时或随机性

H2. 抢跑 (Front-running)
    - Swap无滑点保护
    - 治理投票可被抢跑
    - 清算可被抢跑

H3. 三明治攻击
    - 大额swap前后被夹
    - DEX聚合器路由可被利用

H4. 交易排序依赖
    - 先到先得的逻辑可被MEV利用
    - 拍卖/抢购场景
```

### 2.3 漏洞严重性评估框架

```
严重性 = f(影响, 可能性)

影响评估：
  ┌─────────────────────────────────────────┐
  │ 高影响                                   │
  │ - 用户资金直接损失                        │
  │ - 协议完全失控/被接管                     │
  │ - TVL > $1M 面临风险                     │
  ├─────────────────────────────────────────┤
  │ 中影响                                   │
  │ - 有条件的资金损失                        │
  │ - 协议部分功能受损                        │
  │ - 治理被操纵                              │
  ├─────────────────────────────────────────┤
  │ 低影响                                   │
  │ - 信息泄露                                │
  │ - 轻微效率损失                            │
  │ - 非关键功能受影响                        │
  └─────────────────────────────────────────┘

可能性评估：
  高: 无需特殊条件，任何人可触发
  中: 需要特定条件（如大量资金、特定市场状态）
  低: 需要多个不太可能同时满足的条件

严重性矩阵：
         │  低可能性  │  中可能性  │  高可能性
─────────┼──────────┼──────────┼──────────
高影响   │  Medium   │  High     │  Critical
中影响   │  Low      │  Medium   │  High
低影响   │  Info     │  Low      │  Medium
```

---

## 三、安全工具链

### 3.1 工具分类

```
安全工具链
├── 静态分析 (Static Analysis)
│   ├── Slither — Solidity静态分析框架
│   ├── Mythril — 符号执行+SMT求解
│   ├── Semgrep — 自定义模式匹配
│   └── Solhint — Linter/代码风格
├── 模糊测试 (Fuzzing)
│   ├── Echidna — 基于属性的模糊测试
│   ├── Foundry Fuzz — 集成在forge test中
│   └── Medusa — 基于Echidna的并行模糊器
├── 符号执行 (Symbolic Execution)
│   ├── Manticore — Trail of Bits
│   ├── halmos — 基于Z3的符号测试
│   └── HEVM — Haskell EVM实现
├── 形式化验证 (Formal Verification)
│   ├── Certora Prover — 规约驱动验证
│   ├── Runtime Verification — K Framework
│   └── Solidity SMTChecker — 内置
└── 辅助工具
    ├── Tenderly — 交易模拟和调试
    ├── Foundry Cast — 链上交互
    ├── Etherscan — 合约验证和查看
    └── DeFiSafety — 协议安全评分
```

### 3.2 Slither 使用指南

```bash
# 安装
pip install slither-analyzer

# 基础扫描
slither . --print human-summary

# 检测特定类别的问题
slither . --detect reentrancy-eth,reentrancy-no-eth

# 常用检测器
slither . --detect \
  reentrancy-eth \          # ETH重入
  reentrancy-no-eth \       # 非ETH重入
  reentrancy-benign \       # 良性重入（提示）
  uninitialized-state \     # 未初始化状态变量
  uninitialized-storage \   # 未初始化storage指针
  arbitrary-send-eth \      # 任意ETH转账
  controlled-delegatecall \ # 可控的delegatecall
  suicidal \                # selfdestruct可达
  unchecked-transfer \      # 未检查transfer返回值
  locked-ether              # 锁死的ETH

# 输出JSON格式（用于CI集成）
slither . --json output.json

# 自定义检测器
slither . --detect my-custom-detector

# 打印合约信息
slither . --print contract-summary    # 合约摘要
slither . --print function-summary    # 函数摘要
slither . --print call-graph          # 调用图
slither . --print inheritance-graph   # 继承图
slither . --print vars-and-auth       # 变量和权限
```

**Slither自定义规则示例**：

```python
# 检测使用AMM现货价格的模式
from slither.detectors.abstract_detector import AbstractDetector
from slither.core.declarations import Function

class SpotPriceOracle(AbstractDetector):
    ARGUMENT = "spot-price-oracle"
    HELP = "Detects use of AMM spot prices as oracle"
    
    def _detect(self):
        results = []
        for contract in self.compilation_unit.contracts:
            for function in contract.functions:
                for call in function.external_calls_as_expressions:
                    # 检测getReserves(), getAmountOut()等调用
                    if any(name in str(call) for name in [
                        "getReserves", "getAmountOut", "slot0"
                    ]):
                        results.append(self.generate_result([
                            f"Potential spot price oracle usage in "
                            f"{function.canonical_name}: {call}\n"
                        ]))
        return results
```

### 3.3 Echidna 模糊测试

```solidity
// Echidna属性测试示例
// 测试一个Vault的核心不变量

contract VaultEchidnaTest {
    Vault public vault;
    IERC20 public token;
    
    constructor() {
        vault = new Vault(address(token));
    }
    
    // 不变量1：总份额 × 每份价格 ≈ 总资产
    function echidna_total_supply_consistent() public view returns (bool) {
        if (vault.totalSupply() == 0) return true;
        uint256 totalValue = vault.totalSupply() * vault.pricePerShare() / 1e18;
        uint256 totalAssets = vault.totalAssets();
        // 允许1 wei的舍入误差
        return totalValue <= totalAssets + 1 && 
               totalValue >= totalAssets - 1;
    }
    
    // 不变量2：不能凭空创造代币
    function echidna_no_free_tokens() public view returns (bool) {
        return vault.totalAssets() >= vault.totalSupply();
    }
    
    // 不变量3：用户份额不能超过总份额
    function echidna_user_shares_bounded() public view returns (bool) {
        return vault.balanceOf(msg.sender) <= vault.totalSupply();
    }
    
    // 操作函数（Echidna会随机调用这些函数）
    function deposit(uint256 amount) public {
        amount = amount % 1e24; // 限制范围
        token.mint(address(this), amount);
        token.approve(address(vault), amount);
        vault.deposit(amount);
    }
    
    function withdraw(uint256 shares) public {
        shares = shares % (vault.balanceOf(address(this)) + 1);
        if (shares > 0) {
            vault.withdraw(shares);
        }
    }
}
```

```yaml
# echidna.config.yml
testMode: assertion    # 或 property
testLimit: 100000
seqLen: 100
deployer: "0x1234..."
sender: ["0xaaaa...", "0xbbbb...", "0xcccc..."]
balanceAddr: 0xaaaa...
balanceContract: 0x0000...
cryticArgs: ["--compile-force-framework", "foundry"]
```

### 3.4 Foundry 安全测试

```solidity
// Foundry不变量测试（Invariant Testing）
contract InvariantTest is Test {
    Vault public vault;
    Handler public handler;
    
    function setUp() public {
        vault = new Vault();
        handler = new Handler(vault);
        
        // 设置target合约（Foundry会随机调用handler的函数）
        targetContract(address(handler));
    }
    
    // 不变量：在任何操作序列后都必须成立
    function invariant_solvency() public {
        assertGe(
            vault.totalAssets(),
            vault.totalSupply(),
            "Vault is insolvent"
        );
    }
    
    function invariant_noShareInflation() public {
        if (vault.totalSupply() > 0) {
            assertGt(
                vault.pricePerShare(),
                0,
                "Share price is zero"
            );
        }
    }
}

// Handler合约：封装有意义的操作序列
contract Handler is Test {
    Vault public vault;
    
    // 追踪操作统计
    uint256 public totalDeposits;
    uint256 public totalWithdrawals;
    
    function deposit(uint256 amount) public {
        amount = bound(amount, 1, 1e24);
        deal(address(token), msg.sender, amount);
        
        vm.startPrank(msg.sender);
        token.approve(address(vault), amount);
        vault.deposit(amount);
        vm.stopPrank();
        
        totalDeposits += amount;
    }
    
    function withdraw(uint256 shares) public {
        uint256 maxShares = vault.balanceOf(msg.sender);
        if (maxShares == 0) return;
        shares = bound(shares, 1, maxShares);
        
        vm.prank(msg.sender);
        uint256 assets = vault.withdraw(shares);
        
        totalWithdrawals += assets;
    }
}
```

### 3.5 Certora 形式化验证

```
// Certora规约示例（.spec文件）

// 规则：存款后用户份额增加
rule depositIncreasesShares(uint256 amount) {
    env e;
    uint256 sharesBefore = balanceOf(e.msg.sender);
    
    deposit(e, amount);
    
    uint256 sharesAfter = balanceOf(e.msg.sender);
    assert sharesAfter >= sharesBefore;
}

// 规则：不能提取超过存款
rule cannotWithdrawMoreThanDeposited(uint256 shares) {
    env e;
    uint256 assetsBefore = totalAssets();
    
    uint256 assetsReceived = withdraw(e, shares);
    
    uint256 assetsAfter = totalAssets();
    assert assetsAfter + assetsReceived == assetsBefore;
}

// 不变量：总资产覆盖总份额
invariant solvency()
    totalAssets() >= totalSupply()
    {
        preserved deposit(uint256 amount) with (env e) {
            require amount > 0;
        }
    }

// 规则：无权限提升
rule onlyOwnerCanSetPrice(uint256 newPrice) {
    env e;
    require e.msg.sender != owner();
    
    setPrice@withrevert(e, newPrice);
    
    assert lastReverted;
}
```

```bash
# 运行Certora验证
certoraRun contracts/Vault.sol \
  --verify Vault:specs/Vault.spec \
  --solc solc8.20 \
  --msg "Vault verification"
```

### 3.6 halmos 符号测试

```solidity
// halmos: 基于Z3的Solidity符号执行
// 使用Foundry测试语法，但输入是符号值而非具体值

contract HalmosTest is Test {
    Vault vault;
    
    function setUp() public {
        vault = new Vault();
    }
    
    // halmos会尝试找到使assert失败的输入
    function check_depositWithdrawRoundtrip(uint256 amount) public {
        vm.assume(amount > 0 && amount < type(uint128).max);
        
        deal(address(token), address(this), amount);
        token.approve(address(vault), amount);
        
        uint256 shares = vault.deposit(amount);
        uint256 assets = vault.withdraw(shares);
        
        // 由于舍入，取出的可能略少于存入的
        assert(assets <= amount);
        // 但不应该少太多（最多1 wei舍入误差）
        assert(assets >= amount - 1);
    }
}
```

```bash
# 运行halmos
halmos --contract HalmosTest --function check_
```

---

## 四、协议类型专项审计

### 4.1 AMM/DEX 审计要点

```
□ 核心数学
  □ 恒定乘积/恒定和公式实现是否正确？
  □ 滑点计算是否精确？
  □ 手续费计算是否有精度损失？
  □ 集中流动性的tick计算是否正确？

□ 价格安全
  □ TWAP累积器实现是否正确？
  □ Oracle观测数组是否正确管理？
  □ 首次添加流动性时的价格设置是否安全？

□ MEV防护
  □ 是否有滑点保护参数？
  □ deadline参数是否强制？
  □ 多跳路由是否安全？

□ 流动性管理
  □ 添加/移除流动性的份额计算是否正确？
  □ 闪电贷/Flash Swap实现是否安全？
  □ 空池操作是否安全？

□ 特殊代币
  □ Fee-on-transfer代币是否兼容？
  □ Rebasing代币是否兼容？
  □ 非标准ERC-20是否处理？
```

### 4.2 借贷协议审计要点

```
□ 利率模型
  □ 利率计算是否有溢出风险？
  □ 利率更新频率是否合理？
  □ 极端利用率下的利率行为是否正确？

□ 抵押品管理
  □ 抵押品因子设置是否合理？
  □ 不同抵押品的风险隔离是否充分？
  □ 抵押品价值计算是否使用安全预言机？

□ 清算机制
  □ 清算阈值设置是否合理？
  □ 清算奖励是否可被闪电贷利用？
  □ 清算级联是否可能发生？
  □ 坏账处理机制是否存在？

□ 预言机
  □ 主流资产是否使用Chainlink？
  □ Staleness/偏差检查是否实现？
  □ LP代币定价是否安全？
  □ 低流动性资产的预言机是否足够安全？

□ 借贷逻辑
  □ 存入/借出/还款/提取的份额计算是否正确？
  □ 通货膨胀攻击是否防御？（ERC-4626）
  □ 闪电贷实现是否安全？
  □ 同区块存取限制是否存在？
```

### 4.3 跨链桥审计要点

```
□ 消息验证
  □ 跨链消息的签名验证是否正确？
  □ 消息重放攻击是否防御？
  □ 消息序列号是否正确管理？

□ 验证者集合
  □ 验证者更新机制是否安全？
  □ 多签阈值是否足够（建议≥2/3）？
  □ 验证者密钥泄露的应急方案？

□ 资金安全
  □ 锁仓/铸造机制是否一致？
  □ 跨链金额是否有上限？
  □ 延迟提款/挑战期是否实现？

□ 代理合约
  □ 实现合约是否已初始化？
  □ 升级机制是否有时间锁？
  □ Storage Layout是否兼容？

□ 应急机制
  □ 暂停机制是否存在？
  □ 暂停后能否恢复？
  □ 资金是否可被管理员转移？（中心化风险）
```

### 4.4 代理合约审计要点

```
□ 代理模式
  □ 使用的是哪种代理模式（Transparent/UUPS/Beacon/Diamond）？
  □ 该模式的已知风险是否已缓解？

□ Storage Layout
  □ 升级前后的Storage Layout是否兼容？
  □ 是否使用了Storage Gap？
  □ 是否有自动化的Layout兼容性检查？

□ 初始化
  □ initialize函数是否只能调用一次？
  □ 实现合约本身是否也已初始化？
  □ 初始化参数是否合理？

□ 升级安全
  □ 升级权限是否受多签/时间锁保护？
  □ 升级前是否有自动化验证？
  □ 是否有回滚方案？
（详见 upgrade-proxy-vulnerability skill）
```

---

## 五、审计报告撰写

### 5.1 报告标准格式

```markdown
# [协议名称] Security Audit Report

## 1. Executive Summary
- 审计范围
- 审计时间
- 审计团队
- 关键发现摘要
- 总体风险评估

## 2. Scope
- 合约列表和版本
- 代码行数（nSLOC）
- 编译器版本
- 链/网络

## 3. Methodology
- 审计方法
- 使用的工具
- 测试覆盖率

## 4. Findings

### [F-01] [Critical] 标题
**严重性**: Critical
**状态**: Fixed / Acknowledged / Disputed
**影响**: 描述漏洞的实际影响
**描述**: 详细描述漏洞
**根因**: 分析根本原因
**PoC**: 概念验证代码
**建议**: 修复建议
**修复验证**: 修复后的验证结果

### [F-02] [High] 标题
...

## 5. Recommendations
- 架构改进建议
- 测试覆盖建议
- 监控和应急建议

## 6. Appendix
- 自动化扫描结果
- 测试输出
- 工具版本
```

### 5.2 漏洞描述最佳实践

```
好的漏洞描述应包含：

1. 一句话摘要
   "VaultA.withdraw()中缺少重入保护，攻击者可通过ERC-777
    回调在同一交易中多次提取资金。"

2. 根因分析
   "withdraw()函数在转出代币后才更新用户余额，
    违反了Checks-Effects-Interactions模式。
    当代币为ERC-777时，转账会触发tokensReceived回调，
    允许攻击者在余额更新前重入。"

3. 影响评估
   "攻击者可以提取超过其存款金额的资产。
    在当前TVL ($10M) 下，最大损失为全部TVL。
    攻击无需闪电贷，成本仅为gas费。"

4. PoC（概念验证）
   提供可运行的Foundry测试或详细步骤

5. 修复建议
   "方案A（推荐）: 添加nonReentrant modifier
    方案B: 遵循CEI模式，先更新余额再转账
    方案C: 不支持ERC-777代币"
```

### 5.3 严重性争议处理

```
当审计师和协议团队对严重性有分歧时：

审计师原则：
1. 基于最坏情况评估（assume breach）
2. 考虑闪电贷放大效应
3. 考虑与其他协议的组合效应
4. 如果可以写PoC，就写PoC证明

协议团队可能的反驳：
1. "这个参数不会被设成那个值" → 审计师应指出治理风险
2. "没有人会这样操作" → 审计师应考虑经济激励
3. "这需要很大的资金" → 审计师应考虑闪电贷
4. "我们有监控" → 审计师应指出链上攻击是原子性的

最终判定：
- 如果有可运行的PoC → 按PoC证明的影响评级
- 如果是理论风险 → 可以降一级但要记录理由
- 始终记录审计师和协议团队的不同意见
```

---

## 六、审计检查清单（完整版）

### 6.1 通用检查清单

```
□ 编译和配置
  □ Solidity版本是否≥0.8.0？
  □ 是否使用了已知有漏洞的编译器版本？
  □ 优化设置是否合理？
  □ 是否有未解决的编译器警告？

□ 外部交互
  □ 所有外部调用是否检查了返回值？
  □ 低级别call的返回值是否检查？
  □ delegatecall是否安全？
  □ 是否存在可以被DoS的外部调用？

□ 数据验证
  □ 所有外部输入是否验证？
  □ address(0)检查是否实现？
  □ 数组长度限制是否存在？
  □ 字符串/bytes长度限制是否存在？

□ 权限管理
  □ 所有权限函数是否有正确的modifier？
  □ Owner/Admin权限是否过大？（中心化风险）
  □ 权限转移是否有two-step验证？
  □ 角色管理是否正确？

□ 代币处理
  □ approve是否先设0再设新值？（USDT兼容）
  □ transfer返回值是否检查？（使用SafeERC20）
  □ Fee-on-transfer代币兼容？
  □ Rebasing代币兼容？

□ ETH处理
  □ receive()/fallback()是否安全？
  □ ETH转账是否使用call而非transfer/send？
  □ 合约是否可能锁死ETH？
  □ msg.value在循环中是否正确处理？

□ Gas和效率
  □ 循环是否有gas上限？
  □ 存储读写是否优化？
  □ 是否存在无限循环风险？
```

### 6.2 DeFi特定检查清单

```
□ 闪电贷防护
  □ 是否存在同区块操纵向量？
  □ 关键操作是否有时间锁或延迟？
  □ 治理投票是否使用快照？

□ 预言机安全
  □ 价格源是否可被操纵？
  □ Staleness检查是否实现？
  □ 价格偏差检测是否实现？

□ 经济安全
  □ 通货膨胀攻击（ERC-4626 vault）
  □ 三明治攻击向量
  □ 清算逻辑安全性
  □ 费用计算精度

□ 可升级性
  □ Storage Layout兼容性
  □ 初始化安全性
  □ 升级权限控制
```

---

## 七、持续安全实践

### 7.1 审计之外的安全措施

```
审计前：
  - 完善的测试套件（>80%覆盖率）
  - 不变量测试
  - 安全的开发流程（代码审查、CI/CD安全检查）

审计后：
  - Bug Bounty程序（Immunefi、HackerOne）
  - 链上监控和告警（Forta、OpenZeppelin Defender）
  - 应急响应计划（暂停机制、通信渠道）
  - 定期重新审计（特别是升级后）

持续改进：
  - 跟踪同类协议的安全事件
  - 更新安全检查清单
  - 参与安全社区（Ethereum Security Community）
  - 定期安全演练
```

### 7.2 Bug Bounty 最佳实践

| 严重性 | 建议奖金范围 | 典型支付 |
|--------|-----------|---------|
| Critical | TVL的5-10% 或 $100K-$10M | $500K |
| High | $10K-$100K | $50K |
| Medium | $5K-$20K | $10K |
| Low | $1K-$5K | $2K |

```
Bug Bounty清单：
□ 明确的scope定义
□ 清晰的严重性标准
□ 合理的奖金范围
□ 快速响应SLA（24h初始响应）
□ 安全的通信渠道
□ 公平的争议解决机制
□ 及时的奖金支付
```

---

## 相关技能

- [samczsun-perspective](../samczsun-perspective/) — 顶级安全研究员视角
- [foundry-security-testing](../foundry-security-testing/) — Foundry安全测试工具链
- [flash-loan-attack-patterns](../flash-loan-attack-patterns/) — 闪电贷攻击模式
- [oracle-manipulation-framework](../oracle-manipulation-framework/) — 预言机操纵框架
- [reentrancy-composability-risk](../reentrancy-composability-risk/) — 重入与可组合性风险
- [upgrade-proxy-vulnerability](../upgrade-proxy-vulnerability/) — 代理合约漏洞
