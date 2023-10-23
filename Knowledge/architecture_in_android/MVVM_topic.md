---
title:  "MVVM 总结"
---

## 与MVP的区别
MVVM 架构中的 ViewModel 扮演着与 MVP 架构中的 Presenter 类似的角色。两种架构的区别在于 View 分别与 ViewModel 或 Presenter 通信的方式：
* 当app修改MVVM架构中的ViewModel时，View会被库或框架自动更新。您不能直接从 ViewModel 更新 View，因为 ViewModel 无权访问必要的引用。
* 但是，您可以从 MVP 架构中的 Presenter 更新视图，因为它具有对视图的必要引用。当需要更改时，您可以从 Presenter 显式调用视图来更新它。

数据绑定库确保 View 和 ViewModel 保持双向同步，如下图所示。
<img src="https://github.com/googlesamples/android-architecture/wiki/images/mvvm-databinding.png" alt="数据绑定使视图和视图模型保持同步。"/>
