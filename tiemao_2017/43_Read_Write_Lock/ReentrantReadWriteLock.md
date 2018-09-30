```java
public class ReentrantReadWriteLock
extends Object
implements ReadWriteLock, Serializable
```



An implementation of [`ReadWriteLock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html) supporting similar semantics to [`ReentrantLock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html).

��ʵ��(`ReadWriteLock`)(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html)֧�����Ƶ�����(`ReentrantLock`(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html)��]

This class has the following properties:

���������������:

- **Acquisition order**

- * * * *����order

This class does not impose a reader or writer preference ordering for lock access. However, it does support an optional *fairness* policy.

����಻ǿ�Ӹ����߻�����ƫ�����������ʡ�Ȼ��,��֧��һ����ѡ��* *��ƽ���ߡ�

- **Non-fair mode (default)**

- * * * *�ǿ�ģʽ(Ĭ��)

When constructed as non-fair (the default), the order of entry to the read and write lock is unspecified, subject to reentrancy constraints. A nonfair lock that is continuously contended may indefinitely postpone one or more reader or writer threads, but will normally have higher throughput than a fair lock.

�������(Ĭ��),�����˳���д������,��������Լ��.nonfair���������ƿ����������Ƴ�һ���������߻������߳�,��ͨ�����и��ߵ��������ȹ�ƽ����

- **Fair mode**

- * * * *��ƽ��ʽ

When constructed as fair, threads contend for entry using an approximately arrival-order policy. When the currently held lock is released, either the longest-waiting single writer thread will be assigned the write lock, or if there is a group of reader threads waiting longer than all waiting writer threads, that group will be assigned the read lock.

��������ƽ���߳�����Ŀʹ�ô�Լarrival-order����.Ŀǰ�ͷ���ʱ,Ҫô������Ŷ�ʱ�䵥д�߳̽���ָ��д��,���������һȺ���̵߳ȴ�ʱ������еȴ�д�߳�,���Ⱥ�彫��ָ�ɶ�����

A thread that tries to acquire a fair read lock (non-reentrantly) will block if either the write lock is held, or there is a waiting writer thread. The thread will not acquire the read lock until after the oldest currently waiting writer thread has acquired and released the write lock. Of course, if a waiting writer abandons its wait, leaving one or more reader threads as the longest waiters in the queue with the write lock free, then those readers will be assigned the read lock.

һ���߳���ͼ���һ����ƽ�Ķ���(��������)�������д��,����һ��д�̵߳ȴ�.�����ö������߳�,ֱ��Ŀǰ����ϵĵȴ�д�̻߳�ȡ���ͷ�д��.��Ȼ,���һ���ȴ����ҷ����ȴ�,�뿪һ���������̵߳Ķ�����ķ���Աд������,��ô��Щ���߽���ָ�ɶ�����

