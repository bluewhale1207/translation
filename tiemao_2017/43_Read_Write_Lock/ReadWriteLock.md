```java
public interface ReadWriteLock
```



A `ReadWriteLock` maintains a pair of associated [`locks`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html), one for read-only operations and one for writing. The [`read lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#readLock--) may be held simultaneously by multiple reader threads, so long as there are no writers. The [`write lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#writeLock--) is exclusive.

��д��(`ReadWriteLock`)������һ�����������([`locks`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html)), һ������ֻ������, һ������д������[`read lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#readLock--) �������߳�ͬʱ����, ��Ȼ, ������д������ͬʱ���档 [`write lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html#writeLock--) ֻ�������̶߳�ռ(exclusive)��

All `ReadWriteLock` implementations must guarantee that the memory synchronization effects of `writeLock` operations (as specified in the [`Lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html) interface) also hold with respect to the associated `readLock`. That is, a thread successfully acquiring the read lock will see all updates made upon previous release of the write lock.

`ReadWriteLock` ��ʵ������뱣֤ `writeLock`�������ڴ�ͬ����Ч��(�μ� [`Lock`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Lock.html) �ӿ�˵���ĵ�), ͬʱҲ�������� `readLock` ��صĹ淶�� Ҳ����˵, ĳ���̻߳�ȡ��������ʱ��, �����ܿ���ǰ���д���߳������ĸ��¡�

A read-write lock allows for a greater level of concurrency in accessing shared data than that permitted by a mutual exclusion lock. It exploits the fact that while only a single thread at a time (a *writer* thread) can modify the shared data, in many cases any number of threads can concurrently read the data (hence *reader* threads). In theory, the increase in concurrency permitted by the use of a read-write lock will lead to performance improvements over the use of a mutual exclusion lock. In practice this increase in concurrency will only be fully realized on a multi-processor, and then only if the access patterns for the shared data are suitable.

��д������Ĳ������ʹ������ݴ�������Ļ�����.�����õ���ʵ��:һ��ֻ��һ���߳�(*����*�߳�)�����޸Ĺ�������,���������������������߳̿���ͬʱ��ȡ����(���*����*�߳�).������,�����Ե����������ʹ�ö�д���ᵼ�����ܸĽ���ʹ�û�����.��ʵ�������ֲ����Ե�����ֻ����ȫʵ���ڶദ����,Ȼ��ֻ���ڹ������ݵķ���ģʽ�Ǻ��ʵġ�

Whether or not a read-write lock will improve performance over the use of a mutual exclusion lock depends on the frequency that the data is read compared to being modified, the duration of the read and write operations, and the contention for the data - that is, the number of threads that will try to read or write the data at the same time. For example, a collection that is initially populated with data and thereafter infrequently modified, while being frequently searched (such as a directory of some kind) is an ideal candidate for the use of a read-write lock. However, if updates become frequent then the data spends most of its time being exclusively locked and there is little, if any increase in concurrency.

lock read-write�ڲ�ͬ����̳��ʹ��ҵ��over���໥�ų�,�ɼ�lock��on the read isĳЩ���ݵĵ���֮��ת����,����д�����ĳ���ʱ��,�������ݡ���Ҳ����˵,�̵߳�����,�����Զ���д������ͬһʱ��.����,һ������������,�˺󾭳��޸�,��Ȼ����������(��ĳ��Ŀ¼)��һ������ĺ�ѡ��ʹ�ö�д��.Ȼ��,������±��Ƶ��,���ݻ��ѵĴ󲿷�ʱ�䱻��ȫ����,����û��,��������Ӳ����ԡ�

Further, if the read operations are too short the overhead of the read-write lock implementation (which is inherently more complex than a mutual exclusion lock) can dominate the execution cost, particularly as many read-write lock implementations still serialize all threads through a small section of code. Ultimately, only profiling and measurement will establish whether the use of a read-write lock is suitable for your application.

��һ��,�����ȡ����̫�̶�д��ʵ�ֵĿ���(�������Ǹ����ӵı�һ��������)��������ִ�гɱ�,�ر�������д����ʵ����Ȼ���л������߳�ͨ��һС�δ���.����,ֻ�з����Ͳ���������ʹ�ö�д���Ƿ��ʺ�����Ӧ�ó���

Although the basic operation of a read-write lock is straight-forward, there are many policy decisions that an implementation must make, which may affect the effectiveness of the read-write lock in a given application. Examples of these policies include:

��Ȼ��д���Ļ���������ֱ�ӵ�,�кܶ�����ʵ�ֱ��������ľ���,����Ӱ�����Ч�Զ�д����һ��������Ӧ�ó�����Щ���ߵ����Ӱ���:

- Determining whether to grant the read lock or the write lock, when both readers and writers are waiting, at the time that a writer releases the write lock. Writer preference is common, as writes are expected to be short and infrequent. Reader preference is less common as it can lead to lengthy delays for a write if the readers are frequent and long-lived as expected. Fair, or "in-order" implementations are also possible.
- Determining whether readers that request the read lock while a reader is active and a writer is waiting, are granted the read lock. Preference to the reader can delay the writer indefinitely, while preference to the writer can reduce the potential for concurrency.
- Determining whether the locks are reentrant: can a thread with the write lock reacquire it? Can it acquire a read lock while holding the write lock? Is the read lock itself reentrant?
- Can the write lock be downgraded to a read lock without allowing an intervening writer? Can a read lock be upgraded to a write lock, in preference to other waiting readers or writers?

- �����Ƿ����������д��,�����ߺ����Ҷ��ǵȴ�,��ʱһ�������ͷ�д��.����ƫ���Ǻܳ�����,��ΪдԤ�ƽ��̺ͺ�����.����ƫ�ò�����,��Ϊ���ᵼ�³�ʱ����ӳ�д�������Ƶ���ͳ��١���ƽ��,��˳��Ҳ�п���ʵ�֡�
- ���������Ƿ������ȡ������Ծ�Ķ��ߺ������ǵȴ�,��ö���.���ߵ�ƫ�ÿ������������ӵ�����,��ƫ�����߿��Լ���Ǳ�ڵĲ����ԡ�
- ȷ���Ƿ��������:����д�����߳����»�ȡ��?���ܻ�ö���,ͬʱ����д����?���������������?
- Can the��downgraded to a lock be)ÿ��733����lock read��Ʒ?lock upgraded be read a Can,���϶���lock readers��ʵ����Ѹ�ٽ��лƽ�����?

You should consider all of these things when evaluating the suitability of a given implementation for your application.

���ж�ĳ�ֶ�д��ʵ���Ƿ�����ϵͳ����ʱ, Ӧ���ۺϿ������ϼ��㡣

- Since: 1.5

- �˽ӿ��� JDK 1.5 ����


<https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReadWriteLock.html>



