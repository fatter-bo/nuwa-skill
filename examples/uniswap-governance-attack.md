---
name: uniswap-governance-attack
description: |
  DeFi治理攻击研究与防御框架。覆盖Uniswap Governor Bravo治理架构、
  闪电贷投票攻击、贿选/暗池委托、历史攻击案例、防御机制设计、经济分析。
  基于Compound/Uniswap治理合约源码、Beanstalk攻击事件报告、
  OpenZeppelin治理安全研究、Messari治理数据深度调研。
  核心理念：治理是DeFi协议最高权限的入口——控制治理等于控制一切。
  触发词：「治理攻击」「闪电贷投票」「贿选」「governance attack」「提案安全」。
---

# DeFi治理攻击研究与防御框架

> 治理是协议的"核心密钥"。一个成功的治理攻击可以修改任何参数、转移任何资金、升级任何合约。
> 当攻击成本低于协议控制的资产价值时，攻击就是理性的。

## Skill规则

**此Skill激活后，以DeFi治理攻击研究员身份回应。**

- 所有攻击向量给出具体技术实现
- 攻击成本估算基于真实数据
- 区分理论攻击和已发生攻击
- 防御方案按成本/效果排序
- 历史案例给出完整技术拆解

---

## 回答工作流

| 指令 | 行动 |
|------|------|
| 「治理怎么被攻击」 | -> 输出攻击面全景 + 攻击向量分类 |
| 「闪电贷能不能投票」 | -> 输出技术可行性分析 + 防御机制 |
| 「Beanstalk怎么被攻击的」 | -> 输出完整技术拆解 |
| 「攻击Uniswap治理要多少钱」 | -> 输出经济成本估算 |
| 「怎么防御治理攻击」 | -> 输出防御机制优先级列表 |
| 「分析这个治理提案的安全性」 | -> 按安全检查框架评估 |

---

## 一、治理架构深度解析

### 1.1 Uniswap Governor Bravo架构

```
Uniswap治理系统组件：

UNI Token (ERC-20 + Votes)
  │
  ├─ delegate() → 委托投票权
  │
  └─→ GovernorBravoDelegator (Proxy)
        │
        └─→ GovernorBravoDelegateStorageV1
              │
              ├─ propose()      → 创建提案
              ├─ queue()        → 排队进入Timelock
              ├─ execute()      → 执行提案
              ├─ cancel()       → 取消提案
              └─ castVote()     → 投票
                    │
                    └─→ Timelock (2天延迟)
                          │
                          └─→ 目标合约 (任意操作)
```

### 1.2 关键参数

```
Uniswap治理参数（截至2025年）:

提案门槛 (Proposal Threshold):
  - 需要 2,500,000 UNI (0.25% of supply) 的投票权才能提案
  - 注意：是委托的投票权，不一定自己持有

投票期 (Voting Period):
  - 约7天 (40,320 blocks at 15s/block, 实际随区块时间变化)

投票延迟 (Voting Delay):
  - 约2天 (13,140 blocks)
  - 提案创建后到投票开始的间隔

法定人数 (Quorum):
  - 40,000,000 UNI (4% of supply)
  - 赞成票必须达到此数量，提案才能通过

Timelock延迟:
  - 2天 (172,800 seconds)
  - 提案通过后到执行之间的等待期

总计从提案到执行: 约11天 (2天延迟 + 7天投票 + 2天timelock)
```

### 1.3 提案生命周期

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Pending  │───→│  Active  │───→│ Succeeded│───→│  Queued  │───→│ Executed │
│          │    │ (投票中)  │    │ (通过)   │    │ (排队中)  │    │ (已执行)  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
  2天延迟          7天投票         │                2天timelock
                                  │
                    ┌──────────┐   │  ┌──────────┐
                    │ Defeated │←──┘  │ Expired  │
                    │ (否决)   │      │ (过期)    │
                    └──────────┘      └──────────┘

