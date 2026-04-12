---
name: mev-bundle-construction
description: |
  MEV Bundle构造专家。精通Flashbots Bundle和Builder API。
  涵盖Flashbots Bundle基础（数据结构、eth_sendBundle参数、eth_callBundle模拟、生命周期）、
  高级构造（多交易原子性、条件Bundle、revert保护、nonce管理、gas定价）、
  Builder API与多Builder提交（主要builder API对比、包含率、priority fee vs 直接支付）、
  MEV-Share Bundle（格式、hint机制、退款机制）、L2 Bundle提交（OP Stack、Arbitrum优化）、
  监控与调试（包含失败诊断、stats API、竞争者分析）。
  包含完整Bundle构造代码（TypeScript + ethers.js）和Builder API端点。
  当用户问「怎么构造Bundle」「Flashbots怎么用」「Builder API」时使用。
---

# MEV Bundle 构造指南

> Bundle是MEV策略的执行载体——掌握Bundle构造就是掌握MEV执行的核心技术。

## 使用说明

**本Skill是MEV Bundle构造的完整技术参考。** 当用户询问Flashbots Bundle、Builder API、交易排序、Bundle提交时，按对应章节提供指导。

---

## 一、Flashbots Bundle基础

### 1.1 什么是Bundle

```
Bundle定义：
一组交易的有序集合，作为原子性单元提交给Block Builder。
Bundle中的所有交易要么全部被包含在同一区块中（按指定顺序），
要么全部不被包含。

核心特性：
1. 原子性: 要么全部执行，要么全部不执行
2. 有序性: 交易按照Bundle中的顺序执行
3. 隐私性: Bundle不进入公共mempool
4. Revert保护: 可以指定某些交易revert时整个Bundle失败

生命周期：
Searcher构造Bundle → 提交给Builder → Builder打包进区块 → 
Proposer选择区块 → 区块上链
```

### 1.2 eth_sendBundle参数详解

```javascript
// eth_sendBundle RPC方法
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_sendBundle",
    "params": [
        {
            // 必填：签名的原始交易数组
            "txs": [
                "0x02f8...",   // 签名的交易1（原始字节）
                "0x02f8...",   // 签名的交易2
                // ...
            ],
            
            // 必填：目标区块号（hex）
            "blockNumber": "0x1234567",
            
            // 可选：最早执行时间（Unix timestamp）
            "minTimestamp": 0,
            
            // 可选：最晚执行时间（Unix timestamp）
            "maxTimestamp": 1234567890,
            
            // 可选：可以revert的交易hash列表
            "revertingTxHashes": [
                "0xabc..."
            ],
            
            // 可选：替换现有的Bundle（Flashbots v2）
            "replacementUuid": "550e8400-e29b-41d4-a716-446655440000",
            
            // 可选：构建者可以移除尾部交易（Flashbots v2）
            "droppingTxHashes": [
                "0xdef..."
            ]
        }
    ]
}

注意事项：
- txs中可以包含pending交易的hash（用于backrun/sandwich）
- blockNumber只针对特定区块，不命中则Bundle被丢弃
- 通常需要对连续多个区块号提交相同的Bundle
```

### 1.3 eth_callBundle模拟

```javascript
// 模拟Bundle执行结果（不上链）
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_callBundle",
    "params": [
        {
            "txs": [
                "0x02f8...",
                "0x02f8..."
            ],
            "blockNumber": "0x1234567",
            "stateBlockNumber": "latest",
            // 可选：模拟的时间戳
            "timestamp": 1234567890
        }
    ]
}

// 返回结果
{
    "results": [
        {
            "txHash": "0x...",
            "gasUsed": 150000,
            "value": "0x0",
            // 如果交易revert
            "revert": "0x08c379a0..."  // revert reason
        },
        {
            "txHash": "0x...",
            "gasUsed": 200000,
            "value": "0x0"
        }
    ],
    "totalGasUsed": 350000,
    "stateBlockNumber": 18000000,
    "bundleHash": "0x..."
}

用途：
1. 验证Bundle是否能成功执行
2. 计算精确的gas消耗
3. 确认利润是否符合预期
4. 检测交易是否会revert
```

### 1.4 Bundle签名认证

