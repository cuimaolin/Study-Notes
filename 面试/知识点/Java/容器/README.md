Java的容器有哪些？（华为Cloudbu一面，腾讯CDG一面）

> 容器：
>
> - Collection。存储对象的集合
    >   - Set
    >     - TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如HashSet，HashSet查找的时间复杂度为O(1)，TreeSet为O(logn)。
    >     - HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并失去了元素的插入顺序信息，也就是说使用Iterator遍历HashSet得到的结果是不确定的。
>     - LinkedHashSet：具有HashSet的查找效率，并且内部使用双向链表维护元素的插入顺序。
>   - List
      >     - ArrayList：基于动态数组实现，支持随机访问。
      >     - Vector：和ArrayList类似，但它是线程安全的
>     - LinkedList：基于双向链表实现，只能顺序访问，但是可以快速在链表中间插入和删除元素。不仅如此，LinkedList还可以用于栈、队列和双向队列。
>   - Queue
      >     - LinkedList：可以用它来实现双向队列
      >     - PriorityQueue：基于堆结构实现，可以用它来实现优先队列。
> - Map
    >   - TreeMap：基于红黑树实现
    >   - HashMap：基于哈希表实现
>   - HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
>   - LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

Java HashMap是怎么实现的（今日头条一面，阿里3一面，百度一面）

> JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间

如果HashMap的链表过长是否会影响查找效率（阿里3一面）

HashMap为什么是线程不安全的？（腾讯CDG一面）

> - 多线程的put可能导致元素的丢失
> - put和get并发时，可能导致get为null
> - JDK7中并发put会造成循环链表，导致get出现死循环

JDK8 HashMap是否会导入死循环？（阿里3一面）

> 主要原因在于并发下的 Rehash 会造成元素之间会形成一个循环链表。不过，jdk 1.8 后解决了这个问题，但是还是不建议在多线程下使用 HashMap,因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。并发环境下推荐使用 ConcurrentHashMap 。
>
> 详情请查看：[疫苗：Java HashMap的死循环 | 酷 壳 - CoolShell](https://coolshell.cn/articles/9606.html)

HashMap为什么大于8要转换为红黑树（今日头条一面）

> 理想情况下，在随机哈希码下，哈希表中节点 的频率遵循泊松分布，而且根据统计，忽略方差，列表长度为K的期望出现的次数是以上的结果，可以看到其实在8的时候概率就已经很小了，再往后调整并没有很大意义。

concurrentHashMap如何扩容？（腾讯CDG一面）

> - 当前键值对的数量大于loadFactor✖capacity时即进行扩容
> - 当需要扩容时，令capacity为原来的两倍
> - 扩容操作使用resize()实现，需要把oldTable的所有键值对重新插入newTable中

HashMap和ConcureentHashMap的区别？HashMap如何实现线程安全？ConcurentHashMap锁的实现，是在哪一个版本实现的，哪一个版本进行改进（腾讯CDG一面，腾讯云二面，百度一面）

> JDK1.7
>
> 首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。
>
> **`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。
>
> Segment 实现了 `ReentrantLock`,所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。
>
> JDK1.8
>
> `ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）
>
> `synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。

向两个HashMap分别put1000条和10000条数据，并且其初始容量为1000和100000，在put的过程中会进行扩容吗（腾讯云一面）

> 会啊，HashMap允许的size小于loadFactor×capacity，因此会进行扩容

concurrentHashMap的size如何计算（腾讯云一面）

红黑树的特点，平衡二叉树和红黑树的区别，查询和插入的时间复杂度（阿里3一面，腾讯云二面，百度一面）

> [红黑树深入剖析及Java实现](https://tech.meituan.com/2016/12/02/redblack-tree.html)
>
> 1. 任何一个节点都有颜色，黑色或者红色
> 2. 根节点是黑色的
> 3. 父子节点之间不能出现两个连续的红节点
> 4. 任何一个节点向下遍历到其子孙的叶子节点，所经过的黑节点个数必须相等
> 5. 空节点被认为是黑色的
>
> 红黑树在插入、删除等操作，不会像平衡树那样，频繁着破坏红黑树的规则，所以不需要频繁地调整。红黑树在最坏情况下，也能在O(logn)地时间复杂度下查找某个节点
>
> 整个红黑树的查找，插入和删除都是O(logN)的

红黑树在Java中的应用（腾讯云二面）
