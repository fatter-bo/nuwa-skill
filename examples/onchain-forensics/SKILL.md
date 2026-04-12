---
name: onchain-forensics
description: |
  链上取证分析师技能。使用CLI工具和RPC调用追踪资金流向与交易行为。
  覆盖Foundry/cast完整用法、viem/ethers.js链上查询、Tenderly模拟调试、Phalcon/BlockSec分析、
  交易追踪方法、MEV交易识别、攻击取证、自动化监控。
  触发词：「追踪这笔交易」「资金流向分析」「攻击取证」「MEV识别」「异常监控」「cast查询」。
---

# 链上取证分析 · On-chain Forensics 操作手册

> 链上取证的核心能力：从一个 tx hash 出发，还原完整的资金流向、识别攻击模式、追踪到人。
> 工具是手段，逻辑推理才是根本。

---

## 使用规则

**此 Skill 激活后，以链上取证分析师身份回应。**

### 核心原则
- 一切从 tx hash 开始，逐层展开
- 工具链互补使用，不依赖单一数据源
- 取证结论必须有链上证据支撑，不做无据推测
- 时间线是取证的骨架，所有事件按时间排列

### 触发式工作流

| 用户指令 | 行动 |
|----------|------|
| 「追踪这笔交易」 | → 完整 tx 分析：input decode → internal txs → token transfers → 资金流图 |
| 「资金流向分析」 | → 从目标地址出发，正向/反向追踪资金路径 |
| 「攻击取证」 | → 攻击 tx 逆向分析 → 漏洞定位 → 资金追踪 → 身份线索 |
| 「MEV 识别」 | → 判断 sandwich/backrun/JIT/liquidation 模式 |
| 「异常监控」 | → 设计监控规则 + 告警系统 |
| 「cast 查询」 | → 输出 cast 命令及解析方法 |
| 「反编译合约」 | → 使用 Dedaub/Heimdall 反编译 + 函数签名匹配 |

---

## 一、核心工具链

### 1.1 Foundry / cast 完整用法指南

cast 是链上取证的瑞士军刀，几乎所有链上查询都可以用一行命令完成。

#### 基础查询命令

```bash
# 设置 RPC（建议使用付费节点，公共节点有 rate limit）
export ETH_RPC_URL="https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"

# 查询交易详情
cast tx 0xTXHASH

# 查询交易 receipt（含 logs、gas used、status）
cast receipt 0xTXHASH

# 查询区块信息
cast block 18500000
cast block latest

# 查询账户余额（ETH）
cast balance 0xADDRESS --block 18500000

# 查询 ERC20 余额
cast call 0xTOKEN_CONTRACT "balanceOf(address)(uint256)" 0xADDRESS --block 18500000

# 查询合约 storage slot
cast storage 0xCONTRACT 0 --block 18500000

# 查询合约代码
cast code 0xCONTRACT

# 查询合约代码大小（快速判断是否为合约）
cast codesize 0xADDRESS
```

#### 交易解码与调试

```bash
# 解码 calldata（需要 ABI 或函数签名）
cast calldata-decode "swap(address,uint256,uint256,bytes)" 0xCALLDATA

# 使用 4byte 数据库查找函数签名
cast 4byte 0x38ed1739
# 输出: swapExactTokensForTokens(uint256,uint256,address[],address,uint256)

# 解码 ABI 编码数据
cast abi-decode "swapExactTokensForTokens(uint256,uint256,address[],address,uint256)" 0xDATA

# 解码 event log
cast logs --from-block 18500000 --to-block 18500100 \
  --address 0xUNISWAP_V2_PAIR \
  "Swap(address,uint256,uint256,uint256,uint256,address)"

# 查询指定事件
cast logs --from-block 18500000 --to-block 18500100 \
  --address 0xTOKEN \
  "Transfer(address indexed, address indexed, uint256)"

# Trace 交易（需要 archive node 或支持 debug_traceTransaction 的节点）
cast run 0xTXHASH --trace
```

#### 高级取证命令

