---
title:  "Android Typedef 注解"
---
## 使用场景：
使用 @IntDef 和 @StringDef 注解，您可以创建整数集和字符串集的枚举注解来验证其他类型的代码引用。Typedef 注解可以确保特定参数、返回值或字段引用一组特定的常量。这些注解还会启用代码补全功能，以自动提供允许的常量。

Typedef 注解使用 @interface 来声明新的枚举注解类型。@IntDef 和 @StringDef 注解以及 @Retention 可以对新注解添加注解，是定义枚举类型所必需的。@Retention(RetentionPolicy.SOURCE) 注解可告诉编译器不要将枚举注解数据存储在 .class 文件中。

## 使用方式：
```java
import android.support.annotation.IntDef;
//...
public abstract class ActionBar {
    //...
    // Define the list of accepted constants and declare the NavigationMode annotation
    @Retention(RetentionPolicy.SOURCE)
    @IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
    public @interface NavigationMode {}

    // Declare the constants
    public static final int NAVIGATION_MODE_STANDARD = 0;
    public static final int NAVIGATION_MODE_LIST = 1;
    public static final int NAVIGATION_MODE_TABS = 2;

    // Decorate the target methods with the annotation
    @NavigationMode
    public abstract int getNavigationMode();

    // Attach the annotation
    public abstract void setNavigationMode(@NavigationMode int mode);
}
```
也可以直接将常量定义到@interface中
```
public abstract class ActionBar {
    //...
    // Define the list of accepted constants and declare the NavigationMode annotation
    @Retention(RetentionPolicy.SOURCE)
    @IntDef({NavigationMode.NAVIGATION_MODE_STANDARD, NavigationMode.NAVIGATION_MODE_LIST, NavigationMode.NAVIGATION_MODE_TABS})
    public @interface NavigationMode {
        int NAVIGATION_MODE_STANDARD = 0;
        int NAVIGATION_MODE_LIST = 1;
        int NAVIGATION_MODE_TABS = 2;
    }

    // Decorate the target methods with the annotation
    @NavigationMode
    public abstract int getNavigationMode();

    // Attach the annotation
    public abstract void setNavigationMode(@NavigationMode int mode);
}
```

> 构建此代码时，如果 mode
> 参数未引用任何已定义的常量（NAVIGATION_MODE_STANDARD、NAVIGATION_MODE_LIST
> 或 NAVIGATION_MODE_TABS），系统会生成一条警告。 您还可以结合使用 @IntDef
> 和 @IntRange，以指明某个整数可以是一组给定的常量，也可以是某个范围内的值。

## 注意
跟lint检查有关系，lint的默认检查规则包含改注解检查，如果自定义了lint检查规则，去除了这一项，是不会提示错误的。
