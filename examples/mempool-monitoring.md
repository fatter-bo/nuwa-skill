---
name: mempool-monitoring
description: |
  Mempool监控与交易解析专家。从pending交易中提取价值信号的完整知识体系：
  以太坊主网mempool机制与监听方法、私有mempool生态、pending交易解析
  （ABI decode、Uniswap swap解析、借贷协议交易识别）、L2"mempool等价物"
  （OP Stack sequencer feed、Arbitrum Sequencer Inbox）、高性能监听架构
  （Go/Rust实现、WebSocket管理、实时数据管道）、价值提取信号
  （可frontrun交易特征、巨鲸追踪、gas预测）。
  当用户说「mempool监控」「pending交易」「怎么监听交易」「巨鲸追踪」「MEV机会发现」时使用。
---

# Mempool监控与交易解析 · MEV信号提取系统

> Mempool是MEV的战场入口。谁先看到pending交易、谁先理解交易意图，谁就在MEV竞争中占据先手。

## 使用说明

**这是技术工具Skill，不是角色扮演。** 用户询问mempool监控、交易解析、MEV信号提取时，按工作流提供技术方案和代码示例。

---

## 执行工作流（Agentic Protocol）

```
Step 1: 确认目标链和场景
  → 链：Ethereum Mainnet / Arbitrum / Base / Optimism / 其他？
  → 目标：通用监控 / 特定DEX套利 / 巨鲸追踪 / 清算机器人？
  → 技术栈偏好：Go / Rust / TypeScript？

Step 2: 数据源选型
  → 自建节点 vs 第三方服务
  → 公共mempool vs 私有mempool
  → WebSocket vs P2P直连

Step 3: 交易解析方案
  → ABI decode策略
  → 目标协议识别和解析器
  → 价值信号提取逻辑

Step 4: 架构设计
  → 数据管道设计
  → 延迟优化策略
  → 持久化和回放
```

---

## 一、以太坊Mempool机制详解

### 什么是Mempool

Mempool（Memory Pool）是节点本地维护的**未被打包交易的缓冲池**。严格来说，不存在一个"全局mempool"——每个节点都有自己的本地txpool，通过P2P gossip协议传播交易。

### Mempool生命周期

```
用户发送交易
    │
    ▼
RPC节点接收 → eth_sendRawTransaction
    │
    ▼
本地txpool验证
  ├── nonce检查（是否连续）
  ├── balance检查（是否足够支付gas）
  ├── gas price检查（是否≥baseFee）
  └── txpool容量检查
    │
    ▼
P2P Gossip广播
  └── 发送给connected peers
      └── peers再转发给他们的peers
          └── 约2-4秒扩散到大部分网络
    │
    ▼
Builder/Validator收集交易 → 构建区块
    │
    ▼
区块上链 → 交易从mempool移除
```

### 关键时间窗口

| 阶段 | 延迟 | 说明 |
|------|------|------|
| 交易提交到本节点收到 | 0ms（自建节点直连） | 最快路径 |
| 交易在网络中传播 | 200ms - 2s | 取决于peer数量和网络拓扑 |
| 交易到达大部分builder | 500ms - 3s | builder通常有优化的P2P连接 |
| 区块构建周期 | 12s（1个slot） | 以太坊PoS固定slot时间 |
| 从看到交易到必须提交bundle | ~4-8s | MEV竞争的有效窗口 |

### Geth TxPool架构

```
TxPool
├── pending（已验证，可立即打包）
│   ├── 按sender地址分组
│   └── 每个sender按nonce排序
│
├── queued（nonce不连续，等待前置交易）
│   └── 按sender地址分组
│
└── 配置参数
    ├── globalslots: 最大pending slot数（默认5120）
    ├── globalqueue: 最大queued slot数（默认1024）
    ├── accountslots: 每账户最大pending数（默认16）
    └── accountqueue: 每账户最大queued数（默认64）
```

**MEV优化**：增大txpool容量以看到更多交易

```bash
geth --txpool.globalslots=20000 --txpool.globalqueue=5000 \
     --txpool.accountslots=256 --txpool.accountqueue=128
```

---

## 二、Mempool监听方法

### 方法1：WebSocket订阅 newPendingTransactions

**最常用方法，适用于大部分场景。**

#### TypeScript实现

