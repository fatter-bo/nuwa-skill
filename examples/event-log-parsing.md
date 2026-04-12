---
name: event-log-parsing
description: |
  EVM事件日志解析专家技能。解码和实时处理链上Event Logs。
  覆盖EVM Log结构、topics/indexed参数、核心DeFi事件解析、
  高性能日志检索、实时处理管道、多语言实现（TypeScript/Rust/Go/Python）。
  触发词：「解析日志」「事件监听」「topic0查询」「Swap事件」「实时处理」「eth_getLogs」。
---

# EVM Event Log 解析 · 操作手册

> Event logs 是链上数据的核心——每一笔 Swap、Mint、Burn、Transfer 都通过 logs 记录。
> 理解 log 结构，就掌握了链上数据分析的基本功。

---

## 使用规则

**此 Skill 激活后，以 EVM 事件日志解析专家身份回应。**

### 核心原则
- 解析前先确认 event signature 和 topic0
- indexed 参数在 topics 中，非 indexed 参数在 data 中
- 高效检索的关键是 filter 设计和 block range 控制
- 实时处理必须考虑 reorg 和 missed blocks

### 触发式工作流

| 用户指令 | 行动 |
|----------|------|
| 「解析日志」 | → 解码指定 tx 或地址的 event logs |
| 「事件监听」 | → 输出 WebSocket 实时监听代码 |
| 「topic0 查询」 | → 查找事件签名对应的 topic0 |
| 「Swap 事件」 | → 输出 Uniswap V2/V3 Swap 事件解析代码 |
| 「实时处理」 | → 设计 event → 业务信号 处理管道 |
| 「eth_getLogs」 | → 输出高效的日志检索方案 |

---

## 一、EVM Log 基础

### 1.1 Log 数据结构

每个 EVM event log 包含以下字段：

```
Log {
  address:     0x...     // 产生事件的合约地址
  topics:      [         // 最多 4 个 topic（每个 32 bytes）
    bytes32,             // topics[0] = event signature hash (keccak256)
    bytes32,             // topics[1] = 第 1 个 indexed 参数
    bytes32,             // topics[2] = 第 2 个 indexed 参数
    bytes32,             // topics[3] = 第 3 个 indexed 参数
  ]
  data:        0x...     // 非 indexed 参数的 ABI 编码
  blockNumber: uint256
  blockHash:   bytes32
  transactionHash: bytes32
  transactionIndex: uint256
  logIndex:    uint256
  removed:     bool      // 是否因 reorg 被移除
}
```

### 1.2 Event Signature 与 topic0

topic0 是事件签名的 keccak256 哈希：

```
event Transfer(address indexed from, address indexed to, uint256 value)

topic0 = keccak256("Transfer(address,address,uint256)")
       = 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```

**计算规则：**
- 函数名 + 参数类型列表（不含参数名、不含 indexed 关键字）
- 参数类型之间无空格
- uint256 不简写为 uint
- 数组类型如 `uint256[]` 保留

```bash
# 使用 cast 计算 topic0
cast keccak "Transfer(address,address,uint256)"
# 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```

### 1.3 indexed 与 non-indexed 参数

```solidity
event Swap(
    address indexed sender,      // → topics[1]（indexed，可用于 filter）
    uint256 amount0In,           // → data（非 indexed，ABI 编码）
    uint256 amount1In,           // → data
    uint256 amount0Out,          // → data
    uint256 amount1Out,          // → data
    address indexed to           // → topics[2]
);
```

**关键区别：**

| 特性 | indexed 参数 | non-indexed 参数 |
|------|-------------|-----------------|
| 存储位置 | topics[1-3] | data |
| 可否 filter | 可以（eth_getLogs filter） | 不可以 |
| 最大数量 | 3 个（普通 event）/ 4 个（anonymous event） | 无限制 |
| 值类型 | 直接存储（地址左补零到 32 bytes） | ABI 编码 |
| 动态类型 indexed | 存储 keccak256 哈希（如 string indexed） | 存储原始值 |
| Gas 消耗 | 每个 topic 375 gas | 8 gas/byte |

### 1.4 Anonymous Events

```solidity
// anonymous event 没有 topic0（事件签名不放入 topics）
// 可以有最多 4 个 indexed 参数（普通 event 最多 3 个）
event AnonymousTransfer(
    address indexed from,
    address indexed to,
    uint256 indexed amount,
    uint256 indexed tokenId
) anonymous;
// topics = [from, to, amount, tokenId]（没有 signature hash）
```

---

## 二、核心 DeFi Event 完整参考

### 2.1 完整 Topic0 速查表

