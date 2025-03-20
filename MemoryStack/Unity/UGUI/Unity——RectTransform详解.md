#ugui 

	https://www.jianshu.com/p/4592bf809c8b
### 1.Anchors 锚点

Unity 中的UI元素有严格的父子关系,子位置根据父物体的变化而变化,而子物体和父物体之间联系的桥梁就是Anchor. 在RectTransform面板一个调整锚点的值.

为什么一个点会由四个值来确定呢，其实Anchors准确来说是由两个点确定的。他们就是AnchorMin以及AnchorMax

![[Pasted image 20250319113402.png]]

之所以anchorMax和anchorMin的值是小数，是因为其表征的是该点所在位置占父物体大小的比例，也就是图中**黑色画线部分占灰色物体总长度和总宽度的比例**，所以在示意图的情况下  
二者的值为：anchorMax(0.5,0.5),anchorMin(0,0) _(默认左下角为0，0点，右上角为1，1点)

为了方便后续描述，我把Anchor分为两种情况

- 当anchorMax与anchorMin相等时，Anchor呈现为一个点，我称之为`锚点`
- 当anchorMax与anchorMin不相等时，Anchor呈现为一个框，我称之为`锚框`

![[Pasted image 20250319113525.png]]

### 2绝对与相对布局

想要清晰的理解Recttransform的各个属性，个人认为首先需要建立的第一个概念就是`绝对布局`以及`相对布局`这两个概念。

#### 2.1绝对布局

所谓绝对布局,就是出现锚点的情况,此时的rectTransform面板中的属性变成了
![[Pasted image 20250319114107.png]]

PosX,PosY,PosZ,Width,Height，这五个属性，首先说说`Width`和`Height`，在绝对布局的情况下无论分辨率是多少，父物体多大，该UI元素的大小是恒定的，如下图所示

![[Pasted image 20250319114141.png]]

![[Pasted image 20250319114255.png]]

而剩下的`PosX`,`PosY`,`PosZ`表征的就是Pivot _(第三部分有关于Pivot的讲解)_ 到锚点的距离

![[Pasted image 20250319114616.png]]

![[Pasted image 20250319114628.png]]

**所以如果使用了绝对布局，在采用不同分辨率的时候，该元素的大小恒定，可能就会出现在高分辨率情况下元素太小或者低分辨率情况下元素比屏幕大的情况。**

#### 2.2相对布局

所谓相对布局，就是出现锚框的情况。在这种情况下UI元素的四个角，距离四个对应的锚点的距离是不变的，在这种情况下RectTransform的属性又变为了`Left`,`Top`,`Right`,`Bottom`,`PosZ`，其中的`PosZ`表征的是该元素到父物体在Z轴上的偏移，利用这个值可以调整UI元素的显示顺序，不过我用的不多，这里不作太多讨论。剩下的四个值应该很好理解了，就是UI元素的每一条边距离父物体的每一条边的距离。  
在示意图的情况下，我设定了红色图片（子物体）距离灰色图片（父物体）的每一条边的距离都是200个单位。

![[Pasted image 20250319133817.png]]

![[Pasted image 20250319133832.png]]

### 3.Pivot

Pivot中心点，就是该UI元素旋转缩放的中心点，左下角为(0,0)右上角为(1,1)

所以之前在`绝对布局`的情况下，`PosX`和`PosY`的值就是Pivot到锚点的值

### 4.Offset

首先说说OffsetMax，其实OffsetMin也是同理。接下来会主要解释两个问题

> OffsetMax的值是怎么计算得出来的呢？OffsetMax又有什么用呢？

- 其实没有那么神秘，这个值就是UI元素的右上角的坐标，减去AnchorMax的值，得到一个从AnchorMax指向元素右上角的向量(vector2类型)，各位看官可以自行测试一下。

![[Pasted image 20250319140726.png]]

- 那么这个值有什么用呢，因为这个值是一个可读可写的属性，所以在`锚框`的情况下我们可以在代码里面动态的去调整UI元素相对边界的距离，其次更重要的是，利用这这两个值就可以计算出sizeDelta的值了！

### 5.sizeDelta

以前对这个属性是真的一脸懵逼，网上很多教程说这个值可以设置UI元素的大小，但是真的有时候好用，有时候有不好用，真的一头雾水，官方文档说的也是很笼统，但是现在搞清楚了其中的联系以后，就觉得清晰了不少了。

> **其实sizeDelta的值就是OffsetMax-OffsetMin的值**

所以就会出现有时候sizeDelta得到的是UI元素的大小，有时候又不是的情况，下面就复现一下这两种情况

#### 5.1锚点情况下的sizeDelta

![[Pasted image 20250319141142.png]]

在锚点情况下，offsetMax和Min的起点相同，根据向量相减的三角形法则（不记得是不是这样说得了哈哈哈），可以得到一个新的向量，这个新的向量的X和Y的大小正好UI元素的宽和高相等，**所以在这个时候去设置sizeDelta的值，可以直接调整UI元素的大小**

