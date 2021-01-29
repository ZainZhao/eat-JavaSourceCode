> Object class based on jdk1.8 

[TOC]

## Question & Answer

#### Q1：Java 的两种导包机制 ?

- 分类
  - 单类型导入（single-type-import）：`import java.io.File`
    - 优点：提高编译速度、避免命名冲突
    - 缺点：import语句看起来很长
  - 按需类型导入（type-import-on-demand）：`import java.io.*`
    - 并非导入整个包，而是导入当前类需要使用的类

#### Q2：为什么不能随意使用按需类型导入 ？

- 单类型导入和按需类型导入对类文件的定位算法是不一样
  - 单类型：包名和文件名都已经确定，可以一次性查找定位
  - 按需类型导入：找到类之后不会停止下一步查找，如果编译器发现了两个同名的类，就会报错。不会降低代码执行效率，但会影响编译速度
  
- e.g.

  ```java
  package com;
  import java.io.*;
  import java.util.*;
  
  查找File类，会查找：
      File
      com.File
      java.lang.File
      java.io.File
      java.util.File
  ```

#### Q3：public boolean equals(Object obj)

##### Q3.1：equals() 与 == 的区别 ?

```java
public boolean equals(Object obj) {
    return (this == obj); // Object 中的 equals 与 == 是等价的
}
```

- == 用来比较基本类型的值是否相等或者两个对象的引用是否相等
- equals 用来比较两个对象是否相等

##### Q3.2：为什么要重写 equals() ?

- 因为默认equals在比较两个对象时，是看他们是否指向同一个地址的

  ```java
  // 若不重写，则p1和p2是两个不同的对象，则是不同的，但在我们的业务中，需要他们相同
  person p1 = new person(1,"name");
  person p2 = new person(1,"name");
  ```

##### Q3.3：如何重写 equals() ?

自反性：x.equals(x) 必须返回 true

对称性：如果x.equals(y)返回true，那么y.equals(x)也应该返回true

传递性：如果x.equals(y)返回true，y.equals(z)返回true，那么x.equals(z)也应该返回true

一致性：如果x和y引用的对象没有发生变化，那么反复调用x.equals(y)应该返回同样的结果

非空性：对于任意非空引用x，x.equals(null)应该返回false





#### Q4：private static native void registerNatives()

```java
private static native void registerNatives(); // 本地注册
static {
    registerNatives();
}
```

- 用**native关键字**修饰的函数表明该方法的实现并不是在Java中去完成，而是由C/C++去完成，并被编译成了与操作平台相关文件（**.dll**），由Java去调用。实际上java就是在不同的平台上调用不同的native方法实现对操作系统的访问的
- **静态初始化块**中调用了registerNatives()方法，在类首次进行加载的时候执行
- 使用private来修饰，表面私有的并不被外部调用
- 无方法体



#### Q5：public final native Class<?> getClass()

```java
Object i = new Integer(1);   // 注意返回的是 Integer
System.out.println(i.getClass()); // Integer.class 返回的是一个类对象（Class 类的对象） 
System.out.println(i.getClass().getName()); // 得到包内路径
System.out.println(i.getClass().getSimpleName());

// output
class java.lang.Integer
java.lang.Integer 
Integer
```

返回此对象（Object）的 **运行时类**（没有多态的，是静态解析的，编译时可以确定类型信息）

得到类对象后，可以获取这个类中的相关属性和方法



#### Q6：public native int hashCode()

- Object类的hashCode()在本地实现，JDK中许多包装类重写了hashCode方法

##### Q6.1： 为什么不直接通过hashCode产生唯一索引？

  - 很难有这种算法
  - 生成的hashCode非常大，可能会超出Java所能表示的范围







- instanceof 

  - 测试一个对象是否为一个类的实例

  - obj 必须为引用类型，不能是基本类型

  - obj 为 null    `null instanceof Object => false`

  - obj 为 class 接口的实现类   

    ```java
    ArrayList arrayList = new ArrayList();
    System.out.println(arrayList instanceof List);//true
    // 反过来也是返回 true
    List list = new ArrayList();
    System.out.println(list instanceof ArrayList);//true
    ```

  - obj 为 class 类的直接或间接子类

    ```java
    public class Person {}
    public class Man extends Person{}
    
    Person p1 = new Person();
    Person p2 = new Man();
    Man m1 = new Man();
    System.out.println(p1 instanceof Man);//false Man是Person的子类
    System.out.println(p2 instanceof Man);//true
    System.out.println(m1 instanceof Man);//true
    ```



**toString()**

- `getClass().getName() +"@"+ Integer.toHexString(hashCode())`

**notify() notifyAll() wait()**

**finalize()**

**private static native void registerNatives()**