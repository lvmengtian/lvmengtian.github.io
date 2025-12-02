---
title: Java基础
createTime: 2025/08/20 21:55:46
permalink: /java-guide/hg5e8awy/
---
## java的基本类型以及取值范围
### 基本类型
Java包含8种基本类型，分别是：
- 整数类型：byte、short、int、long
- 浮点数类型：float、double
- 字符类型：char
- 布尔类型：boolean
### 取值范围
想了解这些类型的取值范围，就要先知道不同类型所占用的字节数。在此之前我们先了解一下字节的概念。
在计算机存储中，最小的存储单位是字节（Byte）。一个字节包含8位二进制数，每个二进制数就是1bit；
即`1Byte = 8bit`。一个字节取值范围使用二进制表示就是 `00000000~11111111`，换成十进制就是`0 ~ 255`。
现在我们再来看不同类型所占的字节数以及取值范围：
| 类型描述 | 类型 | 字节数 | 取值范围 |
|---|---|---|---|
| 字节型 |	byte |	1（8bit）|	-128~127（-2^7^～2^7^-1）|
| 短整型 | short |	2（16bit）|	-32768~32767（-2^15^～2^15^-1）|
|整型|	int |	4（32bit）|	-2^31^～2^31^-1 |
|长整型|	long |	8（64bit） |	-2^63^～2^63^-1 |
|单精度浮点型|	float |	4（32bit）	|
|双精度浮点型|	double |	8（64bit）	|
|字符型 | char |	2（16bit）	| |
|布尔型|	boolean |	1（8bit）	| true 或 false|

> **<font color="red">是如何确定取值范围的？（以byte为例）</font>**
> 
> 因为Java是有符号的，所以最高位是用来表示符号的，（0-表示正数 1-表示负数）。所以真正表示数值的就只有7个二进制位。所以取值范围的二进制表示理应是`11111111 ~ 01111111`，即`-128 ~ 128`。但是，数值在计算机中是以补码的方式存储的。什么是补码？ 了解了什么是补码后，会发现01111111是无法表达出来的，所以需要减去1，即变成`01111110`。也就是byte的取值范围是`-128 ~ 127`

## 面向对象的基本特性？
面向对象基本特性是：**封装**、**集成**、**多态**
1. **封装**：将数据（属性）和操作数据的方法（行为）封装在一起，形成一个独立的“对象”。同时，对外部隐藏对象的内部实现细节，只暴露有限的、安全的接口进行交互。
2. **继承**：允许一个类（子类、派生类）继承另一个现有类（父类、基类、超类）的属性和方法。子类可以复用父类的功能，并可以扩展出自己特有的属性和方法。
3. **多态**：指同一个行为（方法）具有多个不同表现形式或形态的能力。具体来说，允许父类的引用指向子类的对象，并且根据这个引用指向的实际对象类型来调用相应的方法。
::: collapse
- :+示例代码

  ```java
  // 父类
  public class Animal {
      // 【封装】属性被封装到Animal里面，外部不可以直接访问；必须通过getName方法访问
      private String name;

      public Animal(String name) {
          this.name = name;
      }

      public String getName() {
        return this.name;
      }

      public void eat() {
          System.out.println(name + " is eating.");
      }

      public void sleep() {
          System.out.println(name + " is sleeping.");
      }
  }

  // 【继承】子类 Dog 继承自 Animal
  public class Dog extends Animal { // 使用 extends 关键字
      public Dog(String name) {
          super(name); // 调用父类的构造方法
      }

      // 子类特有的方法
      public void bark() {
          System.out.println(getName() + " says: Woof!");
      }

      // 【多态】子类重写父类的eat方法，让其具备子类的特点
      public void eat() {
          System.out.println("Dog is eating.");
      } 
  }
  // 使用
  Dog myDog = new Dog("Buddy");
  myDog.eat();   // 继承自Animal的方法
  myDog.sleep(); // 因为Dog重写了，所以这里是Dog自己的方法
  myDog.bark();  // Dog自己的方法
  ```
:::

## 什么是强引用、软引用、弱引用、虚引用？
我们来详细讲解一下Java中的四种引用类型：强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）和虚引用（Phantom Reference）。

