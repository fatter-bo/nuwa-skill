# 维度三：多租户与白标体系

## 1. 白标棋牌平台的架构模型

### 核心理念

白标(White-Label)模式的本质：不同运营商用同一套后端引擎，但各自拥有独立的前端品牌、用户池、运营后台。运营商看到的是"自己的产品"，实际底层是同一个技术平台。

```
┌──────────────────────────────────────────────────────────┐
│                    Operator A (品牌A)                      │
│  独立域名 │ 独立UI │ 独立用户池 │ 独立运营后台 │ 独立钱包  │
├──────────────────────────────────────────────────────────┤
│                    Operator B (品牌B)                      │
│  独立域名 │ 独立UI │ 独立用户池 │ 独立运营后台 │ 独立钱包  │
├──────────────────────────────────────────────────────────┤
│                    Operator C (品牌C)                      │
│  独立域名 │ 独立UI │ 独立用户池 │ 独立运营后台 │ 独立钱包  │
╞══════════════════════════════════════════════════════════╡
│                  共享平台核心引擎                           │
│  游戏引擎 │ 匹配系统 │ 结算系统 │ 风控系统 │ 数据分析     │
└──────────────────────────────────────────────────────────┘
```

来源: [Multi-Tenant Casino Platforms for Multiple Brands](https://bettoblock.com/multi-tenant-casino-platforms-multiple-brands/)
来源: [GR8 Tech White Label Platform](https://gr8.tech/white-label/)

## 2. 数据隔离方案

### 三种隔离模型对比

| 模型 | 隔离级别 | 成本 | 适用场景 |
|------|---------|------|---------|
| 独立数据库(Database per Tenant) | 最高 | 最高 | 合规要求严格的大运营商 |
| 共享数据库独立Schema | 中等 | 中等 | 不推荐（复杂度高/隔离不充分） |
| 共享数据库+tenant_id | 最低 | 最低 | 中小运营商、快速接入 |

### 推荐：混合模型

```
大运营商（月流水>100万USD）→ 独立数据库实例
  优势：完全隔离、独立备份、独立扩容
  实现：每个大租户一个独立的数据库集群

中小运营商 → 共享数据库 + Row-Level Security
  优势：成本低、管理简单
  实现：所有表添加tenant_id字段，数据库层RLS强制隔离
```

### Row-Level Security实现示例

```sql
-- PostgreSQL RLS策略
ALTER TABLE game_records ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON game_records
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- 应用层在每次连接时设置当前租户
SET app.current_tenant = 'tenant_abc_uuid';
```

### 缓存隔离

```
Redis Key命名规范：
  {tenant_id}:{service}:{resource}:{id}
  
示例：
  tenant_abc:user:profile:12345
  tenant_xyz:lobby:rooms:texas_holdem
  tenant_abc:game:state:room_67890
```

来源: [Multi-Tenant Database Architecture Patterns](https://www.bytebase.com/blog/multi-tenant-database-architecture-patterns-explained/)
来源: [Multi-Tenant Architecture Complete Guide](https://dev.to/tak089/multi-tenant-architecture-a-complete-guide-basic-to-advanced-119o)
来源: [Azure SQL Multi-Tenant SaaS Patterns](https://learn.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns)

## 3. 租户自定义配置能力

### 可配置项边界

| 层级 | 可配置 | 不可配置 |
|------|-------|---------|
| 品牌层 | Logo/颜色/字体/域名/APP图标 | - |
| UI层 | 大厅布局/牌桌皮肤/音效包 | 核心交互流程 |
| 玩法层 | 开放哪些游戏/底注级别/抽水比例 | 游戏核心规则逻辑 |
| 运营层 | 活动配置/奖励规则/VIP体系/公告 | 风控策略核心算法 |
| 支付层 | 支付渠道选择/限额设置/提现规则 | 资金安全策略 |
| 数据层 | 报表维度/导出格式/告警阈值 | 底层数据模型 |

### 租户配置系统设计

```json
{
  "tenantId": "operator_abc",
  "brand": {
    "name": "皇家棋牌",
    "logo": "https://cdn.operator-abc.com/logo.png",
    "primaryColor": "#D4AF37",
    "theme": "luxury_gold"
  },
  "games": {
    "enabled": ["texas_holdem", "doudizhu", "mahjong_sichuan"],
    "disabled": ["baccarat", "blackjack"],
    "customConfig": {
      "texas_holdem": {
        "blindLevels": [10, 50, 100, 500, 1000],
        "rakePercent": 3,
        "rakeCap": 500
      }
    }
  },
  "payment": {
    "enabledChannels": ["alipay", "wechat", "usdt"],
    "minDeposit": 10,
    "maxDeposit": 50000,
    "withdrawalReviewThreshold": 10000
  },
  "operations": {
    "welcomeBonus": { "enabled": true, "amount": 88, "wagerRequirement": 10 },
    "vipLevels": 6,
    "dailyLoginReward": true
  }
}
```

来源: [Artoon Solutions - White Label Multi Gaming](https://artoonsolutions.com/white-label-multigaming-platform-provider/)

## 4. 前端白标方案

### 三种前端交付模式

**模式一：iframe嵌入**
- 最简单的接入方式
- 运营商页面中嵌入平台提供的游戏iframe
- 通过postMessage进行跨域通信
- 适合快速上线，但定制能力有限

**模式二：SDK集成**
- 提供JavaScript/Native SDK
- 运营商在自有APP/网站中调用SDK API
- 支持深度UI定制
- 需要一定的开发能力

**模式三：全套白标交付**
- 提供完整的前端源码模板
- 运营商可深度二次开发
- 独立部署、完全自主
- 成本最高、自由度最大

### 前端定制工具链

```
品牌定制工具 → 上传Logo/配色/字体 → 自动生成主题包
  ↓
主题包 = CSS变量 + 资源文件 + 配置JSON
  ↓
CDN分发 → 按tenant_id加载对应主题包
```

来源: [B2B iGaming Platform - Gamingsoft](https://www.gamingsoft.com/)
来源: [NEAPI White Label Platform](https://www.neapi.com/en/whitelabel.html)

## 5. 运营商后台系统

### 分层权限模型

```
超级管理员（平台方）
  ├── 全局配置管理
  ├── 租户管理（创建/冻结/删除运营商）
  ├── 全局风控
  └── 财务结算

运营商管理员
  ├── 本运营商用户管理
  ├── 游戏配置（开关游戏/调整参数）
  ├── 活动管理
  ├── 数据报表
  └── 客服工具

运营商子账号
  ├── 客服（只读用户信息+处理工单）
  ├── 运营（活动配置+数据查看）
  └── 财务（提现审核+财务报表）
```

来源: [Multi-Tenant Casino Platforms](https://bettoblock.com/multi-tenant-casino-platforms-multiple-brands/)
