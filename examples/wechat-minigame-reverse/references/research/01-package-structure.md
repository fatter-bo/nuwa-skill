# 维度1：小游戏包体结构

## wxapkg 文件格式

### 二进制结构
wxapkg 是微信自定义的二进制打包格式（非标准ZIP），结构如下：

| 字段 | 大小 | 说明 |
|------|------|------|
| firstMark | 1 byte | Magic Number = 0xBE |
| reserved | 4 bytes | 保留字段 |
| indexLength | 4 bytes | 文件索引段长度 |
| bodyLength | 4 bytes | 数据段长度 |
| lastMark | 1 byte | 尾部标记 = 0xED |
| fileCount | 4 bytes | 文件数量 |

索引段包含每个文件的：文件名长度、文件名、数据偏移量、数据长度。数据段紧跟其后，存放文件实际内容（明文）。

### PC端加密方案（V1MMWX）
PC版微信对wxapkg采用 AES + XOR 双重加密：

1. **AES加密**（前1023字节）：
   - Key: PBKDF2(pass=小程序AppID, salt="saltiest", iterations=1000, hash=SHA1, keylen=32)
   - IV: 固定值 "the iv: 16 bytes"
   - 对原始包前1023字节进行AES加密

2. **XOR加密**（1023字节后的数据）：
   - XOR Key = AppID倒数第2个字符的ASCII值
   - 若AppID长度<2，则XOR Key = 0x66
   - 逐字节异或

3. **加密标识**：文件头为 "V1MMWX"

来源：[BlackTrace/pc_wxapkg_decrypt](https://github.com/BlackTrace/pc_wxapkg_decrypt)，[renblog/pc_wxapkg_decrypt_python](https://github.com/renblog/pc_wxapkg_decrypt_python) — 可信度：高（开源实现，可验证）

## 解包后目录结构

### 小程序标准结构
```
├── app-service.js      # 所有JS文件汇总（已混淆）
├── app-config.json     # app.json + 页面配置合并
├── page-frame.html     # 所有页面wxml + app.wxss汇总
├── pages/              # 各页面的样式html
└── resources/          # 图片、音频等静态资源
```

### 小游戏标准结构
```
├── game.js             # 入口文件
├── game.json           # 游戏配置
├── project.config.json # 项目配置
└── [引擎相关文件]
```

## 主流引擎打包特征识别

### Cocos Creator
```
├── game.js              # 入口，调用ccRequire
├── ccRequire.js         # Cocos模块加载器
├── adapter-min.js       # 微信适配器
├── main.js              # 主逻辑
├── src/
│   └── settings.js      # 游戏配置（分辨率、场景列表等）
├── cocos/
│   ├── cocos2d-js-min.js  # Cocos引擎（压缩）
│   ├── plugin.json
│   └── signature.json
├── assets/
│   ├── internal/        # 引擎内置资源
│   └── [场景名]/        # 场景资源包
└── subpackages/         # 分包目录（可选）
```
**识别关键词**：`ccRequire`、`cocos2d-js`、`cc.game`、`cc.director`

### LayaAir
```
├── game.js              # 入口
├── laya.js / laya.wxmini.js  # Laya引擎
├── index.js             # 游戏主逻辑
├── fileconfig.json      # 资源配置
└── res/                 # 资源目录
```
**识别关键词**：`Laya`、`laya.wxmini`、`Laya.init`、`Laya3D`

### Egret（白鹭）
```
├── game.js              # 入口
├── egret.wxgame.js      # Egret适配层
├── manifest.js          # 模块清单
├── main.js              # 主入口
└── resource/            # 资源目录
```
**识别关键词**：`egret.`、`eui.`、`egret.wxgame`

### Unity小游戏（通过转换工具）
```
├── game.js              # 入口
├── unity-namespace.js   # Unity命名空间
├── wasmcode/            # WebAssembly代码
├── webgl/               # WebGL渲染相关
└── StreamingAssets/     # Unity资源
```
**识别关键词**：`UnityLoader`、`.wasm`文件、`StreamingAssets`

## game.js 入口分析要点

1. **初始化流程**：game.js是小游戏启动入口，通常包含Canvas创建、引擎初始化、首场景加载
2. **适配器加载**：各引擎通过Adapter层模拟BOM/DOM API（如`wx.createCanvas()`替代`document.createElement('canvas')`）
3. **分包加载**：大型游戏通过`wx.loadSubpackage()`实现分包按需加载
4. **插件使用**：部分游戏使用微信引擎插件（如Cocos引擎插件），减小包体

## 主要解包工具

| 工具 | 语言 | 特点 | 链接 |
|------|------|------|------|
| wux1an/wxapkg | Go | 扫描+解密+解包一体 | [GitHub](https://github.com/wux1an/wxapkg) |
| KillWxapkg | Go | 支持Hook+动态调试+重打包 | [GitHub](https://github.com/Ackites/KillWxapkg) |
| wedecode | Node.js | 全自动源码还原，跨平台 | [GitHub](https://github.com/biggerstar/wedecode) |
| unveilr | Node.js | 经典反编译工具 | [GitHub](https://github.com/BugFans/unveilr) |
| wxappUnpacker | Node.js | 支持分包，历史最久 | [GitHub](https://github.com/luyang668899/wxappUnpacker) |
| mp-wxapkg-unpacker | Node.js | 支持PC端加密包 | [GitHub](https://github.com/halo951/mp-wxapkg-unpacker) |

可信度评估：以上均为开源项目，代码可审计，社区活跃度中-高。
