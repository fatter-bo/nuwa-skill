# 维度四：内存管理

## 1. 纹理内存估算

### 计算公式
```
纹理内存 = 宽 × 高 × 每像素字节数 × Mipmap系数
```

### 每像素字节数参考
| 格式 | 每像素字节数（bpp） | 说明 |
|------|-------------------|------|
| RGBA8888 | 4 bytes (32bpp) | 未压缩，最高质量 |
| RGB888 | 3 bytes (24bpp) | 无 Alpha |
| RGBA4444 | 2 bytes (16bpp) | 低质量，用于不重要的纹理 |
| ASTC 4x4 | 1 byte (8bpp) | GPU 压缩，高质量 |
| ASTC 6x6 | ~0.44 bytes (3.56bpp) | GPU 压缩，中等质量 |
| ASTC 8x8 | 0.25 bytes (2bpp) | GPU 压缩，适合背景 |
| ETC2 RGBA | 1 byte (8bpp) | GPU 压缩，Android 兼容 |

### Mipmap 系数
- 开启 Mipmap 时乘以 1.33（额外增加 33% 内存）
- 2D 游戏 UI 通常不需要 Mipmap
- 3D 场景纹理建议开启 Mipmap 提升远景渲染质量

### 估算示例
一张 1024×1024 RGBA8888 纹理：
- 未压缩：1024 × 1024 × 4 = **4MB**
- ASTC 4x4：1024 × 1024 × 1 = **1MB**
- ASTC 8x8：1024 × 1024 × 0.25 = **256KB**

来源: [Cocos - How One Operation Could Reduce Game Memory by 50%](https://www.cocos.com/en/post/how-one-operation-could-reduce-game-memory-by-50)

## 2. 资源引用计数与释放策略

### 引用计数机制
- Cocos Creator 的 AssetManager 对加载的资源维护引用计数
- `addRef()` 增加引用，`decRef()` 减少引用
- 引用计数归零时资源可被释放

### 释放策略
```typescript
// 手动释放单个资源
assetManager.releaseAsset(texture);

// 释放 Asset Bundle 中的所有资源
bundle.releaseAll();

// 场景切换时自动释放
// 在 director.loadScene 的回调中执行旧场景资源释放
director.loadScene('NewScene', () => {
    assetManager.releaseUnusedAssets();
});
```

### 最佳实践
- 场景切换时释放上一场景的专属资源
- 常驻资源（全局 UI、通用音效）标记为常驻，不随场景释放
- 定期调用 `releaseUnusedAssets()` 清理引用计数为 0 的资源
- 对于大量临时资源（如战斗特效），使用 Asset Bundle 统一管理和释放

## 3. 内存泄漏检测

### 常见泄漏场景
1. **事件监听器未移除**：`node.on()` 注册后节点销毁前未 `off()`
2. **定时器未清除**：`setInterval`/`setTimeout` 引用的闭包持有节点引用
3. **纹理未释放**：动态创建的纹理（RenderTexture、截图）未手动 destroy
4. **闭包引用**：回调函数闭包持有大对象引用，阻止 GC 回收
5. **全局变量/单例**：缓存了节点或资源引用但未在适当时机清除

### 检测工具

**Chrome DevTools Memory**
- Heap Snapshot：对比两个时间点的快照，查找增长的对象
- Allocation Timeline：实时追踪内存分配
- 关键操作：执行一段游戏流程 → 回到初始状态 → 取快照 → 检查"未释放但应释放"的对象

**WebGL-Memory**
- 专门追踪 WebGL 资源（纹理、Buffer、Renderbuffer）的内存使用
- 提供每个资源的大小和创建堆栈
- 安装：在 WebGL 初始化前引入 `webgl-memory.js`
- 使用：`const ext = gl.getExtension('GMAN_webgl_memory'); ext.getMemoryInfo();`

来源: [WebGL-Memory GitHub](https://github.com/greggman/webgl-memory)
来源: [WebGL-Memory npm](https://www.npmjs.com/package/webgl-memory)

### 自动化泄漏检测流程
```
1. 记录初始内存快照 M0
2. 执行目标操作（进入/退出场景、打开/关闭面板）N 次
3. 强制 GC（Chrome DevTools 中点击垃圾桶图标）
4. 记录当前内存快照 M1
5. 对比 M0 和 M1：
   - 若 M1 - M0 线性增长 → 存在泄漏
   - 检查 Delta 中新增的对象类型和数量
```

## 4. WebGL 内存限制

### 浏览器内存预算
- 桌面浏览器：单 Tab 通常可用 1-4GB（取决于系统内存和浏览器）
- 移动浏览器：单 Tab 通常仅 256MB-1GB
- iOS Safari 限制最严格，超限直接杀 Tab 无警告
- Android Chrome 相对宽松，但低端设备内存紧张

### 内存预算规划
```
推荐内存预算：
├── 引擎运行时：       20-50MB
├── 纹理内存：         50-150MB（最大消耗者）
├── 顶点/索引缓冲：   10-30MB
├── 音频缓冲：         10-30MB
├── 脚本堆内存：       20-50MB
└── 总计目标：         ≤ 256MB（移动端安全线）
                       ≤ 512MB（桌面端安全线）
```

### Unity WebGL 特殊注意
- WASM 堆内存默认自动增长，但增长可能失败导致崩溃
- 建议设置固定内存大小（256-512MB）并做好内存预算
- `texImage2D` 可能需要双倍内存（相比 `texStorage2D`）
- 避免频繁的大纹理上传/卸载，会产生内存碎片

来源: [Unity - Memory in Unity WebGL](https://docs.unity3d.com/2021.3/Documentation//Manual/webgl-memory.html)
来源: [Unity - Memory in Unity Web](https://docs.unity3d.com/6000.3/Documentation/Manual/webgl-memory.html)

### 内存优化策略
- 纹理使用 GPU 压缩格式（ASTC/ETC2），内存消耗降低 4-8 倍
- 不使用的纹理及时释放，不要"以备后用"
- 使用纹理图集减少纹理数量（同时减少 DrawCall）
- 音频使用流式加载（Streaming）避免一次性全部载入内存
- 监控内存水位，接近预算上限时主动释放低优先级资源
