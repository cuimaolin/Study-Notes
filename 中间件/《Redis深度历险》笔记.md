### Redis可以做什么

1. 记录帖子的点赞数、评论数和点击数（hash）；
2. 记录用户的帖子ID**列表（排序），便于快速**显示用户的帖子列表（zset）；
3. 记录帖子的标题、**摘要、作者和封面信息，用于列表**页展示（hash）；
4. 记录帖子的点赞用户**ID列表**，评论**ID列表，用于**显示和去重计数（zset）；
5. 缓存近期热帖内容（**帖子内容**空间占用比较大），**减少数据库**压力（hash）；
6. 记录帖子的相关文章ID，**根据内容推荐相关帖子**（list）；
7. **如果帖子ID是整数自增的，可以使用Redis来分配帖子 ID**（计数器）；
8. **收藏集和帖子**之间的关系（zset）；
9. 记录热榜帖子**ID列表**，总热榜和分类热榜（zset）；
10. 缓存用户行为历史，用于恶意行为过滤（zset，hash）。

### Redis基础数据结构

#### String（字符串）

字符串String是Redis最简单的数据结构。Redis所有的数据结构都是以唯一的key字符串作为名称，然后通过这个唯一Key值来获取相应的value数据。不同类型的数据结构的差异在于value结构不一样。

String的一个常见用途是缓存用户信息。我们将用户信息结构体使用JSON序列化为字符串，然后将序列化后的字符串塞进Redis来缓存。同样，取用户信息会经过一次反序列化的过程。

Redis的字符串是动态字符串，是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配，内部的当前字符串时间分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

#### list（列表）

Redis的列表相当于Java语言里面的LinkedList，注意它是链表而不是数组。这意味着list的插入和删除操作非常快，时间复杂度为O(1)，但是索引定位很慢，时间复杂度为O(n)。

当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。

Redis的列表结构常用来保存异步队列使用。将需要延后处理的任务结构体序列化成字符串塞进Redis的列表，另一个线程从这个列表中轮询数据进行处理。

Redis底层存储的不是一个简单的==Linkedlist==，而是被称为快速链表==quicklist==的一个结构。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是==ziplist==，也即是压缩列表。它将所有的元素都紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改为==quicklist==。因为普通的链表需要的附加指针空间太大，会比较浪费空间，而且会加重内存的碎片化。比如这个列表里存的只是==int==类型的数据，结构上还需要两个额外的指针==prev==和==next==。所以Redis将链表和==ziplist==结合起来组成了==quicklist==。也就是多个==ziplist==使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

#### hash（字典）

Redis的字典相当于Java语言里面的HashMap，它是无序字典。内部实现结构上同Java的HashMap也是一致的，同样的数组+链表二维结构。第一维hash的数组位置碰撞时，就会将碰撞的元素使用链表串接起来。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210116121750440.png" alt="image-20210116121750440" style="zoom:50%;" />

不同的是，Redis的字典的值只能是字符串，另外它们rehash的方式也不一样，因为Java的HahMap在字典很大时，rehash是一个耗时的操作，需要一次性全部rehash。Redis为了高性能，不能堵塞服务，所以采用了渐进式rehash策略。

渐进式rehash会在rehash的同时，保留新旧两个hash结构，查询会同时查询两个hash结构，然后在后续的定时任务中以及hash的子指令中，循序渐进地将旧hash的内容一点点迁移到新的hash结构中。

当hash移除了最后一个元素时，该数据结构自动被删除，内存被回收。

hash结构也可以用来存储用户信息，不同于字符串需要全部序列化整个对象，hash可以对用户结构中的每个字段单独存储。这样当我们需要获取用户信息时可以进行部分获取。而以整个字符串的形式来保存用户信息的话就只能一次性全部读取，这样会比较浪费网络流量。

hash的缺点在于其结构的存储消耗要高于单个字符串。

#### set（集合）

Redis的集合相当于Java语言里面的HashSet，它内部的键值对是无序且唯一的。它的内部实现相当于一个特殊的字典，字典所有的value都是一个值NULL。

当集合中的最后一个元素移除之后，数据结构自动删除，内存被回收。set结构可以用来存储活动中奖的用户ID，因为有去重功能，可以保证同一用户不会中将两次。

#### zset（有序列表）

zset类似于Java中的SortedSet和HashMap的结合体，一方面它是一个set，保证了内部value的唯一性，另一方面它可以给每个value赋予一个score，代表这个value的排序权重。它的内部实现用的是一种叫做[跳跃列表]的数据结构。

zset中最后一个value被移除后，数据结构自动删除，内存被回收。zset可以用来保存粉丝列表，value是粉丝的ID，score是关注时间。我们可以对粉丝列表按关注时间进行排序。

zset还刻意用来存储学生的成绩，value的值是学生的ID，score是他的考试成绩。我们可以对成绩按分数进行排序就可以得到他的名次。

##### 跳跃列表

因为zset要支持随机的插入与删除，所以它不好使用数组来表示。我们需要这个链表按照score值进行排序。这意味着每当有新元素需要插入时，要定位到特定位置的插入点，这样才可以继续保证链表是有序的。通常我们会通过二分查找来找到插入点，但是二分查找的对象必须是数组，只有数组才可以支持快速位置定位。

跳跃列表类似于公司的层级制，最下面一层所有的元素都会串起来。然后每隔几个元素挑选出一个代表来，再将这几个代表使用另外一级指针串起来。然后在这些代表里再跳出二级代表，再串起来。最终就形成了金字塔结构。类似于世界地图的位置：亚洲-->中国->安徽省->安庆市->枞阳县->汤沟镇->田间村->xxxx号。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210116135617765.png" alt="image-20210116135617765" style="zoom:67%;" />

[跳跃列表]之所以[跳跃]，是因为内部的元素可能[身兼数职]，比如上图中间的这个元素，同时处于L0、L1和L2层，可以快速在不同层次之间进行[跳跃]。

定位插入点时，现在顶层进行定位，然后下潜到下一级定位，一直下潜到最底层找到合适的位置，将新元素插入进去。

#### 容器型数据结构的通用规则

**create if not exists**：如果容器不存在，那就创建一个。

**drop if no elements**：如果容器里元素没有了，那么立即删除元素，释放内存。