这四种引用类型的强度依次递减，它们与Java的垃圾回收（Garbage Collection, GC）行为密切相关，主要用于管理对象的内存生命周期，是实现内存敏感缓存、防止内存泄漏的强大工具。

---

### 1. 强引用 (Strong Reference)

**定义**：
强引用是程序中最普遍、默认的引用类型。我们平常使用`new`关键字创建的对象，赋值给一个变量，这个变量就是该对象的**强引用**。

**语法**：
```java
Object obj = new Object(); // obj 就是新Object对象的强引用
```

**特点与作用**：
*   **可达性**：只要强引用还存在（即引用变量`obj`还在作用域内且没有被显式地赋值为`null`），垃圾收集器就**永远不会**回收掉被引用的对象。
*   **内存泄漏**：如果一段不必要的强引用长期存在（例如，一个静态Map缓存了不再使用的对象），会导致这些对象无法被回收，从而引发**内存泄漏**。
*   **作用**：构成程序正常运行的基石，用于表示那些“必须存活”的对象。

---

### 2. 软引用 (Soft Reference)

**定义**：
软引用用来描述一些**还有用但并非必需**的对象。

**语法**：
```java
// 创建强引用
Object strongRef = new Object();
// 使用强引用创建软引用
SoftReference<Object> softRef = new SoftReference<>(strongRef);

// 为了让它能被回收，取消强引用
strongRef = null;

// 之后可以通过softRef.get()来尝试获取对象
Object obj = softRef.get();
if (obj != null) {
    // 对象尚未被回收
} else {
    // 对象已被回收
}
```

**特点与作用**：
*   **回收时机**：在系统**内存充足**时，垃圾收集器不会回收软引用关联的对象。
*   **内存敏感**：当系统**内存不足**，即将发生`OutOfMemoryError`之前，垃圾收集器会把这些仅被软引用关联的对象列入回收范围进行第二次回收。如果这次回收后还是没有足够的内存，才会抛出内存溢出异常。
*   **作用**：非常适合实现**内存敏感的缓存**。例如，缓存图片、大文本等数据。当内存吃紧时，缓存会被自动释放，避免OOM；当内存充足时，缓存又能提高性能。
    *   `SoftReference` 非常适合用来做缓存，例如缓存图片等大对象。

---

### 3. 弱引用 (Weak Reference)

**定义**：
弱引用用来描述**非必需**的对象，但其强度比软引用更弱。

**语法**：
```java
Object strongRef = new Object();
WeakReference<Object> weakRef = new WeakReference<>(strongRef);
strongRef = null; // 取消强引用

// 强制触发GC（仅作演示，生产中慎用）
System.gc();

// GC后，weakRef.get()有很大概率返回null
if (weakRef.get() != null) {
    // 侥幸存活
} else {
    // 已被回收
}
```

**特点与作用**：
*   **回收时机**：**无论内存是否充足**，只要垃圾收集器开始工作，并且发现对象**只被弱引用关联**（没有任何强引用或软引用关联它），就会立刻回收该对象。
*   **作用**：
    1.  **规范化映射（Canonicalized Mapping）**：最经典的用途是在`WeakHashMap`类中。在`WeakHashMap`中，键（Key）是弱引用的。一旦某个键对象在外面没有强引用了，它就会被GC回收，然后这个键值对也会自动从`WeakHashMap`中被移除。这完美地解决了Map生命周期长于Key对象而引发的内存泄漏问题。
    2.  **监控对象**：用于构建一种“监视”结构，该结构不应阻止其键/元素被回收。

---

### 4. 虚引用 (Phantom Reference)

**定义**：
虚引用也称为“幽灵引用”或“幻影引用”，是**最弱**的一种引用关系。一个对象是否有虚引用存在，完全不会对其生存时间构成影响。

**语法**：
虚引用必须和**引用队列（ReferenceQueue）** 联合使用。

