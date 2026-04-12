# TG Stars 支付系统

> 来源: https://core.telegram.org/bots/payments-stars

## 1. Stars 支付概述

- Telegram Stars (XTR) 是平台内数字货币
- 数字商品和服务必须使用 Stars 支付（Apple/Google 合规要求）
- 开发者可将 Stars 收入用于投放 Telegram Ads 或提现

## 2. 支付流程

```
用户触发支付 → Bot 创建 Invoice → 用户确认支付
                                      ↓
Bot 收到 pre_checkout_query → 10秒内 answerPreCheckoutQuery
                                      ↓
                           支付成功 → successful_payment 更新
                                      ↓
                           Bot 交付数字商品/服务
```

### 详细步骤

1. **创建 Invoice**
   - `sendInvoice`: 直接发送到聊天
   - `createInvoiceLink`: 创建可分享的支付链接
   - currency 必须设为 "XTR"
   - provider_token 留空（数字商品）

2. **预检查 (pre_checkout_query)**
   - 用户点击支付按钮后触发
   - Bot 必须在 10 秒内调用 `answerPreCheckoutQuery`
   - 可在此步验证库存/资格/定价

3. **支付确认 (successful_payment)**
   - 支付成功后 Bot 收到此更新
   - 包含 `telegram_payment_charge_id`（必须保存，用于退款）
   - 此时交付商品/服务

## 3. Invoice 类型

### Bot Invoice (sendInvoice)
```javascript
await bot.api.sendInvoice(chatId, "Premium Access", "30 days premium", "premium_30d", "", "XTR", [
  { label: "Premium 30 Days", amount: 100 } // 100 Stars
]);
```

### Inline Invoice
- 使用 `InputInvoiceMessageContent`
- 用户可转发到其他聊天，多人可购买
- 适合可分享的商品

### 创建链接
```javascript
const link = await bot.api.createInvoiceLink(
  "Premium Access", "30 days premium", "premium_30d", "", "XTR",
  [{ label: "Premium 30 Days", amount: 100 }]
);
// 通过 openInvoice(link) 在 Mini App 中打开
```

## 4. Mini App 中使用 Stars 支付

```javascript
// Mini App 前端
const invoiceLink = await fetch('/api/create-invoice').then(r => r.json());
window.Telegram.WebApp.openInvoice(invoiceLink.url, (status) => {
  if (status === 'paid') {
    // 更新 UI，显示购买成功
  } else if (status === 'cancelled') {
    // 用户取消
  } else if (status === 'failed') {
    // 支付失败
  }
});
```

## 5. 退款处理

```javascript
// refundStarPayment(user_id, telegram_payment_charge_id)
await bot.api.refundStarPayment(userId, chargeId);
```

- 需要保存 `telegram_payment_charge_id`
- 退款后 Stars 返回给用户
- 开发者需自行处理商品回收逻辑

## 6. 定价策略

### 参考定价
- 入门级道具: 1-10 Stars
- 中级功能: 10-100 Stars
- 高级/订阅: 100-1000 Stars
- 用户实际支付金额因地区 VAT 不同而异

### 最佳实践
- 提供多个价格档位
- 虚拟货币包（批量购买 Stars 折扣由 Telegram 控制）
- 限时优惠促进冲动消费
- 首充奖励降低付费门槛

## 7. 分成机制

- Telegram 抽取一定比例佣金
- 具体分成比例见官方文档表格（因渠道和金额而异）
- 开发者净收入 = 用户支付 - 平台佣金 - VAT
- 收入可用于: 投放 Telegram Ads / 提现

## 8. Stars vs TON 支付选型

| 维度 | Stars 支付 | TON 支付 |
|------|-----------|---------|
| 合规性 | iOS/Android 合规 | 仅限 Web/Desktop |
| 用户门槛 | 低（信用卡/Google Pay） | 高（需钱包+TON） |
| 手续费 | Telegram 抽成 | 链上 Gas 费（极低） |
| 结算 | Stars 余额 | 即时到账 TON |
| 适用场景 | 数字商品/服务 | 链上资产/NFT/DeFi |
| 退款 | 支持（refundStarPayment） | 不可逆 |

**推荐策略**: 数字商品用 Stars（合规+低门槛），链上资产用 TON Connect。

## 9. 上线要求

- 必须实现 `/terms` 命令（展示服务条款）
- 必须实现 `/support` 命令（用户支持入口）
- 开启两步验证
- 稳定的基础设施
- 同意 Telegram Terms of Service
- 保存支付记录用于争议处理
- 及时响应用户支持请求
