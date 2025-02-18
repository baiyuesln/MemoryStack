#RenderTexture #ugui 

	需求:在不如修仙项目中,我需要吧战斗场面通过相机投影到Canvas里面

**适用场景**：需要 **实时显示 Sprite Renderer 的动态变化**（如角色位移、动画或者小地图）。

#### **步骤**

1. **创建 Render Texture**：
    
    - 菜单栏 `Assets → Create → Render Texture`，命名为 `BackgroundRenderTexture`。
    - 修改 **Render Texture 尺寸**（如 1920x1080）以匹配屏幕分辨率。
2. **设置摄像机渲染到 Render Texture**：
    
    - 创建一个新的 `Camera`（命名为 `BackgroundCamera`），调整其参数：
        
        ```cs
        Clear Flags → Solid Color     // 清除背景
        Culling Mask → 仅勾选渲染目标层级（如 "Background"）
        Target Texture → BackgroundRenderTexture  // 关键步骤
        ```
        
    - 将所有需要显示的 `Sprite Renderer` 放入同一层级（如 "Background"）。
        
3. **在 Canvas 的 Image 上显示 Render Texture**：
    
    - 在 `Canvas` 下添加 `RawImage` 组件（因为 `Image` 仅支持 Sprite）。
    - 将 `RawImage` 的 **Texture** 绑定到 `BackgroundRenderTexture`，**调整**其锚点至全屏。
    
    ```cs
    // 动态绑定 Render Texture
    public class BackgroundController : MonoBehaviour
    {
        public RawImage backgroundImage;
        public RenderTexture renderTexture;
    
        void Start()
        {
            backgroundImage.texture = renderTexture;
        }
    }
    ```

#### **注意**
1. 注意相机的位置是否可以拍摄到具体内容(在指定层级下)
2. 注意RawImage的位置和大小