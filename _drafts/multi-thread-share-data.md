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

**Phaser**
```java
public class FileSearch implements Runnable {

    private String initPath;
    private String end;
    private List<String> results;

    private Phaser phaser;

    public FileSearch(String initPath, String end, Phaser phaser) {
        this.initPath = initPath;
        this.end = end;
        this.phaser = phaser;

        results = new ArrayList<>();
    }

    public static void main(String[] args) {
        Phaser phaser = new Phaser(3);

        FileSearch system = new FileSearch("/home/yi", "log", phaser);
        FileSearch app = new FileSearch("/home/yi/bin", "log", phaser);
        FileSearch documents = new FileSearch("/home/yi/app/doc", "log", phaser);

        Thread sys = new Thread(system, "System");
        sys.start();

        Thread ap = new Thread(app, "App");
        ap.start();

        Thread doc = new Thread(documents, "Doc");
        doc.start();

        try {
            sys.join();
            ap.join();
            doc.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Terminated: " + phaser.isTerminated());
    }


    private void directoryProcess(File file) {
        File[] list = file.listFiles();
        if (list != null) {
            for (int i = 0; i < list.length; i++) {
                if (list[i].isDirectory()) {
                    directoryProcess(list[i]);
                } else {
                    fileProcess(list[i]);
                }
            }
        }
    }

    private void fileProcess(File file) {
        if (file.getName().endsWith(end)) {
            results.add(file.getAbsolutePath());
        }
    }

    private void filterResults() {
        results = results
                .stream()
                .filter(filePath ->
                    new File(filePath).lastModified() - (new Date().getTime()) < TimeUnit.MILLISECONDS.convert(1, TimeUnit.DAYS)
                )
                .collect(Collectors.toList());
    }


    private boolean checkResults() {
        if (results.isEmpty()) {
            System.out.printf("%s: Phase %d: 0 results.\n", Thread.currentThread().getName(), phaser.getPhase());
            System.out.printf("%s: Phase %d: end.\n", Thread.currentThread().getName(), phaser.getPhase());

            phaser.arriveAndDeregister();
            return false;
        } else {
            phaser.arriveAndAwaitAdvance();
            return true;
        }
    }

    private void showinfo() {
        results.forEach(path -> {
            System.out.printf("%s: %s\n", Thread.currentThread().getName(), new File(path).getAbsolutePath());
        });
    }

    @Override
    public void run() {
        phaser.arriveAndAwaitAdvance();
        System.out.printf("%s: starting.\n", Thread.currentThread().getName());

        File file = new File(initPath);
        if (file.isDirectory()) {
            directoryProcess(file);
        }

        if (!checkResults()) {
            return;
        }

        filterResults();

        if (!checkResults()) {
            return;
        }

        showinfo();
        phaser.arriveAndDeregister();
        System.out.printf("%s work completed.\n", Thread.currentThread().getName());
    }
}
```