```javascript
// Flashbots要求Bundle请求使用私钥签名
// 签名用于身份识别（reputation）和防止spam

const flashbotsProvider = await FlashbotsBundleProvider.create(
    provider,           // ethers.js provider
    authSigner,         // 认证用的钱包（不是交易签名钱包）
    "https://relay.flashbots.net"  // Flashbots Relay URL
);

// 签名过程：
// 1. 计算请求body的keccak256 hash
// 2. 用authSigner对hash签名
// 3. 将签名放入请求header
// Header: X-Flashbots-Signature: <address>:<signature>

// 注意：authSigner不需要有ETH余额
// 它只用于身份认证，不参与交易签名
```

---

## 二、完整Bundle构造代码

### 2.1 基础设施搭建

```typescript
// bundle-bot.ts - 完整的Bundle构造和提交框架
import { ethers, Wallet, JsonRpcProvider } from "ethers";
import {
    FlashbotsBundleProvider,
    FlashbotsBundleResolution,
    FlashbotsBundleTransaction
} from "@flashbots/ethers-provider-bundle";

// 配置
const CONFIG = {
    RPC_URL: "https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY",
    FLASHBOTS_RELAY: "https://relay.flashbots.net",
    PRIVATE_KEY: process.env.PRIVATE_KEY!,
    AUTH_KEY: process.env.AUTH_KEY!,  // Flashbots认证密钥
    MIN_PROFIT_WEI: ethers.parseEther("0.01"),  // 最小利润
    MAX_BLOCKS_TO_TRY: 5,  // 尝试的区块数
};

class BundleBot {
    private provider: JsonRpcProvider;
    private wallet: Wallet;
    private authSigner: Wallet;
    private flashbotsProvider: FlashbotsBundleProvider | null = null;

    constructor() {
        this.provider = new JsonRpcProvider(CONFIG.RPC_URL);
        this.wallet = new Wallet(CONFIG.PRIVATE_KEY, this.provider);
        this.authSigner = new Wallet(CONFIG.AUTH_KEY);
    }

    async initialize() {
        this.flashbotsProvider = await FlashbotsBundleProvider.create(
            this.provider,
            this.authSigner,
            CONFIG.FLASHBOTS_RELAY
        );
        console.log("Flashbots provider initialized");
        console.log("Auth address:", this.authSigner.address);
    }

    // 构造Backrun套利Bundle
    async buildBackrunBundle(
        targetTxHash: string,
        arbitrageCalldata: string,
        arbitrageContract: string,
        gasLimit: number = 300000
    ): Promise<FlashbotsBundleTransaction[]> {
        const block = await this.provider.getBlock("latest");
        if (!block) throw new Error("Failed to get latest block");
        const baseFee = block.baseFeePerGas!;

        const bundle: FlashbotsBundleTransaction[] = [
            // 目标交易（通过hash引用pending tx）
            {
                signedTransaction: targetTxHash  // pending tx的hash
            },
            // 套利交易
            {
                signer: this.wallet,
                transaction: {
                    to: arbitrageContract,
                    data: arbitrageCalldata,
                    gasLimit: gasLimit,
                    maxFeePerGas: baseFee * 2n,
                    maxPriorityFeePerGas: ethers.parseGwei("3"),
                    type: 2,
                    chainId: 1
                }
            }
        ];

        return bundle;
    }

    // 构造三明治Bundle
    async buildSandwichBundle(
        frontrunCalldata: string,
        victimTxHash: string,
        backrunCalldata: string,
        contractAddress: string,
        frontrunGas: number = 250000,
        backrunGas: number = 200000
    ): Promise<FlashbotsBundleTransaction[]> {
        const block = await this.provider.getBlock("latest");
        if (!block) throw new Error("Failed to get latest block");
        const baseFee = block.baseFeePerGas!;
        const nonce = await this.wallet.getNonce();

        const bundle: FlashbotsBundleTransaction[] = [
            // Frontrun交易
            {
                signer: this.wallet,
                transaction: {
                    to: contractAddress,
                    data: frontrunCalldata,
                    gasLimit: frontrunGas,
                    maxFeePerGas: baseFee * 2n,
                    maxPriorityFeePerGas: ethers.parseGwei("5"),
                    nonce: nonce,
                    type: 2,
                    chainId: 1
                }
            },
            // 受害者交易
            {
                signedTransaction: victimTxHash
            },
            // Backrun交易
            {
                signer: this.wallet,
                transaction: {
                    to: contractAddress,
                    data: backrunCalldata,
                    gasLimit: backrunGas,
                    maxFeePerGas: baseFee * 2n,
                    maxPriorityFeePerGas: ethers.parseGwei("2"),
                    nonce: nonce + 1,
                    type: 2,
                    chainId: 1
                }
            }
        ];

        return bundle;
    }

    // 模拟Bundle
    async simulateBundle(
        bundle: FlashbotsBundleTransaction[],
        targetBlock: number
    ) {
        if (!this.flashbotsProvider) throw new Error("Not initialized");

        const signedBundle = await this.flashbotsProvider.signBundle(bundle);
        const simulation = await this.flashbotsProvider.simulate(
            signedBundle,
            targetBlock
        );

        if ("error" in simulation) {
            console.error("Simulation error:", simulation.error);
            return null;
        }

        console.log("Simulation results:");
        console.log("  Total gas used:", simulation.totalGasUsed);
        for (const result of simulation.results) {
            console.log(`  TX ${result.txHash}:`);
            console.log(`    Gas used: ${result.gasUsed}`);
            if ("revert" in result) {
                console.log(`    REVERTED: ${result.revert}`);
            }
        }

        return simulation;
    }

    // 提交Bundle到多个Builder
    async submitBundle(
        bundle: FlashbotsBundleTransaction[],
        targetBlock: number
    ) {
        if (!this.flashbotsProvider) throw new Error("Not initialized");

        const signedBundle = await this.flashbotsProvider.signBundle(bundle);

        // 提交到Flashbots
        const bundleSubmission = await this.flashbotsProvider.sendRawBundle(
            signedBundle,
            targetBlock
        );

        console.log("Bundle submitted:");
        console.log("  Bundle hash:", bundleSubmission.bundleHash);

        // 等待结果
        const resolution = await bundleSubmission.wait();
        switch (resolution) {
            case FlashbotsBundleResolution.BundleIncluded:
                console.log("Bundle included in block!");
                return true;
            case FlashbotsBundleResolution.BlockPassedWithoutInclusion:
                console.log("Block passed without inclusion");
                return false;
            case FlashbotsBundleResolution.AccountNonceTooHigh:
                console.log("Nonce too high");
                return false;
        }
    }

    // 主循环：对连续多个区块提交Bundle
    async submitForMultipleBlocks(
        bundle: FlashbotsBundleTransaction[],
        startBlock: number,
        numBlocks: number = CONFIG.MAX_BLOCKS_TO_TRY
    ) {
        const submissions = [];
        for (let i = 0; i < numBlocks; i++) {
            const targetBlock = startBlock + i;
            submissions.push(
                this.submitBundle(bundle, targetBlock)
            );
        }

        const results = await Promise.all(submissions);
        return results.some(r => r === true);
    }
}

// 使用示例
async function main() {
    const bot = new BundleBot();
    await bot.initialize();

    // 构造套利Bundle
    const bundle = await bot.buildBackrunBundle(
        "0x...",  // pending交易hash
        "0x...",  // 套利合约calldata
        "0x...",  // 套利合约地址
        300000    // gas limit
    );

    // 模拟
    const currentBlock = await bot["provider"].getBlockNumber();
    const simulation = await bot.simulateBundle(bundle, currentBlock + 1);

    if (simulation) {
        // 提交
        const included = await bot.submitForMultipleBlocks(
            bundle,
            currentBlock + 1,
            5
        );
        console.log("Bundle included:", included);
    }
}
```

