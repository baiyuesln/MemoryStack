
#广告追踪

### 前提

ios升级到14.5版本之后，强制要求app授权AppTrackingTransparency。不然审核不通过。

### 调用步骤

1. 创建一个AppTrackingTransparency.mm文件，里面写ios代码，然后放到unity工程的Plugin/iOS文件夹下。脚本内容：
```CSharp
#import <Foundation/Foundation.h>

#import <AppTrackingTransparency/AppTrackingTransparency.h>

#import "UnityInterface.h"

  

extern "C" {

void _RequestTrackingAuthorizationWithCompletionHandler() {

if (@available(iOS 14, *)) {

[ATTrackingManager requestTrackingAuthorizationWithCompletionHandler:^(ATTrackingManagerAuthorizationStatus status) {

NSString *stringInt = [NSString stringWithFormat:@"%lu",(unsigned long)status];

const char* charStatus = [stringInt UTF8String];

UnitySendMessage("IOSMethod", "GetAuthorizationStatus", charStatus);

}];

} else {

UnitySendMessage("IOSMethod", "GetAuthorizationStatus", "-1");

}

}

int _GetAppTrackingAuthorizationStatus() {

if (@available(iOS 14, *)) {

return (int)[ATTrackingManager trackingAuthorizationStatus];

} else {

return -1;

}

}

}
```

2. 创建一个名为ATTAuth的C#脚本，用于调用IOS代码，脚本内容如下：
```CSharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using UnityEngine;

public class ATTAuth : MonoBehaviour
{
#if UNITY_IOS || UNITY_IPHONE
    [DllImport("__Internal")]
    private static extern void _RequestTrackingAuthorizationWithCompletionHandler();

    [DllImport("__Internal")]
    private static extern int _GetAppTrackingAuthorizationStatus();

    private static Action<int> getAuthorizationStatusAction;

    /// <summary>
    /// 请求ATT授权窗口
    /// </summary>
    /// <param name="getResult"></param>
    public static void RequestTrackingAuthorizationWithCompletionHandler(Action<int> getResult)
    {
        //-1:"ios版本低于14"
        //0: "ATT 授权状态待定";
        //1: "ATT 授权状态受限";
        //2: "ATT 已拒绝";
        //3: "ATT 已授权";
        Debug.Log("RequestTrackingAuthorizationWithCompletionHandler");
        getAuthorizationStatusAction = getResult;
        _RequestTrackingAuthorizationWithCompletionHandler();
    }

    /// <summary>
    /// 获取当前ATT授权状态
    /// </summary>
    /// <returns></returns>
    public static int GetAppTrackingAuthorizationStatus()
    {
        return _GetAppTrackingAuthorizationStatus();
    }

    public void GetAuthorizationStatus(string status)
    {
        getAuthorizationStatusAction?.Invoke(int.Parse(status));
    }
#endif
}
```

3. 在游戏场景中创建一个物体，取名为IOSMethod。把ATTAuth.cs脚本挂到该物体上。

4. 在需要调用ATT弹窗的地方写下面的代码：
```CSharp
if (Application.platform == RuntimePlatform.IPhonePlayer)
{
    int curStatus = ATTAuth.GetAppTrackingAuthorizationStatus();
    if(curStatus == 0)
    {
        ATTAuth.RequestTrackingAuthorizationWithCompletionHandler((status) =>
        {
            Debug.Log("ATT status :" + status);
        });
    }
}

```

5. 打包XCode工程，在MAIN TARGETS中添加AppTrackingTransparency.framework，如下图：

![[Pasted image 20241209175303.png]]
	（步骤5也可以在脚本中完成，自行搜索 BuildIOSProcessor 打包后处理怎么使用，然后给MainTarget添加Framework即可： pbxProject.AddFrameworkToProject(unity_mian_targetGUID, “AppTrackingTransparency.framework”, false);）

6. 修改plist文件。用编辑器打开Info.plist文件添加NSUserTrackingUsageDescription字段：
```
<key>NSUserTrackingUsageDescription</key>
<string>描述文本</string>

```
或者直接在XCode中，选中Info.plist，点击+号，新增 Privacy - Tracking Usage Description
![[Pasted image 20241209175439.png]]

![[Pasted image 20241209175452.png]]