#### 5.2锚框情况下的sizeDelta

![[Pasted image 20250319141514.png]]

在锚框的情况下，offstMax减去Min，得到的将不再是UI元素的大小，而是一个新的奇怪的向量，这个向量代表的物理意义是，**sizeDelta.x值就是锚框的宽度与UI元素的宽度的差值，sizeDelta.y的值就是锚框的的高度与UI元素的高度的差值**

> **所以这个属性之所以叫做sizeDelta，是因为在锚点情况下其表征的是size（大小），在锚框的情况下其表征的是Delta（差值）**

那么我们在锚框的情况下要怎么样才能获得元素的大小呢？这个时候就可以用到rect属性了。

### 6.rect

rect中的属性，不与UI元素所在的位置有关，只和其自身属性相关，所以其中的`rect.width`和`rect.height`属性就可以让我们在任何情况下取得元素的大小，而`rect.x`和`rect.y`如图所示，表示的是以Pivot为原点，UI元素左下角的坐标，可以看到图中Pivot是在UI元素的正中间，所以左下角的坐标就刚好是`(-100,-100)`


![[Pasted image 20250319141720.png]]

但是有一个问题，rect属性是一个只读的属性，如果我们想要设置UI元素的大小的话，这好像又不适用了，所以RectTransform还提供了几个非常有用的方法。

### 7.anchoredPosition

通过直接设置anchoredPosition的值可以改变UI元素的位置，但也是要分`锚点`和`锚框`的情况

> 在使用`锚点`的情况下，anchoredPosition表征的是元素Pivot到Anchor的距离

```cs
 private RectTransform rectTransform;
// Use this for initialization
void Start()
{
	rectTransform = GetComponent<RectTransform>();
	rectTransform.anchoredPosition = new Vector2(0, -100);
}
```
![[Pasted image 20250319141909.png]]

在使用`锚框`的情况下，anchoredPosition表征的是元素Pivot到锚框中心点的距离
![[Pasted image 20250319141941.png]]

### 8.RectTransform类中一些方法的介绍

#### 8.1 SetSizeWithCurrentAnchors(Animations.Axis axis, float size)

这个方法无论在`绝对布局`还是`相对布局`的情况下，都可以通过直接设置rect中的`width`和`height`值来改变UI元素的大小。

```cs
private RectTransform rectTransform;
    // Use this for initialization
    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
	rectTransform.SetSizeWithCurrentAnchors(RectTransform.Axis.Horizontal, 100);
        rectTransform.SetSizeWithCurrentAnchors(RectTransform.Axis.Vertical, 100);    
    }
```

#### 8.2 SetInsetAndSizeFromParentEdge([RectTransform.Edge](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FRectTransform.Edge.html) edge, float inset, float size)

这个方法就比较冷门了可能，不过还是挺强大的。调用这个方法，可以根据父物体的Edge（某一边）去布局。其中第一个参数就是用于确定基准的边，第二个参数是UI元素的该边界与父物体该边界的距离，第三个元素是设定选定轴上UI元素的大小，可能说起来有点复杂，但是我上两张图相信各位就可以秒懂了。

首先以右边界为基准

```cs
  private RectTransform rectTransform;
    // Use this for initialization
    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        rectTransform.SetInsetAndSizeFromParentEdge(RectTransform.Edge.Right, 200, 400);
        //这种情况下我选定父物体的右边界为基准，结果如下图
    }
```

![[Pasted image 20250319142720.png]]

然后以下边界为基准

```cs
 private RectTransform rectTransform;
    // Use this for initialization
    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        //rectTransform.SetInsetAndSizeFromParentEdge(RectTransform.Edge.Right, 200, 400);
        rectTransform.SetInsetAndSizeFromParentEdge(RectTransform.Edge.Bottom, 200, 400);
    }
```
![[Pasted image 20250319142750.png]]

在使用这个方法的时候要注意锚点也会改变，改变的规则为

- 以左边界为基准时，`anchorMin`和`anchorMax` 的`y`不变`x`变为0.
- 以右边界为基准时，`anchorMin`和`anchorMax` 的`y`不变`x`变为1.
- 以上边界为基准时，`anchorMin`和`anchorMax` 的`x`不变`y`变为1.
- 以下边界为基准时，`anchorMin`和`anchorMax` 的`x`不变`y`变为0.

#### 8.3 GetWorldCorners(Vector3[] fourCornersArray)

使用这个方法，可以取得UI元素四个角的世界坐标，具体使用方法，先建立一个长度为4的vector3数组，然后传进这个方法，调用一次后，数组被赋值，里面的四个元素分别是UI的`左下角` ，`左上角`，`右上角`，`右下角`。

```cs
  private RectTransform rectTransform;
    // Use this for initialization
    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        Vector3[] corners = new Vector3[4];
        rectTransform.GetWorldCorners(corners);
        foreach (Vector3 corner in corners)
            Debug.LogWarning(corner);
    }
```

![[Pasted image 20250319142930.png]]