### 2.2 Nonce管理

```typescript
// Nonce管理器 - 处理并发Bundle提交
class NonceManager {
    private currentNonce: number = -1;
    private pendingBundles: Map<number, string> = new Map();  // nonce → bundleHash

    async initialize(wallet: Wallet) {
        this.currentNonce = await wallet.getNonce();
    }

    getNextNonce(): number {
        return this.currentNonce++;
    }

    // 当Bundle被包含时确认nonce
    confirmNonce(nonce: number) {
        this.pendingBundles.delete(nonce);
    }

    // 当Bundle失败时回滚nonce
    rollbackNonce(nonce: number) {
        this.pendingBundles.delete(nonce);
        // 如果没有更高的pending nonce，可以重用
        if (this.pendingBundles.size === 0) {
            this.currentNonce = nonce;
        }
    }

    // 同步链上nonce
    async sync(wallet: Wallet) {
        this.currentNonce = await wallet.getNonce();
        this.pendingBundles.clear();
    }
}
```

---

## 三、Builder API与多Builder提交

### 3.1 主要Builder对比

| Builder | 市场份额(2025) | API端点 | 特点 |
|---------|---------------|---------|------|
| **Flashbots (MEV-Boost)** | ~20% | relay.flashbots.net | 最稳定，最广泛使用 |
| **Titan Builder** | ~25% | rpc.titanbuilder.xyz | 高包含率 |
| **Rsync Builder** | ~15% | rsync-builder.xyz | 快速响应 |
| **Beaver Build** | ~10% | rpc.beaverbuild.org | 专注MEV优化 |
| **Builder0x69** | ~8% | builder0x69.io | 独立builder |
| **Blocknative** | ~5% | api.blocknative.com | 集成mempool数据 |