### 应用1：分布式锁

分布式应用进行逻辑处理时经常会遇到并发问题。

比如一个操作要修改用户的状态，修改状态需要先读出用户的状态，在内存里进行修改，改完了再存回去。如果这样的操作同时进行了，就会出现并发问题，因为读取和保存状态两个状态不是原子的。

#### 分布式锁

分布式锁本质上是要实现的目标就是在Redis里面占一个“茅坑”，当别的进程也要来占时，发现已经有人蹲在那里了，就只好放弃或稍后再试。

占坑一般是使用setnx(set if not exists)指令，只允许被一个客户端占坑。先来占用，用完了，再调用del指令释放茅坑。

```bash
> setnx lock:codehole true
OK
do something critical
> del lock:codehole
(integer) 1
```

但是有个问题，如果逻辑执行到中间出现异常了，可能会导致del指令没有被调用，这样就会陷入死锁，锁永远得不到释放。

于是我们在拿到锁之后，再给锁加上一个过期时间，比如5s，这样即使中间出现异常可以保证5s之后锁会自动释放。

```bash
> setnx lock:codehole true
OK
> expire lock:codehole 5
...do something critical...
> del lock:codehole
(integer) 1
```

但是以上逻辑还有问题。如果在setnx和expire之间服务器进程突然挂掉了，可能是因为机器掉电或者是被人为杀掉了，就会导致expire得不到执行，也会造成死锁。

这种问题的根源就在于setnx和expire是两条指令而不是原子指令。如果这两条可以一起执行就不会出现问题。但是在这里不能使用Redis事务来解决。因为expire是依赖于setnx的执行结果的，如果setnx没抢到锁，expire是不应该执行的。事务里没有if-else分支逻辑，事务的特点是一口气执行，要么全部执行要么一个都不执行。

Redis 2.8 版本中作者假如了set指令的扩展参数，使得setnx和expire指令可以一起执行，彻底解决了分布式锁的乱象。

```bash
> set lock:codehole true ex 5 nx
OK
... do somthing critical ...
> del lock:codehole
```

#### 超时问题

Redis的分布式锁不能解决超时问题，如果在加锁和释放锁之间的逻辑执行地太长，以至于超出了锁的超时限制，就会出现问题。因为这时候锁过期了，第二个线程重新持有了这把锁，但是紧接着第一个线程执行完了业务逻辑，就把锁给释放了，第三个线程就会在第二个线程逻辑执行完之间拿到了锁。

为了避免这个问题，Redis分布式锁不要用于较长时间的任务。如果真的偶尔出现了，数据出现的小波错乱可能需要人工介入解决。

```bash
tag = random.nextint()
if redis.set(key, tag, nx=True, ex=5):
	do_somethind()
	redis.delifequals(key, tag)
```

有一个更加安全的方案是为set指令的value参数设置为一个随机数，释放锁时先匹配随机数是否一致，然后再删除key。但是匹配value和删除key不是一个原子操作，Redis也没有提供类似于delifequals这样的指令，这样需要使用Lua脚本来处理了，因为Lua脚本可以保证连续多个指令的原子性执行。

```lua
# delifequals
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

#### 可重入性

可重入性是指线程在持有锁的情况下再次请求加锁，如果一个锁支持同一个线程多次加锁，那么这个锁是可重入的。比如Java语言的ReentrantLock就是可重入锁。Redis分布式锁如果要支持可重入，需要对客户端的set方法进行包装，使用线程的Threadlock变量存储当前锁的计数。	

```java
public class RedisWithReentrantLock {
	private ThreadLocal<Map> lockers = new ThreadLocal<>();
	private Jedis jedis;
	public RedisWithReentrantLock(Jedis jedis) {
		this.jedis = jedis;
    }
	private boolean _lock(String key) {
        // redis进行加锁
		return jedis.set(key, "", "nx", "ex", 5L) != null;
	}
	private void _unlock(String key) {
        // redis进行解锁
		jedis.del(key);
	}
	private Map <String, Integer> currentLockers() {
		Map <String, Integer> refs = lockers.get();
		if (refs != null) {
			return refs;
		}
		lockers.set(new HashMap<>());
		return lockers.get();
	}
	public boolean lock(String key) {
        Map refs = currentLockers();	// 当前锁
        Integer refCnt = refs.get(key);	// 当前锁的计数
        if (refCnt != null) {			// 若已计算，则直接重入
            refs.put(key, refCnt + 1);
            return true;
        }
        boolean ok = this._lock(key);	// 尝试加锁
        if (!ok) {						// 加锁失败返回false
            return false;
        }
        refs.put(key, 1);				// 加入计数
        return true;
	}
    public boolean unlock(String key) {
        Map refs = currentLockers();	// 当前锁
        Integer refCnt = refs.get(key);	// 当前锁的计数
        if (refCnt == null) {			// 无计数则解锁失败
            return false;
        }
        refCnt -= 1;					// 锁计数减1
        if (refCnt > 0) {				// 减1后还有计数则更新计数
            refs.put(key, refCnt);
        } else {						// 减1后无计数则删除当前锁
            refs.remove(key);
            this ._unlock(key);
        }
        return true;
    }
    public static void main(String[] args) {
    	Jedis jedis = new Jedis();
    	RedisWithReentrantLock redis = new RedisWithReentrantLock(jedis);
    	System.out.println(redis.lock("codehole"));
    	System.out.println(redis.lock("codehole"));
    	System.out.println(redis.unlock("codehole"));
    	System.out.println(redis.unlock("codehole"));
    }
}
```

#### Redlock 算法

在Sentinel集群中，主节点挂掉时，从节点会取而代之，客户端却并没有明显感知。原先第一个客户端在主节点中申请成功了一把锁，但是这把锁还没有来得及同步到从节点时，主节点突然挂掉了。然后从节点变成了主节点，这个新节点内部没有这把锁，所以当另一个客户端请求加锁时，立即就批准了。这样导致系统中同样一把锁呗两个客户端同时持有，不安全性由此产生。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210118104323780.png" alt="image-20210118104323780" style="zoom:80%;" />

不过这种不安全也仅仅是在主从发生failover的情况下才会产生，而且持续时间极短，业务系统多数情况下可以容忍。

为了解决这个问题，Antirez发明了Redlock算法，它的流程比较复杂，已经有了很多开源的library做了良好的封装，用户可以拿来即用，比如redlock-py。

```python
import redlock

