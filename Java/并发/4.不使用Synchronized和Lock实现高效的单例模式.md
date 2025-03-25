# 1、使用饿汉模式

```java
public class Singleton { 
    private static Singleton instance = new Singleton();
    private Singleton (){}
    public static Singleton getInstance() {
      return instance;
    }
}
//使用static来定义静态成员变量或静态代码，借助Class的类加载机制实现线程安全单例
```
# 2、静态内部类

```java
public class Singleton {
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton (){}
    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
//静态内部类，只有显示的调用了getInstance()方法之后，才会装载对象并且实例化instance
```
# 3、枚举

```java
public enum Singleton {
    INSTANCE;
    public void whateverMethod() {
    }
}
//它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象
```
> 以上的几种方式都借助了ClassLoader中的LoadClass方法在加载类的时候使用了synchronized关键字。除非被重写否则在整个装载过程中都是同步的。虽然没有显示的使用synchronized关键字但是底层还是使用了。

# 4、使用Atomic包中的 CAS乐观锁实现

```java
public class Singleton {
    private static final AtomicReference<Singleton> INSTANCE = new AtomicReference<Singleton>();
    private Singleton() {}
    public static Singleton getInstance() {
        for (;;) {
            Singleton singleton = INSTANCE.get();
            if (null != singleton) {
                return singleton;
            }
            singleton = new Singleton();
            if (INSTANCE.compareAndSet(null, singleton)) {
                return singleton;
            }
        }
    }
}
/**
这种CAS方式 借助了底层的CMPXCHG指令实现，这个指令是原子性操作
优点：CAS没有像传统一样的锁机制来保证线程安全，CAS是一种基于忙等待的算法，没有线程切换和阻塞的额外消耗。
缺点：如果大量线程执行到了  new Singleton() 会有大量的对象创建导致内存溢出。
**/
```