### 3.2 多Builder并行提交

```typescript
// 同时向多个Builder提交Bundle以提高包含率

interface BuilderConfig {
    name: string;
    url: string;
    authRequired: boolean;
}

const BUILDERS: BuilderConfig[] = [
    { name: "Flashbots", url: "https://relay.flashbots.net", authRequired: true },
    { name: "Titan", url: "https://rpc.titanbuilder.xyz", authRequired: true },
    { name: "Rsync", url: "https://rsync-builder.xyz", authRequired: true },
    { name: "Beaver", url: "https://rpc.beaverbuild.org", authRequired: true },
    { name: "Builder0x69", url: "https://builder0x69.io", authRequired: true },
];

class MultiBuilderSubmitter {
    private providers: Map<string, FlashbotsBundleProvider> = new Map();

    async initialize(provider: JsonRpcProvider, authSigner: Wallet) {
        for (const builder of BUILDERS) {
            try {
                const fbProvider = await FlashbotsBundleProvider.create(
                    provider,
                    authSigner,
                    builder.url
                );
                this.providers.set(builder.name, fbProvider);
                console.log(`Connected to ${builder.name}`);
            } catch (e) {
                console.error(`Failed to connect to ${builder.name}:`, e);
            }
        }
    }

    async submitToAll(
        signedBundle: string[],
        targetBlock: number
    ): Promise<Map<string, any>> {
        const results = new Map();

        const submissions = Array.from(this.providers.entries()).map(
            async ([name, provider]) => {
                try {
                    const response = await provider.sendRawBundle(
                        signedBundle,
                        targetBlock
                    );
                    results.set(name, {
                        success: true,
                        bundleHash: response.bundleHash
                    });
                    console.log(`${name}: submitted, hash=${response.bundleHash}`);
                } catch (e) {
                    results.set(name, { success: false, error: e });
                    console.error(`${name}: submission failed:`, e);
                }
            }
        );

        await Promise.all(submissions);
        return results;
    }
}
```

### 3.3 Priority Fee vs 直接支付

```
向Builder支付的两种方式：

方式1: Priority Fee（交易内支付）
- 通过交易的 maxPriorityFeePerGas 支付
- Builder从交易执行中获取priority fee
- 简单直接，但不够灵活

方式2: 直接支付到coinbase（区块构建者地址）
- 在Bundle的最后一笔交易中，向block.coinbase转ETH
- 更灵活：可以根据实际利润动态调整支付金额
- 更高效：只在Bundle成功时才支付

// 直接支付到coinbase的交易
{
    to: "0x0000000000000000000000000000000000000000",  // 不重要
    value: 0,
    data: "0x",  // 空
    // 或者使用专门的合约：
    to: mevBotContract,
    data: encodeFunctionData({
        functionName: "payBuilder",
        args: []
    })
}

// 合约中的payBuilder函数
function payBuilder() external {
    uint256 profit = calculateProfit();
    uint256 builderPayment = profit * builderShare / 100;
    
    // 直接向区块构建者支付
    block.coinbase.transfer(builderPayment);
    
    // 剩余利润留在合约
}

最佳实践：
- 直接支付通常更受Builder欢迎（更高的包含概率）
- 支付比例: 利润的70-90%（竞争越激烈，支付比例越高）
- 使用动态比例: 根据竞争程度调整
```