```typescript
import { WebSocket } from 'ws';
import { ethers } from 'ethers';

// 方法A：只获取交易hash（低带宽）
async function subscribeNewPendingTxHashes() {
  const ws = new WebSocket('ws://localhost:8546');

  ws.on('open', () => {
    ws.send(JSON.stringify({
      jsonrpc: '2.0',
      id: 1,
      method: 'eth_subscribe',
      params: ['newPendingTransactions']
    }));
  });

  ws.on('message', async (data: string) => {
    const msg = JSON.parse(data);
    if (msg.method === 'eth_subscription') {
      const txHash = msg.params.result;
      // 需要额外 eth_getTransactionByHash 获取完整交易
      console.log('New pending tx:', txHash);
    }
  });
}

// 方法B：获取完整交易对象（Geth扩展，高带宽）
async function subscribeFullPendingTx() {
  const ws = new WebSocket('ws://localhost:8546');

  ws.on('open', () => {
    ws.send(JSON.stringify({
      jsonrpc: '2.0',
      id: 1,
      method: 'eth_subscribe',
      params: ['newPendingTransactions', { fullTx: true }]  // Geth特有参数
    }));
  });

  ws.on('message', async (data: string) => {
    const msg = JSON.parse(data);
    if (msg.method === 'eth_subscription') {
      const tx = msg.params.result;
      // tx包含完整交易数据：from, to, value, input, gasPrice等
      processPendingTx(tx);
    }
  });
}

// 方法C：使用ethers.js Provider
async function subscribeWithEthers() {
  const provider = new ethers.WebSocketProvider('ws://localhost:8546');

  provider.on('pending', async (txHash: string) => {
    try {
      const tx = await provider.getTransaction(txHash);
      if (tx) {
        processPendingTx(tx);
      }
    } catch (e) {
      // 交易可能已被打包或丢弃
    }
  });
}
```

#### Go实现

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/big"

    "github.com/ethereum/go-ethereum/core/types"
    "github.com/ethereum/go-ethereum/ethclient"
    "github.com/ethereum/go-ethereum/ethclient/gethclient"
    "github.com/ethereum/go-ethereum/rpc"
)

func subscribePendingTx() {
    // 建立WebSocket连接
    rpcClient, err := rpc.Dial("ws://localhost:8546")
    if err != nil {
        log.Fatal(err)
    }
    defer rpcClient.Close()

    gethClient := gethclient.New(rpcClient)
    ethClient := ethclient.NewClient(rpcClient)

    // 方法A: 订阅完整pending交易（推荐）
    txCh := make(chan *types.Transaction, 1000)
    sub, err := gethClient.SubscribeFullPendingTransactions(context.Background(), txCh)
    if err != nil {
        log.Fatal(err)
    }

    chainID, _ := ethClient.ChainID(context.Background())
    signer := types.LatestSignerForChainID(chainID)

    for {
        select {
        case tx := <-txCh:
            from, _ := types.Sender(signer, tx)
            fmt.Printf("Pending TX: hash=%s from=%s to=%s value=%s gas=%d\n",
                tx.Hash().Hex(),
                from.Hex(),
                tx.To().Hex(),
                tx.Value().String(),
                tx.Gas(),
            )
            processTx(tx)

        case err := <-sub.Err():
            log.Fatal("Subscription error:", err)
        }
    }
}
```

### 方法2：P2P直连（最低延迟）

直接连接以太坊P2P网络层，绕过RPC层，获得最早的交易通知。

```go
// 使用devp2p直接监听交易广播
// 这需要实现eth/68协议的Transaction message handler

import (
    "github.com/ethereum/go-ethereum/p2p"
    "github.com/ethereum/go-ethereum/eth/protocols/eth"
)

// 简化示例 - 实际需要完整的P2P协议栈
func handleTransactionMsg(peer *eth.Peer, txs []*types.Transaction) error {
    for _, tx := range txs {
        // 比WebSocket早 50-200ms 收到交易
        processEarlyTx(tx)
    }
    return nil
}
```

### 方法3：bloXroute BDN（商业服务）

```typescript
// bloXroute Gateway WebSocket订阅
const ws = new WebSocket('wss://eth.blxrbdn.com/ws', {
  headers: {
    'Authorization': 'YOUR_BLXR_AUTH_TOKEN'
  }
});

ws.on('open', () => {
  // 订阅所有pending交易
  ws.send(JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'subscribe',
    params: ['newTxs', {
      include: ['tx_hash', 'tx_contents.input', 'tx_contents.to',
                'tx_contents.value', 'tx_contents.gas_price',
                'tx_contents.max_fee_per_gas', 'tx_contents.max_priority_fee_per_gas'],
      filters: `{to} == '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D'`  // 只看Uniswap V2 Router
    }]
  }));
});
```

### 方法4：Fiber Network

```go
// Fiber提供亚毫秒级交易流
import fiber "github.com/chainbound/fiber-go"

