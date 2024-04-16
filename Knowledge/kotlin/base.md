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

## let的用法
```kotlin
// 使用 let 函数处理可空值
val processedValue = optionalValue?.let {
    // 在这里，'it' 就是 optionalValue 的非空版本
    it.capitalize() + " processed"
} ?: "Default value when optionalValue is null"
```
```kotlin
// 上面的?:是kotline中Elvis 运算符，用法如下：即将nullableString赋值给nonNullString时，判断nullableString若为空，则将"Default string"赋值给nonNullString。相当于给一个非空的默认值。
val nonNullString: String = nullableString ?: "Default string"
```

## 双引号的含义
!! 是一个后缀操作符，被称为非空断言（not-null assertion operator）。它用于强制转换一个可空类型的变量或表达式为非空类型。除非明确知道一个变量不会为null，否则就会有空指针风险。谨慎使用该操作符。

## 问号的作用
变量声明时默认都是非空类型的，如要声明可空类型变量，则类型后面要加？
```kotlin
val nullableString: String? = getNullableString()

// !!执行可空强转非空
val nonNullableString: String = nullableString!!
```

## 类型判断
### is 与 !is 操作符用于类型判断
```kotlin
// 判断obj是否是String类型
if (obj is String) {
    print(obj.length)
}

// 判断obj是否不是String类型
if (obj !is String) { // 与 !(obj is String) 相同
    print("Not a String")
}
```

## 类型转换
### as 与 as?用于类型转换
 + as ：如果转换失败，转换操作符会抛出一个异常。比如将Int强转为String
 + as? ：如果转换失败，会返回一个null，而不是抛异常。
 
如：
```kotlin
// 转换失败会抛异常
val x: String = y as String

// 转换失败，会返回null给x
val x: String? = y as? String
```

## 条件
### if表达式
在 Kotlin 中，if 是一个表达式：它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则），因为普通的 if 就能胜任这个角色。
如：
```kotlin
val max = if (a > b) a else b

// 还可以加else if
val maxLimit = 1
val maxOrLimit = if (maxLimit > a) maxLimit else if (a > b) a else b

// 分支还可以是代码块，这种情况最后的表达式作为该块的值
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}

// 所谓代码块的值就是最后一个非空表达式的值成为整个代码块的值
如｛
	val a = 2
	val b = 3
｝此代码块的值就是b的值，即3.
{
	val a = 2
	val b = 3
	a + b
}此代码块的值就是a+b的值，即5
```
### When 表达式
```kotlin
// 类似switch-case，但是比switch-case强大，不需要break，从上到下匹配，匹配到则执行->后面的语句
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> {
        print("x is neither 1 nor 2")
    }
}

//when还可以作为表达式，其匹配到的表达式的值就是when表达式的值。注意when作为表达式时必须要有else(也有例外，但最好统一加else)，如：
val y = when (x) {
    1 -> x + 1
    2 -> x + 2
    else -> {
        x + 3
    }
}

// 对于多条件匹配可以在一行组合，如：
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}


// 可以用任意表达式（而不只是常量）作为分支条件
when (x) {
    s.toInt() -> print("s encodes x")
    else -> print("s does not encode x")
}

// 还可以检测一个值在（in）或者不在（!in）一个区间或者集合中
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}

// 另一种选择是检测一个值是（is）或者不是（!is）一个特定类型的值。
// 这里的Any是什么？？？？？？？？？？？？？？？？？
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}

// when 也可以用来取代 if-else if 链。 如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd")
}

// 可以使用以下语法将 when 的主语（subject，译注：指 when 所判断的表达式）捕获到变量中：
// 上面这句翻译的不太直白，待理解重新组织语言？？？？？？？？？？？？
fun Request.getBody() =
    when (val response = executeRequest()) {
        is Success -> response.body
        is HttpError -> throw HttpException(response.status)
    }
```

