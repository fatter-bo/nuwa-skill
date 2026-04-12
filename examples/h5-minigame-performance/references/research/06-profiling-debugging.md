# 维度六：性能监控与调试

## 1. Chrome DevTools Performance

### Performance 面板核心功能
- **FPS 图表**：动画帧率可视化，目标 60FPS，低于 30FPS 视为卡顿
- **CPU 火焰图**：按时间轴展示 JS 执行栈，定位耗时函数
- **Main 线程**：查看每帧的 Task 分布（Scripting / Rendering / Painting / GC）
- **GPU 线程**：查看 GPU 绘制耗时
- **Network 时间线**：资源加载瀑布图

### H5 游戏性能录制流程
1. 打开 DevTools → Performance 面板
2. 点击录制（或 Ctrl+E）
3. 执行目标游戏操作（进入战斗、切换场景等）
4. 停止录制，分析结果

### 关键分析指标
- **Long Tasks**：超过 50ms 的任务标红，是卡顿源
- **Scripting 占比**：过高说明 JS 逻辑过重
- **Rendering 占比**：过高说明 DOM/Canvas 重排重绘频繁
- **GC 事件**：频繁 GC 说明临时对象过多

来源: [Chrome DevTools - Analyze runtime performance](https://developer.chrome.com/docs/devtools/performance)
来源: [Chrome DevTools - Performance features reference](https://developer.chrome.com/docs/devtools/performance/reference)

## 2. Chrome DevTools Memory

### Heap Snapshot 对比分析
- 操作前取快照 → 执行操作 → 操作后取快照
- Comparison 视图查看新增/删除/保留的对象
- 重点关注：Detached DOM nodes（已从 DOM 移除但仍被 JS 引用）

### Allocation Timeline
- 实时记录内存分配
- 蓝色竖条 = 分配，消失 = 被 GC 回收
- 持续存在的蓝色竖条 = 潜在泄漏

### Memory 面板快捷用法
```
1. 多次执行同一操作（如打开/关闭弹窗 10 次）
2. 手动触发 GC（点击垃圾桶图标）
3. 查看 JS Heap 是否回到初始水平
4. 未回到 → 存在泄漏，取 Snapshot 定位
```

## 3. Cocos Creator Profiler

### 内置调试信息
- 在 Cocos Creator 中开启 Profiler 显示实时统计：
  - **FPS**：当前帧率
  - **DrawCall**：当前 DrawCall 数量
  - **Triangles**：三角面数
  - **Game Logic**：游戏逻辑耗时（ms）
  - **Renderer**：渲染耗时（ms）

### 开启方式
```typescript
// 代码开启
import { profiler } from 'cc';
profiler.showStats(); // 显示
profiler.hideStats(); // 隐藏
```

### Native Performance Profiler（3.8+）
- 更详细的原生层性能分析
- 提供每个系统（Physics、Animation、UI 等）的耗时分解
- 支持自定义标记区间

来源: [Cocos Creator 3.8 - Native Performance Profiler](https://docs.cocos.com/creator/3.8/manual/en/advanced-topics/profiler.html)

## 4. 自定义性能埋点

### Performance API
```typescript
// 标记时间点
performance.mark('scene-load-start');
// ... 加载场景
performance.mark('scene-load-end');

// 计算耗时
performance.measure('scene-load', 'scene-load-start', 'scene-load-end');
const measure = performance.getEntriesByName('scene-load')[0];
console.log(`场景加载耗时: ${measure.duration.toFixed(2)}ms`);
```

### 自定义帧率监控
```typescript
class FPSMonitor {
    private frameCount = 0;
    private lastTime = performance.now();
    private fps = 0;

    update() {
        this.frameCount++;
        const now = performance.now();
        if (now - this.lastTime >= 1000) {
            this.fps = this.frameCount;
            this.frameCount = 0;
            this.lastTime = now;
            // 帧率低于阈值时上报
            if (this.fps < 30) {
                this.reportLowFPS(this.fps);
            }
        }
    }

    private reportLowFPS(fps: number) {
        // 上报到监控平台
        navigator.sendBeacon('/api/perf', JSON.stringify({
            type: 'low_fps',
            fps,
            timestamp: Date.now(),
            scene: director.getScene()?.name,
        }));
    }
}
```

### 关键性能指标埋点体系
```typescript
interface PerfMetrics {
    // 加载阶段
    engineInitTime: number;      // 引擎初始化耗时
    firstSceneLoadTime: number;  // 首场景加载耗时
    totalStartupTime: number;    // 从打开到可交互的总时间

    // 运行阶段
    avgFPS: number;              // 平均帧率
    minFPS: number;              // 最低帧率
    p95FrameTime: number;        // P95 帧耗时
    drawCallCount: number;       // DrawCall 数量
    triangleCount: number;       // 三角面数

    // 内存
    jsHeapUsed: number;          // JS 堆内存使用
    jsHeapTotal: number;         // JS 堆内存总量
    webglMemory: number;         // WebGL 资源内存

    // 网络
    resourceLoadTime: number;    // 资源加载总耗时
    bundleDownloadSize: number;  // 包体下载大小
}
```

### Chrome DevTools Extensibility API
- 可将自定义性能数据注入到 Performance 面板的时间轴中
- 框架作者可为自己的系统创建自定义 Track
- 在 Performance 面板中直观展示游戏特定的性能数据

来源: [Chrome DevTools - Custom performance data with extensibility API](https://developer.chrome.com/docs/devtools/performance/extension)

## 5. 性能监控平台搭建

### 数据采集
- 使用 `navigator.sendBeacon()` 无阻塞上报性能数据
- 采样率控制：非全量上报，按比例采样（如 10%）
- 区分设备等级：高/中/低端设备分别统计

### 告警策略
```
告警规则：
- 平均 FPS < 25 → P1 告警
- 启动时间 > 8s → P2 告警
- 内存使用 > 300MB → P2 告警
- DrawCall > 100（2D 游戏）→ P3 告警
- 崩溃率 > 1% → P0 告警
```

### 设备分级
```typescript
function getDeviceTier(): 'high' | 'mid' | 'low' {
    const canvas = document.createElement('canvas');
    const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');
    if (!gl) return 'low';

    const renderer = gl.getParameter(gl.RENDERER);
    const maxTextureSize = gl.getParameter(gl.MAX_TEXTURE_SIZE);

    if (maxTextureSize >= 8192) return 'high';
    if (maxTextureSize >= 4096) return 'mid';
    return 'low';
}
```
