---
title:  "java单例"
---
#### 特点：
保证类只有一个实例。
####使用场景：
单例模式使用场景必然是那些需要只有一个实例的场景，单例有什么好处？
因为只有一个实例，避免了使用过程中频繁创建对象产生的内存消耗。
同时也因为只有一个实例，也就避免了所访问资源的多重占用。
其使用场景必然也就是对其特点加以利用的场景，比如IO或数据库的访问，这些操作频繁又容易
产生资源占用的操作，就很使用创建单例模式，来统一调配。
####实现方式：
单例模式的实现方式有多种：
1. 懒汉形式
2. 饿汉形式
3. DCL双重检查锁定形式
4. 静态内部类形式
5. 枚举类形式

其实也还有其他的实现形式，各种形式也各自有自己的一些不足。
主要的目的都还是围绕“保证类只有一个实例”来实现。
1.2.3这几种形式算是比较老旧的了，静态内部类的形式解决了他们的不足。
但是静态内部类没有解决的一个缺点是反序列化时的单例不唯一问题。
5枚举形式是解决了所有不足的，但是用的不太习惯，个人喜欢用静态内部类的形式。

**主要说一下静态内部类**：
首先Singleton加载时并不会加载它的静态内部类SingletonHolder，就静态内部类来说，它其实只是恰巧写在了
外部类里，和外部类并没有什么附属关系。所以只有在通过getInstance( )第一次使用到SingletonHolder时，
SingletonHolder才初始化，这就实现了懒汉节约内存。同时又由于sInstance是内部类的静态变量，其初始化发生
在类加载阶段，是单线程，并且初始化就存在静态区，是唯一的。也就保证了线程安全。

```
public class Singleton{
    private Singleton(){ }
    public static Singleton getInstance( ){
        return SingletonHolder.sInstance;
    }
    
    private static class SingletonHolder{
        private static final Singleton sInstance = new Singleton();
    }
}
```
如果要处理序列化，则修改如下：
```
public class Singleton implements Serializable {
    private Singleton(){ }
    public static Singleton getInstance( ){
        return SingletonHolder.sInstance;
    }
    
    private static class SingletonHolder{
        private static final Singleton sInstance = new Singleton();
    }
    
    private Object readResolve() throws ObjectStreamException{
        return getInstance();
    }
}
```