```bash
# 查询合约创建者
cast etherscan-source 0xCONTRACT --chain mainnet

# 获取合约 ABI（从 Etherscan）
cast etherscan-source --abi 0xCONTRACT

# ENS 解析
cast resolve-name vitalik.eth
cast lookup-address 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

# 签名恢复（从签名推导 signer 地址）
cast ecrecover "message" 0xSIGNATURE

# Keccak256 计算（用于匹配 event topic）
cast keccak "Transfer(address,address,uint256)"

# 时间戳转换
cast age 18500000
cast to-dec 0x112a880

# Nonce 查询（追踪地址活跃度）
cast nonce 0xADDRESS

# 多链查询（指定 RPC）
cast balance 0xADDRESS --rpc-url https://arb-mainnet.g.alchemy.com/v2/KEY
```

### 1.2 viem / ethers.js 链上查询

#### viem 查询示例

```typescript
import { createPublicClient, http, parseAbiItem } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: http('https://eth-mainnet.g.alchemy.com/v2/KEY'),
});

// 获取交易详情
const tx = await client.getTransaction({ hash: '0xTXHASH' as `0x${string}` });

// 获取交易 receipt
const receipt = await client.getTransactionReceipt({ hash: '0xTXHASH' as `0x${string}` });

// 获取 logs（核心取证手段）
const logs = await client.getLogs({
  address: '0xTOKEN' as `0x${string}`,
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
  fromBlock: 18500000n,
  toBlock: 18500100n,
});

// 批量查询多个地址的 ERC20 余额
const balances = await Promise.all(
  addresses.map(addr =>
    client.readContract({
      address: tokenAddress,
      abi: erc20Abi,
      functionName: 'balanceOf',
      args: [addr],
    })
  )
);

// Trace 交易（需要节点支持）
const trace = await client.request({
  method: 'debug_traceTransaction' as any,
  params: ['0xTXHASH', { tracer: 'callTracer' }],
});
```

#### ethers.js 查询示例

```typescript
import { ethers } from 'ethers';

const provider = new ethers.JsonRpcProvider('https://eth-mainnet.g.alchemy.com/v2/KEY');

// 获取 internal transactions（通过 trace）
const trace = await provider.send('debug_traceTransaction', [
  '0xTXHASH',
  { tracer: 'callTracer', tracerConfig: { onlyTopCall: false } },
]);

// 遍历 trace 树重建资金流
function extractCalls(trace: any, depth = 0): any[] {
  const calls = [{
    type: trace.type,
    from: trace.from,
    to: trace.to,
    value: trace.value,
    input: trace.input,
    output: trace.output,
    depth,
  }];
  if (trace.calls) {
    for (const subcall of trace.calls) {
      calls.push(...extractCalls(subcall, depth + 1));
    }
  }
  return calls;
}
```

### 1.3 Tenderly 模拟与调试

Tenderly 提供交易模拟和可视化 debugger，是取证分析的关键工具。

#### 核心功能

| 功能 | 用途 | 操作 |
|------|------|------|
| **Transaction Debugger** | 逐步执行交易，查看 stack/memory/storage 变化 | 粘贴 tx hash 进入 debugger |
| **Simulation** | 模拟交易执行而不实际上链 | Fork + simulate API |
| **State Diff** | 查看交易前后 storage 变化 | tx 页面 → State Changes |
| **Event Logs** | 解码后的 event 展示 | tx 页面 → Events |
| **Gas Profiler** | 逐 opcode 的 gas 消耗 | tx 页面 → Gas Profiler |

#### Tenderly API 模拟

```bash
# 模拟交易（不上链执行）
curl -X POST "https://api.tenderly.co/api/v1/account/ACCOUNT/project/PROJECT/simulate" \
  -H "X-Access-Key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "network_id": "1",
    "from": "0xFROM",
    "to": "0xTO",
    "input": "0xCALLDATA",
    "value": "0",
    "block_number": 18500000,
    "save": true,
    "save_if_fails": true,
    "simulation_type": "full"
  }'
```

### 1.4 Phalcon / BlockSec 分析平台

#### Phalcon Explorer

Phalcon（https://explorer.phalcon.xyz/）是 BlockSec 出品的交易分析工具，特别擅长：
- **资金流可视化**：自动绘制 token 转移图
- **调用树展开**：多层嵌套调用一目了然
- **攻击交易分析**：自动标注可疑调用
- **Balance Changes**：交易前后各地址余额变化

#### 使用方法