func connectFiber() {
    client := fiber.NewClient("fiber-endpoint:8080", "YOUR_API_KEY")
    defer client.Close()

    ctx := context.Background()
    ch := make(chan *fiber.Transaction)

    // 订阅交易流
    err := client.SubscribeNewTxs(nil, ch)
    if err != nil {
        log.Fatal(err)
    }

    for tx := range ch {
        // Fiber通常比公共mempool快100-500ms
        processTransaction(tx)
    }
}
```

### 监听方法对比

| 方法 | 延迟 | 覆盖率 | 成本 | 实现复杂度 |
|------|------|--------|------|-----------|
| WebSocket（自建节点） | 中（200-500ms） | 依赖peer数 | 节点运维费 | 低 |
| P2P直连 | 低（100-300ms） | 依赖连接数 | 节点运维费 | 高 |
| bloXroute BDN | 极低（50-100ms） | 极高 | $500-5000/月 | 低 |
| Fiber Network | 极低（50-100ms） | 高 | $300-2000/月 | 低 |
| Flashbots Protect RPC | N/A | 仅自己发送的 | 免费 | 最低 |

---

## 三、私有Mempool生态

### 什么是私有Mempool

2021年Flashbots推出后，越来越多交易通过私有通道直接发送给builder，不经过公共mempool。

**2026年公共vs私有比例**：

| 交易类型 | 公共mempool | 私有/MEV-protected | 说明 |
|---------|------------|-------------------|------|
| 普通转账 | ~70% | ~30% | 大部分用户还用公共RPC |
| DEX swap | ~30% | ~70% | MEV保护普及 |
| 大额DEX swap | ~10% | ~90% | 几乎都走私有通道 |
| MEV bundle | 0% | 100% | 定义上就是私有的 |

### 主要私有交易通道

| 通道 | 类型 | 到达builder | 特点 |
|------|------|------------|------|
| Flashbots Protect | RPC替换 | Flashbots builder | 免费，抗frontrun |
| MEV Blocker | RPC替换 | 多builder | 免费，有rebate |
| bloXroute | 商业服务 | 所有主流builder | 付费，最快 |
| BuilderNet | 去中心化builder | BuilderNet成员 | 新兴 |
| 直接提交给builder | 自定义 | 指定builder | 需要关系 |

### 对MEV搜索者的影响

**关键洞察**：公共mempool中可见的"好机会"越来越少。策略需要调整：

1. **仍可从公共mempool获取的信号**：
   - 新代币的早期买入（散户不了解私有通道）
   - 小额swap（不值得走私有通道）
   - 非EVM链的桥接交易
   - 合约交互（approve, stake等）

2. **需要转向的策略**：
   - Backrun（不依赖看到对方交易，而是看到链上事件后反应）
   - 跨DEX/跨链套利（依赖链上状态而非mempool）
   - 清算机器人（监控链上健康因子）

---

## 四、Pending交易解析

### ABI Decode框架

```typescript
import { ethers } from 'ethers';

// 常用协议ABI片段
const UNISWAP_V2_ROUTER_ABI = [
  'function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] path, address to, uint deadline)',
  'function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] path, address to, uint deadline)',
  'function swapExactETHForTokens(uint amountOutMin, address[] path, address to, uint deadline) payable',
  'function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] path, address to, uint deadline)',
  'function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] path, address to, uint deadline)',
  'function swapETHForExactTokens(uint amountOut, address[] path, address to, uint deadline) payable',
];

const UNISWAP_V3_ROUTER_ABI = [
  'function exactInputSingle((address tokenIn, address tokenOut, uint24 fee, address recipient, uint256 amountIn, uint256 amountOutMinimum, uint160 sqrtPriceLimitX96))',
  'function exactInput((bytes path, address recipient, uint256 amountIn, uint256 amountOutMinimum))',
  'function exactOutputSingle((address tokenIn, address tokenOut, uint24 fee, address recipient, uint256 amountOut, uint256 amountInMaximum, uint160 sqrtPriceLimitX96))',
  'function multicall(uint256 deadline, bytes[] data)',
  'function multicall(bytes32 previousBlockhash, bytes[] data)',
];

const AAVE_V3_POOL_ABI = [
  'function supply(address asset, uint256 amount, address onBehalfOf, uint16 referralCode)',
  'function borrow(address asset, uint256 amount, uint256 interestRateMode, uint16 referralCode, address onBehalfOf)',
  'function repay(address asset, uint256 amount, uint256 interestRateMode, address onBehalfOf)',
  'function liquidationCall(address collateralAsset, address debtAsset, address user, uint256 debtToCover, bool receiveAToken)',
];

