---
name: rpc-endpoint-strategy
description: |
  RPC端点策略专家。构建高可用、低延迟RPC访问层的完整知识体系：
  自建节点vs第三方服务商对比（Alchemy、Infura、QuickNode深度评测）、
  RPC接口优化（WebSocket vs HTTP、batch请求、multicall、eth_getLogs优化）、
  多RPC负载均衡（故障切换、延迟感知路由、健康检查）、
  MEV专用RPC（Flashbots eth_sendBundle、bloXroute、Builder API、多builder提交）、
  成本优化与预算规划。
  包含定价对比表、nginx/HAProxy配置示例、分档位推荐方案。
  当用户说「RPC怎么选」「Alchemy还是Infura」「RPC负载均衡」「bundle怎么提交」时使用。
---

# RPC端点策略 · 高可用低延迟访问层指南

> RPC是应用与区块链交互的门户。选错RPC，就像在高速公路上骑自行车——目的地一样，体验天壤之别。

## 使用说明

**这是技术工具Skill，不是角色扮演。** 用户询问RPC服务商选型、RPC层架构设计、MEV提交通道时，按工作流提供方案。

---

## 执行工作流（Agentic Protocol）

```
Step 1: 需求分析
  → 链：Ethereum / Arbitrum / Base / Polygon / 多链？
  → 用途：dApp后端 / MEV / 数据分析 / 交易提交？
  → 请求量：日均多少请求？峰值？
  → 延迟要求：毫秒级 / 百毫秒级 / 秒级？
  → 预算范围？

Step 2: 方案设计
  → 自建 vs 第三方 vs 混合
  → 单RPC vs 多RPC负载均衡
  → MEV场景的特殊RPC需求

Step 3: 配置方案
  → 具体服务商+套餐选择
  → 负载均衡配置
  → 监控与告警

Step 4: 成本优化
  → 请求合并策略
  → 缓存策略
  → 费用预测
```

---

## 一、自建节点 vs 第三方服务对比

### 总览对比

| 维度 | 自建节点 | 第三方（Alchemy等） |
|------|---------|-------------------|
| **延迟** | 最低（本地访问） | 中等（网络+处理） |
| **可靠性** | 取决于运维能力 | 高（SLA保证） |
| **成本（低量）** | 高（固定服务器费） | 低（免费tier） |
| **成本（高量）** | 低（边际成本趋零） | 高（按请求计费） |
| **灵活性** | 完全控制 | 受限于API提供的功能 |
| **维护负担** | 高（升级、同步、故障） | 低（全托管） |
| **mempool访问** | 完整txpool | 受限或不支持 |
| **archive数据** | 需额外配置 | 通常包含 |
| **适用场景** | MEV、高频、定制需求 | dApp、一般开发、快速启动 |

### 成本交叉点分析

```
月请求量 vs 月成本：

            $3,000 ┤
                   │                        ┌── Alchemy Growth
            $2,000 ┤                   ┌────┘
                   │              ┌────┘
            $1,000 ┤         ┌────┘
                   │    ┌────┘
              $500 ┤────┘
                   │ ────────────────────── 自建节点（Hetzner ~$200/月固定）
              $200 ┤
                   │
                $0 ┼──────┬──────┬──────┬──────┬───────
                   0    50M   100M   200M   500M  请求/月

交叉点：约30-50M请求/月时，自建节点开始比Alchemy便宜
```

---

## 二、第三方RPC服务商深度评测

### 主要服务商对比（2026年）

| 服务商 | 支持链数 | 免费tier | 延迟(ETH) | WebSocket | Archive | 特色 |
|--------|---------|---------|-----------|-----------|---------|------|
| **Alchemy** | 30+ | 300M CU/月 | 50-150ms | 是 | 是 | Trace API、Webhook、NFT API |
| **Infura** | 15+ | 100K req/日 | 60-200ms | 是（付费） | 付费 | MetaMask默认、DIN去中心化 |
| **QuickNode** | 60+ | 有 | 40-120ms | 是 | 是 | 全球端点、Marketplace插件 |
| **Chainstack** | 25+ | 3M req/月 | 50-150ms | 是 | 是 | 专用节点选项 |
| **Ankr** | 45+ | 30M req/月 | 80-200ms | 是 | 部分 | 去中心化RPC网络 |
| **dRPC** | 50+ | 有 | 40-120ms | 是 | 是 | 去中心化、geo-routing |
| **Blast** | 20+ | 40 req/s | 50-150ms | 是 | 是 | 免费tier慷慨 |
| **Tenderly** | 20+ | 有 | 50-150ms | 是 | 是 | 模拟/调试最强 |