addrs = [{
    "host": "localhost",
    "port": 6379,
    "db": 0
}, {
    "host": "localhost",
    "port": 6479,
    "db": 0
}, {
    "host": "localhost",
    "post": 6579,
    "db": 0
}]
dlm = redlock.Redlock(addrs)
success = dlm.lock("user-lck-laoqian", 5000)
if success:
    print "lock success"
    dlm.unlock('user-lck-laoqian')
else:
    print 'lock failed'
```

为了使用Redlock，需要提供多个Redis实例，这些实例之间相互独立没有主从关系。同很多分布式算法一样，redlock也使用[大多数机制]。

加锁时，它会向过半节点发送`set(key, value, nx=True, ex=xxx)`指令，只要过半节点`set`成功，那就认为加锁成功。释放锁时，需要向所有节点发送del指令。不过Redlock算法还需要考虑出错重试、时钟漂移等很多细节问题，同时因为Redlock需要向多个节点进行读写，意味着相比单实例Redis性能会下降一些。

##### 使用场景

如果你很在乎高可用性，希望挂了一台redis完全不受影响，那就应该考虑redlock。不过代价也是有的，需要很多的redis实例，性能也下降了，代码上还需要引入额外的library，运维上也需要特殊对待。

### 应用2：延迟队列

Redis的list(列表)数据结构常用来作为异步消息队列使用，使用`rpush/lpush`操作入队列，使用`lpop`和`rpop`来出队列。

```bash
> rpush notify-queue apple banana pear
(integer) 3
> llen notify-queue
(integer) 3
> lpop notify-queue
"apple"
> llen notify-queue
(integer) 2
> lpop notify-queue
"banana"
> llen notify-queue
(integer) 1
> lpop notify-queue
"pear"
> llen notify-queue
(integer) 0
> lpop notify-queue
(nil)
```

#### 队列空了怎么办

如果队列空了，客户端就会陷入pop的死循环，不停地pop，没有数据，接着再pop，又没有数据。这将会浪费生命的空轮询。空轮询不但拉高了客户端的CPU，redis的QPS也会被拉高，如果这样空轮询的客户端有几十来个，Redis的慢查询可能会显著增多。

通常我们使用sleep来解决这个问题，让线程睡一会，睡个1s就可以了。不但客户端的CPU能降下来，Redis的QPS也降下来。

```java
Thread.sleep(1000) # java 睡 1s
```

#### 队列延迟

用上述睡眠的办法可以解决问题。但是有个小问题，那就是睡眠会导致消息的延迟增大。如果只有1个消费者，那么这个延迟就是1s。如果有多个消费者，这个延迟会有所下降，因为每个消费者的睡觉时间是岔开的。

可以利用`blpop/brpop`替代前面的`lpop/rpop`来显著地降低延迟。这两个指令的前缀字符`b`代表的是`blocking`,也就是阻塞读。

阻塞读在队列没有数据的时候，会立即进入休眠状态，一旦数据到来，则立刻醒过来。消息延迟几乎为0。

#### 空闲连接自动断开

如果线程一直阻塞在那里，Redis的客户端连接就变成了闲置连接，闲置过久，服务器一般会主动断开连接，减少闲置资源占用。这个时候`blpop/brpop`会抛出异常来。所以在编写客户端消费者的时候应注意捕获异常和重试。

#### 锁冲突处理

一般有3种策略来处理加锁失败：

1. 直接抛出异常，通知用户稍后重试；
2. sleep一会再重试；
3. 将请求转移至延迟队列，过一会再试。

##### 直接抛出特定类型的异常

这种方式比较适合由用户直接发起的请求，用户看到错误对话框后，会先阅读对话框的内容，再点击重试，这样就可以起到人工延时的效果。如果考虑到用户体验，可以由前端代码代替用户自己来进行延时重试控制。它本质上是对当前请求的放弃，由用户决定是否重新发起新的请求。

##### sleep

sleep会阻塞当前的消息处理线程，会导致队列的后续消息处理出现延迟。如果碰撞的比较频繁或队列里消息比较多，sleep可能并不合适。如果因为个别死锁的key导致加锁不成功，线程会彻底堵死，导致后续消息永远得不到及时处理。

##### 延迟队列

延迟队列可以使用Redis的zset（有序列表）来实现。我们将消息序列化成一个字符串作为zset的`value`，这个消息的到期处理时间作为`score`，然后用多个线程轮询`zset`获取到期的任务进行处理，多个线程是为了保障可用性，万一挂了一个线程还有其他线程可以继续处理。因为有多个线程，所以需要考虑并发争抢任务，确保任务不能被多次执行。

Redis的zrem方法是多线程多进程争抢任务的关键，它的返回值决定了当前实例有没有抢到任务，因为loop方法可能会被多个线程、多个进程调用，同一个任务可能会被多个进程线程抢到，通过zrem来决定唯一的属主。注意一定要对handle_msg进行异常捕获，避免因为个别任务处理问题导致循环异常退出。

> zrem命令用于移除有序集中的一个或多个成员，不存在的成员将被忽略；
>
> 当key存在但不是有序集类型时，返回一个错误。

```java
import java.lang.reflect.Type;
import java.util.Set;
import java.util.UUID;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import redis.clients.jedis.Jedis;
public class RedisDelayingQueue<T> {
    static class TaskItem<T> {
    	public String id;
    	public T msg;
    }
    // fastjson 序列化对象中存在 generic 类型时，需要使用 TypeReference
    private Type TaskType = new TypeReference<TaskItem<T>>() { }.getType();
    private Jedis jedis;
    private String queueKey;
    public RedisDelayingQueue(Jedis jedis, String queueKey) {
    	this.jedis = jedis;
    	this.queueKey = queueKey;
    }
    public void delay(T msg) {
    	TaskItem task = new TaskItem();
    	task.id = UUID.randomUUID().toString(); // 分配唯一的 uuid
    	task.msg = msg;
    	String s = JSON.toJSONString(task); // fastjson 序列化
    	jedis.zadd(queueKey, System.currentTimeMillis() + 5000, s); // 塞入延时队列 ,5s 后再试
    }
    public void loop() {
    	while (!Thread.interrupted()) {
    		// 只取一条，根据到期时间
    		Set values = jedis.zrangeByScore(queueKey, 0, System.currentTimeMillis(), 0, 1);
    		if (values.isEmpty()) {
    			try {
    				Thread.sleep(500); // 歇会继续
    			}
    			catch (InterruptedException e) {
    				break;
    			}
    			continue;
    		}
    		String s = values.iterator().next();
    		if (jedis.zrem(queueKey, s) > 0) { 					// 抢到了，利用zrem决定任务唯一的属主
    			TaskItem task = JSON.parseObject(s, TaskType); // fastjson 反序列化
    			this.handleMsg(task.msg);
    		}
    	}
    }
	public void handleMsg(T msg) {
		System.out.println(msg);
	}
	public static void main(String[] args) {
		Jedis jedis = new Jedis();
		RedisDelayingQueue queue = new RedisDelayingQueue<>(jedis, "q-demo");
		Thread producer = new Thread() {
			public void run() {
				for (int i = 0; i < 10; i++) {
					queue.delay("codehole" + i);
				}
			}
		};
		Thread consumer = new Thread() {
            public void run() {
                queue.loop();
            }
		};
        producer.start();
        consumer.start();
		try {
			producer.join();		//主线程进入TIMED_WAITTING状态等待producer结束
			Thread.sleep(6000);		//sleep
			consumer.interrupt();	//打断consumer
    		consumer.join();		//主线程等待producuer结束 
		}
		catch (InterruptedException e) {
		}
	}
}
```

##### 进一步优化

上面算法种同一个任务可能会被多个进程取到之后再使用zrem进行争抢，那些没抢到的进程都是白取了一次任务，这是浪费。可以考虑到使用lua scripting来优化一下这个逻辑，将zrangebyscore和zrem一同挪到服务器端进行原子化操作，这样多个进程之间争抢任务时就不可会出现这种浪费了。

### 应用5：布隆过滤器

我们在使用新闻客户端看新闻时，它会给我们不停地推荐新的内容，它每次推荐时要去重，去掉那些已经看过的内容。如果历史记录存储在关系数据库里，去重就需要频繁地对数据库进行exists查询，当系统并发量很高时，数据库是很难抗住压力的。

使用set会存储每个元素的内容，而布隆过滤器仅仅存储元素的指纹。

而这时，布隆过滤器（Bloom Filter）闪亮登场了，它就是专门用来解决这种去重问题的。它在起到去重的同时，在空间上还能节省90％以上，但是有一定误判的概率。

#### 什么是布隆过滤器

布隆过滤器可以理解为一个不怎么精确的set结构，当你使用它的contains方法判断某个对象是否存在时，它可能会误判。但是布隆过滤器也不是特别不精确，只要参数设置的合理，他的精确度可以控制的相对足够惊喜，只会有小小的误判概率。

当布隆过滤器说某个值存在时，这个值可能不存在；当它说不存在时，那就肯定不存在。套在上面的使用场景种，布隆过滤器能准确过滤掉那些已经看过的内容，那些没有看过的新内容，它也会过滤掉极小一部分（误判），但是绝大多数内容它都能准确识别。这样就可以完全保证推荐给用户的内容都是无重复的。

#### 布隆过滤器的原理

每个布隆过滤器对应到Redis的数据结构里面就是一个大型的位数组和几个不一样的无偏hash函数。所谓的无偏就是能把元素的hash值算得比较均匀。

向布隆过滤器中添加key时，会使用多个hash函数对key进行hash算得一个证书索引值然后对位数组长度进行取模运算得到一个位置，每个hash函数都会算得一个不同的位置。再把数组的这几个位置都置为1就完成了add操作。

向布隆过滤器询问key是否存在时，跟add一样，也会把hash的几个位置都算出来，看看位数组中这几个位置是否都为1，只要有一个位为0，那么说明布隆过滤器中这个key不存在。如果都是1，这并不能说明这个key就一定存在，只是极有可能存在，因此这些位被置为1可能是因为其他的key存在所致。如果这个位数组比较稀疏，这个概率就会很大，如果这个位数组比较拥挤，这个概率就会降低。

使用时不要让实际元素远大于初始化大小，当实际元素开始超出初始化大小时，应该对布隆过滤器进行重建，重新分配一个size更大的过滤器，再将所有的历史元素批量add进去（这就要求我们在其他的存储器中记录所有的历史元素）。因为error_rate不会因为数量超出就急剧增加，这就给我们重建过滤器提供了较为宽松的时间。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210118165114044.png" alt="image-20210118165114044" style="zoom:67%;" />

#### 布隆过滤器的其他应用

在爬虫系统中，我们需要对URL进行去重，已经爬过的网页就可以不用爬了。但是URL太多了，几千万几个亿，如果用一个集合装下这些URL地址非常浪费空间的。这时候就可以考虑使用布隆过滤器。它可以大幅度降低去重存储消耗，只不过也会使得爬虫系统错过少量的页面。

布隆过滤器在NoSQL数据库领域使用非常广泛，我们平时用到的HBase、Cassandra还有LevelDB、RocksDB内部都有布隆过滤器结构，布隆过滤器可以显著降低数据库的IO请求数量。当用户来查询某个row时，可以先通过内存的布隆过滤器过滤掉大量不存在的row请求，然后再去磁盘进行查询。

邮箱系统的垃圾邮件过滤功能也普遍用到了布隆过滤器，因为用了这个过滤器，所以平时也会遇到某些正常的邮件被放进了垃圾邮箱目录中，这个就是误判所致，概率很低。

### 原理1：线程IO模型

Redis是个单线程程序！

Redis单线程为什么还能这么快？

> 因为它所有的数据都在内存中，所有的运算都是内存级别的运算。正因为Redis是单线程，所以要小心使用Redis指令。对于那些时间复杂度为O(n)级别的指令，一定谨慎使用，一不小心就可能会导致Redis卡顿。

Redis单线程如何处理那么多的并发客户端连接？

> 多路复用

#### 非阻塞IO(Non-blocking I/O)

当我们调用套接字的读写方法，默认它们是阻塞的，比如==read==方法要传递进去一个参数==n==, 表示读取这么多字节后再返回，如果没有读够线程就会卡在那里，直到新的数据到来或者连接关闭了，==read==方法才可以返回，线程才能继续处理。而==write==方法一般来说不会阻塞，除非内核为套接字分配的写缓冲区已经满了，==write==方法就会阻塞，直到缓存区中有空闲空间挪出来了。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210120090630576.png" alt="image-20210120090630576" style="zoom:67%;" />

非阻塞型IO在套接字对象上提供了一个选项==Non_Blocking==，当这个选项打开时，读写方法不会阻塞，而是能读多少读多少，能写多少写多少。能读多少取决于内核为套接字分配的读缓冲区内部的数据字节数，能写多少取决于内核为套接字分配的写缓冲区的空闲空间字节数。读方法和写方法都会通过返回值来告知程序实际读写了多少字节。

有了非阻塞 IO 意味着线程在读写 IO 时可以不必再阻塞了，读写可以瞬间完成然后线程可以继续干别的事了。

##### 事务轮询（多路复用）

非阻塞IO有个问题，那就是线程要读数据，结果读了一部分就返回了，线程如何知道何时才应该继续读。也就是当数据到来时，线程如何得到通知。写也是一样，如果缓冲区满了，写不完，剩下的数据何时才应该继续写，线程也应该得到通知。

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210120091613363.png" alt="image-20210120091613363" style="zoom:67%;" />

事务轮询API就是用来解决这个问题的，最简单的时间轮询API是==select==函数，它是操作系统提供给用户程序的API。输入是读写描述符列表==read_fds&write_fds==，输出是对应的可读可写事件。同时还提供一个==timeout==参数，如果没有任何事件到来，那么就最多等待==timeout==时间，线程处于阻塞状态。一旦期间有任何事件到来，就可以立即返回。时间过了之后还是没有任何时间到来，也会立即返回。拿到事件后，线程就可以继续挨个处理相应的事件。处理完了继续过来轮询。于是线程就进入了一个死循环，我们把这个死循环称为事件循环，一个循环一个周期。

每个客户端套接字==socket==都有对应的读写文件描述符。

```python
read_events, write_events = select(read_fds, write_fds, timeout)
for event in read_events:
    handle_read(event.fd)