```
1. 访问 https://explorer.phalcon.xyz/
2. 输入 tx hash
3. 查看 Invocation Flow（调用流）
4. 查看 Balance Changes（余额变化）
5. 查看 Fund Flow（资金流图）
```

#### 工具对比

| 工具 | 优势 | 局限 | 适用场景 |
|------|------|------|----------|
| **cast/Foundry** | 快速、CLI、可脚本化 | 需 archive node | 自动化批量分析 |
| **Tenderly** | 可视化调试、模拟 | 免费版限制 | 交易调试、PoC 验证 |
| **Phalcon** | 资金流图、自动解码 | 仅查看，不可编程 | 攻击分析、资金追踪 |
| **Etherscan** | 最全面的基础信息 | 无 trace、无模拟 | 基础查询、合约验证 |
| **Dedaub** | 合约反编译最佳 | 仅合约分析 | 未验证合约分析 |
| **Heimdall** | CLI 反编译 | 准确度有限 | 快速反编译、自动化 |

---

## 二、交易追踪方法

### 2.1 从 tx hash 出发的完整分析流程

```
Step 1: 基础信息
  cast tx 0xHASH
  → from, to, value, input, gasPrice, blockNumber

Step 2: 执行结果
  cast receipt 0xHASH
  → status (1=成功/0=失败), gasUsed, logs

Step 3: Input 解码
  cast 4byte <selector>
  → 识别调用的函数
  cast calldata-decode "functionSig" <input>
  → 解码参数

Step 4: 内部交易追踪
  cast run 0xHASH --trace
  → 或使用 debug_traceTransaction + callTracer
  → 重建完整调用树

Step 5: Event Logs 解析
  遍历 receipt.logs
  → 解码每个 event
  → 重建 token 转移、状态变化

Step 6: 资金流图重建
  从 Transfer events 提取:
  → (token, from, to, amount) 列表
  → 绘制有向图

Step 7: 上下文分析
  → 分析 from 地址历史（Etherscan/cast nonce）
  → 分析合约创建时间
  → 分析 gas price 是否异常（MEV 相关）
```

### 2.2 Internal Transactions 追踪

Internal transactions 是合约间调用产生的 ETH 转移，不记录在 receipt 中，必须通过 trace 获取。

```bash
# 使用 callTracer 获取完整调用树
cast rpc debug_traceTransaction 0xTXHASH \
  '{"tracer":"callTracer","tracerConfig":{"onlyTopCall":false}}'

# 使用 Alchemy 的 trace API（更高效）
curl -X POST "https://eth-mainnet.g.alchemy.com/v2/KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "method": "trace_transaction",
    "params": ["0xTXHASH"],
    "id": 1,
    "jsonrpc": "2.0"
  }'
```

#### 调用树结构解读

```
CALL (EOA → Router)
├── STATICCALL (Router → Factory.getPool)
├── CALL (Router → Pool.swap)
│   ├── CALL (Pool → Token0.transfer)   ← token 转出
│   └── CALL (Pool → Callback)
│       └── CALL (Callback → Token1.transferFrom)  ← token 转入
└── CALL (Router → WETH.withdraw)
    └── CALL (WETH → EOA)  ← ETH 转出 (internal tx)
```

### 2.3 Token Transfer 资金流重建

```typescript
// 从 receipt logs 重建 ERC20 资金流
const TRANSFER_TOPIC = '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef';

function reconstructTokenFlows(receipt: any): TokenTransfer[] {
  return receipt.logs
    .filter((log: any) => log.topics[0] === TRANSFER_TOPIC)
    .map((log: any) => ({
      token: log.address,
      from: '0x' + log.topics[1].slice(26),
      to: '0x' + log.topics[2].slice(26),
      amount: BigInt(log.data),
    }));
}

// ERC721 Transfer 也使用相同 topic0，区别在于 topics[3] 存在（tokenId）
// ERC1155 使用 TransferSingle/TransferBatch
const TRANSFER_SINGLE_TOPIC = '0xc3d58168c5ae7397731d063d5bbf3d657854427343f4c083240f7aacaa2d0f62';
```

### 2.4 合约创建与 selfdestruct 追踪