```
=== ERC20 ===
Transfer(address,address,uint256)
  0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef

Approval(address,address,uint256)
  0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925

=== ERC721 ===
Transfer(address,address,uint256)   // 与 ERC20 相同的 topic0!
  0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
  // 区别：ERC721 的 tokenId 是 indexed（在 topics[3]），ERC20 value 在 data

ApprovalForAll(address,address,bool)
  0x17307eab39ab6107e8899845ad3d59bd9653f200f220920489ca2b5937696c31

=== ERC1155 ===
TransferSingle(address,address,address,uint256,uint256)
  0xc3d58168c5ae7397731d063d5bbf3d657854427343f4c083240f7aacaa2d0f62

TransferBatch(address,address,address,uint256[],uint256[])
  0x4a39dc06d4c0dbc64b70af90fd698a233a518aa5d07e595d983b8c0526c8f7fb

=== Uniswap V2 ===
Swap(address,uint256,uint256,uint256,uint256,address)
  0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822

Mint(address,uint256,uint256)
  0x4c209b5fc8ad50758f13e2e1088ba56a560dff690a1c6fef26394f4c03821c4f

Burn(address,uint256,uint256,address)
  0xdccd412f0b1252819cb1fd330b93224ca42612892bb3f4f789976e6d81936496

Sync(uint112,uint112)
  0x1c411e9a96e071241c2f21f7726b17ae89e3cab4c78be50e062b03a9fffbbad1

=== Uniswap V3 ===
Swap(address,address,int256,int256,uint160,uint128,int24)
  0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67

Mint(address,address,int24,int24,uint128,uint256,uint256)
  0x7a53080ba414158be7ec69b987b5fb7d07dee101fe85488f0853ae16239d0bde

Burn(address,int24,int24,uint128,uint256,uint256)
  0x0c396cd989a39f4459b5fa1aed6a9a8dcdbc45908acfd67e028cd568da98982c

Collect(address,address,int24,int24,uint128,uint128)
  0x70935338e69775456a85ddef226c395fb668b63fa0115f5f20610b388e6ca9c0

Flash(address,address,uint256,uint256,uint256,uint256)
  0xbdbdb71d7860376ba52b25a5028beea23581364a40522f6bcfb86bb1f2dca633

=== Aave V3 ===
Supply(address,address,address,uint256,uint16)
  0x2b627736bca15cd5381dcf80b0bf11fd197d01a037c52b927a881a10fb73ba61

Borrow(address,address,address,uint256,uint8,uint256,uint16)
  0xb3d084820fb1a9decffb176436bd02558d15fac9b0ddfed8c465bc7359d7dce0

Repay(address,address,address,uint256,bool)
  0xa534c8dbe71f871f9f3f77571e500a3eb0116d54455458b05e05f5f7bf6003b8

LiquidationCall(address,address,address,uint256,uint256,address,bool)
  0xe413a321e8681d831f4dbccbca790d2952b56f977908e45be37335533e005286

=== Chainlink ===
AnswerUpdated(int256,uint256,uint256)
  0x0559884fd3a460db3073b7fc896cc77986f16e378210ded43186175bf646fc5f
```

### 2.2 Uniswap V3 Swap 事件详解

```
event Swap(
    address indexed sender,     // topics[1]: 调用 swap 的合约/地址
    address indexed recipient,  // topics[2]: 接收 token 的地址
    int256 amount0,             // data[0:32]: token0 数量变化（正=流入pool，负=流出pool）
    int256 amount1,             // data[32:64]: token1 数量变化
    uint160 sqrtPriceX96,       // data[64:96]: swap 后的 sqrt(price) * 2^96
    uint128 liquidity,          // data[96:128]: swap 时 pool 的流动性
    int24 tick                  // data[128:160]: swap 后的 tick
);

价格计算：
  price = (sqrtPriceX96 / 2^96)^2
  
  如果 token0 = WETH (18 decimals), token1 = USDC (6 decimals):
  price_token0_in_token1 = (sqrtPriceX96 / 2^96)^2 * 10^(18-6)
```

### 2.3 Uniswap V2 Swap 事件详解

```
event Swap(
    address indexed sender,     // topics[1]
    uint256 amount0In,          // data[0:32]: token0 输入量
    uint256 amount1In,          // data[32:64]: token1 输入量
    uint256 amount0Out,         // data[64:96]: token0 输出量
    uint256 amount1Out,         // data[96:128]: token1 输出量
    address indexed to          // topics[2]: 接收地址
);

方向判断：
  if amount0In > 0 && amount1Out > 0 → 卖 token0 买 token1
  if amount1In > 0 && amount0Out > 0 → 卖 token1 买 token0
```

### 2.4 ERC20 Transfer 的 ERC721 区分