### 定价对比（2026年Ethereum Mainnet）

#### Alchemy

| 套餐 | 月费 | Compute Units/月 | 约等于请求数 | 附加功能 |
|------|------|-----------------|------------|---------|
| Free | $0 | 300M CU | ~20M 简单请求 | 基础API |
| Growth | $49 | 400M CU + 弹性 | ~25M | Enhanced API、Webhook |
| Scale | $499 | 5B CU + 弹性 | ~300M | 专属支持、SLA |
| Enterprise | 定制 | 无限 | 无限 | 专用基础设施 |

**CU计算规则**（1个简单请求 ≈ 15 CU）：

| 方法 | CU消耗 |
|------|--------|
| eth_blockNumber | 10 |
| eth_getBalance | 15 |
| eth_call | 26 |
| eth_getTransactionReceipt | 15 |
| eth_getLogs（100 logs） | 75 |
| eth_getLogs（10000 logs） | 750 |
| debug_traceTransaction | 309 |
| trace_transaction | 75 |
| alchemy_getTokenBalances | 79 |

#### Infura

| 套餐 | 月费 | 请求/日 | 请求/秒 | 附加功能 |
|------|------|--------|--------|---------|
| Core (Free) | $0 | 100K | 10 | 基础API |
| Developer | $50 | 200K | 25 | Archive、更多API |
| Team | $225 | 1M | 50 | WebSocket、Priority支持 |
| Growth | $1,000 | 5M | 200 | SLA、专用 |

#### QuickNode

| 套餐 | 月费 | 请求/月 | 说明 |
|------|------|--------|------|
| Free | $0 | 限制 | 1个端点 |
| Starter | $49 | 15M API Credits | 多链支持 |
| Growth | $299 | 100M API Credits | 专用端点 |
| Business | $799 | 300M+ API Credits | 高性能+SLA |

### 服务商选型建议

| 场景 | 推荐 | 理由 |
|------|------|------|
| dApp开发 | Alchemy Growth | 最丰富的增值API（NFT、Token等） |
| MetaMask集成 | Infura | 默认提供商，兼容性最好 |
| 多链项目 | QuickNode / Ankr | 链支持最广 |
| 调试/模拟 | Tenderly | Trace/模拟功能最强 |
| 成本敏感 | Ankr / Blast / dRPC | 免费tier慷慨 |
| 去中心化偏好 | dRPC / Ankr | 分布式节点网络 |
| MEV | 自建节点 | 第三方不支持或受限 |

---

## 三、RPC接口优化

### WebSocket vs HTTP

| 维度 | WebSocket | HTTP |
|------|-----------|------|
| 连接模型 | 长连接 | 短连接 |
| 延迟 | 低（无需重复握手） | 中（每次TCP+TLS） |
| 订阅能力 | 支持`eth_subscribe` | 不支持（只能轮询） |
| 带宽效率 | 高（无HTTP头开销） | 低 |
| 连接管理 | 需要心跳+重连 | 简单 |
| 负载均衡 | 需要支持WS的LB | 标准HTTP LB |
| 适用场景 | 实时数据、事件监听 | 查询、交易提交 |

**建议**：
- 需要实时数据（新区块、新交易、日志事件） → WebSocket
- 只需要查询或提交交易 → HTTP（更简单可靠）
- MEV场景 → WebSocket（监听）+ HTTP（bundle提交）

### Batch请求优化

将多个RPC请求合并成一个HTTP请求：

```typescript
// 单独请求（3次往返，延迟 = 3 * RTT）
const balance = await provider.getBalance(addr);
const nonce = await provider.getTransactionCount(addr);
const code = await provider.getCode(addr);

// Batch请求（1次往返，延迟 = 1 * RTT）
const batchPayload = [
  { jsonrpc: '2.0', id: 1, method: 'eth_getBalance', params: [addr, 'latest'] },
  { jsonrpc: '2.0', id: 2, method: 'eth_getTransactionCount', params: [addr, 'latest'] },
  { jsonrpc: '2.0', id: 3, method: 'eth_getCode', params: [addr, 'latest'] },
];

const response = await fetch(RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(batchPayload),
});

const results = await response.json();
// results[0].result = balance
// results[1].result = nonce
// results[2].result = code
```

