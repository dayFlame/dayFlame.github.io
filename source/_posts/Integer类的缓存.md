---
title: Integer类的缓存
date: 2016-11-02 17:07:15
tags: Java
---

### 引子
``` Java
    Integer a = 127;
    Integer b = 127;   
    System.out.println(a == b);   //true
    
    Integer c = 128;
    Integer d = 128;   
    System.out.println(c == d);   //false
    
    Integer int1 = Integer.valueOf(127);
    Integer int2 = Integer.valueOf(127);
    System.out.println(int1 == int2);  //true
    
    Integer int3 = Integer.valueOf(128);
    Integer int4 = Integer.valueOf(128);
    System.out.println(int3 == int4);  //false
```
<!-- more -->
### 解释
jvm为了节省内存，对于下列包装对象的两个实例，当它们的基本值相同时，进行==操作的结果总为true:
Boolean,Byte,Character(\u0000 - \u007f,7f是十进制的127),Integer(十进制-128 ~ 127)

对于上述代码中的前两小段，jvm会对其进行自动装箱，例如实际上，运行代码时，执行了
``` Java
Integer a = Integer.valueOf(127);
```
在JDK7的Integer类中valueOf()部分源码如下：
``` Java
public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
如果数在-128到127之间的话，就会返回一个静态类IntegerCache中的成员变量cache中的一个对象的引用。
IntegerCache部分源码如下
``` Java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
    }

    private IntegerCache() {}
}
```
在类加载时就将-128 到 127 的Integer对象创建了，并保存在cache数组中，一旦程序调用valueOf 方法，如果i的值是在-128 到127之间就直接在cache缓存数组中去取Integer对象。所以你不管创建多少个这个范围内的Integer调用ValueOf函数返回的都是同一个对象。

### 延伸
Long也有类似的缓存机制，但是无法自行调整参数
``` Java
private static class LongCache {
    private LongCache(){}

    static final Long cache[] = new Long[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}

public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}
```
其他包装类：
```
Byte：(全部缓存)
Character(0 - 127)
Short(-128 — 127缓存)
Long(-128 — 127缓存)
Float(没有缓存)
Doulbe(没有缓存)
```
垃圾回收器也只会回收非缓存对象