```
ERC20 和 ERC721 的 Transfer 事件有相同的 topic0！

区分方法：
1. ERC20 Transfer: topics.length == 3, value 在 data 中
   topics = [topic0, from (indexed), to (indexed)]
   data = value (uint256)

2. ERC721 Transfer: topics.length == 4, tokenId 在 topics[3] 中
   topics = [topic0, from (indexed), to (indexed), tokenId (indexed)]
   data = 0x (空)

代码判断：
  if (log.topics.length === 4) → ERC721
  if (log.topics.length === 3) → ERC20 (或 topics[3] 缺失的 ERC721，需进一步确认)
```

---

## 三、高性能日志检索

### 3.1 eth_getLogs 高效使用

```typescript
// eth_getLogs RPC 调用参数
interface GetLogsParams {
  address?: string | string[];    // 合约地址（可多个）
  topics?: (string | string[] | null)[];  // topic filter
  fromBlock?: string;             // 起始区块
  toBlock?: string;               // 结束区块
  blockHash?: string;             // 指定区块（与 fromBlock/toBlock 互斥）
}
```

#### Filter 设计原则

```typescript
// 1. 最高效：address + topic0 + indexed param
const filter = {
  address: '0xUNISWAP_V3_POOL',
  topics: [
    '0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67', // Swap
    null,                           // sender (any)
    '0x000000000000000000000000TARGET_ADDRESS',  // recipient (specific)
  ],
  fromBlock: '0x11A5B00',
  toBlock: '0x11A5B64',
};

// 2. 多地址查询：address 可以是数组
const filter2 = {
  address: [
    '0xPOOL_1',
    '0xPOOL_2',
    '0xPOOL_3',
  ],
  topics: [SWAP_TOPIC],
  fromBlock: '0x11A5B00',
  toBlock: '0x11A5B64',
};

// 3. 多 topic0 查询：topics[0] 可以是数组（OR 逻辑）
const filter3 = {
  address: '0xUNISWAP_V3_POOL',
  topics: [
    [SWAP_TOPIC, MINT_TOPIC, BURN_TOPIC],  // Swap OR Mint OR Burn
  ],
  fromBlock: '0x11A5B00',
  toBlock: '0x11A5B64',
};
```

### 3.2 Block Range 优化

```typescript
// 不同 RPC 提供商对 block range 有不同限制：
// - Alchemy: 最大 2000 blocks（免费）/ 无限制（付费）
// - Infura: 最大 10000 blocks
// - QuickNode: 最大 10000 blocks
// - 自建节点: 无限制

// 分批查询函数
async function getLogsInBatches(
  client: any,
  filter: any,
  batchSize = 2000
): Promise<any[]> {
  const fromBlock = Number(filter.fromBlock);
  const toBlock = Number(filter.toBlock);
  const allLogs: any[] = [];

  for (let start = fromBlock; start <= toBlock; start += batchSize) {
    const end = Math.min(start + batchSize - 1, toBlock);
    const logs = await client.getLogs({
      ...filter,
      fromBlock: BigInt(start),
      toBlock: BigInt(end),
    });
    allLogs.push(...logs);
    
    // Rate limiting
    if (end < toBlock) {
      await new Promise(r => setTimeout(r, 100));
    }
  }

  return allLogs;
}

// 自适应 batch size（处理 "query returned more than 10000 results" 错误）
async function getLogsAdaptive(
  client: any,
  filter: any,
  maxBatchSize = 10000
): Promise<any[]> {
  const fromBlock = Number(filter.fromBlock);
  const toBlock = Number(filter.toBlock);

  async function fetchRange(from: number, to: number, batchSize: number): Promise<any[]> {
    try {
      return await client.getLogs({
        ...filter,
        fromBlock: BigInt(from),
        toBlock: BigInt(to),
      });
    } catch (error: any) {
      if (error.message?.includes('10000 results') || error.message?.includes('query returned more')) {
        // 结果太多，减半 block range
        const mid = Math.floor((from + to) / 2);
        const first = await fetchRange(from, mid, batchSize);
        const second = await fetchRange(mid + 1, to, batchSize);
        return [...first, ...second];
      }
      throw error;
    }
  }

  return fetchRange(fromBlock, toBlock, maxBatchSize);
}
```

### 3.3 WebSocket vs HTTP Polling