**Go语言batch实现**：

```go
package rpc

import (
    "context"
    "github.com/ethereum/go-ethereum/rpc"
)

func BatchCall(client *rpc.Client, ctx context.Context) error {
    var (
        balance   string
        nonce     string
        blockNum  string
    )

    batch := []rpc.BatchElem{
        {Method: "eth_getBalance", Args: []interface{}{"0x...", "latest"}, Result: &balance},
        {Method: "eth_getTransactionCount", Args: []interface{}{"0x...", "latest"}, Result: &nonce},
        {Method: "eth_blockNumber", Args: []interface{}{}, Result: &blockNum},
    }

    err := client.BatchCallContext(ctx, batch)
    if err != nil {
        return err
    }

    // 检查每个请求的独立错误
    for _, elem := range batch {
        if elem.Error != nil {
            // 处理单个请求错误
        }
    }

    return nil
}
```

### Multicall合约优化

将多个`eth_call`合并成对Multicall合约的单次调用：

```typescript
import { ethers } from 'ethers';

const MULTICALL3_ADDRESS = '0xcA11bde05977b3631167028862bE2a173976CA11'; // 通用地址，几乎所有链

const MULTICALL3_ABI = [
  'function aggregate3(tuple(address target, bool allowFailure, bytes callData)[] calls) returns (tuple(bool success, bytes returnData)[])',
  'function aggregate3Value(tuple(address target, bool allowFailure, uint256 value, bytes callData)[] calls) payable returns (tuple(bool success, bytes returnData)[])',
];

async function multicallExample(provider: ethers.Provider) {
  const multicall = new ethers.Contract(MULTICALL3_ADDRESS, MULTICALL3_ABI, provider);

  const erc20Interface = new ethers.Interface([
    'function balanceOf(address) view returns (uint256)',
    'function symbol() view returns (string)',
    'function decimals() view returns (uint8)',
  ]);

  const tokens = [
    '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
    '0xdAC17F958D2ee523a2206206994597C13D831ec7', // USDT
    '0x6B175474E89094C44Da98b954EedeAC495271d0F', // DAI
  ];
  const userAddress = '0xYourAddress';

  // 构建多个调用
  const calls = tokens.flatMap(token => [
    {
      target: token,
      allowFailure: true,
      callData: erc20Interface.encodeFunctionData('balanceOf', [userAddress]),
    },
    {
      target: token,
      allowFailure: true,
      callData: erc20Interface.encodeFunctionData('symbol', []),
    },
    {
      target: token,
      allowFailure: true,
      callData: erc20Interface.encodeFunctionData('decimals', []),
    },
  ]);

  // 单次RPC调用获取所有数据
  const results = await multicall.aggregate3.staticCall(calls);

  // 解析结果
  for (let i = 0; i < tokens.length; i++) {
    const balanceResult = results[i * 3];
    const symbolResult = results[i * 3 + 1];
    const decimalsResult = results[i * 3 + 2];

    if (balanceResult.success) {
      const balance = erc20Interface.decodeFunctionResult('balanceOf', balanceResult.returnData)[0];
      const symbol = erc20Interface.decodeFunctionResult('symbol', symbolResult.returnData)[0];
      const decimals = erc20Interface.decodeFunctionResult('decimals', decimalsResult.returnData)[0];
      console.log(`${symbol}: ${ethers.formatUnits(balance, decimals)}`);
    }
  }
}
```

### eth_getLogs优化

`eth_getLogs`是最容易导致性能问题的RPC方法——请求范围太大会超时或被限流。

**最佳实践**：

