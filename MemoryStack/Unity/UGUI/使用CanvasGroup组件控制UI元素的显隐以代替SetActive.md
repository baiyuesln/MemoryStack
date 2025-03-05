#ugui  #CanvasGroup

使用 CanvasGroup 组件是控制UI元素显隐的更好方式，它提供了更多的控制选项：
- Alpha 透明度渐变
- 是否可交互
- 是否可以接收射线检测
- 不会完全禁用游戏对象

![[Pasted image 20250305164122.png]]

```cs
//隐藏
hudCanvasGroup.DOFade(0, 0.3f);

hudCanvasGroup.interactable = false;

hudCanvasGroup.blocksRaycasts = false;

//显示
hudCanvasGroup.DOFade(1, 0.3f);

hudCanvasGroup.interactable = true;

hudCanvasGroup.blocksRaycasts = true;

```
优势：

1. 更好的性能：不会触发GameObject的激活/禁用事件
2. 更流畅的视觉效果：可以实现平滑的淡入淡出效果
3. 更细粒度的控制：可以单独控制透明度、交互性和射线检测
4. 更符合Unity UI系统的设计理念

注意：需要在Unity编辑器中给相应UI元素添加CanvasGroup组件，并在Inspector中将其赋值给hudCanvasGroup字段。