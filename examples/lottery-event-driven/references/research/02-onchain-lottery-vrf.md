# 02-onchain-lottery-vrf: 链上彩票方案与VRF

## PoolTogether V5

### 工作原理
1. 用户存入代币到ERC-4626 Prize Vault
2. 存款路由到Aave等收益协议
3. 累积利息汇集为奖池
4. Chainlink VRF选择中奖者
5. 用户随时取回本金（无损）

### V5架构（2024.04）
- 无许可Prize Vault
- 自动奖金分配（Chainlink Automation）
- 分层奖金：prizeCount = 4^t
- Canary tiers：自适应扩展/收缩
- Draw auctions：两阶段RNG（Start RNG + Finish RNG）
- TWAB：时间加权平均余额决定中奖概率

### 指标
- 20,000+活跃玩家
- $500万+已发放
- 部署：Ethereum、Polygon、Optimism、Arbitrum、Base

## Chainlink VRF

### 机制
1. 合约发送随机数请求到VRF Coordinator
2. VRF服务结合区块数据+节点私钥
3. 生成随机数+密码学证明
4. 链上验证证明后接受

### VRF v2.5成本
- Gas成本：回调gas+验证gas
- LINK premium：gas成本的百分比
- 支付选项：LINK（更便宜）或原生代币
- 订阅模式（推荐）vs 直接付费

### VRF限制
1. 每次请求有成本（日开奖可以，每场比赛太贵）
2. 多区块延迟
3. 链特定
4. 只生成随机数——不提供现实世界数据
5. 不适合事件驱动结果

## 其他链上彩票

| 项目 | 链 | 融资 | 特点 |
|------|-----|------|------|
| Megapot | Base | $500万（Dragonfly/Coinbase） | $1票，日开奖 |
| Solotto | Solana | — | Pump.Fun Ascend，持$LOTTO参与 |
| PancakeSwap Lottery | BNB | — | DEX内置 |

## VRF vs 事件预言机对比

| 维度 | VRF | FastEvent |
|------|-----|-----------|
| 输出 | 随机数 | 比赛结果 |
| 触发 | 定时/手动 | 比赛结束 |
| 用户参与 | 被动 | 主动预测 |
| 娱乐性 | 低 | 高 |
| 适用 | 纯抽奖 | 体育竞猜 |
| 互补性 | 可结合 | 可结合 |