```typescript
// 错误：一次查询太大范围
const logs = await provider.getLogs({
  fromBlock: 0,
  toBlock: 'latest',
  address: tokenAddress,
});  // ❌ 超时或被RPC限流

// 正确：分块查询
async function getLogsInChunks(
  provider: ethers.Provider,
  filter: ethers.Filter,
  fromBlock: number,
  toBlock: number,
  chunkSize: number = 2000  // Alchemy推荐≤2000
): Promise<ethers.Log[]> {
  const allLogs: ethers.Log[] = [];

  for (let start = fromBlock; start <= toBlock; start += chunkSize) {
    const end = Math.min(start + chunkSize - 1, toBlock);
    const logs = await provider.getLogs({
      ...filter,
      fromBlock: start,
      toBlock: end,
    });
    allLogs.push(...logs);

    // 如果一个chunk返回太多日志，缩小chunk大小
    if (logs.length > 5000) {
      chunkSize = Math.floor(chunkSize / 2);
    }
  }

  return allLogs;
}

// 最优：使用topic过滤减少数据量
const transferLogs = await provider.getLogs({
  fromBlock: blockNumber - 1000,
  toBlock: blockNumber,
  address: tokenAddress,
  topics: [
    ethers.id('Transfer(address,address,uint256)'),
    null,  // from（不过滤）
    ethers.zeroPadValue(myAddress, 32),  // to（只看转入我的）
  ],
});
```

**各服务商eth_getLogs限制**：

| 服务商 | 最大区块范围 | 最大返回日志数 | 说明 |
|--------|------------|-------------|------|
| Alchemy | 2,000块 | 10,000 | 推荐每次<2000块 |
| Infura | 10,000块 | 10,000 | |
| QuickNode | 100,000块 | 10,000 | 较宽松 |
| 自建节点 | 无限制 | 无限制 | 但大范围查询会很慢 |

---

## 四、多RPC负载均衡

### 架构设计

```
                        ┌──────────────┐
                        │   应用层      │
                        └──────┬───────┘
                               │
                        ┌──────▼───────┐
                        │  RPC Router  │
                        │  (负载均衡)   │
                        └──────┬───────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                  │
       ┌──────▼─────┐  ┌──────▼─────┐  ┌────────▼──────┐
       │ 自建节点    │  │ Alchemy    │  │ QuickNode    │
       │ (Primary)   │  │ (Secondary)│  │ (Fallback)   │
       │ 延迟: 1ms   │  │ 延迟: 80ms │  │ 延迟: 60ms   │
       └────────────┘  └────────────┘  └───────────────┘
```

### Go语言负载均衡实现

