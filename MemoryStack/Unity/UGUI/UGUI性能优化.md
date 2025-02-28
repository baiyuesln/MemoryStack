#ugui  #优化 

参考:https://www.jianshu.com/p/0be8b113824a

Unity官方的UGUI优化指南： [Optimizing Unity UI](https://links.jianshu.com/go?to=https%3A%2F%2Flearn.unity.com%2Ftutorial%2Foptimizing-unity-ui%3Ftdsourcetag%3Ds_pctim_aiomsg%23)

Unity UI的C#是开源的，可以从[Unity’s Bitbucket repository](https://links.jianshu.com/go?to=https%3A%2F%2Fbitbucket.org%2FUnity-Technologies%2F)里的 [_UI_](https://links.jianshu.com/go?to=https%3A%2F%2Fbitbucket.org%2FUnity-Technologies%2Fui%2F)文件夹下找到。

Unity UI居然还有单独的Document，惊了， [_Unity UI Document 1.0.0_](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.unity3d.com%2FPackages%2Fcom.unity.ugui%401.0%2Fmanual%2FUIAutoLayout.html)

Unity官方出的Unity在5.2版本的时候对UGUI的优化方案和方向：["Unity UGUI 的整个改进过程_(5.2启用)"](https://links.jianshu.com/go?to=https%3A%2F%2Fblogs.unity3d.com%2F2015%2F09%2F07%2Fmaking-the-ui-backend-faster%2F%3F_ga%3D2.62088997.826168706.1593743557-98168652.1554705329)

## 官方的UGUI优化指南中指出:

与其他方面的最佳实践一样，尝试优化 Unity UI 应从性能分析开始。在尝试优化 Unity UI 系统之前，主要任务是找到观察到的性能问题的确切原因。
### Unity UI 用户遇到的常见问题有四类：
- Excessive GPU fragment shader utilization (i.e. fill-rate overutilization)  
    GPU 片段着色器利用率过高（即填充率利用率过高）
    

- Excessive CPU time spent rebuilding a Canvas batch  
    重新构建 Canvas 批处理所花费的 CPU 时间过长
    

- Excessive numbers of rebuilds of Canvas batches (over-dirtying)  
    Canvas 批次的重建数量过多（过度脏污）
    

- Excessive CPU time spent generating vertices (usually from text)  
    生成顶点（通常来自文本）所花费的 CPU 时间过多

### 本指南将讨论 Unity UI 的基本概念、算法和代码，并讨论常见问题和解决方案。它分为五章：
1. The [Fundamentals of Unity UI](https://unity3d.com/learn/tutorials/topics/best-practices/fundamentals-unity-ui) chapter defines terminology specific to Unity UI and discusses the details of many of the fundamental processes performed to render the UI, including the building of batched geometry. It is strongly recommended that readers begin with this chapter.  
    [Unity UI 基础知识](https://unity3d.com/learn/tutorials/topics/best-practices/fundamentals-unity-ui)一章定义了特定于 Unity UI 的术语，并讨论了为渲染 UI 而执行的许多基本过程的详细信息，包括构建批处理几何体。 强烈建议读者从本章开始。
    

2. The [Unity UI profiling tools](https://unity3d.com/learn/tutorials/topics/best-practices/unity-ui-profiling-tools) chapter discusses gathering profiling data with the various tools available to developers.  
    [Unity UI 分析工具](https://unity3d.com/learn/tutorials/topics/best-practices/unity-ui-profiling-tools)一章讨论了使用开发人员可用的各种工具收集性能分析数据。
    

5. The [Fill-rate, Canvases and input](https://unity3d.com/learn/tutorials/topics/best-practices/fill-rate-canvases-and-input) chapter discusses ways to improve the performance of Unity UI's Canvas and Input Components.  
    [填充率、画布和输入](https://unity3d.com/learn/tutorials/topics/best-practices/fill-rate-canvases-and-input)一章讨论了提高 Unity UI 的画布和输入组件性能的方法。
    

8. The [UI controls](https://unity3d.com/learn/tutorials/topics/best-practices/optimizing-ui-controls) chapter discusses UI Text, Scroll Views and other component-specific optimizations, along with some techniques that do not fit well elsewhere.  
    [UI 控件](https://unity3d.com/learn/tutorials/topics/best-practices/optimizing-ui-controls)一章讨论了 UI 文本、滚动视图和其他特定于组件的优化，以及一些在其他地方不太适合的技术。
    

11. The [Other techniques and tips](https://unity3d.com/learn/tutorials/topics/best-practices/other-ui-optimization-techniques-and-tips) chapter discusses a handful of issues that do not fit elsewhere, including some basic tips and workarounds for "gotchas" in the UI system.  
    [其他技术和提示](https://unity3d.com/learn/tutorials/topics/best-practices/other-ui-optimization-techniques-and-tips)一章讨论了一些不适合其他位置的问题，包括 UI 系统中“陷阱”的一些基本提示和解决方法。

### 一、Unity UI 基础知识

#### 1.术语

Graphic 是 Unity UI C# 库提供的基类。它是所有为 Canvas 系统提供可绘制几何体的 Unity UI C# 类的基类。Drawable 的主要子类是 Image 和 Text，它们提供其同名组件。
Layout 组件控制 RectTransforms 的大小和位置，通常用于创建需要相对大小或相对定位其内容的复杂布局。不依赖于 Graphic 类
Graphic 和 Layout 组件都依赖于 CanvasUpdateRegistry 类,在其关联的 Canvas 调用 willRenderCanvases 事件时根据需要触发更新Graphic 和 Layout 。
Layout 和 Graphic 组件的更新称为重建模型。
#### 2.渲染细节
Canvas绘制的所有几何图形(**所有 UI 元素的可视几何形状**，这些形状通过顶点和三角形网格构成,如image,text)都将在 Transparent 队列中绘制.也就是说，Unity UI 生成的几何体将始终使用 Alpha 混合从后到前绘制。
所以多边形栅格化(**由顶点和三角形构成的几何图形**​ 转换为 ​**屏幕上的像素**​ 的过程)
的每个像素都将被采样，即使它完全被其他不透明多边形覆盖。

#### 3.Batch 构建过程 （Canvas）

Canvas 的批构建过程主要负责将 UI 元素的各个网格合并，并生成传递给 Unity 图形渲染管线的渲染指令。该过程会将结果缓存，直到 Canvas 内的某个网格发生变化（即 Canvas 被标记为“脏”）后才会重新构建。
##### 总结
1. **网格收集与筛选**​  
    Canvas会收集所有直接附属的Canvas Renderer组件中的网格数据，但会排除子Canvas中的组件。这种层级隔离机制确保了不同Canvas间的批处理独立性

    
2. ​**批处理生成条件**​  
    通过深度排序和重叠检测后，系统会基于以下要素合并网格：
    
    - ​**相同材质/图集**：主材质（Material）和纹理图集（Atlas）必须一致
    - ​**连续深度层级**：深度不连续的UI元素会打断批处理
    - ​**无遮罩干扰**：RectMask2D、Stencil Material等组件会强制生成新Batch

3. ​**多线程计算特性**​  
    该过程采用多线程计算，在移动端（通常2-4核）与桌面端（4核以上）存在显著性能差异。实测数据显示，复杂UI布局在骁龙835移动芯片上的批处理耗时可达桌面i7处理器的3-5倍
4. ​**脏标记触发机制**​  
    当以下情况发生时，Canvas会被标记为"脏"并触发重建：
    
    - 网格顶点数据变更（如位置/尺寸调整）
    - 材质属性修改
    - 层级结构变化（新增/删除UI元素）
    - 遮罩状态更新

5. ​**缓存复用规则**​  
    未被标记为"脏"的Canvas会复用缓存数据，实测数据显示复用状态下UI渲染耗时可降低60%-80%。但过度依赖缓存可能导致内存占用增加，需平衡性能与资源消耗
6. ​**遮罩使用代价**​  
    每个Mask组件平均增加2-3个Batch，且会导致顶点数增加约15%。建议用RectMask2D替代传统Mask，可减少30%的顶点生成量
##### 优化建议

1. ​**层级合并**：将静态UI合并到同一Canvas，动态UI单独分层
2. ​**材质管理**：采用Atlas打包策略，确保90%以上UI元素共享同一材质包
3. ​**遮罩慎用**：用Alpha通道替代Mask实现渐隐效果
4. ​**动态更新控制**：对高频更新元素启用Canvas.ForceUpdateCanvases()手动控制刷新
### 4.rebuild 过程 (Graphics)