投票快照: 在propose()调用时的区块号记录快照
投票权: 使用快照区块的UNI余额/委托量
```

### 1.4 投票权机制

```solidity
// UNI Token的投票权机制
contract Uni is ERC20, ERC20Votes {
    // 每个地址可以delegate给任意地址（包括自己）
    mapping(address => address) public delegates;
    
    // 投票权使用checkpoint机制（按区块号记录历史）
    // 这意味着：
    // 1. 投票权基于快照，不能用闪电贷临时获得
    // 2. 但可以在快照前积累投票权
    
    function delegate(address delegatee) external {
        _delegate(msg.sender, delegatee);
    }
    
    // getPriorVotes: 获取某地址在某个区块号的投票权
    // 这是防闪电贷的关键函数
    function getPriorVotes(address account, uint blockNumber) 
        external view returns (uint96) 
    {
        require(blockNumber < block.number, "Not yet determined");
        // 使用checkpoint二分查找
        return _getVotesAtBlock(account, blockNumber);
    }
}
```

---

## 二、攻击向量详解

### 2.1 攻击向量1：闪电贷投票攻击

**理论可行性分析：**

```
Uniswap治理的闪电贷防御:
  ✅ 使用getPriorVotes()基于历史区块快照
  ✅ Voting Delay确保快照在提案前
  → 直接闪电贷投票对Uniswap不可行

但仍然存在的风险场景:
  1. 某些治理合约没有使用快照机制
  2. 使用block.timestamp而非block.number的快照
  3. 快照时刻可预测，攻击者提前布局

绕过快照的间接策略:
  - 在预期提案创建前购买/借入大量UNI
  - 快照记录后立即卖出
  - "快照攻击"成本 = UNI借入成本 * holding期间
```

**没有快照保护的协议仍然脆弱：**

```solidity
// 脆弱的治理实现（无快照保护）
contract VulnerableGovernor {
    function castVote(uint256 proposalId, bool support) external {
        // 直接使用当前余额 → 闪电贷可攻击！
        uint256 votes = token.balanceOf(msg.sender);
        // ...
    }
}

// 攻击步骤：
// 1. 闪电贷借入大量治理token
// 2. 调用castVote()
// 3. 归还闪电贷
// 全程在一个交易中完成，成本仅为闪电贷手续费
```

### 2.2 攻击向量2：贿选（Vote Buying / Bribery）

**链上贿选机制：**

```solidity
// 链上贿选合约示例
contract VoteBribery {
    IERC20Votes public governanceToken;
    uint256 public targetProposalId;
    bool public targetSupport; // true = 投赞成, false = 投反对
    uint256 public rewardPerVote; // 每个投票权的奖励
    
    mapping(address => bool) public hasClaimed;
    
    // 参与者: 先delegate给本合约，然后合约代为投票
    function depositAndDelegate(uint256 amount) external {
        governanceToken.transferFrom(msg.sender, address(this), amount);
        // 记录参与者的投票权
    }
    
    // 合约集中投票
    function executeVote() external {
        IGovernor(governor).castVote(targetProposalId, targetSupport);
    }
    
    // 投票后领取奖励
    function claimReward() external {
        require(!hasClaimed[msg.sender], "Already claimed");
        hasClaimed[msg.sender] = true;
        uint256 reward = userVotingPower[msg.sender] * rewardPerVote;
        payable(msg.sender).transfer(reward);
    }
}
```

**链下贿选渠道：**
```
1. Dark Pool Delegation:
   - 大户私下将投票权委托给攻击者
   - 通过OTC交易或私下协议
   - 不在链上留下明显痕迹

2. 暗标投票市场:
   - 类似Vocdoni/Snapshot式的链下投票市场
   - 匿名出售投票权
   - 使用零知识证明验证投票但隐藏身份

3. 协议间贿选:
   - Protocol A贿赂Protocol B的治理参与者
   - 例：Curve Wars中的CVX/vlCVX生态
   - Votium等平台使贿选标准化

4. 交易所shadow voting:
   - CEX持有大量用户存入的治理token
   - 理论上可以用用户的token投票
   - 已有先例（Steemit/Tron事件，2020）
```

### 2.3 攻击向量3：Timelock绕过

**理论上的Timelock弱点：**

```
1. Timelock本身的admin权限:
   - 如果治理可以修改Timelock的delay
   - 先提案将delay改为0
   - 然后恶意提案可以立即执行
   → 防御: Timelock delay修改本身需要经过Timelock

2. 多步攻击:
   - 第一个提案: 看似无害的参数修改
   - 创造条件让第二个恶意提案可以通过
   - 社区可能不会注意到系统性风险
   
3. 紧急函数:
   - 某些协议有emergency functions绕过Timelock
   - Guardian/Emergency Admin可以直接执行
   → 风险: Guardian被攻破 = Timelock无效
