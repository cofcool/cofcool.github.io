# List的sort函数实现解析

最近在使用List的`void sort(Comparator<? super E> c)`函数时在想了解内部是如何实现的，于是看了源码，该默认方法在JDK1.8的时候添加，排序算法采用TimSort算法，如下所述。

```
The implementation was adapted from Tim Peters's list sort for Python
 (<a href="http://svn.python.org/projects/python/trunk/Objects/listsort.txt">
 TimSort</a>).  It uses techniques from Peter McIlroy's "Optimistic
 Sorting and Information Theoretic Complexity", in Proceedings of the
 Fourth Annual ACM-SIAM Symposium on Discrete Algorithms, pp 467-474,
 January 1993.
```

ArrayList的实现与List接口的默认实现稍有不同，如果排序时修改了对象内部元素的值，ArrayList的sort方法会抛出`ConcurrentModificationException`异常。

## 1. TimSort算法介绍



## 2. 源码解析

**List**:
```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();

    // 在此处调用Arrays的sort方法
    // 该方法要求传入一个数组和Comparator对象
    Arrays.sort(a, (Comparator) c);

    // 把排序好的数组回写到list对象中
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```
Arrays
```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        // 未传入Comparator对象时调用
        // 该sort函数调用ComparableTimSort.sort(a, 0, a.length, null, 0, 0)来实现排序，与TimSort的实现类似
        sort(a);
    } else {
        // 传统的排序算法，此处是为了兼容旧版本JDK，在未来会移除
        // 该排序实现：
        // 1. 数组长度小于7时，采用冒泡排序
        // 2. 长度大于7的数组用归并排序
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            // TimSort算法
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}
```

TimSort
```java
static <T> void sort(T[] a, int lo, int hi, Comparator<? super T> c,
                     T[] work, int workBase, int workLen) {
    assert c != null && a != null && lo >= 0 && lo <= hi && hi <= a.length;

    int nRemaining  = hi - lo;
    if (nRemaining < 2)
        return;  // Arrays of size 0 and 1 are always sorted

    // If array is small, do a "mini-TimSort" with no merges
    if (nRemaining < MIN_MERGE) {
        int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
        binarySort(a, lo, hi, lo + initRunLen, c);
        return;
    }

    /**
     * March over the array once, left to right, finding natural runs,
     * extending short natural runs to minRun elements, and merging runs
     * to maintain stack invariant.
     */
    TimSort<T> ts = new TimSort<>(a, c, work, workBase, workLen);
    int minRun = minRunLength(nRemaining);
    do {
        // Identify next run
        int runLen = countRunAndMakeAscending(a, lo, hi, c);

        // If run is short, extend to min(minRun, nRemaining)
        if (runLen < minRun) {
            int force = nRemaining <= minRun ? nRemaining : minRun;
            binarySort(a, lo, lo + force, lo + runLen, c);
            runLen = force;
        }

        // Push run onto pending-run stack, and maybe merge
        ts.pushRun(lo, runLen);
        ts.mergeCollapse();

        // Advance to find next run
        lo += runLen;
        nRemaining -= runLen;
    } while (nRemaining != 0);

    // Merge all remaining runs to complete sort
    assert lo == hi;
    ts.mergeForceCollapse();
    assert ts.stackSize == 1;
}

private static <T> void binarySort(T[] a, int lo, int hi, int start,
                                   Comparator<? super T> c) {
    assert lo <= start && start <= hi;
    if (start == lo)
        start++;
    for ( ; start < hi; start++) {
        T pivot = a[start];

        // Set left (and right) to the index where a[start] (pivot) belongs
        int left = lo;
        int right = start;
        assert left <= right;
        /*
         * Invariants:
         *   pivot >= all in [lo, left).
         *   pivot <  all in [right, start).
         */
        while (left < right) {
            int mid = (left + right) >>> 1;
            if (c.compare(pivot, a[mid]) < 0)
                right = mid;
            else
                left = mid + 1;
        }
        assert left == right;

        /*
         * The invariants still hold: pivot >= all in [lo, left) and
         * pivot < all in [left, start), so pivot belongs at left.  Note
         * that if there are elements equal to pivot, left points to the
         * first slot after them -- that's why this sort is stable.
         * Slide elements over to make room for pivot.
         */
        int n = start - left;  // The number of elements to move
        // Switch is just an optimization for arraycopy in default case
        switch (n) {
            case 2:  a[left + 2] = a[left + 1];
            case 1:  a[left + 1] = a[left];
                     break;
            default: System.arraycopy(a, left, a, left + 1, n);
        }
        a[left] = pivot;
    }
}

private static <T> int countRunAndMakeAscending(T[] a, int lo, int hi,
                                                Comparator<? super T> c) {
    assert lo < hi;
    int runHi = lo + 1;
    if (runHi == hi)
        return 1;

    // Find end of run, and reverse range if descending
    if (c.compare(a[runHi++], a[lo]) < 0) { // Descending
        while (runHi < hi && c.compare(a[runHi], a[runHi - 1]) < 0)
            runHi++;
        reverseRange(a, lo, runHi);
    } else {                              // Ascending
        while (runHi < hi && c.compare(a[runHi], a[runHi - 1]) >= 0)
            runHi++;
    }

    return runHi - lo;
}
```

## 引用

1. [维基百科上关于TimSort算法的介绍](https://en.wikipedia.org/wiki/Timsort)
