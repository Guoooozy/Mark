# Java并发进阶常见面试题

## 1.synchronized关键字

### 1.1.传统的生产者消费者问题，防止虚假唤醒

线程交替执行，A B操作同一个变量 num=1
A num+1     B num-1    

```java
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{
            try {data.increment();} catch (InterruptedException e) {e.printStackTrace();}
        }).start();
        new Thread(()->{
            try {data.decrement();} catch (InterruptedException e) {e.printStackTrace();}
        }).start();
    }
}
class Data{
    private int number = 0;
    public synchronized void increment() throws InterruptedException {
        if(number!=0){this.wait();}//将if换成while则可以解决虚假唤醒问题
        number++;//业务代码
        this.notifyAll();
    }
    public synchronized void decrement() throws InterruptedException {
        if(number==0){this.wait();}//将if换成while则可以解决虚假唤醒问题
        number--;//业务代码
        this.notifyAll();
    }
}
```

> 虚假唤醒（线程可以唤醒，而不会被通知，中断或超时）

解决虚假唤醒问题，就应该将等待放在循环中（while）

### 1.2.Lock版的生产者消费者问题

使用Condition中的await来等待，signalAll来唤醒全部，signal来唤醒单个condition

```java
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}
class Data{
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition =lock.newCondition();
    public void increment() throws InterruptedException {
        try {
            lock.lock();
            while (number!=0){
                condition.await();//线程等待
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"==>"+number);
            condition.signalAll();//唤醒全部
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    public  void decrement() throws InterruptedException {
        try {
            lock.lock();
            while (number==0){
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName()+"==>"+number);
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

### 1.3.八锁现象，（判断锁的是谁）

如何判断锁的是谁？

修饰`实例方法`,进入同步代码前要获得 **当前对象`实例`的锁**

```java
//修饰实例方法
//锁的是方法的调用者，调用的对象
synchronized void method() {
  //业务代码
}
```

**2.修饰静态方法:** 也就是给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。因为静态成员不属于任何一个实例对象，是类成员（ _static 表明这是该类的一个静态资源，不管 new 了多少个对象，只有一份_）。所以，如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，**因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁**。

```java
synchronized void staic method() {
  //业务代码
}
```

**3.修饰代码块** ：指定加锁对象，对给定对象/类加锁。`synchronized(this|object)` 表示进入同步代码库前要获得**给定对象的锁**。`synchronized(类.class)` 表示进入同步代码前要获得 **当前 class 的锁**

```java
synchronized(this) {
  //业务代码
}
```

总结

- `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁。
- `synchronized` 关键字加到实例方法上是给对象实例上锁。
- 尽量不要使用 `synchronized（String a）`,因为JVM中，字符串具有缓存功能

#### 双重校验锁实现对象单例（线程安全）

```java
public class Singleton {
    private volatile static Singleton uniqueInstance;
    private Singleton() {}
    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

另外，需要注意 `uniqueInstance` 采用 **`volatile`** 关键字修饰也是很有必要。

`uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

使用 `volatile` 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

### 1.4.构造方法可以使用synchronized关键字修饰吗？

不能，构造方法本来就是线程安全的，不存在同步的构造方法

### 1.5. synchronized关键字的底层原理

#### 1.5.1 synchronized修饰同步代码块的情况(锁的是对象的class模板)

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