```java
ReferenceQueue<Object> queue = new ReferenceQueue<>();
Object strongRef = new Object();
PhantomReference<Object> phantomRef = new PhantomReference<>(strongRef, queue);

strongRef = null;

// ... 之后某个时间
// 强制GC
System.gc();

// 检查引用队列
Reference<?> ref = queue.poll();
if (ref != null) {
    // 这意味着ref所指的对象已经被GC回收了
    // 可以在这里执行一些清理工作，例如关闭该对象持有的本地资源
}
```

**特点与作用**：
*   **无法获取对象**：`phantomRef.get()`方法**总是返回`null`**。这意味着你根本无法通过虚引用来获取对象的实例。
*   **回收跟踪**：虚引用的唯一目的就是**跟踪对象被垃圾回收的事件**。当垃圾收集器准备回收一个对象时，如果发现它还有虚引用，就会在回收该对象**之后**，将这个虚引用加入到与之关联的引用队列中。
*   **作用**：
    *   用于在对象被GC**之后**，收到一个系统通知。
    *   主要用于执行一些**复杂的、特殊的资源清理工作**。例如，在NIO中，`DirectByteBuffer`对象分配的是堆外内存，Java的GC管不到这块内存。它就可以使用虚引用来进行跟踪：当`DirectByteBuffer`对象被回收后，相应的虚引用会被放入队列，然后后台的清理线程（如`Cleaner`）可以从队列中获取到这个通知，进而释放掉那块堆外内存，防止内存泄漏。

---

### 总结对比

| 引用类型 | 强度 | 被垃圾回收的时机 | `get()` 方法 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| **强引用** | 最强 | **永不回收**（只要强引用存在） | 返回对象本身 | 程序默认引用，所有正常对象。 |
| **软引用** | 次之 | **内存不足时**进行回收 | 返回对象本身（若未被回收） | 实现内存敏感的高速缓存。 |
| **弱引用** | 较弱 | **下一次GC时**即被回收 | 返回对象本身（若未被回收） | 实现规范化映射（如`WeakHashMap`），防止内存泄漏。 |
| **虚引用** | 最弱 | **不影响生命周期**，用于跟踪回收事件 | **总是返回null** | 在对象被回收后收到通知，进行特殊的资源清理。 |

**核心思想**：通过使用不同强度的引用，开发者可以与垃圾收集器进行“沟通”，更精细地控制对象的生命周期，从而在实现特定功能（如缓存）的同时，有效避免内存泄漏的风险。

## Integer、Long里面的cache有什么区别？
通过直接查看IntegerCache和LongCache的源码就可发现区别。
::: code-tabs

@tab IntegerCache源码

  ```java
    /**
  * Cache to support the object identity semantics of autoboxing for values between
  * -128 and 127 (inclusive) as required by JLS.
  *
  * The cache is initialized on first usage.  The size of the cache
  * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
  * During VM initialization, java.lang.Integer.IntegerCache.high property
  * may be set and saved in the private system properties in the
  * sun.misc.VM class.
  */

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
              try {
                  int i = parseInt(integerCacheHighPropValue);
                  i = Math.max(i, 127);
                  // Maximum array size is Integer.MAX_VALUE
                  h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
              } catch( NumberFormatException nfe) {
                  // If the property cannot be parsed into an int, ignore it.
              }
          }
          high = h;

          cache = new Integer[(high - low) + 1];
          int j = low;
          for(int k = 0; k < cache.length; k++)
              cache[k] = new Integer(j++);

          // range [-128, 127] must be interned (JLS7 5.1.7)
          assert IntegerCache.high >= 127;
      }

      private IntegerCache() {}
  }
  ```

@tab LongCache源码

  ```java
    private static class LongCache {
      private LongCache(){}

      static final Long cache[] = new Long[-(-128) + 127 + 1];

      static {
          for(int i = 0; i < cache.length; i++)
              cache[i] = new Long(i - 128);
      }
  }
  ```
:::
**通过源码可以看出Long的缓存就是固定的[-128, 127]的范围；Integer的缓存默认是[-128, 127]，但是上限可以通过参数进行配置。**

## 接口和抽象类的区别？

### 核心概念