// 已知合约地址映射
const KNOWN_CONTRACTS: Record<string, { name: string; abi: string[] }> = {
  '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D': {
    name: 'Uniswap V2 Router',
    abi: UNISWAP_V2_ROUTER_ABI,
  },
  '0xE592427A0AEce92De3Edee1F18E0157C05861564': {
    name: 'Uniswap V3 Router',
    abi: UNISWAP_V3_ROUTER_ABI,
  },
  '0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45': {
    name: 'Uniswap Universal Router (V3)',
    abi: UNISWAP_V3_ROUTER_ABI,
  },
  '0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2': {
    name: 'Aave V3 Pool',
    abi: AAVE_V3_POOL_ABI,
  },
};

interface DecodedTx {
  protocol: string;
  method: string;
  args: Record<string, any>;
  value: bigint;
  gasPrice: bigint;
  from: string;
  to: string;
  hash: string;
}

function decodePendingTx(tx: any): DecodedTx | null {
  const toAddr = tx.to?.toLowerCase();
  if (!toAddr) return null; // 合约创建交易

  const contract = KNOWN_CONTRACTS[ethers.getAddress(toAddr)];
  if (!contract) return null;

  try {
    const iface = new ethers.Interface(contract.abi);
    const decoded = iface.parseTransaction({ data: tx.input || tx.data, value: tx.value });

    if (!decoded) return null;

    return {
      protocol: contract.name,
      method: decoded.name,
      args: Object.fromEntries(
        decoded.fragment.inputs.map((input, i) => [input.name, decoded.args[i]])
      ),
      value: BigInt(tx.value || '0'),
      gasPrice: BigInt(tx.gasPrice || tx.maxFeePerGas || '0'),
      from: tx.from,
      to: toAddr,
      hash: tx.hash,
    };
  } catch (e) {
    return null;
  }
}
```

### Uniswap Swap深度解析

```typescript
interface SwapInfo {
  direction: 'BUY' | 'SELL';
  tokenIn: string;
  tokenOut: string;
  amountIn: bigint;
  amountOutMin: bigint;
  slippageBps: number;  // 基点
  isLargeSwap: boolean; // >$10K
  deadline: number;
  recipient: string;
}

function parseUniswapV2Swap(decoded: DecodedTx): SwapInfo | null {
  const WETH = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2';

  if (decoded.method === 'swapExactTokensForTokens' || decoded.method === 'swapExactTokensForETH') {
    const path: string[] = decoded.args.path;
    const tokenIn = path[0];
    const tokenOut = path[path.length - 1];
    const isETHIn = tokenIn.toLowerCase() === WETH.toLowerCase();
    const isETHOut = tokenOut.toLowerCase() === WETH.toLowerCase();

    return {
      direction: isETHIn ? 'BUY' : (isETHOut ? 'SELL' : 'BUY'),
      tokenIn,
      tokenOut,
      amountIn: BigInt(decoded.args.amountIn),
      amountOutMin: BigInt(decoded.args.amountOutMin),
      slippageBps: calculateSlippage(decoded.args.amountIn, decoded.args.amountOutMin),
      isLargeSwap: false, // 需要价格oracle判断
      deadline: Number(decoded.args.deadline),
      recipient: decoded.args.to,
    };
  }

  if (decoded.method === 'swapExactETHForTokens') {
    const path: string[] = decoded.args.path;
    return {
      direction: 'BUY',
      tokenIn: WETH,
      tokenOut: path[path.length - 1],
      amountIn: decoded.value,
      amountOutMin: BigInt(decoded.args.amountOutMin),
      slippageBps: calculateSlippage(decoded.value, decoded.args.amountOutMin),
      isLargeSwap: decoded.value > ethers.parseEther('5'),
      deadline: Number(decoded.args.deadline),
      recipient: decoded.args.to,
    };
  }

  return null;
}

function calculateSlippage(amountIn: any, amountOutMin: any): number {
  // 简化：需要链上价格oracle来精确计算
  // 返回估算的滑点容忍度
  return 0; // placeholder
}
```

### Go语言交易解析

```go
package decoder

import (
    "math/big"
    "strings"

    "github.com/ethereum/go-ethereum/accounts/abi"
    "github.com/ethereum/go-ethereum/common"
    "github.com/ethereum/go-ethereum/core/types"
)

// 已知路由器地址
var (
    UniswapV2Router = common.HexToAddress("0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D")
    UniswapV3Router = common.HexToAddress("0xE592427A0AEce92De3Edee1F18E0157C05861564")
    SushiRouter     = common.HexToAddress("0xd9e1cE17f2641f24aE83637ab66a2cca9C378B9F")
)

type SwapInfo struct {
    Protocol    string
    Method      string
    TokenIn     common.Address
    TokenOut    common.Address
    AmountIn    *big.Int
    AmountOutMin *big.Int
    Recipient   common.Address
    IsLargeSwap bool
}