---

## 四、MEV-Share Bundle

### 4.1 MEV-Share原理

```
MEV-Share（由Flashbots推出）改变了MEV的分配模式：

传统模式：
用户 → Mempool → Searcher提取100% MEV → Builder → Proposer

MEV-Share模式：
用户 → MEV-Share → 部分信息披露给Searcher → Searcher竞价 → 
MEV利润按比例返还给用户

关键机制：
1. Hints（信息披露）: 用户选择披露多少交易信息
2. Bundle格式: Searcher构造包含用户交易的Bundle
3. Refund（退款）: MEV利润的一部分自动退给用户

Hints类型：
- contract_address: 交易涉及的合约地址
- function_selector: 调用的函数选择器
- logs: 交易会产生的事件日志
- calldata: 完整的calldata（最大信息量）
- hash: 只有交易hash（最小信息量）
```

### 4.2 MEV-Share Bundle格式

```javascript
// MEV-Share Bundle提交格式
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "mev_sendBundle",
    "params": [
        {
            // 版本
            "version": "v0.1",
            
            // 包含条件
            "inclusion": {
                "block": "0x1234567",
                "maxBlock": "0x1234570"  // 可以指定区间
            },
            
            // Bundle体
            "body": [
                // 引用MEV-Share中的用户交易
                {
                    "hash": "0xabc..."  // 用户交易的hash
                },
                // Searcher的交易
                {
                    "tx": "0x02f8...",   // 签名的原始交易
                    "canRevert": false
                }
            ],
            
            // 有效性条件
            "validity": {
                "refund": [
                    {
                        // 退款配置：退给用户
                        "bodyIdx": 0,  // 退给body中的第0笔交易的sender
                        "percent": 90  // 退还90%的利润
                    }
                ],
                "refundConfig": [
                    {
                        "address": "0x...",  // 退款地址
                        "percent": 90
                    }
                ]
            }
        }
    ]
}
```

### 4.3 订阅MEV-Share事件流

```typescript
// 监听MEV-Share事件流，获取用户交易的hints
import { SSEClient } from "sse-client";

class MEVShareListener {
    private eventSource: SSEClient;

    constructor() {
        this.eventSource = new SSEClient(
            "https://mev-share.flashbots.net/api/v1/bundle/stream"
        );
    }

    listen(callback: (event: MEVShareEvent) => void) {
        this.eventSource.on("message", (msg: string) => {
            const event: MEVShareEvent = JSON.parse(msg);
            callback(event);
        });
    }
}

interface MEVShareEvent {
    hash: string;          // 交易hash
    logs?: LogEntry[];     // 事件日志hints
    txs?: TxHint[];        // 交易hints
}

interface TxHint {
    to?: string;           // 目标合约
    functionSelector?: string;  // 函数选择器
    callData?: string;     // 完整calldata
}

interface LogEntry {
    address: string;
    topics: string[];
    data: string;
}

// 使用示例
const listener = new MEVShareListener();
listener.listen((event) => {
    // 解析hints
    if (event.txs && event.txs[0]?.to === UNISWAP_ROUTER) {
        // 这是一笔Uniswap交易
        // 根据hints评估backrun机会
        evaluateBackrunOpportunity(event);
    }
});
```

---

## 五、高级Bundle技巧

### 5.1 条件Bundle（Conditional Bundles）

```
条件Bundle允许指定Bundle执行的前提条件：

用途：
1. 只在特定状态下执行
2. 依赖于其他交易的结果
3. 链上条件验证

// 在合约中实现条件检查
contract ConditionalExecutor {
    function executeIfProfitable(
        bytes calldata swapData,
        uint256 minProfit
    ) external {
        uint256 balanceBefore = IERC20(token).balanceOf(address(this));
        
        // 执行swap
        (bool success,) = router.call(swapData);
        require(success, "Swap failed");
        
        uint256 balanceAfter = IERC20(token).balanceOf(address(this));
        uint256 profit = balanceAfter - balanceBefore;
        
        // 条件检查：利润不够则revert整个交易（和Bundle）
        require(profit >= minProfit, "Insufficient profit");
        
        // 向Builder支付
        uint256 builderPayment = profit * 80 / 100;
        block.coinbase.transfer(builderPayment);
    }
}
```

