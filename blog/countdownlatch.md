# CountDownLatch usages
*22-11-2021 - Shai Zambrovski*

------------
Basically a counter which validate all workers `threads` are done and the main `thread` who created those, are no longer block and can continue.

Once we create a `CountDownLatch` with the number of the parallel `threads` that will work together we will call to `countdown()` upon finish the task will decrease the number of remaining threads.
## Wait for all threads to complete
We will create simple application that upon each worker `thread` we will print to console and when all threads are done, we will notify it.
```java
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class Main {

    private static final int THREADS_LIMIT = 10;

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(THREADS_LIMIT);

        List<Thread> threads = Stream
                .generate(() ->
                        new Thread(() -> {
                            System.out.println("in worker thread");
                            countDownLatch.countDown();
                        }))
                .limit(THREADS_LIMIT)
                .collect(Collectors.toList());

        threads.forEach(Thread::start);

        countDownLatch.await();
        System.out.println("finished");
    }
}
```
We will get the next output:

    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    in worker thread
    finished

## `awaite` with timeout
In some scenarios, we would like to terminate the `CountDownLatch` without wating for all `threads` to finish and to avoid blocking the caller `thread` forever.

To do so, we can use the `await(long timeout, TimeUnit unit)` function.
```java
public class Main {

    private static final int THREADS_LIMIT = 10;

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(THREADS_LIMIT);

        List<Thread> threads = Stream
                .generate(() ->
                        new Thread(() -> {
                            if(true) {
                                throw new RuntimeException("An Error");
                            } else {
                                countDownLatch.countDown();
                            }
                        }))
                .limit(THREADS_LIMIT)
                .collect(Collectors.toList());

        threads.forEach(Thread::start);

        countDownLatch.await(5L, TimeUnit.SECONDS);
        System.out.println("finished");
    }
}
```

