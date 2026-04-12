# 维度3：代码逆向分析

## JavaScript混淆还原

### 常见混淆类型

| 混淆类型 | 特征 | 难度 | 还原方法 |
|---------|------|------|---------|
| 压缩/Minify | 变量名缩短、空格移除 | 低 | js-beautify自动格式化 |
| 变量名混淆 | 变量名为 `_0x1a2b`、`a`、`b` | 中 | 手动重命名+语义推断 |
| 字符串加密 | 字符串被替换为函数调用 `_0x2f1c('0x1')` | 中 | 执行解密函数，替换结果 |
| 控制流平坦化 | switch-case状态机替代原始逻辑 | 高 | AST分析还原控制流 |
| 死代码注入 | 大量无用代码混入 | 中 | 静态分析+删除不可达代码 |
| VM混淆 | 自定义虚拟机执行字节码 | 极高 | 需逆向VM指令集，无通用方案 |

来源：[Kasada - JavaScript Deobfuscation](https://www.kasada.io/javascript-deobfuscation-and-reverse-engineering/)，[0xdevalias/deobfuscation notes](https://gist.github.com/0xdevalias/d8b743efb82c0e9406fc69da0d6c6581) — 可信度：高

### 还原工具链

| 工具 | 用途 | 链接 |
|------|------|------|
| js-beautify | 代码格式化 | [npm](https://www.npmjs.com/package/js-beautify) |
| de4js | 在线反混淆 | [de4js](https://lelinhtinh.github.io/de4js/) |
| AST Explorer | AST可视化分析 | [astexplorer.net](https://astexplorer.net/) |
| Babel + 自定义插件 | AST级别的代码转换 | [Babel](https://babeljs.io/) |
| Chrome DevTools | 运行时调试 | 内置 |
| obfuscator.io检测 | 识别javascript-obfuscator特征 | [obfuscator.io](https://obfuscator.io/) |

### 实战还原流程

1. **格式化**：`js-beautify` 恢复缩进和换行
2. **字符串解密**：
   - 找到字符串数组（通常在文件顶部，如 `var _0x5a2e = ['atob', 'push', ...]`）
   - 找到解密函数（接受索引，返回解密后字符串）
   - 用Node.js执行解密函数，批量替换
3. **变量重命名**：根据上下文语义手动重命名关键变量
4. **控制流还原**：识别while-switch结构，按case顺序重排逻辑
5. **模块拆分**：识别立即执行函数（IIFE），拆分为独立模块

## 核心游戏逻辑识别

### 状态机识别
搜索关键词：`state`、`status`、`phase`、`FSM`、`switch`、`case`
```javascript
// 典型的游戏状态机模式
switch(this.gameState) {
    case 'INIT': ...
    case 'PLAYING': ...
    case 'PAUSE': ...
    case 'GAMEOVER': ...
}
```

### 关卡数据定位
- 搜索 `level`、`stage`、`map`、`config` 等关键词
- 关卡数据常以JSON数组形式存在，或在 `settings.js`、`config.js` 中
- Cocos Creator项目的关卡数据通常在 `assets/` 下的 `.json` 场景文件中

### 合成树/配方系统
- 搜索 `recipe`、`merge`、`combine`、`craft`、`formula`
- 合成数据通常为 `{input: [id1, id2], output: id3}` 结构

### 出牌/匹配规则
- 搜索 `match`、`check`、`valid`、`rule`、`compare`
- 三消类：搜索方向检测逻辑（上下左右+对角）
- 棋牌类：搜索牌型判断函数

## 网络协议分析

### 抓包工具配置

| 工具 | 适用场景 | 配置要点 |
|------|---------|---------|
| Charles | HTTP/HTTPS/WebSocket | 安装CA证书到手机，开启SSL Proxying |
| mitmproxy | HTTP/HTTPS + 脚本化 | `pip install mitmproxy`，端口8080，手机配代理 |
| Fiddler | Windows环境HTTP | 开启HTTPS解密，信任证书 |
| Wireshark | 底层协议分析 | 适合分析非标准协议 |

来源：[腾讯云-羊了个羊抓包分析](https://cloud.tencent.com/developer/article/2189975)，[TesterHome-mitmproxy游戏协议测试](https://testerhome.com/topics/30487) — 可信度：高

### 抓包流程（以mitmproxy为例）
1. 安装：`pip install mitmproxy`
2. 启动：`mitmdump -p 8080`
3. 手机配置WiFi代理指向电脑IP:8080
4. 访问 `http://mitm.it/` 安装证书
5. 打开小游戏，观察请求

### 协议分析要点

**HTTP接口分析**：
- 关注域名（识别游戏服务器 vs CDN vs 广告 vs 统计）
- 分析请求参数和响应结构
- 识别鉴权方式（token、签名算法）

**WebSocket分析**：
- 连接建立时的握手参数
- 消息格式（JSON/Protobuf/自定义二进制）
- 心跳机制（ping/pong间隔）
- 关键消息类型（登录、房间、游戏状态同步）

### 实战案例：羊了个羊
- 核心接口：`cat-match.easygame2021.com/sheep/v1/game/map_info_new`
- 关卡配置通过 `map_id` 参数获取
- V2版本引入加密参数 `MatchPlayInfo`（混淆JS生成）
- 通过mitmproxy篡改 `map_info_ex` 响应中的 `map_md5`，可将高难度关卡替换为低难度

来源：[52pojie.cn - 羊了个羊协议分析](https://www.52pojie.cn/thread-1692161-1-1.html)，[V2EX讨论](https://www.v2ex.com/t/881121) — 可信度：高（可重现）
