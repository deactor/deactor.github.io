---
title:  "android中代替java的集合类"
---
## SparseArray、ArrayMap与HashMap比较
SparseArray与ArrayMap内部都是采用二分查找。SparseArray的key必须是Integer类型。
应用场景：
+ 数据量小于1000：使用SparseArray与ArrayMap。
+ 数据量大于1000：使用HashMap。

各自优势：
+ SparseArray与ArrayMap：省内存。
+ HashMap：大数据量下性能好。