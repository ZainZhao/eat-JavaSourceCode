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

- 重写的 equals 方法应该遵守 **自反性、对称性、传递性、一致性**

##### Q3.2：如何重写 equals() ?





#### Q4：private static native void registerNatives()

```java
private static native void registerNatives(); // 本地注册
static {
    registerNatives();
}
```

- 用**native关键字**修饰的函数表明该方法的实现并不是在Java中去完成，而是由C/C++去完成，并被编译成了与操作平台相关文件（**.dll**），由Java去调用
- **静态初始化块**中调用了registerNatives()方法，在类首次进行加载的时候执行
- 使用private来修饰，表面私有的并不被外部调用
- 无方法体



#### Q5：







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









**public final native Class<?>getClass()**

- native 修饰的方法是由操作系统帮忙实现的
- calss是类的一个属性，能获取该类编译时的类对象。没有多态的，是静态解析的，编译时可以确定类型信息。
- getClass() 是一个类的方法，它获取该类的类对象。有多态能力，运行时可以返回子类的类型信息。
- ` Class<?>` 这样也能编译通过 `Class<? extends String> c = "".getClass()`

**public native inthashCode()**

- 为避免hash冲突，也就是算法获得的元素要尽量均匀分布
- JDK中许多包装类也重写了hashCode方法
- 算法获得的元素应该尽量均匀分布
- 为什么不直接通过hashCode产生唯一索引？
  - 很难有这种算法
  - 生成的hashCode非常大，可能会超出Java所能表示的范围

- String 类的 hashCode

  ```java
  public int hashCode() {
      int h = hash; // 默认值是0
      if (h == 0 && value.length > 0) {
          char val[] = value; // 字符串对应的char数组
  
          for (int i = 0; i < value.length; i++) {
              h = 31 * h + val[i]; // val[0]*31^(n-1) + val[1]*31^(n-2) + ... + val[n-1]
          }
          hash = h;
      }
      return h;
  }
  ```

- 对于需要大量并且快速的对比，如果都用equals()去实现显然效率太低，所以可以考虑先比较hashCode

**toString()**

- `getClass().getName() +"@"+ Integer.toHexString(hashCode())`

**notify() notifyAll() wait()**

**finalize()**

**private static native void registerNatives()**