```typescript
// 方式 1: WebSocket 实时订阅（推荐用于实时监控）
import { createPublicClient, webSocket, parseAbiItem } from 'viem';
import { mainnet } from 'viem/chains';

const wsClient = createPublicClient({
  chain: mainnet,
  transport: webSocket('wss://eth-mainnet.g.alchemy.com/v2/KEY'),
});

// 订阅新 logs
const unwatch = wsClient.watchEvent({
  address: '0xUNISWAP_V3_POOL' as `0x${string}`,
  event: parseAbiItem('event Swap(address indexed sender, address indexed recipient, int256 amount0, int256 amount1, uint160 sqrtPriceX96, uint128 liquidity, int24 tick)'),
  onLogs: (logs) => {
    for (const log of logs) {
      console.log('New swap:', {
        sender: log.args.sender,
        amount0: log.args.amount0,
        amount1: log.args.amount1,
        tick: log.args.tick,
      });
    }
  },
  onError: (error) => {
    console.error('Subscription error:', error);
    // 需要重连逻辑
  },
});

// 方式 2: HTTP Polling（简单但有延迟）
async function pollLogs(client: any, filter: any, intervalMs = 12000) {
  let lastBlock = await client.getBlockNumber();

  setInterval(async () => {
    const currentBlock = await client.getBlockNumber();
    if (currentBlock <= lastBlock) return;

    const logs = await client.getLogs({
      ...filter,
      fromBlock: lastBlock + 1n,
      toBlock: currentBlock,
    });

    for (const log of logs) {
      processLog(log);
    }

    lastBlock = currentBlock;
  }, intervalMs);
}
```

**对比：**

| 特性 | WebSocket | HTTP Polling |
|------|-----------|-------------|
| 延迟 | ~100ms（新区块即推送） | intervalMs（通常 12s） |
| 可靠性 | 需处理断连重连 | 天然可靠 |
| 服务器支持 | 部分 RPC 不支持 | 所有 RPC 支持 |
| 资源消耗 | 维持长连接 | 每次请求新连接 |
| 适用场景 | 实时交易、MEV | 定期统计、监控 |

### 3.4 批量处理与索引架构

```typescript
// 高性能日志处理流水线
interface LogProcessor {
  name: string;
  filter: { address?: string; topics: string[] };
  handler: (log: any) => Promise<void>;
}

class LogPipeline {
  private processors: LogProcessor[] = [];
  private db: any; // PostgreSQL/ClickHouse 连接
  private lastProcessedBlock: bigint = 0n;

  addProcessor(processor: LogProcessor) {
    this.processors.push(processor);
  }

  async backfill(fromBlock: bigint, toBlock: bigint) {
    console.log(`Backfilling blocks ${fromBlock} to ${toBlock}`);
    const BATCH_SIZE = 2000n;

    for (let start = fromBlock; start <= toBlock; start += BATCH_SIZE) {
      const end = start + BATCH_SIZE - 1n > toBlock ? toBlock : start + BATCH_SIZE - 1n;

      // 合并所有 processor 的 filter，一次 RPC 调用获取所有需要的 logs
      const allTopics = [...new Set(this.processors.flatMap(p => p.filter.topics))];
      const allAddresses = this.processors
        .filter(p => p.filter.address)
        .map(p => p.filter.address!);

      const logs = await client.getLogs({
        address: allAddresses.length > 0 ? allAddresses as any : undefined,
        topics: [allTopics],
        fromBlock: start,
        toBlock: end,
      });

      // 分发 logs 到对应的 processor
      for (const log of logs) {
        for (const processor of this.processors) {
          if (this.matchesFilter(log, processor.filter)) {
            await processor.handler(log);
          }
        }
      }

      // 记录进度
      this.lastProcessedBlock = end;
      if (Number(end - start) % 10000 === 0) {
        console.log(`Processed up to block ${end}`);
      }
    }
  }

  private matchesFilter(log: any, filter: LogProcessor['filter']): boolean {
    if (filter.address && log.address.toLowerCase() !== filter.address.toLowerCase()) {
      return false;
    }
    return filter.topics.includes(log.topics[0]);
  }
}

// 使用示例
const pipeline = new LogPipeline();

pipeline.addProcessor({
  name: 'uniswap_v3_swaps',
  filter: {
    topics: ['0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67'],
  },
  handler: async (log) => {
    // 解码并存入数据库
    const decoded = decodeV3Swap(log);
    await db.insert('swaps', decoded);
  },
});

pipeline.addProcessor({
  name: 'erc20_transfers',
  filter: {
    topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'],
  },
  handler: async (log) => {
    const decoded = decodeTransfer(log);
    if (decoded.value > BigInt('1000000000000000000000')) { // > 1000 tokens
      await alertLargeTransfer(decoded);
    }
  },
});
```

---

## 四、实时处理管道

### 4.1 Event → 业务信号链

