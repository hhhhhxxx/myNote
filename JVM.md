# Java虚拟机



## 自动内存管理机制

### 概述

对于Java程序员来说，在虚拟机自动内存管理机制的帮助下，不再需要为每一个new操作去写配对的delete/free代码，不容易出现内存泄露和内存溢出问题。

### 运行时数据区域

**程序计数器**

程序计数器是一块比较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。（计算机组成原理中的程序计数器是存放下一条指令的地址）。在多线程中，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为线程私有的内存。

程序计数器是唯一一个java虚拟器规范中没有规定任何OutOfMemoryError情况的区域。

**Java虚拟机栈**

与程序计数器一样，Java虚拟机栈也是线程私有的。每个方法在执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

如果线程请求的栈的深度大于虚拟机所运行的深度，将抛出StackOverFlowError异常；如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

**本地方法栈**

与java虚拟机栈相似，区别：本地方法栈为虚拟机使用到的Native方法服务。

**Java堆**

Java堆是Java虚拟机所管理的内存中最大的一块。此内存区域的唯一目的就是存放对啊实例，几乎所有的对象实例都要在这里分配内存。线程共享的Java堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。

如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

**方法区**

方法区与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的内信息、常量、静态变量、即时编译器编译后的代码等数据。虽然java虚拟机规范把方法区描述为堆的一个逻辑部分，但是他却有一个别名叫做非堆，目的是与java堆区分开。

**运行时常量池**

是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

**直接内存**

并不是虚拟机运行的一部分，也不是java虚拟机规范中的定义的区域。但这部分内存也被频繁使用，而且也可能导致OutOfMemoryError异常出现。在JDK1.4中新加入了NIO（New Input/Output）类，引入一种基于通道（Channel）与缓冲区（BUffer）的IO方式，它可以使用Native函数直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

## 垃圾收集器与内存分配策略

### 引用计数算法

给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的。

缺点：它很难解决对象之间相互循环引用的问题

### 可达性分析算法

基本思想就是通过一系列的称为"GC Roots"的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是从GC Roots 到这个对象不可达）时，则证明此对象是不可用的。

### 再谈引用

在JDK1.2之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）4种，这4种引用强度依次逐渐减弱。

**强引用**就是指在程序代码之中普遍存在的，类似“Object = new Object()”，这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉引用的对象。

**软引用**就是用来描述一些还有用但并非必需的对象。对于软引用关联的对象、在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。JDK提供了SoftReference类来实现软引用

**弱引用**也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集器发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。JDK提供了WeakReference类来实现弱引用。

**虚引用**也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生成时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。提供了PhantomReference类来实现虚引用。

### **生存还是死亡** （对象死亡机制）

即使在可达性分析算法中不可达的对象，也并非是非死不可的，这时候它们暂时处于缓刑阶段，要真正宣告一个对象的死亡，至少要经历两次标记过程；如果对象在进行可达性分析后发现没有与GC Roots相连的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 **finalize()**  方法。当对象没有覆盖finalize()方法，或者finalize()方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行” （不被筛选到）



没有覆写finalize方法则该对象必然成为“死亡”状态。



如果这个对象被判断为有必要执行（被筛选到），会被放在一个F-Queue的队列中执行这个方法，但不会承诺他运行结束（可能死循环之类）。finalize（）方法是对象逃脱死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象重新建立引用链，它将被移除“即将回收的集合”（不被回收），如果对象这时候还没有逃脱，那基本它就真的被回收了。

### 垃圾回收算法

只是介绍几种算法的思想，实现涉及大量细节

#### 标记-清除算法

首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

缺点：

​	1.标记和清除两个过程效率都不高

​	2.标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致在程序运行过程中需要分配较大对象时，无法找到足够的连续空间而不得不提前进行触发另一次垃圾收集动作。

#### 复制算法