for event in write_events:
    handle_write(event.fd)
handle_others()
```

因为我们通过==select==系统调用同时处理多个通道描述符的读写事件，因此我们将这类系统调用称为多路复用API。现代操作系统的多路复用API已经不再使用select系统调用，而改用==epoll(linux)==和==kqueue(freebsd&macosx)==，因为==select==系统调用的性能在描述符特别多时性能会非常差。它们使用起来在形式上可能会有差异，但是本质上还是差不多的。

服务器套接字==serversocket==对象的读操作是指调用==accept==接受客户端新连接。何时有新连接到来，也是通过==select==系统调用的读时间来得到通知的。

事务轮询API就是Java语言里面的NIO技术。

#### 指令队列

Redis会将每个客户端套接字都关联一个指令队列。客户端的指令通过队列来排队进行顺序处理，先到先服务。

#### 响应队列

Redis同样也会为每个客户端套接字关联一个响应队列。Redis服务器通过响应队列来将指令的返回结果回复给客户端。如果队列为空，那么意味着连接暂时处于空闲状态，不需要去获取写事件，也就是可以将当前的客户端描述符从==write_fds==里面移出来。等到队列有数据了，再将描述符放进去。避免==select==系统调用立即返回写事件，结果发现没什么数据可以写。出这种情况的线程会飙高CPU。

#### 定时任务

服务器要响应IO事件外，还要处理其他事情。比如定时任务就是非常重要的一件事。如果线程阻塞在select系统调用上，定时任务将无法得到准时调度。

Redis的定时任务会记录在一个称为==最小堆==的数据结构中。这个堆中，最快要执行的任务排在堆的最上方。在这个循环周期，Redis都会将最小堆里面已经到点的任务立即进行处理。处理完毕后，将最快要执行的任务还需要的时间记录下来，这个时间就是select系统调用的timeout参数。因为Redis知道未来timeout时间内，没有其他定时任务需要处理，所以可以安心睡眠==timeout==的时间。

### 原理2：通信协议

Redis的作者认为数据库系统的瓶颈一般不在于网络流量，而是数据库自身内部逻辑处理上。所以即使Redis使用了浪费流量的文本协议，依然可以取得极高的访问性能。Redis将所有数据都放在内存，用一个线程对外提供服务，单个节点在跑满一个CPU核心的情况下可以达到10w/s的超高QPS。

#### RESP(Redis Serialization Protocol)

RESP是Redis序列化协议的简写。它是一种直观的文本协议，优势在于实现异常简单，解析性能极好。

Redis协议将传输的结构数据分为5种最小单元类型，单元结束时统一加上回车换行符\r\n。

1. 单行字符串 **以+符号开头**；
2. 多行字符串 **以$符号开头，后跟字符串长度**；
3. 整数值 **以：符号开头，后跟整数的字符串形式**；
4. 错误消息 **以-符号开头**；
5. 数组 **以*符号开头，后跟数组长度**

### 原理3：管道

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210120215552105.png" alt="image-20210120215552105" style="zoom:67%;" />

流程描述：

1. 客户端进程调用==write==将消息写到操作系统内核为套接字分配的发送缓冲==send buffer==；
2. 客户端操作系统内核将发送缓冲的内容发送到网卡，网卡硬件将数据通过[网际路由]送到服务器的网卡；
3. 服务器操作系统内核将网卡的数据放到内核为套接字分配的接受缓冲==recv buffer==；
4. 服务器进程调用==read==从接受缓冲中取出消息进行处理；
5. 服务器进程调用==write==将响应消息写到内核为套接字分配的发送缓冲==send buffer==；
6. 服务器操作系统内核将发送缓冲的内容发送到网卡，网卡硬件将数据通过[网际路由]送到客户端的网卡；
7. 客户端操作系统内核将网卡的数据放到内核为套接字分配的接受缓冲==reve buffer==；
8. 客户端进程调用==read==从接受缓冲中取出消息返回给上层业务逻辑进行处理；
9. 结束。

我们开始以为==write==操作是要等到对方收到消息才会返回，但实际不是这样的。==write==操作只负责将数据写到本地操作内核的发送缓冲然后就返回了。剩下的事较给操作系统内核异步将数据同步到目标机器。但是如果发送缓冲满了，那么就需要等待缓冲空出空闲空间来，这个就是写操作IO操作的真正耗时。

我们开始以为==read==操作事从目标机器拉取数据，但实际不是这样的。==read==操作只负责从本地操作系统内核的接受缓冲中取出来就了事了。但如果缓冲是空的，那么就需要等待数据到来，这个就是写操作IO操作的真正耗时。

对于管道来说，连续的==write==操作根本就没有耗时，之后第一个==read==操作会等待一个网络的来回开销，然后所有的响应消息就都已经回送到内核的读缓冲中，后续的==read==操作就可以从缓冲拿到结果，瞬间就返回了。

### 原理8：主从同步

#### CAP原理

- C - Consistent， 一致性；
- A - Availability， 可用性；
- P - Partition tolerance， 分区容忍性；

分布式系统的节点往往都是分布在不同机器上进行网络隔离开的，这意味必然会有网络断开的风险，这个网络断开的场景称为[网络分区]。

在网络分区发生时，两个分布式节点之间无法进行通信，我们对一个节点进行的修改操作将无法同步到另一个节点，所以数据的[一致性]将无法满足，因为两个分布式节点的数据将不再保持一致。除非我们牺牲[可用性]，也就是暂停分布式节点服务，在网络分区发生时，不再提供修改数据的功能，知道网络状况完全恢复正常再继续对外提供服务。

即：网络分区发生时，一致性和可用性两难全。

#### 最终一致

Redis的主从数据是异步同步的，所以分布式的Redis系统并不满足[一致性]要求。当客户端在Redis的主节点修改数据后，立即返回，即使在主从网络断开的情况下，主节点依然可以正常对外提供修改服务，所以Redis满足[可用性]。

Redis保证[最终一致性]，从节点会努力追赶主节点，最终从节点的状态会和主节点的状态将保持一致。如果网络断开了，主从节点的数据将会出现大量不一致，一旦网络恢复，从节点会采用多种策略努力追赶落后的数据，继续尽力保持和主节点一致。

#### 增量同步

Redis同步的是指令流，主节点会将那些对自己的状态产生修改性影响的指令记录在本地的内存buffer中，然后异步将buffer中的指令同步到从节点，从节点一边执行同步的指令流来达到和主节点一样的状态，一边向主节点反馈自己同步到哪里了（偏移量）。

因为内存的buffer是有限的，所以Redis主库不能将所有的记录都记录在内存buffer中。Redis的复制内存buffer是一个定长的环形数组，如果数组内容满了，就会从头开始覆盖前面的内容。

如果因为网络状况不好，从节点在短时间内无法和主节点进行同步，那么当网络状况恢复时，Redis的主节点中那些没有同步的指令在buffer中有可能已经被后续的指令覆盖掉了，从节点将无法直接通过指令流来进行同步，这个时候就需要更加复杂的同步机制--快照同步。

#### 快照同步

快照同步首先需要在主库上进行一次`bgsave`将当前内存的数据全部快照到磁盘文件中，然后再将快照文件的内容全部传送到从节点。从节点将快照文件接受完毕后，立即执行一次全量加载，加载之前先要将当前内存的数据清空。加载完毕后通知主节点继续进行增量同步。

在整个快照同步到进行的过程中，主节点的复制buffer还在不停的往前移动，如果快照同步的时间过长或者复制buffer太小，都会导致同步期间的增量指令在复制buffer中被覆盖，这样就会导致快照同步完成后无法进行增量复制，然后会再次发起快照同步，如此极有可能会陷入快照同步的死循环。

所以务必配置一个合适的复制buffer大小参数，避免快照复制的死循环。

#### 无盘复制

主节点在进行快照同步时，会进行很重的文件IO操作，特别是对于非SSD磁盘存储时，快照会对系统的负载产生较大的影响。特别适当系统在进行AOF的fsync操作时如果发生快照，fsync将会被推迟执行，这就会严重影响主节点的服务效率。

无盘复制是指主服务器直接通过套接字将快照内容发送到从节点，生成快照是一个遍历的过程，主节点会一边遍历内存，一边将序列化的内容发送到从节点，从节点还是跟以前一样，先将收到的内容存储到磁盘文件中，在进行一次性加载。

#### Wait指令

Redis的复制是异步执行的，wait指令可以让异步复制变身同步复制，确保系统的强一致性(不严格)

```bash
> set key value
OK
> wait 1 0
(integer) 1
```

wait提供两个参数，第一个参数是从库的数量N，第二个参数是时间t，以毫秒为单位。它表示wait指令之前的所有写操作同步到N个从库（也就是确保N个从库的同步没有滞后），最多等待时间t。如果时间t=0，表示无限等待知道N个从库同步完成达到一致。

假设出现了网络分区，wait指令第二个参数时间t=0，主从同步无法继续进行，wait指令会永远阻塞，Redis服务器将丧失可用性。

### 源码 1：探索[字符串]内部结构

Redis中的字符串是可以修改的字符串，在内存中它是以字节数组的形式存在的。我们知道C语言里面的字符串标准形式是以NULL作为结束符，但是在Redis里面字符串不是这么表示的。因为要获取NULL结尾的字符串的长度使用的是==strlen==标准库函数，这个函数的算法复杂度是O(n)，它需要对字节数组进行遍历扫描，作为单线程的Redis承受不起。

Redis的字符串叫做[SDS]，也就是==Simple Dynamic String==。它的结构是一个带长度信息的字节数组。

```c
struct SDS<T>{
    T capacity; // 数组容量
    T len; // 数组长度
    byte flags; // 特殊标记位
    byte[] content;	//数组内容
}
```

如代码所示，==content==里面存储了真正的字符串内容，==capacity==表示所分配数组的长度，len表示字符串的实际长度。前面我们提到字符串是可以修改的字符串，它要支持==append==操作。如果数组没有冗余空间，那么追缴操作必然涉及分配新数组，然后将旧内容复制过来，再==append==新内容。如果字符串的长度非常长，这样的内存分配和复制开销就非常大。

```c
/* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
* end of the specified sds string 's'.
*
* After the call, the passed sds string is no longer valid and all the
* references must be substituted with the new pointer returned by the call. */
sds sdscatlen(sds s, const void *t, size_t len) {
	size_t curlen = sdslen(s); // 原字符串长度
	// 按需调整空间，如果 capacity 不够容纳追加的内容，就会重新分配字节数组并复制原字符串的内容到新数组中
	s = sdsMakeRoomFor(s,len);
	if (s == NULL) return NULL; // 内存不足
	memcpy(s+curlen, t, len); // 追加目标字符串的内容到字节数组中
	sdssetlen(s, curlen+len); // 设置追加后的长度值
	s[curlen+len] = '\0'; // 让字符串以\0 结尾，便于调试打印，还可以直接使用 glibc 的字符串函数进行操作
	return s;
}
```

上面的SDS结构使用了泛型==T==，为什么不直接使用==int==呢？

> 这时因为当字符串比较短时，==len==和==capacity==可以使用==byte==和==short==来表示，Redis为了对内存做极致的优化，不同长度的字符串使用不同的结构体来表示。

Redis规定字符串的长度不得超过512M字节。创建字符串时==len==和==capacity==一样长，不会多分配冗余空间，这是因为绝大多数场景下我们不会使用append操作来修改字符串。

#### embsrt vs raw

Redis的字符串有两种存储方法，在长度特别短时，使用==emb==形式存储(embeded)，当长度超过44时，使用==raw==形式存储。

这两种类型有什么区别呢？为什么分界线是44呢？

```bash
> set codehole abcdefghijklmnopqrstuvwxyz012345678912345678
OK
> debug object codehole
Value at:0x7fec2de00370 refcount:1 encoding:embstr serializedlength:45 lru:5958906 lru_seconds_idle:1
> set codehole abcdefghijklmnopqrstuvwxyz0123456789123456789
OK
> debug object codehole
Value at:0x7fec2dd0b750 refcount:1 encoding:raw serializedlength:46 lru:5958911 lru_seconds_idle:1
```

注意上面==debug object==输出有个==encoding==字段，一个字符的差别，存储形式就发生了变化。

为了解释这种现象，我们首先来了解一下Redis对象结构体，所有Redis对象都有下面的这个结构头：

```c
struct RedisObject{
    int4 type;//	4bits
    int4 encoding;//	4bits
    int24 lru;//	24bits
    int32 refcount;// 4bytes
    void *ptr;//	8bytes, 64-bit system
}robj;
```

不同的对象具有不同的类型==type(4bit)==，同一个类型的type会有不同的存储形式==encoding(4bit)==，为了记录对象LRU信息，使用了24个bit来记录LRU信息。每个对象都有个引用计数，当引用计数为0时，对象就会被销毁，内存被回收。==ptr==指针将指向对象内容（body）的具体存储位置。这样一个RedisObject对象头需要占据16字节的存储空间。

接着我们再看SDS结构体的大小，在字符串比较小时，SDS对象头的大小是==capacity+3==，至少是3。意味着分配一个字符串的最小空间占用为19字节（16+3）。

```c
struct SDS {
int8 capacity; // 1byte
int8 len; // 1byte
int8 flags; // 1byte
byte[] content; // 内联数组，长度为 capacity
}
```

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210122215545700.png" alt="image-20210122215545700" style="zoom:67%;" />

如图所示，==embstr==存储形式是这样一种存储形式，它将RedisObject对象头和SDS对象连续存在一起，使用==malloc==方法一次分配。而==raw==存储形式不一样，它需要两次==malloc==，两个对象头在内存地址上一般是不连续的。

而内存分配器 jemalloc/tcmalloc 等分配内存大小的单位都是 2、4、8、16、32、64 等等，为了能容纳一个完整的 ==embstr== 对象，==jemalloc== 最少会分配 32 字节的空间，如果字符串再稍微长一点，那就是 64 字节的空间。如果总体超出了 64 字节，Redis 认为它是一个大字符串，不再使用==emdstr== 形式存储，而该用 ==raw== 形式。

前面我们提到SDS结构体中的==content==中的字符串是以==\0==结尾的字符串，之所以多出这样一个字节，是为了便于直接使用==glibc==的字符串处理函数，以及为了便于字符串调试打印输出。

因此，留给==content==的内容最多只有45（64-19）字节了。字符串又是以\0结尾，所以==embstr==最大能容纳的字符串长度就是44。

#### 扩容策略

字符串在长度小于1M之前，扩容空间采用加倍策略，也就是保留100％的冗余空间。当长度超过1M知乎，为了避免加倍后的冗余空间过大而导致浪费，每次扩容只会多分配1M大小的冗余空间。

### 源码2：探索[字典]内部

dict是Redis服务器中出现最为频繁的复合型数据结构，除了hash结构的数据会用到字典外，整个Redis数据库的所有key和value也组成了一个全局字典，还有带过期时间的key集合也是一个字典。zset集合中存储value和score值的映射关系也是通过dict结构实现的。

```c
struct RedisDb{
    dict* dict;// all keys key=>value
    dict* expires;// all expired keys key=>long(timestamp)
    ...
}

