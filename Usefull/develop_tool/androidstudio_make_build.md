---
title:  "AndroidStudio的make与build"
---
可以执行编译的途径：
+ 顶部菜单build中有make、make module、rebuild。
  + 菜单栏中的make、rebuild都是执行assembelDebug或assembleRelease，只是rebuild会先clean，并且不能选择单个module。
+ 侧边gradle栏目有assemble、assembleDebug、assembleRelease、buld等task。
  + gradle栏目的task，buld任务会同时编译debug和release，并且有执行assembleDebug和assembleRelease生成apk。

所以一般模块化的项目，就用make指定module来编译，或者使用assembleDebug任务，实际执行都是一样的。
> **注意**：对于hide的api，我使用菜单栏中的选项编译没有报错提示，正常生成了apk，但是用assembleDebug居然报错提示了。没弄明白。