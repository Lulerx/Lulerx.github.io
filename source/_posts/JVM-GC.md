---
title: JVM垃圾回收机制
date: 2020-08-26
categories: [JVM,JVM垃圾回收]    #分类
tags: [JVM垃圾回收,GC]          #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

## 1. 对象标记

> 程序计数器、虚拟机栈、本地方法栈 3 个区域随线程生灭(因为是线程私有)，栈中的栈帧随着方法的进入和退出而有条不紊地执行着出栈和入栈操作。而 Java 堆和方法区则不一样，一个接口中的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存也可能不一样，我们只有在程序处于运行期才知道那些对象会创建，这部分内存的分配和回收都是动态的，垃圾回收期所关注的就是这部分内存。

在进行垃圾回收之前，首先要确定的是哪些对象还“活着”，哪些对象已经“死去”。判断对象是否存活主要有以下2种算法：

### 1.1 引用计数算法

​	给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用的。

> 主流的Java虚拟机里面没有选用引用计数算法来管理内存，其中最主要的原因是**它很难解决对象之间相互循环引用的问题**。

> 举个例子：
>
> 对象objA和objB都有字段instance，赋值令objA.instance=objB及objB.instance=objA，除此之外，这两个对象再无任何引用，实际上这两个对象已经不可能再被访问，但是它们因为互相引用着对方，导致它们的引用计数都不为0，于是引用计数算法无法通知GC收集器回收它们。

### 1.2 可达性分析算法

​	这个算法的基本思路就是通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的。

![](/images/java/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvNThiZmFjMTVjYTZkMzA3NmRlZjUxNzRlZDVjYTVhOTk_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ.jpg)

> 在Java语言中，可作为GC Roots的对象包括下面几种：
>
> - 虚拟机栈（栈帧中的本地变量表）中引用的对象。
> - 方法区中类静态属性引用的对象。
> - 方法区中常量引用的对象。
> - 本地方法栈中JNI（即一般说的Native方法）引用的对象。

### 1.3 四种引用

​	在JDK1.2之后，Java对引用的概念进行了扩充，将引用分为强引用、软引用、弱引用、虚引用4种，这4种引用强度依次逐渐减弱。

​	**强引用：**就是指在程序代码之中普遍存在的，类似“Object obj=new Object（）”这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。

​	**软引用：**是用来描述一些还有用但并非必需的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK 1.2之后，提供了SoftReference类来实现软引用。

​	**弱引用：**也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在JDK 1.2之后，提供了WeakReference类来实现弱引用。

​	**虚引用：**也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。在JDK 1.2之后，提供了PhantomReference类来实现虚引用。

### 1.4 生存还是死亡

> 即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，这时候它们暂时出于“缓刑”阶段，一个对象的真正死亡至少要经历两次标记过程：如果对象在进行中可达性分析后发现没有与 GC Roots 相连接的引用链，那他将会被第一次标记并且进行一次筛选，筛选条件是此对象是否有必要执行 finalize() 方法。当对象没有覆盖 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。
>
> 如果这个对象被判定为有必要执行 finalize() 方法，那么这个对象竟会放置在一个叫做 F-Queue 的队列中，并在稍后由一个由虚拟机自动建立的、低优先级的 Finalizer 线程去执行它。这里所谓的“执行”是指虚拟机会出发这个方法，并不承诺或等待他运行结束。finalize() 方法是对象逃脱死亡命运的最后一次机会，稍后 GC 将对 F-Queue 中的对象进行第二次小规模的标记，如果对象要在 finalize() 中成功拯救自己 —— 只要重新与引用链上的任何一个对象简历关联即可。
>
> 值得注意的是，finalize() 方法只会被系统自动调用一次。如果对象面临下一次回收，它的finalize() 方法不会被再次执行。

### 1.5 回收方法区

在方法区进行垃圾收集的“性价比”一般比较低：在堆中，尤其是在新生代中，一次垃圾回收一般可以回收 70% ~ 95% 的空间，而永久代的垃圾收集效率远低于此。

永久代垃圾回收主要两部分内容：**废弃的常量和无用的类**。

> 判断废弃常量：一般是判断没有该常量的引用。
>
> 判断无用的类，要同时满足以下三个条件：
>
> - 该类所有的实例都已经回收，也就是 Java 堆中不存在该类的任何实例
> - 加载该类的 ClassLoader 已经被回收
> - 该类对应的 java.lang.Class 对象没有任何地方呗引用，无法在任何地方通过反射访问该类的方法

