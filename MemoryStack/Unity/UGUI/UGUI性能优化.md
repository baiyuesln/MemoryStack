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
#### 4.rebuild 过程 (Graphics)
##### 整体流程
Rebuild 过程是重新计算 Unity UI 的 C# Graphic 组件的布局和网格的过程,图形重建是Unity UI系统中用于更新页面元素渲染的核心机制,CanvasUpdateRegistry类管理,通过PerformUpdate方法触发,主要流程包括:
- **触发时机**：每当 Canvas 组件调用 [WillRenderCanvases](http://docs.unity3d.com/ScriptReference/Canvas-willRenderCanvases.html) 事件时，都会调用PerformUpdate方法。此事件每帧调用一次。
	- PerformUpdate 运行一个三步过程：
		- 通过 [ICanvasElement.Rebuild](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html) 方法请求脏布局组件重新构建其布局。
		- 任何已注册的 Clipping 组件（如 Masks）都会被请求剔除任何被剪切的组件。这是通过 ClippingRegistry.Cull 完成的。
		- 要求 Dirty Graphic 组件重新构建其图形元素。
- ​**执行顺序**：分为**布局重建（Layout Rebuild）​**和**图形重建（Graphic Rebuild）​**两部分。对于 Layout 和 Graphic 重建，该过程分为多个部分。布局重建分三个部分（PreLayout、Layout 和 PostLayout）运行，而图形重建分两个部分（PreRender 和 LatePreRender）运行。
##### Layout rebuilds
1. 要重新计算一个或多个 Layout 组件中包含的组件的适当位置（以及可能的大小），必须按适当的层次结构顺序应用 Layouts。在游戏对象层次结构中更靠近根的布局可能会更改可能嵌套在其中的任何布局的位置和大小，因此必须首先计算。
2. 为此，Unity UI 按它们在层次结构中的深度对脏 Layout 组件列表进行排序。层次结构中较高的项（即父变换较少）将移动到列表的前面。
3. 然后，请求 Layout 组件的排序列表以重新构建其布局;这是 Layout 组件控制的 UI 元素的位置和大小实际更改的位置。有关布局如何影响各个元素位置的更多详细信息，请参阅 Unity 手册的 [UI Auto Layout](http://docs.unity3d.com/Manual/UIAutoLayout.html) 部分。

Graphic rebuilds
1. 重新构建图形组件时，Unity UI 会将控制权传递给 [ICanvasElement](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.html) 接口的 [Rebuild](http://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html) 方法 。Graphic 实现了这一点，并在 Rebuild 过程的 PreRender 阶段运行两个不同的重建步骤:
	1. 如果顶点数据被标记为脏数据（例如，当组件的 RectTransform 更改大小时），则会重新构建网格。
		1. - ​**触发条件**：当顶点数据被标记为“脏”（例如`RectTransform`的尺寸变化、颜色属性修改或文本内容更新时）
		2. **实现方式**：通过`Graphic`类的`OnPopulateMesh`方法生成新的网格顶点数据，并传递给`VertexHelper`工具类处理
		3. **注意事项**：若未正确设置顶点颜色（如Alpha为0），会导致UI完全透明
	2. 如果材质数据被标记为脏（例如，当组件的材质或纹理发生更改时），则附加的 Canvas Renderer 的材质将被更新。
		1. - ​**触发条件**：当材质或纹理发生变化（例如更换贴图或调整材质属性）
		2. **实现方式**：调用`SetMaterialDirty()`更新关联的`CanvasRenderer`材质属性，确保渲染器使用正确的材质和纹理
2. 图形重建模型不会以任何特定顺序遍历图形组件列表，并且不需要任何排序作。
#### 5.优化建议
** 触发图形重建的常见操作**
以下操作会导致图形重建：
- **顶点变化**：修改RectTransform的尺寸、颜色属性（如Graphic.color）、文本内容或字体。
- **材质变化**：更换纹理（通过texture属性）、调整材质参数或切换Shader。
- **动态UI交互**：如按钮状态变化（启用/禁用）、遮罩内容更新或动画驱动的属性修改。

**性能优化建议**

图形重建的性能开销主要与**频繁触发重建的组件数量**有关。以下优化策略可参考：

- **减少动态元素**：将静态UI与动态UI分离到不同的Canvas中，避免因局部更新触发全局重建。
- **避免频繁修改属性**：例如减少逐帧修改RectTransform或颜色属性，改用缓存或状态机控制
- **合批优化**：确保相同材质的UI元素相邻排列，减少因材质切换导致的批次中断
- **使用图集**：通过SpriteAtlas合并纹理，减少材质切换和Draw Call数量

布局重建的优化策略:
1. **避免频繁布局计算**  
    动态UI元素（如滚动列表）应尽量使用对象池，减少运行时布局组件的增减操作。
2. **禁用不必要的****Raycast Target**  
    非交互式UI元素（如背景图）关闭射线检测，可降低GraphicRaycaster的遍历开销。
3. **拆分****Canvas****层级**  
    将静态与动态UI分离到不同Canvas中，可限制重建范围。例如：HUD元素使用独立Canvas，避免因其他UI变动触发全局重建。

### 二、Unity UI Profiling Tools

有几种分析工具可用于分析 Unity UI 的性能。关键工具包括：

- Unity Profiler  

- Unity Frame Debugger

- Xcode’s Instruments or Intel VTune

- Xcode’s Frame Debugger or Intel GPA