### 5.2 Revert保护策略

```
revertingTxHashes的使用：

// Bundle中某些交易可以被允许revert而不影响整个Bundle
{
    "txs": [
        "0xAAA...",  // 交易A（不允许revert）
        "0xBBB...",  // 交易B（受害者交易，可能revert）
        "0xCCC..."   // 交易C（backrun，不允许revert）
    ],
    "revertingTxHashes": [
        "0xBBB..."   // 允许交易B revert
    ]
}

使用场景：
1. Backrun中受害者交易可能因为deadline过期而revert
   → 即使受害者交易revert，backrun仍然执行
   
2. 多笔套利交易组合，其中一部分可能失败
   → 允许部分失败，只要总体仍然盈利

注意：
- 允许revert的交易仍会消耗gas
- 需要确保即使某些交易revert，剩余交易仍然有利可图
```

### 5.3 Gas定价策略

```typescript
// 动态Gas定价策略
class GasPricer {
    // 基于利润的gas定价
    static calculateOptimalGas(
        expectedProfit: bigint,
        totalGasUsed: number,
        currentBaseFee: bigint,
        competitionLevel: "low" | "medium" | "high"
    ): { maxFeePerGas: bigint; maxPriorityFeePerGas: bigint } {
        
        // 竞争程度决定支付比例
        const profitSharePercent = {
            low: 60n,    // 低竞争：支付60%利润
            medium: 80n, // 中等竞争：支付80%利润
            high: 95n    // 高竞争：支付95%利润
        }[competitionLevel];

        const maxPayment = expectedProfit * profitSharePercent / 100n;
        
        // 转换为每gas单价
        const maxPriorityFeePerGas = maxPayment / BigInt(totalGasUsed);
        const maxFeePerGas = currentBaseFee * 2n + maxPriorityFeePerGas;

        return { maxFeePerGas, maxPriorityFeePerGas };
    }

    // 使用直接coinbase支付（更推荐）
    static calculateCoinbasePayment(
        actualProfit: bigint,
        competitionLevel: "low" | "medium" | "high"
    ): bigint {
        const sharePercent = {
            low: 70n,
            medium: 85n,
            high: 95n
        }[competitionLevel];

        return actualProfit * sharePercent / 100n;
    }
}
```

---

## 六、L2 Bundle提交

### 6.1 OP Stack (Optimism/Base) Bundle

```
OP Stack L2的Bundle支持现状：

特点：
- 中心化sequencer排序交易
- 没有原生的Bundle API
- 交易按priority fee排序

替代方案：
1. 高priority fee提交（模拟frontrun）
   - 设置极高的maxPriorityFeePerGas
   - 不保证原子性

2. Flashbots OP Stack扩展（开发中）
   - 部分builder开始支持L2 Bundle
   - 原子性取决于sequencer配合

3. 直接与sequencer通信
   - 非公开API
   - 需要与L2运营商合作

// Base上的高优先级交易提交
async function submitToBase(txData: any) {
    const baseFee = await baseProvider.getFeeData();
    
    const tx = {
        ...txData,
        maxPriorityFeePerGas: ethers.parseGwei("0.1"),  // Base上0.1 gwei已经很高
        maxFeePerGas: baseFee.maxFeePerGas! + ethers.parseGwei("0.1"),
        chainId: 8453,
    };
    
    return wallet.sendTransaction(tx);
}
```

### 6.2 Arbitrum优化

```
Arbitrum的交易排序特点：

FCFS模型：
- 交易按到达sequencer的时间排序
- 没有priority fee竞价
- 延迟是核心竞争因素

优化策略：
1. 直连sequencer
   - 使用Arbitrum sequencer的直连端点
   - 减少中间环节延迟

2. 地理位置
   - Arbitrum sequencer位于特定数据中心
   - 距离越近，延迟越低

3. 交易构造优化
   - 最小化交易数据大小
   - 预签名交易，减少签名延迟
   - 使用专用的提交通道

4. Arbitrum AnyTrust（Orbit链）
   - 部分Orbit链支持自定义排序
   - 可能有Bundle支持

// Arbitrum交易快速提交
async function fastSubmitToArbitrum(signedTx: string) {
    // 直接发送到sequencer
    const result = await fetch("https://arb1-sequencer.arbitrum.io/rpc", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            jsonrpc: "2.0",
            method: "eth_sendRawTransaction",
            params: [signedTx],
            id: 1
        })
    });
    return result.json();
}
```

