# Spring boot GC实现过程原理解析

> 这篇文章主要介绍了Spring boot GC实现过程原理解析,文中通过示例代码介绍的非常详细，对大家的学习或者工作具有一定的参考学习价值,需要的朋友可以参考下

- 内存中不可达对象（没有引用指向此对象）会被标记为垃圾对象

- 手动将对象变为垃圾对象：将指向对象的变量置为null

- 如何GC：查找，标记，清除，整理

## **控制台查看是否启动GC:**

- -XX:+PrintGC

- -XX:+PrintGCDetils

  **执行时添加参数：**

  ![img](https://img.jbzj.com/file_images/article/202008/2020810143616906.png?2020710143626)

## **手动启动GC**

System.gc()

## **自动启动GC**

（系统底层会随着创建对象的增加,然后基于内存情况，启动GC）

重复创建大量对象，内存不足时自动启动GC

## **查看对象是否被GC**

重写Object的finalize方法（此方法在垃圾回收之前执行）

## **spring Boot Bean池中的对象何时GC :**

1. 外界没有指向，
2. Bean池进行clean(spring Boot 在启动和关闭时会将池clean)
   - protoType:多实例，需要时创建，外界没有引用时变为垃圾对象
   - singleton:单实例，外界没有引用，Bean池进行clean时会变为垃圾对象

