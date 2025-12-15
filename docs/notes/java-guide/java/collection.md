---
title: Java集合
createTime: 2025/08/20 21:55:46
permalink: /java-guide/h9mgtev9/
---

## 常用的Java集合有哪些，有哪些线程安全的集合

### 📌 常用Java集合分类图谱

下图清晰地展示了常用集合的接口继承与实现关系：
![](/images/jihe.png)

### 🧱 线程安全集合

当多个线程同时读写一个集合时，普通的集合类会因数据竞争而导致状态错乱。Java提供了多种方式来实现线程安全，其演进体现了性能与安全的权衡。

#### 方案一：古老的同步包装器（性能差，不推荐）
通过 `Collections.synchronizedXXX()` 方法，给普通集合加一个同步外壳。
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
```
- **原理**：在方法内部使用 `synchronized` 关键字进行粗粒度锁。
- **缺点**：**性能差**，整个集合对象被锁，高并发下串行化严重。

#### 方案二：并发包（JUC）下的现代线程安全集合（强烈推荐）
这是应对高并发的标准答案，其核心思想是 **“锁细化”和“无锁化”**。

| 接口/类 | 线程安全原理 | 特点与适用场景 |
| :--- | :--- | :--- |
| **`CopyOnWriteArrayList`** | **写时复制**。修改时复制新数组，在新数组上操作，最后替换引用。读操作完全无锁。 | **读多写极少**（如监听器列表）。写性能差，有内存开销。 |
| **`CopyOnWriteArraySet`** | 底层基于 `CopyOnWriteArrayList`。 | 同 `CopyOnWriteArrayList`，适用于**小型、读多写少的唯一集合**。 |
| **`ConcurrentHashMap`** | **分段锁（JDK7） / CAS + synchronized（JDK8）**。只锁住操作的桶（或链表头/红黑树根），极大提升并发度。 | **高并发HashMap的绝对首选**。性能远高于 `Hashtable`。 |
| **`ConcurrentSkipListMap`** | **跳表**。利用空间换时间，实现无锁的并发有序访问。 | 需要**高并发且Key有序**的场景。 |
| **`ConcurrentSkipListSet`** | 基于 `ConcurrentSkipListMap`。 | 需要**高并发且元素有序**的Set。 |
| **`ArrayBlockingQueue`** | **可重入锁**。经典的有界阻塞队列。 | 固定大小的线程池任务队列。 |
| **`LinkedBlockingQueue`** | **双锁**。基于链表，可选有界/无界。 | `Executors.newFixedThreadPool` 的默认队列。 |
| **`PriorityBlockingQueue`** | **可重入锁**。支持优先级的无界阻塞队列。 | 需要按优先级处理任务的场景。 |

### 🌰 经典面试示例
::: tip HashMap 和 ConcurrentHashMap 的对比
| 维度 | HashMap | ConcurrentHashMap (JDK8+) |
| :--- | :--- | :--- |
| **线程安全** | **不安全**，多线程put会导致死循环或数据丢失。 | **安全**。 |
| **锁粒度** | 无锁。 | **桶级别**（对链表头或树根加`synchronized`），极致细化。 |
| **Null支持** | 允许一个null key和多个null value。 | **不允许**null key或value，因无法区分“不存在”和“值为null”。 |
| **性能** | 单线程下最快。 | 接近HashMap，并发下远胜同步包装器。 |
::: 

::: tip 如何选择线程安全集合
1.  **需要 `Map`**：无脑选择 **`ConcurrentHashMap`**，替代任何 `Hashtable` 或 `synchronizedMap`。
2.  **需要 `List`**：
    - **写少读多**：选 `CopyOnWriteArrayList`。
    - **写多或均衡**：考虑使用 `Collections.synchronizedList(new ArrayList<>())` 或更高级的并发队列。
3.  **需要 `Set`**：
    - **基于 `CopyOnWriteArrayList`**：用 `CopyOnWriteArraySet`。
    - **需要排序**：用 `ConcurrentSkipListSet`。
4.  **需要 `Queue`**：根据有界/无界、公平/非公平、优先级等需求，在JUC的阻塞队列中选择。
:::

::: tip HashTable为什么被淘汰
`Hashtable` 使用 `synchronized` 修饰所有方法，是**对象级锁**，性能极差。`ConcurrentHashMap` 的**分段锁**和**CAS**设计性能比`Hashtable` 高很多。
::: 

::: tip ConcurrentHashMap在JDK7和JDK8的实现有何不同
**JDK7**：采用 **Segment 分段锁**（继承自`ReentrantLock`），锁住一个Segment。

**JDK8**：抛弃Segment，改用 **`synchronized` 锁桶的头节点 + CAS + `volatile`**，锁粒度更细，并发度更高，且利用了JVM对`synchronized`的优化。
:::

::: tip ConcurrentHashMap的size()方法是如何实现的
这是一个设计亮点。它并非直接遍历计数（那样会慢且不一致）。在JDK8中，它采用**分端计数**的思想，先尝试无锁求和（遍历所有CounterCell），如果失败且有竞争，会创建额外的CounterCell来分摊计数，最终返回一个**弱一致性的近似值**，这在并发场景下是更优的设计。
:::



## ArrayList和LinkedList的区别及各自适用场景

## HashMap的工作原理及其在JDK 1.8中的优化

## ConcurrentHashMap是如何实现线程安全的

## 如何实现自定义对象在HashSet中的去重？

## ConcurrentHashMap 为什么不支持 null 值？而 HashMap 为什么可以?