*   **抽象类**：是对一类事物的**抽象**，即对**类的抽象**。它表示的是“**is-a**”（是一个）的关系。例如，`Animal`是一个抽象类，`Dog` *是一个* `Animal`。
*   **接口**：是对行为的**抽象**，即对**行为的抽象**。它表示的是“**can-do**”（能做什么）的关系。例如，`Flyable`是一个接口，`Bird` *能* `Fly`（实现`Flyable`接口）。

---

### 详细区别对比

| 特性 | 抽象类 (Abstract Class) | 接口 (Interface) (Java 8以前) | 接口 (Java 8及以后) |
| :--- | :--- | :--- | :--- |
| **定义关键字** | `abstract class` | `interface` | `interface` |
| **成员变量** | 可以是**任何**类型的变量（普通变量、静态常量、静态变量） | **默认**且**只能是** `public static final` 的常量 | **默认**且**只能是** `public static final` 的常量 |
| **构造方法** | **有**构造方法（用于子类初始化） | **没有**构造方法 | **没有**构造方法 |
| **方法实现** | 可以包含**抽象方法**和**具体实现的方法** | 所有方法都是**抽象方法**（无方法体） | 可以包含 **`default`方法**、**`static`** 方法和**抽象方法** |
| **继承** | 使用 `extends` 关键字**单继承**（一个子类只能继承一个抽象类） | 使用 `implements` 关键字**多实现**（一个类可以实现多个接口） | 使用 `implements` 关键字**多实现** |
| **设计目的** | **代码复用**和**定义模板**。提供基础实现，要求子类“是一个”父类。 | **定义契约**和**声明能力**。要求实现类“能做什么”，不关心如何实现。 | **定义契约**、**声明能力**并**提供默认实现**（向后兼容）。 |
| **访问修饰符** | 方法可以使用任意访问修饰符（`public`, `protected`, `private`） | 方法**默认**为 `public abstract`（Java 9起也可为 `private`) | 方法可以是 `public abstract`, `default`, `public static`, `private` (Java 9+) |

---

### 代码示例

::: code-tabs
@tab 抽象类示例
```java
// 抽象类：表示一种“是”的关系
abstract class Animal {
    // 成员变量（可以是普通的）
    protected String name;
    
    // 构造方法
    public Animal(String name) {
        this.name = name;
    }
    
    // 具体实现的方法（代码复用）
    public void eat() {
        System.out.println(name + " is eating.");
    }
    
    // 抽象方法（由子类实现）
    public abstract void makeSound();
}

class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says: Woof!");
    }
}
```

@tab 接口示例

```java
// 接口：表示一种“能”的关系
interface Flyable {
    // 常量 (默认 public static final)
    int MAX_ALTITUDE = 10000;
    
    // 抽象方法 (默认 public abstract)
    void fly();
    
    // Default方法 (Java 8+ 提供默认实现，用于向后兼容)
    default void takeOff() {
        System.out.println("Taking off...");
    }
    
    // Static方法 (Java 8+)
    static boolean canFly(Object obj) {
        return obj instanceof Flyable;
    }
}

interface Swimmable {
    void swim();
}

// 一个类可以实现多个接口，表示多种能力
class Duck extends Animal implements Flyable, Swimmable {
    public Duck(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says: Quack!");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying not very high.");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " is swimming in the pond.");
    }
}
```
:::

### 如何选择？

根据你的设计目的来决定使用接口还是抽象类：

*   **使用抽象类，当：**
    *   你想在多个紧密相关的类之间**共享代码**（提供公共的方法实现）。
    *   你需要定义**非`public`的成员变量或方法**（`protected`, `private`）。
    *   你需要定义**状态**（成员变量），而不仅仅是行为。

*   **使用接口，当：**
    *   你想定义**一个契约**，让**不相关**的类都能实现它。例如，`Comparable`和`Serializable`接口可以被任何类实现。
    *   你希望实现**多重继承**（多重行为）。
    *   你只想定义**行为**，而不关心具体的实现（在Java 8之前），或者想为这些行为提供默认实现（Java 8之后）。


## String、StringBuilder、StringBuffer的区别？

### 核心区别总结