```
链上事件 → 解码 → 业务逻辑 → 信号输出

示例流程：
Uniswap V3 Swap Event
  → 解码 amount0, amount1, sqrtPriceX96
  → 计算实际价格
  → 与历史价格比较
  → if 偏离 > 2%: 发出价格异常信号
  → if 金额 > $100K: 发出大额交易信号
  → 写入时序数据库
  → 更新 Dashboard
```

### 4.2 大额 Swap 检测

```typescript
import { createPublicClient, webSocket, parseAbiItem, formatUnits } from 'viem';
import { mainnet } from 'viem/chains';

const WETH_USDC_POOL = '0x88e6A0c2dDD26FEEb64F039a2c41296FcB3f5640' as const; // 0.05% pool
const WETH_DECIMALS = 18;
const USDC_DECIMALS = 6;

const client = createPublicClient({
  chain: mainnet,
  transport: webSocket('wss://eth-mainnet.g.alchemy.com/v2/KEY'),
});

const swapEvent = parseAbiItem(
  'event Swap(address indexed sender, address indexed recipient, int256 amount0, int256 amount1, uint160 sqrtPriceX96, uint128 liquidity, int24 tick)'
);

// WETH/USDC pool: token0 = WETH, token1 = USDC
client.watchEvent({
  address: WETH_USDC_POOL,
  event: swapEvent,
  onLogs: (logs) => {
    for (const log of logs) {
      const { amount0, amount1, sqrtPriceX96, tick } = log.args;

      // 计算 swap 金额（USD）
      const usdcAmount = Math.abs(Number(formatUnits(amount1!, USDC_DECIMALS)));
      const ethAmount = Math.abs(Number(formatUnits(amount0!, WETH_DECIMALS)));

      // 计算价格
      const price = (Number(sqrtPriceX96!) / 2 ** 96) ** 2 * 10 ** (WETH_DECIMALS - USDC_DECIMALS);

      if (usdcAmount > 100_000) {
        console.log(`[LARGE SWAP] $${usdcAmount.toFixed(0)} USDC`);
        console.log(`  ETH: ${ethAmount.toFixed(4)}`);
        console.log(`  Price: $${price.toFixed(2)}`);
        console.log(`  Tick: ${tick}`);
        console.log(`  Tx: ${log.transactionHash}`);
        console.log(`  Block: ${log.blockNumber}`);
      }
    }
  },
});
```

### 4.3 实时价格计算

```typescript
// 从 Sync 事件（V2）计算价格
function priceFromSync(reserve0: bigint, reserve1: bigint, decimals0: number, decimals1: number): number {
  // price of token0 in terms of token1
  return (Number(reserve1) / Number(reserve0)) * (10 ** (decimals0 - decimals1));
}

// 从 sqrtPriceX96（V3）计算价格
function priceFromSqrtPriceX96(sqrtPriceX96: bigint, decimals0: number, decimals1: number): number {
  const price = (Number(sqrtPriceX96) / 2 ** 96) ** 2;
  return price * 10 ** (decimals0 - decimals1);
}

// 从 tick（V3）计算价格
function priceFromTick(tick: number, decimals0: number, decimals1: number): number {
  return 1.0001 ** tick * 10 ** (decimals0 - decimals1);
}
```

### 4.4 异常告警引擎

```typescript
interface AlertRule {
  name: string;
  condition: (event: DecodedEvent) => boolean;
  action: (event: DecodedEvent) => Promise<void>;
}

const rules: AlertRule[] = [
  {
    name: 'large_swap',
    condition: (event) => event.type === 'swap' && event.amountUsd > 500_000,
    action: async (event) => {
      await sendTelegramAlert(`Large swap: $${event.amountUsd.toFixed(0)} on ${event.pool}`);
    },
  },
  {
    name: 'price_impact',
    condition: (event) => {
      if (event.type !== 'swap') return false;
      const priceImpact = Math.abs(event.priceAfter - event.priceBefore) / event.priceBefore;
      return priceImpact > 0.02; // > 2% price impact
    },
    action: async (event) => {
      await sendDiscordAlert(`High price impact: ${(event.priceImpact * 100).toFixed(2)}%`);
    },
  },
  {
    name: 'liquidity_removal',
    condition: (event) => event.type === 'burn' && event.liquidityUsd > 1_000_000,
    action: async (event) => {
      await sendAlert(`Large liquidity removal: $${event.liquidityUsd.toFixed(0)}`);
    },
  },
];
```

---

## 五、多语言实现

### 5.1 TypeScript (ethers.js)

