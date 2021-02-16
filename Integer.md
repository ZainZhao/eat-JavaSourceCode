> Integer class based on jdk1.8 

[TOC]

## TODO

- 反射 https://blog.csdn.net/swadian2008/article/details/100679877



## Question & Answer

<img src="pic\Integer.png" alt="image-20210209105645213" style="zoom: 67%;" />



`public final class Integer extends Number implements Comparable<Integer>`

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

#### Q2.2：创建对象的五种方式

1. 使用new关键字  

   调用了构造方法

2. 使用Class类的newInstance()方法

   ```java
   Employee emp2 = (Employee)Class.forName("org.mitra.exercises.Employee").newInstance();
   
   //或者
   
   Employee emp2 = Employee.class.newInstance();
   ```

3. 使用Constructor类的newInstance方法

   ```java
   Constructor<Employee> constructor = Employee.class.getConstructor();
   Employee emp3 = constructor.newInstance();
   ```

   > 事实上Class的newInstance方法内部调用Constructor的newInstance方法

4. 使用clone方法

   `Employee emp4 = (Employee) emp3.clone();`

5. 使用反序列化

### Q3： public static String toString()

```java
// radix 进制
public static String toString(int i, int radix) {
    
    // 基数的范围为 2-36
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;

    /* Use the faster version */
    if (radix == 10) {
        return toString(i);
    }
	
    //int型为32位，加上符号位，最长可以为33位，故先创建一个33位的char数组
    char buf[] = new char[33];
    boolean negative = (i < 0);
    int charPos = 32;

    if (!negative) { // 将正数转换为负数
        i = -i;
    }
	
    // 至于为什么要将正数转为负数来比较:
    // int型的最大值为2147483647，最小值为 -2147483648
    // 如果将负数转为正数，最小值在转时会溢出，故使用负数来进行运行
    
    while (i <= -radix) {
        buf[charPos--] = digits[-(i % radix)];
        i = i / radix;
    }
    buf[charPos] = digits[-i];

    if (negative) {
        buf[--charPos] = '-';
    }
    // 最后在根据buf数组、数组有效值位置charPos以及有效长度33-charPos创建一个string对象
    return new String(buf, charPos, (33 - charPos));
}
```

### Q：构造方法

```java
public Integer(int value) {
     this.value = value;
}

public Integer(String s) throws NumberFormatException {
    this.value = parseInt(s, 10);
}
```




### Q：getChar()







### Q：private final int value

一旦一个Integer对象被初始化之后，就无法再改变value的值



### Q：自动装箱和自动拆箱 & valueOf()

- 八种基本类型都对应着相应的包装类型

- `==`  比较时会自动拆箱，`equals` 比较时不会拆箱，`+` 等算数运算会先拆箱再装箱

- 装箱就是自动将基本数据类型转换为包装器类型；拆箱就是自动将包装器类型转换为基本数据类型

  ```java
  Integer total = 99
  装箱 实际调用：Integer total = Integer.valueOf(99)    
      
  int totalPrim = total    
  拆箱 实际调用： int totalPrim = total.intValue()
  
  public static Integer valueOf(int i) {
          if (i >= IntegerCache.low && i <= IntegerCache.high)
              return IntegerCache.cache[i + (-IntegerCache.low)];
          return new Integer(i); 
  }
  ```
### Q：private static class IntegerCache

```java
 private static class IntegerCache {
     	// 缓存的下界，-128，不可变
        static final int low = -128;
        //缓存上界，暂为null
        static final int high;
        //缓存的整型数组
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            
            // 可修改 -XX:AutoBoxCacheMax
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127); // 默认最小的 max 值
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1); // //确保cache数组的大小不超过Integer的最大限度
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];  // 创建缓存数组，给定大小
            int j = low;
            for(int k = 0; k < cache.length; k++) 
                cache[k] = new Integer(j++); // 初始化缓存数组

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

当系统需要频繁使用Integer时，或者说堆内存中存在大量的Integer对象时，可以考虑提高Integer缓存上限，避免JVM重复创造对象，提高内存的使用率，减少GC的频率，从而提高系统的性能。

- 大部分Integer对象通过Integer.valueOf()产生。说明代码里存在大量的拆箱与装箱操作。这时候设置这个参数会系统性能有所提高。
- 大部分Integer对象通过反射，new产生。这时候Integer对象的产生大部分不会走valueOf()方法，所以设置这个参数也是无济于事。



### Q：Integer 创建对象的几种方式

```java
1 常量池  -128 ---- 127
 		Integer t1 = 99;
        Integer t2 = 99;
        System.out.println(t1 == t2);//true  

2 堆
     	Integer t3 = 128;
        Integer t4 = 128;
        System.out.println(t3 == t4);//false

3 new
    	Integer n1 = new Integer(99);//堆
        Integer n2 = new Integer(99);//堆
        System.out.println(n1 == n2);//fasle

4 valueOf
        Integer tt1 = Integer.valueOf(99);//常量池
        Integer tt2 = Integer.valueOf(99);//常量池
        System.out.println(tt1 == tt2);//true  

        Integer tt3 = Integer.valueOf(128);//堆
        Integer tt4 = Integer.valueOf(128);//堆
        System.out.println(tt3 == tt4);//fasle  

// 基本类型
Integer.parseInt("123")
```



### Q：int 和 Integer 的区别

```markdown
- 包装类与数据类型（对象与基本数据类型的区别）
- Integer的默认值是null，int的默认值是0
```

