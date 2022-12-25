## 1 什么是JUC

java.util工具包

**业务：普通的线程代码 Thread**

**Runnable** 没有返回值，效率比Callable相比较低。

## 2 进程和线程

**进程：**一个程序

**线程：**一个进程至少包含一个线程，cpu调度的最小的单位

java默认几个线程？main GC

Java真的可以开启线程吗？不行，通过**本地方法**底层C++开启。

>  并行和并发

**并行:**cpu多核。多个线程可以同时执行。

**并发：**多线程操作同一个资源

- cpu一核，模拟出多条线程，天下武功，唯快不破。

> 线程的状态

NEW、RUNNABLE、 BLOCK、 WAITING、 TIME_WAITING、 TERMINATED(6种)

> wait与sleep区别

- 来自不同的类：wait =>Object；sleep=>Thread。企业当中休眠不会使用sleep,而是使用java.util.concurrent.TimeUnit中的**TimeUnit.DAYS.sleep(1)**;
- 关于释放锁：wait会释放锁，sleep不会释放锁。
- 适用范围：wait只能使用在同步代码块中；sleep可以在任何地方睡。
- 是否需要捕获异常：wait不需要；sleep需要捕获异常。

## 3 Lock锁

> synchronized 锁

```java
//基本卖票例子
/*
* 真正的线程开发，公司中的开发
* 线程就是一个单独的资源类，没有任何附属操作
*
* */


public class SaleTicketDemo01 {
    public static void main(String[] args) {
        //并发，多个线程操作同一个资源,把资源类丢入线程
        Ticket ticket = new Ticket();
        //@FunctionalInterface 函数式接口,在jdk8之后使用lambda表达式 （参数）->{代码}
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"b").start();
        new Thread(()->{
            for (int i = 0; i < 64; i++) {
                ticket.sale();
            }
        },"c").start();
    }
}
//资源类 oop
class Ticket{
    //属性方法
    private int number = 50;
    //卖票
    //synchronized 本质就是排队 锁
    public synchronized void sale(){
        if(number>0){
            System.out.println(Thread.currentThread().getName()+"卖出了第"+number--+"张票，剩余："+number);
        }
    }
}
```

> Lock接口

![image-20210711151224414](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210711151224414.png)

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210711152017690.png" alt="image-20210711152017690" style="zoom:50%;" />

公平锁：十分公平，必须先来后到

非公平锁：十分不公平，可以插队（默认）

```java
public class SaleTicketDemo02 {

    public static void main(String[] args) {
        //并发，多个线程操作同一个资源,把资源类丢入线程
        Ticket ticket = new Ticket();
        //@FunctionalInterface 函数式接口,在jdk8之后使用lambda表达式 （参数）->{代码}
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "a").start();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "b").start();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }, "c").start();


    }
}

//资源类 oop
class Ticket1 {
    //属性方法
    private int number = 50;
    //1.新建锁
    Lock lock = new ReentrantLock();

    //卖票
    public void sale() {
        lock.lock();//2.加锁
        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了第" + number-- + "张票，剩余：" + number);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();//3.解锁
        }
    }

}
```

> Synchronized 与Lock区别

- Synchronized 内置关键字，Lock是一个Java类
- Synchronized无法判断锁的状态，Lock可以判断锁状态
- Synchronized会自动释放锁，Lock必须要手动释放锁，否则会产生死锁
- Synchronized 线程1（获得锁、堵塞）、线程2（等待、傻傻的等）；而Lock就不会一直等待下去。
- Synchronized可重入锁（**允许同一个线程多次获取同一把锁**），不可以中断（非公平）；Lock可重入锁，可以判断锁，非公平（可以自己设置）ReentrantLock(true),加一个参数，就是公平锁，否则默认非公平锁。
- 适用范围：Synchronized适合锁少量的代码同步问题，Lock适合锁大量的同步代码。

## 4 生产者和消费者

> 如果式四个线程 ABCD

将if改成while,可交易解决虚假唤醒的问题。

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210711173526668.png" alt="image-20210711173526668" style="zoom:67%;" />

```java
public class pc {
    public static void main(String[] args) {
        data dd=new data();
               new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.decrement();
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.decrement();
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "C").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "D").start();

    }

}
//判断等待，业务，资源

class data{

    private int number=0;
    public synchronized void increment() throws InterruptedException {
        //if (number!=0){
        while(number!=0){
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+"------->"+number);
        number++;
        this.notifyAll();
    }
    public synchronized void decrement() throws InterruptedException {
        while(number==0){
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+"------->"+number);
        number--;
        this.notifyAll();
    }

}
```