```bash
# 查询合约创建交易
# 合约创建交易的 to 为 null
cast tx 0xCREATION_TX_HASH
# → to: null, input: creation bytecode

# 查看合约是否已 selfdestruct
cast code 0xCONTRACT
# 返回 0x 表示已销毁或非合约

# 追踪合约创建者
# 方法1: Etherscan API
curl "https://api.etherscan.io/api?module=contract&action=getcontractcreation&contractaddresses=0xCONTRACT&apikey=KEY"

# 方法2: 通过 trace 查找 CREATE/CREATE2 opcode
cast rpc debug_traceTransaction 0xTXHASH \
  '{"tracer":"prestateTracer"}'
```

---

## 三、MEV 交易识别

### 3.1 Sandwich Attack 链上特征

```
典型 Sandwich 结构（同一区块内）：

Tx N-1:  Attacker → DEX.swap(tokenA→tokenB)     ← frontrun
Tx N:    Victim   → DEX.swap(tokenA→tokenB)      ← 被夹交易
Tx N+1:  Attacker → DEX.swap(tokenB→tokenA)      ← backrun

链上识别特征：
1. 三笔 tx 在同一区块
2. 前后两笔 tx 的 from 地址相同（或属于同一 MEV bot）
3. 交易对相同，方向相反
4. frontrun 买入 + 受害者买入推高价格 + backrun 卖出获利
5. 通常 gas price: frontrun > victim > backrun（或使用 Flashbots bundle）
```

#### 识别脚本

```typescript
async function detectSandwich(blockNumber: number) {
  const block = await client.getBlock({ blockNumber: BigInt(blockNumber), includeTransactions: true });
  const swapTopic = '0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822'; // Uniswap V2

  // 获取该区块所有 swap events
  const logs = await client.getLogs({
    fromBlock: BigInt(blockNumber),
    toBlock: BigInt(blockNumber),
    topics: [swapTopic],
  });

  // 按 pool 地址分组
  const poolSwaps = new Map<string, any[]>();
  for (const log of logs) {
    const pool = log.address.toLowerCase();
    if (!poolSwaps.has(pool)) poolSwaps.set(pool, []);
    poolSwaps.get(pool)!.push(log);
  }

  // 检测 sandwich 模式：同一 pool 在同一区块有 3+ 笔 swap
  for (const [pool, swaps] of poolSwaps) {
    if (swaps.length >= 3) {
      // 进一步分析：检查 tx sender 和 swap 方向
      console.log(`Potential sandwich at pool ${pool}, ${swaps.length} swaps in block ${blockNumber}`);
    }
  }
}
```

### 3.2 Backrun Arbitrage 模式

```
典型 Backrun 结构：

Tx N:   大额 swap 导致价格偏移
Tx N+1: Arbitrageur 在多个 DEX 之间套利

链上特征：
1. N+1 紧跟 N（通常在同一 bundle 或相邻位置）
2. N+1 的 from 地址是已知 MEV bot
3. N+1 涉及多个 DEX pool
4. N+1 起始和结束 token 相同（循环套利）
5. N+1 的 ETH 余额净增加
```

### 3.3 JIT Liquidity 检测

```
JIT (Just-In-Time) Liquidity 结构：

Tx N-1: LP mint（添加集中流动性到即将被交易的 tick range）
Tx N:   大额 swap（LP 赚取手续费）
Tx N+1: LP burn（移除流动性 + 收集手续费）

检测方法：
1. 在 Uniswap V3 pool 中查找同一区块的 Mint → Swap → Burn 序列
2. Mint 和 Burn 的 owner 相同
3. Mint 的 tickLower/tickUpper 精确覆盖 swap 的价格范围
4. 从 mint 到 burn 间隔极短（通常同一区块）
```

### 3.4 Flashbots Bundle 识别

```bash
# 查询 Flashbots 的 bundle 数据
curl -X POST "https://relay.flashbots.net" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "flashbots_getBundleStatsV2",
    "params": [{ "bundleHash": "0xBUNDLE_HASH", "blockNumber": "0x11A5B00" }],
    "id": 1
  }'

# 使用 mev-inspect-py（开源 MEV 检测工具）
# https://github.com/flashbots/mev-inspect-py
```

#### Flashbots Blocks API

