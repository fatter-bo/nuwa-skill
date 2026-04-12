# 维度二：首屏加载优化

## 1. 关键渲染路径

### H5 游戏的关键渲染路径
传统网页的关键渲染路径关注 HTML → CSS → DOM → Layout → Paint；H5 游戏的关键路径更关注：
1. HTML 外壳页面加载（极轻量）
2. 引擎运行时 JS/WASM 加载与初始化
3. 首屏必要资源加载（Loading 界面背景、字体、启动场景资源）
4. 游戏引擎初始化 → 首帧渲染

### 优化策略
- 引擎 JS 使用 `<script async>` 或 `<script defer>` 避免阻塞 HTML 解析
- 首屏 CSS 内联到 HTML `<head>` 中（仅 Loading 界面样式）
- 非关键 CSS/JS 延迟加载
- 使用 `<link rel="preload">` 预加载关键资源（引擎 WASM、首屏纹理）

来源: [web.dev - Optimize resource loading](https://web.dev/learn/performance/optimize-resource-loading)
来源: [Preload critical assets](https://web.dev/articles/preload-critical-assets)

## 2. 预加载策略

### 资源优先级分层
```
P0 - 立即需要：Loading界面资源、核心引擎代码
P1 - 首屏需要：启动场景纹理、UI图集、必要音效
P2 - 即将需要：下一关卡资源、常用特效
P3 - 延迟加载：高级关卡、可选音乐、设置界面
```

### Cocos Creator 预加载 API
```typescript
// 预加载 Asset Bundle
assetManager.loadBundle('game-level-1', (err, bundle) => {
    // Bundle 元信息已加载，资源可按需 load
});

// 预加载具体资源（仅下载，不实例化）
resources.preload('textures/hero', SpriteFrame);
resources.preload('prefabs/enemy', Prefab);

// 预加载场景
director.preloadScene('GameScene');
```

### 预测式流加载
- 根据玩家进度预判下一步需要的资源
- 在空闲帧执行预加载，避免与游戏逻辑争抢 CPU
- 利用 `requestIdleCallback` 或引擎的帧调度器分帧加载

## 3. Loading 界面设计

### 纯 HTML/CSS Loading（推荐）
- 在引擎初始化前用原生 HTML/CSS 显示 Loading 界面
- 不依赖引擎，即刻可见，给用户"已启动"的心理暗示
- 加载进度通过 JS 回调更新 DOM 元素

### 进度反馈最佳实践
- 显示真实的加载进度百分比（而非假进度条）
- 将加载过程拆分为有意义的阶段提示（"正在加载资源..."→"正在初始化游戏..."）
- 小技巧：前 90% 用真实进度，最后 10% 用引擎初始化时间补足
- 加入游戏提示/小贴士转移用户等待焦虑

来源: [Unity WebGL Custom Loading Screen](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)

## 4. 资源懒加载

### 按需加载原则
- 非首屏资源一律不在启动时加载
- 场景切换时加载目标场景资源，卸载上一场景资源
- 弹窗/二级界面在首次打开时加载

### 实现模式
```typescript
// 懒加载 UI 面板
async openSettingsPanel() {
    if (!this._settingsPanel) {
        const prefab = await resources.loadAsync<Prefab>('ui/SettingsPanel');
        this._settingsPanel = instantiate(prefab);
    }
    this.node.addChild(this._settingsPanel);
}
```

### 分帧加载大量资源
```typescript
// 避免一帧内实例化大量节点导致卡顿
async loadEnemies(prefab: Prefab, count: number) {
    for (let i = 0; i < count; i++) {
        const enemy = instantiate(prefab);
        this.node.addChild(enemy);
        if (i % 5 === 0) {
            await this.nextFrame(); // 每5个等一帧
        }
    }
}
```

来源: [MDN - Lazy loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading)

## 5. CDN 与缓存策略

### CDN 部署
- 静态资源（纹理、音频、配置）部署到 CDN
- 引擎代码和 WASM 文件也应通过 CDN 分发
- 使用就近节点加速资源下载

### 缓存策略
| 资源类型 | 缓存策略 | Cache-Control |
|----------|---------|---------------|
| 引擎 JS/WASM | 强缓存 + 文件名 Hash | max-age=31536000, immutable |
| 纹理图集 | 强缓存 + Hash | max-age=31536000, immutable |
| 配置文件 | 协商缓存 | no-cache（每次校验 ETag） |
| HTML 入口 | 不缓存 | no-store |

### 微信小游戏缓存
- 利用 `wx.getFileSystemManager()` 缓存已下载的远程资源
- 首次加载从 CDN 下载，后续从本地文件系统读取
- 设置缓存上限和 LRU 淘汰策略，避免存储空间溢出

### Service Worker 缓存（H5）
- 注册 Service Worker 实现离线缓存
- 使用 Cache-First 策略加速重复访问
- 版本更新时通过新 SW 刷新缓存

### Cache Priming
- 预热 CDN 缓存：在用户访问前将关键资源推送到 CDN 边缘节点
- 避免首批用户遭遇 CDN 回源延迟

来源: [Cache Priming - Contentstack](https://www.contentstack.com/docs/developers/launch/cache-priming)
来源: [Frontend Performance Checklist 2025](https://crystallize.com/blog/frontend-performance-checklist)
