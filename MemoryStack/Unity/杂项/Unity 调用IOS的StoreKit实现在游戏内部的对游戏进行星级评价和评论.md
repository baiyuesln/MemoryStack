#store评分

1,在Unity目录下 新建一个文件夹  Plugins\IOS

2,IOS文件夹下 新建两个文件  UnityStoreKit.m, UnityStoreKit.h

UnityStoreKit.m:   记得替换appid
```
 
#import "UnityStoreKit.h"
 
 
@implementation UnityStoreKit
 
#if defined(__cplusplus)
 
extern "C"{
    
#endif
    
    void _goComment()
    
    {
        
        if([SKStoreReviewController respondsToSelector:@selector(requestReview)]) {// iOS 10.3 以上支持
            
            [SKStoreReviewController requestReview];
            
        } else { // iOS 10.3 之前的使用这个
            
            NSString *appId = @"1488291408"; //项目在苹果后台的appid
            
            NSString  * nsStringToOpen = [NSString  stringWithFormat: @"itms-apps://itunes.apple.com/app/id%@?action=write-review",appId];//替换为对应的APPID
            
            //[[UIApplication sharedApplication] openURL:[NSURL URLWithString:nsStringToOpen]];
			//好像这句快一点
			dispatch_async(dispatch_get_main_queue(), ^{
                [[UIApplication sharedApplication] openURL:[NSURL URLWithString:nsStringToOpen] options:@{} completionHandler:nil];
            });
            
        }
        
    }
    
#if defined(__cplusplus)
    
}
 
#endif
 
 
 
@end
```

UnityStoreKit.h:
```

 
//  UnityStoreKit.h
 
//  test
 
//
 
//  Created by HH on 2018/4/13.
 
//  Copyright © 2018年 HH. All rights reserved.
 
//
 
 
 
#import <Foundation/Foundation.h>
 
#import <StoreKit/StoreKit.h>
 
 
 
@interface UnityStoreKit : NSObject
 
 
 
@end
 
 
```

声明:
```
#if UNITY_IOS

[System.Runtime.InteropServices.DllImport("__Internal")]

public static extern void _goComment();

#endif
```

调用:

```
//app评分

public static void AppMark()

{
//仅弹出一次

if (PlayerPrefs.GetInt("StoreMark", 0) == 0)

{

#if !UNITY_EDITOR && UNITY_IOS

Init._goComment();

#endif

PlayerPrefs.SetInt("StoreMark", 1);

}

}
```