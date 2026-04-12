# TON Connect 与 Web3 集成

> 来源: https://docs.ton.org/develop/dapps/ton-connect/overview, https://docs.ton.org/develop/dapps/ton-connect/react

## 1. TON Connect 2.0 协议

### 概述
- TON 生态标准钱包连接协议
- 支持 30+ 钱包和数百个主流应用
- 安全通信模型: 用户授权交易，同时保持私钥控制
- 通过 HTTP bridge 实现钱包与 dApp 间通信

### 核心组件
- **App Manifest**: dApp 声明文件（名称/URL/图标）
- **Bridge**: 钱包与 dApp 间的通信桥接
- **Session**: 加密会话，持久化连接状态

## 2. SDK 选型

| SDK | 用途 | 适用场景 |
|-----|------|---------|
| @tonconnect/sdk | 底层协议实现 | 自定义 UI |
| @tonconnect/ui | 预制 UI 组件 | 原生 JS 项目 |
| @tonconnect/ui-react | React 封装 | React 项目（推荐） |
| @tonconnect/protocol | 协议层 | 钱包开发者 |

## 3. Manifest 配置

```json
// tonconnect-manifest.json（部署在应用根目录）
{
  "url": "https://your-app.com",
  "name": "Your App Name",
  "iconUrl": "https://your-app.com/icon.png"
}
```

**要求**: CORS 必须关闭，不能有授权或中间代理。

## 4. React 集成（推荐方案）

### 安装
```bash
npm i @tonconnect/ui-react
```

### Provider 配置
```tsx
import { TonConnectUIProvider } from '@tonconnect/ui-react';

function App() {
  return (
    <TonConnectUIProvider manifestUrl="https://your-app.com/tonconnect-manifest.json">
      {/* 应用内容 */}
    </TonConnectUIProvider>
  );
}
```

### 连接按钮
```tsx
import { TonConnectButton } from '@tonconnect/ui-react';

// 显示"Connect Wallet"按钮，连接后变为钱包菜单
<TonConnectButton />
```

### 核心 Hooks
```tsx
import { useTonConnectUI, useTonWallet, useTonAddress } from '@tonconnect/ui-react';

// 获取 UI 控制器
const [tonConnectUI, setOptions] = useTonConnectUI();

// 获取钱包信息
const wallet = useTonWallet();

// 获取地址（userFriendly / raw）
const address = useTonAddress();
const rawAddress = useTonAddress(false);
```

### 手动触发连接
```tsx
// 打开钱包选择弹窗
tonConnectUI.openModal();

// 指定钱包连接
tonConnectUI.openSingleWalletModal('tonkeeper');
```

## 5. 发送交易

```tsx
const transaction = {
  validUntil: Math.floor(Date.now() / 1000) + 300, // 5分钟有效期
  network: CHAIN.MAINNET, // 或 CHAIN.TESTNET
  messages: [
    {
      address: "EQxxxx...", // 目标地址
      amount: "1000000000", // nanoToncoin (1 TON = 10^9)
      // payload: "base64编码的Cell" // 可选，用于合约调用
    }
  ]
};

const result = await tonConnectUI.sendTransaction(transaction);
```

**注意**: 金额使用 nanoToncoin，1 TON = 10^9 nanoToncoin，需考虑手续费。

## 6. UI 定制

```tsx
tonConnectUI.uiOptions = {
  language: 'zh', // 界面语言
  uiPreferences: {
    theme: THEME.DARK // DARK / LIGHT / SYSTEM
  }
};
```

## 7. TON 区块链基础

### 地址
- 两种格式: User-Friendly（base64url编码）和 Raw（workchain:hex）
- Testnet 和 Mainnet 地址不同
- 合约部署后地址由代码和初始数据决定

### Jetton (FT)
- TON 上的同质化代币标准
- 每个持有者有独立的 Jetton Wallet 合约
- 转账通过 Jetton Wallet 间的内部消息实现
- 查询余额需请求对应 Jetton Wallet 合约

### NFT
- 每个 NFT 是独立的合约
- Collection 合约管理集合元数据
- 支持链上元数据和链下元数据（IPFS/HTTP）

### 智能合约交互库
- **tonweb**: 轻量级 JS 库，适合简单交互
- **ton-core / ton**: 全功能 TypeScript SDK
- **tonclient**: C++ 绑定，高性能

## 8. 链上积分方案

### Jetton 积分
- 发行自定义 Jetton 作为游戏积分
- 合约控制铸造/销毁/转账权限
- 可与 DEX 集成实现交易

### NFT 奖励
- 游戏道具/成就 NFT 化
- 玩家间自由交易
- 提升用户粘性和社区经济

### 混合模式（推荐）
- 链下积分（高频操作，低延迟）
- 链上资产（低频操作，可验证/可交易）
- 定期结算: 链下积分批量上链
