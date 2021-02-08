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

##### Q3.4：自定义类中完整的 equals() ?

```java
// Person 类
public boolean euqals(Object otherObject) {
    // 1 比较引用
    if (this == otherObject) {
        return true
    }
    
    // 2
    if (null == otherObject) {
        return false;
    }
    
    // 3.1 如果 equals 的语义在每个子类中都有所改变，就使用 getClass() 检测
    if (getClass() != otherObject.getClass()) {
        return false;
    }
    
    // 3.2 如果所有子类都有统一的定义，就使用 instanceof 检测
    if (!(otherObejct instanceof Person)) {
        return false;
    }
    
    // 4 将 otherObject 转换成对应的类型变量
    Person other = (Person)otherObject;  // 因为这个方法是 Person 类的，所以转为 Person 对象
    
    // 5 最后，对对象的属性进行比较    
    
}

```

> 如果用自定义对象作为集合中的key，那么一定要重写 equals() 和 hashCode() 



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

```java
Class<? extend String> c = “”.getClass();
```

类型为T的变量getClass()的返回值类型为 `Class<? extend T>`，而非getClass()方法声明中的 `Class<?>`



#### Q6：public native int hashCode()

- Object类的hashCode()在本地实现，JDK中许多包装类重写了hashCode方法
- hashcode() 返回的不是对象在内存中的地址，应该是做了一些处理之后的

##### Q6.1： 为什么不直接通过hashCode产生唯一索引？

  - 很难有这种算法
  - 生成的hashCode非常大，可能会超出Java所能表示的范围



#### Q7：protected native Object clone()

```java
protected native Object clone() throws CloneNotSupportedException;
```

一个类在覆盖clone（）方法时，需修改成public访问修饰符，保证其他类都能够访问

一个类想要覆盖clone（）方法，必须本身实现 `java.lang.Cloneable` 接口，否则会抛出 `CloneNotSupportedException `异常

##### Q7.1：浅拷贝与深拷贝

**浅拷贝**（shallow copy）：只是增加了一个指针指向已存在的内存地址

<img src="pic\浅拷贝.png" alt="image-20210130174653973" style="zoom: 25%;" />

- 只会将**对象的各个属性**进行复制，并不会进行递归复制
- 如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址

```java
try {
    // 直接调用父类的clone()方法
    return super.clone();
} catch (CloneNotSupportedException e) {
    return null;
}
```

- 对象拷贝后没有生成新的对象，二者的对象地址是一样的；而浅拷贝的对象地址是不一样的。

```java
Student studentB = (Student) studentA.clone() // 浅拷贝
Student studentB = studentA  // 对象拷贝
```

**深拷贝**（deep copy）：增加了一个指针并且申请了一个新的内存，使这个指针指向这个新的内存

<img src="pic\深拷贝.png" alt="image-20210130174746468" style="zoom:25%;" />

- 在拷贝引用类型成员变量时，为引用类型的数据成员另辟了一个独立的内存空间，实现真正内容上的拷贝，花销较大

- 实现方法
  - 在递归路线上的所有引用属性都实现 `supper.clone()`
  - 序列化
  
  ```java
  public Object deepClone() throws Exception {
      // 序列化
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream oos = new ObjectOutputStream(bos);
      oos.writeObject(this);
      
      // 反序列化
      ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
      ObjectInputStream ois = new ObjectInputStream(bis);
      
      return ois.readObject();
      
  }
  
  ```
  
  

##### Q7.2：基本数据类型 & 引用类型

基本数据类型：char、boolean、byte、short、int、long、float、double

引用数据类型：类、接口、数组、枚举





#### **Q8：public String toString()**

```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```



#### Q9：protected void finalize()

```java
 protected void finalize() throws Throwable { }
```

- 当对象的内存不再被使用时，GC时才调用。一旦垃圾收集器准备好释放对象占用的存储空间，首先调用finalize()，而且只有在下一次垃圾收集过程中，才会真正回收对象的内存

- `Runtime.getRuntime().runFinalization()`等方法建议GC去调用这个方法，但还是不能保证一定会被调用。 Java语言规范中不仅不保证终结方法会被及时地执行，而且根本不保证他们会被执行。当一个程序终止时，某些已经无法访问的对象上的终结方法却根本没有执行，这是完全有可能的，因此，**不应该依赖终结方法来更新重要的持久状态**。
- 回收没有使用new关键字分配的内存，那么允许程序员根据前面特殊的分配方法去执行一个**收尾机制**。



#### Q10：instanceof

- 测试一个对象是否为一个类的实例

- obj 必须为引用类型，不能是基本类型

- obj 为 null    `null instanceof Object => false`

- obj 为 class 类的直接或间接子类

  ```java
  public class Person {}
  public class Man extends Person{}
  
  Person p1 = new Person();
  Person p2 = new Man();
  Man m1 = new Man();
  System.out.println(p1 instanceof Man);//false Man是Person的子类
  System.out.println(p2 instanceof Man);//true  因为这是引用
  System.out.println(m1 instanceof Man);//true
  ```











TODO

notify() notifyAll() wait()