---

## 七、监控与调试

### 7.1 Bundle包含失败诊断

```
Bundle未被包含的常见原因：

1. Nonce冲突
   - 原因：Bundle中的nonce与链上nonce不匹配
   - 诊断：检查链上最新nonce
   - 修复：同步nonce后重新提交

2. Gas不足
   - 原因：priority fee太低，被其他Bundle竞争掉
   - 诊断：检查同区块中其他Bundle的gas价格
   - 修复：提高priority fee或coinbase支付

3. 交易revert
   - 原因：Bundle中的交易执行失败
   - 诊断：使用eth_callBundle模拟
   - 修复：修复revert原因或添加到revertingTxHashes

4. 目标交易未出现
   - 原因：引用的pending tx已被打包或取消
   - 诊断：检查pending tx状态
   - 修复：重新获取新的目标交易

5. 时间窗口过期
   - 原因：Bundle指定的区块已经过去
   - 诊断：检查当前区块号
   - 修复：对未来区块重新提交

6. Builder未选中
   - 原因：提交的Builder没有赢得该区块
   - 诊断：检查哪个Builder构建了该区块
   - 修复：向更多Builder提交
```

### 7.2 Flashbots Stats API

```javascript
// 获取Bundle统计信息
const statsV2 = await flashbotsProvider.getBundleStatsV2(
    bundleHash,
    targetBlockNumber
);

console.log("Bundle stats:", {
    isSimulated: statsV2.isSimulated,
    isHighPriority: statsV2.isHighPriority,
    simulatedAt: statsV2.simulatedAt,
    receivedAt: statsV2.receivedAt,
    consideredByBuildersAt: statsV2.consideredByBuildersAt,
    sealedByBuildersAt: statsV2.sealedByBuildersAt
});

// 获取用户统计
const userStats = await flashbotsProvider.getUserStatsV2();
console.log("User stats:", {
    isHighPriority: userStats.isHighPriority,
    allTimeMinerPayments: userStats.allTimeMinerPayments,
    allTimeGasSimulated: userStats.allTimeGasSimulated,
    last7dMinerPayments: userStats.last7dMinerPayments,
    last7dGasSimulated: userStats.last7dGasSimulated,
    last1dMinerPayments: userStats.last1dMinerPayments,
    last1dGasSimulated: userStats.last1dGasSimulated
});
```

### 7.3 竞争者分析

```
分析竞争者的方法：

1. 区块内分析
   - 查看目标区块中被包含的Bundle
   - 分析竞争者的gas支付策略
   - 比较自己的Bundle和被包含的Bundle的差异

2. 链上Bot追踪
   - 通过Etherscan追踪已知MEV Bot合约
   - 分析其交易模式和利润
   - Arkham/Nansen标记的MEV实体

3. Builder数据分析
   - 使用MEV-Boost Relay Data API
   - 分析不同Builder的区块构建模式
   - 统计各Builder的平均交易数和MEV利润

// 获取relay数据
async function getRelayData(slot: number) {
    const response = await fetch(
        `https://boost-relay.flashbots.net/relay/v1/data/bidtraces/proposer_payload_delivered?slot=${slot}`
    );
    return response.json();
}

// 分析Builder竞争格局
async function analyzeBuilderMarketShare(numSlots: number) {
    const latestSlot = await getLatestSlot();
    const builderStats: Map<string, number> = new Map();

    for (let i = 0; i < numSlots; i++) {
        const data = await getRelayData(latestSlot - i);
        if (data && data.length > 0) {
            const builder = data[0].builder_pubkey;
            builderStats.set(builder, 
                (builderStats.get(builder) || 0) + 1
            );
        }
    }

    return builderStats;
}
```

---

## 八、常见陷阱与最佳实践

### 8.1 常见陷阱

| 陷阱 | 描述 | 解决方案 |
|------|------|---------|
| **Nonce不同步** | 链上nonce和本地nonce不一致 | 每次构造Bundle前同步nonce |
| **模拟与实际不一致** | 模拟时状态和实际执行时不同 | 使用最新区块状态模拟 |
| **Gas估算不准** | 实际gas消耗超过限制 | 添加20%的安全余量 |
| **单Builder依赖** | 只提交给Flashbots | 多Builder并行提交 |
| **忽略revert保护** | 未使用revertingTxHashes | 合理配置revert策略 |
| **硬编码gas价格** | 不跟随市场变化 | 动态计算gas价格 |
| **忽略EIP-4844** | blob交易影响baseFee | 考虑L1数据成本变化 |

### 8.2 最佳实践清单

```
Bundle构造最佳实践：