```go
package rpcrouter

import (
    "context"
    "encoding/json"
    "fmt"
    "math/rand"
    "net/http"
    "sync"
    "sync/atomic"
    "time"
)

type RPCEndpoint struct {
    Name          string
    URL           string
    Priority      int           // 优先级（越小越优先）
    Weight        int           // 权重（用于加权轮询）
    MaxRetries    int
    Timeout       time.Duration
    IsHealthy     atomic.Bool
    AvgLatency    atomic.Int64  // 微秒
    ErrorCount    atomic.Int64
    RequestCount  atomic.Int64
}

type RPCRouter struct {
    endpoints []*RPCEndpoint
    mu        sync.RWMutex
    client    *http.Client
    strategy  RoutingStrategy
}

type RoutingStrategy int

const (
    StrategyPriority       RoutingStrategy = iota // 按优先级，高优先级不可用时fallback
    StrategyLatencyAware                          // 选延迟最低的
    StrategyWeightedRound                         // 加权轮询
)

func NewRPCRouter(strategy RoutingStrategy) *RPCRouter {
    return &RPCRouter{
        endpoints: make([]*RPCEndpoint, 0),
        client: &http.Client{
            Transport: &http.Transport{
                MaxIdleConns:        100,
                MaxIdleConnsPerHost: 20,
                IdleConnTimeout:     90 * time.Second,
                DisableCompression:  true,
            },
        },
        strategy: strategy,
    }
}

func (r *RPCRouter) AddEndpoint(ep *RPCEndpoint) {
    ep.IsHealthy.Store(true)
    r.mu.Lock()
    r.endpoints = append(r.endpoints, ep)
    r.mu.Unlock()
}

// SelectEndpoint 根据策略选择最佳端点
func (r *RPCRouter) SelectEndpoint() *RPCEndpoint {
    r.mu.RLock()
    defer r.mu.RUnlock()

    var candidates []*RPCEndpoint
    for _, ep := range r.endpoints {
        if ep.IsHealthy.Load() {
            candidates = append(candidates, ep)
        }
    }

    if len(candidates) == 0 {
        // 所有节点都不健康，返回第一个（尝试恢复）
        if len(r.endpoints) > 0 {
            return r.endpoints[0]
        }
        return nil
    }

    switch r.strategy {
    case StrategyPriority:
        // 选优先级最高（数字最小）的健康节点
        best := candidates[0]
        for _, ep := range candidates[1:] {
            if ep.Priority < best.Priority {
                best = ep
            }
        }
        return best

    case StrategyLatencyAware:
        // 选平均延迟最低的
        best := candidates[0]
        bestLatency := best.AvgLatency.Load()
        for _, ep := range candidates[1:] {
            lat := ep.AvgLatency.Load()
            if lat < bestLatency {
                best = ep
                bestLatency = lat
            }
        }
        return best

    case StrategyWeightedRound:
        // 加权随机
        totalWeight := 0
        for _, ep := range candidates {
            totalWeight += ep.Weight
        }
        r := rand.Intn(totalWeight)
        for _, ep := range candidates {
            r -= ep.Weight
            if r < 0 {
                return ep
            }
        }
        return candidates[0]

    default:
        return candidates[0]
    }
}

// Call 发送RPC请求（带重试和failover）
func (r *RPCRouter) Call(ctx context.Context, method string, params interface{}) (json.RawMessage, error) {
    var lastErr error

    // 尝试所有端点
    tried := make(map[string]bool)
    for attempt := 0; attempt < len(r.endpoints); attempt++ {
        ep := r.SelectEndpoint()
        if ep == nil {
            return nil, fmt.Errorf("no available RPC endpoints")
        }
        if tried[ep.Name] {
            continue
        }
        tried[ep.Name] = true

        result, err := r.callEndpoint(ctx, ep, method, params)
        if err == nil {
            return result, nil
        }

        lastErr = err
        ep.ErrorCount.Add(1)

        // 连续错误超过阈值，标记为不健康
        if ep.ErrorCount.Load() > 5 {
            ep.IsHealthy.Store(false)
        }
    }

    return nil, fmt.Errorf("all RPC endpoints failed, last error: %w", lastErr)
}

func (r *RPCRouter) callEndpoint(ctx context.Context, ep *RPCEndpoint, method string, params interface{}) (json.RawMessage, error) {
    start := time.Now()
    defer func() {
        latency := time.Since(start).Microseconds()
        // 指数移动平均更新延迟
        old := ep.AvgLatency.Load()
        newAvg := (old*7 + latency*3) / 10  // alpha=0.3
        ep.AvgLatency.Store(newAvg)
        ep.RequestCount.Add(1)
    }()

    payload := map[string]interface{}{
        "jsonrpc": "2.0",
        "id":      1,
        "method":  method,
        "params":  params,
    }

    body, _ := json.Marshal(payload)

    reqCtx, cancel := context.WithTimeout(ctx, ep.Timeout)
    defer cancel()

    req, _ := http.NewRequestWithContext(reqCtx, "POST", ep.URL, bytes.NewReader(body))
    req.Header.Set("Content-Type", "application/json")

    resp, err := r.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var rpcResp struct {
        Result json.RawMessage `json:"result"`
        Error  *struct {
            Code    int    `json:"code"`
            Message string `json:"message"`
        } `json:"error"`
    }

    json.NewDecoder(resp.Body).Decode(&rpcResp)

    if rpcResp.Error != nil {
        return nil, fmt.Errorf("RPC error %d: %s", rpcResp.Error.Code, rpcResp.Error.Message)
    }

    // 成功，重置错误计数
    ep.ErrorCount.Store(0)
    ep.IsHealthy.Store(true)

    return rpcResp.Result, nil
}

// StartHealthCheck 启动定期健康检查
func (r *RPCRouter) StartHealthCheck(ctx context.Context, interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            for _, ep := range r.endpoints {
                go func(ep *RPCEndpoint) {
                    checkCtx, cancel := context.WithTimeout(ctx, 3*time.Second)
                    defer cancel()

                    _, err := r.callEndpoint(checkCtx, ep, "eth_blockNumber", []interface{}{})
                    if err != nil {
                        ep.IsHealthy.Store(false)
                    } else {
                        ep.IsHealthy.Store(true)
                        ep.ErrorCount.Store(0)
                    }
                }(ep)
            }
        case <-ctx.Done():
            return
        }
    }
}
```

### Nginx反向代理配置

