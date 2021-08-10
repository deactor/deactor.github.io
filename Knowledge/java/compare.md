---
title:  "list的comparable和字符串的compareTo"
---
### String的compareTo
+ 大于：返回值 > 0
+ 小于：返回值 < 0
+ 等于：返回值 = 0

```java
System.out.println("3 compare 4 = " + ("3".compareTo("4"))); // 返回 -1
System.out.println("3 compare 2 = " + ("3".compareTo("2"))); // 返回 1
System.out.println("3 compare 3 = " + ("3".compareTo("3"))); // 返回 0
```

### List的comparable
List的sort方法需要传递Comparator对象，在其compare方法中确定排序方式。对于compare的返回值：
+ 正数：交换位置
+ 负数或0：不交换位置

> **注意**：并不是返回1或者-1代表正序或倒叙，而是sort方法根据返回值判断是否进行交换位置从而实现正序或倒叙。

如倒叙可以：
```
public int compare(int a, int b) {
    if(a > b) {
        return -1; // 不交换，保持现在的a在b前面。
    } else {
        return 1; // 交换，也就是a和b互换位置，b放到a前面。
    }
}
```
也可以：
```
public int compare(int a, int b) {
    if(a < b) {
        return 1; // 交换，也就是a和b互换位置，b放到a前面。
    } else {
        return -1; // 不交换，保持现在的a在b前面。
    }
}
```

### 示例（倒序）：
```java
public class testjava {

    public static void main(String[] args) {
        ArrayList<Info> list = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            Info info = new Info("time" + i,"name" + i);
            if(i == 1 || i == 3){
                info.time = null;
            }
        
            list.add(info);
        }

        list.sort(new Comparator<Info>() {

            public int compare(Info info1, Info info2) {
                // 交换
                if(info1.time == null){
                    return 1;
                }

                // 不交换
                if(info2.time == null){
                    return -1;
                }


                int ret = 0 - info1.time.compareTo(info2.time);
                return ret; 
            }

        });
        
        for (Info info : list) {
            System.out.println("info.time -> " + info.time);
        }

    }

    private static class Info{
        public String time;
        public String name;

        public Info(String time ,String name){
            this.time = time;
            this.name = name;
        }
    }
}
```

运行结果：
>info.time -> time9  
info.time -> time8  
info.time -> time7  
info.time -> time6  
info.time -> time5  
info.time -> time4  
info.time -> time2  
info.time -> time0  
info.time -> null  
info.time -> null  