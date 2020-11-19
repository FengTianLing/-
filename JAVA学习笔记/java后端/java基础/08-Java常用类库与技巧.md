# Java异常

异常处理机制能清晰回答了主要的三个问题

What：异常类型 回答了什么被抛出

where：异常堆栈跟踪 回答了在哪里抛出

why：异常信息回答了为什么被爆出



**Error和Exception区别**

![image-20200906221404090](md-images/08-Java常用类库与技巧/image-20200906221404090.png)

Error：程序无法处理的系统级别错误，编译器不做检查。一般是指与jvm相关，比如系统奔溃，虚拟机错误，内存空间不足。

Exception：程序可以处理的异常，捕获后可能恢复。

总结：前者是程序无法处理的错误，后者是可以处理的异常。



**RuntimeException**：不可预知的，程序应当自行避免，比如空指针异常，下标数组越界

**非RuntimeException**：可预知的，从编译器校验的异常。



从责任角度看：

1.Error属于JVM需要负担的责任

2.RuntimeException是程序应该负责的责任

3.Checked Exception可检查异常是JAVA编译器应负担的责任



**常见的Error以及Exception**

![image-20200906223221751](md-images/08-Java常用类库与技巧/image-20200906223221751.png)

# 异常处理机制

抛出异常：创建异常对象，交由运行时系统处理

捕获异常：寻找合适的异常处理器处理异常，否则终止运行



![image-20200906231828170](md-images/08-Java常用类库与技巧/image-20200906231828170.png)



**高效主流的异常处理框架**

在用户看来，应用系统发生的所有异常都是应用系统内部的异常

+ 设计一个通用的继承自RuntimeException的异常来同一处理
+ 其余异常都同一转义为上述异常AppException
+ 在catch之后，抛出上述异常的子类，并提供足以定位的信息



# Java集合

工作中小时而面试却长存的算法和数据结构。优秀的算法和数据结构都被封装到了Java的集合框架之中了，

**数据结构的考点：**

+ 数组和链表的区别
+ 链表的操作，如反转，链表环路检测，双向链表，循环链表相关操作
+ 队列，栈的作用
+ 二叉树的遍历方式及递归的和非递归的实现
+ 红黑树的旋转



**算法的考点**

+ 内部排序:递归排序，交换排序（冒泡，快排），选择排序，插入排序
+ 外部排序：应掌握如何利用有限的内存配合海量的外部存储处理超大的数据集，写不出来也要由相关的思路

**扩展考点**

+ 哪些排序是不稳定的，稳定意味着什么
+ 不同数据集，各种排序最好或最差的情况
+ 如何优化算法

![image-20200907202534182](md-images/08-Java常用类库与技巧/image-20200907202534182.png)

![image-20200907202804379](md-images/08-Java常用类库与技巧/image-20200907202804379.png)

 

**HashMap，HashTable，ConccurentHashMap区别**

**HashMap**:散列通，存储的内容是键值对映射。(JAVA8以前)采用的是数组+链表的数据结构。HashMap的数组长度，默认初始长度是16，这个16长度数组中，每个元素存储的是链表的头节点。通过Hash函数取模，获得要添加的元素索要存储数组的位置。这里会有一种比较极端的情况，如果添加到Hash表里的值的键位，经过哈希散列运算，总是得出相同的值，即分配到同一个桶中，这样会使得某个桶的链表变的很长。由由于链表查询需要从头部开始逐个遍历，哈希Map的性能将会恶化。从O(1)变成O(n)

JAVA8以后，将会使用一个常量，TREEIFY_THRESHOLD 来控制是否将链表转换成红黑树。采用的数组+链表+红黑树的数据结构来存储。

JAVA8以前，数组里面的元素是保存的Entry,JAVA8以后变成Node



![image-20200907211044254](md-images/08-Java常用类库与技巧/image-20200907211044254.png)

put方法的逻辑

+ 若HashMap未被初始化，则进行初始化操作
+ 对Key求Hash值，依据Hash值计算下标
+ 若未发生碰撞，则直接存放入桶中，
+ 若发生碰撞，则以链表的方式链接到后面
+ 若链表长度超过阙值，且HashMap元素超过最低树化容量，则将链表转成红黑树
+ 若节点已经存在，则用新值替换旧值
+ 若桶满了（默认容量16*扩容因子0.75），就需要resize（扩容2倍后重排）



HashMap:如何有效的减少碰撞

除了被动的通过树化提升性能。减少碰撞也是提升性能关键

+ 扰动函数：促使元素位置均匀分布，减少碰撞几率。原理是两个不相等的对象，返回不同的hashCode，或者位置分布均匀
+ 使用final对象，并采用合适的equals()和hashCode()方法

**HashMap扩容问题**

+ 多线程环境下，调整大小会存在条件竞争，容易造成死锁
+ rehashing是一个比较耗时的过程，即HashMap里面的键值对，要移动到新的HashMap里面



**HashTable**

+ Java早期类库提供的哈希表实现
+ 线程安全：涉及到修改HashTable的方法，使用synchronized修饰
+ 多个线程对调用Hashtable对象的同步发，串行化方式运行，性能较差
+ 行为上大致与HashMap类似。没有树化逻辑

 



前面的HashMap也不是线程安全的，要使其线程安全，可以 Collections.synchronizedMap()方法，就可以将HashMap包装成一个线程安全的实例。synchronizedMap线程安全的原因，它有一个Object mutex实例，用来互斥对象成员，对里面所有public方法使用synchronized对mutex进行加锁。而Hashtble的锁是this，他门的原理几乎没有差别，只是锁定的对象不一样。因此这两者再线程环境下，由于都是串行执行的，效率比较低。为了解决这个问题，ConccurentHashMap应运而生。