[x] 每次提交前使用eth_callBundle模拟
[x] 向至少3个Builder并行提交
[x] 使用动态gas定价而非固定值
[x] 实现自动nonce管理和同步
[x] 对连续多个区块号提交同一Bundle
[x] 使用直接coinbase支付而非priority fee
[x] 实现完整的日志和监控
[x] 设置利润下限，避免微利交易
[x] 处理所有可能的错误情况
[x] 定期分析包含率和利润率
[x] 监控竞争者的策略变化
[x] 保持authSigner的reputation
```

---

## 九、Builder API端点参考

### 9.1 标准端点

```
Flashbots Relay:
  - 主网: https://relay.flashbots.net
  - Goerli: https://relay-goerli.flashbots.net
  - Sepolia: https://relay-sepolia.flashbots.net

Titan Builder:
  - 主网: https://rpc.titanbuilder.xyz

Rsync Builder:
  - 主网: https://rsync-builder.xyz

Beaver Build:
  - 主网: https://rpc.beaverbuild.org

Builder0x69:
  - 主网: https://builder0x69.io

Blocknative:
  - 主网: https://api.blocknative.com/v1/bundle

MEV-Share:
  - 事件流: https://mev-share.flashbots.net/api/v1/bundle/stream
  - 提交: https://relay.flashbots.net (使用mev_sendBundle方法)

注意：端点地址可能更新，以各Builder官方文档为准。
```

---

## 十、工具速查表

| 用途 | 工具 | 说明 |
|------|------|------|
| Bundle SDK | @flashbots/ethers-provider-bundle | Flashbots官方TypeScript SDK |
| 交易模拟 | eth_callBundle / Tenderly | 模拟Bundle执行 |
| 区块分析 | MEV-Boost Relay Data | 分析Builder和区块数据 |
| Gas追踪 | Blocknative Gas Platform | 实时gas价格和预测 |
| Bot框架 | mev-flood / mev-inspect | 开源MEV工具 |
| 测试网 | Goerli/Sepolia | Bundle测试环境 |
| 链上数据 | Etherscan / Dune | 交易和合约分析 |

---

## 诚实边界

- **Bundle包含不保证**：即使模拟成功，实际包含取决于Builder竞价和Proposer选择
- **Builder市场快速变化**：Builder的市场份额和API随时可能变化
- **MEV-Share仍在演进**：协议细节和利润分配比例可能调整
- **L2 Bundle支持有限**：大多数L2目前没有原生Bundle API
- **代码示例是简化版**：生产环境需要更完善的错误处理、重试逻辑和监控
- **竞争极度激烈**：Bundle构造只是技术基础，实际盈利还需要策略、速度和资金优势

---

## 关联Skills

- [sandwich-attack-mechanics](../sandwich-attack-mechanics/) — 三明治攻击机制（使用Bundle执行）
- [backrun-arbitrage](../backrun-arbitrage/) — Backrun套利策略（Bundle提交）
- [low-latency-infrastructure](../low-latency-infrastructure/) — 低延迟基础设施

---

## 附录：调研来源

### 核心参考
- Flashbots文档: https://docs.flashbots.net/
- Flashbots Bundle SDK: https://github.com/flashbots/ethers-provider-flashbots-bundle
- MEV-Share文档: https://docs.flashbots.net/flashbots-mev-share/overview
- Builder API Spec: https://ethereum.github.io/builder-specs/
- MEV-Boost文档: https://boost.flashbots.net/
- Flashbots Forum: https://collective.flashbots.net/
- Ethereum Builder Specs: https://github.com/ethereum/builder-specs

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