```bash
# 获取指定区块的所有 Flashbots bundle
curl "https://blocks.flashbots.net/v1/blocks?block_number=18500000"

# 返回格式包含：
# - bundle_hash
# - transactions[]（bundle 内的有序交易列表）
# - miner_reward（支付给验证者的费用）
# - total_gas_used
# - profit（MEV 利润）
```

---

## 四、攻击取证完整流程

### 4.1 攻击 tx 逆向分析

```
完整取证流程：

Phase 1: 事件确认
├── 监控告警触发 / 社区报告
├── 确认受影响协议和金额
└── 收集所有相关 tx hash

Phase 2: 交易分析
├── 在 Phalcon 中打开攻击 tx
├── 分析调用树：哪些合约被调用
├── 分析资金流：钱从哪里来、到哪里去
├── 分析 State Changes：哪些 storage slot 被修改
└── 识别攻击向量：重入/闪电贷/预言机操纵/逻辑漏洞

Phase 3: 漏洞定位
├── 获取受影响合约源码（verified 或反编译）
├── 定位被利用的函数
├── 重建攻击逻辑
└── 编写 PoC 验证

Phase 4: 资金追踪
├── 追踪攻击者地址的后续操作
├── 是否通过 Tornado Cash / Railgun 混币
├── 是否通过 CEX 出金
├── 跨链转移追踪
└── 时间线记录

Phase 5: 身份线索
├── 攻击者地址的历史交易
├── 合约创建者关联
├── Gas funding 来源
├── ENS / 社交媒体关联
├── CEX deposit 地址关联
└── 链上留言（calldata 中的消息）
```

### 4.2 攻击合约反编译

```bash
# 方法1: Dedaub 在线反编译
# https://app.dedaub.com/decompile
# 粘贴 bytecode 或合约地址

# 方法2: Heimdall CLI
heimdall decompile 0xATTACK_CONTRACT --rpc-url $ETH_RPC_URL

# 方法3: Panoramix（Dedaub 的开源版本）
panoramix 0xBYTECODE

# 方法4: cast disassemble（获取 opcode）
cast disassemble 0xBYTECODE
```

#### 函数签名匹配

```bash
# 从 bytecode 提取 function selector
cast selectors $(cast code 0xCONTRACT)

# 在 4byte.directory 查询
cast 4byte 0x38ed1739

# 在 sig.eth 查询
# https://sig.eth.samczsun.com/
curl "https://sig.eth.samczsun.com/api/v1/signatures?function=0x38ed1739"
```

### 4.3 资金追踪：攻击者 → 混币器

```
资金追踪路径示例：

Attacker EOA
  → 攻击合约执行（获得被盗资金）
  → Attacker EOA（收回资金）
  → 中间地址1（分散资金）
  ├── 中间地址2 → Tornado Cash 0.1 ETH pool
  ├── 中间地址3 → Tornado Cash 1 ETH pool
  ├── 中间地址4 → Tornado Cash 10 ETH pool
  └── 中间地址5 → CEX deposit address ← 关键线索!

追踪工具：
- Chainalysis Reactor（商业）
- Arkham Intelligence（半商业）
- Breadcrumbs（免费基础版）
- 手动 cast + Etherscan 追踪
```

#### 自动化追踪脚本

```typescript
import { createPublicClient, http, parseAbiItem } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: http(process.env.ETH_RPC_URL!),
});

const TRANSFER_EVENT = parseAbiItem(
  'event Transfer(address indexed from, address indexed to, uint256 value)'
);

// 正向追踪：从攻击者地址出发，追踪资金去向
async function traceForward(address: string, token: string, depth = 3) {
  if (depth === 0) return;

  const logs = await client.getLogs({
    address: token as `0x${string}`,
    event: TRANSFER_EVENT,
    args: { from: address as `0x${string}` },
    fromBlock: 'earliest',
    toBlock: 'latest',
  });

  for (const log of logs) {
    const to = log.args.to!;
    const amount = log.args.value!;
    console.log(`${address} → ${to}: ${amount}`);

    // 检查是否为已知混币器/CEX
    const label = await checkAddressLabel(to);
    if (label) {
      console.log(`  [!] Known entity: ${label}`);
    }

    // 递归追踪
    await traceForward(to, token, depth - 1);
  }
}

async function checkAddressLabel(address: string): Promise<string | null> {
  // 已知地址标签库
  const KNOWN_ADDRESSES: Record<string, string> = {
    '0xd90e2f925da726b50c4ed8d0fb90ad053324f31b': 'Tornado Cash Router',
    '0x722122df12d4e14e13ac3b6895a86e84145b6967': 'Tornado Cash Proxy',
    '0x28c6c06298d514db089934071355e5743bf21d60': 'Binance Hot Wallet',
    '0x21a31ee1afc51d94c2efccaa2092ad1028285549': 'Binance Hot Wallet 2',
  };
  return KNOWN_ADDRESSES[address.toLowerCase()] || null;
}
```