func DecodeSwap(tx *types.Transaction) (*SwapInfo, error) {
    if tx.To() == nil {
        return nil, nil // 合约创建
    }

    to := *tx.To()
    data := tx.Data()
    if len(data) < 4 {
        return nil, nil // 简单转账
    }

    // 获取function selector (前4字节)
    selector := data[:4]

    switch to {
    case UniswapV2Router:
        return decodeV2Swap(selector, data, tx.Value())
    case UniswapV3Router:
        return decodeV3Swap(selector, data, tx.Value())
    default:
        return nil, nil
    }
}

func decodeV2Swap(selector []byte, data []byte, value *big.Int) (*SwapInfo, error) {
    // swapExactETHForTokens: 0x7ff36ab5
    if selector[0] == 0x7f && selector[1] == 0xf3 && selector[2] == 0x6a && selector[3] == 0xb5 {
        parsedABI, _ := abi.JSON(strings.NewReader(`[{
            "name": "swapExactETHForTokens",
            "type": "function",
            "inputs": [
                {"name": "amountOutMin", "type": "uint256"},
                {"name": "path", "type": "address[]"},
                {"name": "to", "type": "address"},
                {"name": "deadline", "type": "uint256"}
            ]
        }]`))

        args, err := parsedABI.Methods["swapExactETHForTokens"].Inputs.Unpack(data[4:])
        if err != nil {
            return nil, err
        }

        path := args[1].([]common.Address)

        return &SwapInfo{
            Protocol:    "Uniswap V2",
            Method:      "swapExactETHForTokens",
            TokenIn:     path[0],
            TokenOut:    path[len(path)-1],
            AmountIn:    value,
            AmountOutMin: args[0].(*big.Int),
            Recipient:   args[2].(common.Address),
            IsLargeSwap: value.Cmp(new(big.Int).Mul(big.NewInt(5), new(big.Int).Exp(big.NewInt(10), big.NewInt(18), nil))) > 0,
        }, nil
    }

    return nil, nil
}

func decodeV3Swap(selector []byte, data []byte, value *big.Int) (*SwapInfo, error) {
    // multicall解码需要递归解析内部调用
    // 这里简化处理
    return nil, nil
}
```

---

## 五、L2 "Mempool等价物"

### OP Stack（Optimism/Base）

OP Stack L2没有传统mempool。交易流程：

```
用户发送交易 → Sequencer接收 → 立即排序 → 出unsafe block（~2s）
                                          → 批量提交到L1（每隔几分钟）
                                          → L1 finalize（~15分钟）
```

**监听方法**：

```typescript
// 方法1：订阅op-node的unsafe block（P2P gossip）
// op-node启动时配置P2P
// --p2p.listen.tcp=9003 --p2p.listen.udp=9003

// 方法2：WebSocket订阅新区块（每~2s一个unsafe block）
const provider = new ethers.WebSocketProvider('ws://YOUR_OP_NODE:8546');
provider.on('block', async (blockNumber: number) => {
  const block = await provider.getBlock(blockNumber, true);
  if (block) {
    for (const tx of block.transactions) {
      // 这些交易刚被sequencer排序，在L1确认前就可以看到
      processL2Tx(tx);
    }
  }
});
```

**延迟比较**：

| 数据源 | 延迟（相对sequencer出块） | 说明 |
|--------|------------------------|------|
| 自建op-node P2P | 50-200ms | 最快的公开方法 |
| 自建op-node RPC轮询 | 100-500ms | WebSocket订阅 |
| 公共RPC（Alchemy等） | 500ms-2s | 经过中间层 |
| 直连Sequencer | 0ms | 需要特殊接入 |

### Arbitrum

Arbitrum的数据更早可通过Sequencer Feed获取：

```typescript
// Arbitrum Sequencer Feed（WebSocket）
const ws = new WebSocket('wss://arb1.arbitrum.io/feed');

ws.on('message', (data: Buffer) => {
  // Sequencer Feed使用自定义二进制格式
  // 包含交易在被打包成区块前的排序信息
  const feedMsg = parseArbitrumFeed(data);
  if (feedMsg.transactions) {
    for (const tx of feedMsg.transactions) {
      processArbitrumTx(tx);
    }
  }
});
```

详细L2 feed解析请参考 **[l2-sequencer-feed](../l2-sequencer-feed/SKILL.md)** Skill。

---

## 六、高性能架构设计

### 整体架构

```
                          ┌─────────────────┐
                          │  多节点P2P监听   │
                          │  (Geth x3)       │
                          └────────┬─────────┘
                                   │
                          ┌────────▼─────────┐
                          │  去重 & 排序      │
                          │  (hash去重)       │
                          └────────┬─────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                   │
        ┌────────▼──────┐  ┌──────▼────────┐  ┌──────▼────────┐
        │  ABI Decoder   │  │  Gas分析器     │  │  地址标签器    │
        │  (协议识别)     │  │  (gas价格预测) │  │  (巨鲸识别)    │
        └────────┬──────┘  └──────┬────────┘  └──────┬────────┘
                 │                │                   │
                 └────────────────┼───────────────────┘
                                  │
                         ┌────────▼─────────┐
                         │  信号评估引擎     │
                         │  (MEV机会评分)    │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │               │
           ┌────────▼──────┐  ┌──▼──────┐  ┌────▼──────┐
           │  套利执行器    │  │  告警    │  │  持久化    │
           │  (bundle提交)  │  │  系统    │  │  (分析)    │
           └───────────────┘  └─────────┘  └───────────┘