A thread that tries to acquire a fair write lock (non-reentrantly) will block unless both the read lock and write lock are free (which implies there are no waiting threads). (Note that the non-blocking [`ReentrantReadWriteLock.ReadLock.tryLock()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.ReadLock.html#tryLock--) and [`ReentrantReadWriteLock.WriteLock.tryLock()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.WriteLock.html#tryLock--)methods do not honor this fair setting and will immediately acquire the lock if it is possible, regardless of waiting threads.)

һ���߳���ͼ���һ����ƽ��д��(��������)����ֹ,���Ƕ�����д��������ѵ�(����ζ��û�еȴ��߳�).(ע��,������`ReentrantReadWriteLock.ReadLock.tryLock()``ReentrantReadWriteLock.WriteLock.tryLock()`)(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.WriteLock.html # tryLock����)�������������ֹ�ƽ�����ý����������,������ǿ��ܵ�,���ܵȴ��߳�)��

- **Reentrancy**

- Reentrancy * * * *

This lock allows both readers and writers to reacquire read or write locks in the style of a [`ReentrantLock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html). Non-reentrant readers are not allowed until all write locks held by the writing thread have been released.

�����������ߺ��������»�ȡ��д���ķ��(`ReentrantLock`)(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html)����������Ķ����ǲ�����ֱ������дд�̳߳��е������ͷš�

Additionally, a writer can acquire the read lock, but not vice-versa. Among other applications, reentrancy can be useful when write locks are held during calls or callbacks to methods that perform reads under read locks. If a reader tries to acquire the write lock it will never succeed.

����,һ�����ҿ��Ի�ö���,��������Ȼ..���һ��������ͼ��ȡд��������Զ����ɹ���

- **Lock downgrading**

- * * * *������

Reentrancy also allows downgrading from the write lock to a read lock, by acquiring the write lock, then the read lock and then releasing the write lock. However, upgrading from a read lock to the write lock is **not** possible.

�������Ի������д����������,ͨ���չ�д��,Ȼ�����,Ȼ���ͷ�д��.Ȼ��,�Ӷ�������д��* *��* *�ǿ��ܵġ�

- **Interruption of lock acquisition**

- * * * * lock of�����ж�

The read lock and write lock both support interruption during lock acquisition.

������д������ȡ�ڼ䶼֧���жϡ�

- **Condition support**

- * * * *��λ֧��

The write lock provides a [`Condition`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html) implementation that behaves in the same way, with respect to the write lock, as the [`Condition`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html) implementation provided by[`ReentrantLock.newCondition()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html#newCondition--) does for [`ReentrantLock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html). This [`Condition`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html) can, of course, only be used with the write lock.

д���ṩ��(`Condition`)(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html)ʵ�ֵ���Ϊͬ��,����д��,(`Condition`https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html](ִ��)��[`ReentrantLock.newCondition()`Ϊ[](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html newCondition����)`ReentrantLock`(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html)��]��һ[`Condition`)(https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html),��Ȼ,ֻ����д����

The read lock does not support a [`Condition`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html) and `readLock().newCondition()` throws `UnsupportedOperationException`.

������֧��(`Condition`https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html](��)`readLock().newCondition()`�׳�`UnsupportedOperationException`��

- **Instrumentation**

- * * * *������

This class supports methods to determine whether locks are held or contended. These methods are designed for monitoring system state, not for synchronization control.

����֧�ַ�����ȷ���Ƿ������á���Щ�������ڼ��ϵͳ״̬,����������ͬ�����ơ�

Serialization of this class behaves in the same way as built-in locks: a deserialized lock is in the unlocked state, regardless of its state when serialized.

���л�������Ϊ��ͬ���ķ�ʽ��Ϊ������:�����л������ڽ���״̬,�������л�ʱ��״̬��

**Sample usages**. Here is a code sketch showing how to perform lock downgrading after updating a cache (exception handling is particularly tricky when handling multiple locks in a non-nested fashion):

��λ��;* * * *.����һ�������ͼ��ʾ���ִ������������»���(�쳣����ʱ��Ϊ���ֵĴ�������non-nested��ʽ):

```java
 class CachedData {
   Object data;
   volatile boolean cacheValid;
   final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

   void processCachedData() {
     rwl.readLock().lock();
     if (!cacheValid) {
       // Must release read lock before acquiring write lock
       rwl.readLock().unlock();
       rwl.writeLock().lock();
       try {
         // Recheck state because another thread might have
         // acquired write lock and changed state before we did.
         if (!cacheValid) {
           data = ...
           cacheValid = true;
         }
         // Downgrade by acquiring read lock before releasing write lock
         rwl.readLock().lock();
       } finally {
         rwl.writeLock().unlock(); // Unlock write, still hold read
       }
     }

     try {
       use(data);
     } finally {
       rwl.readLock().unlock();
     }
   }
 }
```



ReentrantReadWriteLocks can be used to improve concurrency in some uses of some kinds of Collections. This is typically worthwhile only when the collections are expected to be large, accessed by more reader threads than writer threads, and entail operations with overhead that outweighs synchronization overhead. For example, here is a class using a TreeMap that is expected to be large and concurrently accessed.

ReentrantReadWriteLocks�����������Ʋ�������ĳЩʹ��һЩ���͵ļ���.��ͨ�����м�ֵ��ֻ�е�Ԥ�ƽ��󼯺�,�ɱ����Ҹ����̵߳��̷߳���,����Ҫ�����Ŀ�������ͬ������������,����һ����ʹ��TreeMap�����󲢷����ʡ�

```java
 class RWDictionary {
   private final Map<String, Data> m = new TreeMap<String, Data>();
   private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
   private final Lock r = rwl.readLock();
   private final Lock w = rwl.writeLock();

   public Data get(String key) {
     r.lock();
     try { return m.get(key); }
     finally { r.unlock(); }
   }
   public String[] allKeys() {
     r.lock();
     try { return m.keySet().toArray(); }
     finally { r.unlock(); }
   }
   public Data put(String key, Data value) {
     w.lock();
     try { return m.put(key, value); }
     finally { w.unlock(); }
   }
   public void clear() {
     w.lock();
     try { m.clear(); }
     finally { w.unlock(); }
   }
 }
```



### Implementation Notes

### ʵ��ע������

This lock supports a maximum of 65535 recursive write locks and 65535 read locks. Attempts to exceed these limits result in [`Error`](https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html) throws from locking methods.

�������֧��65535�ݹ�д65535�����Ͷ�������ͼ��Խ��Щ���Ƶ���(`Error`)(https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html)���������ķ�����

- Since:

- ��:

1.5

1.5

<https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.html>



