# 一、理论基础

## 1、什么是线程？什么是进程？

**:horse: 进程**:

进程是程序的一次执行过程，是**系统运行程序的基本单位**，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。在一个JAVA程序中main方法开始执行就表示启动了一个JVM进程，其中main方法就是一个主线程。

:tiger:**线程：**

**线程是进程当中的一条执行流程。**同一个进程内多个线程之间可以共享代码段、数据段、打开的文件等资源，但每个线程各自都有一套独立的寄存器和栈，这样可以确保线程的控制流是相对独立的。对于一个JAVA程序，每个线程的程序计数器，本地方法栈，虚拟机栈是线程私有的，各个线程在切换时开销要比进程间切换小得多，所以**线程也被称为轻量级进程。**

## 2、进程与线程有什么区别？

- **进程**是**资源（内存、文件等）分配**的基本单位，**线程**是**CPU调度**的单位。
- 不同进程间的资源是私有的，而同一进程间的不同资源只独享一部分资源，即寄存器，栈等
- 线程切换时的开销要小于线程

**线程相比进程能减少开销，体现在：**

- 线程的创建时间比进程快，因为进程在创建的过程中，还需要资源管理信息，比如内存管理信息、文件管理信息，而**线程在创建的过程中，不会涉及这些资源管理信息，而是共享它们**；
- **线程的终止时间比进程快**，因为线程释放的资源相比进程少很多；
- 同一个进程内的线程切换比进程切换快，因为线程具有相同的地址空间（虚拟内存共享），这意味着**同一个进程的线程都具有同一个页表，那么在切换的时候不需要切换页表**。而对于进程之间的切换，切换的时候要把页表给切换掉，而页表的切换过程开销是比较大的；
- 由于同一进程的各线程间**共享内存和文件资源**，那么在线程之间数据传递的时候，就不需要经过内核了，这就使得**线程之间的数据交互效率更高了；**

## 3、什么是上下文切换？

CPU通过时间片分配算法来循环执行任务，当一个任务用完了CPU分配给自己的CPU时间片时，就会切换到下一个任务，这时就发生了上下文切换。并且在切换前CPU会保存上一个任务的状态，用于下次任务状态的恢复。

## 4、什么是死锁？如何避免？

### :monkey_face: 死锁发生的四个必要条件：

1. 互斥条件：多个线程**不能同时共享**一个资源。
2. 请求与保持条件：线程 A 在等待线程B持有的资源 2 的同时并**不会释放自己已经持有的资源 1**。
3. 不剥夺条件：当线程已经持有了资源 ，**在自己使用完之前不能被其他线程获取**
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

### :monkey:避免死锁的方法：

产生死锁的四个必要条件是：互斥条件、持有并等待条件、不可剥夺条件、环路等待条件。

那么避免死锁问题就只需要**破环其中一个条件**就可以，最常见的并且可行的就是**使用资源有序分配法，来破环环路等待条件**。

- 避免一个线程同时获取多个锁；
- 避免一个线程在锁内同时占有多个资源，尽量保证每个锁只占有一个资源；
- 尝试使用定时锁，使用`tryLock(timeout) `来替代使用内部锁机制;
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况；

## 5、并发三要素（JMM的三大特性）

### :money_mouth_face:**原子性：**

即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。一条赋值语句在底层是由多条CPU指令构成的，由于**CPU分时复用（线程切换）**的存在，对于线程A对变量i的++操作可能还未执行到写回这一阶段就发生了上下文切换，此时线程B读到i的值任为0，这将导致两个线程在执行完以后，i的值为1而并发我们期待的2.

```java
static int i = 0;
i++;
```



### :money_mouth_face: 有序性：

在执行程序时为了提高性能，**编译器和处理器会在不影响程序数据依赖关系的情况下对指令进行重排序**

指令重排一般分为**以下三种：**

- **编译器优化重排**

编译器在**不改变单线程程序语义**的前提下，可以重新安排**语句**的执行顺序。

- **指令并行重排**

现代处理器采用了指令级并行技术来将多条指令重叠执行。如果**不存在数据依赖性**(即后一个执行的语句无需依赖前面执行的语句的结果)，处理器可以改变语句对应的机器指令的执行顺序。

- **内存系统重排**

由于处理器使用缓存和读写缓存冲区，这使得加载(load)和存储(store)操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。

指令重排序可以保证单线程环境下语义的一至，**但无法保证多线程环境下语义的一至**，所以可能会发生意料之外的错误。

比如**单例模式volatile就是为了解决这一问题。**

### :money_mouth_face: 可见性：

一个线程对变量的修改对另一个线程立马可见。由于CPU存在L1，L2，L3三级缓存（在JMM中简化为主内存和工作内存），为提高程序运行性能，一个线程对一个变量的写入或修改并非立马写回主存中，而是暂时写回自己所在CPU中的缓存（工作内存）中，所以另一个线程来读取这个变量时无法立马感知到变量的变更。所以在JAVA中，volatile关键字就解决了多线程并发下的可见性问题。

## 6.线程安全的实现方法

### :mouse:互斥同步：

synchronized关键字和和ReentrantLock类可实现互斥同步。

- 在早期版本中synchronized关键字是依赖底层的**操作系统的 `Mutex Lock`** 来实现的，**Java 的线程是映射到操作系统的原生线程之上的**。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。在JDK1.6之后官方对其进行了大量优化，支持轻量级锁，偏向锁，锁消除等。
- ReentrantLock则是JAVA并发包下的一个类，它是可重入的并且支持公平锁和非公平锁，是基于AQS实现的，其底层加锁的方法是由CAS支持的。

### :ox:非阻塞同步：

互斥同步是一种悲观的做法，即同步时会发生阻塞。而非阻塞同步是一种乐观的同步，乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。在JUC中有很多类的方法就采用了非阻塞同步，如AtomicInteger原子类同步底层就使用了CAS（compare and swap）来更新变量值，CAS是由底层硬件支持的，比如在x86硬件平台下，CAS对应于一条CPU的原子指令(cmpxchg)

### :tiger:无同步方案：

因为栈中的变量是线程私有的，所以对于栈中变量的操作不存在线程安全问题，就无需采用同步方案。

## 7、什么是协程？

协程可以理解为一种轻量级的线程，通俗易懂的讲，**线程是操作系统的资源**，当java程序创建一个线程，虚拟机会向操作系统请求创建一个线程这个过程发生了pthread_create()系统调用，虚拟机本身没有能力创建线程。所以每创建一个线程都要经过深思熟虑，因为线程的创建消耗系统资源，并且在线程的切换时也会陷入内核，CPU要为线程保留寄存器，栈等资源。而协程，看起来和线程差不多，但**创建一个协程却不用调用操作系统的功能**，编程语言自身就能完成这项操作，所以协程也被称作**用户态线程**。



# 二、线程基础

## 1、说一说JAVA线程的六个状态？

<img src="https://pdai.tech/_images/pics/ace830df-9919-48ca-91b5-60b193f593d2.png" alt="image" style="zoom:67%;" />

- 新建（NEW）：于NEW状态的线程此时尚未启动。这里的尚未启动指的是还没调用**Thread实例的start()方法**。
- 可运行（RUNABLE）：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。处于RUNNABLE状态的线程可能在JVM中运行，也有可能在等待CPU分配资源。
- 阻塞（BLOCKED）：等待获取别的线程持有的一个排他锁，如lock.lock()阻塞。
- 等待状态（WAITING）：等待其它线程的唤醒，否则会无限期等待，如Object.wait();
- 超时等待（TIME_WATING）：如调用了sleep()方法或调用了设置了超时时间的等待方法，如Object.wait(long timeout)。
- 死亡（TERMINATED）：线程执行结束或发生异常自动退出。

## 2、与线程状态相关的方法总结

### :rabbit: 线程由运行状态进入无限等待状态：

如下3个方法会使线程进入**等待状态：**

- Object.wait()：使当前线程处于等待状态直到另一个线程唤醒它；
- Thread.join()：等待线程执行完毕，底层调用的是Object实例的wait方法；
- LockSupport.park()：除非获得调用许可，否则禁用当前线程进行线程调度。

### :dragon:线程由运行状态进入超时等待状态:

- Thread.sleep(long millis)：使当前线程睡眠指定时间；
- Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
- Thread.join(long millis)：在T1线程中执行t2.join()方法,会让T1让出自己的时间片给T2,T1进入等待状态。等待当T2最多执行millis毫秒，如果millis为0，则会一直执行；
- LockSupport.parkNanos(long nanos)： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；
- LockSupport.parkUntil(long deadline)：同上，也是禁止线程进行调度指定时间；

### :snake:sleep方法与wait方法的区别：

- sleep()是Thread类的静态方法，wait()是Object类的实例方法
- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。
- Object.wait(long timeout)，假如没有被notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；

### :horse:为什么 wait() 方法不定义在 Thread 中？为什么sleep()要定义在Thread类中？

wait()方法存在的**目的是让获得对象锁的线程实现等待并释放对象锁让别的线程获取**，每个对象(Object)都有自己的对象锁，**对象头里的锁标志位会记录下当前锁被哪个线程持有**，所以要释放当前锁并让线程进入WAITING状态，自然要操作Object.

`sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。

### :sheep:可以直接调用Thread的run()方法吗？

不行。new 一个 `Thread`，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。而调用start()方法的目的就是让线程在运行前进行相应的准备工作然后自动执行run()方法里的内容，但是**直接执行run()方法会把其当作一个main()方法来执行，这种方式并不是以多线程的方式来执行的。**

### :monkey:可以多次调用start()方法吗？

不行。调用start()方法以后，该线程实例内部的**`threadStatus`成员变量会被修改为1**，如果再次调用，会抛出异常`IllegalThreadStateException`;

## 3、线程创建的方法

有三种使用线程的方法:

- 实现 Runnable 接口，重写Run方法；
- 实现 Callable 接口，重写call方法；
- 继承 Thread 类。重写Run方法；

还可以补充一种通过线程池创建，可以使用Executors工具类创建或者ThreadPoolExecutor来创建自定义线程池。

- 实现Callable接口与另外两者的区别在于Callable接口有返回值，可以**通过FutureTask来接收**。
- 继承Thread类和实现Runnable接口则无返回值
- 由于Java只支持单继承，为了提高代码的可拓展性，推荐使用实现Runnable接口来创建线程

## 4、线程中断的方法

在某些情况下，我们在线程启动后发现并不需要它继续执行下去时，需要中断线程。目前在**Java里还没有安全直接的方法来停止线程**，但是Java提供了线程中断机制来处理需要中断线程的情况。

- `public void interrupt`：这是一个**实例方法**，可以在一个线程中将设置另一个线程的中断标志，底层调用了native方法,如果该线程**处于阻塞或等待状态**，就会抛出异常`InterruptedException`。这里的中断线程**并不会立即停止线程**，而是设置线程的中断状态为true（默认是flase）；

- `public static boolean interrupted`：它有两个功能，一是**检测当前线程是否被中断**,二是**将当前中断状态清理并重写设置为false，在源码中体现为设置清理标志位为true;**

  - ```java
    public static boolean interrupted() {
            return currentThread().isInterrupted(true);
        }
    ```

- `public boolean isInterrupted`：测试当前线程是否被中断。与上面方法不同的是调用这个方法并不会影响线程的中断状态。

**由以下demo可见当t2线程被打断之后，其while条件判断中的Thread.interrupted()方法返回true并将其打断标志位再次清零，变为false。**

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Thread t2 = new Thread(() -> {
            while(!Thread.interrupted()) {

            }
            System.out.println(Thread.currentThread().getName() + "end...");
            
            }, "t2");
        System.out.println("the state of t2 is " + t2.isInterrupted());//false
        t2.interrupt();
        System.out.println("the state of t2 is " + t2.isInterrupted());//false

    }
}
```

![image-20221022165340047](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221022165340047.png)

# 三、JMM详解

## 1、JMM的定义：

**JMM是一种抽象出来的概念，是Java并发编程中的一种规范**，它是围绕着多线程三大特性**原子性、有序性、可见性**展开的，JMM定义了**线程和主内存之间的抽象关系**：线程之间的**共享变量存储在主内存**中，每一个**线程拥有自己的本地内存**，本地内存存贮了该线程与其它**线程共享的变量副本**，本地内存是一个抽象概念。由于JMM这种抽象概念的规范，**屏蔽了不同硬件平台和OS对内存访问差异**以实现让Java程序在各种平台下运行可得一致内存访问效果。

### 2、什么是重排序？

JVM能根据处理器的特性（CPU多级缓存、多核处理器等）适当对机器指令进行重排序，使指令执行更符合CPU的执行效率，更加充分发挥程序性能。

重排序分三种类型：

- **编译器优化的重排序：**编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- **指令级并行的重排序：**现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- **内存系统的重排序：**由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

## 3、为什么需要Happens-before原则？

**描述两个操作之间的内存可见性，方便程序员编写代码**，happens-before 原则的诞生是为了程序员和编译器、处理器之间的平衡。程序员追求的是易于理解和编程的强内存模型，遵守既定规则编码即可。对编译器和处理器来说，**只要不改变程序的执行结果（单线程程序和正确同步了的多线程程序），编译器和处理器怎么优化都行。**

## 4、Java中的Happends-before原则

- 程序顺序规则：一个线程中的每一个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
- start规则：如果线程A执行操作ThreadB.start()启动线程B，那么A线程的ThreadB.start（）操作happens-before于线程B中的任意操作、
- join规则：如果线程A执行操作ThreadB.join（）并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

## 5、JMM中的三大特性由什么保证？

- **原子性**

在 Java 中，可以借助`synchronized` 、各种 `Lock` 以及各种**原子类**实现原子性。

`synchronized` 和各种 `Lock` 可以保证任一时刻只有一个线程访问该代码块，因此可以保障原子性。各种原子类是利用 CAS (compare and swap) 操作（可能也会用到 `volatile`或者`final`关键字）来保证原子操作。

- **可见性**

在 Java 中，可以借助`synchronized` 、`volatile` 以及各种 `Lock` 实现可见性。

如果我们将变量声明为 `volatile` ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

- **有序性**

在 Java 中，`volatile` 关键字可以禁止指令进行重排序优化

# 四、Java并发机制底层实现原理

## 1、volatile关键字详解

### :pig:volatile关键字的作用是什么？

#### I.禁止重排序

由于编译器和处理器在没有数据依赖性的语句下为了提高程序执行性能会对底层指令进行重排序，这种重排序在单线程模式下是没有问题的，但在多线程模式下，就可能出现问题，如单例模式对象实例化的过程就**可能将一个未初始化的对象引用暴露出来。**

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

实例化一个对象其实可以分为三个步骤：

- 分配内存空间。
- 初始化对象。
- 将内存空间的地址赋值给对应的引用。

但是由于操作系统可以`对指令进行重排序`，所以上面的过程也可能会变成如下过程：

- 分配内存空间。
- 将内存空间的地址赋值给对应的引用。
- 初始化对象

在重排序后，在多线程环境下，在线程A中的singleton已经指向了一片内存空间，所以线程B进来在经过if判断后发现singleton != null,于是直接就将为singleton引用返回出去，但该引用指向的对象还未初始化，这就会发生意外错误。

#### II.保证可见性

指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。volatile关键字能有效的解决这个问题。

#### III.无法保证一个复和操作的原子性但可以保证单条读写指令的原子性

