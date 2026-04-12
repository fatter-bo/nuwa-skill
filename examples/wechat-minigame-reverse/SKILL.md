---
name: wechat-minigame-reverse
description: |
  微信小游戏逆向工程专家。系统化完成任意微信小游戏的技术分析、资源提取、
  玩法拆解、数值逆向，最终输出一份"可复刻规格书"。
  融合wxapkg解包技术、JavaScript反混淆方法论、游戏引擎识别（Cocos Creator/Egret/Laya/Unity）、
  网络协议抓包分析、数值体系逆向、GDD规格书标准等6个维度的系统性逆向分析工具。
  目标用户：游戏开发者、技术预研团队、竞品分析师。
  用途：对目标微信小游戏进行完整技术拆解，输出可直接指导开发的复刻规格书。
  当用户说「逆向分析」「小游戏拆解」「反编译」「解包」「wxapkg」「复刻分析」「技术预研」时使用。
  即使用户只是说「帮我分析这个小游戏」「拆一下这个游戏」「看看这个游戏怎么做的」也应触发。
---

# 微信小游戏逆向工程专家

> 任何小游戏都是代码、资源、数值三者的组合。逆向的本质是把打包后的黑盒还原为可理解的白盒。

## 使用说明

**这个Skill不扮演角色，是一个工具。** 用户说"帮我逆向分析XXX小游戏"时，按照下面的工作流自动执行。

---

## 执行工作流（Agentic Protocol）

**用户提供目标小游戏后，按以下流程执行：**

```
Phase 0: 目标确认与信息收集（5分钟）
  → 确认游戏名称、获取AppID
  → 判断分析深度（快速概览 / 完整逆向）
  → 确认用户已准备好wxapkg文件（或指导获取方法）

Phase 1: 包体解构（技术基础层）
  → 解包wxapkg
  → 识别引擎类型
  → 梳理目录结构
  → 输出：引擎识别报告 + 文件结构图

Phase 2: 资源提取（素材层）
  → 提取图片/音频/动画资源
  → 还原图集为独立sprite
  → 处理加密资源
  → 输出：资源清单 + 资源文件

Phase 3: 代码逆向（逻辑层）
  → JavaScript反混淆
  → 核心逻辑识别
  → 网络协议分析
  → 输出：关键代码注释 + 接口文档

Phase 4: 玩法拆解（设计层）
  → 核心循环提取
  → 付费/广告点识别
  → 社交裂变分析
  → 输出：玩法状态机图 + 变现方案

Phase 5: 数值逆向（策划层）
  → 概率表/掉落表提取
  → 经济参数分析
  → 平衡性验证
  → 输出：数值配置表集

Phase 6: 规格书输出（交付层）
  → 按模板整合所有分析结果
  → 生成可复刻规格书
  → 工作量评估
  → 输出：完整复刻规格书
```

---

## 核心方法论框架

### 框架1：引擎指纹识别法

**原理**：每种游戏引擎在打包时会留下独特的文件结构和代码特征，通过这些"指纹"可快速判断引擎类型，从而选择对应的逆向策略。

| 引擎 | 文件指纹 | 代码指纹 | 资源特征 |
|------|---------|---------|---------|
| Cocos Creator | `ccRequire.js`、`cocos/cocos2d-js-min.js`、`src/settings.js` | `cc.game`、`cc.director`、`cc.Node` | `assets/` 下按bundle组织 |
| LayaAir | `laya.wxmini.js`、`laya.js` | `Laya.init`、`Laya.stage`、`Laya3D` | `res/` 目录 |
| Egret | `egret.wxgame.js`、`manifest.js` | `egret.`、`eui.`、`RES.` | `resource/` 目录 |
| Unity小游戏 | `unity-namespace.js`、`wasmcode/`、`.wasm` | `UnityLoader`、`Module` | `StreamingAssets/` |
| 原生JS | 无引擎文件 | 直接操作 `wx.createCanvas()` | 自定义结构 |

**验证方式**：跨域复现——在任意已知引擎项目的构建产物中均可观察到对应指纹。