```

### 2.4 攻击向量4：提案内容混淆

```
攻击方式:
  1. 提案描述文字与实际执行的calldata不一致
  2. 看似简单的函数调用，实际参数恶意
  3. 利用代理合约的delegatecall执行任意代码
  
示例:
  提案描述: "将ETH/USDC池费率从0.3%改为0.25%"
  实际calldata: 调用treasury.transfer()转走所有资金

防御:
  - 所有提案必须有审计过的calldata解码
  - 前端展示必须解析和验证calldata
  - 社区工具（Tally, Boardroom）自动标记异常
```

---

## 三、历史攻击案例

### 3.1 Beanstalk闪电贷治理攻击（2022年4月）

**攻击概述：**
```
协议: Beanstalk (算法稳定币)
日期: 2022年4月17日
损失: ~$182M
攻击者获利: ~$80M
方法: 闪电贷 + 治理投票
```

**完整技术拆解：**

```
Beanstalk治理的致命缺陷:
  1. 使用emergencyCommit()函数
  2. 如果提案超过67% Stalk(投票权)支持，可以立即执行
  3. 没有Timelock延迟
  4. 投票权基于当前存款，不是历史快照

攻击步骤（单个交易中完成）:

Step 1: 准备闪电贷
  - 从Aave借入 $350M DAI
  - 从Aave借入 $500M USDC  
  - 从Aave借入 $150M USDT
  - 从Uniswap借入 $32M BEAN
  - 总计约$10亿

Step 2: 获取投票权
  - 将借来的资金存入Beanstalk的Silo获取Stalk
  - 由于Beanstalk使用当前余额计算投票权
  - 攻击者立即获得大量Stalk

Step 3: 投票
  - 对预先创建的BIP-18（恶意提案）投赞成票
  - 达到67%超级多数
  - 调用emergencyCommit()立即执行

Step 4: 恶意提案执行
  - BIP-18将Beanstalk的所有资产转移到攻击者地址
  - 包括: BEAN, Curve LP tokens, 各种稳定币

Step 5: 归还闪电贷
  - 将足够的资金归还给Aave和Uniswap
  - 利润 = 窃取的资产 - 闪电贷金额
  - 净利润约$80M

Step 6: 洗钱
  - 攻击者将$80M通过Tornado Cash混币
```

**Beanstalk攻击的关键教训：**
```
1. 投票权必须基于历史快照（checkpoints）
2. 紧急执行函数必须有额外保护
3. Timelock是最基本的安全要求
4. 治理参数（quorum/threshold）必须防止单体控制
```

### 3.2 Compound治理攻击（2022-2024多次）

```
事件1: Compound Proposal 117 (2022)
  内容: 向教育基金分配COMP token
  争议: 提案被批评为治理参与者利益输送
  结果: 通过并执行
  教训: 合法提案也可能不符合社区利益

事件2: Compound Proposal 119 Bug (2022)
  内容: 修改COMP分发参数
  Bug: 由于合约bug，额外分发了约$80M COMP
  结果: 社区通过另一提案修复，但耗时多天
  教训: 治理提案的代码需要严格审计

事件3: Humpy/Balancer Compound Attack Attempt (2024)
  内容: 大户试图通过提案控制Compound Treasury
  方法: 通过Compound上的借贷获取大量COMP投票权
  结果: 社区发现并阻止
  教训: 借入治理token获取投票权是真实威胁
```

### 3.3 MakerDAO治理危机

```
2020年 Maker "黑色星期四"及后续治理争议:

背景:
  - 2020年3月12日，ETH暴跌约50%
  - Maker清算系统失灵，产生约$4.5M坏账
  - 社区对如何应对产生分歧

治理层面的风险:
  1. MKR token高度集中: 前20个地址控制>50%投票权
  2. 投票参与率极低: 通常<10%
  3. 多次出现a16z等大户单方面决定投票结果
  
  教训:
  - 投票权集中 = 事实上的寡头治理
  - 低参与率使攻击门槛降低
  - 大机构的利益可能与散户不一致
```

### 3.4 Uniswap Fee Switch争议

```
背景:
  Uniswap协议可以对LP收取的手续费分成（protocol fee）
  目前fee switch处于关闭状态（LP获得100%手续费）
  开启fee switch需要治理投票

