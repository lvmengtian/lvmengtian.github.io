---
title: Java线程
createTime: 2025/09/03 19:08:09
permalink: /java-guide/5qe4rxai/
---

## 介绍什么是ThreadLocal，以及为什么会有内存泄露风险

### ThreadLocal介绍
为了解决“在多线程下访问共享变量时，存在的并发问题”，我们可以使用同步机制（如: `synchronized`）, 同时也可以使用`ThreadLocal`。`ThreadLocal` 提供了一个“每个线程都有自己的专属变量”的机制，避免了多线程环境下共享变量的并发问题。通过`ThreadLocal`的`set`方法可以看出来。`ThreadLocal`通过`Map`结构来存储数据，Key就是当前线程，Value就是存储的数据。
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value)
    }
}
```
基本用法如下：
```java
public class ThreadLocalExample {
    // 存储用户信息
    private static final ThreadLocal<User> CURRENT_USER = new ThreadLocal<>();

    public void setCurrentUser(User user) {
        CURRENT_USER.set(user); // 将用户信息绑定到当前线程
    }

    public User getCurrentUser() {
        return CURRENT_USER.get(); // 从当前线程获取用户信息
    }

    public void clearCurrentUser() {
        CURRENT_USER.remove(); // 重要！使用完后清理，防止内存泄漏
    }
}
```
### 为什么存在内存泄露（Memory Leak）风险
这就要理解上文提到的`ThreadLocal`使用的`Map`结构了，源码如下：
```java
static class ThreadLocalMap {
  static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value; // 注意这里的value依然是强引用

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```
ThreadLocalMap 中的 Entry：
- **Key：** 是一个弱引用，指向 threadLocal 实例。
- **Value：** 是一个强引用，指向设置的数据实例。

> 想了解强引用和弱引用，可以参考[ Java基础](../java/base.md) 中“什么是强引用、软引用、弱引用、虚引用”相关内容。

当程序使用完`ThreadLocal`并且正确释放(例如调用`remove()`方法)，是不会出现内存泄露风险的。但是如果创建 ThreadLocal 的强引用一直存在，或者线程迟迟不结束（例如使用线程池，线程会复用且长期存活），那么弱引用就失去了其加速回收的关键作用。

此时：引用链变成了一个强引用链，导致无法回收：
> Thread A (强引用) -> ThreadA.threadLocals (强引用) -> Entry (强引用) -> Entry.value (强引用) -> 数据实例

即使你的业务代码已经不再需要这个 数据实例，但由于这条强引用链的存在，只要线程 ThreadA 依然存在，这个 数据实例 就会一直驻留在内存中，造成内存泄露。如果线程池中每个线程都这么存一个大数据对象，最终可能导致 OutOfMemoryError。

**所以存在内存泄露风险的原因是：**
1. `ThreadLocalMap` 的Entry中，Key是弱引用，但Value是强引用
2. 线程生命周期过长，导致强引用一致存在，无法被GC回收
3. 未及时调用`remove()`方法