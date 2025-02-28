#ugui  #框架

UIManager--->跨场景的全局UI管理器
BasePanel--->所有页面的父类
页面配置关系表--->页面预制件的配置路径
![[Pasted image 20250226204244.png]]

UIManager:
维护一个面板名字和面板预制体映射的字典
```
private static Dictionary<string, GameObject> instancesPanel;
```
在OpenPanel时,首先查看映射中有没有,如果没有,就LoadAsset这个面板预制体,并加入这个缓存中.打开时可以做一些Tween动画,也可以调用BasePanel的虚方法向打开的这个面板传递参数
```
public static async UniTask<T> OpenPanel<T>(Ease openAnimatin = Ease.Unset, PanelType panelType = PanelType.全局,...) where T : BasePanel
```

在ClosePanel中,同理.


BasePanel:
```

```