假设线程A和线程B同时对volatile修饰的共享变量i进行+1操作，虽然可以保证赋值操作结束之后立马将值写入主内存，但这并不能保证原子性，因为两个线程读取到i的值可能是同一个，所以两次赋值操作相当于只执行了一次。用原子类可以解决这一问题。

### :dog: 共享的long和double变量的为什么要用volatile?

因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此**普通的long或double类型读/写可能不是原子**的,即一次写入可能只写入了变量的高32位，另一个线程读取就出现了奇怪的数据。因此，最好将**共享的long和double变量设置为volatile类型**，这样能保证任何情况下对long和double的单次读/写操作都具有原子性。

### :elephant:volatile底层实现原理

#### I.可见性的实现

volatile 变量的内存可见性是基于**内存屏障(Memory Barrier)**实现:

- 内存屏障，又称内存栅栏，是一个 **CPU 指令**。
- 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过**插入特定类型的内存屏障**来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：**不管什么指令都不能和这条 Memory Barrier 指令重排序。**

```assembly
 0x000000000295158c: lock cmpxchg %rdi,(%rdx)  #在 volatile 修饰的共享变量进行写操作的时候会多出 lock 前缀的指令
```

lock 前缀的指令在多核处理器下会引发两件事情:

- 将当前处理器缓存行的数据**写回到系统内存。**
- 写回内存的操作会使在**其他 CPU 里缓存了该内存地址的数据无效。**

为了保证各个处理器的缓存是一致的，实现了缓存一致性协议(MESI)，每个处理器通过**嗅探在总线上传播的数据**来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

所有多核处理器下还会完成：当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

#### II.有序性的实现

- **happens-before 规则**中有一条是 volatile 变量规则：**对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。**

```java
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;
    
    public void writer() {
        a = 1;              // 1 线程A修改共享变量
        flag = true;        // 2 线程A写volatile变量
    } 
    
    public void reader() {
        if (flag) {         // 3 线程B读同一个volatile变量
        int i = a;          // 4 线程B读共享变量
        ……
        }
    }
}
```

根据程序次序规则：1 happens-before 2 且 3 happens-before 4。

**根据 volatile 规则：2 happens-before 3。**

根据 happens-before 的传递性规则：1 happens-before 4。

**因此当线程A将flag修改位true后，线程B能立马感知到。**

- **JMM 提供了内存屏障阻止重排序**

在 Java 中，`Unsafe` 类提供了三个开箱即用的**内存屏障相关的方法，屏蔽了操作系统底层的差异：**

```java
public native void loadFence();
public native void storeFence();
public native void fullFence();
```

为了实现 volatile 内存语义时，**编译器在生成字节码时**，会在指令序列中**插入内存屏障**来禁止特定类型的处理器重排序。

**读屏障（LoadBarrier）：在读指令之前插入读屏障，让工作内存或CPU高速缓存当中的缓存数据失效，重新回到主内存中获取最新数据**

**写屏障（StoreBarrier）：在写指令之后插入写屏障，强制把写缓冲区的数据刷回主内存中。**

**JMM提供的四种内存屏障：**

![image-20221022200425871](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221022200425871.png)

![img](https://pdai.tech/_images/thread/java-thread-x-key-volatile-2.png)

- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。

- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。

- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。

- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。


### :smiley_cat:说一下volatile的应用场景

使用 volatile 必须具备的条件

- 对变量的写操作不依赖于当前值。
- 该变量没有包含在具有其他变量的不变式中。
- 只有在状态真正独立于程序内其他内容时才能使用 volatile。

应用场景：

- 线程安全的单例模式
- 状态标志完成状态转化
- 较低开销的读多写少场景

## 2、synchronized关键字详解

### I.synchronized可以作用在哪里？

- 对象锁：
  - 代码块形式：手动指定对象或this，也可以是自定义的锁。
  - 修饰普通方法，锁定对象默认为this

- 类锁：
  - 修饰静态方法，默认的锁就是当前所在的Class类，所以无论是哪个线程访问它，需要的锁都只有一把
  - 代码块形式：指定锁对象为Class对象，所有线程一把锁

### II.synchronized 关键字的底层原理

synchronized 关键字底层原理属于 JVM 层面。通过 JDK 自带的 `javap`类的相关字节码信息,**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

**Monitor**

上面经常提到monitor，**它内置在每一个对象中，任何一个对象都有一个monitor与之关联**，synchronized在JVM里的实现就是基于进入和退出monitor来实现的，底层则是通过成对的MonitorEnter和MonitorExit指令来实现，因此每一个Java对象都有成为Monitor的潜质。所以我们可以理解monitor是一个同步工具。

### III.说一说锁升级

在 Java 早期版本中，`synchronized` 属于 **重量级锁**，效率低下。 因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

<img src="https://img-blog.csdnimg.cn/b26dfeb728a646cea614872296f3e8b6.png#pic_center" alt="img" style="zoom: 67%;" />

- 第一步，检查MarkWord里面是不是放的自己的ThreadId ,如果是，表示当前线程是处于 “偏向锁” 。
- 第二步，如果MarkWord不是自己的ThreadId，锁升级，这时候，用CAS来执行切换，新的线程根据MarkWord里面现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。
- 第三步，两个线程都把锁对象的HashCode复制到自己新建的用于存储锁的记录空间，接着开始通过CAS操作， 把锁对象的MarKword的内容修改为自己新建的记录空间的地址的方式竞争MarkWord。
- 第四步，第三步中成功执行CAS的获得资源，失败的则进入自旋 。
- 第五步，自旋的线程在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于 轻量级锁的状态，如果自旋失败 。
- 第六步，进入重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执行完成并唤醒自己

synchronized用的锁是存在java对象头Mark Word中的。**其中存储了轻量级锁、重量级锁、偏向锁的标志等。**

**无锁：初始状态，一个对象被实例化后，如果还没有被任何线程竞争锁，那么它就为无锁状态（001）**无锁没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

**偏向锁：**一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，降低获取锁的代价。在大多数情况下，锁总是由同一线程多次获得，不存在多线程竞争，所以出现了偏向锁。其目标就是在只有一个线程执行同步代码块时能够提高性能。（101）。

**轻量级锁：**是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过CAS的形式尝试获取锁，不会阻塞，从而提高性能，**但如果多次CAS后仍然获取不到锁就说明锁竞争激烈，这时就会升级为重量级锁。**

**重量级锁：在锁竞争激烈的情况下，Java中的重量级锁是基于Moniter对象实现的。在编译时会将同步块的开始位置插入moniter enter指令，这时会创建一个MoniterObject对象，在结束位置插入moniter exit指令。**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221023105156584.png" alt="image-20221023105156584" style="zoom:67%;" />

### IV.锁升级与hashcode的关系

当一个对象计算过hashcode之后，就无法再次偏向锁状态了，在第一次获取时会直接进入轻量级锁，一个对象正处于偏向锁状态，又收到计算hashcode的请求，它的偏向锁就会立马被撤销，并且会膨胀为重量级锁。

无锁状态是对象头是有位置存储hashcode的，而变为偏向锁状态是没有位置存储hashcode的，所以在计算过hashcode之后，对象头中就没有位置来存储偏向线程的id了。

### V.JIT即时编译器对锁的优化

**锁消除：**

锁消除是指虚拟机即时编译器再运行时，对一些代码上要求同步，但是被检测到**不可能存在共享数据竞争的锁进行消除**。锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM会判断再一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那JVM就把它们当作栈上数据对待，认为这些数据是线程独有的，不需要加同步。此时就会进行锁消除。

**锁粗化：**

我们都知道在加同步锁时，尽可能的将同步块的作用范围限制到尽量小的范围，如**方法中首尾相接，前后相邻的都是同一个锁对象，那JIT编译器就会把这几个synchronized块合并成一个大块。**

### VI、synchronized 的缺点

- 效率低：锁的释放情况少，只有代码执行完毕或者异常结束才会释放锁；试图获取锁的时候不能设定超时，不能中断一个正在使用锁的线程，相对而言，Lock可以中断和设置超时
- 不够灵活：加锁和释放的时机单一，每个锁仅有一个单一的条件(某个对象)，相对而言，读写锁更加灵活
- 无法知道是否成功获得锁，死锁发生时难以处理。

### VII.**使用Synchronized有哪些要注意的？**

- 锁对象不能为空，因为锁的信息都保存在对象头里
- 作用域不宜过大，影响程序执行的速度，控制范围过大，编写代码也容易出错
- 避免死锁
- 在能选择的情况下，既不要用Lock也不要用synchronized关键字，用java.util.concurrent包中的各种各样的类，如果不用该包下的类，在满足业务的情况下，可以使用synchronized关键，因为代码量少，避免出错

### VIII.**synchronized是公平锁吗？**

synchronized实际上是非公平的，新来的线程有可能立即获得监视器，而在等待区中等候已久的线程可能再次等待，这样有利于提高性能，但是也可能会导致饥饿现象。

### IX.synchronized可重入原理？

当前实例对象锁后进入 synchronized 代码块执行同步代码，并在代码块中调用了当前实例对象的另外一个 synchronized 方法，再次请求当前实例锁时，将被允许。需要特别注意另外一种情况，**当子类继承父类时，子类也是可以通过可重入锁调用父类的同步方法。**注意由于 synchronized 是**基于 monitor 实现的，因此每次重入，monitor 中的计数器仍会加 1**

### X.synchronized & Lock

**synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。**独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。

**而Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是CAS操作**（Compare and Swap）。我们可以进一步研究ReentrantLock的源代码，会发现其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。

- synchronized是Java语法的一个关键字，加锁的过程是在JVM底层进行。Lock是一个类，是JDK应用层面的，在JUC包里有丰富的API。
- synchronized在加锁和解锁操作上都是自动完成的，Lock锁需要我们手动加锁和解锁。
- Lock锁有丰富的API能知道线程是否获取锁成功，而synchronized不能。
- synchronized能修饰方法和代码块，Lock锁只能锁住代码块。
- Lock锁有丰富的API，可根据不同的场景，在使用上更加灵活。
- synchronized是非公平锁，而Lock锁既有非公平锁也有公平锁，可以由开发者通过参数控制。

## 3.CAS原理

### I.CAS底层

CAS的全称为Compare-And-Swap，直译就是对比交换，比如一个共享变量i原来的值为1，线程A与线程B都想把它+1,然后在修改前，线程A需要先去将读取到的值与i=1作比较，如果i=1说明没有其它线程修改过，这时就可以i进行修改，否则失败。CAS是由底层硬件支持的，比如Linux的X86下主要是通过`cmpxchgl`这个指令在CPU级完成CAS操作的，但在多处理器情况下必须使用`lock`指令加锁来完成。当然**不同的操作系统和处理器的实现会有所不同**

**在Java中，有一个`Unsafe`类**，它在`sun.misc`包中。它里面是一些`native`方法，其中就有几个关于CAS的：

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

他们都是`public native`的。Unsafe中对CAS的实现是C++写的，它的具体实现和**操作系统、CPU都有关系。**

### II.ABA问题

所谓ABA问题，就是一个值原来是A，变成了B，又变回了A。这个时候使用CAS是检查不出变化的，但实际上却被更新了两次。

ABA问题的解决思路是在变量前面追加上**版本号或者时间戳**。

JDK的Atomic包里提供了一个类**AtomicStampedReference**来解决ABA问题。这个类的compareAndSet方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

```java
public class AtomicStampedDemo
{
    public static void main(String[] args)
    {
        Book javaBook = new Book(1,"javaBook");

        AtomicStampedReference<Book> stampedReference = new AtomicStampedReference<>(javaBook,1);//第二个参数代表版本号

        System.out.println(stampedReference.getReference()+"\t"+stampedReference.getStamp());

        Book mysqlBook = new Book(2,"mysqlBook");

        boolean b;
        b = stampedReference.compareAndSet(javaBook, mysqlBook, stampedReference.getStamp(), stampedReference.getStamp() + 1);

        System.out.println(b+"\t"+stampedReference.getReference()+"\t"+stampedReference.getStamp());

	//四个参数：1.期望值 2.新值 3.期望版本号 4.新版本号
        b = stampedReference.compareAndSet(mysqlBook, javaBook, stampedReference.getStamp(), stampedReference.getStamp() + 1);

        System.out.println(b+"\t"+stampedReference.getReference()+"\t"+stampedReference.getStamp());

    }
}
```

## 4.原子类引入

### I.介绍一下Java中的原子类？

在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

所以，所谓原子类说简单点就是具有原子/原子操作特征的类**。原子类是线程安全的，底层都是由Unsafe类与CAS相关的方法实现的，在对原子类进行操作时不用加锁。**

### II.JUC 包中的原子类是哪 4 类?

**基本类型**

使用原子的方式更新基本类型

- `AtomicInteger`：整型原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整型数组原子类
- `AtomicLongArray`：长整型数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- `AtomicMarkableReference` ：原子更新带有标记位的引用类型

**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整型字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整型字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器

## 5.final关键字详解

### I.final关键字的使用

- **final修饰类：**该类不可被继承，被final修饰的类中的所有方法都隐式被final修饰，不能被重写。
- **final修饰方法：**final方法可以被重载和继承，但不能被重写。private修饰的方法是隐式的final，所以不可被重写。
- **final修饰参数：**Java允许在参数列表中以声明的方式将参数指明为final，这意味这你无法在方法中更改参数引用所指向的对象（指针常量）。
- **final修饰变量**：final修饰**基本类型则初始化后不能被改变**，修饰的变量是**引用类型初始化后不能再指向其它的对象**。

**final修饰变量都是编译器常量吗？no**

编译期常量和非编译期常量

```java
public class Test {
    //编译期常量
    final int i = 1;
    final static int J = 1;
    final int[] a = {1,2,3,4};
    //非编译期常量
    Random r = new Random();
    final int k = r.nextInt();

    public static void main(String[] args) {

    }
}
```

k的值由随机数对象决定，所以不是所有的final修饰的字段都是编译期常量，只是**k的值在被初始化后无法被更改。**

**static final:**

一个既是static又是final 的字段只占据一段不能改变的存储空间，它**必须在定义的时候进行赋值，否则编译器将不予通过。**

**blank final**:

Java允许生成空白final，也就是说被声明为final但又没有给出定值的字段,但是必须在该字段被使用之前被赋值，这给予我们两种选择：

- 在定义处进行赋值(这不叫空白final)
- 在构造器中进行赋值，**保证了该值在被使用前赋值。**

这**增强了final的灵活性**。

```java
public class Test {
    final int i1 = 1;
    final int i2;//空白final
    public Test() {
        i2 = 1;
    }
    public Test(int x) {
        this.i2 = x;
    }
}
```

### II.final域的重排规则

**按照final修饰的数据类型分类：**

- 基本数据类型:
  - `final域写`：禁止final域写与构造方法重排序，即**禁止final域写重排序到构造方法之外**，从而保证该对象对所有线程可见时，该对象的final域全部已经初始化过。
  - `final域读`：禁止初次读对象的引用与读该对象包含的final域的重排序，即**先读取引用在读取引用的final域数据。**
- 引用数据类型：
  - `额外增加约束`：禁止在构造函数对一个final修饰的对象的成员域的写入与随后将这个被构造的对象的引用赋值给引用变量重排序，**在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用。**
  
  ```java
  public class FinalReferenceDemo {
      final int[] arrays;
      private FinalReferenceDemo finalReferenceDemo;
  
      public FinalReferenceDemo() {
          arrays = new int[1];  //1
          arrays[0] = 1;        //2
      }
  
      public void writerOne() {
          finalReferenceDemo = new FinalReferenceDemo(); //3
      }
  
      public void writerTwo() {
          arrays[0] = 2;  //4
      }
  
      public void reader() {
          if (finalReferenceDemo != null) {  //5
              int temp = finalReferenceDemo.arrays[0];  //6
          }
      }
  }
  ```
  
  针对上面的实例程序，线程线程A执行wirterOne方法，执行完后线程B执行writerTwo方法，然后线程C执行reader方法。下图就以这种执行时序出现的一种情况来讨论(耐心看完才有收获)。
  
  <img src="https://pdai.tech/_images/thread/java-thread-x-key-final-3.png" alt="img" style="zoom:67%;" />
  
  由于对final域的写禁止重排序到构造方法外，因此1和3不能被重排序。由于一个final域的引用对象的成员域写入不能与随后将这个被构造出来的对象赋给引用变量重排序，因此2和3不能重排序。
  
  JMM可以确保线程C至少能看到写线程A对final引用的对象的成员域的写入，即能看下arrays[0] = 1，而写线程B对数组元素的写入可能看到可能看不到。JMM不保证线程B的写入对线程C可见，线程B和线程C之间存在数据竞争，此时的结果是不可预知的。如果可见的，可使用锁或者volatile。
  
  

### III.final域的实现原理

写final域会要求编译器在**final域写之后，构造函数返回前插入一个StoreStore屏障。 **读final域的重排序规则会要求**编译器在读final域的操作前插入一个LoadLoad屏障。**

很有意思的是，如果以X86处理器为例，X86不会对写-写重排序，所以StoreStore屏障可以省略。由于不会对有间接依赖性的操作重排序，所以在X86处理器中，读final域需要的LoadLoad屏障也会被省略掉。也就是说，以X86为例的话，对final域的读/写的内存屏障都会被省略！具体是否插入还是得看是什么处理器。

### IV.使用final的限制条件和局限性

**当声明一个 final 成员时，必须在构造函数退出前设置它的值。**

```java
public class MyClass {
  private final int myField = 1;
  public MyClass() {
    ...
  }
}

public class MyClass {
  private final int myField;
  public MyClass() {
    ...
    myField = 1;
    ...
  }
}
```

**将指向对象的成员声明为 final 只能将该引用设为不可变的，而非所指的对象。**

```java
private final List myList = new ArrayList();
myList.add("Hello");
//以下操作非法
myList = new ArrayList();
myList = someOtherList;
```

# 五、JAVA中的锁

1. 乐观锁 VS 悲观锁
   1. 乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的。
   2. 悲观锁：synchronized，ReentrantLock
2. 自旋锁 VS 自适应自旋锁
   1. 自旋锁的实现原理同样也是CAS，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。
   2. JDK 6引入了自适应的自旋锁（适应性自旋锁）。自适应意味着**自旋的时间（次数）不再固定**，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，**自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中**，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。**如果对于某个锁，自旋很少成功**获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。
3. 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁：synchronized锁升级。
4. 可重入锁 VS 非可重入锁：ReentrantLock和synchronized都是重入锁
5. 独占锁 VS 共享锁：ReentrantReadWriteLock就实现了共享锁，读读共享，读写互斥，写写互斥。
6. 公平锁 VS 非公平锁：公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。公平锁的优点是等待锁的线程**不会饿死。**缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。**ReentrantLock默认使用非公平锁，也可以通过构造器来显示的指定使用公平锁。**

# 六、LockSupport详解

## 1.LockSupport是什么？

LockSupport用来创建锁和其他同步类的基本线程阻塞原语。简而言之，当调用LockSupport.park时，表示当前线程将会等待，直至获得许可，当调用LockSupport.unpark时，必须把等待获得许可的线程作为参数进行传递，好让此线程继续运行。LockSupport是由**Unsafe类的part()和unpark()支持，在Linux系统下，是用的Posix线程库pthread中的mutex（互斥量），condition（条件变量）来实现的。**

```java
    private LockSupport() {} // Cannot be instantiated.

    @HotSpotIntrinsicCandidate
    public native void park(boolean isAbsolute, long time);
	public native void unpark(Thread thread);
```

## 2.核心函数分析

1. `void park()`：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回
2. `void park(Object blocker)`：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
3. `void parkNanos(long nanos)`：阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性；
4. `void parkNanos(Object blocker, long nanos)`：功能同方法3，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
5. `void parkUntil(long deadline)`：阻塞当前线程，知道deadline；
6. `void parkUntil(Object blocker, long deadline)`：功能同方法5，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；

```java
public static void park()；
public static void park(Object blocker)；//指定线程使其阻塞
    
public static void unpark(Thread thread) {//指定线程发放许可证
        if (thread != null)
            U.unpark(thread);//释放许可
    }
```

**LockSupport没有加锁解锁要求的限制并且支持先唤醒再等待，如果先被unpark了（先发放了许可证），相当于之后park无效，两者相互抵消。**

**park和unpark成对使用并且只能出现一对（许可证只能有一个）！**

```java
//此函数表示在许可可用前禁用当前线程，并最多等待指定的等待时间。具体函数如下。
public static void parkNanos(Object blocker, long nanos) {
    if (nanos > 0) { // 时间大于0
        // 获取当前线程
        Thread t = Thread.currentThread();
        // 设置Blocker
        setBlocker(t, blocker);
        // 获取许可，并设置了时间
        UNSAFE.park(false, nanos);
        // 设置许可
        setBlocker(t, null);
    }
}

//在指定的时限前禁用当前线程，除非许可可用,
public static void parkUntil(Object blocker, long deadline) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    UNSAFE.park(true, deadline);
    // 设置Blocker为null
    setBlocker(t, null);
}
```

## 3.LockSupport的中断响应

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before interrupt");        
        try {
            // 休眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }    
        Thread thread = (Thread) object;
        // 中断线程
        thread.interrupt();
        System.out.println("after interrupt");
    }
}