> 在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的，由[ObjectMonitor](https://github.com/openjdk-mirror/jdk7u-hotspot/blob/50bdefc3afe944ca74c3093e7448d6b889cd20d1/src/share/vm/runtime/objectMonitor.cpp)实现的。每个对象中都内置了一个 `ObjectMonitor`对象。
>
> 另外，**`wait/notify`等方法也依赖于`monitor`对象，这就是为什么只有在同步的块或者方法中才能调用`wait/notify`等方法，否则会抛出`java.lang.IllegalMonitorStateException`的异常的原因。**

在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

#### 1.5.2 synchronized修饰方法的情况（锁的是方法的调用者，引用对象）

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

#### 1.5.3 总结

`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。

**不过两者的本质都是对对象监视器 monitor 的获取。**

### 1.6  Jdk1.6之后的synchronized的关键字底层做了哪些优化

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。
锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

> java对象头（存储锁类型）
>
> 在HotSpot虚拟机中, 对象在内存中的布局分为三块区域: 对象头, 示例数据和对其填充.
> 对象头中包含两部分: MarkWord 和 类型指针.
> 如果是数组对象的话, 对象头还有一部分是存储数组的长度.
> 多线程下synchronized的加锁就是对同一个对象的对象头中的MarkWord中的变量进行CAS操作.
>
> #### **CAS操作：**
>
> 从某一内存上取值V，和预期值A进行比较，如果内存值V和预期值A的结果相等，那么我们就把新值B更新到内存，如果不相等，那么就重复上述操作直到成功为止。
>
> > 我们假设有A、B两个线程同时执行一个int值value自增的代码，并且同时获取了当前的value，我们还要假设线程B比A快了那么0.00000001s，所以B先执行，线程B执行了cas操作之后，发现当前值和预期值相符，就执行了自增操作，此时这个value = value + 1;然后A开始执行，A也执行了cas操作，但是此时value的值和它当时取到的值已经不一样了，所以此次操作失败，重新取值然后比较成功，然后将value值更新，这样两个线程进入，value值自增了两次，符合我们的预期。
>
> #### MarkWord
>
> Mark Word用于存储对象自身的运行时数据, 如HashCode, GC分代年龄, 锁状态标志, 线程持有的锁, 偏向线程ID等等.
> 占用内存大小与虚拟机位长一致(32位JVM -> MarkWord是32位, 64位JVM->MarkWord是64位).
>
> #### 类型指针
>
> 类型指针指向对象的类元数据, 虚拟机通过这个指针确定该对象是哪个类的实例.

> 优化后synchronized锁的分类

级别从低到高依次是:

1. 无锁状态
2. 偏向锁状态
3. 轻量级锁状态
4. 重量级锁状态

锁可以升级, 但不能降级. 即: 无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁是单向的.

每个锁状态时, 对象头中的MarkWord这一个字节中的内容是什么. 以32位为例.

#### 无锁状态

|     25bit      |     4bit     | 1bit(是否是偏向锁) | 2bit(锁标志位) |
| :------------: | :----------: | :----------------: | :------------: |
| 对象的hashCode | 对象分代年龄 |         0          |       01       |

#### 偏向锁状态

| 23bit  | 2bit  |     4bit     | 1bit | 2bit |
| :----: | :---: | :----------: | :--: | :--: |
| 线程ID | epoch | 对象分代年龄 |  1   |  01  |

#### 轻量级锁状态

|        30bit         | 2bit |
| :------------------: | :--: |
| 指向栈中锁记录的指针 |  00  |

#### 重量级锁状态

|           30bit            | 2bit |
| :------------------------: | :--: |
| 指向互斥量(重量级锁)的指针 |  10  |

#### synchronize的各个锁的状态含义

##### 1.偏向锁

偏向锁是针对于一个线程而言的, 线程获得锁之后就不会再有解锁等操作了, 这样可以省略很多开销. 假如有两个线程来竞争该锁话, 那么偏向锁就失效了, 进而升级成轻量级锁了.
为什么要这样做呢? 因为经验表明, 其实大部分情况下, 都会是同一个线程进入同一块同步代码块的. 这也是为什么会有偏向锁出现的原因.
在Jdk1.6中, 偏向锁的开关是默认开启的, 适用于只有一个线程访问同步块的场景.

###### 偏向锁的加锁

当一个线程访问同步块并获取锁时, 会在锁对象的`对象头（markword）`和`栈帧`中的锁记录里存储锁偏向的`线程ID`, 以后该线程进入和退出同步块时不需要进行CAS操作来加锁和解锁, 只需要简单的测试一下锁对象的对象头的MarkWord里是否存储着指向当前线程的偏向锁(线程ID是当前线程), 如果测试成功, 表示线程已经获得了锁; 如果测试失败, 则需要再测试一下MarkWord中偏向锁的标识是否设置成1(表示当前是偏向锁), 如果没有设置, 则使用CAS竞争锁, 如果设置了, 则尝试使用CAS将锁对象的对象头的偏向锁指向当前线程.

###### 偏向锁的撤销

偏向锁使用了一种等到竞争出现才释放锁的机制, 所以当其他线程尝试竞争偏向锁时, 持有偏向锁的线程才会释放锁. 偏向锁的撤销需要等到全局安全点(在这个时间点上没有正在执行的字节码). 首先会暂停持有偏向锁的线程, 然后检查持有偏向锁的线程是否存活, 如果线程不处于活动状态, 则将锁对象的对象头设置为无锁状态; 如果线程仍然活着, 则锁对象的对象头中的MarkWord和栈中的锁记录要么重新偏向于其它线程要么恢复到无锁状态, 最后唤醒暂停的线程(释放偏向锁的线程).

###### 总结

偏向锁在Java6及更高版本中是默认启用的, 但是它在程序启动几秒钟后才激活. 
可以使用-XX:BiasedLockingStartupDelay=0来关闭偏向锁的启动延迟
也可以使用-XX:-UseBiasedLocking=false来关闭偏向锁, 那么程序会直接进入轻量级锁状态.

##### 2.轻量级锁

###### 轻量级锁加锁

线程在执行同步块之前, JVM会先在当前线程的栈帧中创建用户存储锁记录的空间, 并将对象头中的MarkWord复制到锁记录中. 然后线程尝试使用CAS将对象头中的MarkWord替换为指向锁记录的指针. 如果成功, 当前线程获得锁; 如果失败, 表示其它线程竞争锁, 当前线程便尝试使用自旋来获取锁, 之后再来的线程, 发现是轻量级锁, 就开始进行自旋.

###### 轻量级锁解锁

轻量级锁解锁时, 会使用原子的CAS操作将当前线程的锁记录替换回到对象头, 如果成功, 表示没有竞争发生; 如果失败, 表示当前锁存在竞争, 锁就会膨胀成重量级锁.

###### 总结

总结一下加锁解锁过程, 有线程A和线程B来竞争对象c的锁(如: synchronized(c){} ), 这时线程A和线程B同时将对象c的MarkWord复制到自己的锁记录中, 两者竞争去获取锁, 假设线程A成功获取锁, 并将对象c的对象头中的线程ID(MarkWord中)修改为指向自己的锁记录的指针, 这时线程B仍旧通过CAS去获取对象c的锁, 因为对象c的MarkWord中的内容已经被线程A改了, 所以获取失败. 此时为了提高获取锁的效率, 线程B会循环去获取锁, 这个循环是有次数限制的, 如果在循环结束之前CAS操作成功, 那么线程B就获取到锁, 如果循环结束依然获取不到锁, 则获取锁失败, 对象c的MarkWord中的记录会被修改为重量级锁, 然后线程B就会被挂起, 之后有线程C来获取锁时, 看到对象c的MarkWord中的是重量级锁的指针, 说明竞争激烈, 直接挂起.

解锁时, 线程A尝试使用CAS将对象c的MarkWord改回自己栈中复制的那个MarkWord, 因为对象c中的MarkWord已经被指向为重量级锁了, 所以CAS失败. 线程A会释放锁并唤起等待的线程, 进行新一轮的竞争.

#### 6-3.锁的比较

| 锁       | 优点                                                         | 缺点                                            | 适用场景                           |
| -------- | ------------------------------------------------------------ | ----------------------------------------------- | ---------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗, 和执行非同步代码方法的性能相差无几. | 如果线程间存在锁竞争, 会带来额外的锁撤销的消耗. | 适用于只有一个线程访问的同步场景   |
| 轻量级锁 | 竞争的线程不会阻塞, 提高了程序的响应速度                     | 如果始终得不到锁竞争的线程, 使用自旋会消耗CPU   | 追求响应时间, 同步快执行速度非常快 |
| 重量级锁 | 线程竞争不适用自旋, 不会消耗CPU                              | 线程堵塞, 响应时间缓慢                          | 追求吞吐量, 同步快执行时间速度较长 |

### 1.7. 谈谈synchronized 和ReentrantLock的区别

#### 1.7.1 两者都是可重入锁

**“可重入锁”** 指的是自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁。

#### 1.7.2 synchronized依赖于JVM而ReenttrantLock依赖于API

`synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

#### 1.7.3ReenttrantLock比synchronized增加了一些高级功能

相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来制定是否是公平的。
- **可实现选择性通知（锁可以绑定多个条件）**: `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。`ReentrantLock`类当然也可以实现，但是需要借助于`Condition`接口与`newCondition()`方法。

> `Condition`是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器），**线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用`notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用`ReentrantLock`类结合`Condition`实例可以实现“选择性通知”** ，这个功能非常重要，而且是 Condition 接口默认提供的。而`synchronized`关键字就相当于整个 Lock 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而`Condition`实例的`signalAll()`方法 只会唤醒注册在该`Condition`实例中的所有等待线程。

## 2.volatile 关键字

作用：防止JVM指令重排，保证变量的可见性

### 2.1并发编程的三个重要特性

1. **原子性** : 一个的操作或者多次操作，要么所有的操作全部都得到执行并且不会收到任何因素的干扰而中断，要么所有的操作都执行，要么都不执行。`synchronized` 可以保证代码片段的原子性。
2. **可见性** ：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。`volatile` 关键字可以保证共享变量的可见性。
3. **有序性** ：代码在执行的过程中的先后顺序，Java 在编译器以及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。`volatile` 关键字可以禁止指令进行重排序优化。

### 2.2 volatile关键字和synchronized关键字的区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- **`volatile` 关键字**是线程同步的**轻量级实现**，所以**`volatile `性能肯定比`synchronized`关键字要好**。但是**`volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块**。
- **`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。**
- **`volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。**

## 3.ThreadLocal

### 3.1ThreadLocal简介：

**`ThreadLocal`类主要解决的就是让每个线程绑定自己的值，可以将`ThreadLocal`类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。**

### 3.2 ThreadLocal原理：

`ThreadLocal`类的`set()`方法：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

通过上面这些内容，我们足以通过猜测得出结论：**最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是`ThreadLocalMap`的封装，传递了变量值。** `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

**每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。**

### 3.3 ThreadLocal 内存泄漏问题

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法

## 4.线程池

### 4.1 线程池简介

> **池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。**
>
> 事先准备好一些资源，有人要用，就来我这里拿，用完之后还给我

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。

- **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。

- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

  > 线程复用，可以控制最大并发数，管理线程

### 4.2 .实现Runnable和Callable 的区别

实现Runnable接口可以使代码更加简介，但没有返回值，无法抛出异常

实现Callable接口可以有返回值和抛出异常

### 4.3.执行execute()方法和执行submit() 方法的区别

1. **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
2. **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### 4.4.如何创建线程池

不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

```java
public class Demo01 {
    public static void main(String[] args) {
        //不推荐使用以下三种方式创建线程池
        ExecutorService threaPpool1 = Executors.newSingleThreadExecutor();//单个线程
        ExecutorService threaPpool2 = Executors.newFixedThreadPool(10);//创建一个固定的线程池大小
        ExecutorService threaPpool3 = Executors.newCachedThreadPool();//可伸缩
        //推荐使用下面的方法创建线程池
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor();
        try {
            for(int i=0;i<10;i++){
                threaPpool1.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 4.4 .构造函数的重要参数分析

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` :** 核心线程数线程数定义了最小可以同时运行的线程数量。（workQueue未满时，同时运行的线程数）
- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。（阻塞队列）

触发拒绝策略的临界值为corePoolSize+maximumPoolSize+workQueue

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **`unit`** : `keepAliveTime` 参数的时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。
4. **`handler`** :饱和策略。关于饱和策略下面单独介绍一下。

### 4.5 四个拒绝策略

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

具体参数应该如何填写？（调优）

```java
    //1.CPU 密集型，maximumPoolSize  填服务器CPU核心数  
    //2.IO 密集型， 判断你程序中十分耗IO的线程 ， maximumPoolSize 填 程序中十分耗IO的线程数*2
Runtime.getRuntime().avilableProcessors()  //通过该代码拿到运行的CPU核心数
```

## 5.集合类不安全

**会遇到 ConcurrentModificationException  并发修改异常**

> List不安全，并发下ArrayList不安全

解决方案：1.使用Vector解决（不好的解决方案，效率不高）
				  2.Collections.synchronizedList(new ArrayList<>())
			      3.CopyOnWriteArrayList<>()（最佳回答）COW写入时复制，在写入的时候避免覆盖，造成数据问题
3比1强在哪里？
效率更高，底层使用Lock锁

> Set不安全，并发下HashSet不安全

解决方案：1.Collections.synchronizedSet   使用集合类下面的synchronized的set来解决线程不安全
			      2.CopyOnWriteSetArraySet类比List

HashSet底层：就是HashMap，key是无法重复的

> HashMap 不安全

解决方案：1.Collections.synchronizedMap   使用集合类下面的synchronized的set来解决线程不安全
			      2.ConcurrentHashMap<>()

## 6.三大常用辅助类

### 6.1.CountDownLatch

```java
//计数器,总数是6，必须要执行任务的时候，再使用
CountDownLatch countDownLatch = new CountDownLatch(6);

for(int i=1;i<=6;i++){
    new Thread(()->{
        System.out.println(Thread.currentThread().getName()+"Go out");
        countDownLatch.countDown();//-1操作,数量减一
    },String.valueOf(i)).start();
}
countDownLatch.await();//等待计数器归零，再向下执行
System.out.println("关门");
```

### 6.2 CyclicBarrier

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
    System.out.println("召唤神龙成功！");
});
for (int i = 1; i <= 7; i++) {
    final int temp = i;
    new Thread(()->{
        System.out.println(Thread.currentThread().getName()+"收集"+temp+"个龙珠");
        try {
            cyclicBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }).start();
```

### 6.3 Semaphore

```java
//参数为线程数量，停车位!限流！
        Semaphore semaphore = new Semaphore(6);
        for (int i = 1; i <=6 ; i++) {
            new Thread(()->{
                //acquire 得到
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
                //release 释放
            }).start();
        }
```











## 7.函数型接口和段定型接口（重点，4大）

lambda表达式，链式编程，函数式接口，Stream流式计算

> 函数式接口：只有一个方法的接口

### 5.1