它将内存按容量划分为大小相等的两块，每次只使用其中的一块。当一块内存用完了，就将还存活者的对象复制到另外一块上面，然后将已使用过的内存一次清理(原来这块）

现在都采用这种收集算法来回收新生代。



将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中Survivor，当回收时，将Eden空间和Survivor中还存活的对象一次性复制到另外一块Survivor空间上，最后清理Eden和刚才用过的Survivor空间。

当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保，如果另外一块Surivor空间没有足够空间存放上一次新生代收集下来的存活对象

#### 标记-整理算法

老年代存活率高，复制算法在存活率高时需要进行较多的复制，还需要分配担保

根据老年代的特点，提出标记-整理算法

和标记清除一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存。

#### 分代收集算法

当前都采用分代收集算法，但是没有新的思想，只是根据对象存活周期的不同，将内存划分为几块，一般是将java堆划分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法，在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用**复制算法**，只需要付出少量存活对象的复制成本就可以完成收集（需要幸存区），而老年代中因为对象存活率高，没有额外的空间对他进行分配担保，就必须使用**标记清除**或者标**记整理算法**来进行回收



### **HotSpot的算法实现**

hotSpot Vm是Sun JDK和OpenJDk所带的虚拟机，也是目前使用范围最广的Java虚拟机。

#### 枚举根节点

可达性分析必须在一个能确保一致性的快照中进行，整个分析期间整个执行系统看起来就像被冻结在某个时间节点，导致GC进行时必须停顿所有的java执行线程（Sun将这件事情成为**Stop The World**）



准确式GC：当执行系统停顿下来后，并不需要一个不漏地检查完所有执行上下文和全局变量的引用位置。

虚拟机应当是有办法直接得知哪些地方存放着对象引用。



HotSpot：使用一组OopMap 的数据结构来达到这个目的(Ordinary Object Pointer普通对象指针) ，在类加载完成的时候，HotSpot就把对象内什么偏移量上是什么数据计算出来，在编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。（就是一个记录引用的数据结构 ）

#### 安全点

如果每条指令都生成对应的OopMap，成本变得很高。

HotSpot没有为每条指令都生成OopMap，只是在特定的位置，记录了这些信息，这些位置成为安全点。

程序执行时并非在所有地方能都停顿下来开始GC，只有在到达安全点时才能暂停。

安全点的选定基本上是已程序“是否具有让程序长时间执行的特征”，例如方法调用、循环跳转、异常跳转等，具有这些功能才会产生SafePoint。

如果让所有线程都跑到最近的安全点上再停顿下来

**抢先式中断：**GC时把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它跑到安全点上。

**主动式中断：**设置一个中断标志，线程执行时，主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的

#### 安全区域

线程处于Sleep或者Blocked，无法响应JVM的中断请求。

安全区域就是指一段代码中，引用关系不会发生变化。在这个区域中的任何地方开始GC都是安全的，我们也可以把Safe Region看做是被扩展了的Safe point。

(开一条线程去标识哪些线程处于安全区域)

线程在安全区域就可以直接开始GC，线程离开安全区域，就检查线程是否完成了根节点枚举（或者整个GC过程，对象有引用就不用GC了），如果完成就继续运行，否则它就等待直到收到可以离开安全区域的信号为止（等到GC结束）

### 垃圾收集器

**Serial收集器：**单线程收集器，进行垃圾收集是，必须暂停其他所有的工作线程，直到它收集结束

**ParNew收集器：**其实是Serial收集器的多线程版本，并没太大创新，但却是许多Server模式下的虚拟机中首选的新生代收集器。

**Parallel Scavenge收集器：**新生代收集器，使用复制算法，关注于尽可能缩短垃圾收集时用户线程的暂停时间，达到一个可控制的吞吐量。

**Serial Old收集器：**serial收集器的老年代版本，同样是单线程，使用标记整理算法，这个收集器在于给Client模式下虚拟机使用。

**Parallel Old收集器：**Parallel Scavenge收集器的老年代版本，使用标记整理算法。

**CMS收集器：**关注于获取最短回收暂停为目标的收集器，采用标记清除算法实现。

4个过程 （初始标记和重新标记这两个步骤任然需要Stop the World）

**1.初始标记**

仅仅只是标记一下GC Roots能直接关联到的对象，速度很快

**2.并发标记**

进行GC Roots Tracing 的过程

**3.重新标记**

修正并发标记因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，停顿时间一般会比初始标记阶段稍长一些

**4.并发清除**





G1收集器：

### 内存分配与回收策略



## 虚拟机类加载机制



## 虚拟机执行子系统