public class InterruptDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
before park
before interrupt
after interrupt
after park
```

在主线程调用park被阻塞后，在myThread线程中发出了对主线程的中断信号，此时主线程会继续运行，也就是说明此时**interrupt起到的作用与unpark一样**。

## 4.Object.wait()和Condition.await()的区别

Object.wait()和Condition.await()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程。

## 5.Thread.sleep()和LockSupport.park()的区别

- 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，**且都不会释放当前线程占有的锁资源；**
- Thread.sleep()没法从外部唤醒，只能自己醒过来；
- LockSupport.park()方法可以被另一个线程调用LockSupport.unpark(Thread t)方法或Thread.interrupt()唤醒；
- Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；
- LockSupport.park()方法不需要捕获中断异常；
- Thread.sleep()本身就是一个native方法；
- LockSupport.park()底层是调用的**Unsafe的native方法**

## 6.Object.wait()和LockSupport.park()的区别

- Object.wait()方法需要在synchronized块中执行；
- LockSupport.park()可以在任意地方执行；
- Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
- LockSupport.park()不需要捕获中断异常；
- Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；
- LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；

park()/unpark()底层的原理是“二元信号量”，你可以把它相像成只有一个许可证的Semaphore，只不过这个信号量在重复执行unpark()的时候也不会再增加许可证，最多只有一个许可证。

## 7.如果在一个线程wait()之前另一个线程执行了notify()会怎样?

如果当前的线程不是此对象锁的所有者，却调用该对象的notify()或wait()方法时抛出IllegalMonitorStateException异常；

如果当前线程是此对象锁的所有者，wait()将一直阻塞，因为后续将没有其它notify()唤醒它。

## 8.LockSupport.park()会释放锁资源吗?

不会，它只负责阻塞当前线程，释放锁资源实际上是在**Condition的await()方法**中实现的。

# 七、AQS详解

## 1.AQS是什么？AQS的核心思想是什么？

- AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出大量应用广泛的同步器，比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器
- AQS**核心思想**是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用**CLH队列锁实现**的，即将暂时获取不到锁的线程加入到队列中。

- CLH(Craig,Landin,and Hagersten)队列是一个**虚拟的双向队列**(虚拟的双向队列即不存在队列实例，**仅存在结点之间的关联关系**)。AQS是将**每条请求共享资源的线程**封装成一个CLH锁队列的一个**结点(Node)**，每个节点包含对一条线程的引用，以此来实现锁的分配。

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS%E5%8E%9F%E7%90%86%E5%9B%BE.png" alt="AQS原理图" style="zoom:80%;" />

AQS 使用一个 **int 成员变量来表示同步状态**，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 **CAS 对该同步状态进行原子操作**实现对其值的修改。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性，1代表占用状态，0代表非占用状态，state为0时获取锁
```

状态信息通过 protected 类型的 **getState，setState，compareAndSetState 进行操作**

```java
//返回同步状态的当前值
protected final int getState() {
    return state;
}
//设置同步状态的值
protected final void setState(int newState) {
    state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### AQS内部Node类

```java
static final class Node {
    // 模式，分为共享与独占
    // 共享模式
    static final Node SHARED = new Node();
    // 独占模式
    static final Node EXCLUSIVE = null;        
    // 结点状态
    // CANCELLED，值为1，表示当前的线程被取消
    // SIGNAL，值为-1，表示当前节点的后继节点包含的线程需要运行，也就是unpark
    // CONDITION，值为-2，表示当前节点在等待condition，也就是在condition队列中
    // PROPAGATE，值为-3，表示当前场景下后续的acquireShared能够得以执行
    // 值为0，表示当前节点在sync队列中，等待着获取锁
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;        

    // 结点状态
    volatile int waitStatus;        
    // 前驱结点
    volatile Node prev;    
    // 后继结点
    volatile Node next;        
    // 结点所对应的线程
    volatile Thread thread;        
    // 下一个等待者
    Node nextWaiter;
    
    // 结点是否在共享模式下等待
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    
    // 获取前驱结点，若前驱结点为空，抛出异常
    final Node predecessor() throws NullPointerException {
        // 保存前驱结点
        Node p = prev; 
        if (p == null) // 前驱结点为空，抛出异常
            throw new NullPointerException();
        else // 前驱结点不为空，返回
            return p;
    }
    
    // 无参构造方法
    Node() {    // Used to establish initial head or SHARED marker
    }
    
