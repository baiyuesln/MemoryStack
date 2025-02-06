#混淆 #Editor #审核

参考:

	https://zhuanlan.zhihu.com/p/113436475
	https://www.cnblogs.com/zhaoqingqing/p/17832371.html
	https://developer.aliyun.com/article/1360617
	https://www.jianshu.com/p/9e069f708b3c
	https://www.ipaguard.com/doc/hot/start.html

UI全新、代码重构，全新类名、函数名，新开发者账户送审，打包设备、全新IP送审

前面对以往的失败经历进行了总结，也在网上搜集了很多更严格的处理方式。最后我们制定具体的实施条例来进行检验。

1.UI设计层面：设计全新的UI，特别是注意主页的差异性以及精致度，能用苹果的新功能新特性就尽量用 [[对项目中未引用的资源进行一键删除]]

2.代码层面： [[Untiy代码混淆  对指定文件夹下的.cs代码进行类名,函数名,变量名的一键混淆(保留Unity Editor引用)]]

修改文件夹名称、文件夹的结构、文件名；修改资源文件hash值、类名、方法名、属性名；

添加混淆函数方法体、添加混淆属性、自动调用生成的混淆方法、字符串混淆加密（优化现有的脚本处理，添加这些自动化处理功能）；

如果某个文件代码量比较大，就需要多去修改里面的方法名，变量名，以及顺序；

国际化的宏，通知的宏，打包的宏，广告宏，[Key值](https://zhida.zhihu.com/search?content_id=113338484&content_type=Article&match_order=1&q=Key%E5%80%BC&zhida_source=entity)宏，静态字符串全部进行修改；

换代码框架，可以是第三方类库，也可以是苹果的系统框架。如果实在换不了，则导入几个无用的框架（3个苹果框架，3个第三方类库）。

3.垃圾代码：[[垃圾代码生成器]]

添加垃圾代码，垃圾代码量占总代码的30% - 40%（垃圾[代码生成](https://zhida.zhihu.com/search?content_id=113338484&content_type=Article&match_order=1&q=%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90&zhida_source=entity)脚本，增加随机性，降低其自身的重复率，也要注意不同马甲包之间垃圾代码的重复率）

加减代码注释（优化现有脚本）

4.服务器层面：

技术网站、隐私协议用独立域名处理；

域名修改

5.账号，证书，打包等方面：

申请证书时使用不同电脑

申请证书时添加不同测试设备

登录账号的IP和传包的IP保持一致

有条件最好不要用同样的MAC打包，如无条件，尽可能不超过5个克隆包

上传克隆包IP，尽量避免与其他克隆包的IP相同

6.如果前面的方式还不能通过审核，则可以逐步尝试下面的方式：

代码加密混淆（审核时可能需要勾选加密选项）[https://github.com/chenxiancai/STCObfuscator](https://link.zhihu.com/?target=https%3A//github.com/chenxiancai/STCObfuscator)

采用swift重写工程（暂不考虑，时间成本太高）

服务器接口名修改

收费工具：[https://zfj1128.blog.csdn.net/article/details/95482006](https://link.zhihu.com/?target=https%3A//zfj1128.blog.csdn.net/article/details/95482006)（传说通过率挺高）


## checklist[#](https://www.cnblogs.com/zhaoqingqing/p/17832371.html#854912385)

1. 代码层面：对文件名，方法名进行修改
    
2. Resources目录添加无用和不同的shader
    
3. 服务器ip，域名不要重复
    
4. 资源方面：不同包使用不同的图集，图集添加后缀
    
5. level名字不要一样
    
6. 闪屏，sdk图片的md5不要一样
    
7. 屏蔽打印，别让抓到一样的日志
    
8. 图标检查有没有带透明通道
    
9. 使用不同的打包机，不同的证书