```typescript
import { ethers } from 'ethers';

const provider = new ethers.WebSocketProvider('wss://eth-mainnet.g.alchemy.com/v2/KEY');

// 解码 ERC20 Transfer
const TRANSFER_ABI = ['event Transfer(address indexed from, address indexed to, uint256 value)'];
const iface = new ethers.Interface(TRANSFER_ABI);

// 方式 1: 直接解码 raw log
function decodeTransferLog(log: ethers.Log) {
  const decoded = iface.parseLog({ topics: log.topics as string[], data: log.data });
  if (!decoded) return null;
  return {
    from: decoded.args[0] as string,
    to: decoded.args[1] as string,
    value: decoded.args[2] as bigint,
  };
}

// 方式 2: 使用 Contract filter
const token = new ethers.Contract(
  '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
  TRANSFER_ABI,
  provider
);

token.on('Transfer', (from: string, to: string, value: bigint, event: any) => {
  if (value > 1_000_000n * 10n ** 6n) { // > $1M
    console.log(`Large USDC transfer: ${ethers.formatUnits(value, 6)} from ${from} to ${to}`);
  }
});

// 方式 3: 历史 log 查询
async function getHistoricalTransfers(tokenAddress: string, fromBlock: number, toBlock: number) {
  const filter = {
    address: tokenAddress,
    topics: [ethers.id('Transfer(address,address,uint256)')],
    fromBlock,
    toBlock,
  };
  const logs = await provider.getLogs(filter);
  return logs.map(log => decodeTransferLog(log)).filter(Boolean);
}
```

### 5.2 TypeScript (viem)

```typescript
import { createPublicClient, http, parseAbiItem, decodeEventLog } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: http('https://eth-mainnet.g.alchemy.com/v2/KEY'),
});

// 方式 1: 使用 parseAbiItem（推荐）
const transferEvent = parseAbiItem(
  'event Transfer(address indexed from, address indexed to, uint256 value)'
);

const logs = await client.getLogs({
  address: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
  event: transferEvent,
  fromBlock: 18500000n,
  toBlock: 18500100n,
});

for (const log of logs) {
  console.log(`${log.args.from} → ${log.args.to}: ${log.args.value}`);
}

// 方式 2: 手动解码 raw log
const rawLogs = await client.getLogs({
  address: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48',
  topics: ['0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'],
  fromBlock: 18500000n,
  toBlock: 18500100n,
});

for (const log of rawLogs) {
  const decoded = decodeEventLog({
    abi: [{ type: 'event', name: 'Transfer', inputs: [
      { type: 'address', name: 'from', indexed: true },
      { type: 'address', name: 'to', indexed: true },
      { type: 'uint256', name: 'value', indexed: false },
    ]}],
    data: log.data,
    topics: log.topics,
  });
  console.log(decoded.args);
}
```

### 5.3 Rust (alloy)

```rust
use alloy::{
    primitives::{address, b256, Address, U256},
    providers::{Provider, ProviderBuilder, WsConnect},
    rpc::types::{Filter, Log},
    sol,
    sol_types::SolEvent,
};
use futures::StreamExt;

// 定义事件（使用 sol! 宏）
sol! {
    event Transfer(address indexed from, address indexed to, uint256 value);

    // Uniswap V3 Swap
    event Swap(
        address indexed sender,
        address indexed recipient,
        int256 amount0,
        int256 amount1,
        uint160 sqrtPriceX96,
        uint128 liquidity,
        int24 tick
    );
}

#[tokio::main]
async fn main() -> eyre::Result<()> {
    // HTTP 连接用于历史查询
    let provider = ProviderBuilder::new()
        .on_http("https://eth-mainnet.g.alchemy.com/v2/KEY".parse()?);

    // 查询历史 Transfer 事件
    let usdc = address!("A0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48");
    let filter = Filter::new()
        .address(usdc)
        .event_signature(Transfer::SIGNATURE_HASH)
        .from_block(18500000)
        .to_block(18500100);

    let logs: Vec<Log> = provider.get_logs(&filter).await?;

    for log in &logs {
        match Transfer::decode_log(&log.inner, true) {
            Ok(decoded) => {
                println!(
                    "Transfer: {} -> {} amount: {}",
                    decoded.from, decoded.to, decoded.value
                );
            }
            Err(e) => eprintln!("Decode error: {}", e),
        }
    }

    // WebSocket 实时订阅
    let ws = WsConnect::new("wss://eth-mainnet.g.alchemy.com/v2/KEY");
    let ws_provider = ProviderBuilder::new().on_ws(ws).await?;

    let sub_filter = Filter::new()
        .address(usdc)
        .event_signature(Transfer::SIGNATURE_HASH);

    let mut stream = ws_provider.subscribe_logs(&sub_filter).await?.into_stream();

    while let Some(log) = stream.next().await {
        if let Ok(transfer) = Transfer::decode_log(&log.inner, true) {
            let amount = transfer.value / U256::from(10u64.pow(6));
            if amount > U256::from(1_000_000u64) {
                println!("[ALERT] Large USDC transfer: ${}", amount);
            }
        }
    }

    Ok(())
}
```