### 4.4 真实攻击取证案例：Euler Finance (2023.03.13)

```
攻击概要：
- 被盗金额: ~$197M
- 漏洞类型: 捐赠攻击 + 清算逻辑缺陷
- 攻击者最终归还了全部资金

取证时间线：
1. 攻击 tx: 0xc310a0af...
   → cast tx 0xc310a0af...
   → 在 Phalcon 中打开查看调用树

2. 攻击步骤还原：
   a. 闪电贷借入大量 DAI
   b. 存入 Euler 作为抵押品
   c. 利用 donateToReserves 函数让自己的头寸变成可清算状态
   d. 自我清算，由于 liquidation discount 计算缺陷获利
   e. 偿还闪电贷

3. 资金追踪：
   → 攻击者将资金转入多个地址
   → 部分转入 Tornado Cash
   → 通过链上消息与 Euler 团队沟通
   → 最终归还全部资金

4. 身份线索：
   → 攻击者 gas funding 来源
   → 链上消息内容分析
   → 地址关联分析
```

---

## 五、自动化监控

### 5.1 异常检测规则设计

```typescript
// 监控规则配置
interface MonitorRule {
  name: string;
  type: 'large_transfer' | 'contract_interaction' | 'flash_loan' | 'price_deviation';
  params: Record<string, any>;
  severity: 'info' | 'warning' | 'critical';
}

const rules: MonitorRule[] = [
  {
    name: '大额 ETH 转账',
    type: 'large_transfer',
    params: { token: 'ETH', threshold: '100' /* ETH */ },
    severity: 'warning',
  },
  {
    name: '大额稳定币转账',
    type: 'large_transfer',
    params: {
      tokens: ['USDC', 'USDT', 'DAI'],
      threshold: '1000000', // $1M
    },
    severity: 'warning',
  },
  {
    name: '闪电贷使用',
    type: 'flash_loan',
    params: {
      protocols: ['aave', 'dydx', 'balancer'],
      minAmount: '500000', // $500K
    },
    severity: 'critical',
  },
  {
    name: '新合约异常交互',
    type: 'contract_interaction',
    params: {
      targetContracts: ['0xPROTOCOL_CONTRACT'],
      callerMaxAge: 86400, // 合约创建不超过 24 小时
    },
    severity: 'critical',
  },
];
```

### 5.2 大额转账监控

```typescript
import { createPublicClient, webSocket, parseAbiItem, formatEther, formatUnits } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: webSocket('wss://eth-mainnet.g.alchemy.com/v2/KEY'),
});

// 监控 ERC20 大额转账
const USDC = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';
const THRESHOLD = 1_000_000n * 10n ** 6n; // $1M USDC

const unwatch = client.watchEvent({
  address: USDC as `0x${string}`,
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
  onLogs: (logs) => {
    for (const log of logs) {
      if (log.args.value! >= THRESHOLD) {
        const amount = formatUnits(log.args.value!, 6);
        console.log(`[ALERT] Large USDC transfer: $${amount}`);
        console.log(`  From: ${log.args.from}`);
        console.log(`  To: ${log.args.to}`);
        console.log(`  Tx: ${log.transactionHash}`);
        // 发送告警通知
        sendAlert({
          type: 'large_transfer',
          token: 'USDC',
          amount,
          from: log.args.from!,
          to: log.args.to!,
          txHash: log.transactionHash!,
        });
      }
    }
  },
});

// 监控大额 ETH 转账（通过 pending transactions）
client.watchPendingTransactions({
  onTransactions: async (hashes) => {
    for (const hash of hashes) {
      try {
        const tx = await client.getTransaction({ hash });
        if (tx.value >= BigInt('100000000000000000000')) { // 100 ETH
          console.log(`[ALERT] Large ETH transfer: ${formatEther(tx.value)} ETH`);
          console.log(`  From: ${tx.from}, To: ${tx.to}`);
        }
      } catch {}
    }
  },
});
```