争议焦点:
  1. 开启fee switch → UNI持有者获得协议收入
     但LP收入减少 → 可能导致流动性外流
  
  2. 2024年的费率调整提案引发大量讨论
     Uniswap Foundation持有约4.7亿UNI（47%）
     理论上Foundation+a16z可以单方面通过任何提案

  3. 治理权力集中问题:
     - Uniswap Foundation: ~4.7亿 UNI
     - a16z: ~1.5亿 UNI
     - 二者合计 > quorum (4000万)
     - 如果二者一致，任何提案都能通过

安全启示:
  - 协议treasury持有大量治理token = 内部人控制
  - Foundation的受托责任与token持有者利益可能冲突
  - 需要制衡机制防止内部人滥用治理权
```

---

## 四、防御机制

### 4.1 时间加权投票（Time-Weighted Voting）

```
原理: 投票权不仅取决于token数量，还取决于持有时间

实现:
  votingPower = tokenBalance * min(holdingDuration / maxDuration, 1)
  
  例: maxDuration = 180天
  - 刚买入: votingPower = balance * 0
  - 持有90天: votingPower = balance * 0.5
  - 持有180天+: votingPower = balance * 1.0

优点:
  - 短期投机者/攻击者的投票权降低
  - 激励长期持有和参与

缺点:
  - 降低流动性（token锁定才有用）
  - 不利于新参与者
  - 实现复杂度增加
```

### 4.2 投票冷却期（Cooldown）

```solidity
// 投票冷却期实现
contract CooldownGovernor {
    uint256 public constant COOLDOWN_PERIOD = 7 days;
    
    // 记录token最后一次转移时间
    mapping(address => uint256) public lastTransferTime;
    
    function castVote(uint256 proposalId, uint8 support) external {
        // 只有cooldown期之前就持有的token才能投票
        require(
            block.timestamp - lastTransferTime[msg.sender] >= COOLDOWN_PERIOD,
            "Tokens too new to vote"
        );
        // ...
    }
    
    // 在token transfer时更新
    function _afterTokenTransfer(address from, address to, uint256 amount) 
        internal override 
    {
        if (to != address(0)) {
            lastTransferTime[to] = block.timestamp;
        }
    }
}
```

### 4.3 乐观治理（Optimistic Governance）

```
原理: 提案默认通过，只有在反对票超过阈值时才被否决

流程:
  1. 提案创建（需要最低token持有量）
  2. 挑战期（如14天）
  3. 如果反对票 < 否决阈值 → 自动执行
  4. 如果反对票 >= 否决阈值 → 提案被否决

优点:
  - 正常治理不需要高参与率
  - 攻击者需要"沉默通过"，但任何人都可以提出反对
  - 适合常规/低风险提案

缺点:
  - 社区可能不够警觉（注意力衰退）
  - 需要有效的监控和通知系统
  - 对高风险提案需要额外保护

项目: Optimism采用了类似机制
```

### 4.4 Veto机制

```
设计:
  - 设立独立的Security Council（安全委员会）
  - 安全委员会有权否决已通过的提案
  - 否决权有明确的使用条件和限制

实现:
  Governor通过 → Timelock排队 → Security Council审查期 → 执行
  
  如果Security Council在审查期内行使否决:
    → 提案取消
    → 必须公开否决理由
    → 社区可以重新提案（更高quorum通过可以覆盖否决）

示例: Arbitrum DAO的SecurityCouncil
  - 12成员的multisig
  - 可以否决紧急提案
  - 可以执行紧急升级（在极端情况下）
  - 成员由社区选举产生
```

### 4.5 Token锁定要求

```solidity
// ve-model (vote escrow): 锁定token获取投票权
contract VeToken {
    struct Lock {
        uint256 amount;
        uint256 unlockTime;
    }
    
    mapping(address => Lock) public locks;
    uint256 public constant MAX_LOCK = 4 * 365 days;
    
    function createLock(uint256 amount, uint256 duration) external {
        require(duration <= MAX_LOCK);
        
        token.transferFrom(msg.sender, address(this), amount);
        locks[msg.sender] = Lock({
            amount: amount,
            unlockTime: block.timestamp + duration
        });
    }
    
    function getVotingPower(address user) public view returns (uint256) {
        Lock memory lock = locks[user];
        if (block.timestamp >= lock.unlockTime) return 0;
        
        // 线性衰减: 锁定时间越长，投票权越大
        uint256 remainingTime = lock.unlockTime - block.timestamp;
        return lock.amount * remainingTime / MAX_LOCK;
    }
}

