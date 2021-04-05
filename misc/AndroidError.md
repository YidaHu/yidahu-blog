# Android常见错误解决

## 前言

在Android开发中，经常会遇到各种各样的问题，特在此处把经常遇到的问题以及解决方案记录如下，方便自己的同时也希望给遇到同样错误的朋友减少麻烦，如有疏漏敬请谅解。

### 错误提示

	Error:Execution failed for task ':app:processDebugManifest'.
	> Manifest merger failed with multiple errors, see logs
### 解决方案

这个报错肯定是你导报依赖出的问题

##### 方案第一步：

在Manifest.xml的application标签下添加

	tools:replace="Android:icon, android:theme"

多个属性用,隔开，并且记住在manifest根标签上加入

	xmlns:tools="http://schemas.android.com/tools"
##### 第二步：

##### 不用看了，问题百分之百，在你的builde.gradle的位置

	minSdkVersion 16
	targetSdkVersion 23

你看清楚，你的这个值和你要依赖的那个包，是不是builde.gradle不一样。
改完就OK了
如果还错，OK，那就是你代码有问题，看看导包，是不是导错包了

如果以上都不行,最终方案是把build.gradle中依赖的版本号改为"+".表示最新版本。比如:

	compile 'com.android.support:cardview-v7:+'