```

### Go语言高性能实现

```go
package mempool

import (
    "context"
    "sync"

    "github.com/ethereum/go-ethereum/common"
    "github.com/ethereum/go-ethereum/core/types"
)

// TxProcessor 高性能交易处理管道
type TxProcessor struct {
    // 去重用的bloom filter或map
    seen     map[common.Hash]struct{}
    seenMu   sync.RWMutex

    // 处理管道channel
    rawTxCh     chan *types.Transaction  // 原始交易输入
    decodedCh   chan *DecodedTx          // 解码后的交易
    signalCh    chan *Signal             // 提取的信号

    // 处理器
    decoders    []TxDecoder
    evaluators  []SignalEvaluator
}

type Signal struct {
    Type        string  // "LARGE_SWAP", "LIQUIDATION", "WHALE_MOVE"
    Severity    float64 // 0-1
    TxHash      common.Hash
    Description string
    Metadata    map[string]interface{}
}

func NewTxProcessor(bufferSize int) *TxProcessor {
    return &TxProcessor{
        seen:     make(map[common.Hash]struct{}, 100000),
        rawTxCh:  make(chan *types.Transaction, bufferSize),
        decodedCh: make(chan *DecodedTx, bufferSize),
        signalCh: make(chan *Signal, bufferSize),
    }
}

func (p *TxProcessor) Start(ctx context.Context) {
    // 启动多个解码worker
    for i := 0; i < 8; i++ {
        go p.decodeWorker(ctx)
    }

    // 启动信号评估worker
    for i := 0; i < 4; i++ {
        go p.evaluateWorker(ctx)
    }

    // 启动信号消费
    go p.consumeSignals(ctx)
}

// IngestTx 接收新交易（去重）
func (p *TxProcessor) IngestTx(tx *types.Transaction) bool {
    hash := tx.Hash()

    p.seenMu.RLock()
    _, exists := p.seen[hash]
    p.seenMu.RUnlock()
    if exists {
        return false
    }

    p.seenMu.Lock()
    p.seen[hash] = struct{}{}
    p.seenMu.Unlock()

    select {
    case p.rawTxCh <- tx:
        return true
    default:
        // channel满了，丢弃（或者记录指标）
        return false
    }
}

func (p *TxProcessor) decodeWorker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        case tx := <-p.rawTxCh:
            for _, decoder := range p.decoders {
                if decoded, err := decoder.Decode(tx); err == nil && decoded != nil {
                    p.decodedCh <- decoded
                    break
                }
            }
        }
    }
}

func (p *TxProcessor) evaluateWorker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        case decoded := <-p.decodedCh:
            for _, evaluator := range p.evaluators {
                if signal := evaluator.Evaluate(decoded); signal != nil {
                    p.signalCh <- signal
                }
            }
        }
    }
}
```

### WebSocket连接管理（生产级）

```go
package wsmanager

import (
    "context"
    "log"
    "sync"
    "time"

    "github.com/gorilla/websocket"
)

// Manager 管理多个WebSocket连接的重连和健康检查
type Manager struct {
    endpoints []string
    conns     map[string]*websocket.Conn
    mu        sync.RWMutex
    msgCh     chan []byte
}

func NewManager(endpoints []string) *Manager {
    return &Manager{
        endpoints: endpoints,
        conns:     make(map[string]*wsocket.Conn),
        msgCh:     make(chan []byte, 10000),
    }
}

func (m *Manager) Start(ctx context.Context) {
    for _, ep := range m.endpoints {
        go m.maintainConnection(ctx, ep)
    }
}

