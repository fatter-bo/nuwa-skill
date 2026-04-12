# 维度1：Egret技术栈全景

## 调研来源
- [egret-labs/egret-core GitHub](https://github.com/egret-labs/egret-core)
- [Egret Engine官方文档](https://docs.egret.com/engine)
- [DragonBones官方文档](https://docs.egret.com/dragonbones/en)
- [Egret Core - JavaScripting](https://www.javascripting.com/view/egret-core)

## 核心发现

### 引擎定位
Egret Engine是白鹭科技推出的开源HTML5游戏引擎，使用TypeScript开发，主要面向移动端H5游戏和微信小游戏。项目在GitHub上获得4000+ Star，最新版本为5.4.1，目前处于维护模式。

### 版本演进路线
| 版本 | 关键特性 |
|------|---------|
| 2.x | 基础2D渲染（Canvas/WebGL）、事件系统、显示列表 |
| 3.x | 引入EUI组件系统、RES资源管理器重构、性能优化 |
| 4.x | DragonBones深度集成、脏矩形渲染优化、WebGL默认渲染 |
| 5.0+ | WebAssembly渲染核心、引擎核心C++化(.wasm)、TypeScript API层不变 |
| 5.4.1 | 最终稳定版本，修复和维护性更新 |

### 核心模块
1. **egret核心** — 显示列表、事件系统、2D渲染（Canvas/DOM/WebGL三模式无缝切换）
2. **EUI** — 基于EXML的UI组件框架（类似Flex/MXML），皮肤机制、数据绑定、布局系统
3. **RES** — 资源管理器，基于default.res.json配置的资源组加载机制
4. **DragonBones** — 骨骼动画系统，支持JSON和二进制格式导出
5. **Tween** — 缓动动画库
6. **game** — 游戏生命周期管理（帧率控制、暂停恢复）
7. **socket** — WebSocket网络通信
8. **egret-wasm** — 5.0+的WebAssembly渲染后端

### 渲染体系
- 三种渲染模式：Canvas、DOM、WebGL
- 脏矩形技术：只重绘变化区域提升性能
- 5.0+引入WebAssembly，引擎核心逻辑编译为.wasm，JS/TS API层保持不变

### TypeScript开发模式
- 项目全程使用TypeScript开发
- API设计高度借鉴ActionScript3（AS3），对Flash开发者友好
- 编译工具链：egret命令行工具（egret create / egret build / egret startserver）
- 支持平台：iOS 8.0+、Android 4.0+、主流桌面浏览器

### DragonBones骨骼动画
- 开源免费的2D骨骼动画方案
- 支持JSON数据格式和二进制格式(.dbbin)
- 通过EgretFactory解析数据并创建骨骼实例
- 支持导出到多个引擎运行时（Egret/Cocos/Unity/Phaser等）

### EgretWing IDE
- 基于VS Code的定制IDE
- 集成EXML可视化编辑器
- 项目创建向导、调试器、性能分析器
- 已停止更新，现有项目可使用VS Code+插件替代

### 当前状态
- 引擎处于维护模式，无重大功能更新
- 白鹭科技战略转向，但存量项目仍大量运行
- 社区活跃度下降，新项目建议使用Cocos Creator或其他引擎
