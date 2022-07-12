# AutomicInteger简介

J2SE 5.0提供了一组atomic class来帮助我们简化同步处理。基本工作原理是使用了同步synchronized的方法实现了对一个long, integer, 对象的增、减、赋值（更新）操作. 比如对于++运算符AtomicInteger可以将它持有的integer 能够atomic 地递增。在需要访问两个或两个以上 atomic变量的程序代码（或者是对单一的atomic变量执行两个或两个以上的操作）通常都需要被synchronize以便两者的操作能够被当作是一个atomic的单元。

## java多线程用法-使用AtomicInteger

- 下面通过简单的两个例子的对比来看一下 `AtomicInteger` 的强大的功能：

  ```java
  class Counter {
            private volatile int count = 0;
            public synchronized void increment() {
                    count++;  //若要线程安全执行执行count++，需要加锁
            }
      
            public int getCount() {return count;}
  }
  
  class Counter {
                private AtomicInteger count = new AtomicInteger();
                public void increment() {
                      count.incrementAndGet();
                  }       
                //使用AtomicInteger之后，不需要加锁，也可以实现线程安全.
                 public int getCount() {return count.get();
    }
  }
  ```

从上面的例子中我们可以看出：使用AtomicInteger是非常的安全的。  
那么为什么不使用记数器自加呢，例如count++这样的，因为这种计数是线程不安全的，高并发访问时统计会有误，而**AtomicInteger**为什么能够达到多而不乱，处理高并发应付自如呢？  
这是由硬件提供原子操作指令实现的。在非激烈竞争的情况下，开销更小，速度更快。

- Java.util.concurrent中实现的原子操作类包括：
  1. AtomicBoolean、
  2. AtomicInteger、
  3. AtomicLong、
  4. AtomicReference。