| 特性 | `String` | `StringBuilder` | `StringBuffer` |
| :--- | :--- | :--- | :--- |
| **可变性** | **不可变** | **可变** | **可变** |
| **线程安全** | 安全（因为不可变） | **不安全** | **安全**（方法有 `synchronized` 修饰） |
| **性能** | **最低**（频繁修改会产生大量垃圾对象） | **最高** | **中等**（因线程安全开销略低于StringBuilder） |
| **使用场景** | 操作少量的字符串，或需要保持常量性 | 单线程下频繁进行字符串修改 | 多线程下频繁进行字符串修改 |

---

### 详细解析

#### 1. String（不可变字符串）

*   **核心特点：不可变性**
    一旦一个 `String` 对象被创建，它的值就无法被改变。任何看似修改它的操作（如 `concat()`, `+`, `substring()`），实际上都是**创建了一个全新的 `String` 对象**，而原来的对象依然存在于内存中。

*   **性能影响：**
    在循环中进行字符串拼接等操作时，会产生大量中间临时字符串对象，不仅性能低下，还会给垃圾回收器带来压力。
    ```java
    String str = "Hello";
    for (int i = 0; i < 1000; i++) {
        str += i; // 每次循环都会 new 一个新的String对象，极其低效！
    }
    ```

*   **线程安全：**
    因为不可变，所以天然线程安全。多个线程可以同时读取一个字符串，而不会产生任何问题。

#### 2. StringBuilder（可变字符串，非线程安全）

*   **核心特点：可变性 & 高性能**
    `StringBuilder` 内部维护了一个可变的字符数组（`char[] value`）。当进行追加、插入、删除等操作时，**直接在原数组上进行修改**，只有在容量不足时才会进行扩容（创建一个新的更大数组）。这避免了大量临时对象的创建。

*   **性能影响：**
    在**单线程**环境下，进行频繁的字符串修改，其性能远高于 `String`。

*   **线程安全：**
    **非线程安全**。它的方法没有使用同步(`synchronized`)锁，因此效率更高，但不能在多线程环境下共享使用，否则可能导致数据不一致。

#### 3. StringBuffer（可变字符串，线程安全）

*   **核心特点：可变性 & 线程安全**
    `StringBuffer` 可以看作是 `StringBuilder` 的线程安全版本。它们的功能API几乎完全一样。

*   **性能影响：**
    为了保证线程安全，它的所有公开方法都使用了 `synchronized` 关键字进行同步。这带来了额外的**线程安全开销**，导致其性能在单线程环境下**低于 `StringBuilder`**。

*   **线程安全：**
    **线程安全**。可以在多线程环境下安全使用。

---

### 性能差异深度分析

性能排序（在字符串修改操作中）： **`StringBuilder` > `StringBuffer` > `String`**

**为什么？**

1.  **`String` vs 可变类（`StringBuilder/Buffer`）**：
    *   `String` 的每次修改都涉及新对象的创建、数组拷贝和旧对象的回收。
    *   可变类只在必要时（容量不足时）扩容，大部分操作都是原地修改，开销小得多。

2.  **`StringBuilder` vs `StringBuffer`**：
    *   `StringBuffer` 的 `synchronized` 方法在调用时需要获取和释放**锁（monitor lock）**，即使在单线程无竞争的情况下，这个操作也有一定的性能损耗。
    *   `StringBuilder` 没有这个开销，因此速度更快。Java 虚拟机（JVM）可以对 `StringBuilder` 的方法进行更好的优化（如内联）。

**一个简单的性能测试示例：**
```java
public class PerformanceTest {
    public static void main(String[] args) {
        int count = 100000;

        // String 测试
        long startTime = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < count; i++) {
            str += i;
        }
        long endTime = System.currentTimeMillis();
        System.out.println("String 耗时: " + (endTime - startTime) + " ms");

        // StringBuilder 测试
        startTime = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < count; i++) {
            sb.append(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuilder 耗时: " + (endTime - startTime) + " ms");

        // StringBuffer 测试
        startTime = System.currentTimeMillis();
        StringBuffer sbf = new StringBuffer();
        for (int i = 0; i < count; i++) {
            sbf.append(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuffer 耗时: " + (endTime - startTime) + " ms");
    }
}
```
运行结果会清晰显示性能差距（`String` 会慢好几个数量级）。