---



## 2. 垃圾收集算法

### 2.1 标记--清除算法

​	如同它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

两个不足：

- 标记和清除两个过程的效率都不高
- 会产生大量不连续的内存碎片

### 2.2 复制算法

>  将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

> 只是这种算法的代价是将内存缩小为了原来的一半，未免太高了一点

​	不过现在的商业虚拟机都采用这种收集算法来回收新生代，因为新生代中的对象98%是熬不过第一次GC的，所以并不需要按照1:1的比例来划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一次性地复制到另外一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。

​	HotSpot虚拟机默认Eden和Survivor的大小比例一般是 8 : 1 : 1，每次浪费 10% 的 Survivor 空间。但是这里有一个问题就是如果存活的大于 10% 怎么办？这里采用一种分配担保策略：多出来的对象直接进入老年代。

### 2.3 标记--整理算法

​	是根据老年代的特点，提出的一种算法。进行标记之后，让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

> 算是对标记--清除算法的一种改进。

### 2.4分代收集算法

​	当前商业虚拟机的垃圾收集都采用“分代收集”算法，这种算法并没有什么新的思想，只是根据对象存活周期的不同将内存划分为几块。一般是分为**新生代**和**老年代**，然后根据各个年代的特点制定相应的回收算法。

新生代：

> 每次垃圾回收都有大量对象死去，只有少量存活，选用`复制算法`比较合理。

老年代：

> 老年代中对象存活率较高、没有额外的空间分配对它进行担保。所以必须使用 `标记--清除` 或者 `标记--整理` 算法回收。

---

## 3. 垃圾收集器

​	垃圾收集算法是内存回收的方法论，而垃圾收集器是内存回收的具体实现。

HotSpot虚拟机包含的所有收集器如下图：

![11](/images/java/1598431191.jpg)

> 说明：如果两个收集器之间存在连线说明他们之间可以搭配使用。

### 3.1 Serial 收集器

​	这是一个单线程收集器。意味着它只会使用一个 CPU 或一条收集线程去完成收集工作，并且在进行垃圾回收时必须暂停其它所有的工作线程直到收集结束。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvYjE4NDk0YjFlNTQ4NTFiYmJkMmVlNTI3NjBjYzM3NTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

> 值得注意的是，实际上到现在为止，它依然是虚拟机运行在Client模式下的默认新生代收集器。
>
> 它有着优于其他收集器的地方：简单而高效（与其他收集器的单线程比）

> 在用户的桌面应用场景中，分配给虚拟机管理的内存一般来说不会很大，收集几十兆甚至一两百兆的新生代（仅仅是新生代使用的内存，桌面应用基本上不会再大了），停顿时间完全可以控制在几十毫秒最多一百多毫秒以内，只要不是频繁发生，这点停顿是可以接受的。所以，Serial收集器对于运行在Client模式下的虚拟机来说是一个很好的选择。

### 3.2 ParNew 收集器

**ParNew收集器其实就是Serial收集器的多线程版本**

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMTU0NjVmYjJlMTdjYjVkNjY1YzI1YmI5OGFjZmVhOTM_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

> 它是许多运行在Server模式下的虚拟机中首选的新生代收集器。
>
> 其中有一个与性能无关但很重要的原因是，除了Serial收集器外，目前只有它能与CMS收集器配合工作

### 3.3 Parallel Scavenge 收集器

​	这是一个新生代收集器，也是使用复制算法实现，同时也是并行的多线程收集器。

​	Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS 等收集器的关注点是尽可能地缩短垃圾收集时用户线程所停顿的时间，而 Parallel Scavenge 收集器的目的是达到一个可控制的吞吐量。

~~~
吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）
~~~

​	作为一个吞吐量优先的收集器，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。这种调节方式称为GC自适应的调节策略(GC Ergonomics)。

### 3.4 Serial Old 收集器

> 是Serial收集器的老年代版本，单线程收集器，使用 `标记--整理` 算法

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvYjE4NDk0YjFlNTQ4NTFiYmJkMmVlNTI3NjBjYzM3NTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

​	这个收集器的主要意义也是在于给Client模式下的虚拟机使用。如果在Server模式下，那么它主要还有两大用途：一种用途是在JDK 1.5以及之前的版本中与Parallel Scavenge收集器搭配使用；另一种用途就是作为CMS收集器的后备预案，在并发收集发生ConcurrentMode Failure时使用。

### 3.5 Parallel Old 收集器

