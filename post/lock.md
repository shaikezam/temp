# Lock Interface
*01-12-2021 - Shai Zambrovski*

------------
The `Lock` interface (unlike the `synchronized` mechanism), provides us an implementations to handle in more sophisticated way with concurrency issues.

The Lock interface has several methods:
- `void lock()` - Acquires the lock, if the lock is not available then the current thread becomes disabled for thread scheduling purposes and lies dormant until the lock has been acquired.
- `void lockInterruptibly() throws InterruptedExceptionlockInterruptibly()`- This method acquires the lock if the block is free while allowing for the thread to be interrupted by some other thread while acquiring the lock and return immediately without acquiring the lock.
- `boolean tryLock() `- Acquires the lock if it is available and returns immediately with the value `true`, If the lock is not available then this method will return immediately with the value `false`.
- `boolean tryLock(long time, TimeUnit unit)` throws InterruptedException -  Same as above and waits for a certain time period as defined by arguments.
- `void unlock() `- Releases the lock.

There are some key differences between the `Lock` interface and the `synchronized` keyword:
- It is possible to set timeout while wating the Lock to be release
- It is not possible to interrupt the `synchronized` block, while using the `lockInterruptibly` can interrupt the current worker thread.
- `synchronized` block must be handled in single method, while `Lock\Unlock` can be handled in seperate methods.

We always unlocked the lock object to avoid deadlocks.

A recommended way to achieve it; call to `lock` in the `try` clause and call to `unlock` in the `finally` clause.
## `Lock` implementations
### ReentrantLock
`Lock` Implementations that's provides synchronization to methods while accessing shared resources.

As the name suggest, `ReentrantLock` can enter the shared resources more then once.

every time it lock, an internal counter increament by 1 and each time it unlock, the counter will decreace by 1.

Once the internal counter set to 0 (by calling to unlock), the shared reasorce is free to be locked by any other thread.
```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class SynchronizedHashMap {

    private static SynchronizedHashMap singleSynchronizedHashMap = null;

    private Map<String, String> map = new HashMap<>();
    private Lock lock = new ReentrantLock();

    private SynchronizedHashMap() {
    }

    public static SynchronizedHashMap getInstance() {
        if (singleSynchronizedHashMap == null) {
            synchronized (SynchronizedHashMap.class) {
                singleSynchronizedHashMap = new SynchronizedHashMap();
            }
        }
        return singleSynchronizedHashMap;
    }

    public String get(String key) {
        try {
            lock.lock();
            return map.get(key);
        } finally {
            lock.unlock();
        }
    }

    public void put(String key, String value) {
        try {
            lock.lock();
            map.put(key, value);
        } finally {
            lock.unlock();
        }
    }

    public String remove(String key) {
        try {
            lock.lock();
            return map.remove(key);
        } finally {
            lock.unlock();
        }
    }

    public boolean containsKey(String key) {
        try {
            lock.lock();
            return map.containsKey(key);
        } finally {
            lock.unlock();
        }
    }
}
```
There are some drawbacks regarding this scenario.

The read oporations doesn't need to be thread safe (unless we write in the same time), that can cause some performance issues.
### ReentrantReadWriteLock
ReentrantReadWriteLock implements the ReadWriteLock interface with the methods:
- `Lock readLock()` - Returns the lock used for reading.
- `Lock writeLock()` - Returns the lock used for writing.

A `ReadWriteLock` maintains a pair of associated locks, one for read-only operations and one for writing.

The `readLock` may be held simultaneously by multiple reader threads, so long as there are no writers.

The `writeLock` is exclusive.
```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class SynchronizedHashMap {

    private static SynchronizedHashMap singleSynchronizedHashMap = null;

    private Map<String, String> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private Lock readLock = readWriteLock.readLock();
    private Lock writeLock = readWriteLock.writeLock();

    private SynchronizedHashMap() {
    }

    public static SynchronizedHashMap getInstance() {
        if (singleSynchronizedHashMap == null) {
            synchronized (SynchronizedHashMap.class) {
                singleSynchronizedHashMap = new SynchronizedHashMap();
            }
        }
        return singleSynchronizedHashMap;
    }

    public String get(String key) {
        try {
            readLock.lock();
            return map.get(key);
        } finally {
            readLock.unlock();
        }
    }

    public void put(String key, String value) {
        try {
            writeLock.lock();
            map.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }

    public String remove(String key) {
        try {
            writeLock.lock();
            return map.remove(key);
        } finally {
            writeLock.unlock();
        }
    }

    public boolean containsKey(String key) {
        try {
            readLock.lock();
            return map.containsKey(key);
        } finally {
            readLock.unlock();
        }
    }
}
```