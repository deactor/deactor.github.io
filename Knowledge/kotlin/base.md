---
title:  "Kotlin基础总结"
---
## 变量
关键字：
+ val : 声明只读变量，即常量，只能赋值一次
+ var : 声明可写变量，即通常意义的变量

在kotlin中，官方推荐尽量使用val来保证数据安全性。

> kotlin中的数据类型：String,Int,Double,Boolean,Float等注意首字母都是大写的。
> 此外kotlin中还提供了无符号数据类型：UByte，UShort，UInt，ULong 

> 在kotlin中无论是变量定义还是函数返回值，类型都是跟在变量后面，以冒号隔开

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

## 函数
### 无参函数
常规函数定义
```kotlin
 fun name () : return type {
    
    body

    return statement
 }
```
如果函数不需要返回值, 返回类型写Unit，相当于java中的void
```kotlin
 fun name () : Unit {
    
    body
    
    //返回值为Unit，不再需要return；
    //return statement
 }
```
此外如果函数返回Unit，Unit是可以不加的，kotlin会隐式返回一个Unit。即
```kotlin
 fun name () {
    
    body
    
 }
```
### 有参函数
常规定义
```kotlin
 fun name (param1:type, param2:type) : return type {
    
    body

    return statement
 }
```
> 警告：与某些语言（例如在 Java 中，函数可以更改传递到形参中的值）不同，Kotlin 中的形参是不可变的。您不能在函数主体中重新分配形参的值。

> 与java一样，函数签名包含函数名和输入参数，而不包括返回值


具名实参 ： 指传入的实参指明了对应的形参。那么就不再依赖参数的位置来确定实参和形参的关系了。
> 好处：见下面的默认实参

```kotlin
//以下方法：
fun birthdayGreeting(name: String, age: Int): String {
    val nameGreeting = "Happy Birthday, $name!"
    val ageGreeting = "You are now $age years old!"
    return "$nameGreeting\n$ageGreeting"
}

//如要传递name为rex， age为2。常规用法是：
birthdayGreeting("Rex", 2)

//还可以使用具名实参来调整参数位置，如：
birthdayGreeting(age = 2, name = "Rex")
```

默认实参：函数形参可以指定默认实参，即给参数默认值
```kotlin
// 此方法name参数给默认值“Rover”,即若调用该参数时不传递name参数则name默认为“Rover”
fun birthdayGreeting(name: String = "Rover", age: Int): String {
    return "Happy Birthday, $name! You are now $age years old!"
}

// 如何能调用方法而不传递对应的参数呢？那就是利用具名实参。如下：
birthdayGreeting(age = 5)
// 应为指明了5是要赋值给age的，所以不会被误认为与name的类型不匹配而报错。

// 同时利用kotlin的类型推导，上面的应该还可以简化为：
fun birthdayGreeting(name = "Rover", age: Int): String {
    return "Happy Birthday, $name! You are now $age years old!"
}
```

函数表达式：函数体可以是表达式。其返回类型可以推断出来。
```kotlin
//sampleStart
fun sum(a: Int, b: Int) = a + b
//sampleEnd

fun main() {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}
```
