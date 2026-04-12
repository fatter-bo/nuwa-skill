# 04 跨游戏积分互通

## 核心命题

同一套"链上欢乐豆"在斗地主、麻将、德州扑克等不同游戏间通用，但各游戏经济体量不同，必须解决"购买力平价"问题——100积分在斗地主能玩10局，在德州扑克也应该能玩等值局数。

---

## 跨游戏积分架构

### 单一代币 vs 代币+汇率

| 方案 | 优势 | 劣势 |
|------|------|------|
| 单一代币全游戏通用（推荐） | 用户理解简单，无汇率摩擦 | 需要每个游戏精心平衡入场费 |
| 各游戏独立代币+中央汇率 | 游戏可独立调控经济 | 用户混淆，管理复杂 |
| Hub-Spoke模型（中心积分+游戏积分） | 跨游戏用中心积分，游戏内用本地积分 | 兑换损耗降低体验 |

**推荐：单一代币 + 游戏侧参数调控**

```
所有游戏共享一个 HappyPoints 合约
↓
每个游戏独立设置：
- 入场费 = baseAmount × gameDifficultyFactor
- 奖励 = baseReward × gameRewardFactor
- 道具价格 = basePrice × gamePriceFactor

链上合约只管 mint/burn/balance
参数调控在链下游戏服务器完成
```

---

## 购买力平价（PPP）设计

### 问题定义

不同游戏的"时间成本"和"娱乐价值"不同：
- 一局斗地主 ≈ 5分钟
- 一局麻将 ≈ 20分钟
- 一局德州 ≈ 30分钟+

如果统一入场费100积分，斗地主玩家消耗速度是德州的6倍。

### PPP平衡公式

```
核心原则：1小时游戏消耗的净积分应相近

净消耗/小时 = (入场费 × 局数/小时) - (平均奖励 × 局数/小时) - 时间奖励/小时

目标：所有游戏的"净消耗/小时"在同一数量级

示例：
斗地主：入场费50 × 12局/h - 均奖40 × 12局/h = 净消耗120/h
麻将：  入场费200 × 3局/h - 均奖160 × 3局/h = 净消耗120/h
德州：  入场费300 × 2局/h - 均奖240 × 2局/h = 净消耗120/h
```

### 动态调节机制

```
监控指标：
- 各游戏DAU占比
- 各游戏净积分流出/流入
- 跨游戏用户的积分消耗分布

调节规则：
- 若某游戏积分净流出过大 → 降低该游戏入场费或提高奖励
- 若某游戏积分净流入过大 → 提高入场费或增加消耗道具
- 目标：各游戏积分池不出现单向大幅流动
```

---

## 技术实现

### 合约架构

```solidity
// 中央积分合约
contract HappyPoints is ERC20, AccessControl {
    bytes32 public constant GAME_ROLE = keccak256("GAME_ROLE");
    
    // 只有授权的游戏服务器可以 mint/burn
    function gameMint(address player, uint256 amount) external onlyRole(GAME_ROLE) {
        _mint(player, amount);
    }
    
    function gameBurn(address player, uint256 amount) external onlyRole(GAME_ROLE) {
        _burn(player, amount);
    }
    
    // 用户间转账被禁止
    function _beforeTokenTransfer(address from, address to, uint256 amount) internal override {
        require(from == address(0) || to == address(0) || hasRole(GAME_ROLE, msg.sender),
                "Transfer blocked");
    }
}

// 每个游戏注册为 GAME_ROLE
// 游戏A: gameMint/gameBurn 通过自己的逻辑调用中央合约
// 游戏B: 同上，但参数（入场费/奖励）由自己的链下服务器决定
```

### 跨链场景（未来扩展）

如果不同游戏部署在不同链上：

| 方案 | 适用场景 | 复杂度 |
|------|---------|--------|
| 单链部署（推荐初期） | 所有游戏在同一L2 | 低 |
| Bridge桥接 | 游戏在不同L2 | 高（安全风险） |
| CCIP (Chainlink) | 企业级跨链 | 中（安全性高） |
| 状态同步 | 链下同步+各链独立合约 | 中 |

**推荐初期策略**：所有游戏部署在同一L2（如Base），避免跨链复杂性。等生态成熟后再考虑CCIP跨链。

---

## 用户体验设计

### 积分展示

```
用户看到的：
┌──────────────────────┐
│ 我的欢乐豆: 12,500   │
│                      │
│ 今日斗地主: +800     │
│ 今日麻将:   -200     │
│ 日常奖励:   +1000    │
└──────────────────────┘

用户不需要知道的：
- 这是一个ERC-20代币
- 斗地主和麻将的积分来自同一个合约
- 数据经过了批量结算上链
```

### 跨游戏流转无感化

- 用户在斗地主赢了500积分 → 打开麻将 → 余额自动包含这500
- 无需"转移"、"兑换"等操作
- 后端：同一合约地址，查询同一balance

---

## 关键调研来源

- Chainlink CCIP: Cross-Chain Interoperability Protocol for Gaming
- Portal Coin: Cross-Game Token Interoperability Model
- Power Protocol: Unified Token Across Gaming Ecosystem
- Gala Games: Multi-Game Single Token Architecture
- QQ游戏：欢乐豆跨棋牌游戏通用机制分析