---

### 使用场景推荐

1.  **使用 `String`：**
    *   **不需要改变**的字符串常量或字符串字面量（例如，定义消息、配置键等）。
    *   字符串操作非常少的情况。
    *   因为其不可变性，作为 `HashMap` 的键非常安全。

2.  **使用 `StringBuilder` (首选的可变字符串类)：**
    *   **单线程**环境下，需要**频繁进行字符串拼接、修改、删除**等操作。
    *   例如：在循环中动态构建SQL语句、拼接URL参数、处理大型文本等。
    *   **95% 的情况你都应该使用它**，因为大部分字符串操作都发生在方法内部（局部变量，线程安全）。

3.  **使用 `StringBuffer` (已逐渐淡出)：**
    *   **多线程**环境下，需要频繁修改字符串，并且需要保证线程安全。
    *   例如：多个线程同时操作同一个字符串缓冲区。
    *   **注意**：在现代Java开发中，即使是在多线程环境下，也往往可以通过线程隔离（如每个线程使用自己的 `StringBuilder`）或其他同步机制来避免直接使用 `StringBuffer`，因为它的性能开销通常是不必要的。

### 最佳实践

*   **简单的字符串拼接**：如果拼接操作很少，可以直接用 `+`，编译器可能会自动优化成 `StringBuilder`。
*   **复杂的字符串构建**：**毫不犹豫地使用 `StringBuilder`**。
*   **初始大小**：如果可以预估最终字符串的大致长度，在创建 `StringBuilder` 或 `StringBuffer` 时指定初始容量（如 `new StringBuilder(1024)`），可以减少扩容次数，进一步提升性能。
*   **原则**：**除非有明确的、必须的多线程共享修改需求，否则永远优先使用 `StringBuilder`。**

## Java异常分类？可检查异常和不可检查异常有什么区别？
### 一、Java异常的分类体系

Java中所有的异常和错误都有一个共同的根父类：Throwable。它有两个直接子类：Error 和 Exception。而 Exception 又分为两大类：可检查异常（Checked Exception） 和 不可检查异常（Unchecked Exception），其中不可检查异常主要指 RuntimeException 及其子类

![123](/images/throwable-class.png)
---
### 二、各类异常详解

#### 1. Error (错误)
*   **定义**：`Error` 及其子类表示的是应用程序**无法处理**的严重问题，通常与代码编写者无关，而是JVM运行时系统本身的问题（如系统资源耗尽）。
*   **特点**：
    *   是 **不可检查异常（Unchecked）**，编译器不会要求你捕获或声明它们。
    *   应用程序**不应该尝试捕获**（`catch`）它们，因为即使捕获了，通常也无法让程序恢复正常。
    *   发生`Error`时，JVM通常会终止线程甚至整个虚拟机。
*   **常见例子**：
    *   `OutOfMemoryError`：内存耗尽。
    *   `StackOverflowError`：栈溢出（如无限递归）。
    *   `VirtualMachineError`：虚拟机错误。

#### 2. Exception (异常)
`Exception` 及其子类表示的是程序本身**可以处理**的问题。这是我们关注的重点，它分为两类：

**A. 可检查异常 (Checked Exception)**
*   **定义**：除了 `RuntimeException` 以外的所有 `Exception` 的子类。
*   **特点**：
    *   **编译器会检查（Check）** 它们。如果一个方法可能抛出可检查异常，编译器会**强制**要求方法的使用者要么用 `try-catch` 块捕获它，要么用 `throws` 关键字在方法声明中抛出它。**不处理就无法通过编译**。
    *   通常表示程序**可以预料且可恢复**的问题，例如用户输入错误、文件不存在、网络连接中断等。
*   **常见例子**：
    *   `IOException`及其子类（如`FileNotFoundException`）
    *   `SQLException`
    *   `ClassNotFoundException`

**B. 不可检查异常 (Unchecked Exception)**
*   **定义**：主要指 `RuntimeException` 及其子类。
*   **特点**：
    *   编译器**不会检查**它们。即使方法抛出了不可检查异常，方法声明上也**不需要**用 `throws` 声明，调用者也**不需要**强制捕获。它们会在运行时自动抛出。
    *   通常表示程序中的**逻辑错误**或**编程错误**，是程序员应该避免而不是去捕获的bug。
