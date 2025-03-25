## Java基础知识



## 1. 创建线程

目前简单的创建方式有三种也是最常见的三种方式，其中都复写了 **run()** 方法，然后调用了 **start()** 其中当前方法的目的就是告诉操作系统当前线程准备完毕，可以开始分配资源进行执行了

```java
public static void main(String[] args) throws InterruptedException {
 		//实现Thread类的方式
        new MyThread().start();
		//使用接口实现的方式
        new Thread(new MyThreadRunner()).start();
		//使用lambda表达式的方式
        new Thread(() -> {
            System.out.println("运行inner thread");
        }).start();
        //将主线程暂停，等待其他的线程执行完
        Thread.currentThread().join();
    }
----------------------------------分割线-----------------------------------

public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("运行线程");
        }
    }

public static class MyThreadRunner implements Runnable {
    @Override
    public void run() {
        System.out.println("运行runner线程");
    }
}
```

### run()

可以看到 **Thread** 源码中的 **run()** 方法实际是调用了 target对象的 **run()** 方法，也就是说操作系统默认会调用 Thread 自身的run方法，而根据是否实现了 run 来取决于是调用内部的 Runnable接口的方法 还是调用实现的 run 方法

```java
private Runnable target;
public void run() {
    if (target != null) {
        target.run();
    }
}
```

### start()

```java
public synchronized void start() {
    //线程状态，0表示是否是初始化状态
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
    //将当前线程加入到线程组当中
        group.add(this);
        boolean started = false;
        try {
            //调用start0方法，这个方法是native标记由C++实现
            start0();
            started = true;
        } finally {
            try {
                //如果启动失败，将当前线程从当前组中移除
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                
            }
        }
    }
```