    // 构造方法
        Node(Thread thread, Node mode) {    // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }
    
    // 构造方法
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

- `CANCELLED`，值为1，表示当前的线程被取消。
- `SIGNAL`，值为-1，表示当前节点的后继节点包含的线程需要运行，需要进行unpark操作。
- `CONDITION`，值为-2，表示当前节点在等待condition，也就是在condition queue中。
- `PROPAGATE`，值为-3，表示当前场景下后续的acquireShared能够得以执行。
- 值为0，表示当前节点在sync queue中，等待着获取锁。

## 2.AQS提供了哪些核心方法？底层运用了什么设计模式？

同步器的设计是基于**模板方法模式**的，如果需要自定义同步器一般的方式是这样(模板方法模式很经典的一个应用)：

使用者**继承AbstractQueuedSynchronizer并重写指定的方法**。(这些重写方法很简单，无非是对于共享资源state的获取和释放) 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

## 3.AQS核心方法分析

**流程梳理：**

获取资源的入口是**acquire(int arg)方法**。首先调用tryAcquire(arg)尝试去获取资源。**如果获取资源失败，就通过addWaiter(Node.EXCLUSIVE)方法把这个线程插入到等待队列中**。由于会出现**多个线程同时插入节点的操作**，在这里是通过**CAS自旋的方式保证了操作的线程安全性。**节点进入等待队列的后会判断自己的前驱节点是否时头节点，如果是则会tryAcquire尝试获取同步状态，否则调用park使它进入阻塞状态的，一直**等待其前驱节点释放资源时调用release()方法的同时调用unpark()来唤醒自己**。

### 获取资源

获取资源的入口是**acquire(int arg)方法**。arg是要获取的资源的个数，在**独占模式下始终为1**。我们先来看看这个方法的逻辑：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

首先调用tryAcquire(arg)尝试去获取资源。前面提到了这个方法是在子类具体实现的。

**如果获取资源失败，就通过addWaiter(Node.EXCLUSIVE)方法把这个线程插入到等待队列中**。其中传入的参数代表要插入的Node是独占式的。这个方法的具体实现：

```java
private Node addWaiter(Node mode) {
    // 生成该线程对应的Node节点
    Node node = new Node(Thread.currentThread(), mode);
    // 将Node插入队列中
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        // 使用CAS尝试，如果成功就返回
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 如果等待队列为空或者上述CAS失败，再自旋CAS插入
    enq(node);
    return node;
}

// 自旋CAS插入等待队列
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

由于AQS中会存在多个线程同时争夺资源的情况，因此肯定会出现**多个线程同时插入节点的操作**，在这里是通过**CAS自旋的方式保证了操作的线程安全性。**

现在通过addWaiter方法，已经把一个**Node放到等待队列尾部**了。而处于等待队列的结点是从头结点一个一个去获取资源的。具体的实现我们来看看acquireQueued方法

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋
        for (;;) {
            final Node p = node.predecessor();
            // 如果node的前驱结点p是head，表示node是第二个结点，就可以尝试去获取资源了
            if (p == head && tryAcquire(arg)) {
                // 拿到资源后，将head指向该结点。
                // 所以head所指的结点，就是当前获取到资源的那个结点或null。
                setHead(node); 
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 如果自己可以休息了，就进入waiting状态，直到被unpark()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

**结点进入等待队列后，是调用park使它进入阻塞状态的。只有头结点的线程是处于活跃状态的**。

获取资源的方法除了acquire外，还有以下三个：

- acquireInterruptibly：申请可中断的资源（独占模式）
- acquireShared：申请共享模式的资源
- acquireSharedInterruptibly：申请可中断的资源（共享模式）

### 释放资源

释放资源相比于获取资源来说，会简单许多，**当前线程释放资源以后会通知其后继节点让其获取锁。**在AQS中只有一小段实现。源码：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    // 如果状态是负数，尝试把它设置为0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    // 得到头结点的后继结点head.next
    Node s = node.next;
    // 如果这个后继结点为空或者状态大于0
    // 通过前面的定义我们知道，大于0只有一种可能，就是这个结点已被取消
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 等待队列中所有还有用的结点，都向前移动
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 如果后继结点不为空，
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

## 4.AQS定义什么样的资源获取方式? 

AQS定义了**两种资源获取方式**：

`独占`(只有一个线程能访问执行，又根据是否按队列的顺序分为**`公平锁`和`非公平锁`**，如`ReentrantLock`) 

`共享`(多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许**多个线程同时对某一资源进行读。**

# 八、ReentrantLock详解

## 1.ReentrantLock的核心是什么？怎么实现？

ReentrantLock**总共有三个内部类**，并且三个内部类是紧密相关的，下面先看三个类的关系。

![image](https://pdai.tech/_images/thread/java-thread-x-juc-reentrantlock-1.png)

说明: ReentrantLock类内部总共存在**Sync、NonfairSync、FairSync三个类**，**NonfairSync与FairSync类继承自Sync类**，**Sync类继承自AbstractQueuedSynchronizer抽象类**。

## 2.ReentrantLock是如何实现公平锁与非公平锁的？

ReentrantLock默认是非公平锁，可以在创建对象时设置fair参数来创建公平锁。**何谓公平性，是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求上的绝对时间顺序，满足FIFO**。 

非公平锁获取时（nonfairTryAcquire方法）只是简单的获取了一下当前状态做了一些逻辑处理，**并没有考虑到当前同步队列中线程等待的情况**。我们来看看公平锁的处理逻辑是怎样的，核心方法为：

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

这段代码的逻辑与nonfairTryAcquire基本上一直，唯一的不同在于**增加了hasQueuedPredecessors的逻辑判断，方法名就可知道该方法用来判断当前节点在同步队列中是否有前驱节点的判断**，如果有前驱节点说明有线程比当前线程更早的请求资源，根据公平性，当前线程请求资源失败。如果当前节点没有前驱节点的话，再才有做后面的逻辑判断的必要性。

**公平锁每次都是从同步队列中的第一个节点获取到锁，而非公平性锁则不一定，有可能刚释放锁的线程能再次获取到锁**。

**非公平锁**执行流程：

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/2/171d2d43c2112839~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

当**线程二**释放锁的时候，唤醒被挂起的**线程三**，**线程三**执行`tryAcquire()`方法使用`CAS`操作来尝试修改`state`值，如果此时又来了一个**线程四**也来执行加锁操作，同样会执行`tryAcquire()`方法。

这种情况就会出现竞争，**线程四**如果获取锁成功，**线程三**仍然需要待在等待队列中被挂起。这就是所谓的**非公平锁**，**线程三**辛辛苦苦排队等到自己获取锁，却眼巴巴的看到**线程四**插队获取到了锁。

**公平锁**执行流程：

![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/2/171d2d43c3462d6a~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

公平锁在加锁的时候，会先判断`AQS`等待队列中是存在节点，**如果存在节点则会直接入队等待**

## 3.ReentrantLock是如何实现可重入的？

重入性，就要解决两个问题：

1. 在线程获取锁的时候，如果已经获取锁的线程是当前线程的话则直接再次获取成功；
2. 由于锁会被获取n次，那么只有锁在被释放同样的n次之后，该锁才算是完全释放成功。

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    //1. 如果该锁未被任何线程占有，该锁能被当前线程获取
	if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	//2.若被占有，检查占有线程是否是当前线程
    else if (current == getExclusiveOwnerThread()) {
		// 3. 再次获取，计数加一
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

为了支持重入性，在第二步增加了处理逻辑，如果该锁**已经被线程所占有**了，会继续**检查占有线程是否为当前线程**，如果是的话，**同步状态加1返回true**，表示可以再次获取成功。每次重新获取都会对同步状态进行加一的操作，那么释放的时候处理思路是怎样的了？（依然还是以非公平锁为例）。

总结：当该锁对象已被占有时，当一个线程尝试获取锁的时候会对判断该锁是否被自己持有，若有，则修改将state变量+1,返回true，否则返回false并阻塞。**state变量相当于一个计数器，是AQS中一个属性。**

# 九、Condition接口详解

## 1.说一说Condition接口

任何一个java对象都天然继承于Object类，在线程间实现通信的往往会应用到Object的几个方法：

- wait()
- wait(long timeout)
- wait(long timeout, int nanos)
- notify()
- notifyAll()

同样的， 在**java Lock体系**下依然会有同样的方法实现等待/通知机制。

从整体上来看**Object的wait和notify/notify是与对象监视器配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性**。

## 2.说一说Condition的实现原理

创建一个condition对象是通过`lock.newCondition()`,而这个方法实际上是会new出一个**ConditionObject**对象，该类是**AQS的一个内部类**，ConditionObject实现了Condition接口。

```java
 /*
  @author Doug Lea
*/
public interface Condition {

    void await() throws InterruptedException;

    void awaitUninterruptibly();

    long awaitNanos(long nanosTimeout) throws InterruptedException;

    boolean await(long time, TimeUnit unit) throws InterruptedException;
    
    boolean awaitUntil(Date deadline) throws InterruptedException;

    void signal();

    void signalAll();
}
```

**condition是要和lock配合使用**的也就是condition和Lock是绑定在一起的，而lock的实现原理又依赖于AQS，自然而然ConditionObject就成为了AQS的一个内部类。

AQS内部维护了一个同步队列，如果是独占式锁的话，所有获取锁失败的线程的尾插入到**同步队列**，同样的，condition内部也是使用同样的方式，内部维护了一个 **等待队列**，所有调用condition.await方法的线程会加入到等待队列中，并且线程状态转换为等待状态。

ConditionObject中有**两个成员变量**，这两个节点不代表线程，相当于哨兵节点：

```java
public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        /** First node of condition queue. */
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        private transient Node lastWaiter;

        /**
         * Creates a new {@code ConditionObject} instance.
         */
        public ConditionObject() { }

        // Internal methods

        /**
         * Adds a new waiter to wait queue.
         * @return its new wait node
         */
        private Node addConditionWaiter() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            if (t != null && t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }

            Node node = new Node(Node.CONDITION);

            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
//…………
}
```

这样我们就可以看出来ConditionObject通过持有等待队列的头尾指针来管理等待队列。Node类有这样一个属性：

```java
//后继节点
Node nextWaiter;
```

**等待队列是一个单向队列**，而在之前说AQS时知道同步队列是一个双向队列。

<img src="http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/condition-02.png" alt="等待队列的示意图" style="zoom:67%;" />

我们可以多次调用`lock.newCondition()`方法创建多个condition对象，也就是一个lock可以持有多个等待队列。

而在之前利用Object的方式实际上是指在**对象Object对象监视器上只能拥有一个同步队列和一个等待队列，而并发包中的Lock拥有一个同步队列和多个等待队列**。

<img src="http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/condition-03.png" alt="AQS持有多个Condition" style="zoom:80%;" />

## 3.await()、singnal、singnalAll()流程

**当调用`condition.await()`方法后会会先释放同步状态(释放锁)，然后将同步队列中的首节点移动到等待队列中，并插入尾部，之后调用LockSupport.park（）使当前线程阻塞。**

当调用condition.singnal()时，会先检查当前线程是否获取了锁，接着获取**等待队列的首节点**，将其**移动到同步队列，尾插法**中并用LockSupport.unpark（）将其唤醒，线程被唤醒后，将从await()方法中的while循环退出并开始尝试获取同步状态，获取成功后从await方法中返回。

Condition.singnalAll()相当于对等待队列中的**每一个节点均执行一次signal()**，效果就是将等待队列中**的所有节点都全部移动到同步队列中，并唤醒每个节点的线程，让它们去尝试获取同步状态**。

![signal执行示意图](http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/condition-05.png)

## 4.wait()与await()的区别与联系

1.  wait()是Object超类中的方法，而await()是ConditionObject类里面的方法.


2. await会导致当前线程被阻塞，会释放锁，这点和wait是一样的

3. await中的lock不再使用synchronized把代码同步包装起来

4. await的阻塞需要另外的一个对象condition

5. notify是用来唤醒使用wait的线程；而signal是用来唤醒await线程。

6. 所在的超类不同使用场景也不同，wait一般用于Synchronized中，而await只能用于ReentrantLock锁中

# 十、ReentrantReadWriteLock详解

## 1.为什么要有ReentrantReadWriteLock？

而在一些业务场景中，大部分只是读数据，写数据很少，如果仅仅是读数据的话并不会影响数据正确性，而如果在这种业务场景下，依然使用独占锁的话，很显然这将是出现性能瓶颈的地方。针对这种读多写少的情况，java还提供了另外一个实现Lock接口的ReentrantReadWriteLock(读写锁)。**读写锁允许同一时刻被多个读线程访问，但是在写线程访问时，所有的读线程和其他的写线程都会被阻塞**。

1. **公平性选择**：支持非公平性（默认）和公平的锁获取方式，吞吐量还是非公平优于公平；
2. **重入性**：支持重入，读锁获取后能再次获取，写锁获取之后能够再次获取写锁，同时也能够获取读锁；
3. **锁降级**：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁

## 2.ReentrantReadWriteLock底层实现原理？

ReentrantLock实现的是Lock接口而ReentrantReadWriteLock实现的是ReadWriteLock接口，它也是基于AQS实现的。ReentrantReadWriteLock内部有三个核心类，分别是Sync抽象类继承自AQS，Sync抽象类内部还有两个类，分别为**HoldCounter**和**ThreadLocalHoldCounter**

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        // 版本序列号
        private static final long serialVersionUID = 6317671515068378041L;        
        // 高16位为读锁，低16位为写锁
        static final int SHARED_SHIFT   = 16;
        // 读锁单位
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        // 读锁最大数量
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        // 写锁最大数量
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
        // 本地线程计数器
        private transient ThreadLocalHoldCounter readHolds;
        // 缓存的计数器
        private transient HoldCounter cachedHoldCounter;
        // 第一个读线程
        private transient Thread firstReader = null;
        // 第一个读线程的计数
        private transient int firstReaderHoldCount;

    /** Returns the number of shared holds represented in count. */
    static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    /** Returns the number of exclusive holds represented in count. */
    static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

    // 计数器
    static final class HoldCounter {
        // 计数
        int count = 0;
        // Use id, not reference, to avoid garbage retention
        // 获取当前线程的TID属性的值
        final long tid = getThreadId(Thread.currentThread());
    }

    // 本地线程计数器
    static final class ThreadLocalHoldCounter
        extends ThreadLocal<HoldCounter> {
        // 重写初始化方法，在没有进行set的情况下，获取的都是该HoldCounter值
        public HoldCounter initialValue() {
            return new HoldCounter();
        }
    }
```

## 3.ReentrantReadWriteLock读写状态是如何设计的？

![读写锁的读写状态设计](http://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/ReentrantReadWriteLock-f714bdd6-917a-4d25-ac11-7e85b0ec1b14.png)

在AQS的Node内部类中的state属性记录了同步状态，state是一个32为int类型，ReentrantReadWriteLock中用高16位表示读锁，用低16位标识写锁，同时还设计了一个**独占状态掩码EXCLUSIVE_MASK**它的值是1 << 16 - 1（`0000FFFF`），在获取写锁之前将同步状态state与该掩码做与运算即可得到当**前写锁被获取的次数**。计算读锁的方法是将state右移16位，当获取读锁前会先去获取写锁数量，若写锁被写线程占有则获取失败，如果读锁获取成功则利用CAS更新同步状态。释放写锁只需同步状态减去写状态即可，然后判断state的低16位是否已经为0，为0才可真正释放锁

```java
// 高16位为读锁，低16位为写锁
static final int SHARED_SHIFT   = 16;
// 读锁单位
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
// 读锁最大数量
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
// 写锁最大数量
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
```

## 4.ThreadLocalHoldCounter和HoldCounter的作用是什么？

HoldCounter主要与读锁配套使用,HoldCounter主要有两个属性，count和tid，其中**count表示某个读线程重入的次数**，**tid表示该线程的tid字段的值**，该字段可以用来唯一标识一个线程。

```java
// 计数器
static final class HoldCounter {
    // 计数
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    // 获取当前线程的TID属性的值
    final long tid = getThreadId(Thread.currentThread());
}


// 本地线程计数器
static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
    // 重写初始化方法，在没有进行set的情况下，获取的都是该HoldCounter值
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}
```

ThreadLocalHoldCounter重写了ThreadLocal的initialValue方法，**ThreadLocal类可以将线程与对象相关联**。在没有进行set的情况下，get到的均是initialValue方法里面生成的那个HolderCounter对象。ThreadLocalHoldCounter实现了**不同线程HoldCounter之间的隔离，保证每个线程独一份HoldCounter。**

## 5.什么是锁降级？

**锁降级指的是写锁降级成为读锁。**如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指**把持住(当前拥有的)写锁，再获取到读锁，随后释放(先前拥有的)写锁的过程。**

读写锁支持锁降级，**遵循按照获取写锁，获取读锁再释放写锁的次序，写锁能够降级成为读锁**，不支持锁升级

### 锁降级前要先获取读锁的原因？

主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程(记作线程T)获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。 

### 为什么不支持锁升级？

目的也是为了保证数据的可见性，如果多个线程获取了读锁，如果一个线程在获取了读锁之后可以获取写锁发生锁升级，那么该线程对数据修该了其它线程是感知不到的，这中并发编程逻辑是错误的。

# 十一、ConcurrentHashMap详解

## 1.为什么HashTable慢? 它的并发度是什么? 

该类基本上所有的方法都采用synchronized进行线程安全的控制，所以他是为每个对象加锁，可想而知，在高并发的情况下，每次只有一个线程能够获取对象监视器锁，这样的并发性能的确不令人满意。

在JDK1.7之前，ConcurrentHashMap是由分段锁机制实现的。

简而言之，ConcurrentHashMap在对象中保存了一个**Segment数组**，即将整个Hash表划分为多个分段；而**每个Segment元素**，即每个分段则类似于一个Hashtable；这样，在执行put操作时首先根据**hash算法定位到元素属于哪个Segment**，然后对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可是实现多线程put操作。

在JDK1.7之前，ConcurrentHashMap是通过分段锁机制来实现的，所以其**最大并发度受Segment的个数限制**。因此，在JDK1.8中，ConcurrentHashMap的实现原理摒弃了这种设计，而是选择了与HashMap类似的数组+链表+红黑树的方式实现，而加锁则采用CAS和synchronized实现。**并发粒度缩小到了每一个Entry节点。**

## 2.说一下ConcurrentHashMap在JDK1.7之前的实现？

`concurrencyLevel`: 并行级别、并发数、Segment 数，**默认是 16**，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，**最多可以同时支持 16 个线程并发写**，只要它们的操作分别分布在不同的 Segment 上。这个值可以在**初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。**

### I.几个重要属性：

- initialCapacity: 初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。
- loadFactor: 负载因子，Segment 数组不可以扩容，所以这个负载因子是给**每个 Segment 内部使用的**。
- Segment 数组长度为 16，不可以扩容
- Segment[i] 的**默认大小为 2**，负载因子是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容
- 这里初始化了 segment[0]，其他位置还是 null，至于为什么要初始化 segment[0]，后面的代码会介绍



<img src="https://pdai.tech/_images/thread/java-thread-x-concurrent-hashmap-1.png" alt="img" style="zoom:50%;" />

### II.说一下put过程

根据 hash 值很快就能找到相应的 Segment，之后就是 Segment 内部的 put 操作了。由于外部Segment已经被加锁了，内部的实现就简单了，进入Segment内部以后，再利用 hash 值，求应该放置的数组下标，如果没有发生Hash冲突，直接加入，如果发生了，则顺着链表将其加在链表尾部。

### III.说一下扩容过程

首先，我们要回顾一下触发扩容的地方，put 的时候，如果判断该值的插入会导致该 segment 的元素个数超过阈值，那么先进行扩容，再插值，该方法不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的。扩容的时候，首先会创建一个原数组容量两倍的数组，这时对原数组里的元素进行rehash重新计算散列值再插入到新的数组里。

## 3.说一下ConcurrentHashMap在JDK1.8的实现？

ConcurrentHashMap的在JDK1.8的实现原理选择了与HashMap类似的**数组+链表+红黑树**的方式实现，而**加锁同步则采用CAS和synchronized实现。**

<img src="https://pdai.tech/_images/thread/java-thread-x-concurrent-hashmap-2.png" alt="img" style="zoom:50%;" />

ConcurrentHashMap中会大量使用CAS修改它的属性和一些操作。

**1.tabAt**

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```

该方法用来**获取table数组中索引为i的Node元素。**

**2.casTabAt**

```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

利用CAS操作**设置table数组中索引为i的元素**

**3.setTabAt**

```java
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

该方法用来**设置table数组中索引为i的元素**

### **ConcurrentHashMap的关键属性**

1. **table**`volatile Node<K,V>[] table`:

装载Node的数组，作为ConcurrentHashMap的数据容器，采用懒加载的方式，直到第一次插入数据的时候才会进行初始化操作，数组的大小总是为2的幂次方。

1. **nextTable**`volatile Node<K,V>[] nextTable;`

扩容时使用，平时为null，只有在扩容的时候才为非null

1. **sizeCtl**`volatile int sizeCtl;`

该属性用来控制table数组的大小，根据是否初始化和是否正在扩容有几种情况：

- **当值为负数时：** 如果为-1表示正在初始化，如果为-N则表示当前正有N-1个线程进行扩容操作；
- **当值为正数时：** 如果当前数组为null的话表示table在初始化过程中，sizeCtl表示为需要新建数组的长度；
- 若已经初始化了，表示当前数据容器（table数组）可用容量也可以理解成临界值（插入节点数超过了该临界值就需要扩容），具体指为数组的长度n 乘以 加载因子loadFactor；
- 当值为0时，即数组长度为默认初始值。

1. `sun.misc.Unsafe U`

在ConcurrentHashMapde的实现中可以看到大量的U.compareAndSwapXXXX的方法去修改ConcurrentHashMap的一些属性。

这些方法实际上是利用了CAS算法保证了线程安全性，这是一种乐观策略，假设每一次操作都不会产生冲突，当且仅当冲突发生的时候再去尝试。

而CAS操作依赖于现代处理器指令集，通过底层**CMPXCHG**指令实现。CAS(V,O,N)核心思想为：**若当前变量实际值V与期望的旧值O相同，则表明该变量没被其他线程进行修改，因此可以安全的将新值N赋值给变量；若当前变量实际值V与期望的旧值O不相同，则表明该变量已经被其他线程做了处理，此时将新值N赋给变量操作就是不安全的，在进行重试**。

### 初始化过程：

```java
public ConcurrentHashMap(int initialCapacity) {
	//1. 小于0直接抛异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
	//2. 判断是否超过了允许的最大值，超过了话则取最大值，否则再对该值进一步处理
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
	//3. 赋值给sizeCtl
    this.sizeCtl = cap;
}
```

**调用构造器方法的时候并未构造出table数组（可以理解为ConcurrentHashMap的数据容器），只是算出table数组的长度，并把长度大小赋值给sizeCtl属性当第一次向ConcurrentHashMap插入数据的时候才真正的完成初始化创建table数组的工作**。ConcurrentHashMap的大小一定是2的幂次方，比如，当指定大小为18时，为了满足2的n次幂特性，实际上concurrentHashMapd的大小为2的5次幂（32）。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
			// 1. 保证只有一个线程正在进行初始化操作
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
					// 2. 得出数组的大小
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
					// 3. 这里才真正的初始化数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
					// 4. 计算数组中可用的大小：实际大小n*0.75（加载因子）
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

正在进行初始化的线程会调用**U.compareAndSwapInt方法将sizeCtl改为-1即正在初始化的状态。**有可能存在一个情况是多个线程同时执行初始化方法，为了保证能够正确初始化，在第1步中会先通过if进行判断，若当前已经有一个线程正在初始化**即sizeCtl值变为-1**，这个时候其他线程在If判断为true从而调用**Thread.yield()让出CPU时间片。** **sizeCtl**的不同值来代表不同含义，起到了控制的作用。

### put()方法流程

1. 首先对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在 table中的位置；
2. 如果当前table数组还未初始化，先将table数组进行初始化操作；
3. 如果这个位置是null的，那么使用**CAS操作直接放入**；
4. 如果这个位置存在结点，说明发生了hash碰撞，首先判断这个节点的类型。如果该节点fh==MOVED(代表forwardingNode,数组正在进行扩容)的话，说明正在进行扩容，这时会让当前线程去帮助扩容(resize),t提高扩容效率；
5. 如果是链表节点（fh>0）,则得到的结点f就是hash值相同的节点组成的链表的头节点。此时会用**synchronized锁住头节点**，然后需要依次向后遍历确定这个新加入的值所在位置。如果遇到key相同的节点，则只需要覆盖该结点的value值即可。否则依次向后遍历，直到链表尾插入这个结点；
6. 如果这个节点的类型是TreeBin的话，直接调用红黑树的插入方法进行插入新的节点；
7. 插入完节点之后再次检查链表长度，如果长度大于8，就把这个链表转换成红黑树；
8. 对当前容量大小进行检查，如果超过了临界值（实际大小*加载因子）就需要扩容。

### transfer方法（扩容）

当ConcurrentHashMap容量不足的时候，需要对table进行扩容。这个方法的基本思想跟HashMap是很像的，但是由于它是**支持并发扩容的**，所以要复杂的多。原因是**它支持多线程进行扩容操作**，而并没有加锁。我想这样做的目的不仅仅是为了满足concurrent的要求，而是希望利用**并发处理去提高扩容的效率**。

扩容操作分为**两个部分**：

**第一部分**是构建一个nextTable,它的容量是原来的两倍，这个操作是**单线程完成的。**

**第二个部分**就是将原来table中的元素迁移到到nextTable中，**原数组长度为 n，所以我们有 n 个迁移任务**，让每个线程每次负责一个小任务是最简单的，每做完一个任务再检测是否有其他没做完的任务，帮助迁移就可以了，其实就是将一个大的迁移任务分为了一个个任务包，多个线程配合迁移数据。

# 十二、ConcurrentLinkedQueue详解

## 1.ConcurrentLinkedQueue的底层数据结构是什么？

ConcurrentLinkedQueue的数据结构与LinkedBlockingQueue的数据结构相同，都是使用的**链表结构**。ConcurrentLinkedQueue的数据结构如下:

![img](https://pdai.tech/_images/thread/java-thread-x-juc-concurrentlinkedqueue-1.png)l

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {}
```

说明: ConcurrentLinkedQueue继承了抽象类AbstractQueue，AbstractQueue定义了对队列的基本操作；同时实现了Queue接口，Queue定义了对队列的基本操作，同时，还实现了Serializable接口，表示可以被序列化

## 2.ConcurrentLinkedQueue保证多线程安全的底层原理？

ConcurrentLinkedQueue实现多线程安全是**全程无锁**的，底层使用**CAS保证线程安全**的。

在队列进行出队入队的时候免不了对节点需要进行操作，在多线程就很容易出现线程安全的问题。可以看出在处理器指令集能够支持**CMPXCHG**指令后，在java源码中涉及到并发处理都会使用CAS操作，那么在ConcurrentLinkedQueue对Node的CAS操作有这样几个：

```java
//更改Node中的数据域item	
boolean casItem(E cmp, E val) {
    return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
}
//更改Node中的指针域next
void lazySetNext(Node<E> val) {
    UNSAFE.putOrderedObject(this, nextOffset, val);
}
//更改Node中的指针域next
boolean casNext(Node<E> cmp, Node<E> val) {
    return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
}
```

可以看出这些方法实际上是通过调用UNSAFE实例的方法，UNSAFE为**sun.misc.Unsafe**类，该类是hotspot底层方法，目前为止了解即可，知道CAS的操作归根结底是由该类提供就好。

## 3.说一说ConcurrentLinkedQueue的HOPS（延迟更新策略）的设计？

让tail节点永远是队列的尾节点，这样虽实现起来简单，但是当有频繁的入队操作时，就要频繁地CAS更新尾节点，这样**入队效率比较低**，所以采用了延迟更新策略(HOPS)，不是每次入队时都更新尾节点，而是当tail节点的距离和真正尾节点的距离大于等于常量（HOPS，默认为1）时才更新尾节点。

`tail更新触发时机`：当tail指向的节点的下一个节点不为null的时候，会执行定位队列真正的队尾节点的操作，找到队尾节点后完成插入之后才会通过casTail进行tail更新；**当tail指向的节点的下一个节点为null的时候，只插入节点不更新tail。**

`head更新触发时机`：当head指向的节点的item域`（head的value值）`为null的时候，会执行定位队列真正的队头节点的操作，找到队头节点后完成删除之后才会通过updateHead进行head更新；**当head指向的节点的item域不为null的时候，只删除节点不更新head。**

- 如上图所示，为ConcurrentLinkedQueue的初始状态，remove(10)后的状态如下图所示

![img](https://pdai.tech/_images/thread/java-thread-x-juc-concurrentlinkedqueue-9.png)

- 如上图所示，当执行remove(10)后，head指向了head结点之前指向的结点的下一个结点，并且head结点的item域置为null。继续执行remove(20)，状态如下图所示

![img](https://pdai.tech/_images/thread/java-thread-x-juc-concurrentlinkedqueue-10.png)

- 如上图所示，执行remove(20)后，head与tail指向同一个结点，item域为null。

**head和tail的更新是“跳着的”即中间总是间隔了一个**

## 4.ConcurrentLinkedQueue适合的场景

ConcurrentLinkedQueue通过**无锁**来做到了更高的并发量，是个高性能的队列，但是使用场景相对不如阻塞队列常见，毕竟取数据也要不停的去循环，不如阻塞的逻辑好设计，但是在**并发量特别大的情况下，是个不错的选择，性能上好很多**，而且这个队列的设计也是特别费力，尤其的使用的改良算法和对哨兵的处理。整体的思路都是比较严谨的，这个也是使用了无锁造成的，我们自己使用无锁的条件的话，这个队列是个不错的参考。

# 十三、CopyOnWriteArrayList详解

## 1.为什么要有CopyOnWriteArrayList？

ArrayList并不是线程安全的，在读线程在读取ArrayList的时候如果有写线程在写数据的时候，基于fast-fail机制，会抛出**ConcurrentModificationException**异常，也就是说ArrayList并不是一个线程安全的容器，当然您可以用**Vector**,或者使用Collections的静态方法将ArrayList包装成一个线程安全的类，但是这些方式都是采用java关键字**synchronzied对方法进行修饰**，利用独占式锁来保证线程安全的。但是，由于独占式锁在同一时刻只有一个线程能够获取到对象监视器，很显然这种方式效率并不是太高。

CopyOnWriteArrayList容器可以保证线程安全，**保证读读之间在任何时候都不会被阻塞**，CopyOnWriteArrayList也被广泛应用于很多业务场景之中

## 2.说一说COW设计思想

COW通俗的理解是当我们往一个容器添加元素的时候，**不直接往当前容器添加**，而是先**将当前容器进行Copy**，复制出一个新的容器，然后**向新的容器里添加元素**，添加完元素之后，再将原容器的引用指向新的容器。

对CopyOnWrite容器进行并发的读的时候，不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种**读写分离的思想**，延时更新的策略是通过在写的时候针对的是不同的数据容器来实现的，放弃数据实时性达到数据的最终一致性。

## 3.说一说CopyOnWriteArrayList的实现原理

### I.get方法实现原理

get方法的源码为：

```java
/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

并且该数组引用是被volatile修饰，注意这里**仅仅是修饰的是数组引用**，其中另有玄机，稍后揭晓。关于volatile很重要的一条性质是它能够够保证可见性。

```java
public E get(int index) {
    return get(getArray(), index);
}
/**
 * Gets the array.  Non-private so as to also be accessible
 * from CopyOnWriteArraySet class.
 */
final Object[] getArray() {
    return array;
}
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

可以看出来get方法实现非常简单，几乎就是一个“单线程”程序，没有对多线程添加任何的线程安全控制，也**没有加锁也没有CAS操作**等等，原因是，所有的读线程只是会读取数据容器中的数据，并不会进行修改

### II.add方法实现原理

add方法的源码为：

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
	  //1. 使用Lock,保证写线程在同一时刻只有一个
    lock.lock();

    try {
				//2. 获取旧数组引用
        Object[] elements = getArray();
        int len = elements.length;

				//3. 创建新的数组，并将旧数组的数据复制到新数组中
        Object[] newElements = Arrays.copyOf(elements, len + 1);

				//4. 往新数组中添加新的数据	        
				newElements[len] = e;

				//5. 将旧数组引用指向新的数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

add方法的逻辑也比较容易理解，请看上面的注释。需要注意这么几点：

1. 采用ReentrantLock，保证同一时刻**只有一个写线程正在进行数组的复制**，否则的话内存中会有多份被复制的数据；
2. 前面说过数组引用是volatile修饰的，因此**将旧的数组引用指向新的数组**，根据volatile的happens-before规则，**写线程对数组引用的修改对读线程是可见的**。
3. 由于在写数据的时候，是在新的数组中插入数据的，从而保证读写是在两个不同的数据容器中进行操作

### **III.COW vs 读写锁**

**相同点：**

1. 两者都是通过读写分离的思想实现；
2. 读线程间是互不阻塞的

**不同点：**

对读写锁而言，为了实现数据实时性，在写锁被获取后，读线程会等待或者当读锁被获取后，写线程会等待，从而解决“脏读”等问题。也就是说如果使用读写锁依然会出现读线程阻塞等待的情况。

而COW则**完全放开了牺牲数据实时性而保证数据最终一致性**，即读线程对数据的**更新是延时感知**的，因此读线程不会存在等待的情况。

### **IV.COW的缺点**

CopyOnWrite容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题。所以在开发的时候需要注意一下。

1. **内存占用问题**：因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对 象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对 象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比 如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能**造成频繁的minor GC和major GC。**

1. **数据一致性问题**：CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

# 十四、BlockingQueue详解

## 1.BlockingQueue是什么?

BlockingQueue（阻塞队列）是一个接口，定义了阻塞插入和阻塞移除操作。相比于普通的队列，支持阻塞插入和阻塞移除两种操作。

- 阻塞插入的方法：当队列满时，一个线程请求向队列中插入数据，该线程会阻塞直到队列不满。
- 阻塞移除的方法：当队列为空时，一个线程请求向对列中移除元素，该线程会阻塞直到有元素加入队列。

**在Java7中提供了7种阻塞队列的实现类：**

- ArrayBlockingQueue：数组结构的有界阻塞队列
- LinkedBlockingQueue：链表结构的无界阻塞队列
- PriorityBlockingQueue：支持优先级排序的无界阻塞队列
- DelayQueue：使用优先级队列实现的无界阻塞队列
- SynchronousQueue：不存在元素的阻塞队列，它的内部同时只能够容纳单个元素。即有一个元素就无法插入，必须边拿边放。
- LinkedBlockingDeque：链表结构的双向阻塞队列
- LinkedTransferQueue：链表结构的无界阻塞队列

## 2.BlockingQueue 的方法

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下:

![image-20221023191813262](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221023191813262.png)

- **抛异常:** 如果试图的操作无法立即执行，抛一个异常。
- **特定值:** 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
- **阻塞:** 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
- **超时:** 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

## 3.BlockingQueue的应用场景

阻塞队列（BlockingQueue）被广泛使用在“生产者-消费者”问题中，其原因是BlockingQueue提供了可阻塞的插入和移除的方法。**当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止**。

## 4.BlockingDeque 与BlockingQueue关系

**BlockingDeque 接口继承自 BlockingQueue 接口。**这就意味着你可以像使用一个 BlockingQueue 那样使用 BlockingDeque。

BlockingDeque 类是一个双端队列，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。

在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。

## 5.BlockingDeque 的方法

一个 BlockingDeque - 线程在双端队列的两端都可以插入和提取元素。 一个线程生产元素，并把它们插入到队列的任意一端。如果双端队列已满，插入线程将被阻塞，直到一个移除线程从该队列中移出了一个元素。如果双端队列为空，移除线程将被阻塞，直到一个插入线程向该队列插入了一个新元素。

BlockingDeque 具有 4 组不同的方法用于插入、移除以及对双端队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下:

|      | 抛异常         | 特定值        | 阻塞         | 超时                             |
| ---- | -------------- | ------------- | ------------ | -------------------------------- |
| 插入 | addFirst(o)    | offerFirst(o) | putFirst(o)  | offerFirst(o, timeout, timeunit) |
| 移除 | removeFirst(o) | pollFirst(o)  | takeFirst(o) | pollFirst(timeout, timeunit)     |
| 检查 | getFirst(o)    | peekFirst(o)  |              |                                  |

|      | 抛异常        | 特定值       | 阻塞        | 超时                            |
| ---- | ------------- | ------------ | ----------- | ------------------------------- |
| 插入 | addLast(o)    | offerLast(o) | putLast(o)  | offerLast(o, timeout, timeunit) |
| 移除 | removeLast(o) | pollLast(o)  | takeLast(o) | pollLast(timeout, timeunit)     |
| 检查 | getLast(o)    | peekLast(o)  |             |                                 |

四组不同的行为方式解释:

- 抛异常: 如果试图的操作无法立即执行，抛一个异常。
- 特定值: 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
- 阻塞: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
- 超时: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

## 6.数组阻塞队列 ArrayBlockingQueue

**ArrayBlockingQueue 类实现了 BlockingQueue 接口。**用ReentrantLock实现同步，用condition实现阻塞与唤醒。

ArrayBlockingQueue **是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。**有界也就意味着，它不能够存储无限多数量的元素。**可以在对其初始化的时候设定这个上限**，**但之后就无法对这个上限进行修改了**(译者注: 因为它是基于数组实现的，也就具有数组的特性: 一旦初始化，大小就无法修改)。 ArrayBlockingQueue 内部以 FIFO(先进先出)的顺序对元素进行存储。队列中的头元素在所有元素之中是放入时间最久的那个，而尾元素则是最短的那个。

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {

    private static final long serialVersionUID = -817911632652898426L;

    /** The queued items */
    final Object[] items;

    /** items index for next take, poll, peek or remove */
    int takeIndex;

    /** items index for next put, offer, or add */
    int putIndex;

    /** Number of elements in the queue */
    int count;

    /** Main lock guarding all access */
    final ReentrantLock lock;

    /** Condition for waiting takes */
    private final Condition notEmpty;

    /** Condition for waiting puts */
    private final Condition notFull;
```

## 7.延迟队列 DelayQueue

**DelayQueue 实现了 BlockingQueue 接口，是一个支持延时获取元素的阻塞队列，队列中的元素必须实现Delayed接口。**

DelayQueue 对元素进行持有**直到一个特定的延迟到期**。注入其中的元素必须实现 java.util.concurrent.Delayed 接口，该接口定义:

```java
public interface Delayed extends Comparable<Delayed> {
    public long getDelay(TimeUnit timeUnit);
}
  
```

DelayQueue 将会在每个元素的 **getDelay() 方法返回的值的时间段之后才释放掉该元素。**如果返回的是 0 或者负值，延迟将被认为过期，该元素将会在 DelayQueue 的下一次 take  被调用的时候被释放掉。

传递给 getDelay 方法的 getDelay 实例是一个枚举类型，它表明了将要延迟的时间段。TimeUnit 枚举将会取以下值:

- DAYS
- HOURS
- INUTES
- SECONDS
- MILLISECONDS
- MICROSECONDS
- NANOSECONDS

**DelayQueue可以运用于以下场景：**

- 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，是一个线程循环查询DelayQueue，一但能从DelayQueue中获取元素，表示缓存有效期到了。
- 定时任务调度：使用DelayQueue保存当天将会执行的任务和时间，一单从DelayQueue获取到任务，该任务就可以开始执行了。

正如你所看到的，Delayed 接口也继承了 java.lang.Comparable 接口，这也就意味着 Delayed 对象之间可以进行对比。这个可能在对 DelayQueue 队列中的元素进行排序时有用，因此它们可以根据过期时间进行有序释放。 以下是使用 DelayQueue 的例子:

```java
public class DelayQueueExample {
 
    public static void main(String[] args) {
        DelayQueue queue = new DelayQueue();
        Delayed element1 = new DelayedElement();
        queue.put(element1);
        Delayed element2 = queue.take();
    }
}

```

DelayedElement 是我所创建的一个 DelayedElement 接口的实现类，它不在 java.util.concurrent 包里。你需要自行创建你自己的 Delayed 接口的实现以使用 DelayQueue 类。

## 8.同步队列 SynchronousQueue

SynchronousQueue 类实现了 BlockingQueue 接口。

SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。 据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。



## 9.具有优先级的阻塞队列 PriorityBlockingQueue

PriorityBlockingQueue 类实现了 BlockingQueue 接口。

PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。 所有**插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。**因此该队列中元素的排序就取决于你自己的 Comparable 实现。 注意 PriorityBlockingQueue 对于具有相等优先级(compare() == 0)的元素并不强制任何特定行为。

同时注意，如果你从一个 PriorityBlockingQueue 获得一个 Iterator 的话，该 Iterator 并不能保证它对元素的遍历是以优先级为序的。 以下是使用 PriorityBlockingQueue 的示例:

```java
BlockingQueue queue   = new PriorityBlockingQueue();
//String implements java.lang.Comparable
queue.put("Value");
String value = queue.take();
```

## 10.链阻塞双端队列 LinkedBlockingDeque

LinkedBlockingDeque 类实现了 BlockingDeque 接口。

deque(双端队列) 是 "Double Ended Queue" 的缩写。因此，双端队列是一个你可以从任意一端插入或者抽取元素的队列。

LinkedBlockingDeque 是一个双端队列，在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

以下是 LinkedBlockingDeque 实例化以及使用的示例:

```java
BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
deque.addFirst("1");
deque.addLast("2");
 
String two = deque.takeLast();
String one = deque.takeFirst()
```

# 十五、ThreadLocal详解

## 1.说一说你对ThreadLocal的理解

在多线程编程中通常解决线程安全的问题时，我们会**利用 synchronzed 或者 lock 控制线程对临界区资源的同步顺序**，但是这种加锁的方式会让未获取到锁的线程进行阻塞等待，很显然这种方式的时间效率并不是特别好。

ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类;，当不同线程将同一个共享的变量保存在ThreadLocal中时，**ThreadLocal会为每个线程创建单独的变量副本**, 避免因多线程操作共享变量而导致的数据不一致的情况。

有了ThreadLocal以后，多线程可以实现对临界资源的同时操作，不需要获取锁而发生阻塞或自旋，提高了时间效率，但同时也增加了内存消耗，所以ThreadLocal是一种以空间换时间的方式来实现对临界资源进行并发访问到手段。

## 2.说一说ThreadLocal的实现原理

### I.如何实现线程隔离？

<img src="https://ucc.alicdn.com/pic/developer-ecology/95e0a9bb21194b05ac2262944e83ee98.png" alt="1658590040855-f69d5460-ffa7-49b9-9f49-e93b9988977b.png" style="zoom:47%;" />

![ThreadLocal 数据结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/concurrent/threadlocal-data-structure.png![img](https://pic3.zhimg.com/v2-a39ad53e2dff823223d5e25dab26ce96_r.jpg)

ThreadLocal的内部维护这一个**ThreadLocalMap内部类**，在**Thread类中也维护了ThreadLocal.ThreadLocalMap成员变量**，对ThreadLocal的set()，get()，remove()操作实际上都是操作ThreadLocalMap,ThreadLocalMap是一种Hash表结构，内部维护着一个键值对形式的Entry数组，当向ThreadLocal中set元素时，实际上是以ThreadLocal对象本身为key,把要放的元素当作value加入到map中。

那么是如何实现线程隔离的呢？很简单，当线程第一次set元素时会先获取当前线程t，同过线程本身去初始化它的成员变量ThreadLocalMap,**实现了一个线程与一个ThreadLocalMap绑定在一起**。当下次获取get()元素时，都是通过线程自己去自己的成员ThreadLocalMap，然后操作ThreadLocalMap里的value，最终实现了共享变量私有化。

- 每个Thread线程内部都有一个ThreadLocalMap。
- Map里面存储线程本地对象ThreadLocal（key）和线程的变量副本（value）。
- Thread内部的Map是由ThreadLocal维护，ThreadLocal负责向map获取和设置线程的变量值。
- 一个Thread可以有多个ThreadLocal。

每个线程都有其独有的Map结构，而Map中存有的是ThreadLocal为Key变量副本为Vaule的键值对，以此达到变量隔离的目的。

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        
        private static final int INITIAL_CAPACITY = 16;

        private Entry[] table;

        private int size = 0;
    
        private int threshold; // Default to 0
```

### II.详细说一说ThreadLocalMap?

本质上来讲, 它就是一个Map, 但是这个ThreadLocalMap与我们平时见到的Map有点不一样

- 它没有实现Map接口;
- 它没有public的方法, 最多有一个default的构造方法, 因为这个ThreadLocalMap的方法仅仅在ThreadLocal类中调用, 属于静态内部类
- ThreadLocalMap的**Entry实现继承了WeakReference<ThreadLocal<?>>**
- 该方法**仅仅用了一个Entry数组来存储Key, Value**; Entry并不是链表形式, 而是**每个bucket里面仅仅放一个Entry;**

Entry 是一个以 ThreadLocal 为 key，Object 为 value 的键值对，另外需要注意的是这里的**ThreadLocal 是弱引用，因为 Entry 继承了 WeakReference，在 Entry 的构造方法中，调用了 super(k)方法，会将 ThreadLocal 实例包装成一个 WeakReferenece。**

Entry 中的 **key 是弱引用**，当 **ThreadLocal 外部强引用被置为 null**(`ThreadLocalInstance=null`)时，那么系统 GC 的时候，根据可达性分析，这个 ThreadLocal 实例就没有任何一条链路能够引用到它，此时 ThreadLocal 势必会被回收

#### 内存泄漏问题？

由于ThreadLocal实例被包装成了一个弱引用，所以当GC时，ThreadLocalMap 中的key就会被回收，这时中就会出现 **key 为 null 的 Entry**，就没有办法访问这些 key 为 null 的 Entry 的 value，如果当前线程再迟迟不结束的话，这些 **key 为 null 的 Entry 的 value** 就会一直存在一条强引用链：**Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value 永远无法回收，造成内存泄漏。**

**解决办法：**

在调用 ThreadLocal 的 get()、set() 方法操作数据，**从指定位置开始遍历 Entry 时，会找到 Entry 不为 null，但 key 为 null 的 Entry，并删除 key 为 null 的 Entry 的 value 和对应的 Entry。**

但是，如果 ThreadLocal 实例对象的强引用被删除后，线程长时间存活，又没有再对该线程的 ThreadLocalMap 实例对象进行操作，**也就是没有再调用 get()、set() 方法，那么依然会存在内存泄漏。**

所以，避免内存泄漏最好的做法是：主动调用 ThreadLocal 对象的 remove() 方法，将设置的线程本地变量的值删除。 **remove,** 其实内部实现就是调用 ThreadLocalMap 的remove方法:**找到Key对应的Entry, 并且清除Entry的Key(ThreadLocal)置空,** 随后清除过期的Entry即可避免内存泄露。 

## 3.以ThreadLocal为key为什么要用弱引用？

- **强引用**：我们常常 new 出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
- **软引用**：使用 SoftReference 修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- **弱引用**：使用 WeakReference 修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- **虚引用**：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

**如果使用强引用**

假设threadLocal使用的是强引用，在业务代码中执行`threadLocalInstance == null`操作，以清理掉threadLocal实例的目的，但是因为threadLocalMap的Entry强引用threadLocal，因此在gc的时候进行可达性分析，threadLocal依然可达，对threadLocal并不会进行垃圾回收，这样就无法真正达到业务逻辑的目的，出现逻辑错误

**如果使用弱引用**

假设Entry弱引用threadLocal，尽管会出现内存泄漏的问题，但是在threadLocal的生命周期里（set,getEntry,remove）里，都会针对key为null的脏entry进行处理。

从以上的分析可以看出，使用弱引用的话在threadLocal生命周期里会**尽可能的保证不出现内存泄漏的问题**，达到安全的状态。

## 4.ThreadLocalMap如何解决Hash冲突的？

`HashMap`中解决冲突的方法是在数组上构造一个**链表**结构，冲突的数据挂载到链表上，如果链表长度超过一定数量则会转化成**红黑树**。

**而 `ThreadLocalMap` 中并没有链表结构，它仅仅是一个数组，所以这里不能使用 `HashMap` 解决冲突的方式了。**

为了解决散列冲突，ThreadLocalMap采用的是**开放地址法**（open addressing）

开放地址法：

开放地址法不会创建链表，当关键字散列到的数组单元已经被另外一个关键字占用的时候，就会尝试在数组中寻找其他的单元，直到找到一个空的单元。

探测数组空单元的方式有很多，这里介绍一种最简单的 -- 线性探测法。线性探测法就是从**冲突的数组单元开始，依次往后搜索空单元**，如果到数组尾部，再从头开始搜索（环形查找），**当发生hash冲突时将当前hash值+1 再mod tab.length,即重新寻找bucket**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221024144546780.png" alt="image-20221024144546780" style="zoom:67%;" />

**ThreadLocalMap 中使用开放地址法来处理散列冲突**主要是因为：

在 ThreadLocalMap 中的**散列值分散的十分均匀，很少会出现冲突。**并且 **ThreadLocalMap 经常需要清除无用的对象**，使用纯数组更加方便。

**散列分布均匀的原因：**

```java
private final int threadLocalHashCode = nextHashCode();
```

 threadLocalHashCode 是一个常量，它通过 `nextHashCode()` 函数产生。`nextHashCode()` 函数其实就是在一个 AtomicInteger 变量（初始值为0）的基础上每次累加 0x61c88647，使用 AtomicInteger 为了保证每次的加法是原子操作。而 0x61c88647 这个就比较神奇了，它可以**使 hashcode 均匀的分布在大小为 2 的 N 次方的数组里。**

## 5.ThreadLocal如何扩容？

也几乎和大多数容器一样，ThreadLocalMap 会有扩容机制，那么它的 **threshold** 又是怎样确定的呢？

```java
private int threshold; // Default to 0
/**
 * The initial capacity -- MUST be a power of two.
 */
private static final int INITIAL_CAPACITY = 16;

ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.ThreadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

/**
 * Set the resize threshold to maintain at worst a 2/3 load factor.
 */
private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```

根据源码可知，在第一次为 ThreadLocal 进行赋值的时候会创建初始大小为 16 的 ThreadLocalMap，并且通过 setThreshold 方法设置 threshold，其值为当前哈希数组长度乘以（2/3），也就是说**加载因子为 2/3。**

(加载因子是衡量哈希表密集程度的一个参数，如果加载因子越大的话，说明哈希表被装载的越多，出现 hash 冲突的可能性越大，反之，则被装载的越少，出现 hash 冲突的可能性越小。同时如果过小，很显然内存使用率不高，该值的取值应该考虑到内存使用率和 hash 冲突概率的一个平衡，如 HashMap、ConcurrentHashMap 的加载因子都为 0.75)。

这里**ThreadLocalMap 初始大小为 16**，**加载因子为 2/3**，所以哈希表可用大小为：16*2/3=10，即**哈希表可用容量为 10。**

> 扩容 resize

从 set 方法**中可以看出当 hash 表的 size 大于 threshold 的时候，会通过 resize 方法进行扩容。**

```java
/**
 * Double the capacity of the table.
 */
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
	//新数组为原数组的2倍
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
			//遍历过程中如果遇到脏entry的话直接另value为null,有助于value能够被回收
            if (k == null) {
                e.value = null; // Help the GC
            } else {
				//重新确定entry在新数组的位置，然后进行插入
                int h = k.ThreadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }
	//设置新哈希表的threshHold和size属性
    setThreshold(newLen);
    size = count;
    table = newTab;
}
```

方法逻辑**请看注释**，新建一个大小为原来数组长度的**两倍的数组**，然后遍历旧数组中的 entry 并将其插入到新的 hash 数组中，需要注意的是，**在扩容的过程中针对脏 entry （key 为 null）的话会把 value 设为 null，以便能够被垃圾回收器回收，解决隐藏的内存泄漏的问题**。

------

### **可以直接看这个：**

在 ThreadLocalMap.set() 方法的最后，如果执行完启发式清理工作后，未清理到任何 Entry，且当前数组中 Entry 的数量已经达到了扩容阈值（数组长度的三分之二），就开始执行 rehash() 逻辑。

rehash() **首先是会进行探测式清理工作，**从数组的起始位置开始遍历，**查找 key 为 null 的 Entry 并清理**。清理完成之后如果 ThreadLocal 的个数仍然大于等于扩容阈值的四分之三，那么就进行扩容操作，扩容为原来数组长度的两倍，并且设置下一次的扩容阈值为新数组长度的三分之二。

```java
private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)//启发式清理工作
                rehash();
        }
-------------------------
private void rehash() {
            expungeStaleEntries();//探测式清理工作

            // Use lower threshold for doubling to avoid hysteresis
            if (size >= threshold - threshold / 4)//果 ThreadLocal 的个数仍然大于等于扩容阈值的四分之三
                resize();
        }
---------------------------
  private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (Entry e : oldTab) {
                if (e != null) {
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }
```



## 6.ThreadLocalMap 和HashMap区别

I.HashMap 的数据结构是数组+链表

II.ThreadLocalMap的数据结构仅仅是数组

III.HashMap 是通过链地址法解决hash 冲突的问题

IV.ThreadLocalMap 是通过开放地址法来解决hash 冲突的问题,当发生hash冲突时将当前hash值+1 再mod tab.length,即重新寻找bucket。缺点：

- ​	容易产生堆积问题，不适于大规模的数据存储。
- ​	散列函数的设计对冲突会有很大的影响，插入时可能会出现多次冲突的现象。
- ​	删除的元素是多个冲突元素中的一个，需要对后面的元素作处理，实现较复杂。

V.HashMap 里面的Entry 内部类的引用都是强引用

VI.ThreadLocalMap里面的Entry 内部类中的key 是弱引用，value 是强引用

## 7.使用场景

**场景一：在重入方法中替代参数的显式传递**

 假如在我们的业务方法中需要调用其他方法，同时其他方法都需要用到同一个对象时，可以使用ThreadLocal替代参数的传递或者static静态全局变量。这是因为使用参数传递造成代码的耦合度高，使用静态全局变量在多线程环境下不安全。当该对象用ThreadLocal包装过后，就可以保证在该线程中独此一份，同时和其他线程隔离。

 例如在Spring的@Transaction事务声明的注解中就使用ThreadLocal保存了当前的Connection对象，避免在本次调用的不同方法中使用不同的Connection对象。

**场景二：全局存储用户信息**

可以尝试使用ThreadLocal替代Session的使用，当用户要访问需要授权的接口的时候，可以现在拦截器中将用户的Token存入ThreadLocal中；之后在本次访问中任何需要用户用户信息的都可以直接从ThreadLocal中拿取数据。例如自定义获取用户信息的类AuthHolder：

```java
public class AuthNHolder {
    private static final ThreadLocal<Map<String,String>> threadLocal = new ThreadLocal<>();
    public static void map(Map<String,String> map){
        threadLocal.set(map);
    }
    // 获取用户id
    public static String userId(){
        return get("userId");
    }
    // 根据键值获取对应的信息
    public static String get(String key){
        Map<String,String> map = getMap();
        return map.get(key);
    }
    // 用完清空ThreadLocal
    public static void clear(){
        threadLocal.remove();
    }
}
```

# 十六、FutureTask详解

## 1.FutureTask是什么？

FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。**如果任务尚未完成，获取任务执行结果时将会阻塞。 ** **一旦执行结束，任务就不能被重启或取消**(除非使用runAndReset执行计算)。FutureTask **常用来封装 Callable 和 Runnable**，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全**由CAS来保证**。**FutureTask的实现也是基于AQS实现的。**

<img src="https://pdai.tech/_images/thread/java-thread-x-juc-futuretask-1.png" alt="img" style="zoom:80%;" />

FutureTask实现了**RunnableFuture**接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。

## 2.说一说Future接口

Future接口代表异步计算的结果，通过Future接口提供的方法可以查看异步计算是否执行完成，或者等待执行结果并获取执行结果，同时还可以取消执行。Future接口的定义如下:

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}   
```

- `cancel()`:cancel()方法用来**取消异步任务的执行。**如果异步任务已经完成或者已经被取消，或者由于某些原因不能取消，则会返回false。如果任务还没有被执行，则会返回true并且异步任务不会被执行。如果任务已经开始执行了但是还没有执行完成，若mayInterruptIfRunning为true，则会立即中断执行任务的线程并返回true，若mayInterruptIfRunning为false，则会返回true且不会中断任务执行线程。
- `isCanceled()`:判断任务是否被取消，如果任务在结束(正常执行结束或者执行异常结束)前被取消则返回true，否则返回false。
- `isDone()`:判断任务是否已经完成，如果完成则返回true，否则返回false。需要注意的是：任务执行过程中发生异常、任务被取消也属于任务已完成，也会返回true。
- `get()`:获取任务执行结果，如果任务**还没完成则会阻塞等待直到任务执行完成**。如果任务被取消则会抛出CancellationException异常，如果任务执行过程发生异常则会抛出ExecutionException异常，如果阻塞等待过程中被中断则会抛出InterruptedException异常。
- `get(long timeout,Timeunit unit)`:带超时时间的get()版本，如果阻塞等待过程中超时则会抛出TimeoutException异常。

## 3.FutureTask的核心属性有哪些？

```java
//内部持有的callable任务，运行完毕后置空
private Callable<V> callable;

//从get()中返回的结果或抛出的异常
private Object outcome; // non-volatile, protected by state reads/writes

//运行callable的线程
private volatile Thread runner;

//使用Treiber栈保存等待线程
private volatile WaitNode waiters;

//任务状态
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;

```

有一个volatile修饰的state变量，**用于更新当前任务状态**，只要有任何一个线程修改了这个变量，那么其他所有的线程都会知道最新的值。7种状态具体表示：

- `NEW`:表示是个新的任务或者还没被执行完的任务。这是初始状态。
- `COMPLETING`:任务已经执行完成或者执行任务的时候发生异常，但是任务执行结果或者异常原因还没有保存到outcome字段(outcome字段用来保存任务执行结果，如果发生异常，则用来保存异常原因)的时候，状态会从NEW变更到COMPLETING。但是这个状态会时间会比较短，属于中间状态。
- `NORMAL`:任务已经执行完成并且任务执行结果已经保存到outcome字段，状态会从COMPLETING转换到NORMAL。这是一个最终态。
- `EXCEPTIONAL`:任务执行发生异常并且异常原因已经保存到outcome字段中后，状态会从COMPLETING转换到EXCEPTIONAL。这是一个最终态。
- `CANCELLED`:任务还没开始执行或者已经开始执行但是还没有执行完成的时候，用户调用了cancel(false)方法取消任务且不中断任务执行线程，这个时候状态会从NEW转化为CANCELLED状态。这是一个最终态。
- `INTERRUPTING`: 任务还没开始执行或者已经执行但是还没有执行完成的时候，用户调用了cancel(true)方法取消任务并且要中断任务执行线程但是还没有中断任务执行线程之前，状态会从NEW转化为INTERRUPTING。这是一个中间状态。
- `INTERRUPTED`:调用interrupt()中断任务执行线程之后状态会从INTERRUPTING转换到INTERRUPTED。这是一个最终态。 有一点需要注意的是，所有**值大于COMPLETING的状态都表示任务已经执行完成(任务正常执行完成，任务执行异常或者任务被取消)。**

**注意！！！：每个FutureTask被执行过之后，它的state就不再是NEW状态了，其它线程再执行就会失败，FutureTask.run()只能被执行一次。**

## 4.FutureTask的构造函数

- **FutureTask(Callable<V> callable)**

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Callable变量保存在this.callable字段中，该字段定义为`private Callable<V> callable`;用来保存底层的调用，在被执行完成以后会指向null,接着会初始化state字段为NEW。

- **FutureTask(Runnable runnable, V result)**

```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Runnable封装成一个Callable对象保存在callable字段中，同时如果任务执行成功的话就会**返回传入的result**。这种情况下如果**不需要返回值的话可以传入一个null。**

```java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
       throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

在底层对Runnable的封装采用了适配器模式，RunnableAdapter<T>在**call()实现中调用Runnable.run()方法**，然后把传入的r**esult作为任务的结果返回。**

## 5.核心方法

### **run()**

在new了一个FutureTask对象之后，接下来就是在另一个线程中执行这个Task,无论是通过直接new一个Thread还是通过线程池，**执行的都是run()方法，run方法只能执行一次!!!!!!。**

```java
public void run() {
    //新建任务，CAS替换runner为当前线程
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
```

- 运行任务，如果任务状态为NEW状态，则利用CAS修改为当前线程。执行完毕调用set(result)方法设置执行结果。

### get()

```java
//获取执行结果
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

如果任务处于未完成的状态(`state <= COMPLETING`)，就调用awaitDone方法等待任务完成,即这时会被**加入等待队列。**

###  cancel(boolean mayInterruptIfRunning)

说明：尝试取消任务。如果任务已经完成或已经被取消，此操作会失败。

- 如果当前Future状态为NEW，根据参数修改Future状态为INTERRUPTING或CANCELLED。
- 如果当前状态不为NEW，则根据参数mayInterruptIfRunning决定是否在任务运行中也可以中断。中断操作完成后，调用finishCompletion移除并唤醒所有等待线程

## **6.常用使用方式：**

- 第一种方式: Future + ExecutorService
- 第二种方式: FutureTask + ExecutorService
- 第三种方式: FutureTask + Thread

# 十七、线程池之ThreaPoolExecutor详解

## 1、ThreadPoolExecutor中的关键属性

```java
//这个属性是用来存放 当前运行的worker数量以及线程池状态的
//int是32位的，这里把int的高3位拿来充当线程池状态的标志位,后29位拿来充当当前运行worker的数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//存放任务的阻塞队列，维护着等待执行的Runnable任务对象。
private final BlockingQueue<Runnable> workQueue;
//worker的集合,用set来存放
private final HashSet<Worker> workers = new HashSet<Worker>();
//历史达到的worker数最大值
private int largestPoolSize;
//当队列满了并且worker的数量达到maxSize的时候,执行具体的拒绝策略
private volatile RejectedExecutionHandler handler;
//超出coreSize的worker的生存时间
private volatile long keepAliveTime;
//常驻worker的数量
private volatile int corePoolSize;
//最大worker的数量,一般当workQueue满了才会用到这个参数
private volatile int maximumPoolSize;
  

-----------------------------------------
   // Worker时ThreadPoolExecutor中的一个内部类，实现了AQS接口，是线程池中真正执行任务的线程
    private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;
		........
    }
```

- - `ArrayBlockingQueue`: 基于数组结构的**有界阻塞队列**，按FIFO排序任务；
  - **`LinkedBlockingQueue`**: 基于链表结构的**无界阻塞队列**，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQueue；
  - **`SynchronousQueue`**: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，**只有在使用无界线程池或者有饱和策略时才建议使用该队列。**
  - **`PriorityBlockingQueue`**: 具有优先级的无界阻塞队列；
  - **`DelayQueue`**：延迟队列，该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。

`LinkedBlockingQueue`比`ArrayBlockingQueue`在插入删除节点性能方面更优，但是二者在`put()`, `take()`任务的时均需要加锁，`SynchronousQueue`使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是`Transfer.transfer()`.

**~线程池重要参数：**

核心线程会一直存活，即使没有任务需要执行。

- **corePoolSize：核心线程数**

  只要被创建，即使有线程空闲，当线程数小于核心线程数时，线程池也会优先创建新线程处理。

  设置allowCoreThreadTimeout=true（默认false）时，核心线程会超时关闭。

  可以通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法来提前启动线程池中的基本线程。

- **`maximumPoolSize` 线程池中允许的最大线程数：**

  如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.

- **`keepAliveTime`  ** **非核心线程闲置超时时长**：

  非核心线程如果处于闲置状态超过该值，就会被销毁。如果设置**`allowCoreThreadTimeOut(true)`**，则会也作用于核心线程。

- **`unit` keepAliveTime的单位**

- **`threadFactory` 创建线程的工厂：**

  底层用new Thread()为我们创建线程，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory

 - **workQueue：工作队列：**

  存放待执行任务的队列：当提交的任务数超过核心线程数大小后，再提交的任务就存放在工作队列，任务调度时再从队列中取出任务。它仅仅用来存放被execute()方法提交的Runnable任务。工作队列**实现了BlockingQueue接口**。

1. ArrayBlockingQueue 数组型阻塞队列：数组结构，初始化时传入大小，有界，FIFO，使用一个重入锁，默认使用非公平锁，入队和出队共用一个锁，互斥。
2. LinkedBlockingQueue 链表型阻塞队列：链表结构，默认初始化大小为Integer.MAX_VALUE，有界（近似无界），FIFO，使用两个重入锁分别控制元素的入队和出队，用Condition进行线程间的唤醒和等待。
3. SynchronousQueue 同步队列：容量为0，添加任务必须等待取出任务，这个队列相当于通道，不存储元素。
4. PriorityBlockingQueue 优先阻塞队列：无界，默认采用元素自然顺序升序排列。
5. DelayQueue 延时队列：无界，元素有过期时间，过期的元素才能被取出。

- **`handler` 线程池的饱和策略：**

  当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略:

  - `AbortPolicy`: 直接抛出异常，默认策略；
  - `CallerRunsPolicy`: 用调用者所在的线程来执行任务；
  - `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
  - `DiscardPolicy`: 直接丢弃任务；


![image-20221008211042430](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221008211042430.png)

## 2.ThreadPool运行流程

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221014205641526.png" alt="image-20221014205641526" style="zoom: 80%;" />

- 刚开始时，线程池中没有线程，当第一个任务提交后线程池才会创建一个**核心线程**来处理任务
- 之后每提交一次任务，线程池都会创建一个核心线程，直到核心线程被使用完毕
- 核心线程使用完毕之后，多余的任务会进入阻塞队列等待，当一个核心线程处理完一个任务时又从阻塞队列中取下一个任务来执行。
- 随着任务越来越多，核心线程顶不住了，阻塞队列也会被塞满。
- 当阻塞队列被塞满以后，若再有任务进来，线程池就会创建出急救线程来处理任务（救急线程数 = 最大线程数 - 核心线程数）
- 如果线程数量已经达到了最大线程数，这是还有任务进来，就会执行我们规定的拒绝策略

## 3、内部状态

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

![image-20221008210854259](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221008210854259.png)

- 线程池创建后处于**RUNNING**状态。

- 调用shutdown()方法后处于**SHUTDOWN**状态，线程池不能接受新的任务，清除一些空闲worker,不会等待阻塞队列的任务完成。

- 调用shutdownNow()方法后处于**STOP**状态，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃。此时，poolsize=0,阻塞队列的size也为0。

- 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为**TIDYING**状态。接着会执行terminated()函数。

  > ThreadPoolExecutor中有一个控制状态的属性叫`ctl`，它是一个AtomicInteger类型的变量。线程池状态就是通过AtomicInteger类型的成员变量`ctl`来获取的。
  >
  > 获取的`ctl`值传入`runStateOf`方法，与`~CAPACITY`位与运算(`CAPACITY`是低29位全1的int变量)。
  >
  > `~CAPACITY`在这里相当于掩码，用来获取ctl的高3位，表示线程池状态；而另外的低29位用于表示工作线程数

- 线程池处在TIDYING状态时，**执行完terminated()方法之后**，就会由 **TIDYING -> TERMINATED**， 线程池被设置为TERMINATED状态。

```java
/*
线程池状态变为 SHUTDOWN
- 不会接收新任务
- 但已提交任务会执行完
- 此方法不会阻塞调用线程的执行
*/
void shutdown();

/*
线程池状态变为 STOP
- 不会接收新任务
- 会将队列中的任务返回
- 并用 interrupt 的方式中断正在执行的任务
*/
List<Runnable> shutdownNow();
```

## 4、拒绝策略 & 任务执行流程

如果当前同时运行的**线程数量达到最大线程数量并且队列也已经被放满了任务时**，`ThreadPoolTaskExecutor` **定义一些拒绝策略策略:**

- **AbortPolicy：**中止策略。**默认的拒绝策略，直接抛出异常**，拒绝新任务的处理。调用者可以捕获这个异常，然后根据需求编写自己的处理代码。
- **DiscardPolicy：**抛弃策略。**什么都不做，直接抛弃被拒绝的任务。**
- **DiscardOldestPolicy：** **丢弃阻塞队列中靠最前的任务，并执行当前任务抛弃阻塞队列中最老的任务**，相当于就是队列中下一个将要被执行的任务，然后重新提交被拒绝的任务。如果阻塞队列是一个优先队列，那么“抛弃最旧的”策略将导致抛弃优先级最高的任务，因此最好不要将该策略和优先级队列放在一起使用。
- **CallerRunsPolicy：** **用调用者所在的线程来执行任务。**也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。

**执行流程：**

1. 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量快速达到corePoolSize，在核心线程数量 < corePoolSize时）。**注意，这一步需要获得全局锁。**
2. 线程总数量 >= corePoolSize时，新来的线程任务会进入任务队列（workQueue）中等待，然后空闲的核心线程会依次去缓存队列中取任务来执行（体现了**线程复用**）。
3. 当缓存队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建非核心线程去执行这个任务。**注意，这一步需要获得全局锁。**
4. 缓存队列满了， 且总线程数达到了maximumPoolSize，则会采取上面提到的拒绝策略进行处理。

### 如何实现线程复用？

ThreadPoolExecutor在创建线程时，会将线程封装成**工作线程worker**,并放入**工作线程组**中，然后这个worker反复从阻塞队列中拿任务去执行。

## 5、线程池里有个 ctl，你知道它是如何设计的吗？

ctl 是一个打包两个概念字段的原子整数。

1）workerCount：指示线程的有效数量；

2）runState：指示线程池的运行状态，有 RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED 等状态。

int 类型有32位，其中 ctl 的低29为用于表示 workerCount，高3位用于表示 runState，如下图所示。
![image-20221008213036568](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221008213036568.png)

## 6、Executors工具类生成的三种线程池

### I、newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
 return new ThreadPoolExecutor(nThreads, nThreads,
 0L, TimeUnit.MILLISECONDS,
 new LinkedBlockingQueue<Runnable>());
}
//核心线程数等于最大线程数
//阻塞队列是无界的，可以存放任意多个任务
```

**核心线程数量和总线程数量相等，都是传入的参数nThreads**，所以只能创建核心线程，不能创建非核心线程。因为LinkedBlockingQueue的默认大小是Integer.MAX_VALUE，故如果核心线程空闲，则交给核心线程处理；如果核心线程不空闲，则入列等待，直到核心线程空闲。

**与CachedThreadPool的区别**：

- 因为 corePoolSize == maximumPoolSize ，所以FixedThreadPool只会创建核心线程。 而CachedThreadPool因为corePoolSize=0，所以只会创建非核心线程。
- 在 getTask() 方法，如果队列里没有任务可取，线程会一直阻塞在 LinkedBlockingQueue.take() ，线程不会被回收。 CachedThreadPool会在60s后收回。
- 由于线程不会被回收，会一直卡在阻塞，所以**没有任务的情况下， FixedThreadPool占用资源更多**。
- 都几乎不会触发拒绝策略，但是原理不同。FixedThreadPool是因为阻塞队列可以很大（最大为Integer最大值），故几乎不会触发拒绝策略；CachedThreadPool是因为线程池很大（最大为Integer最大值），几乎不会导致线程数量大于最大线程数，故几乎不会触发拒绝策略。

### II、newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
 return new FinalizableDelegatedExecutorService
 (new ThreadPoolExecutor(1, 1,
 0L, TimeUnit.MILLISECONDS,
 new LinkedBlockingQueue<Runnable>()));
}
/**
初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行. 由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效
/
```

### III、newCachedThreadPool

`CachedThreadPool` 的`corePoolSize` 被设置为空（0），`maximumPoolSize`被设置为 `Integer.MAX.VALUE`，即它是无界的，这也就意味着如果主线程提交任务的速度高于 `maximumPool` 中线程处理任务的速度时，`CachedThreadPool` 会不断创建新的线程。极端情况下，**这样会导致耗尽 cpu 和内存资源**。

```java
public static ExecutorService newCachedThreadPool() {
 return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
 60L, TimeUnit.SECONDS,
 new SynchronousQueue<Runnable>());
}
/**

线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列；
有线程来取，任务才能被加入阻塞队列
和newFixedThreadPool创建的线程池不同，newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销；
执行过程与前两种稍微不同:
/
```

### IV.newScheduledThreadPool

创建一个定长线程池，支持定时及周期性任务执行。

线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用DelayedWorkQueue作为阻塞队列。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

## 7、为什么线程池推荐使用Executors去创建? 推荐方式是什么?

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

- newFixedThreadPool和newSingleThreadExecutor:  主要问题是**堆积的请求处理队列可能会耗费非常大的内存**，甚至OOM。
- newCachedThreadPool和newScheduledThreadPool:  主要问题是**线程数最大数是Integer.MAX_VALUE**，可能会创建数量非常多的线程，甚至OOM。

推荐手动new ThreadPoolExecutor来创建线程

## 8、配置线程池要考虑的因素

**corePoolSize核心线程数的配置：**

对于计算密集型，设置 线程数 = CPU数 + 1，通常能实现最优的利用率。

对于I/O密集型，网上常见的说法是设置 线程数 = CPU数 * 2 ，这个做法是可以的，但个人觉得不是最优的。

在我们日常的开发中，我们的任务几乎是离不开I/O的，常见的网络I/O（RPC调用）、磁盘I/O（数据库操作），并且I/O的等待时间通常会占整个任务处理时间的很大一部分，在这种情况下，开启更多的线程可以让 CPU 得到更充分的使用，一个较合理的计算公式如下：

线程数 = CPU数 * CPU利用率 * (任务等待时间 / 任务计算时间 + 1)

例如我们有个定时任务，部署在4核的服务器上，该任务有100ms在计算，900ms在I/O等待，则线程数约为：4 * 1 * (1 + 900 / 100) = 40个。

**`maxPoolSize` 最大线程数在生产环境上我们往往设置成`corePoolSize`一样，这样可以减少在处理过程中创建线程的开销。**

**`rejectedExecutionHandler`：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理。**

**`keepAliveTime`和`allowCoreThreadTimeout`采用默认通常能满足。**

当然，具体我们还要结合实际的使用场景来考虑。如果要求比较精确，可以通过压测来获取一个合理的值。

## 9、监控线程池状态

可以使用ThreadPoolExecutor以下方法:

- `getTaskCount()` Returns the approximate total number of tasks that have ever been scheduled for execution.
- `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于getTaskCount()。
- `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于maximumPoolSize
- `getPoolSize()` Returns the current number of threads in the pool.
- `getActiveCount()` Returns the approximate number of threads that are actively executing tasks.

## 10.执行 execute()方法和 submit()方法的区别是什么呢？

1. **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
2. **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get(long timeout，TimeUnit unit)`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

## 11、线程池的优势是什么(为什么要使用线程池)？

- **降低资源消耗**：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**：当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性**：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

# 十八、线程工具类详解

## 1.CountDownLatch详解

### I.什么是CountDownLatch?

`CountDownLatch` 的作用就是 允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。其底层是由**AQS提供支持，**是以共享锁的方式来获取同步状态（**tryAcquireShared**）。

### II.说一说CountDownLatch的底层实现原理

CountDownLatch底层是由AQS支持的，同ReentrantLock一样，其内部维护者一个Sync类。当创建CountDownLatch对象时要初始化其count值，该count值实际上就是AQS中的state变量。

CountDownLatch中有两个重要方法：

- countDown()：**在共享模式下释放资源，每一次调用countDown，state变量值减1。**

调用countDown()实际上是有一条**调用链**，

<img src="https://pdai.tech/_images/thread/java-thread-x-countdownlatch-2.png" alt="img" style="zoom: 80%;" />

```java
private void doReleaseShared() {
        
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!h.compareAndSetWaitStatus(Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !h.compareAndSetWaitStatus(0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

- await()：此函数将会使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断，**当state变量变为0时继续往下执行。**

```java
public void await() throws InterruptedException {
    // 转发到sync对象上
    sync.acquireSharedInterruptibly(1);
}
    
```

说明: 由源码可知，对CountDownLatch对象的await的调用会转发为对Sync的acquireSharedInterruptibly(从AQS继承的方法)方法的调用。

- acquireSharedInterruptibly源码如下:

```java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

说明: 从源码中可知，acquireSharedInterruptibly又调用了CountDownLatch的内部类Sync的tryAcquireShared和AQS的doAcquireSharedInterruptibly函数。

- tryAcquireShared函数的源码如下:

```java
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

说明: 该函数只是简单的判断AQS的state是否为0，为0则返回1，不为0则返回-1。

### III.说出使用CountDownLatch 代替wait notify 好处?

使用CountDownLatch 代替wait notify 好处是**通讯方式简单，不涉及锁定**  Count 值为0时当前线程继续执行

## 2.CyclicBarrier简介

### I.CyclicBarrier介绍

CyclicBarrirer从名字上来理解是“循环的屏障”的意思。前面提到了CountDownLatch一旦计数值`count`被降为0后，就不能再重新设置了，它只能起一次“屏障”的作用。而CyclicBarrier拥有CountDownLatch的所有功能，还可以使用`reset()`方法重置屏障。

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行await()方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier的计数器通过**调用 reset() 方法可以循环使用**，所以它才叫做循环屏障。

CyclicBarrier有两个构造函数，**其中parties指示计数器的初始值，barrierAction(一个线程)在所有线程都到达屏障的时候会执行一次，可以通过这个线程来汇总其它线程的运行结果。**



```java
public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
public CyclicBarrier(int parties) {
        this(parties, null);
    }
```

**当调用await()方法时，锁计数-1，当计数减为0时，执行CyclicBarrier构造函数里的方法，之后等待的线程继续往下执行，并且计数又重写变为初始值。同时可以用reset()方法重置锁计数器。**

```java
package com.csg.blockingqueue;

import java.util.concurrent.*;

public class BlockingQueueTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2, () -> {
            System.out.println("task1, task2 end...");
        });
        for (int i = 0; i < 3; i++) {
            executorService.execute(() -> {
                System.out.println("task1 begin...");
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }

            });
            executorService.execute(() -> {
                System.out.println("task2 begin...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                });
        }
        executorService.shutdown();
    }
}
task1 begin...
task2 begin...
task1, task2 end...
task1 begin...
task2 begin...
task1, task2 end...
task1 begin...
task2 begin...
task1, task2 end...
```

**注意：线程数最好要和CyclicBarrier设置的初始值一样，这样才可保证执行过程的逻辑顺序。**

### II.CyclicBarrier原理

CyclicBarrier虽说功能与CountDownLatch类似，但是实现原理却完全不同，CyclicBarrier内部使用的是Lock + Condition实现的等待/通知模式。详情可以查看这个方法的源码：

```java
private int dowait(boolean timed, long nanos)
```

## 3.Semaphore简介

Semaphore翻译过来是信号的意思。顾名思义，这个工具类提供的功能就是多个线程彼此“打信号”。而这个“信号”是一个`int`类型的数据，也可以看成是一种“资源”。

可以在构造函数中**传入初始资源总数**，以及是否使用“公平”的同步器。默认情况下，**是非公平的。**

```java
// 默认情况下使用非公平
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

最主要的方法是acquire方法和release方法。**acquire()方法会申请一个permit，而release方法会释放一个permit。**当然，你也可以申请多个acquire(int permits)或者释放多个release(int permits)。

每次acquire，permits就会减少一个或者多个。**如果减少到了0，再有其他线程来acquire，那就要阻塞这个线程直到有其它线程release permit为止。**

每个线程可以获取多个许可，也可以释放多个许可，许可与线程是没有绑定在一起的，假设线程1获取了一个许可，线程2可以去释放。

```java
package com.csg.blockingqueue;

import java.util.concurrent.*;

public class BlockingQueueTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Semaphore semaphore = new Semaphore(5);
        for (int i = 0; i < 5; i++) {
            semaphore.release();//释放资源
        }
        Thread.sleep(1000);
        executorService.execute(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    semaphore.acquire();//获取资源
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("get semaphore");
        });
        executorService.shutdown();
    }
}
//输出：get semaphore
```

### Semaphore原理

Semaphore内部有一个继承了AQS的同步器Sync，重写了`tryAcquireShared`方法。在这个方法里，会去尝试获取资源。

如果获取失败（想要的资源数量小于目前已有的资源数量），就会返回一个负数（代表尝试获取资源失败）。然后当前线程就会进入AQS的等待队列。

# 十九、Coding

## 1.多线程交替打印

```java
public class LoopPrintTest {
    static volatile int i = 0;
    static volatile boolean flag = true;
    public static void main(String[] args) {
        ReentrantLock lock= new ReentrantLock();
        Condition condition = lock.newCondition();
        Thread t1 = new Thread(() -> {
            while(true) {
                lock.lock();
                try {
                    if (flag) { //true不能打印
                        condition.await();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() +" : "+ i++);
                flag = true;//打印完要改为true,否则t2为false,t2是false时不能打印
                condition.signalAll();
                lock.unlock();
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            while (true) {
                lock.lock();
                try {
                    if (!flag) {//false不能打印
                        condition.await();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() +" : "+ i++);
                flag = false;//打印完要改为false,否则t2为true,t2是true时不能打印
                condition.signalAll();
                lock.unlock();
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

