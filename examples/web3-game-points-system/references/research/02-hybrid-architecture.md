# 02 链上vs链下混合架构

## 核心命题

休闲棋牌游戏每局产生大量小额积分变动（一局斗地主可能涉及3-6次余额变更），全部上链既不经济也无必要。关键是找到"哪些数据值得上链"的边界。

---

## 数据分层模型

### 三层架构

```
┌─────────────────────────────────────┐
│  Layer 1: 链上（不可篡改层）          │
│  - 积分总余额快照（定期结算）          │
│  - 大额mint/burn事件                 │
│  - 跨游戏转移记录                    │
│  - 合约治理参数变更                   │
├─────────────────────────────────────┤
│  Layer 2: 链下+证明层                │
│  - 每局对局结果摘要（Merkle Root上链） │
│  - 批量积分变动（Batch Settlement）   │
│  - 道具购买记录                      │
├─────────────────────────────────────┤
│  Layer 3: 纯链下（游戏服务器）         │
│  - 实时对局过程数据                   │
│  - 出牌记录、AI辅助判断              │
│  - 匹配队列、房间状态                │
│  - 临时Buff/道具使用状态              │
└─────────────────────────────────────┘
```

### 上链判断标准

| 判断因素 | 上链 | 链下 |
|---------|------|------|
| 用户是否需要自证拥有？ | 是 | 否 |
| 跨游戏/跨平台是否需要读取？ | 是 | 否 |
| 是否涉及资产所有权变更？ | 是 | 否 |
| 频率是否超过1次/秒？ | 否 | 是 |
| 数据丢失是否可恢复？ | 否→上链 | 是→链下 |

---

## Gas费优化策略

### 策略1: 批量结算（Batch Settlement）

```
游戏服务器累积N局结果 → 一次性上链
- 触发条件：每100局 或 每5分钟 或 单用户余额变动超过阈值
- 链上操作：批量调用 batchUpdateBalances(address[], int256[])
- Gas节省：100次单独update ≈ 300万gas；1次batch ≈ 15万gas（节省95%）
```

### 策略2: Merkle Tree证明

```
每日生成全服余额的 Merkle Tree
- Root Hash 上链（单次写入，约45000 gas）
- 用户可通过 Merkle Proof 自证余额
- 争议时提交proof到链上验证合约
```

### 策略3: Layer 2 选型

| 方案 | Gas成本 | 适用场景 | TPS |
|------|---------|---------|-----|
| Immutable zkEVM | 接近0（游戏免Gas） | 游戏资产，NFT道具 | 4000+ |
| Base (Coinbase L2) | <$0.01/tx | 通用积分结算 | 2000+ |
| Arbitrum | ~$0.01/tx | 复杂合约逻辑 | 4000+ |
| 自建App Chain (OP Stack) | 可控（自定义Gas策略） | 大规模休闲游戏 | 自定义 |

**推荐**：Base或自建OP Stack App Chain
- Base：Coinbase生态，合规友好，用户已有Coinbase账户可无缝接入
- 自建App Chain：完全控制Gas策略，可实现"用户0 Gas"

### 策略4: Paymaster Gas代付

```
ERC-4337 Paymaster合约：
- 游戏方部署Paymaster → 为玩家代付Gas
- 玩家无需持有ETH/原生代币
- 成本由游戏方承担（计入运营成本）
- 每用户每月Gas成本估算：$0.01-0.05（L2环境下）
```

---

## 结算流程设计

### 对局结算时序

```
1. 玩家A/B/C进入牌桌 → 链下：扣除入场费（本地余额）
2. 对局进行中 → 纯链下：出牌/加注/过牌
3. 对局结束 → 链下：计算积分变动
4. 结果写入缓冲队列 → 链下：等待批量提交
5. 批量提交触发 → 链上：batchUpdateBalances + Merkle Root
6. 用户查询余额 → 先查链下缓存，需要证明时查链上
```

### 异常处理

| 场景 | 处理方式 |
|------|---------|
| 对局中服务器宕机 | 链下事务回滚，积分退还到上一个链上快照 |
| 批量提交失败 | 重试3次后报警，期间链下正常运行 |
| 链上链下余额不一致 | 以链上最近快照 + 未提交的链下delta为准 |
| 恶意服务器篡改余额 | Merkle Proof + 争议仲裁合约（类似Optimistic Rollup挑战期） |

---

## 关键调研来源

- EIP-4844: Blob Transaction for Rollup Cost Reduction
- Immutable zkEVM: Gas-Free Gaming Architecture
- Base Network: Sub-cent Transaction Costs Post-Dencun
- OP Stack: App Chain Deployment Documentation
- ERC-4337: Account Abstraction & Paymaster Pattern
- Layer 2 Performance Benchmarks 2025 (markaicode.com)