*   **常见例子**：
    *   `NullPointerException`：空指针异常。
    *   `ArrayIndexOutOfBoundsException`：数组下标越界。
    *   `IllegalArgumentException`：非法参数异常。
    *   `ArithmeticException`：算术异常（如除以零）。
    *   `ClassCastException`：类型转换异常。

---

### 三、可检查异常 vs 不可检查异常：核心区别

| 特性 | 可检查异常 (Checked Exception) | 不可检查异常 (Unchecked Exception) |
| :--- | :--- | :--- |
| **检查时机** | **编译时检查** | **运行时检查** |
| **处理强制性** | **必须**被捕获或声明抛出，否则**编译错误** | 不强制处理，可选择性捕获 |
| **包涵类型** | `Exception` 的子类（不包括`RuntimeException`) | `RuntimeException` 和 `Error` |
| **设计初衷** | 表示**可预见的、可恢复的**问题（如外部错误） | 表示**程序员的逻辑错误**或**不可控的系统错误** |
| **代码示例** | `try-catch` 或 `throws` | 通常不处理，由程序员修复代码逻辑 |

---

### 四、如何处理异常？（代码示例）

#### 1. 处理可检查异常（两种方式）

**方式一：使用 `try-catch` 块捕获并处理**
```java
try {
    FileReader file = new FileReader("somefile.txt"); // 可能抛出FileNotFoundException
    // ... 读取文件操作
} catch (FileNotFoundException e) {
    System.out.println("文件没找到，请检查路径！");
    e.printStackTrace(); // 打印异常堆栈跟踪，有助于调试
}
```

**方式二：使用 `throws` 声明抛出，交给调用者处理**
```java
public void readFile() throws FileNotFoundException { // 在方法签名中声明
    FileReader file = new FileReader("somefile.txt");
    // ...
}
// 此时，调用readFile()的方法就必须处理这个异常了。
```

#### 2. 处理不可检查异常（通常不强制处理，但可以捕获）

```java
// 这段代码能通过编译，但运行时会抛出ArithmeticException
public void calculate() {
    int result = 10 / 0;
}

// 可以选择性捕获，但这不是必须的
public void calculateSafely() {
    try {
        int result = 10 / 0;
    } catch (ArithmeticException e) {
        System.out.println("除数不能为零！");
    }
}
// 更好的做法是修复逻辑，在除法前检查除数
public void calculateBetter(int divisor) {
    if (divisor == 0) {
        System.out.println("除数不能为零！");
        return;
    }
    int result = 10 / divisor;
}
```

### 五、如何选择？最佳实践

1.  **对于可检查异常**：如果你知道如何从错误中**恢复**（如提示用户重新输入文件名），就用 `try-catch`。如果不知道如何处理，只是想向上汇报，就用 `throws`。

2.  **对于不可检查异常**：首要任务不是去捕获它们，而是**通过代码审查和测试来避免它们**（例如，在使用对象前检查是否为`null`，检查数组下标和除数等）。在系统的顶层（如Controller层或主方法）可以有一个全局的异常处理器来捕获所有未处理的异常，给用户一个友好的错误提示，而不是让程序崩溃。

3.  **不要生吞异常（Swallowing Exceptions）**：空的 `catch` 块是极其危险的，它隐藏了错误，让程序在未知的状态下运行，使得调试变得极其困难。
    ```java
    // ❌ 绝对不要这样做！
    try {
        // ... 某些操作
    } catch (Exception e) {
        // 什么都不做
    }
    ```

4.  **具体优于泛化**：捕获异常时，应优先捕获最具体的异常类型，而不是直接捕获通用的 `Exception`。
    ```java
    try {
        // ... 某些操作
    } catch (FileNotFoundException e) { // 先捕获具体的
        // 处理文件未找到
    } catch (IOException e) { // 再捕获更通用的
        // 处理其他IO错误
    }
    ```

