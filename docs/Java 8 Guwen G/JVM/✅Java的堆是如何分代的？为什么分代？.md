# 典型回答


Java的堆内存分代是指将不同生命周期的堆内存对象存储在不同的堆内存区域中，这里的不同的堆内存区域被定义为“代”。这样做有助于提升垃圾回收的效率，因为这样的话就可以为不同的"代"设置不同的回收策略。



一般来说，**Java中的大部分对象都是朝生夕死的**，同时也有一部分对象会持久存在。因为如果把这两部分对象放到一起分析和回收，这样效率实在是太低了。**通过将不同时期的对象存储在不同的内存池中，就可以节省宝贵的时间和空间，从而改善系统的性能。**



**Java的堆由新生代（Young Generation）和老年代（Old Generation）组成。**新生代存放新分配的对象，老年代存放长期存在的对象。



新生代（Young）由年轻区（Eden）、Survivor区组成（<font style="color:rgb(85, 85, 85);">From Survivor、To Survivor</font>）。默认情况下，新生代的Eden区和Survivor区的空间大小比例是8:2，可以通过-XX:SurvivorRatio参数调整。



![](https://cdn.nlark.com/yuque/0/2022/png/5378072/1671680879420-22b483c1-b11c-483c-908b-5a8ef7ad7959.png)



很多对象都会出现在Eden区，当Eden区的内存容量用完的时候，GC会发起，非存活对象会被标记为死亡，存活的对象被移动到Survivor区。



如果Survivor的内存容量也用完，那么存活对象会被移动到老年代。



老年代（Old）是对象存活时间最长的部分，它由单一存活区（Tenured）组成，并且把经历过若干轮GC回收还存活下来的对象移动而来。在老年代中，大部分对象都是活了很久的，所以GC回收它们会很慢。



# 扩展知识
## 对象的分代晋升


一般情况下，对象将在新生代进行分配，首先会尝试在Eden区分配对象，当Eden内存耗尽，无法满足新的对象分配请求时，将触发新生代的GC(Young GC、MinorGC)，在新生代的GC过程中，没有被回收的对象会从Eden区被搬运到Survivor区，这个过程通常被称为"晋升"



同样的，**对象也可能会晋升到老年代，触发条件主要看对象的大小和年龄**。对象进入老年代的条件有三个，满足一个就会进入到老年代：

+ 1、躲过15次GC。每次垃圾回收后，存活的对象的年龄就会加1，累计加到15次（jdk8默认的），也就是某个对象躲过了15次垃圾回收，那么JVM就认为这个是经常被使用的对象，就没必要再待在年轻代中了。具体的次数可以通过 -XX:MaxTenuringThreshold 来设置在躲过多少次垃圾收集后进去老年代。
+ 2、动态对象年龄判断。规则：**如果在Survivor空间中小于等于某个年龄的所有对象大小的总和大于Survivor空间的一半时，那么就把大于等于这个年龄的对象都晋升到老年代。**
+ 3、大对象直接进入老年代。-XX:PretenureSizeThreshold 来设置大对象的临界值，大于该值的就被认为是大对象，就会直接进入老年代。（PretenureSizeThreshold默认是0，也就是说，默认情况下对象不会提前进入老年代，而是直接在新生代分配。然后就GC次数和基于动态年龄判断来进入老年代。）

针对上面的三点来逐一分析。



### 动态年龄判断
				

为了能更好地适应不同程序的内存状况，HotSpot虚拟机并不是永远要求对象的年龄必须达到- XX:M axTenuringThreshold才能晋升老年代，他还有一个动态年龄判断的机制。



在《深入理解Java虚拟机（第三版）》中是这么描述动态年龄判断的过程的：



![](https://cdn.nlark.com/yuque/0/2023/png/5378072/1697110000643-5b845e48-a453-4f6f-ae57-f3249777fd8f.png)

 					

**<font style="color:#DF2A3F;">但是，这段描述是不正确的</font>**<font style="color:#DF2A3F;">！</font>



JVM中，动态年龄判断的代码如下：



```c
uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
  
  size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
  size_t total = 0;
  uint age = 1;
  while (age < table_size) {
    total += sizes[age];
    if (total > desired_survivor_size) break;
    age++;
  }
  uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
    ...
}
```

 			

它的过程是从年龄小的对象开始，不断地累加对象的大小，当年龄达到N时，刚好达到<font style="color:rgb(31, 35, 40);">TargetSurvivorRatio这个阈值，那么就把所有年龄大于等于N的对象全部晋升到老年代去！</font>

<font style="color:rgb(31, 35, 40);"></font>

<font style="color:rgb(31, 35, 40);">所以，这过程应该是这样的：</font>

<font style="color:rgb(31, 35, 40);"></font>

**如果在Survivor空间中小于等于某个年龄的所有对象大小的总和大于Survivor空间的一半时，那么就把大于等于这个年龄的对象都晋升到老年代。**



## 新生代如果只有两个区域可以吗？


[新生代如果只有两个区域可以吗？](https://www.yuque.com/hollis666/qyhor6/eigm8iqgpwmd2eg8)



## 什么是永久代？


永久代（Permanent Generation）是HotSpot虚拟机在以前版本中使用的一个永久内存区域，是JVM中垃圾收集堆之外的另一个内存区域，它主要用来实现方法区的，其中存储了Class类信息、常量池以及静态变量等数据。



Java 8以后，永久代被重构为元空间（MetaSpace）。



但是，和新生代、老年代一样，永久代也是可能会发生GC的。而且，永久代也是有可能导致内存溢出。只要永久代的内存分配超过限制指定的最大值，就会出现内存溢出。



## 不同分代的GC方式？


Minor GC（YoungGC），主要用于对新生代垃圾回收。

Full GC，除了会对新生代垃圾回收以外，还会对老年代或者永久代进行回收。相比较于Minor GC，Full GC的收集频率更低，耗时更长。



[新生代和老年代的垃圾回收器有何区别？](https://www.yuque.com/hollis666/qyhor6/nqra2l)