## 循环
### For循环
for 循环可以对任何提供迭代器（iterator）的对象进行遍历，这相当于java中的 foreach 循环。
```kotlin
for (item in collection) {
    print(item)
}

// 也可以
for (item: Int in collection) {
    // ……
}

// 对于数字区间还可以用区间表达式
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}

// 如果你想要通过索引遍历一个数组或者一个 list，你可以这么做：
val array = arrayOf("a", "b", "c")
for (i in array.indices) {
    println(array[i])
}
// 或者你可以用库函数 withIndex：
val array = arrayOf("a", "b", "c")
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```
### while
while与其它语言比较类似：
```kotlin
while (x > 0) {
    x--
}

do {
  val y = retrieveData()
} while (y != null) // y 在此处可见
```
## 返回与跳转
涉及return，break，continue标签，具体详见[官网](https://book.kotlincn.net/text/returns.html)

## 异常
> kotlin没有受检异常是什么意思？？？？？？？

### throw
```kotlin
fun main() {
//sampleStart
    throw Exception("Hi There!")
//sampleEnd
}
```

### try-catch
```kotlin
// 与java相同
try {
    // 一些代码
} catch (e: SomeException) {
    // 处理程序
} finally {
    // 可选的 finally 块
}

// 与java不同的地方：try 是一个表达式，可以有返回值
// try-表达式的返回值是 try 块中的最后一个表达式或者是（所有）catch 块中的最后一个表达式。 finally 块中的内容不会影响表达式的结果。
val a: Int? = try { input.toInt() } catch (e: NumberFormatException) { null }
```

### Nothing 类型
在 Kotlin 中 throw 是表达式，throw 表达式的类型是 Nothing 类型，这个类型没有值，而是用于标记永远不能达到的代码位置。 在你自己的代码中，你可以使用 Nothing 来标记一个永远不会返回的函数：
```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```
当你调用该函数时，编译器会知道在该调用后就不再继续执行了：
```kotlin
val s = person.name ?: fail("Name required")
println(s)     // 在此已知“s”已初始化
```
**还是没明白Nothing哪里好用，有什么好处？？？？？？？？？？？？**

## 类
型如：
```kotlin
// class 类名 类头 {类体} （类头，类体都是可选的）
class Person constructor(firstName: String) { /*……*/ }
```
### 构造函数
类有一个主构造函数并可能有一个或多个次构造函数。
```kotlin
// 主构造函数在类名后面，主构造函数的参数可以在属性初始化和初始化块中直接使用
// 不显示声明主构造函数，隐式默认也会有一个无参的主构造函数
class Person(name: String, age: Int) {
    // 属性初始化可以直接使用主构造函数的参数
    val mName: String = name
    var mAge: Int = age
    var mHight: Int = 0
    var mWight: Int = 0

    private val mSax: String

    // 初始化块，类似java中的游离代码块，会在构造对象时调用。
    // 只是kotlin中给这种代码块加了明确的init修饰
    init {
        mSax = "man"
    }

    // 次构造函数要委托给主构造函数，可以直接委托，也可以通过别的次构造函数间接委托。
    // 通过this委托到同一个类的另一个构造函数
    // 该次构造函数直接委托主构造函数
    constructor(name: String, age: Int, hight: Int): this(name, age){
        // 注意：委托发生在次构造函数的第一条语句时，相当与这里有一条java的super()
        // 同时属性直接初始化和init初始化块实际都相当于在主构造函数内部。
        // 所以代码的执行是属性初始化 -> init初始化块 -> 接下来下面的语句执行
        mHight = hight
    }

    // 该次构造函数委托上面的次构造函数
    constructor(name: String, age: Int, hight: Int, wight: Int): this(name, age, hight){
        mWight = wight
    }
}

// 注意：kotlin中是没有new关键字的，要创建对象，直接调用构造函数即可。如：
val person = Person("tom", 32)
```

### 抽象类
与java一样，子类继承抽象类，必须实现抽象方法。
```kotlin
abstract class Student{
    abstract fun study()
}

// 继承，在：后面声明父类的构造函数
class HighStudent: Student() {
    override fun study() {
        TODO("Not yet implemented")
    }
}
```
另外kotlin中抽象类还可以继承覆盖一个非抽象的开放成员
```kotlin
// open类 和 open方法
open class ChildPerson {
    open fun study(){}
}

// 抽象类可以复写open方法，继承Student的类都会覆盖ChildPerson中的默认实现
abstract class Student : ChildPerson(){
    abstract override fun study()
}
```
> open是什么作用？？？？ 这么做有什么好处？？？？？？？
> open是描述一个类、函数、属性是否可重写或继承。kotlin中默认类不可继承，只有open修饰的类可继承。同时open修饰方法，表示该方法可被子类重写，否则子类不可重写该方法。
> open修饰属性表示属性可被重写，常用于重写val修饰的不可变属性。同时open修饰属性和方法一样，重写的属性或方法都要用override来修饰。同时属性或方法声明为open，其所属的类必须是open的。
> override的属性和方法默认具有open的属性，可继续被其子类重写，若不想被重写，则在override前加final。抽象类默认就具有open属性，不需要再用open修饰

### 继承
kotlin中所有类都有一个共同的父类，即Any，类似java中的Object  
默认情况下kotlin中的类都是final的，即不可被继承。要是类可被继承，需要用open关键字修饰。  
同java一样，子类构造时必须调用父类的构造函数。
```kotlin
// 父类，有个主构造函数Base(p: Int)
open class Base(p: Int)

// 子类主构造函数调用父类的主构造函数Base(p: Int)
class Derived(p: Int) : Base(p)


class MyView : View {
    // 子类没有主构造函数，次构造函数必须使用super来调用父类的构造函数
    constructor(ctx: Context) : super(ctx)
    // 上面这个次构造函数或者写成下面这样
    //constructor(ctx: Context) : this(ctx, null)


    // 子类没有主构造函数，次构造函数必须使用super来调用父类的构造函数
    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```
> 注意：同java一样，子类构造时先构造父类。这意味着，基类构造函数执行时，派生类中声明或覆盖的属性都还没有初始化。在基类初始化逻辑中（直接或者通过另一个覆盖的 open 成员的实现间接）使用任何一个这种属性，都可能导致不正确的行为或运行时故障。 设计一个基类时，应该避免在构造函数、属性初始化器或者 init 块中使用 open 成员。

同java，super用来调用父类的方法，不同点在于内部类的调用方式，如下：
```kotlin
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle: Rectangle() {
    override fun draw() {
        val filler = Filler()
        filler.drawAndFill()
    }

    // inner用于声明内部类
    inner class Filler {
        fun fill() { println("Filling") }
        fun drawAndFill() {
            // 调用外部类使用super@外部类类名
            super@FilledRectangle.draw() // 调用 Rectangle 的 draw() 实现
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // 使用 Rectangle 所实现的 borderColor 的 get()
        }
    }
}
```
此外Kotlin支持多继承，那么super如何用？如下：
```kotlin
open class Rectangle {
    open fun draw() { /* …… */ }
}

interface Polygon {
    fun draw() { /* …… */ } // 接口成员默认就是“open”的
}

class Square() : Rectangle(), Polygon {
    // 编译器要求覆盖 draw()：
    override fun draw() {
        super<Rectangle>.draw() // 调用 Rectangle.draw()
        super<Polygon>.draw() // 调用 Polygon.draw()
    }
}
```