## 重写和重载的区别，构造函数可以重写吗
```java
class Person {
    String name;
    public Person() {}
    public Person(String name) {this.name = name;}
    public void talk() {System.out.println("person talk");}
}
class Student extends Person {
    String stuNumber;
    public Student() {}
    public Student(String name, String stuNumber) {
        super(name);
        this.stuNumber = stuNumber;
    }
    @Override
    public void talk() {System.out.println("student talk");}
}
```
**重写：** 需要有继承关系，子类重写父类的方法。一般使用@Override注解标识，不标识也无所谓。上面代码中Student类就重写了Person类的talk方法。

**重载：** 函数名相同，参数个数不同或者参数类型不同。注意方法返回值不同是不算重载的。上面代码中对构造函数就是通过参数个数不同进行重载。

构造函数不能被重写，因为重写要求方法名一致。而构造函数的方法名就是类名。子类不可能和父类同名，所以也不可能有相同的构造函数。所以构造函数不能重写，但是可以重载。

## 介绍一些常用的Java工具命令
Java自带的命令都在JDK的bin目录下。
|命令|说明|
|---|---|
| jps | 虚拟机进程状态工具，可以列出虚拟机进程 |
| jstat | 虚拟机统计信息监视工具，监视虚拟机各种运行状态信息 |
| jinfo | Java配置信息工具 |
| jmap | 生成堆转储快照 |
| javap | java字节码信息查看工具 |
| jstack | java虚拟机进程堆栈跟踪工具, 制作线程Dump |
| jhat |  java虚拟机堆转储快照分析工具 |
| jconsole | 用于提供JVM活动的图形化视图,包括线程的使用、类的使用和GC活动. 
| jvisualvm | 监控JVM的GUI工具,可用来剖析运行的应用,分析JVM堆转储 |

## JDK、JRE、JVM的区别
> 参考
> 1. [JDK vs JRE vs JVM in Java – Difference Between Them](https://www.guru99.com/difference-between-jdk-jre-jvm.html)

JDK、JRE、JVM是Java编程语言的核心内容，虽然日常开发中我们不关注这些概念，但是作为一名合格的Java程序员，还是应该了解下它们之间的区别。
- JDK（Java Development Kit，Java开发工具包）包含了JRE以及开发小程序和应用程序所需的编译器和调试器等工具。
- JRE（Java Runtime Environment）包含类库、加载器类和 JVM。如果你想运行Java程序，你需要JRE。 如果你无需开发Java代码，则不需要安装JDK，只需要安装JRE即可运行Java程序。 不过，所有 JDK 版本都捆绑了 Java 运行时环境，因此您无需在 PC 中单独下载并安装 JRE。（即安装JDK的时候也附带了JRE）。
- JVM（Java Virtual Machine）是一个提供运行时环境来驱动Java代码或应用程序的引擎。 它将 Java 字节码转换为机器语言。 JVM 是 Java 运行时环境 (JRE) 的一部分。 在其他编程语言中，编译器为特定系统生成机器代码，所以跨平台特性不是很好。然而，Java 编译器是将 Java 代码编译成 JVM 认识的机器代码，这也是为什么Java支持跨平台。因为更换了平台只要有 JVM，JVM 就认识编译后的机器码。

**具体不同对比如下：**
| JDK | JRE | JVM |
|-------|-------|--------|
| Java Development Kit | Java Runtime Environment | Java Virtual Machine |
| JDK是一个软件开发工具包，用于开发Java应用程序 | JRE是一个软件包，提供 Java 类库以及运行 Java 代码所需的组件 | JVM执行Java字节码并提供执行环境。是Java支持跨平台的关键 |
| 包含用于开发、调试和监控 java 代码的工具 | 包含 JVM 执行程序所需的类库和其他支持文件 | |
| JDK包含JRE | JRE包含JVM | |
| JDK 使开发人员能够创建可由 JRE 和 JVM 执行和运行的 Java 程序 | JRE 是 Java 中创建 JVM 的部分 | JVM是执行源代码的Java平台组件 |

在下图中可以看出JDK、JRE、JVM三者的包含关系以及具体组成部分：
![](/images/jdk.png)