func (m *Manager) maintainConnection(ctx context.Context, endpoint string) {
    var backoff time.Duration = time.Second

    for {
        select {
        case <-ctx.Done():
            return
        default:
        }

        conn, _, err := websocket.DefaultDialer.DialContext(ctx, endpoint, nil)
        if err != nil {
            log.Printf("Failed to connect to %s: %v, retrying in %v", endpoint, err, backoff)
            time.Sleep(backoff)
            if backoff < 30*time.Second {
                backoff *= 2
            }
            continue
        }

        backoff = time.Second // 重置退避
        log.Printf("Connected to %s", endpoint)

        m.mu.Lock()
        m.conns[endpoint] = conn
        m.mu.Unlock()

        // 发送订阅消息
        subscribeMsg := `{"jsonrpc":"2.0","id":1,"method":"eth_subscribe","params":["newPendingTransactions"]}`
        conn.WriteMessage(websocket.TextMessage, []byte(subscribeMsg))

        // 设置Pong handler
        conn.SetPongHandler(func(string) error {
            conn.SetReadDeadline(time.Now().Add(60 * time.Second))
            return nil
        })

        // 启动ping
        go func() {
            ticker := time.NewTicker(30 * time.Second)
            defer ticker.Stop()
            for {
                select {
                case <-ticker.C:
                    if err := conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                        return
                    }
                case <-ctx.Done():
                    return
                }
            }
        }()

        // 读取消息
        for {
            _, msg, err := conn.ReadMessage()
            if err != nil {
                log.Printf("Read error from %s: %v, reconnecting", endpoint, err)
                conn.Close()
                break
            }
            m.msgCh <- msg
        }
    }
}

func (m *Manager) Messages() <-chan []byte {
    return m.msgCh
}
```

---

## 七、价值信号提取

### 可Frontrun交易特征

| 特征 | 信号强度 | 说明 |
|------|---------|------|
| 大额DEX swap（>$50K） | ★★★★★ | 高滑点容忍=sandwich机会 |
| 高滑点设置（>2%） | ★★★★★ | 明确的利润空间 |
| Uniswap V2 swap（非V3） | ★★★★ | V2更容易被sandwich |
| 新代币首次大买入 | ★★★★ | 可能的pump信号 |
| 借贷协议清算调用 | ★★★★ | 可backrun套利 |
| 大额approve | ★★★ | 后续swap的前置信号 |
| 跨链桥接 | ★★★ | 套利窗口 |
| NFT mint（热门项目） | ★★ | gas竞争信号 |

### 巨鲸追踪系统

```typescript
// 已知巨鲸/机构地址标签
const WHALE_LABELS: Record<string, string> = {
  '0x56Eddb7aa87536c09CCc2793473599fD21A8b17F': 'Binance Hot Wallet',
  '0x28C6c06298d514Db089934071355E5743bf21d60': 'Binance Hot Wallet 2',
  '0xDFd5293D8e347dFe59E90eFd55b2956a1343963d': 'Binance Cold Wallet',
  '0x47ac0Fb4F2D84898e4D9E7b4DaB3C24507a6D503': 'Binance Reserve',
  '0xF977814e90dA44bFA03b6295A0616a897441aceC': 'Binance 8',
  '0x8103683202aa8DA10536036EDef04CDd865045aF': 'Jump Trading',
  '0x9507c04B10486547584C37bCBd931B5a4Ec4041e': 'Wintermute',
  '0x0000006daea1723962647b7e189d311d757Fb793': 'Wintermute 2',
  '0xE8c19DB00287e3536075114B2576C70773E039Bd': 'Alameda Research',
  // ... 持续维护
};

interface WhaleAlert {
  address: string;
  label: string;
  action: 'DEPOSIT_TO_CEX' | 'WITHDRAW_FROM_CEX' | 'LARGE_SWAP' | 'LARGE_TRANSFER';
  token: string;
  amountUSD: number;
  txHash: string;
  timestamp: number;
}

function detectWhaleActivity(tx: any): WhaleAlert | null {
  const from = tx.from.toLowerCase();
  const to = tx.to?.toLowerCase();

  // 检查是否涉及已知地址
  const fromLabel = WHALE_LABELS[ethers.getAddress(from)];
  const toLabel = WHALE_LABELS[ethers.getAddress(to || '')];

  if (!fromLabel && !toLabel) return null;

  // CEX充值检测（可能的卖出信号）
  if (toLabel?.includes('Binance') || toLabel?.includes('Coinbase')) {
    return {
      address: from,
      label: fromLabel || 'Unknown',
      action: 'DEPOSIT_TO_CEX',
      token: 'ETH', // 需要进一步解析ERC20
      amountUSD: 0,  // 需要价格oracle
      txHash: tx.hash,
      timestamp: Date.now(),
    };
  }

  return null;
}
```

### Gas价格预测

```typescript
// 基于当前pending交易分布预测下一个区块的gas价格
interface GasPrediction {
  baseFee: bigint;        // 预测的下一区块baseFee
  lowPriority: bigint;    // 30%概率被打包的priority fee
  mediumPriority: bigint; // 70%概率被打包的priority fee
  highPriority: bigint;   // 95%概率被打包的priority fee
  urgentPriority: bigint; // 99%概率被打包（MEV用）
}

