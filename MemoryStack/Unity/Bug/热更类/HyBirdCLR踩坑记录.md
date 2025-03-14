#HyBirdCLR #热更新
### 报错一:No member named 'GetUnderlyingInterpreterImage' in 'hybridclr::metadata::MetadataModule'

更换引擎版本或者打包机器平台后一定要重新install!!
![[Pasted image 20250303150045.png]]
原因是这里的install按钮点击后会将Unity Editor的本地文件夹中的[il2cpp](https://zhida.zhihu.com/search?content_id=240757731&content_type=Article&match_order=1&q=il2cpp&zhida_source=entity)代码拷贝到 项目名/HybridCLRData/LocalIl2CppData-{current platform}[Editor文件夹](https://zhida.zhihu.com/search?content_id=240757731&content_type=Article&match_order=1&q=Editor%E6%96%87%E4%BB%B6%E5%A4%B9&zhida_source=entity)下，不同的Unity版本中的il2cpp源码不一样，可能会导致生成出来的cpp文件中有编译错误，同理，不同平台的Unity中的il2cpp源码也不一样，假设开发机是windows，打包机是mac，那么打包机上也要执行一次install。
