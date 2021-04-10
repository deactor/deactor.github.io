---
sort: 2
title:  "python基础概要"
---
## 简述
python当前是非常火的一门语言，既然新学习一门语言，那么有以下几个问题：  
1. 它与其他语言有什么区别？
2. 它的主要优势是什么？
3. 它主要用来做什么？

这些问题先放在这里，之后使用python有了深切体会后再来回答。本文使用的python版本是3.6。

### 基础语法
python是类型松散型语言。学习一门语言，首先学习它的数据类型。Python的基本数据类型有：  
+ 整数(int)     : 8
+ 浮点数(float) : 8.8
+ 浮点数(str)   : "8"
+ 布尔值(bool)  : True/False

#### python程序的主函数
在学习c和java时都知道他们的主函数是main函数，这是程序的入口，那么python程序的入口在哪里？  
python其实是没有定义死主函数的。新建一个python脚本文件（后缀名为.py），执行这个脚本文件，python运行环境就一条一条的执行函数调用。先用java的语法来演示，如：  
```
void function1(){
    ...
}

void function2(){
    ...
}

void function3(){
    ...
}

function1();
function2();
function3();
```
执行这个脚本文件，就会依次执行function1(),function2(),function3()函数，执行完结束。这个样子方便也不方便，程序还是要定义一个入口，后面再说。

#### 接下来python语言中如何定义变量呢？
python定义变量不需要声明数据类型，而且连关键字都不需要。形如：
```
int_val = 8
float_val = 8.8
str_val = "python"
bool_val = True

#print函数占位符同c语言。
print("int_val = %d ,float_val = %f ,str_val = %s , bool_val = %s" 
        %(int_val float_val str_val bool_val))

#result:
#int_val = 8 ,float_val = 8.800000 ,str_val = python , bool_val = True
```
>可以看到python语言不用分号“；”来表示结尾，那么怎么来判断语句块呢？是用4个空格来确定的。Tab键要转成4个空格，不然代码里存在Tab键是会报错的，具体详见后文。

这里print函数输出bool型使用的是字符串的占位符 %s，在pyton中基础类型间可以互相转换，如：  
+ int("88") 可以将字符串“88”转换成数字88
+ float("8.8") 可以将字符串“8.8”转换成数字8.8
+ int(True) 可以将bool值True转换为数字1，False转换为数字0
+ str(88) 可以将数字88转换为字符串“88”
+ str(True) 可以将bool值True转换为字符串“True”
+ bool(15) 可以将任意非0数字(包括浮点数)转为Ture，而将0转为False

>在python中字符串可以使用双引号和单引号来定义，两者并没有任何区别，唯一的作用就是：  
单引号定义的字符串中可以正常显示双引号  
双引号定义的字符串中可以正常显示单引号


变量定义了，有**局部变量和全局变量的区分吗？**  
有，写在函数内部的是局部变量，写在函数外部的是全部变量。如果要将函数内部的变量声明为全局变量，则要使用global关键字声明，而且要声明后再赋值，而不能声明的同时赋值，会报错。
```
函数内部｛
    global a
    a = 10
｝

函数内部｛
    global a = 10 #这样会报错
｝
```

变量部分差不多就这些了。  
#### 那么如何定义函数呢？
python使用def关键字来定义函数，函数也同样不需要声明返回值类型和参数类型。其实在python中基本就见不到变量类型声明。
```
def function_test(a,b):
    return a + b 
```
而且python的语法格式没有使用“｛｝”的语句块，使用“：”来间隔函数名和函数体。
函数体范围有多大？就是到按照Tab来区分的。如下，abc都在function_a函数中，而d开始就不是在function_a中了。
```
def function_a():
    print("a")
    print("b")
    print("c")
print("d")
```
变量和函数都有了，那么对python就有了大概的认识了。后面再稍微细致的看一看。