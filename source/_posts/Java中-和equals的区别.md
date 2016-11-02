---
title: Java中==和equals的区别
date: 2016-10-31 14:01:40
tags: Java
---

### 概述
==是一个关系运算符或者关系操作符，它生成的是一个boolean布尔结果，计算的是操作数的值之间的关系。
equals()是一个方法，在Object类中已经实现。
```
public boolean equals(Object obj) {
    return (this == obj);
}
```
可以看出，原始的equals方法是和==一样的。
<!-- more -->
### 基本数据类型
对于byte,char,int,float,boolean等基本数据类型，==比较的是它们存储的“值”。除了他们的包装类型（Integer,Float，Byte等），它们无法使用equals方法。

### 对象
对于Integer,String，用户自定义的类等，==比较的是对象的引用（在内存中的地址）。
所有继承了基类Object的类都可以覆写这个方法，很多类都将这个方法覆写为比较存储的内容。

###### 延伸
Integer中用到了缓存，在被自动装箱且在[-128,127]之间的情况下,装箱结果是从缓存中取出来的，因此==和equals的结果都是true；
{% codeblock _title%}
	//Integer a = 整数; 这种情况都会自动装箱，整数在-128到127之间的时候，
	//装箱结果都是在缓存中取的，它们指向的都是同一个缓存对象，==结果相同
	Integer a = 127;
	Integer b = 127;

	System.out.println(a == b);   //true
	System.out.println(a.equals(b));   //true

	//不再使用缓存
	Integer c = 128;
	Integer d = 128;
	System.out.println(c == d);  //false
	System.out.println(c.equals(d));  //true
{% endcodeblock %}
```
	//Integer a = 整数; 这种情况都会自动装箱，整数在-128到127之间的时候，
	//装箱结果都是在缓存中取的，它们指向的都是同一个缓存对象，==结果相同
	Integer a = 127;
	Integer b = 127;

	System.out.println(a == b);   //true
	System.out.println(a.equals(b));   //true

	//不再使用缓存
	Integer c = 128;
	Integer d = 128;
	System.out.println(c == d);  //false
	System.out.println(c.equals(d));  //true	
```
String也比较特殊
```
//"loveu"是字符串常量，存放在常量池里，共用1块内存空间，
//所以foo1和bar1内存地址相同，== 结果为true。
//String类里的equals方法已经被重写（其中一小段源码就是在逐位比较字符），只要内容相同，返回true，所以equals的结果也为true
String foo1 = "loveu";  
String bar1 = "loveu";  
System.out.println(foo1 == bar1);      // 输出为 true  
System.out.println(foo1.equals(bar1)); // 输出为 true  

//新建2个字符串对象，内存地址不同，内容相同，故结果分别为false和true
String foo2 = new String("loveu");  
String bar2 = new String("loveu");  
System.out.println(foo2 == bar2);      // 输出为 false  
System.out.println(foo2.equals(bar2)); // 输出为 true  
```

总结来说：
　　1）对于==，如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等；
　　　　如果作用于引用类型的变量，则比较的是所指向的对象的地址

　　2）对于equals方法，注意：equals方法不能作用于基本数据类型的变量
　　　　如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；
　　　　诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。
