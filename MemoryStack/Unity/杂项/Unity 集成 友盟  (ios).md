
#友盟



打开友盟官网下载SDK（https://www.umeng.com）
![[Pasted image 20241204142750.png]]![[Pasted image 20241204142814.png]]

打开Unity集成开发文档下载https://developer.umeng.com/docs/119267/detail/2746830

解压文件后，点击Unity3D，先导入common里的package再导入analytics里的common

导入后发现打包配置脚本 BuildPostProcessor.cs报错,然后

把
```
string targetName = PBXProject.GetUnityTargetName();

string targetGUID = project.TargetGuidByName(targetName);
```
改为
```
string targetGUID = project.GetUnityFrameworkTargetGuid();
```

之后即可正常使用

初始化:
```
// 开始友盟初始化 【注意要换成自己申请的AppKey】

// 参数:友盟appKey, 渠道名称

GA.StartWithAppKeyAndChannelId("675113388f232a05f1c922e1", "umeng");

  

// 设置是否打印sdk的信息 【正式包关闭】

GA.SetLogEnabled(Debug.isDebugBuild);
```

调用参考UmengGameExample即可.