> JUC版本生产者消费者

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210711190615910.png" alt="image-20210711190615910" style="zoom:67%;" />



```java
//生产者消费者
/*
 * 线程交替执行
 *
 * */
public class PC2 {
    public static void main(String[] args) {
        data1 dd = new data1();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.decrement();
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.decrement();
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "C").start();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    dd.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "D").start();
    }
}
//判断等待，业务，资源

class data1 {

    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    /*
     *condition.await();  等待
     * condition.signalAll();  通知
     * */
    public void increment() throws InterruptedException {
        locklock.lock();
        try {
            while (number != 0) {
//            this.wait();
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "------->" + number);
            number++;
//        this.notifyAll();
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
//            this.wait();
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "------->" + number);
            number--;
//        this.notifyAll();
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> Condition的优势

- 可以实现精准的通知唤醒线程

```java
public class LockTest {
    public static void main(String[] args) {
        Data3 data3 = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.A();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.B();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.C();
            }
        }, "C").start();
    }
}

//
class Data3 {
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    private int num = 1;

    public void A() {
        lock.lock();
        try {
            while (num != 1) {
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "-->AAAA");
            //唤醒指定的人
            num = 2;
            condition2.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
        lock.unlock();
    }

    public void B() {
        lock.lock();
        try {
            while (num != 2) {
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + "-->BBBB");
            //唤醒指定的人
            num = 3;
            condition3.signal();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
        lock.unlock();
    }

    public void C() {
        lock.lock();
        try {
            while (num != 3) {
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + "-->CCCC");
            //唤醒指定的人
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
        lock.unlock();
    }
}
```

## 5 8锁现像

**深刻理解我们的锁**

new this 具体的一个手机

static 锁的是class

```java
import java.util.concurrent.TimeUnit;

public class Lock84 {
    public static void main(String[] args) throws InterruptedException {
        /*
         * 两个对象，两个调用者，两把锁，各自执行各自的
         * */
        /*
         * 方法是静态方法
         * 锁的是class。两个对象只有一个class，谁先拿到class就先执行谁
         * */
        Phone4 phone = new Phone4();
        Phone4 phone2 = new Phone4();
        new Thread(()->{
            try {
                phone.sendSms();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(()->{phone2.call();}).start();
    }

}
class Phone4{
    /*
     * synchronized 锁的对象是方法调用者，
     * 两个方法用的是同一个锁，谁先拿到谁执行
     * */
    /*
     * static 静态方法
     * 类一加载就有了
     * phone3只有唯一一个class对象，锁的是class
     * */
    //静态同步方法
    public static synchronized void sendSms() throws InterruptedException {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("sendSms");
    }

    /*
    * 静态方法和非静态方法使用的是两把锁
    *
    * */
    //普通同步方法
    public synchronized void call() {
        System.out.println("call");
    }
    /*
     * 增加一个普通方法，先执行普通方法 没有锁，不受锁的影响
     * */
    public void happy(){
        System.out.println("happy");
    }
}
```

## 6 集合类不安全

> List 不安全

```java
//java.util.ConcurrentModificationException 并发修改异常
public class ListTest {
    public static void main(String[] args) {
        //并发下，ArrayList 不安全
        /*
        * 解决方案
        * List<String> list = new Vector<>();
        * List<String> list =Collections.synchronizedList(new ArrayList<>());
        * CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();//
        * 写入时复制 COW 优化策略
        * 多个线程调用的时候，list，读取的时候固定的，写入（覆盖）
        * 在写入的时候避免覆盖，造成数据问题
        * CopyOnWriteArrayList比Vector好在那里？
        * Vector使用Synchronized效率低
        * CopyOnWriteArrayList 使用Lock锁，效率高
        * */
//        List<String> list = new ArrayList<>();
//        List<String> list = new Vector<>();
//        List<String> list =Collections.synchronizedList(new ArrayList<>());
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

> Map 不安全 

```java
public class MapTest {
    public static void main(String[] args) {
        //Map是这样用的吗？默认等价什么？
        //加载因子，初始化容量
//        HashMap<>(16,0.75)
        //工作中不用map
        /*
        *
        *
        * */
//        HashMap<String, String> map = new HashMap();
//        Map<Object, Object> map = Collections.synchronizedMap(new HashMap());
        ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```

## 7 Callable

> 类似于Runnable接口，他们都是为其实例科能由另一个线程执行的类设计的。然而Runnable不返回结构，也不会抛出异常

- 可以有返回值
- 可以抛出异常
- 方法不同 run()/call()

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712162320851.png" alt="image-20210712162320851" style="zoom:50%;" />

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712162341792.png" alt="image-20210712162341792" style="zoom:80%;" />

![image-20210712162656268](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712162656268.png)

```java
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
       /* new Thread(new Runnable).start();
        new Thread(new FutureTask<>()).start();
        new Thread(new FutureTask<>(Callable)).start();*/
        MyThread myThread = new MyThread();
        FutureTask futureTask = new FutureTask(myThread);
        new Thread(futureTask,"A").start();//启动Callable
        new Thread(futureTask,"B").start();//结果会被缓存，效率高
        Object o = futureTask.get();//获取返回结果 get方法可能产生堵塞   如果return之前是一个耗时的操作 可以使用异步通信
        System.out.println(o);


    }
}
class MyThread implements Callable{
    @Override
    public String call() throws Exception {
        System.out.println("call");
        return "1024";
    }
}
```

细节：

有缓存

结果可能需要等待，会产生堵塞





## 8 常用辅助类

### 8.1 CountDownLatch

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712163503003.png" alt="image-20210712163503003" style="zoom:67%;" />

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"go out");
                countDownLatch.countDown();

            },String.valueOf(i)).start();
        }
        countDownLatch.await();//等到计算器归零，然后再向下执行
        System.out.println("close door");
    }
}
```

countDownLatch.countDown(); //-1

countDownLatch.await();//等到计算器归零，然后再向下执行

countDown()数量-1，假设计算器变为0，await()就会被唤醒，继续执行。







### 8.2 CyclicBarrier

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712164346456.png" alt="image-20210712164346456" style="zoom:67%;" />

> 减法计数器

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("神龙出现"));
        for (int i = 0; i < 7; i++) {
            //我们无法在Thread里，无法直接拿到for里的i，因为lambda也是新建的一个类，无法拿到类外面的内容
            final int temp=i;
            new Thread(()->{
//                System.out.println(i);
                System.out.println(Thread.currentThread().getName()+"-->"+temp);
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```





### 8.3 Semaphore

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712165602812.png" alt="image-20210712165602812" style="zoom:67%;" />



抢车位

6车--3车位

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();//获得车位
                    System.out.println(Thread.currentThread().getName()+"拿到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"放弃车位");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();//释放
                }
            },String.valueOf(i)).start();
        }
    }
}
```

原理：

semaphore.acquire()//获得 假如现在已经满了，等待被释放为止

semaphore.release()//释放 会将当前信号量释放+1，然后唤醒等待的线程

作用：多个资源互斥使用，开发限流，控制最大的线程数！





## 9 读写锁ReadWriteLock

实现类：ReentrantReadWriteLock

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712171428741.png" alt="image-20210712171428741" style="zoom:67%;" />

```java
/*
*独占锁（写锁）依次只能被一个线程使用
* 共享锁（读锁）可以同时被多个线程使用
*  写、读，所有人都可以读，但是只能一个写
* 读写 不可共存
* 写写 不可共存
* 读读 可以共存
* */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        CacheLock cache = new CacheLock();
        for (int i = 0; i < 5; i++) {
            final int temp=i;
            new Thread(()->{
                cache.push(temp+"",temp+"");
            },String.valueOf(i)).start();
        }
        for (int i = 0; i < 5; i++) {
            final int temp=i;
            new Thread(()->{
                cache.pop(temp+"");
            },String.valueOf(i)).start();
        }

    }
}