> 是 Parallel Scavenge 收集器的老年代版本，使用多线程和 `标记--整理` 算法。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMjU2NDEzNjZiNDNkOTcxMzEwYTBhN2NlZGU0ZTQwNmE_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

​	这个收集器是在JDK 1.6中才开始提供的，在此之前，新生代的Parallel Scavenge收集器一直处于比较尴尬的状态。原因是，如果新生代选择了Parallel Scavenge收集器，老年代除了Serial Old（PS MarkSweep）收集器外别无选择。

​	直到Parallel Old收集器出现后，“吞吐量优先”收集器终于有了比较名副其实的应用组合，在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

### 3.6 CMS 收集器

> CMS (Concurrent Mark Sweep  并发标记清除 ) 收集器是一种以获取最短回收停顿时间为目标的收集器。基于 `标记--清除` 算法实现。

运作步骤:

1. 初始标记(CMS initial mark)：标记 GC Roots 能直接关联到的对象
2. 并发标记(CMS concurrent mark)：进行 GC Roots Tracing
3. 重新标记(CMS remark)：修正并发标记期间的变动部分
4. 并发清除(CMS concurrent sweep)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvNmY0ZDY4MzY0NGExNTQ1MzdiM2UyM2Q2MGQ0OWMwNzQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

优点：并发收集、低停顿

缺点：对 CPU 资源敏感、无法收集浮动垃圾、 `标记 —— 清除` 算法带来的空间碎片

### 3.7 G1 收集器

​	G1（Garbage-First）收集器是当今收集器技术发展的最前沿成果之一，早在JDK 1.7刚刚确立项目目标，Sun公司给出的JDK 1.7 RoadMap里面，它就被视为JDK 1.7中HotSpot虚拟机的一个重要进化特征。

​	**G1是一款面向服务端应用的垃圾收集器。**

相比其他GC收集器，有如下特点：

+ **并行与并发：**G1能充分利用多CPU、多核环境下的硬件优势，缩短Stop-The-World停顿的时间
+ **分代收集：**分代概念在G1中依然得以保留
+ **空间整合：**G1运作期间不会产生内存空间碎片
+ **可预测的停顿：**这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关
  注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一
  个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实
  时Java（RTSJ）的垃圾收集器的特征了

运作步骤:

1. **初始标记(Initial Marking)：**标记GC Roots能直接关联到的对象并且修改TAMS（Next Top at Mark Start）的值。**需停顿线程，但耗时很短。**
2. **并发标记(Concurrent Marking)：**进行可达性分析，找出存活的对象。耗时较长，但可与用户程序并发执行。
3. **最终标记(Final Marking)：**修正并发标记期间的变动部分
4. 筛选回收(Live Data Counting and Evacuation)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvNDBhNTc1OTMxYjI1NGE4ZjQwYmI1NDNjMjRlOGZhZGY_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

## 4. 内存分配与回收策略

### 4.1 对象优先在 Eden 分配

> 对象主要分配在新生代的 Eden 区上，如果启动了本地线程分配缓冲区，将线程优先在 (TLAB) 上分配。少数情况会直接分配在老年代中。

一般来说 Java 堆的内存模型如下图所示：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvOTcxMDE4MDMxNWQzNTc1NmI2OGU5YzVkYWY0NGQ2ZTU_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

> + 新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
> + 老年代GC（Major GC/Full GC）：指发生在老年代的GC，出现了Full GC，经常会伴随至少一次的Minor GC（但非绝对的）。Full GC的速度一般会比Minor GC慢10倍以上。

### 4.2 大对象直接进入老年代

​	所谓的大对象是指，需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串以及数组。

> 大对象对虚拟机的内存分配来说就是一个坏消息（比遇到一个大对象更加坏的消息就是遇到一群“朝生夕灭”的“短命大对象”，写程序的时候应当避免），经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来“安置”它们。

### 4.3 长期存活的对象将进入老年代

> 既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这点，虚拟机给每个对象定义了一个对象年龄（Age）计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中每“熬过”一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就将会被晋升到老年代中。
>
> 对象晋升老年代的年龄阈值，可以通过参数-XX：MaxTenuringThreshold设置。

### 4.4 动态对象年龄判定

​	为了能更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了年龄阈值才能晋升老年代，**如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到年龄阈值中要求的年龄**。

> 就比如说，假如Survivor空间大小为1MB，年龄同为5的A和B加起来的大小已经超过512KB，A对象和B对象以及年龄大于5的对象，直接晋升到老年代。

### 4.5 空间分配担保

​	在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。