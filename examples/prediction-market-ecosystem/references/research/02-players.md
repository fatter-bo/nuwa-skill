# 02-players: 预测市场头部玩家深度分析

## Polymarket

### 核心数据
- 2025年交易量：$215亿
- 2026.02月交易量：$70亿+（单日记录~$4.7亿）
- 估值：$90亿+
- ICE累计投资：$16亿（2025.10 $10亿 + 2026.04 $6亿）
- CFTC DCM：2025.11获批

### 费率体系
- 动态概率费率：越接近50%概率 = 越高费率
- US版（DCM）：2%净利润费
- DeFi版：maker/taker模型+返佣
- 2026.03扩展至所有加密市场（1H/4H/日/周）
- 长期市场4% APR持仓奖励

### 体育合作
- NHL：首个预测市场联赛授权（2025.10）
- MLS：官方预测市场合作伙伴
- MLB：独家官方预测市场交易所合作伙伴（2026.03，通过Sportradar数据）
- LALIGA：合作关系

### 技术升级（2026.04）
- CTF Exchange V2
- Polymarket USD（1:1 USDC支持）
- 新CLOB v2
- POLY治理代币预期（未发行）

## Kalshi

### 核心数据
- 2025年收入：~$2.6亿（10倍YoY）
- 2025年交易量：$229亿
- 活跃市场：350,000+
- 体育占比：~90%
- 用户：数百万

### 差异化
- 从创立起就持有CFTC DCM牌照
- 完全中心化（无区块链）
- 法币入金（ACH、PayPal、Venmo、加密）
- 与Pyth合作将合规数据推上100+链

### 事件合约品类
体育、政治、经济、技术、气候、流行文化、加密、天气、国际事件、科学/文化

## Azuro

### 核心数据
- 累计预测量：$10亿+
- Take rate：4.28%
- 生态dApp：50+
- 人均交易量：$3,318；人均投注：$212；人均笔数：25
- 足球占比：69%

### 技术架构
- Liquidity Tree（段树数据结构）：单例池支撑所有并发市场
- vAMM赔率引擎
- 许可数据提供者 + Chainlink keeper节点
- 部署：Polygon、Gnosis、Chiliz

### Live Betting V3
- HostCore：Gnosis链上，注册比赛、管理生命周期
- LiveCore：动态投注层，快照保存法处理延迟投注
- Relayer：用户签EIP-712消息，Relayer执行
- 赛前事件无缝转为现场事件

## 新入场者

| 平台 | 时间 | 关键特点 |
|------|------|---------|
| DraftKings Predictions | 2025.12 | 17州，收购Railbird（CFTC注册） |
| FanDuel Predicts | 2025.12 | CME合作，5州 |
| OG.com（Crypto.com） | 2026.02 | 首个保证金交易预测市场，CDNA清算 |
| Robinhood | 2026预计 | 收购LedgerX，Susquehanna做市 |
| Betr × Polymarket | 2026.03 | 合作进入预测市场 |
| Fanatics Markets | 2025.12 | 最新入场 |

## 历史教训

### Augur
- 首个去中心化预测市场（2018）
- 失败原因：UX太差、gas太高、流动性不足、结算太慢
- 教训：去中心化≠好产品

### Gnosis/Omen
- 发明了CTF框架（现在Polymarket用）
- 自己的预测市场不成功
- 教训：基础设施价值>应用层价值