class Cache {
    private Map<String, Object> map = new HashMap<>();

    public void push(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "放入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "放入ok");
    }

    public void pop(String key) {
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取ok");
    }

}
class CacheLock {
    private ReentrantReadWriteLock readWriteLock=new ReentrantReadWriteLock();
    private Map<String, Object> map = new HashMap<>();

    public void push(String key, Object value) {
        readWriteLock.writeLock().lock();
        System.out.println(Thread.currentThread().getName() + "放入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "放入ok");
        readWriteLock.writeLock().unlock();
    }

    public void pop(String key) {
        readWriteLock.readLock().lock();
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取ok");
        readWriteLock.readLock().unlock();
    }

}
```

## 10 堵塞队列

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712193108709.png" alt="image-20210712193108709" style="zoom:67%;" />

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712193236075.png" alt="image-20210712193236075" style="zoom:67%;" />

> BlockingQueue

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712193914209.png" alt="image-20210712193914209" style="zoom:80%;" />

什么情况使用堵塞队列？

多并发线程处理，线程池。

学会使用队列

添加、移除

四组API

ArrayBlockingQueue list=new ArrayBlockingQueue();

| 方式         | 抛出异常                   | 不会抛出异常              | 堵塞等待（一直堵塞） | 超时等待（过时不候）                     |
| ------------ | -------------------------- | ------------------------- | -------------------- | ---------------------------------------- |
| 添加         | add//满了，抛出异常        | offer//满了，返回false    | put                  | offer("a",2,TimeUnit.SECOND)//超过2s退出 |
| 移除         | remove//全部取出，抛出异常 | poll //全部取出，返回null | take                 | poll(2,TimeUnit.SECOND)//超过2s退出      |
| 检测队首元素 | element                    | peek                      |                      |                                          |

```java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        test4();
    }
    /*
    * 抛出异常
    * */
    public static void test1(){
        ArrayBlockingQueue queue = new ArrayBlockingQueue(3);
        System.out.println(queue.add(1));
        System.out.println(queue.add(2));
        System.out.println(queue.add(3));
        //java.lang.IllegalStateException: Queue full
        //System.out.println(queue.add(4));
        //判断队首元素
        System.out.println(queue.peek());
        System.out.println("==============");
        System.out.println(queue.remove());
        System.out.println(queue.remove());
        System.out.println(queue.remove());
        //java.util.NoSuchElementException
        //System.out.println(queue.remove());
        // java.util.NoSuchElementException
        //System.out.println(queue.element());
    }
    /*
    * 不会抛出异常
    * */
    public static void test2() throws InterruptedException {
        ArrayBlockingQueue queue = new ArrayBlockingQueue(3);
        System.out.println(queue.offer(1));
        System.out.println(queue.offer(2));
        System.out.println(queue.offer(3));
        System.out.println(queue.offer(4));//false
        //判断队首元素
        System.out.println(queue.peek());
        System.out.println("==============");
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());//null
    }
    /*
    * 一直都塞
    * */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue queue = new ArrayBlockingQueue(3);
        queue.put(1);
        queue.put(2);
        queue.put(3);
        //queue.put(4); 一直都塞，直到有位置空出
        System.out.println(queue.take());
        System.out.println(queue.take());
        System.out.println(queue.take());
        //System.out.println(queue.take());一直都塞，直到有元素可以取出
    }
    /*
    * 超时放弃
    * */
    public static void test4() throws InterruptedException {
        ArrayBlockingQueue queue = new ArrayBlockingQueue(3);
        System.out.println(queue.offer(1));
        System.out.println(queue.offer(2));
        System.out.println(queue.offer(3));
        System.out.println(queue.offer(4,2, TimeUnit.SECONDS));//false
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll(2, TimeUnit.SECONDS));


    }
}
```

> SynchronousQueue 同步队列

同步队列，没有容量，进入一个元素，必须等到取出以后，才能往里面放入一个元素

put take

## 11 线程池（重点）

三大方法、七大参数、四种拒绝策略。

> 池化技术

程序的运行：本质就是占用系统的资源，优化资源的使用。

池化技术：事先准备一些资源，有人使用，就来这里拿，用完之后还给我。

**线程池的好处**

1、降低资源的消耗

2、提高响应速度

3、方便管理

线程复用，可以最大控制并发数，管理线程。

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712211441044.png" alt="image-20210712211441044" style="zoom:67%;" />



### 11.1 三大方法

```java
//Executors 工具类
public class Demo1 {
    public static void main(String[] args) {
    //三大方法
        Executors.newSingleThreadExecutor();//单个线程
        Executors.newFixedThreadPool(5);//创建一个固定大小的线程池
        ExecutorService threadpoll =Executors.newCachedThreadPool();//可伸缩，遇强则强，遇弱则弱
        try {
            for (int i = 0; i < 10; i++) {
                threadpoll.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadpoll.shutdown();
        }
    }
}
```

### 11.2 七大参数

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

这三大方法都有调用 ThreadPoolExecutor

```java
public ThreadPoolExecutor(int corePoolSize, //核心线程池大小
                          int maximumPoolSize,//最大核心线程池大小
                          long keepAliveTime,//超时没人调用就会释放
                          TimeUnit unit,//超时单位
                          BlockingQueue<Runnable> workQueue,//堵塞队列
                          ThreadFactory threadFactory,//线程工厂
                          RejectedExecutionHandler handler//拒绝策略) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712213450627.png" alt="image-20210712213450627" style="zoom:67%;" />

> 手动创建线程池

```java
public class Demo1 {
    public static void main(String[] args) {
        ExecutorService threadPoolExecutor=new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy());
        try {
            //最大承载 deque+max
            //超过最大承载 java.util.concurrent.RejectedExecutionException:
            for (int i = 0; i < 10; i++) {
                threadPoolExecutor.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPoolExecutor.shutdown();
        }
    }
}
```

### 11.3 四大拒绝策略

![image-20210712215220381](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712215220381.png)

> 最大线程该如何确定

CPU密集型:几个核就是几

获取cpu核数：System.out.println(Runtime.getRuntime().availableProcessors());

IO密集型:判断你程序中十分消耗IO的线程，在这个基础上让线程数>这个数

## 12 四大函数式接口（必须掌握）

> 新世纪程序员必会技能

- lambda表达式

- 链式编程

- 函数式接口：只有一个方法的接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
//FunctionalInterface
//简化编程模型，在版本的框架底层有大量使用
```



- Stream流式计算

### 12.1 Function 函数式接口

> 有一个输入，有一个输出

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210712221042853.png" alt="image-20210712221042853" style="zoom:50%;" />

```java
/*
* Function 函数式接口，有一个输入，有一个输出
* 只要是函数接口 可以用lambda表达式简化
* */
public class FunctionDemo {
    public static void main(String[] args) {
        /*Function<String ,String> function=new Function<String, String>() {
            @Override
            public String apply(String s) {
                return s;
            }
        };*/
        //lambda表达式
        Function<String,String> function=(str)->{return str;};
        System.out.println(function.apply("ihopper"));
    }
}
```

### 12.2 Predicate 函数式接口

> 断定型函数接口 ，有一个输入参数，返回值只能是布尔值

```java
public class PredicateDemo {
    public static void main(String[] args) {
//        Predicate predicate=new Predicate<String>(){
//
//            @Override
//            public boolean test(String s) {
//                return s.isEmpty();
//            }
//        };
        Predicate<String> predicate=(str)->{return str.isEmpty();};
        boolean s = predicate.test("s");
        System.out.println(s);
    }
}
```

### 12.3 Consumer 消费型接口

> 只有输入，没有返回值

```java
public class ConsumerDemo {
    public static void main(String[] args) {
       /* Consumer<String> consumer=new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };*/
        //lambda 表达式
        Consumer<String> consumer=(str)->{
            System.out.println(str);
        };
        consumer.accept("ihopper");
    }
}
```

### 12.4 Supplier 供给型接口

> 没有输入，只有输出

```java
public class SupplierDemo {
    public static void main(String[] args) {
//        Supplier<Integer> supplier=new Supplier() {
//            @Override
//            public Integer get() {
//                return 1024;
//            }
//        };
        //lambda表达式
        Supplier<Integer> supplier=()->{return 1024;};
        Integer integer = supplier.get();
        System.out.println("integer = " + integer);
    }
}
```

## 13 Stream流式计算

```java
public class StreamDemo {
    public static void main(String[] args) {
        User u1 = new User(1, "A", 12);
        User u2 = new User(2, "B", 22);
        User u3 = new User(3, "C", 23);
        User u4 = new User(4, "D", 24);
        User u5 = new User(5, "E", 25);
        //将这些数据存储起来
        List<User> list= Arrays.asList(u1,u2,u3,u4,u5);
        //将计算交给流  链式编程
        list.stream()
                .filter((u)->{return u.getId()%2==0;})
                .filter((u)-> {return u.getAge()>23;})
                .map((u)->{u.getName().toUpperCase();})
                .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```



## 14 ForkJoin（分支合并）

大数据+计算

ForkJoin在JDK1.7 变更发执行任务，提高效率，大数据量。

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713082253060.png" alt="image-20210713082253060" style="zoom:67%;" />

> 特点：工作窃取

这里买你维护的双端队列，可以两头拿任务

> <img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713082334882.png" alt="image-20210713082334882" style="zoom:50%;" />

可以提高效率

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713091302445.png" alt="image-20210713091302445" style="zoom:67%;" />

```java
/*
 *如何使用forkJoin
 * 1，通过forkJoinPool来执行
 * 2，计算任务forkjoinPool.execute(ForkJoinTask task)
 * 3,计算类要继承ForkJoinTask
 *
 **/

public class ForkJoinDemo extends RecursiveTask<Long> {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
            //test1();//time:5830
            test2();//time:4159
            //test3();//time:191
    }

    //普通方法
    public static void test1() {
        Long start = System.currentTimeMillis();
        Long sum = 0L;
        for (Long i = 0L; i < 10_0000_0000L; i++) {
            sum += i;
        }
        Long end = System.currentTimeMillis();
        System.out.println("sum:" + sum + " time:" + (end - start));
    }

    //使用forkJoin
    public static void test2() throws ExecutionException, InterruptedException {
        Long start = System.currentTimeMillis();

        ForkJoinPool joinPool = new ForkJoinPool();
        ForkJoinTask<Long> forkJoinDemo = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = joinPool.submit(forkJoinDemo);
        Long sum = submit.get();

        Long end = System.currentTimeMillis();
        System.out.println("sum:" + sum + " time:" + (end - start));
    }
    //Stream并行流

    public static void test3() {
        Long sum = 0L;
        Long start = System.currentTimeMillis();
        LongStream.rangeClosed(0L,10_0000_0000L).parallel().reduce(0,Long::sum);
        Long end = System.currentTimeMillis();
        System.out.println("sum:" + sum + " time:" + (end - start));
    }
    private Long start;
    private Long end;
    //临界值
    private Long temp = 10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if ((end - start) < temp) {
            Long sum = 0L;
            for (Long i = start; i < end; i++) {
                sum += i;
            }
            return sum;
        } else {
            Long middle = (start + end) / 2;
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();//拆分任务，把任务压入线程
            ForkJoinDemo task2 = new ForkJoinDemo(middle + 1, end);
            task2.fork();
            return task1.join() + task2.join();
        }
    }
}
```

## 15 异步回调

> Future 设计的初衷：对将来的某件事情建模

不一定要马上拿到结果，可以先进行其他线程

```java
/*
* 异步调用：Ajax
* 异步执行
* 成功返回
*
*
* */
public class Future {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //发起一个请求 runAsync 没有返回值
        CompletableFuture<Void> future = CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"runAsync");
        });
        System.out.println(111111111);
        future.get();//获得执行结果


        System.out.println("===================================");
        //有返回值的异步回调    供给型接口 只有返回值
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"supplyAsync");
            return 1024;
        });
        completableFuture.whenComplete((u,t)->{//成功了
            System.out.println("u"+u);//正常返回结果
            System.out.println("t"+t);//错误信息
        }).exceptionally((e)->{//失败了
            System.out.println(e.getMessage());
            return 233;//获取错误返回结果
        }).get();
    }
}
```

## 16 JMM

Volatile是Java虚拟机提供的**轻量级的同步机制**

- 保证可见性
- 不保证原子性
- 禁止指令重排

> 谈一谈保证可见性 什么是JMM

JMM：java内存模型，不存在的东西，概念！约定！

关于JMM同步的一些约定？

- 线程解锁前：必须把共享变量立刻刷回主存。
- 线程加锁前，必须读取主存中最新的值存到工作内存中。
- 加锁和解锁是同一把锁。

线程 **工作内存 主内存**

> 8中操作

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713100720284.png" alt="image-20210713100720284" style="zoom:67%;" />

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713100859421.png" alt="image-20210713100859421" style="zoom:67%;" />

内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）

- - lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
  - unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
  - read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
  - load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
  - use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
  - assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
  - store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
  - write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

　　JMM对这八种指令的使用，制定了如下规则：

- - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
  - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
  - 不允许一个线程将没有assign的数据从工作内存同步回主内存
  - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
  - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
  - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
  - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  - 对一个变量进行unlock操作之前，必须把此变量同步回主内存

　　JMM对这八种操作规则和对[volatile的一些特殊规则](https://www.cnblogs.com/null-qige/p/8569131.html)就能确定哪里操作是线程安全，哪些操作是线程不安全的了。但是这些规则实在复杂，很难在实践中直接分析。所以一般我们也不会通过上述规则进行分析。更多的时候，使用java的happen-before规则来进行分析。

> JMM 无法保证可见性

## 17 volatile

- 保证可见性

```java
public class VolatileTest {
    //不加volatile就会死循环
    //加volatile可以保证可见性
    private volatile static int num=0;
    public static void main(String[] args) {

        new Thread(()->{
            while (num==0){//线程1 对于主内存中的变化不知道

            }
        }).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        num=1;
        System.out.println(num);
    }
}
```

- 不保证原子性

线程A在执行任务的时候，不能被盗扰，也不能被分割，要么同时成功，要么同时失败。

```java
public class VolatileTest {
    //每次的结果都不一样，可见 volatile 不保证原子性
    private volatile static int num=0;
    public static void add(){
        num++;//不是一个原子性操作
    }
    public static void main(String[] args) {

        for (int i = 0; i <=20; i++) {
            new Thread(()->{
                for (int i1 = 0; i1 < 1000; i1++) {
                    add();
                }
            }).start();
        }
        //判断我们的线程是否执行完了
        while (Thread.activeCount()>2){//main gc 是一定存在的
            /*
            *暂停当前正在执行的线程对象，并执行其他线程。
            * yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。
            * 因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。
            * 但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。
            * 结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。
            *  */
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}
```

> 不加lock 和synochronized如何保证原子性？

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713104117519.png" alt="image-20210713104117519" style="zoom:67%;" />

使用原子类解决这个问题。

> 原子类为什么高级？

```java
public class VolatileTest {
    //每次的结果都不一样，可见 volatile 不保证原子性
    private volatile static AtomicInteger num=new AtomicInteger();
    public static void add(){
//        num++; //不是原子操作
        num.getAndIncrement();// AtomicInteger  CAS+1
    }
    public static void main(String[] args) {

        for (int i = 1; i <=20; i++) {
            new Thread(()->{
                for (int i1 = 0; i1 < 1000; i1++) {
                    add();
                }
            }).start();
        }
        //判断我们的线程是否执行完了
        while (Thread.activeCount()>2){//main gc 是一定存在的
            /*
            *暂停当前正在执行的线程对象，并执行其他线程。
            * yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。
            * 因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。
            * 但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。
            * 结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。
            *  */
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}
//总结：类的底层都是直接和操作系统挂钩的！在内存中修改值！Unsafe是一个很特殊的操作
```

- 禁止指令重排

> 什么是指令重排？

写的程序计算机并不是按照你写的执行的。

源代码->编译器优化->指令并行也可能会重拍->内存系统也可能回重排->执行。

处理器在进行指令重排的时候，考虑，数据之间的依赖性。

![image-20210713105202935](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713105202935.png)

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713105424741.png" alt="image-20210713105424741" style="zoom:67%;" />

> 加入volitale 可以避免指令重排。

内存屏障 是一个cpu指令 它的作用

- 可以保证特定的执行顺序（禁止顺序的颠倒）

- 保证某些变量的内存可见性（利用这些特性voliatile实现了可见性）



## 18 深入理解CSA

> 什么是CAS

```JAVA
public class CASDemo {
    //比较并交换
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);
        //compareAndSet 如果我们的期望达到了就更新，否则就不更新 CAS 是cpu的并发原语
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        int i = atomicInteger.get();
        System.out.println(i);
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        int j = atomicInteger.get();
        System.out.println(j);
    }
}
```

> unsafe

![image-20210713144016755](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713144016755.png)

```java
//看看源码咋来的
atomicInteger.getAndIncrement();//++
```

![image-20210713145308109](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713145308109.png)

CAS:比较当前的工作内存中的值和主内存的值，如果这个值是期望的，那么则执行操作！否则就一直循环！

**缺点**

循环会耗时

一次性只能保证一个共享变量的原子性

ABA问题

> CSA:ABA问题（狸猫换太子） 引入原子引用 对应的思想乐观锁

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713150155652.png" alt="image-20210713150155652" style="zoom:67%;" />

```java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);
        System.out.println("捣乱线程++++++++++++++++++++++++++++++++++");
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(2021, 2020));
        System.out.println(atomicInteger.get());
        System.out.println("期望线程++++++++++++++++++++++++++++++++++");
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
    }
}
```

## 19 原子引用

带版本号的原子引用

![image-20210713152639280](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713152639280.png)

```java
public class ABADemo {
    public static void main(String[] args) {
        //添加的值 和时间戳
        AtomicStampedReference<Integer> stampedReference=new AtomicStampedReference(1,1);
        new Thread(()->{
            int astamp = stampedReference.getStamp();
            System.out.println("A1=>"+astamp);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //ABA--->A
            System.out.println(stampedReference.compareAndSet(1, 2, stampedReference.getStamp(), stampedReference.getStamp() + 1));
            System.out.println("A2=>"+stampedReference.getStamp());
            //ABA--->AB
            System.out.println(stampedReference.compareAndSet(2, 1, stampedReference.getStamp(), stampedReference.getStamp() + 1));
            System.out.println("A3=>"+stampedReference.getStamp());

        },"A").start();
        new Thread(()->{
            int bstamp = stampedReference.getStamp();
            System.out.println("B1=>"+bstamp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //ABA--->ABA
            System.out.println(stampedReference.compareAndSet(1, 6, bstamp, stampedReference.getStamp() + 1));
            System.out.println("B2=>"+stampedReference.getStamp());
        },"B").start();
    }
}
```

## 20 各种锁

- 公平锁        非常公平，不可以插队
- 非公平锁    非常不公平，可以插队

```java
//默认非公平锁
public ReentrantLock(){
    sync=new NonfairSync();
}
//加一个true，变成非公平锁
public ReentrantLock(boolean fair){
    sync=fair?new FairSync():new NonfairSync();
}
```

- 可重入锁

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713154553923.png" alt="image-20210713154553923" style="zoom:67%;" />

```java
//可重入锁，拿到一把锁，起他得也就拿到了
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{phone.sendMSM();},"A").start();
        new Thread(()->{phone.sendMSM();},"B").start();
    }
}

