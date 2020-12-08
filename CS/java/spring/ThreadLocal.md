**ACA系统API Server层需要从BCE-WebFramework框架获取request-id等信息，涉及ThreadLocal相关内容，稍作整理。**

# ThreadLocal
ThreadLocal用于提供线程局部变量。在多线程环境可以保证各个线程里的变量独立于其它线程里的变量，即ThreadLocal可以为每个线程创建一个单独的变量副本，相当于线程里的 private static类型变量。
ThreadLocal的作用和同步机制有些相反：同步机制是为了保证多线程环境下数据的一致性；ThreadLocal是保证多线程环境下数据的独立性。

# 代码实现
ThreadLocal的构造方法是一个简单无参方法，并且没有任何实现。

## set()方法
``` java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
首先获取当前线程，然后获取当前线程的ThreadLocalMap，如果不为null，则将value保存到map中，并用当前ThreadLocal作为key，否则创建一个ThreadLocalMap并给到当前线程，然后保存value。

## get()
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
同样，get()也会获取当前线程的ThreadLoalMap，如果不为null，则获取以当前线程为key的value；否则调用setInitialValue()返回初始值（默认设置为null，子类可重写），并保存到新创建的ThreadLocalMap中。

## remove()
```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```
用来移除当前ThreadLocal对应的值，通过当前线程的ThreadLocalMap来移除相应的值。

# 场景
