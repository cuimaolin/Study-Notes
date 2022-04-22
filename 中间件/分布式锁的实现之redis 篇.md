#### 一、引言

我们在系统中修改已有数据时，需要先读取，然后进行修改保存，此时很容易遇到并发问题。由于修改和保存不是原子操作，在并发场景下，部分对数据的操作可能会丢失。在单服务器系统我们常用本地锁来避免并发带来的问题，然后，当服务采用集群方式部署时，本地锁无法在多个服务器之间剩下，这时候保证数据的一致性就需要分布式锁来实现。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-01.png)

#### 二、实现

Redis锁主要利用Redis的setnx命令。

- 加锁命令：`SETNX key value`，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY是锁的唯一标识，一般按业务来决定命名；
- 解锁命令：`DEL key`，通过删除键值对释放锁，以便其他线程可以通过SETNX命令来获取锁；
- 锁超时：`EXPIRE key timeout`，设置key的超时时间，以保证即使锁没有被显式释放，锁也可以在第一时间后自动释放，避免资源用于被锁住。

则加锁解锁伪代码如下

```lua
if (setnx(key, 1) == 1){
    expire(key, 30)
    try {
        //TODO 业务逻辑
    } finally {
        del(key)
    }
}
```

##### 1. SETNX和EXPIRE非原子性

如果SETNX成功，在设置锁超时时间后，服务器挂掉、重启或网络问题等，导致EXPIRE命令没有执行，锁没有设置超时时间变成死锁。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-02.png)

```lua
if (redis.call('setnx', KEYS[1], ARGV[1]) < 1)
then return 0;
end;
redis.call('expire', KEYS[1], tonumber(ARGV[2]));
return 1;

// 使用实例
EVAL "if (redis.call('setnx',KEYS[1],ARGV[1]) < 1) then return 0; end; redis.call('expire',KEYS[1],tonumber(ARGV[2])); return 1;" 1 key value 100
```

##### 2. 锁误接触

如果线程A成功获取得到了锁，并设置了过期时间30秒，但是线程A执行时间超过了30秒，锁过期自动释放，此时线程B获取到了锁；随后A执行完成，线程A使用DEL命令来释放锁，但此时线程B加的锁还没有执行完成，线程A实际释放的是线程B加的锁。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-03.png)

通过在value中设置当前线程加锁的标识，在删除之前验证key对应的value判断锁释放是当前线程持有。可生成一个UUID标识当前线程，使用lua脚本做验证标识和解锁操作

```lua
/ 加锁
String uuid = UUID.randomUUID().toString().replaceAll("-","");
SET key uuid NX EX 30
// 解锁
if (redis.call('get', KEYS[1]) == ARGV[1])
    then return redis.call('del', KEYS[1])
else return 0
end
```

##### 3. 超时解锁导致并发

如果线程A成功获取锁并设置过期时间30s，但线程A执行时间超过了30秒，锁过期自动释放，此时线程B获取到了锁，线程A和线程B并发执行。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-04.png)

A、B两个线程发生并发线然是不被允许的，一般有两种方法解决该问题：

- 将过期时间设置足够长，确保代码逻辑在锁释放之前能够执行完成；
- 为获取锁的线程增加守护线程，为将要过期但未释放的锁增加有效时间。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-05.png)

##### 4. 不可重入

当线程在持有锁的情况下在此请求加锁，如果一个锁支持一个线程多次加锁，那么这个锁就是可重入的，如果一个不可重入锁被再次加锁，由于该锁已经被持有，再次加锁会失败。Redis可通过对锁进行重入计数，加锁时加1，解锁时减1，当计数归0时释放锁：

```java
private static ThreadLocal<Map<String, Integer>> LOCKERS = ThreadLocal.withInitial(HashMap::new);
// 加锁
public boolean lock(String key) {
  Map<String, Integer> lockers = LOCKERS.get();
  if (lockers.containsKey(key)) {
    lockers.put(key, lockers.get(key) + 1);
    return true;
  } else {
    if (SET key uuid NX EX 30) {
      lockers.put(key, 1);
      return true;
    }
  }
  return false;
}
// 解锁
public void unlock(String key) {
  Map<String, Integer> lockers = LOCKERS.get();
  if (lockers.getOrDefault(key, 0) <= 1) {
    lockers.remove(key);
    DEL key
  } else {
    lockers.put(key, lockers.get(key) - 1);
  }
}
```

本地记录重入次数虽然高效，但如果考虑到过期时间和本地、Redis一致性的问题，就会增加代码的复杂性。另一种方式是Redis Map数据结构来实现分布式锁，既存锁也对重入次数进行计数：

```lua
// 如果 lock_key 不存在
if (redis.call('exists', KEYS[1]) == 0)
then
    // 设置 lock_key 线程标识 1 进行加锁
    redis.call('hset', KEYS[1], ARGV[2], 1);
    // 设置过期时间
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
    end;
// 如果 lock_key 存在且线程标识是当前欲加锁的线程标识
if (redis.call('hexists', KEYS[1], ARGV[2]) == 1)
    // 自增
    then redis.call('hincrby', KEYS[1], ARGV[2], 1);
    // 重置过期时间
    redis.call('pexpire', KEYS[1], ARGV[1]);
    return nil;
    end;
// 如果加锁失败，返回锁剩余时间
return redis.call('pttl', KEYS[1]);
```

##### 5. 无法等待锁释放

上述命令执行都是立即返回的，如果客户端可以等待锁释放就无法使用：

- 可以通过客户端轮询的方式解决该问题，当未获取到锁时，等待一段时间重新获取锁，直到成功获取锁或等待超时。这种方式比较消耗服务器资源，并且并发量较大时，会影响服务器效率；
- 另一种方式是使用Redis的发布订阅功能，当获取锁失败时，订阅锁释放消息，获取锁成功后释放时，发送锁释放消息：

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-06.png)

#### 三、集群

##### 1. 主备切换

为了保证Redis的可用性，一般采用主从方式部署。主从数据同步有异步和同步两种方式，Redis将指令记录在本地内存buffer中，然后异步将buffer中的指令同步到从节点中，从节点一边执行同步的指令流来达到和主节点一致的状态，一边向主节点反馈同步情况。

在包含主从模式的集群部署方式中，当主节点挂掉时，从节点会取而代之，但客户端无明显感知。当客户端A成功加锁，指令还未同步，此时主节点挂掉，从节点提升为主节点，新的主节点没有锁的数据，当客户端B加锁时就会成功。

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-07.png)

##### 2. 集群脑裂

集群脑裂指因为网络问题，导致Redis master节点跟slave节点和sentinel集群处于不同的网络分区，因为sentinel集群无法感知到master的存在，所以将slave节点提升为master节点，此时存在两个不同的master节点。Redis Cluster集群部署方式同理。

当不同的客户端连接不同的master节点时，两个客户端就可以同时拥有一把锁。如下：

![img](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/redis-lock-08.png)