### 5.3 合约交互异常检测

```typescript
// 检测未验证合约的异常调用
async function checkCallerContract(callerAddress: string): Promise<boolean> {
  // 1. 检查是否为合约
  const code = await client.getCode({ address: callerAddress as `0x${string}` });
  if (!code || code === '0x') return false; // EOA，不告警

  // 2. 检查合约年龄
  // 通过 Etherscan API 获取创建时间
  const response = await fetch(
    `https://api.etherscan.io/api?module=contract&action=getcontractcreation&contractaddresses=${callerAddress}&apikey=KEY`
  );
  const data = await response.json();
  const creationTx = data.result?.[0]?.txHash;
  if (!creationTx) return false;

  const receipt = await client.getTransactionReceipt({ hash: creationTx });
  const block = await client.getBlock({ blockNumber: receipt.blockNumber });
  const ageSeconds = Date.now() / 1000 - Number(block.timestamp);

  // 合约创建不足 1 小时 → 高风险
  if (ageSeconds < 3600) {
    console.log(`[CRITICAL] New contract (${Math.floor(ageSeconds / 60)} min old) interacting with protocol`);
    return true;
  }

  return false;
}
```

### 5.4 告警系统搭建

```typescript
// 告警通道配置
interface AlertConfig {
  telegram: { botToken: string; chatId: string };
  discord: { webhookUrl: string };
  email: { smtp: any; recipients: string[] };
}

async function sendAlert(alert: {
  type: string;
  severity?: string;
  message?: string;
  [key: string]: any;
}) {
  const message = formatAlertMessage(alert);

  // Telegram 告警
  await fetch(`https://api.telegram.org/bot${config.telegram.botToken}/sendMessage`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      chat_id: config.telegram.chatId,
      text: message,
      parse_mode: 'HTML',
    }),
  });

  // Discord Webhook 告警
  await fetch(config.discord.webhookUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ content: message }),
  });
}

function formatAlertMessage(alert: any): string {
  return `🚨 <b>${alert.type.toUpperCase()}</b>\n` +
    `Severity: ${alert.severity || 'unknown'}\n` +
    `Token: ${alert.token || '-'}\n` +
    `Amount: ${alert.amount || '-'}\n` +
    `From: ${alert.from || '-'}\n` +
    `To: ${alert.to || '-'}\n` +
    `Tx: https://etherscan.io/tx/${alert.txHash}`;
}
```

---

## 六、取证 Checklist

### 快速取证清单

```
□ 收集所有相关 tx hash
□ 确认受影响协议和资金规模
□ 在 Phalcon 中分析攻击 tx 调用树
□ 重建 token 转移资金流图
□ 下载/反编译攻击合约代码
□ 定位被利用的漏洞
□ 追踪攻击者地址的资金去向
□ 检查是否使用 Tornado Cash / 跨链桥
□ 查找攻击者身份线索（gas funding、ENS、历史交易）
□ 编写取证报告（时间线 + 技术分析 + 资金追踪）
□ 提交链上证据给执法机构/安全团队
```

### 常用 RPC 节点

| 提供商 | Archive Node | Trace API | 免费额度 |
|--------|-------------|-----------|----------|
| **Alchemy** | 是 | debug_traceTransaction | 300M CU/月 |
| **Infura** | 付费 | 不支持 | 100K req/天 |
| **QuickNode** | 是 | trace_transaction | 有限 |
| **Ankr** | 是 | 有限支持 | 无限制（限速） |
| **Erigon 自建** | 是 | 全部支持 | 无限制 |

---

## 关联技能

- [foundry-security-testing](../foundry-security-testing/SKILL.md) — 使用 Foundry 编写漏洞 PoC 验证取证发现
- [dune-analytics-mev](../dune-analytics-mev/SKILL.md) — 使用 Dune SQL 批量分析 MEV 活动
- [event-log-parsing](../event-log-parsing/SKILL.md) — EVM 事件日志解析，取证数据的核心来源
