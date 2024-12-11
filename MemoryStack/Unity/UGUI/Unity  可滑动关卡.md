
#关卡设计

### 步骤：

1. 创建UI元素：

- 在Unity的Hierarchy面板中，右键点击，选择UI -> Canvas，创建一个画布。

- 在Canvas下，右键点击，选择UI -> Scroll View，创建一个滚动视图。

2.  设置Scroll View：

- 选中Scroll View，在Inspector面板中，你会看到Scroll Rect组件。

- 在Scroll Rect组件中，确保Horizontal或Vertical选项被勾选，具体取决于你想要的滑动方向。

3. 添加长图片：

- 在Scroll View下，找到Viewport，然后在Viewport下找到Content。

- 在Content下，右键点击，选择UI -> Image，添加一张长图片。

- 在Image组件中，设置你的长图片的Source Image。

4. 调整Content的大小：

- 选中Content，在Inspector面板中，调整RectTransform的Width和Height，==**确保它大于Viewport的大小**==，以便能够滑动。
5. 设置Content的Layout：

- 在Content上添加一个Layout组件，例如Vertical Layout Group或Horizontal Layout Group，根据你的需求进行设置。
 ![[Pasted image 20241211191128.png]]