class Phone {
    public synchronized void sendMSM() {
        System.out.println(Thread.currentThread().getName() + " get sendMSM");
        call();//这也有锁
    }

    public synchronized void call() {
        System.out.println(Thread.currentThread().getName() + " get call");
    }
}
```

> Lock版

```java
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            phone.sendMSM();
        }, "A").start();
        new Thread(() -> {
            phone.sendMSM();
        }, "B").start();
    }
}

class Phone {
    ReentrantLock lock = new ReentrantLock();

    public void sendMSM() {
        lock.lock();
        System.out.println(Thread.currentThread().getName() + " get sendMSM");
        call();
        lock.unlock();
    }

    public void call() {
        lock.lock();
        System.out.println(Thread.currentThread().getName() + " get call");
        lock.unlock();
    }
}
```

-  自旋锁（spinlock）

一个执行单元要想访问被自旋锁保护的共享资源，必须先得到锁，在访问完共享资源后，必须释放锁。如果在获取自旋锁时，没有任何执行单元保持该锁，那么将立即得到锁；如果在获取自旋锁时锁已经有保持者，那么获取锁操作将自旋在那里，一直去尝试获取锁，直到该自旋锁的保持者释放了锁。

![image-20210713160322866](C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713160322866.png)

- 死锁

相当于两个人互相抢夺资源

<img src="C:\Users\kangr\AppData\Roaming\Typora\typora-user-images\image-20210713162015151.png" alt="image-20210713162015151" style="zoom:67%;" />

```java
public class Deadlock {
    public static void main(String[] args) {
        String LockA = "LockA";
        String LockB = "LockB";
        new Thread(new MyThread(LockA,LockB),"A").start();
        new Thread(new MyThread(LockB,LockA),"B").start();
    }

}

class MyThread implements Runnable {
    private String LockA;
    private String LockB;

    public MyThread(String lockA, String lockB) {
        LockA = lockA;
        LockB = lockB;
    }

    @Override
    public void run() {
        synchronized (LockA) {
            System.out.println(Thread.currentThread().getName() + " get LockA wants to get LockB");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (LockB) {
                System.out.println(Thread.currentThread().getName() + " get LockB wants to get LockA");
            }

        }
    }
}
```

> 如何排除死锁

- 使用jps定位进程号 jps -l
- 使用jstack +进程号 