```nginx
# /etc/nginx/conf.d/rpc-proxy.conf

upstream eth_rpc_backends {
    # 自建节点（最高权重）
    server 127.0.0.1:8545 weight=10 max_fails=3 fail_timeout=10s;
    # Alchemy（备用）
    server alchemy-proxy:8545 weight=3 max_fails=5 fail_timeout=30s backup;
    # QuickNode（备用）
    server quicknode-proxy:8545 weight=2 max_fails=5 fail_timeout=30s backup;
}

# 限流配置
limit_req_zone $binary_remote_addr zone=rpc_limit:10m rate=100r/s;

server {
    listen 8080;
    server_name rpc.yourdomain.com;

    # 基本安全
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;

    location / {
        # 限流
        limit_req zone=rpc_limit burst=200 nodelay;

        # JSON-RPC只接受POST
        if ($request_method != POST) {
            return 405;
        }

        proxy_pass http://eth_rpc_backends;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;

        # 超时设置
        proxy_connect_timeout 5s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        # 缓冲
        proxy_buffering on;
        proxy_buffer_size 16k;
        proxy_buffers 4 32k;

        # 失败重试
        proxy_next_upstream error timeout http_502 http_503;
        proxy_next_upstream_tries 3;
    }

    # WebSocket代理
    location /ws {
        proxy_pass http://127.0.0.1:8546;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }

    # 健康检查端点
    location /health {
        access_log off;
        return 200 'OK';
    }
}
```

### HAProxy配置（更适合RPC负载均衡）

```haproxy
# /etc/haproxy/haproxy.cfg

global
    maxconn 50000
    log stdout format raw local0
    stats socket /var/run/haproxy.sock mode 660

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s
    retries 3
    option httpchk POST / HTTP/1.1\r\nContent-Type:\ application/json\r\n\r\n{\"jsonrpc\":\"2.0\",\"method\":\"eth_blockNumber\",\"params\":[],\"id\":1}
    http-check expect status 200

# RPC HTTP端点
frontend rpc_frontend
    bind *:8080
    default_backend rpc_backends

    # 只接受POST请求
    acl is_post method POST
    http-request deny unless is_post

    # 限流
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    acl rate_limit_exceeded sc0_http_req_rate gt 200
    http-request deny deny_status 429 if rate_limit_exceeded
    http-request track-sc0 src

backend rpc_backends
    balance leastconn  # 最少连接优先

    # 主要端点（自建节点）
    server local-geth 127.0.0.1:8545 check inter 5s fall 3 rise 2 weight 100

    # 备用端点
    server alchemy ALCHEMY_HOST:443 check inter 10s fall 5 rise 2 weight 30 ssl verify none backup
    server quicknode QUICKNODE_HOST:443 check inter 10s fall 5 rise 2 weight 20 ssl verify none backup

    # 响应时间追踪
    option httplog

# WebSocket端点
frontend ws_frontend
    bind *:8081
    default_backend ws_backends

backend ws_backends
    balance roundrobin
    server local-geth-ws 127.0.0.1:8546 check

# 统计面板
frontend stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
```

---

## 五、MEV专用RPC

### Flashbots eth_sendBundle

```typescript
import { ethers, Wallet } from 'ethers';
import { FlashbotsBundleProvider } from '@flashbots/ethers-provider-bundle';

async function sendFlashbotsBundle() {
  const provider = new ethers.JsonRpcProvider('http://localhost:8545');
  const authSigner = new Wallet('0xYOUR_AUTH_KEY'); // 用于签名bundle请求

  const flashbotsProvider = await FlashbotsBundleProvider.create(
    provider,
    authSigner,
    'https://relay.flashbots.net',  // Flashbots Relay
    'mainnet'
  );

  const targetBlockNumber = await provider.getBlockNumber() + 1;

  const bundleResponse = await flashbotsProvider.sendBundle(
    [
      {
        signer: wallet,
        transaction: {
          to: targetAddress,
          data: txData,
          gasLimit: 300000,
          maxFeePerGas: ethers.parseUnits('50', 'gwei'),
          maxPriorityFeePerGas: ethers.parseUnits('3', 'gwei'),
          chainId: 1,
          type: 2,
        },
      },
    ],
    targetBlockNumber
  );

  if ('error' in bundleResponse) {
    console.error('Bundle error:', bundleResponse.error);
    return;
  }

  // 等待bundle被打包
  const waitResponse = await bundleResponse.wait();
  console.log('Bundle status:', waitResponse);

  // 模拟检查盈利性
  const simulation = await bundleResponse.simulate();
  console.log('Simulation result:', simulation);
}
```