// 优点: 攻击者必须锁定资金，增加攻击成本
// 缺点: 降低token流动性，不利于小持有者
```

---

## 五、经济分析

### 5.1 Uniswap治理攻击成本估算

```
方法1: 直接购买UNI达到quorum

所需UNI: 40,000,000 (quorum)
UNI价格: ~$7 (2025年估算)
购买成本: $280,000,000
市场影响: 大量买入会推高价格，实际成本可能2-5x
预估实际成本: $500M - $1.4B

评估: 经济上不可行（除非协议控制资产远超此值）
```

```
方法2: 借入UNI

Aave/Compound上的UNI借入:
  可借量有限（通常<5M UNI）
  借入利率在大量借入时飙升
  且需要超额抵押（~150%）

评估: 不够达到quorum，但可以影响边际投票
```

```
方法3: 贿选

假设需要贿赂40M UNI的投票权:
  如果UNI持有者要求年化10%补偿:
  投票期7天成本 = 40M * $7 * 10% * 7/365 = $536,986
  
  但实际参与率很低，可能只需要购买10-15M UNI的投票权:
  成本降至约 $134,000 - $200,000

评估: 贿选是最经济的攻击方式
  如果协议Treasury > $5M，攻击可能是正期望值的
```

### 5.2 成本-收益分析框架

```python
def governance_attack_feasibility(
    quorum_tokens,
    token_price,
    current_participation_rate,
    protocol_treasury_value,
    bribery_cost_per_token_annual_pct=0.10,
    voting_period_days=7
):
    """
    评估治理攻击的经济可行性
    """
    # 已有的参与投票量
    typical_votes = quorum_tokens * current_participation_rate
    
    # 攻击者需要的额外投票权
    needed_extra = max(0, quorum_tokens - typical_votes)
    # 实际上需要超过对手方，假设需要51%的quorum
    needed_for_majority = quorum_tokens * 0.51
    
    # 方法1: 直接购买
    direct_purchase_cost = needed_for_majority * token_price * 2  # 2x for price impact
    
    # 方法2: 贿选
    bribery_period = voting_period_days / 365
    bribery_cost = needed_for_majority * token_price * bribery_cost_per_token_annual_pct * bribery_period
    
    # 方法3: 借入
    borrow_rate_annual = 0.15  # 假设15%年化借入利率
    borrow_cost = needed_for_majority * token_price * borrow_rate_annual * bribery_period
    collateral_needed = needed_for_majority * token_price * 1.5  # 150%抵押
    
    min_attack_cost = min(direct_purchase_cost, bribery_cost, borrow_cost)
    
    return {
        'needed_votes': needed_for_majority,
        'direct_purchase_cost': direct_purchase_cost,
        'bribery_cost': bribery_cost,
        'borrow_cost': borrow_cost,
        'collateral_for_borrow': collateral_needed,
        'min_attack_cost': min_attack_cost,
        'protocol_treasury': protocol_treasury_value,
        'attack_profitable': min_attack_cost < protocol_treasury_value * 0.5,
        'risk_level': 'CRITICAL' if min_attack_cost < protocol_treasury_value * 0.1 
                      else 'HIGH' if min_attack_cost < protocol_treasury_value * 0.5
                      else 'MEDIUM' if min_attack_cost < protocol_treasury_value
                      else 'LOW'
    }

# Uniswap评估
result = governance_attack_feasibility(
    quorum_tokens=40_000_000,
    token_price=7.0,
    current_participation_rate=0.15,  # 约15%参与率
    protocol_treasury_value=3_000_000_000,  # ~$3B treasury
)
print(f"最低攻击成本: ${result['min_attack_cost']:,.0f}")
print(f"协议资产价值: ${result['protocol_treasury']:,.0f}")
print(f"风险等级: {result['risk_level']}")
```

### 5.3 UNI投票权分布与参与率

```
UNI Token分布（截至2025年估算）:

Uniswap Treasury/Foundation: ~47% (4.7亿 UNI)
  → 理论上可以单方面决定所有投票
  → 但Foundation有受托责任限制

投资者/团队: ~20%
  → a16z, Paradigm, 团队成员等
  → 通常参与率较高

交易所存放: ~15%
  → Binance, Coinbase等
  → 通常不参与投票（但理论上可以）

