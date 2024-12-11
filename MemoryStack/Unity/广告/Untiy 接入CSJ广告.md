
#csj广告 

# 环境

unity editor : 2021.3.16
csj Untiy SDK 6.4.0

# 过程 (IOS)

1. 下载融合untiy sdk  https://www.csjplatform.com/union/media/union/download/pangle

2. 接入文档:https://www.csjplatform.com/union/media/union/download/detail?id=202&docId=28039&locale=zh-CN&osType=

3. 打开项目，然后依次选择 Assets > Import Package > Custom Package，并找到您下载的 SDK.unitypackage和Example.untiypackage文件。确保选择所有文件，然后点击 Import导入。

4. 删除Assets/Plugins下多余的Newtonsoft.Json.dll

5. 使用example测试

6. 完成

# 过程 (Android)

1. 下载融合untiy sdk  https://www.csjplatform.com/union/media/union/download/pangle

2. 接入文档:https://www.csjplatform.com/union/media/union/download/detail?id=202&docId=28039&locale=zh-CN&osType=

3. 打开项目，然后依次选择 Assets > Import Package > Custom Package，并找到您下载的 SDK.unitypackage和Example.untiypackage文件。确保选择所有文件，然后点击 Import导入。
4. 在UnityEditor的顶部状态栏中，打开File - Build Settings - Player Settings... - Publishing Settings，将以下三项选中，UnityEditor会自动生成相关文件（右图二）【注意】：若项目内在勾选前已存在如下图二文件，需先手动删除Unity编辑器会自动重新生成。（以下三项Build选项，不同的unity编辑器版本存在差异，仅勾选存在的选项即可）。
![[Pasted image 20241210114947.png]]
![[Pasted image 20241210115034.png]]
5. 将下载得到的CSJ.androidlib，copy到项目的Assets/CSJ/Plugins/Android/目录下
6. **将步骤5目录中build.gradle文件中compileSdkVersion、buildToolsVersion**修改成**开发者项目**匹配的版本，如下图，笔者的版本是30和30.0.2。
![[Pasted image 20241210133631.png]]
7. 在顶部状态栏中，打开Assets - External Dependency Manager - Android Resolver - Resolve，进行SDK的依赖解析，等待解析成功。如果解析失败，请重试几次。
8. 使用example测试

完成.