### 多Builder提交

```typescript
// Builder API端点列表（2026年）
const BUILDER_ENDPOINTS = {
  flashbots: {
    url: 'https://relay.flashbots.net',
    method: 'eth_sendBundle',
  },
  beaverbuild: {
    url: 'https://rpc.beaverbuild.org',
    method: 'eth_sendBundle',
  },
  titan: {
    url: 'https://rpc.titanbuilder.xyz',
    method: 'eth_sendBundle',
  },
  rsync: {
    url: 'https://rsync-builder.xyz',
    method: 'eth_sendBundle',
  },
  bloxroute: {
    url: 'https://mev.api.blxrbdn.com',
    method: 'blxr_submit_bundle',   // bloXroute使用自己的方法名
    authHeader: 'Authorization',
  },
  buildernet: {
    url: 'https://rpc.buildernet.org',
    method: 'eth_sendBundle',
  },
};

async function submitToAllBuilders(bundle: any, targetBlock: number) {
  const results = await Promise.allSettled(
    Object.entries(BUILDER_ENDPOINTS).map(async ([name, config]) => {
      const start = Date.now();
      try {
        const response = await fetch(config.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'X-Flashbots-Signature': `${authSignerAddress}:${signature}`,
          },
          body: JSON.stringify({
            jsonrpc: '2.0',
            id: 1,
            method: config.method,
            params: [{
              txs: bundle.signedTransactions,
              blockNumber: `0x${targetBlock.toString(16)}`,
              minTimestamp: 0,
              maxTimestamp: 0,
            }],
          }),
        });

        const data = await response.json();
        return {
          builder: name,
          success: !data.error,
          latency: Date.now() - start,
          response: data,
        };
      } catch (e) {
        return {
          builder: name,
          success: false,
          latency: Date.now() - start,
          error: e,
        };
      }
    })
  );

  // 汇总结果
  for (const result of results) {
    if (result.status === 'fulfilled') {
      const r = result.value;
      console.log(`${r.builder}: ${r.success ? 'OK' : 'FAIL'} (${r.latency}ms)`);
    }
  }
}
```

### MEV-protected交易提交

用户侧防止被MEV攻击的RPC：

| RPC服务 | 端点 | 保护类型 | 费用 |
|---------|------|---------|------|
| Flashbots Protect | https://rpc.flashbots.net | 防frontrun | 免费 |
| MEV Blocker | https://rpc.mevblocker.io | 防frontrun+backrun退款 | 免费 |
| bloXroute Protect | 需配置 | 防frontrun | 免费tier有 |
| SecureRPC (Manifold) | https://api.securerpc.com/v1 | 防frontrun | 免费 |

**集成到MetaMask/钱包**：
```
MetaMask → 设置 → 网络 → 添加自定义RPC
  网络名称: Ethereum (MEV Protected)
  RPC URL: https://rpc.flashbots.net
  链ID: 1
  符号: ETH
```

---

## 六、成本优化策略

### 请求合并

```
优化前（100个代币余额查询）：
  100次 eth_call → 100 CU/次 = 10,000 CU

优化后（Multicall合并）：
  1次 eth_call (multicall) → 26 CU + 数据量附加 ≈ 200 CU

节省：98%
```

### 缓存策略

```typescript
// 分层缓存策略
interface CacheStrategy {
  // 永不变数据：chainId, contractCode（部署后不变）
  immutable: {
    ttl: 'forever';
    methods: ['eth_chainId', 'eth_getCode'];
  };

  // 每区块更新：blockNumber, balance, nonce
  perBlock: {
    ttl: '12s';  // 以太坊出块时间
    methods: ['eth_blockNumber', 'eth_getBalance', 'eth_getTransactionCount'];
  };

  // 短缓存：gasPrice
  shortLived: {
    ttl: '3s';
    methods: ['eth_gasPrice', 'eth_maxPriorityFeePerGas'];
  };

  // 不缓存：交易提交、pending状态
  noCache: {
    methods: ['eth_sendRawTransaction', 'eth_getTransactionReceipt'];
  };
}
```

### 预算分档推荐

#### 入门方案（$0-100/月）