散户: ~18%
  → 参与率极低（<5%的散户投票）

实际投票参与数据:
  - 典型提案: 5000万-1.5亿 UNI参与
  - 争议性提案: 可达2亿+
  - 日常提案: 有时仅5000万左右
  
  quorum = 4000万，但通常远超quorum通过
  
  真正的风险:
  - 低关注度提案可能仅有5000-8000万UNI投票
  - 此时2000-3000万UNI就可能决定结果
  - 贿赂2000万UNI需要约$26,000-$40,000（按上述模型）
```

---

## 六、治理攻击检测与监控

### 6.1 预警指标

```
实时监控应关注:

1. 异常委托活动:
   - 大量UNI突然委托给未知地址
   - 新地址获得大量委托 → 可能是贿选
   - 监控 DelegateChanged / DelegateVotesChanged 事件

2. 异常token流动:
   - 从CEX大量提取UNI
   - 大额UNI转移到新合约
   - Aave/Compound上UNI借入量突增

3. 异常提案:
   - calldata执行非常规操作
   - 提案描述模糊或误导性
   - 提案者是新地址或低信誉地址

4. 投票模式异常:
   - 投票最后时刻突然涌入大量赞成/反对票
   - 多个独立地址同时投票（协调行为）
   - 投票方向与社区讨论不一致
```

### 6.2 监控工具

```
推荐监控栈:

1. OpenZeppelin Defender:
   - 自动监控治理合约事件
   - 自定义告警规则
   - 支持自动响应

2. Tally:
   - 治理提案前端
   - 投票权分布可视化
   - 委托追踪

3. Boardroom:
   - 跨协议治理聚合
   - 投票历史分析
   - 参与率追踪

4. 自定义监控:
   - 监听 ProposalCreated 事件
   - 解析calldata → 标记高风险操作
   - 追踪delegate变化 → 识别异常集中
```

### 6.3 治理安全评估框架

```
评估维度 | 低风险 | 中风险 | 高风险
─────────┼────────┼────────┼────────
Timelock延迟 | >=7天 | 2-7天 | <2天或无
投票快照 | 基于历史区块 | 基于当前区块 | 无快照
Quorum | >10%供应量 | 4-10% | <4%
投票权集中度 | 前10地址<30% | 30-50% | >50%
参与率 | >20% | 10-20% | <10%
安全委员会 | 有veto权 | 有但权限有限 | 无
紧急函数 | 无或有严格限制 | 有multisig保护 | 有单人控制
代码审计 | 治理代码经审计 | 部分审计 | 未审计
```

---

## 七、决策模板

### 模板A：治理提案安全评估

```
═══════════════════════════════════════════
     Governance Proposal Security Review
═══════════════════════════════════════════
Protocol: [名称]
Proposal: [编号和标题]

─── Calldata Analysis ─────────────────
Target Contract: [地址]
Function: [函数签名]
Parameters: [参数解码]
Risk Level: [Low/Medium/High/Critical]

─── Voting Power Check ────────────────
Proposer Voting Power: [数量] ([百分比])
Top 5 Supporters: [列表]
Suspicious Delegation Activity: [是/否]

─── Economic Impact ───────────────────
Treasury Impact: [金额]
Token Supply Impact: [稀释/燃烧/无变化]
LP Impact: [正面/负面/中性]

─── Recommendation ────────────────────
[通过/反对/需要更多信息]
[理由]
═══════════════════════════════════════════
```

### 模板B：协议治理安全审计

```
审计项目 | 状态 | 风险
─────────┼──────┼─────
Timelock存在 | [✓/✗] | ...
Timelock延迟>=48h | [✓/✗] | ...
投票权使用快照 | [✓/✗] | ...
无紧急执行函数 | [✓/✗] | ...
Quorum合理 | [✓/✗] | ...
投票权分散 | [✓/✗] | ...
有veto机制 | [✓/✗] | ...
calldata审计流程 | [✓/✗] | ...
delegate监控 | [✓/✗] | ...
```

---

## 相关技能

- [flash-loan-attack-patterns](../flash-loan-attack-patterns/) - 闪电贷攻击模式（Beanstalk的技术基础）
- [hayden-adams-perspective](../hayden-adams-perspective/) - Hayden Adams对Uniswap治理的立场
- [defi-audit-methodology](../defi-audit-methodology/) - DeFi审计方法论（治理审计子集）
