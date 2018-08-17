# Java 多线程下的线程控制和数据共享

```java
static void semaphoreTest() {
    Semaphore semaphore = new Semaphore(2);

    ExecutorService service = Executors.newFixedThreadPool(4);
    for (int i = 0; i < 10; i++) {
        service.submit(() -> {
            while (!semaphore.tryAcquire()) {
                System.out.println(Thread.currentThread().getId() + " wait permit");
            }

            System.out.println(Thread.currentThread().getId() + " get permit");
            semaphore.release();
        });
    }
    try {
        service.awaitTermination(2, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    service.shutdown();
    System.out.println("done with " + semaphore.availablePermits() + " permit");

//        11 get permit
//        14 wait permit
//        11 get permit
//        12 get permit
//        13 wait permit
//        12 get permit
//        11 get permit
//        14 wait permit
//        11 get permit
//        12 get permit
//        13 wait permit
//        11 get permit
//        14 wait permit
//        13 get permit
//        14 get permit
//        done with 2 permit
}

static void exchangerTest() {
    Exchanger<Integer> exchanger = new Exchanger<>();

    new Thread(() -> {
        try {
            System.out.println("1: " + exchanger.exchange(10));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    new Thread(() -> {
        try {
            System.out.println("2: " +exchanger.exchange(5));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    System.out.println("done");

//        done
//        1: 5
//        2: 10
}

static void cyclicBarrierTest() {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> System.out.println("start barrier"));

    new Thread(() -> {
        try {
            cyclicBarrier.await();
            System.out.println("1 done");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }).start();
    new Thread(() -> {
        try {
            cyclicBarrier.await();
            System.out.println("2 done");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }).start();

    try {
        cyclicBarrier.await();
    } catch (InterruptedException | BrokenBarrierException e) {
        e.printStackTrace();
    }
    System.out.println(cyclicBarrier.getNumberWaiting());

//        start barrier
//        2 done
//        1 done
//        0
}

static void countDownLatchTest() {
    CountDownLatch countService = new CountDownLatch(2);
    new Thread(() -> {
        countService.countDown();
        System.out.println("1 done");
    }).start();
    new Thread(() -> {
        countService.countDown();
        System.out.println("2 done");
    }).start();

    try {
        countService.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    System.out.println(countService.getCount());

//        1 done
//        2 done
//        0
}
```