来源：[微信开放文档-引擎适配](https://developers.weixin.qq.com/minigame/dev/guide/game-engine/cocos-laya-egret.html)，[Cocos Creator发布文档](https://docs.cocos.com/creator/3.8/manual/en/editor/publish/publish-wechatgame.html)

### 框架2：分层剥离法（Layered Peeling）

**原理**：逆向分析应自底向上分层进行，每层的输出是上一层的输入。跳过底层直接分析上层会导致理解偏差。

```
第1层：包体层 → 解包、解密、获取原始文件
第2层：资源层 → 提取、还原、分类所有素材
第3层：代码层 → 反混淆、理解核心逻辑
第4层：设计层 → 从代码中抽象出玩法机制
第5层：数值层 → 从代码和配置中提取数值体系
第6层：架构层 → 综合所有信息形成规格书
```

**关键约束**：层间存在依赖关系——不解包就无法提取资源，不理解代码就无法拆解玩法。每一层的分析质量直接决定上层的准确性。

### 框架3：双通道验证法（Code-Behavior Cross-Validation）

**原理**：代码分析（静态）和行为观察（动态）必须相互验证。单靠任何一个通道都可能产生误判。

| 场景 | 静态分析发现 | 动态验证方法 |
|------|------------|------------|
| 概率系统 | 代码中的概率权重 | 大量抽样统计实际概率 |
| 关卡配置 | 配置文件中的参数 | 实际游玩验证难度感受 |
| 网络协议 | 代码中的接口调用 | 抓包验证实际数据交互 |
| 广告触发 | 代码中的广告位逻辑 | 实际体验各触发场景 |

**注意**：客户端代码中展示的概率可能与服务端实际概率不同。服务端逻辑无法通过客户端逆向获得，只能通过统计验证。

### 框架4：加密对抗决策树

**原理**：不同的资源保护方案需要不同的破解策略。盲目尝试会浪费时间。

```
资源文件异常？
├── 文件头不匹配标准格式
│   ├── 搜索代码中的decrypt/decipher函数 → 找到密钥 → 编写解密脚本
│   └── 未找到解密函数 → 服务端解密 → 只能抓包获取解密后资源
├── 文件名为hash值
│   └── 从配置文件（settings.js/fileconfig.json）中查找映射
├── 图片打开全黑/花屏
│   └── TexturePacker内容保护 → IDA调试或从代码搜索content protection key
└── 本地无资源文件
    └── 远程动态加载 → 抓包获取CDN地址 → 批量下载
```

来源：[IDA远程调试破解TexturePacker加密素材](https://blog.jtwo.me/ida-remote-debug-texturepacker-material/)

### 框架5：变现模型逆向矩阵

**原理**：微信小游戏的变现模式决定了其设计取向。通过识别变现模型可以快速理解设计意图。

| 变现模式 | 代码特征 | 设计影响 |
|---------|---------|---------|
| 纯IAA（广告变现） | `wx.createRewardedVideoAd`、多处`adUnitId` | 高频广告触发点、免费但有摩擦 |
| IAP为主（内购） | `wx.requestMidasPayment`、商店系统 | 付费墙设计、VIP特权、限时礼包 |
| IAA+IAP混合 | 两者兼有 | 广告去除特权、氪金加速 |
| 社交裂变驱动 | 大量`wx.shareAppMessage`、排行榜 | 分享奖励、好友PK、群排名 |

### 框架6：wxapkg双重解密流程

**原理**：PC端微信对wxapkg采用AES+XOR双重加密，必须按顺序解密。

```
加密文件（V1MMWX头）
  → 验证文件头为"V1MMWX"
  → 提取AppID
  → 生成AES Key: PBKDF2(AppID, "saltiest", 1000, SHA1, 32bytes)
  → AES解密前1023字节（IV="the iv: 16 bytes"）
  → XOR解密剩余字节（key=AppID倒数第2字符的ASCII值）
  → 得到标准wxapkg（0xBE头）
  → 按索引结构提取文件
```

来源：[BlackTrace/pc_wxapkg_decrypt](https://github.com/BlackTrace/pc_wxapkg_decrypt)，[pc_wxapkg_decrypt_python](https://github.com/renblog/pc_wxapkg_decrypt_python)

---

## 决策启发式

### 快速判断规则

1. **如果** game.js中出现`ccRequire` → **则** 使用Cocos Creator逆向工具链，关注`src/settings.js`和`assets/`目录
2. **如果** 文件头为"V1MMWX" → **则** 需要先解密再解包，使用KillWxapkg或wxapkg工具自动处理
3. **如果** 解包后发现`.wasm`文件 → **则** 游戏使用Unity引擎，代码逆向难度显著增加（C#→IL2CPP→WASM��
4. **如果** 抓包发现关卡配置从服务端下发 → **则** 关卡数据无法仅从客户端获取，需记录多关数据做统计分析
5. **如果** 代码中`Math.random()`调用极少 → **则** 概率计算可能在服务端完成，客户端概率数据不可信
6. **如果** 资源文件名为MD5 hash → **则** 在`settings.js`或引擎配置文件中搜索UUID-文件名映射表
7. **如果** 游戏包体中存在`subpackages/`目录 → **则** 游戏使用分包加载，需确保所有分包都已获取
8. **如果** JS代码中存在大量`_0x`前缀变量 → **则** 使用了javascript-obfuscator，先用de4js或自定义AST脚本还原字符串
9. **如果** 广告位只有Banner没有激励视频 → **则** 游戏可能以IAP为主要变现，关注商店和付费系统
10. **如果** 代码中频繁出现`wx.getOpenDataContext` → **则** 游戏重度依赖社交排行，需单独分析开放数据域的子包

---

## 工具与资源推荐

### 解包工具链

| 工具 | 语言 | 核心能力 | 推荐场景 | 链接 |
|------|------|---------|---------|------|
| KillWxapkg | Go | 解密+解包+Hook+重打包 | 全流程自动化 | [GitHub](https://github.com/Ackites/KillWxapkg) |
| wux1an/wxapkg | Go | 扫描+解密+解包+美化 | Windows快速操作 | [GitHub](https://github.com/wux1an/wxapkg) |
| wedecode | Node.js | 源码还原(wxml/wxss) | 小程序类项目 | [GitHub](https://github.com/biggerstar/wedecode) |
| unveilr | Node.js | 反编译还原 | 经典方案 | [GitHub](https://github.com/BugFans/unveilr) |

### 代码分析工具

| 工具 | 用途 | 链接 |
|------|------|------|
| js-beautify | JS代码格式化 | [npm](https://www.npmjs.com/package/js-beautify) |
| de4js | 在线JS反混淆 | [de4js](https://lelinhtinh.github.io/de4js/) |
| AST Explorer | AST可视化分析 | [astexplorer.net](https://astexplorer.net/) |
| Chrome DevTools | 运行时调试（配合开发者工具） | 内置 |

### 资源还原工具

| 工具 | 用途 | 链接 |
|------|------|------|
| texture-unpacker | 图集还原(plist/json) | [GitHub](https://github.com/pavle-goloskokovic/texture-unpacker) |
| TextureUnpacker | 图集还原(plist/atlas) | [GitHub](https://github.com/aa13058219642/TextureUnpacker) |
| Spine Viewer | Spine动画预览 | [esotericsoftware.com](http://esotericsoftware.com/) |

### 抓包工具

| 工具 | 适用场景 | 核心优势 | 链接 |
|------|---------|---------|------|
| Charles | HTTP/HTTPS/WebSocket | GUI友好，SSL代理 | [charlesproxy.com](https://www.charlesproxy.com/) |
| mitmproxy | HTTP/HTTPS+脚本化 | Python可编程，自动化篡改 | [mitmproxy.org](https://mitmproxy.org/) |
| Fiddler | Windows HTTP | 规则引擎强大 | [telerik.com](https://www.telerik.com/fiddler) |
| Wireshark | 底层协议 | 全协议支持 | [wireshark.org](https://www.wireshark.org/) |

---

## 复刻规格书输出模板

完整分析后，按以下结构输出规格书：

```
一、项目概述（游戏名/引擎/包体大小/核心玩法）
二、技术架构图（引擎层/逻辑层/UI层/平台层）
三、核心玩法状态机（所有状态+转换条件）
四、资源清单与替换规划（分类+规格+替换方案）
五、数值配置表（概率表/掉落表/合成配方/经济参数/成长曲线）
六、UI/UX线框图（页面列表+跳转关系+交互元素）
七、网络接口清单（接口路径/方法/参数/响应）
八、广告与变现方案（广告位/类型/触发条件/频次）
九、社交功能清单（功能/实现方式/触发场景）
十、开发工作量评估（模块/复杂度/预估工时/依赖）
```

详细模板参见 `references/research/06-spec-template.md`。

---

## 诚实边界

此工具的局限性和适用边界：

1. **服务端逻辑不可逆**：客户端逆向只能获取客户端代码。服务端的核心逻辑（如真实概率计算、反作弊检测、匹配算法）无法通过客户端逆向获得，只能通过抓包观察输入输出来推测。

2. **混淆越强，还原越不完整**：标准压缩混淆可高质量还原；变量名混淆后需大量人工推断语义；VM混淆（自定义虚拟机执行字节码）目前无通用自动化方案，需逐案手动逆向。

3. **WASM/Unity逆向难度跨越式增加**：Unity小游戏的C#代码经IL2CPP编译为WebAssembly，逆向难度远高于JavaScript。本工具对Unity小游戏的分析能力有限。

4. **动态下发内容无法离线获取**：部分游戏的关卡、活动、配置通过服务端实时下发，解包只能获取离线时的默认配置。需配合抓包进行持续数据采集。

5. **法律与伦理风险**：逆向工程可能涉及著作权法和计算机软件保护条例。本工具仅供技术学习和竞品分析参考，不鼓励直接复制他人作品的美术资源、代码或创意。使用者需自行承担法律风险。

6. **数值还原存在偏差**：从代码中提取的数值可能是旧版本配置，游戏可能已通过服务端热更新修改了数值。实际数值应以统计验证为准。

7. **微信平台更新可能使工具失效**：wxapkg的加密方式、小游戏运行时API、平台安全策略都可能随微信版本更新而变化，解包工具需要持续适配。

---

## 内在张力与权衡

- **分析深度 vs 时间成本**：完整的6层逆向可能需要数天。快速预研时应优先Phase 1-2（包体+资源），确认技术可行性后再深入Phase 3-5。
- **自动化 vs 准确性**：工具自动反混淆的结果可能有错误（如变量名推断错误、控制流还原不完整）。关键逻辑需人工审核。
- **客户端数据 vs 服务端数据**：过度依赖客户端逆向可能导致对游戏机制的误判。概率和数值应始终通过统计验证。

---

## 附录：调研来源

调研过程详见 `references/research/` 目录（6个文件）。

### 一手技术来源
- [微信开放文档 - 小游戏引擎适配](https://developers.weixin.qq.com/minigame/dev/guide/game-engine/cocos-laya-egret.html) — 官方引擎适配文档
- [Cocos Creator - 发布到微信小游戏](https://docs.cocos.com/creator/3.8/manual/en/editor/publish/publish-wechatgame.html) — Cocos官方发布指南
- [微信开放文档 - 小游戏商业化激励政策](https://developers.weixin.qq.com/minigame/introduction/commercialization/) — 官方变现文档
- [TexturePacker官方文档](https://www.codeandweb.com/texturepacker/documentation/sprite-sheet-cutter) — 图集打包/拆分

### 开源工具与社区
- [KillWxapkg](https://github.com/Ackites/KillWxapkg) — Go实现的全功能wxapkg反编译工具
- [wux1an/wxapkg](https://github.com/wux1an/wxapkg) — wxapkg扫描+解密+解包
- [wedecode](https://github.com/biggerstar/wedecode) — 全自动化源码还原
- [BlackTrace/pc_wxapkg_decrypt](https://github.com/BlackTrace/pc_wxapkg_decrypt) — PC端wxapkg解密算法参考实现
- [texture-unpacker](https://github.com/pavle-goloskokovic/texture-unpacker) — 图集还原工具
- [0xdevalias - JS逆向笔记](https://gist.github.com/0xdevalias/d8b743efb82c0e9406fc69da0d6c6581) — JavaScript反混淆方法论

### 技术分析文章
- [Kasada - JavaScript Deobfuscation and Reverse Engineering](https://www.kasada.io/javascript-deobfuscation-and-reverse-engineering/) — JS混淆类型与对抗策略
- [腾讯云 - 羊了个羊抓包与响应篡改](https://cloud.tencent.com/developer/article/2189975) — 小游戏抓包实战案例
- [康祖彬 - 微信小程序反编译实战](https://kangzubin.com/wxapp-decompile-1/) — wxapkg格式与解包流程
- [看雪论坛 - 游戏逆向工程](https://bbs.kanxue.com/thread-11623.htm) — 游戏逆向方法论
- [52pojie - 羊了个羊协议分析](https://www.52pojie.cn/thread-1692161-1-1.html) — 协议逆向实战

### 游戏设计文档参考
- [GitBook - How to Write a Game Design Document](https://www.gitbook.com/blog/how-to-write-a-game-design-document) — GDD编写指南
- [Nuclino - Game Design Document Template](https://www.nuclino.com/articles/game-design-document-template) — GDD模板参考

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
