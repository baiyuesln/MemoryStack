#图集 #ugui #优化
引用:
https://zhuanlan.zhihu.com/p/456101373

搬运(以防找不到):
## **[精灵图集](https://zhida.zhihu.com/search?content_id=189486917&content_type=Article&match_order=1&q=%E7%B2%BE%E7%81%B5%E5%9B%BE%E9%9B%86&zhida_source=entity)(Sprite Atlas)简介**

在_Unity_中，通常渲染一个纹理会调用一次_[DrawCall](https://zhida.zhihu.com/search?content_id=189486917&content_type=Article&match_order=1&q=DrawCall&zhida_source=entity)_。一个2D项目，可能包含大量的纹理，如果绘制每个纹理都调用一次_DrawCall_，这会占用过多的资源，从而影响整个应用程序的性能，看下面一个例子。

在_Unity_中新建一个场景，为了排除其他干扰，删掉_Camera_和_Directional Light_，添加两个_Image_为它们赋值相同的_Sprite_

![](https://pic2.zhimg.com/v2-9e997b842db69ee28bf7ded0e6e3020b_1440w.jpg)

可以看到此时的_DrawCall_为1

![](https://pica.zhimg.com/v2-3ff1828f7e89d81689108b651cd589a2_1440w.jpg)

将上面两个_Image_的_Sprite_更换成不同的

![](https://picx.zhimg.com/v2-ab2b56ca20bfb3dcf1dfc915c559fb57_1440w.jpg)

这时_DrawCall_变成了2

![](https://pic2.zhimg.com/v2-1f67696e26010eb493d356e3c049a513_1440w.jpg)

**为了降低性能消耗，我们可以使用精灵图集（_Sprite Atlas_）技术，它能够将多个纹理（_texture_）合并成一个大纹理，当访问图集中的多个纹理时，也只需要调用一次_DrawCall_** 。听起来挺不错，看一下如何使用。

> **什么是_DrawCall_？**  
> [unity文档的原文](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Manual/optimizing-draw-calls.html)是：A draw call tells the graphics API what to draw and how to draw it. Each draw call contains all the information the graphics API needs to draw on the screen, such as information about textures,shaders, and buffers.Draw calls can be resource intensive, but often the preparation for a draw call is more resource intensive than the draw call itself.  
> 简单来说drawcall就是cpu告诉gpu要绘制哪些东西，以及怎么绘制它们，比如物体的几何信息、贴图、shader等信息。drawcall本身就很消耗性能，而且drawcall的准备工作通常比它本身更消耗资源。  
> 此外，渲染中还有一个重要的概念[SetPass Call](https://zhida.zhihu.com/search?content_id=189486917&content_type=Article&match_order=1&q=SetPass+Call&zhida_source=entity)，[文档](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Manual/ProfilerRendering.html)上说的是：The number of times Unity switched which shader pass it used to render GameObjects during a frame. A shader might contain several shader passes and each pass renders GameObjects in the scene differently.  
> [两者的区别](https://link.zhihu.com/?target=https%3A//discussions.unity.com/t/setpass-calls-vs-drawcalls/685754)为：SetPass refers to the material setup, and DrawCalls refer to each item submitted for rendering.When batching is enabled, Unity can perform a single SetPass call, and then submit multiple draw calls, re-using the same material state.  
> 对此，个人的理解是drawcall是指cpu向gpu调用渲染指令，在这之前准备渲染信息计算量很大。而setpass call是渲染时切换shader pass的次数。

  

这是上面用到的两张图片，需要的自取

![](https://picx.zhimg.com/v2-79294aed685a42771428d8bc54e1d075_1440w.jpg)

![](https://picx.zhimg.com/v2-36a36bd6a5f10c85a445288cb91ba00b_1440w.jpg)

## **如何打包图集**

在Unity2017之前，打包图集使用的是旧版工具_[SpritePacker](https://zhida.zhihu.com/search?content_id=189486917&content_type=Article&match_order=1&q=SpritePacker&zhida_source=entity)_，后来引入了新的_SpriteAtlas_，并且Unity2020.1之后的版本都已经放弃使用旧版的_SpritePacker_了，因此本文主要讲解如何使用_SpriteAtlas_打包图集。

在使用之前，需要先安装_2D Sprite_插件，具体步骤为 `Window -> Package Manager`，将_Packages_选择为_Unity Registry_，搜索_2D Sprite_并且下载，如下图所示：

![](https://picx.zhimg.com/v2-417623368b505448a7cbe97dd89f7187_1440w.jpg)

安装完成后，开启图集打包的功能， `Edit -> Project Settings -> Editor -> Sprite Packer-> Mode`，默认是_Disable_，我们可以选择下面四种模式之一(不同版本的选项可能并不一致，我用的是Unity2020.3)：

![](https://pic4.zhimg.com/v2-2024bb52277b2393ac3e3c1b4d95b58d_1440w.jpg)

- _**Disabled**_：禁用图集相关功能。
- _**Sprite Atlas V1 - Enabled For Builds**_：仅当发布（_Build_）的时候构建图集，在_Editor_和_Play_模式中引用原始纹理而非图集中的纹理。
- _**Sprite Atlas V1 - Always Enabled**_：在运行时（_Build_和_Play_模式）引用图集中的纹理，在_Editor_模式中引用原始纹理。
- **_Sprite Atlas V2 (Experimental) - Enabled_** ：Unity2020.3版本这个功能还处于实验中，默认在运行时引用图集中的纹理。_Sprite Atlas V1_ 不支持缓存服务器（_Cache Server_），_Unity_只能将打包的图集数据存储在`Library/AtlasCache`文件夹中，并且也不能有依赖项，不支持命名对象导入器（_named objects importer_），而_Sprite Atlas V2_提供了对上述功能的支持。具体可以参考 [Unity Manual：Sprite Atlas Version 2](https://link.zhihu.com/?target=https%3A//docs.unity.cn/Manual/SpriteAtlasExperimental.html)

这里选用_Sprite Atlas V2_，选择好之后，创建一个图集，`Create -> 2D -> Sprite Atlas.`

![](https://pic3.zhimg.com/v2-3f4d72ebba8ddee28c73932b5fab9438_1440w.jpg)

我们看看新建的图集包含哪些属性，选中新建的图集，下面是它的属性面板

![](https://pic4.zhimg.com/v2-ce75bf366894fa64c2e092e45298ddfb_1440w.jpg)

![](https://picx.zhimg.com/v2-45bce8bea4be25a649b94119a99389f9_1440w.jpg)

对于其中比较模糊的概念，单独拎出来讲解

### **图集类型（_Type_）**

图集分为主图集和变体图集，那什么是_Variant_（变体图集）呢？切换成_Variant_模式看一下，截取属性面板中部分不同的内容

![](https://pic1.zhimg.com/v2-d86d3c9e3af3b66112c8b3c139d6d7dc_1440w.jpg)

可以发现，它多了_Master Atlas_和_Scale_属性。

- _Master Atlas_：设置主图集内容的副本作为自己的内容。
- _Scale_：设置变体图集的缩放因子，介于0.1~1之间。

变体图集的作用是什么呢？**它的主要目的是创建与主图集不同分辨率的图集，主图集中_Sprite_的分辨率 * _Scale_（缩放因子）得到的结果就是变体图集中_Sprite_的分辨率**，它自身不包含_Objects for Packing_属性，因此**变体图集中的内容都是主图集的副本**。

当项目中既有主图集，又有该主图集的变体图集时，可以使用这两个图集中任意一个的_Sprite_。如果要自动从变体图集而不是主图集中加载_Sprite_，那就仅为变体图集启用_Include in Build_选项，并关闭主图集的这个选项。

### **_Include in Build_**

**_Unity_在打包的项目中会包含图集，并且在运行时自动加载它们**，取消勾选图集的_Include in Build_选项可以禁用此行为。

如果禁用_Include in Build_，_Unity_仍会将图集打包到项目_Assets_文件夹中的** .spriteatlas * 文件中，只是**运行的时候不会加载到内存中**。因此，当精灵引用已禁用的图集中的纹理，由于引用纹理不可用或未加载（_not available or loaded_），该纹理将无法被找到，引用它的图片将显示为空白。此时要加载精灵图集，必须使用脚本通过**后期绑定（_Late Binding_）**执行此操作。（后期绑定稍后会讲）

## **使用和加载图集资源**

将_Sprite_添加进图集前需要将自身的_Texture Type_设置为_Sprite(2D and UI)_，然后才能将其加入图集的_Objects for Packing_进行打包，添加后记得点击_Pack Preview_按钮保存更改(因版本而异，使用_Sprite Atlas V1_不需要点击)，打包完成后我们可以对图集进行预览。

![](https://pic1.zhimg.com/v2-0b500af2687e58d033c502dae0cb8d0a_1440w.jpg)

### **加载图集**

图集可以放在_Resources_文件夹中加载，也可以打包到_AssetBundle_中进行加载。

我们依然使用上面例子中的两个_Image_，假设打包的图集位于_Resources_文件夹中并且名为_FirstAtlas_，图集中的两张Sprite名字分别为_Message_和_Lovely_，场景中的两个_Image_分别是_firstImg_和_secondImg_，挂载脚本并为_Image_赋值。我们可以通过下面代码获取图集并且加载里面的_Sprite_。

```csharp
SpriteAtlas atlas = Resources.Load<SpriteAtlas>("FirstAtlas");  //图集名称
Sprite message = atlas.GetSprite("Message");    //精灵体名称
Sprite lovely = atlas.GetSprite("Lovely");
​
firstImg.sprite = message;
secondImg.sprite = lovely;
```

运行后可以看到，虽然有两张_Sprite_不同的图片，但它们的_DrawCall_依然为1，说明合理使用图集确实能降低_DrawCall_。

![](https://pic2.zhimg.com/v2-66c20f8df43ca01e1ee35d75692f56e3_1440w.jpg)

![](https://pic4.zhimg.com/v2-093df94e8ee0156ab94f7a17476981b1_1440w.png)

接下来看一下怎么通过AB包加载，代码如下，效果和上图一致 。

```csharp
//AB包保存路径及名称
AssetBundle bundle = AssetBundle.LoadFromFile(Application.streamingAssetsPath + @"\AssetBundles\myatlas");
//从AB包中加载图集
SpriteAtlas atlas = bundle.LoadAsset<SpriteAtlas>("FirstAtlas");
Sprite message = atlas.GetSprite("Message");
Sprite lovely = atlas.GetSprite("Lovely");
​
firstImg.sprite = message;
secondImg.sprite = lovely;
```

### **后期绑定（_Late Binding_）**

前面我们提到，一个图集如果没有勾选_Include in Build_选项，只能通过后期绑定来加载里面的Sprite。**所谓后期绑定，就是Unity在加载资源时，对于引用到的_Sprite_，_Unity_由于不知道_Sprite_和图集的引用关系，所以会发送一个请求加载这个_Sprite_的事件**。使用者需要监听这个事件（即`SpriteAtlasManager.atlasRequested`回调），然后把这个_Sprite_所属的图集返回给_Unity_。这也就意味着，使用者需要自己加载图集。

依然以上面的两个_Image_为例，我们将它们的_Source Image_分别设置为_Lovely_和_Message_，并且取消勾选图集的_Include in Build_选项，在_Editor_模式下，图片虽然能正常显示，但运行项目后，会发现图片成了空白的。打包程序并且运行，**查看它的Profile，发现图集并没有被加载进内存。**这也印证了前面说的**禁用_Include in Build_后_Unity_将不在运行时自动加载图集，引用的_Sprite_将不可见。**

![](https://pic3.zhimg.com/v2-6d2988e05a1cf3bcd6b294b3bd7190c4_1440w.jpg)

与此同时还出现了一条警告信息 ⚠️，它提示我们图集的_atlasRequested_事件没有被监听。

![](https://pic4.zhimg.com/v2-c1d0dbe6e4697ea7e5e686d8aa6c4995_1440w.png)

_SpriteMananger_是在运行期间管理_SpriteAtlas_的一个类，它包含_atlasRequested_和_atlasRegisterer_两个事件：

**_atlasRequested_事件**

当一个_Sprite_打包进了图集但是在运行期间无法找到该图集的位置时触发，**通常情况下是因为没有勾选_Include in Build_选项**。用户可以通过_Action_委托在必要的时候进行响应，进行响应的时候需要使用_SpriteAtlas_参数，下面是这个事件的构造

```csharp
//string:精灵体打包图集的名称
//Action委托会自动加载图集
public static event Action<string, Action<SpriteAtlas>> atlasRequested;
```

**_atlasRegisterer_事件**

当_atlasRequested_中的_Action_委托调用后执行。

```csharp
public static event Action<SpriteAtlas> atlasRegistered;
```

了解了这两个事件，我们继续上面的例子。既然Unity警告没有监听_atlasRequested_事件，那我们注册这个事件。

```csharp
private void OnEnable()
{
    //注册事件
    SpriteAtlasManager.atlasRequested += RequestAtlas;
}
​
private void OnDisable()
{
    //注销事件
    SpriteAtlasManager.atlasRequested -= RequestAtlas;
}
​
private void RequestAtlas(string atlasName, Action<SpriteAtlas> callback)
{
    //获取指定图集并且调用委托
    //具体代码根据项目决定，我是将图集放在Resources文件夹
    var sa = Resources.Load<SpriteAtlas>(atlasName);
    callback(sa);
}
```

现在_Image_能够正常显示了！

我们也可以根据需要实现`SpriteAtlasManager.atlasRegistered`事件，这里就不作说明了。

**后期绑定更多时候是和_AssetBundle_一起使用的**，比如我们将UI元素_Image_放在一个AB包中，_Image_中引用的图集放在另一个AB包中，我们加载_Image_的时候并不需要知道两个AB包之间的依赖，这时可以通过后期绑定来解决这个问题。

## **需要注意的问题**

图集虽然能够有效降低_DrawCall_，但使用不当容易造成内存资源的浪费。**当图集中有一个_Sprite_处于活跃状态时，_Unity_会加载该_Sprite_所属图集中的所有纹理。**如果一个图集中的有许多纹理，就算场景中只有一个纹理被引用，那么整个图集都会被加载，会造成较大的内存消耗。

假设场景中有一张_Image_，_Image_的_Sprite_被打包到一个图集中，这个图集中还包含其他资源。

![](https://pic4.zhimg.com/v2-0bdf38ab2f3e44e04696107d7d5dfd13_1440w.jpg)

![](https://pic4.zhimg.com/v2-f47404242f63f70e29a9069f70b76329_1440w.jpg)

程序运行的时候可以看到整张图集都被加载进了内存，虽然只引用了其中一张贴图。

![](https://pic3.zhimg.com/v2-456b36f7a4bc66ccc058e97e11f0c80c_1440w.jpg)

为了解决这个问题，可以将场景中激活的所有或大多数纹理都打包到同一个图集，最好根据纹理的常见用途将分别打包成多个较小的图集，不过这样也可能造成_DrawCall_增加 。

**当打包的图集空白空间过多时，可以手动减少打包图集的大小，从而减少占用的内存。**通过调整图集_Max Texture Size_值，然后选择_Pack Preview_重新生成图集。

![](https://pic2.zhimg.com/v2-02e955a5a79efcc45a7a31eea501f76b_1440w.jpg)

当 _Max Texture Size_ 值小于精灵图集纹理的当前尺寸时，_Unity_ 会减小打包纹理尺寸以尽可能与设置的 _Max Texture Size_ 匹配，并自动修剪掉任何多余的空白空间。如果选择的一些精灵纹理超过精灵图集的 _Max Texture Size_ 设置，则精灵图集将忽略 _Max Texture Size_ 设置，并保持能够包含精灵纹理原始尺寸而需要的最小尺寸，此时编辑器也会报错。

![](https://pic4.zhimg.com/v2-c1d0dbe6e4697ea7e5e686d8aa6c4995_1440w.png)

但打包的图集仍然是可用的，只是大小与设置的不一致。

## **总结**

![](https://pic2.zhimg.com/v2-3e1897310c4b1be35058aa49ab8e8411_1440w.jpg)

## **参考**

[1] [Unity Manual: Sprite Atlas](https://link.zhihu.com/?target=https%3A//docs.unity.cn/Manual/class-SpriteAtlas.html)

[2] [Unity Forum: About "Include in Build" Behaviour](https://link.zhihu.com/?target=https%3A//forum.unity.com/threads/about-include-in-build-behaviour.481433/)

[3] [UGUI Late Binding注意事项](https://zhuanlan.zhihu.com/p/38341616)

[4] [UWA论坛：如何理解Atlas和Include in Build选项](https://link.zhihu.com/?target=https%3A//answer.uwa4d.com/question/5b9f99a41a1e9733ee03300b)