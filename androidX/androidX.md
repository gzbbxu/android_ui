**androidx**

Jetpack 是一系列的代码库工具和指南。用来帮助各位开发者更容易地写出高品质的应用。

三个环节：

最佳实践 ，限制样板文件代码  简化复杂任务。

让你能够真正集中关注你真正在乎的代码。

和androidX有什么关系？

androidX 是 Jetpack 内部所有代码库的包名称。

所以我们把androidX 看作一个开源项目。

**我们用它来开发 测试 迭代 发布 jetpack 代码库。**

和 Support Library 有什么关系？

2008 io 大会，当时宣布 会把Support Library 重构到AndroidX 命名空间。

使Support Library 28 完成了这个动作。并且发布了Android 1.0 

**为什么要迁移？**

1，support 已经是最后一个android.support 命名空间的版本。不在维护这个命名空间。

如果大家想要获取通常会出现在Supprot Library 中的Bug 修复或者新功能。就需要迁移到AndroidX .

2，更好的包管理：

之所以重构，也就是标准化的独立版本迭代 更好的标准化命名。以及更频繁的发布 原因就在于此。

这些都是AndroidX 开箱可用的东西。

3，有些其他的代码库已经迁移到了AndroidX 命名空间。

其中很多值得一提 包括Google play 服务、Firebase 、Butterknife ,mockito2,SQLDelight

4，这些代码库 随着androidX 推出。在androidX命名空间取得了很多进展。很多人已经听说了 Jetpack Compose 或CameraX ,它们也都会进入AndroidX 命名空间。前提是要迁移到androidX.

**如何迁移？**

首先做一些事情，为将来的迁移做好准备。让迁移更加顺利。

1，备份你的项目。版本控制系统。迁移会改变项目内很多文件。

2，迁移过程中 尽量少进行开发工作。

3，单独的分支上进行。



开始迁移：

一，需要把Support Library 升级到版本28.

如果采用的是旧版本的Support Library 如26,27 ，

并尝试直接迁移到AndroidX 那么可能不会很顺利。

因为 你不仅要解决命名空间更改的问题，还要解决api 26,27,28 和 androidX 的更改问题。

所以建议，**首先试着升级到版本28，并解决所有api 更改问题，并采用api 28 进行应用编译。**

届时，你就可以直接跳到AndroidX . **Support Library  版本28 和AndroidX 在二进制层面是等效的** 。这是什么意思呢？

意思是，**这两个版本的差异之处仅仅限于代码包名称。**

**一切api 均相同。**

这样， 你就只需要做最少的事情，来解决从版本28 迁移到androidX 的问题。

把应用升级到Support Library  28之后，

二，接下来要启用Jetifier ,Jetifier 是什么 它的存在有何必要性？

Jetifier  会帮助你把第三方依赖迁移到AndroidX.

也就是说Jetifier 会更改这些依赖的flight 代码。

从而实现你使用Androidx的项目兼容。

**但是有几件事Jetifier 不会做的。**

**不会更改你的源代码，也不会迁移你生成的代码**，

如何在应用中启用Jetifier 呢？

在gradle.properties 文件中添加几行代码。

```
android.useAndroidX = true
android.enableJetifier=true
```

把 android.useAndroidX，android.enableJetifier 赋值为true.

useAndroidX true 意味着，当你使用代码自动完成功能 导入代码库时，就会导入那个代码库的AndroidX版本，而不是旧的Support Library 版本。

三：启动了Jetifier 之后，需要更新依赖。

很多第三方依赖需要在你真正开始迁移前完成更新，

这些都是我们刚提过的依赖

Butterknife ,Glide,Mockito 2,SqlDelight.

有一个旗舰样本应用叫Plaid,它使用Glid图像加载代码库。

Plaid 举例，如何迁移到androidX .我们发现 如果不首先迁移Glide依赖，就开始迁移进程，就会遭遇大量编译问题，我们经查找发现 我们使用的那个版本的Glide,并不兼容AndroidX ,于是我们首先更新了Glide以及其他依赖，随后进行了AndroidX 迁移，同样的错误不在出现。所以，在**开始迁移之前 请尽量更新第三方库依赖。**

如果你在使用代码生成库，那么Jetifier 不会修改它，所以你可能需要查看，代码生成库是否与AndroidX 兼容。

一些常见的错误：

如果你忽略2,3 步骤， 可能会出现这种错误：

一个错误是 你在使用的第三方代码库 与AndroidX 不兼容。你会发现 它会试图引入较老版本的Support Library .

此外，如果你的而项目完成了部分的迁移，那么你可能会收到一个**重复类错误**。它会尝试从Support Library 和 AndroidX 引入相同的代码。

四： 真正的源代码

我们有Support Library 28 ,启动了Jetifier ，安排好了依赖，在studio 3.2 稳定版 或者更高的版本中。有一个“迁移androidX”按钮 用起来很方便。

Migrate to AndroidX 

它的作用是 迁移你的源代码。适用于大多数情况，

有些情况可能不是用studio ,或者更复杂的应用构架是无法自动化的，这种情况下，有两个选择

goo.gle/androidx-migration-script

grep 命令 和 一个sed 

注意：它是相当暴力的bash 脚本。有时候可能会出现一些迁移错误。

另外就是手工操作：

前往developer.android.com  的 Migrate to Androd页面。那里面有从suppoert library 代码包到新代码包的映射指南。 以及同样的support library 的类映射指南。

根据具体的情况制定策略。

















































