Q：如何优化Hashtble

A：通过锁的细粒度话，将整个锁拆解成多个锁进行优化。

**ConccurentHashMap**

+ 早期的ConcurrentHashMap是通过分段锁Segment来实现的。数组+链表结构
+ Java8之后，取消了分段锁，采用了CAS+synchronized使锁更细化来保证并发安全

 早期的ConcurrentHashMap，通过分段所Segment 来实现的  。当一个线程占用一把锁，然后访问一段数据的时候，位于其他Segment的数据也能被其他线程同时访问，默认会分配16个segment。理论上会比Hasptable提高了16倍。逻辑上就是将HashMap里面的table数组多个子数组，每个子数组配置一把锁，线程再获取到某把分段锁的时候，才能操作子数组，而其他线程想要操作该子数组，只能被阻塞。如果其他线程操作的其他未被占用的子数组，是不会被阻塞的。

Java8之后，取消了分段锁，采用了CAS+synchronized使锁更细化来保证并发安全。同时还做了更进一步的优化，跟JAVA8的HashMap数据结构一样，数组+链表+红黑树。synchronized只锁定当前链表和红黑树的首节点，只要Hash不冲突，就不会产生并发问题。



**ConcurrentHashMap put方法的逻辑**

1. 判断 Node[]数组是否初始化，没有则进入初始化操作
2. 通过hash定位数组的索引坐标，是否有有Node节点，如果没有则使CAS进行进行添加（链表的头节点），添加失败则进入下次循环。
3. 检查到内部正在扩容，就帮助一起扩容。
4. 如果 f!=null,则使用synchronized锁住f元素（链表/红黑二叉树的头元素）
   1. 如果Node(链表结果)则执行链表的添加操作
   2. 如果是TreeNode(树形结构)则执行数的添加擦着
5. 判断链表长度邮局达到临界值8，当然这个8是默认值，也可以去做调整，当节点数草果这个值就需要把链表结果转树结构



**ConcurrentHashMap总结**

+ 比起Segment 锁拆的更细
+ 首先使用无所操作CAS插入头节点，失败则循环重试
+ 如果头节点已经存在，则尝试获取头节点的同步锁，在进行操作



ConcurrentHashMap需要注意的店

+ size()方法和mappingCount()方法的异同，两者计算是否准确？
+ 多线程环境下如何进行扩容



**三者的区别**

+ HashMap 线程不安全的，底层是通过 数组+链表+红黑树
+ Hashtable是线程安全的，锁住整个对象 。底层是 数组+链表
+ ConcurrentHashMap线程安全，CAS+同步锁，数组+链表+红黑树
+ HashMap的key和value骏可以为null，其他两个不行





# J.U.C知识点梳理

java.util.concurrent:提供了并发编程的解决方案

**两大核心:**

+ CAS:是java.util.concurrent.atomic包的基础。
+ AQS:是java.util.concurrent.locks包以及一些常用类 Semophtore，ReentrantLock等类的基础

**J.U.C包的分类**

+ 线程执行器executor
+ 锁locks
+ 原子变量类atomic
+ 并发工具tools
+ 并发集合collections

![image-20200909211239556](md-images/08-Java常用类库与技巧/image-20200909211239556.png)





**并发工具类tools**

闭锁ConuntDownLatch:让主线等待一组事件发生后继续执行。事件指的是CountDownLatch里的countDown()方法。

![image-20200909212604649](md-images/08-Java常用类库与技巧/image-20200909212604649.png)

![image-20200909212658625](md-images/08-Java常用类库与技巧/image-20200909212658625.png)



栅栏CyclicBarrer：阻塞当前线程，等待其他线程。等待其他线程，且会阻塞自己当前线程，所有线程必须同时到达栅栏位置后，才能继续执行。所有线程达到栅栏除，可以触发执行另外一个预先设置的线程。

![image-20200909212830901](md-images/08-Java常用类库与技巧/image-20200909212830901.png)

![image-20200909214534643](md-images/08-Java常用类库与技巧/image-20200909214534643.png)





信号量Semaphore：控制某个资源可以被同时访问的线程个数

![image-20200909214635163](md-images/08-Java常用类库与技巧/image-20200909214635163.png)

![image-20200909214704117](md-images/08-Java常用类库与技巧/image-20200909214704117.png)



交换器Exchanger

两个线程达到同步点后，互相交换数据

![image-20200909214804109](md-images/08-Java常用类库与技巧/image-20200909214804109.png)

![image-20200909214853974](md-images/08-Java常用类库与技巧/image-20200909214853974.png)





**BlockingQueue**:提供可阻塞的入队和出队操作。主要用于生产者-消费者模式，在多线程场景时生产者线程在队列的尾部添加元素，而消费者线程则在队列的头部消费元素，通过这种方式能够达到将任务的生产者和消费者进行隔离的目的。

+ ArrayBlockingQueue:一个由数组结构组成的有界阻塞队列
+ LinkedBlockingQueue:一个有链表结构组成的有界/无界阻塞队列
+ PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列
+ DealyQueue:一个使用优先级队列实现的无界阻塞队列
+ SynchronousQueue:一个不存储元素的阻塞队列
+ LinkedTransferQueue:一个由链表结构组成的无界阻塞队列
+ LinkedBlockingDeque:一个由链表结构组成的双向阻塞队列

![image-20200909215157970](md-images/08-Java常用类库与技巧/image-20200909215157970.png)

