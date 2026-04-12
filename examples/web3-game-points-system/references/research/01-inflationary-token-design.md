# 01 通胀型代币设计 — 与DeFi代币的根本区别

## 核心命题

休闲棋牌积分（如欢乐豆）的本质是"无限供给的游戏燃料"，与DeFi代币追求升值的逻辑完全相反。链上积分设计必须从"通胀是feature，不是bug"出发。

---

## 为什么通胀是休闲游戏的正确选择

### DeFi代币 vs 休闲积分的本质差异

| 维度 | DeFi代币 | 休闲游戏积分 |
|------|---------|------------|
| 供给模型 | 有限/通缩 | 无限/通胀 |
| 设计目标 | 升值、投资回报 | 保持游戏流动性 |
| 可交易性 | 自由交易 | 不可交易（Soulbound） |
| 用户心理 | 囤积、投机 | 消耗、娱乐 |
| 归零恐惧 | 极高（影响投资） | 无（免费获取渠道存在） |
| 新玩家门槛 | 需购买代币 | 登录即送 |

### 通胀的三个核心优势

1. **新玩家友好**：通胀确保新玩家永远能免费获得足够积分入场，不存在"买不起入场费"问题
2. **消除投机动机**：当积分持续贬值时，囤积无意义，推动玩家将积分用于游戏本身
3. **运营弹性**：通过活动发放大量积分不会造成经济危机，因为积分本身不与法币挂钩

---

## 通胀模型设计

### Faucet（水龙头）— 积分产出

| 来源 | 频率 | 数量策略 |
|------|------|---------|
| 每日登录工资 | 每日4次 | 余额低于阈值时触发（参考欢乐豆：余额<1000时发1000） |
| 对局奖励 | 每局结束 | 赢家获得固定基础奖励 + 浮动倍数 |
| 活动发放 | 节假日/运营活动 | 大量一次性发放，制造"富足感" |
| 新手礼包 | 注册时一次 | 足够玩30局以上 |
| 任务系统 | 每日/每周 | 引导玩家完成特定行为 |

### Sink（消耗）— 积分去向

| 消耗点 | 机制 | 占比目标 |
|--------|------|---------|
| 对局入场费 | 进入牌桌的必要条件 | 40-50% |
| 道具购买 | 表情包/记牌器/加倍卡 | 20-30% |
| 高级场门槛 | 更高赌注的牌桌 | 15-20% |
| 系统税 | 每局抽水（如5%） | 5-10% |
| 外观装饰 | 牌桌皮肤/头像框 | 5-10% |

### 通胀率控制公式

```
目标：周通胀率 = 3-8%
即每周全服积分总量增长3-8%

调控杠杆：
- 日工资发放量 = f(DAU, 平均余额, 目标通胀率)
- 入场费 = f(场次等级, 当前通胀率)  
- 若实际通胀 > 目标 → 提高入场费 / 推出限时高消耗活动
- 若实际通胀 < 目标 → 加大日工资 / 降低入场费
```

---

## 链上实现方案

### 不可交易积分的技术选择

| 方案 | 标准 | 优劣 |
|------|------|------|
| ERC-20 + transfer限制 | 重写_beforeTokenTransfer | 兼容性好，工具链完善，但需自行限制 |
| ERC-1238 (NTT) | 原生不可转让同质化代币 | 语义最清晰，但标准未广泛采用 |
| ERC-5727 | 半同质化Soulbound Token | 支持slot分类（积分/经验/等级），但复杂度高 |
| 自定义合约 | 纯自定义mint/burn逻辑 | 最灵活，但丧失标准互操作性 |

**推荐方案**：ERC-20 + `_beforeTokenTransfer` override

```solidity
// 核心逻辑：只允许 mint（from=0）和 burn（to=0），禁止用户间转账
function _beforeTokenTransfer(address from, address to, uint256 amount) internal override {
    require(
        from == address(0) || to == address(0) || hasRole(GAME_SERVER, msg.sender),
        "Transfer blocked: non-tradeable points"
    );
    super._beforeTokenTransfer(from, to, amount);
}
```

### 为什么不用SBT（ERC-5192）

SBT基于ERC-721（NFT），是非同质化的。积分是同质化的——100个积分和另外100个积分没有区别。用NFT标准表达同质化积分是语义错误。

---

## 关键调研来源

- Machinations.io: Game Economy Inflation Detection & Prevention
- 1KX Network: Sinks & Faucets in Virtual Game Economies
- Shima Capital: Web3 Gaming Token Economy (Kevin Eun)
- OpenZeppelin: ERC-20 _beforeTokenTransfer Pattern
- EIP-1238: Non-transferable Token Standard
- EIP-5727: Semi-Fungible Soulbound Token
- QQ欢乐豆经济系统：日工资机制、余额阈值触发模型
