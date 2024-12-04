
> [!NOTE]
> [[
> ]]> 
> 

#相机 #适配 #分辨率
### 1. **定义：**

- **正交相机**（Orthographic Camera）是一种视野不随物体与相机的距离变化而变化的相机模式。这与透视相机不同，后者的视野会根据物体与相机的距离而发生变化（远离相机的物体会显得更小）。
    
- **`orthographicSize`** 定义了从正交相机的视点向上延伸的垂直视野的大小。它控制了相机可以看到的场景的上下范围。
    

### 2. **详细解释：**

- **`orthographicSize`** 控制了正交相机视野的**垂直**范围。值越大，视野上下可见的区域就越大，物体显示的就越小；值越小，视野上下可见的区域就越小，物体显示的就越大。
    
- 在正交视图中，相机的水平视野（水平可见区域的宽度）与屏幕的纵横比（宽高比）相关。也就是说，`orthographicSize` 主要控制垂直视野大小，而水平视野的大小会根据屏幕的宽高比自动调整。

**水平视野（宽度）** = `orthographicSize * 2 * aspect ratio`

aspect ratio` = (float)Screen.width / Screen.height;

所以size =  水平视野/2/分辨率

不同的分辨率根据统一的水平视野计算不同的正交size

代码如下:

```
void AdjustCamera()

{

// 获取屏幕的宽高比

float screenAspect = (float)Screen.width / Screen.height;

// 计算适配的比例

Camera camera = GetComponent<Camera>();

  

// 4.620853f = Size(5)*2*iphone12分辨率

camera.orthographicSize = 4.620853f/screenAspect/2;

}
```