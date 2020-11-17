# Jenkins Build

## 关键词 
---------
**Jenkins Build**   
**ARCHIVE FAILED**  
**security unlock-keychain**


## 内容
---------
jenkins build的时候一直遇到这样的问题
```
15:41:02  ** ARCHIVE FAILED **
15:41:02  
15:41:02  
15:41:02  The following build commands failed:
15:41:02  	PhaseScriptExecution [CP]\ Embed\ Pods\ Frameworks /Users/XXXX/Library/Developer/Xcode/DerivedData/XXXX-eemhelbuxitxblfrzukthqviqrpn/Build/Intermediates.noindex/ArchiveIntermediates/ReleaseInt/IntermediateBuildFilesPath/XXXX.build/ReleaseInt-iphoneos/XXXX.build/Script-B424A4CDDDFB0A0A827CDA03.sh
```

一开始以为是pod的问题，试了很多种解决方案无效，后来看到这篇博客 [【Jenkins mac-slave iOS CI 集成(二)】](https://www.jianshu.com/p/0cca1befa847)

我们的jenkins build log中同样有这样的提示
```
15:41:02  Code Signing /Users/USER/Library/Developer/Xcode/DerivedData/XXXX-eemhelbuxitxblfrzukthqviqrpn/Build/Intermediates.noindex/ArchiveIntermediates/ReleaseInt/InstallationBuildProductsLocation/Applications/XXXX.app/Frameworks/AFNetworking.framework with Identity iPhone Distribution: XXXX Limited
15:41:02  /usr/bin/codesign --force --sign 8E1AB67AF6954B318A64C1D95D4B55D26C1F8AB6  --preserve-metadata=identifier,entitlements '/Users/USER/Library/Developer/Xcode/DerivedData/XXXX-eemhelbuxitxblfrzukthqviqrpn/Build/Intermediates.noindex/ArchiveIntermediates/ReleaseInt/InstallationBuildProductsLocation/Applications/XXXX.app/Frameworks/AFNetworking.framework'
15:41:02  /Users/USER/Library/Developer/Xcode/DerivedData/XXXX-eemhelbuxitxblfrzukthqviqrpn/Build/Intermediates.noindex/ArchiveIntermediates/ReleaseInt/InstallationBuildProductsLocation/Applications/XXXX.app/Frameworks/AFNetworking.framework: errSecInternalComponent
15:41:02  Command PhaseScriptExecution failed with a nonzero exit code
```
所以明确了是证书访问权限相关的问题。

根据博客内容把证书从登录目录拷贝到系统目录，打包成功。

但是后来再次打包 又报同样的问题, 不过这次知道问题是证书权限相关，所以解决方案有了具体的方向。

然后 在另一个博客[【Jenkins + Xcode + 蒲公英】配置 Jenkins 遇到的坑](https://www.jianshu.com/p/3e0b27d0a389) 中搜索到 需要在脚本中加入```security unlock-keychain``` 

## 解决方案

在xcodebuild clean之前执行

```security unlock-keychain -p 开机密码 /Users/用户名/Library/Keychains/login.keychain```


至此问题解决