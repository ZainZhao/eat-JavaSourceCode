> Integer class based on jdk1.8 

[TOC]

## TODO

- 反射 https://blog.csdn.net/swadian2008/article/details/100679877



## Question & Answer

### Q1：MIN_VALUE & MAX_VALUE

$-2^{31} $ 至 $ 2^{31}-1$

```java
@Native 
public static final int   MIN_VALUE = 0x80000000; 
@Native 
public static final int   MAX_VALUE = 0x7fffffff;
// 注意
Integer.MAX_VALUE + 1 == Integer.MIN_VALUE
Math.abs(Integer.MIN_VALUE) ==  Integer.MIN_VALUE
```

#### Q1.1：@Native的作用

主要是为了自动生成本地代码（本地头文件）的需要，以便本地代码可以很容易地使用这些常量



### Q2：TYPE

在八大基本数据类型的包装类中都有一个常量：TYPE，表示该包装类对应的基本数据类型的Class对象（即基本类型的类对象，注意不是其包装类型的类对象）

```java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int") // 返回虚拟机中的类对象
    
    
// 注意
Integer.class == int.class  // False
Integer.TYPE == int.class; // True
Integer.TYPE == Integer.class;// False
```

#### Q2.1：获取Class对象的方式

1. 使用对象的 *getClass*() 获得（仅限于引用类型）

```java
String string = new String();
Class  class1 = string.getClass();
```

2. 使用类的class成员属性

```java
Class class2 = String.class;
// 不仅能用于引用类型，基本类型也可以。数组也是可以的
Class class3 = int.class;
Class class4 = int[][].class

// 用.class来创建对Class对象的引用时，不会自动地初始化该Class对象
```

3. 使用Class类的forName("类完整路径")方法 （仅限于引用类型）

```java
Class<?> strClass = Class.forName("java.lang.String");
```

4. 使用包装类的TYPE属性来获取包装类对应基本类型的类对象

```java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