### 5.4 Go (go-ethereum)

```go
package main

import (
	"context"
	"fmt"
	"log"
	"math/big"
	"strings"

	"github.com/ethereum/go-ethereum"
	"github.com/ethereum/go-ethereum/accounts/abi"
	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
)

var (
	// ERC20 Transfer event signature
	transferTopic = crypto.Keccak256Hash([]byte("Transfer(address,address,uint256)"))

	// Uniswap V3 Swap event signature
	v3SwapTopic = crypto.Keccak256Hash([]byte(
		"Swap(address,address,int256,int256,uint160,uint128,int24)",
	))
)

func main() {
	client, err := ethclient.Dial("wss://eth-mainnet.g.alchemy.com/v2/KEY")
	if err != nil {
		log.Fatal(err)
	}

	// 历史查询
	usdc := common.HexToAddress("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
	query := ethereum.FilterQuery{
		FromBlock: big.NewInt(18500000),
		ToBlock:   big.NewInt(18500100),
		Addresses: []common.Address{usdc},
		Topics:    [][]common.Hash{{transferTopic}},
	}

	logs, err := client.FilterLogs(context.Background(), query)
	if err != nil {
		log.Fatal(err)
	}

	// 解析 ABI
	transferABI, _ := abi.JSON(strings.NewReader(`[{
		"anonymous": false,
		"inputs": [
			{"indexed": true, "name": "from", "type": "address"},
			{"indexed": true, "name": "to", "type": "address"},
			{"indexed": false, "name": "value", "type": "uint256"}
		],
		"name": "Transfer",
		"type": "event"
	}]`))

	for _, vLog := range logs {
		// indexed 参数从 topics 中提取
		from := common.BytesToAddress(vLog.Topics[1].Bytes())
		to := common.BytesToAddress(vLog.Topics[2].Bytes())

		// non-indexed 参数从 data 中解码
		event := struct {
			Value *big.Int
		}{}
		err := transferABI.UnpackIntoInterface(&event, "Transfer", vLog.Data)
		if err != nil {
			continue
		}

		fmt.Printf("Transfer: %s -> %s, amount: %s\n", from.Hex(), to.Hex(), event.Value.String())
	}

	// 实时订阅
	subQuery := ethereum.FilterQuery{
		Addresses: []common.Address{usdc},
		Topics:    [][]common.Hash{{transferTopic}},
	}

	logCh := make(chan types.Log)
	sub, err := client.SubscribeFilterLogs(context.Background(), subQuery, logCh)
	if err != nil {
		log.Fatal(err)
	}

	for {
		select {
		case err := <-sub.Err():
			log.Fatal(err)
		case vLog := <-logCh:
			from := common.BytesToAddress(vLog.Topics[1].Bytes())
			to := common.BytesToAddress(vLog.Topics[2].Bytes())
			value := new(big.Int).SetBytes(vLog.Data)
			fmt.Printf("Real-time Transfer: %s -> %s, %s\n", from.Hex(), to.Hex(), value.String())
		}
	}
}
```

### 5.5 Python (web3.py)

