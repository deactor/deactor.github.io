---
title:  "AndroidManifest合并规则"
---

## 合并规则

Apk只能包含一个AndroidManifest.xml文件，所以Android studio项目中存在多个清单文件时会在构建时被合并。总的说就是按照优先级将低优先级的清单文件合并到高优先级的清单文件中。详细的优先级规则和冲突解决方式见官网[合并多个清单文件](https://developer.android.com/studio/build/manifest-merge.html)

>主要合并就是：添加，忽略，覆盖，删除