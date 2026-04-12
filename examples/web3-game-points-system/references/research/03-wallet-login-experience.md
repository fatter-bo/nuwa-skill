# 03 钱包登录体验 — 无感接入设计

## 核心命题

休闲棋牌玩家画像：非加密原住民、可能连"钱包"概念都不了解。如果要求安装MetaMask或记助记词，99%的用户直接流失。钱包必须"隐形"。

---

## 技术方案对比

### 三种嵌入式钱包方案

| 方案 | 代表产品 | 登录方式 | 密钥管理 | 用户感知 |
|------|---------|---------|---------|---------|
| MPC钱包 | Web3Auth, Privy, Portal | 邮箱/社交登录 | 密钥分片存储(TSS/SSS) | 完全无感 |
| AA钱包(ERC-4337) | Alchemy AA, Biconomy | 邮箱/社交 + 智能合约钱包 | 合约控制 | 无感 + 高级功能 |
| 嵌入式EOA(EIP-7702) | MetaMask Embedded | 社交登录 | MPC + EOA升级 | 无感 + 兼容性好 |

### 推荐组合：MPC + AA

```
用户视角：
1. 打开游戏 → 点击"微信登录"/"手机号登录"
2. 后台自动生成MPC钱包地址
3. AA Paymaster代付所有Gas
4. 用户全程不知道自己有个"链上地址"

技术实现：
微信OAuth → Web3Auth SDK → MPC密钥分片生成 → 
ERC-4337 Smart Account创建 → Paymaster注册 → 
游戏积分合约授权
```

---

## 主流服务商对比（2025-2026）

| 服务商 | SDK平台 | 社交登录 | Gas代付 | 游戏集成 | 收购/归属 |
|--------|---------|---------|---------|---------|---------|
| Web3Auth | Web/iOS/Android/Unity | Google/Apple/Discord/Twitter/邮箱 | 支持(配合AA) | Unity SDK | MetaMask (ConsenSys) |
| Privy | Web/React Native | 邮箱/手机号/社交 | 内置Paymaster | React为主 | Stripe收购(2025) |
| Dynamic | Web | 多链社交登录 | 支持 | Web优先 | Fireblocks收购 |
| Particle Network | Web/Unity/Unreal | 全社交平台 | 内置 | 游戏引擎深度集成 | 独立 |
| Openfort | Web/Unity/Unreal | 多种 | 内置 | 游戏专用 | 独立 |

**推荐：Particle Network 或 Web3Auth**
- Particle：游戏引擎原生支持最好，Unity/Unreal SDK成熟
- Web3Auth：被MetaMask收购，生态最大，73%新Web3项目使用ERC-4337

---

## 传统登录与钱包绑定

### 绑定流程

```
场景1: 新用户（从未有钱包）
手机号注册 → 自动创建MPC钱包 → 绑定到游戏账户
→ 钱包地址存储在用户Profile中（用户不可见）

场景2: 已有Web3钱包的用户
手机号登录 → 设置页面"连接钱包" → 
MetaMask/OKX签名验证 → 绑定外部钱包地址
→ 积分跟随游戏账户（非外部钱包）

场景3: 多设备同步
设备A登录 → MPC密钥分片已关联到账户
设备B登录同一账户 → 通过OAuth恢复密钥分片 → 
同一钱包地址，余额同步
```

### 密钥恢复方案

| 场景 | 恢复方式 |
|------|---------|
| 换手机 | 重新OAuth登录 → MPC分片从云端恢复 |
| 忘记密码 | 手机验证码 → 重置OAuth → 钱包地址不变 |
| 社交账号被封 | 预设备份恢复邮箱 / 安全问题 |
| 极端场景 | 游戏方作为MPC参与方可协助恢复（需KYC验证） |

---

## Session Key — 游戏场景的关键优化

### 问题
每次链上操作都需要签名 → 弹出确认框 → 打断游戏体验

### 解决方案：Session Key（ERC-4337扩展）

```
一次授权，限时自动签名：
- 用户进入游戏时签名一次（或登录时自动）
- 创建临时Session Key，有效期24小时
- Session Key权限范围：
  ✅ 积分合约的mint/burn操作
  ✅ 道具购买（单笔上限）
  ❌ 大额转移（超过阈值需重新授权）
  ❌ 合约升级等管理操作
- 到期自动失效，下次登录重新创建
```

---

## 关键调研来源

- ERC-4337: Account Abstraction Specification
- Web3Auth Documentation: MPC Key Management (TSS/SSS)
- Privy Documentation: Embedded Wallet Integration
- Particle Network: Unity/Unreal SDK for Web3 Gaming
- Alchemy 2025 Web3 Developer Report: 73% ERC-4337 Adoption
- MetaMask Embedded Wallets: Frictionless Onboarding
- EIP-7702: EOA Account Upgrade Proposal