async function predictGas(provider: ethers.Provider): Promise<GasPrediction> {
  const [block, feeHistory] = await Promise.all([
    provider.getBlock('latest'),
    provider.send('eth_feeHistory', ['0x14', 'latest', [10, 30, 70, 95, 99]]),
  ]);

  if (!block) throw new Error('Cannot get latest block');

  // EIP-1559 baseFee计算
  // 下一区块baseFee = 当前baseFee * (1 + 1/8 * (gasUsed - target) / target)
  const gasUsedRatio = Number(block.gasUsed) / Number(block.gasLimit);
  const target = 0.5; // 50%利用率为目标
  const currentBaseFee = block.baseFeePerGas!;

  let nextBaseFee: bigint;
  if (gasUsedRatio > target) {
    const delta = currentBaseFee * BigInt(Math.floor((gasUsedRatio - target) * 1000 / target)) / 8000n;
    nextBaseFee = currentBaseFee + (delta > 0n ? delta : 1n);
  } else {
    const delta = currentBaseFee * BigInt(Math.floor((target - gasUsedRatio) * 1000 / target)) / 8000n;
    nextBaseFee = currentBaseFee - delta;
  }

  // 从feeHistory提取priority fee百分位
  const rewards = feeHistory.reward;
  const lastRewards = rewards[rewards.length - 1];

  return {
    baseFee: nextBaseFee,
    lowPriority: BigInt(lastRewards[0]),
    mediumPriority: BigInt(lastRewards[1]),
    highPriority: BigInt(lastRewards[2]),
    urgentPriority: BigInt(lastRewards[3]),
  };
}
```

---

## 八、决策模板

### Mempool监控方案选型决策树

```
场景是什么？
├── MEV搜索（需要最低延迟）
│   ├── 预算>$1000/月 → bloXroute BDN + Fiber + 自建节点3个
│   ├── 预算$300-1000/月 → Fiber + 自建节点2个
│   └── 预算<$300/月 → 自建节点2个（不同地理位置）
│
├── 巨鲸追踪/市场分析
│   ├── 实时性要求高 → 自建节点 + WebSocket
│   └── 延迟几秒可接受 → 第三方WebSocket API（Alchemy等）
│
├── 清算机器人
│   ├── 主要靠链上状态（健康因子）→ 不需要mempool
│   └── 需要抢先清算 → 自建节点 + Flashbots bundle
│
└── 安全监控/告警
    └── 延迟几秒可接受 → Alchemy/Infura WebSocket即可
```

### 检查清单：上线Mempool监控系统

```
□ 数据源已确认（自建节点/第三方/混合）
□ WebSocket连接有重连和健康检查
□ 交易去重逻辑已实现（hash map或bloom filter）
□ 目标协议ABI已加载（Uniswap/Aave/Curve等）
□ multicall递归解析已实现
□ 已知地址标签库已加载
□ Gas价格预测模块已部署
□ 信号评估规则已配置
□ 监控指标已接入（处理延迟、吞吐量、丢弃率）
□ 磁盘持久化/回放能力已验证
□ 内存占用已监控（seen map需要定期清理）
□ 多节点去重合并已测试
```

---

## 九、性能优化技巧

| 优化点 | 具体措施 | 效果 |
|--------|---------|------|
| 减少RPC往返 | 使用 `fullTx: true` 订阅替代hash+getTransaction | 延迟减少50-100ms |
| 并行解码 | 多goroutine/worker处理不同交易 | 吞吐量提升4-8x |
| 预编译ABI | 启动时解析ABI而非每次decode | 单交易解码<1us |
| selector快速匹配 | 用4字节selector map替代完整ABI匹配 | 过滤速度提升10x |
| 内存池复用 | sync.Pool复用解码结构体 | GC压力降低 |
| seen map清理 | 每10分钟清理已上链的交易hash | 避免内存泄漏 |
| 批量获取 | eth_getTransactionReceipt批量请求 | RPC调用数减少 |

---

## Skill联动

- **[node-optimization](../node-optimization/SKILL.md)**：mempool监控的前提是高性能节点，节点的txpool配置直接影响可见交易数量
- **[sandwich-attack-mechanics](../sandwich-attack-mechanics/SKILL.md)**：从mempool中发现的大额swap信号如何用于sandwich策略
- **[backrun-arbitrage](../backrun-arbitrage/SKILL.md)**：mempool交易触发的链上状态变化如何用于backrun套利

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
