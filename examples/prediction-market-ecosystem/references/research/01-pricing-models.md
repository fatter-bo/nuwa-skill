# 01-pricing-models: 预测市场定价模型与条件代币框架

## LMSR（对数市场评分规则）

**发明者**：Robin Hanson
**成本函数**：C(q) = b × log(Σᵢ exp(qᵢ / b))
**边际价格**：Pᵢ = exp(qᵢ / b) / Σₖ exp(qₖ / b)

- b控制流动性深度，最大亏损 = b × ln(n)
- 始终有流动性（无需对手方）
- 价格联动（一个结果概率上升，其他自动下降）
- 使用者：Augur v1、企业预测市场

## CLOB订单簿（Polymarket）

- 链下订单簿撮合 + 链上CTF结算
- Yes/No代币$0-$1.00定价
- 依赖专业做市商（Susquehanna、Jump Trading）
- Maker返佣（负费率鼓励挂单）
- 长期市场4% APR持仓奖励（抵消国债收益率机会成本）

**做市商分布（2026.02 Polymarket）**：
- 中频交易者（11-1000笔）：44.7%交易量
- 高频做市商（>10000笔）：35.2%交易量
- 单次交易者：<0.2%

## vAMM（Azuro虚拟AMM）

- 不需要引导流动性池——从Liquidity Tree单例池继承
- 数据提供者设置基准赔率和Virtual Fund规格
- 投注量动态调整：Fᵢ = Fᵢ - a（投注结果的虚拟资金减少）
- Reinforcement参数封顶最大亏损

## Parimutuel

- 所有同类投注汇入池中
- 扣除抽成后按比例分配给赢家
- 赔率在关闭前不确定
- 零庄家风险（纯中间费用）

## 条件代币框架（Gnosis CTF / ERC-1155）

### 创建流程
1. questionId（UMA辅助数据哈希）+ oracle + outcomeSlotCount → conditionId
2. Collection ID = 父集合 + conditionId + indexSet
3. Position ID = 抵押代币 + Collection ID → ERC-1155 token ID

### 三个操作
- Split：1 USDC → 1 Yes + 1 No
- Merge：1 Yes + 1 No → 1 USDC
- Redeem：结算后赢方代币 → $1.00

### Neg Risk Markets
NegRiskCTFExchange——多结果市场（如「谁会赢大选？」）的特殊变体

### CTF Exchange V2（2026.04公告）
- 重建的交易引擎
- EIP-1271签名支持（智能合约钱包、账户抽象）
- Builder codes用于链上订单归因
- 新CLOB v2
- SDK：TypeScript、Python、Go
