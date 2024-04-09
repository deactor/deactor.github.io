---
title:  "Kotlin基础总结"
---
## 变量
关键字：
+ val : 声明只读变量，即常量，只能赋值一次
+ var : 声明可写变量，即通常意义的变量

在kotlin中，官方推荐尽量使用val来保证数据安全性。

> kotlin中的数据类型：String,Int,Double,Boolean,Float等注意首字母都是大写的。

完整的变量声明：
```kotlin
val name: type = initial_value
// 如：
val value: Int = 2;
```
kotlin具备类型推断能力，如果声明变量的同时进行赋值，则可简化为：
```kotlin
val name = initial_value
// 如：
val value = 2;
```
如果声明的时候不赋初值，则一定要写明类型，否则kotlin无法自动推断类型。