```
RPC策略：
├── 主要：Alchemy Free（300M CU/月）
├── 备用：Ankr Free（30M请求/月）
├── WebSocket：Alchemy Free
└── 预算：$0

适用：
├── 个人项目
├── 低频dApp
└── 开发/测试
```

#### 中端方案（$100-500/月）

```
RPC策略：
├── 主要：自建节点 Reth（Hetzner ~$200/月）
├── 备用：Alchemy Growth（$49/月）
├── WebSocket：自建节点
├── 负载均衡：nginx（同机部署）
└── 预算：~$250/月

适用：
├── 生产dApp
├── 中等请求量（<100M/月）
├── 需要WebSocket实时数据
└── 入门MEV
```

#### 高端方案（$500-5000/月）

```
RPC策略：
├── 主要：自建节点集群（2-3台 Hetzner ~$600/月）
├── 备用：Alchemy Scale（$499/月）
├── 第三方补充：QuickNode Growth（$299/月）
├── MEV：Flashbots + 多builder直连
├── 加速：Fiber Network（$500-2000/月）
├── 负载均衡：HAProxy集群
└── 预算：~$2,000-4,000/月

适用：
├── 高频交易
├── MEV搜索者
├── 高可用dApp服务
└── 数据分析平台
```

---

## 七、决策模板

### RPC方案选型决策树

```
你的月请求量？
├── <10M请求
│   ├── 需要WebSocket？
│   │   ├── 是 → Alchemy Free + WebSocket
│   │   └── 否 → Alchemy Free / Ankr Free
│   └── 预算 = $0
│
├── 10M-100M请求
│   ├── 需要低延迟（<50ms）？
│   │   ├── 是 → 自建节点 + 第三方备用
│   │   └── 否 → Alchemy Growth
│   └── 预算 = $50-300/月
│
├── 100M-1B请求
│   ├── MEV场景？
│   │   ├── 是 → 自建节点集群 + Fiber + 多builder
│   │   └── 否 → 自建节点 + Alchemy Scale作为fallback
│   └── 预算 = $300-2000/月
│
└── >1B请求
    └── 自建节点集群 + HAProxy + 按需扩展
        └── 预算 = $2000+/月
```

### 上线前检查清单

```
□ 主RPC端点已测试（延迟、可用性）
□ 备用RPC端点已配置
□ 负载均衡/failover已测试
□ 健康检查间隔已设置
□ 限流已配置（防止滥用导致账单爆炸）
□ WebSocket重连逻辑已实现
□ Batch请求已实现（减少RPC调用次数）
□ Multicall已集成（合并eth_call）
□ eth_getLogs分块查询已实现
□ 缓存已实现（不变数据、每区块数据）
□ 监控已部署（RPC延迟、错误率、请求量）
□ 告警已配置（endpoint不可用、延迟异常）
□ 成本追踪已设置（CU消耗/请求量预算告警）
□ API密钥已安全存储（不硬编码在代码中）
□ CORS/安全头已配置（如果公开暴露RPC代理）
```

---

## 八、常见问题排查

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 429 Too Many Requests | 超过服务商限流 | 增加限流缓冲、升级套餐、添加备用RPC |
| eth_getLogs超时 | 查询范围太大 | 分块查询，每次≤2000块 |
| WebSocket断开 | 服务商连接限制/idle timeout | 实现心跳+自动重连 |
| 数据不一致 | 不同节点链头不同步 | 使用特定block number查询，避免"latest" |
| 交易提交失败 | nonce冲突/gas不足 | 本地管理nonce，使用eth_maxPriorityFeePerGas |
| archive数据不可用 | 节点不是archive模式 | 使用archive节点或第三方archive API |
| 延迟抖动大 | 共享基础设施负载波动 | 自建节点或使用专用tier |
| CU消耗太快 | debug/trace API消耗高 | 仅在必要时调用，缓存结果 |

---

## Skill联动

- **[node-optimization](../node-optimization/SKILL.md)**：自建节点是RPC策略的核心组件，节点性能直接决定自建RPC的质量
- **[low-latency-infrastructure](../low-latency-infrastructure/SKILL.md)**：RPC端点策略是低延迟基础设施的数据访问层
- **[mev-bundle-construction](../mev-bundle-construction/SKILL.md)**：MEV bundle的提交通道选择依赖于RPC端点策略

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