```python
from web3 import Web3
from web3.middleware import ExtraDataToPOAMiddleware
import json

# HTTP 连接
w3 = Web3(Web3.HTTPProvider('https://eth-mainnet.g.alchemy.com/v2/KEY'))

# ERC20 Transfer topic0
TRANSFER_TOPIC = '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef'

# 查询历史 logs
usdc_address = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48'

logs = w3.eth.get_logs({
    'address': usdc_address,
    'topics': [TRANSFER_TOPIC],
    'fromBlock': 18500000,
    'toBlock': 18500100,
})

for log in logs:
    from_addr = '0x' + log['topics'][1].hex()[26:]  # 去掉左补零
    to_addr = '0x' + log['topics'][2].hex()[26:]
    value = int(log['data'].hex(), 16)
    amount_usdc = value / 1e6
    print(f"Transfer: {from_addr} -> {to_addr}: ${amount_usdc:,.2f}")

# Uniswap V3 Swap 解码
V3_SWAP_TOPIC = '0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67'

def decode_v3_swap(log):
    """解码 Uniswap V3 Swap 事件"""
    sender = '0x' + log['topics'][1].hex()[26:]
    recipient = '0x' + log['topics'][2].hex()[26:]

    # data 解码: int256, int256, uint160, uint128, int24
    data = log['data'].hex()
    amount0 = int(data[2:66], 16)
    if amount0 >= 2**255:
        amount0 -= 2**256  # int256 处理负数

    amount1 = int(data[66:130], 16)
    if amount1 >= 2**255:
        amount1 -= 2**256

    sqrt_price_x96 = int(data[130:194], 16)
    liquidity = int(data[194:258], 16)
    tick = int(data[258:322], 16)
    if tick >= 2**23:
        tick -= 2**24  # int24

    return {
        'sender': sender,
        'recipient': recipient,
        'amount0': amount0,
        'amount1': amount1,
        'sqrtPriceX96': sqrt_price_x96,
        'liquidity': liquidity,
        'tick': tick,
    }

# WebSocket 实时监听
def start_realtime_listener():
    w3_ws = Web3(Web3.WebSocketProvider('wss://eth-mainnet.g.alchemy.com/v2/KEY'))

    # 创建 filter
    event_filter = w3_ws.eth.filter({
        'address': usdc_address,
        'topics': [TRANSFER_TOPIC],
    })

    print("Listening for USDC transfers...")
    while True:
        for log in event_filter.get_new_entries():
            from_addr = '0x' + log['topics'][1].hex()[26:]
            to_addr = '0x' + log['topics'][2].hex()[26:]
            value = int(log['data'].hex(), 16) / 1e6
            if value > 1_000_000:
                print(f"[ALERT] Large USDC transfer: ${value:,.0f}")
                print(f"  From: {from_addr}")
                print(f"  To: {to_addr}")
                print(f"  Tx: {log['transactionHash'].hex()}")

        import time
        time.sleep(2)
```

---

## 六、性能优化要点

### 6.1 检索优化

| 优化策略 | 说明 | 效果 |
|----------|------|------|
| **缩小 block range** | 尽量用小范围查询 | 减少节点扫描量 |
| **指定 address** | 不要全链扫描 | 利用布隆过滤器索引 |
| **用 topics filter** | indexed 参数可过滤 | 减少返回数据量 |
| **并行请求** | 多个不重叠 range 并行 | 提高吞吐量 |
| **自适应 batch** | 结果太多时自动分割 | 避免超限错误 |
| **缓存已处理区块** | 记录 checkpoint | 避免重复处理 |

### 6.2 解码优化

```typescript
// 预计算 topic0 → 解码器映射（避免每次计算 keccak256）
const DECODERS = new Map<string, (log: any) => any>([
  ['0xddf252ad...', decodeERC20Transfer],
  ['0xc42079f9...', decodeV3Swap],
  ['0xd78ad95f...', decodeV2Swap],
]);

function processLog(log: any) {
  const decoder = DECODERS.get(log.topics[0]);
  if (decoder) {
    return decoder(log);
  }
  return null; // unknown event
}
```

### 6.3 存储优化

```
高频 event 数据存储方案对比：

PostgreSQL:
  - 适合：结构化查询、JOIN、聚合
  - 不适合：超高写入量（> 10K events/s）
  - 建议：分区表按 block_number/日期

ClickHouse:
  - 适合：分析型查询、大量聚合
  - 不适合：频繁更新、点查
  - 建议：MergeTree 引擎、按 block_number 排序

TimescaleDB:
  - 适合：时序分析、连续聚合
  - 建议：hypertable 按 block_time 分区

Redis:
  - 适合：实时状态缓存（最新价格等）
  - 不适合：历史数据
```

### 6.4 Reorg 处理

```typescript
// 处理区块链重组（reorg）
// 当 reorg 发生时，之前处理的 logs 可能被 "removed"

// 方式 1: WebSocket 订阅会自动推送 removed=true 的 logs
client.watchEvent({
  onLogs: (logs) => {
    for (const log of logs) {
      if (log.removed) {
        // 这条 log 因 reorg 被移除，需要回滚对应的数据库操作
        console.log(`[REORG] Log removed: ${log.transactionHash}`);
        rollbackLog(log);
      } else {
        processLog(log);
      }
    }
  },
});

// 方式 2: 确认延迟（等待 N 个区块确认后再处理）
const CONFIRMATION_BLOCKS = 12; // 12 blocks ≈ 2.4 分钟
const safeBlock = (await client.getBlockNumber()) - BigInt(CONFIRMATION_BLOCKS);
// 只处理 safeBlock 之前的 logs
```

---

## 关联技能

- [mempool-monitoring](../mempool-monitoring/SKILL.md) — Mempool 监控，事件发生前的信号源
- [onchain-forensics](../onchain-forensics/SKILL.md) — 链上取证分析，基于 event log 的深度调查
- [dune-analytics-mev](../dune-analytics-mev/SKILL.md) — Dune SQL 分析，event log 的批量分析工具