struct zset{
    dict *dict;	// all values value=>score
    zskiplist *zsl;
}
```

#### dict 内部结构

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210123152529702.png" alt="image-20210123152529702" style="zoom:67%;" />

dict结构内部包含两个hashtable，通常情况下只有一个hashtable是有值的。但是在dict扩容缩容时，需要分配新的hashtable，然后进行渐进式搬迁，这时候两个hashtable存储的分别是旧的hashtable和新的hashtable。待搬迁结束后，旧的hashtable被删除，新的hashtable取而代之。

```c
struct dict {
    ...
    dictht ht[2];
}
```

<img src="C:\Users\cuimaolin\AppData\Roaming\Typora\typora-user-images\image-20210123152846882.png" alt="image-20210123152846882" style="zoom:67%;" />

所以，字典数据结构的精华就落在了hashtable结构上了。hashtable的结构和Java的HashMap几乎是一样的，都是通过分桶的方式解决hash冲突。第一维是数组，第二维是链表。数组中存储的是第二维链表的第一个元素的指针。

```c
struct dictEntry{
    void* key;
    void* val;
    dictEntry* next;	//链接下一个entry
}
struct dictht{
    dictEntry** table;	//二维
    long size;			//第一维数组的长度
    long used;			//hash表的元素个数
    ...
}
```

#### 渐进式rehash

大字典的扩容是比较耗时间的，需要重新申请新的数组，然后将旧字典所有链表中的元素重新挂接到新的数组下面，这时一个O（n）级别的操作，单线程的Redis难以承受这个耗时的过程，的因此使用渐进式rehash小步迁移。

```c
dictEntry *dictAddRaw(dict *d, void *key, dictEntry **existing)
{
    long index;
    dictEntry *entry;
    dictht *ht;
    // 这里进行小步搬迁
    if (dictIsRehashing(d)) _dictRehashStep(d);
    /* Get the index of the new element, or -1 if
    * the element already exists. */
    if ((index = _dictKeyIndex(d, key, dictHashKey(d,key), existing)) == -1)
    	return NULL;
    /* Allocate the memory and store the new entry.
    * Insert the element in top, with the assumption that in a database
    * system it is more likely that recently added entries are accessed
    * more frequently. */
    // 如果字典处于搬迁过程中，要将新的元素挂接到新的数组下面
    ht = dictIsRehashing(d) ? &d->ht[1] : &d->ht[0];
    entry = zmalloc(sizeof(*entry));
    entry->next = ht->table[index];

    ht->table[index] = entry;
    ht->used++;
    /* Set the hash entry fields. */
    dictSetKey(d, entry, key);
    return entry;
}
```

搬迁操作埋伏在当前字典的后续指令中（来自客户端的hset/hdel指令等），但是有可能客户端闲下来了，没有了后续指令来触发这个搬迁，那么Redis就置之不理了么？Redis还会再定时任务中对字典进行主动搬迁。

```c
// 服务器定时任务
void databaseCron() {
...
	if (server.activerehashing) {
		for (j = 0; j < dbs_per_call; j++) {
			int work_done = incrementallyRehash(rehash_db);
			if (work_done) {
                /* If the function did some work, stop here, we'll do
                * more at the next cron loop. */
                break;
			} else {
                /* If this db didn't need rehash, we'll try the next one. */
                rehash_db++;
                rehash_db %= server.dbnum;
			}
		}
	}
}
```

#### 查找过程

插入和删除操作都依赖于查找，必须先把元素找到，才可以进行数据结构的修改操作。hashtable的元素是在第二维的链表上，所以首先我们得想办法定位出元素在哪个链表上。

```c
func get(key) {
	let index = hash_func(key) % size;
	let entry = table[index];
	while(entry != NULL) {
		if entry.key == target {
			return entry.value;
		}
		entry = entry.next;
	}
}
```

值得注意得是代码中的==hash_func==，它会将key映射为一个整数，不同的key会被映射成分布比较均匀散乱的整数。只有hash值均匀了，整个hashtable才是平衡的，所有二维链表的长度就不会差距很远，查找算法的性能也就比较稳定。

#### hash函数

hashtable的性能好不好完全取决于hash函数的质量。hash函数如果可以将key打散的比较均匀，那么这个hash函数就是好函数。Redis的字典默认的hash函数是siphash。siphash算法即使在输入key很小的情况下，也可以产生随机性特征好的输出，而且它的性能也非常突出。

#### hash攻击

如果hash函数存在偏向性，黑客就可能利用这种偏向性对服务器进行攻击。存在偏向性的hash函数在特定模式下的输入会导致hash第二维链表的长度极为不均匀，甚至所有的元素都集中到个别链表中，直接导致查询效率下降，从==O(1)==退化为==O(n)==。有限的服务器计算能力将会被hashtable的查找效率彻底拖垮。这就是所谓的hash攻击。

#### 扩容条件

```c
/* Expand the hash table if needed */
static int _dictExpandIfNeeded(dict *d)
{
    /* Incremental rehashing already in progress. Return. */
    if (dictIsRehashing(d)) return DICT_OK;
    /* If the hash table is empty expand it to the initial size. */
    if (d->ht[0].size == 0) return dictExpand(d, DICT_HT_INITIAL_SIZE);
    /* If we reached the 1:1 ratio, and we are allowed to resize the hash
    * table (global setting) or we should avoid it but the ratio between
    * elements/buckets is over the "safe" threshold, we resize doubling
    * the number of buckets. */
    if (d->ht[0].used >= d->ht[0].size && (dict_can_resize || d->ht[0].used/d->ht[0].size > dict_force_resize_ratio){
    	return dictExpand(d, d->ht[0].used*2);
    }
    return DICT_OK;
}
```

正常情况下，当hash表中元素的个数等于第一维数组的长度时，就会开始扩容，扩容的新数组是原数组大小的2倍。不过如果Redis正在做bgsave，为了减少内存页的过多分离（Copy On Write），Redis尽量不去扩容（==dict_can_resize==），但是如果hash表已经非常满了，元素的个数已经达到了第一维数组长度的5倍（==dict_force_resize_ratio==），说明hash表已经过于拥挤了，这个时候就会强制扩容。

#### 缩容条件

```c
int htNeedsResize(dict *dict) {
    long long size, used;
    size = dictSlots(dict);
    used = dictSize(dict);
    return (size > DICT_HT_INITIAL_SIZE && (used*100/size < HASHTABLE_MIN_FILL));
}
```

当hash表因为元素的逐渐删除变得越来越稀疏时，Redis会对hash表进行缩容来减少hash表的第一维数组占用空间。缩容的条件是元素个数低于数组长度的10％。缩容不会考虑Redis是否在做bgsave。

> BGSAVE
>
> 在后台异步(Asynchronously)保存当前数据库的数据到磁盘。
>
> BGSAVE命令执行之后立即返回`OK`，然后Redis fork出一个新子进程，原来的Redis进程（父进程）继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

#### set的结构

Redis里面set的结构底层也是字典，只不过所有的value都是NULL，其他的特性和字典一摸一样。