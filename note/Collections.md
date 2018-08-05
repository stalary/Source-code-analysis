- [介绍](#%E4%BB%8B%E7%BB%8D)
- [sort](#sort)
- [arch](#%1Barch)
- [reverse](#reverse)
- [fill](#fill)
- [min](#min)
- [rotate](#rotate)
- [replaceAll](#replaceall)
### 介绍
```java
    /** 一些算法所用到的常量，主要是些阈值 **/
    private static final int BINARYSEARCH_THRESHOLD   = 5000;
    private static final int REVERSE_THRESHOLD        =   18;
    private static final int SHUFFLE_THRESHOLD        =    5;
    private static final int FILL_THRESHOLD           =   25;
    private static final int ROTATE_THRESHOLD         =  100;
    private static final int COPY_THRESHOLD           =   10;
    private static final int REPLACEALL_THRESHOLD     =   11;
    private static final int INDEXOFSUBLIST_THRESHOLD =   35;
```

### sort
```java
    // 排序，传入集合和比较器
    public static <T> void sort(List<T> list, Comparator<? super T> c) {
        // 直接调用list的排序方法
        list.sort(c);
    }
```

```java
    default void sort(Comparator<? super E> c) {
        // 将集合转化为数组
        Object[] a = this.toArray();
        // 调用Arrays的排序方法
        Arrays.sort(a, (Comparator) c);
        // 迭代添加排好序的元素
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```

```java
    public static <T> void sort(T[] a, Comparator<? super T> c) {
        // 比较器为null时，直接调用不使用迭代器的排序方法
        if (c == null) {
            sort(a);
        } else {
            // 兼容老版本，当用户设置时，调用老的排序
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a, c);
            else
                // 使用TimSort进行排序
                TimSort.sort(a, 0, a.length, c, null, 0, 0);
        }
    }
```

```java
    static <T> void sort(T[] a, int lo, int hi, Comparator<? super T> c,
                         T[] work, int workBase, int workLen) {
        assert c != null && a != null && lo >= 0 && lo <= hi && hi <= a.length;
        // 需要排序的元素
        int nRemaining  = hi - lo;
        // 当小于2时代表不需要排序
        if (nRemaining < 2)
            return;  // Arrays of size 0 and 1 are always sorted

        // If array is small, do a "mini-TimSort" with no merges
        // 当元素数量较小时(32)，使用mini-TimSort
        if (nRemaining < MIN_MERGE) {
            // 获取连续升序元素的个数
            int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
            // 二分插入排序，跳过升序元素进行排序
            binarySort(a, lo, hi, lo + initRunLen, c);
            return;
        }

        /**
         * March over the array once, left to right, finding natural runs,
         * extending short natural runs to minRun elements, and merging runs
         * to maintain stack invariant.
         */
        TimSort<T> ts = new TimSort<>(a, c, work, workBase, workLen);
        // 计算run的数量
        int minRun = minRunLength(nRemaining);
        do {
            // Identify next run
            // 获取连续递增最大数量
            int runLen = countRunAndMakeAscending(a, lo, hi, c);

            // If run is short, extend to min(minRun, nRemaining)
            // 当数量小于minRun时，先使用二分插入排序
            if (runLen < minRun) {
                int force = nRemaining <= minRun ? nRemaining : minRun;
                binarySort(a, lo, lo + force, lo + runLen, c);
                runLen = force;
            }

            // Push run onto pending-run stack, and maybe merge
            // 压入堆栈，等待合并
            ts.pushRun(lo, runLen);
            // 归并
            ts.mergeCollapse();

            // Advance to find next run
            lo += runLen;
            nRemaining -= runLen;
        } while (nRemaining != 0);

        // Merge all remaining runs to complete sort
        assert lo == hi;
        // 归并所有的run
        ts.mergeForceCollapse();
        assert ts.stackSize == 1;
    }
```

```java
    // 计算出最小的run的长度，数组大小为二次幂，则直接返回16，否则找到介于16-32之间的数
    private static int minRunLength(int n) {
        assert n >= 0;
        int r = 0;      // Becomes 1 if any 1 bits are shifted off
        while (n >= MIN_MERGE) {
            r |= (n & 1);
            n >>= 1;
        }
        return n + r;
    }
```

```java
    private static <T> void binarySort(T[] a, int lo, int hi, int start,
                                       Comparator<? super T> c) {
        assert lo <= start && start <= hi;
        if (start == lo)
            start++;
        for ( ; start < hi; start++) {
            // 选择起始元素为轴
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
             // 使用二分进行缩小排序范围
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
            // 移动元素
            switch (n) {
                case 2:  a[left + 2] = a[left + 1];
                case 1:  a[left + 1] = a[left];
                         break;
                default: System.arraycopy(a, left, a, left + 1, n);
            }
            // 将轴归位
            a[left] = pivot;
        }
    }
```

### arch
```java
    public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        // 当list为随机访问列表或者长度小于5000时，使用索引二叉搜索
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            // 否则使用迭代二叉搜索
            return Collections.iteratorBinarySearch(list, key);
    }
```

```java
    // 索引二叉搜索
    private static <T> int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            // 获取中间值
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = list.get(mid);
            // 指定元素与中间值的大小比较
            int cmp = midVal.compareTo(key);
            // 更新上界和下界，缩小查找范围
            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        // 当为查找到时，返回负的最小下标+1
        return -(low + 1);  // key not found
    }
```

```java
    // 迭代器二叉搜索
    private static <T> int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
    {
        int low = 0;
        int high = list.size()-1;
        // 与索引搜索不同，使用迭代器
        ListIterator<? extends Comparable<? super T>> i = list.listIterator();

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = get(i, mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```

### reverse
```java
    public static void reverse(List<?> list) {
        int size = list.size();
        // 当数量小于18或者为随机访问列表时，直接对头尾依次交换
        if (size < REVERSE_THRESHOLD || list instanceof RandomAccess) {
            for (int i=0, mid=size>>1, j=size-1; i<mid; i++, j--)
                swap(list, i, j);
        } else {
            // instead of using a raw type here, it's possible to capture
            // the wildcard but it will require a call to a supplementary
            // private method
            // 使用迭代器进行交换
            ListIterator fwd = list.listIterator();
            ListIterator rev = list.listIterator(size);
            for (int i=0, mid=list.size()>>1; i<mid; i++) {
                Object tmp = fwd.next();
                fwd.set(rev.previous());
                rev.set(tmp);
            }
        }
    }
```

### fill
```java
    // 将元素填入list每一个位置
    public static <T> void fill(List<? super T> list, T obj) {
        int size = list.size();
        // 当数量小于25并且时是随机访问列表时，直接使用set进行赋值
        if (size < FILL_THRESHOLD || list instanceof RandomAccess) {
            for (int i=0; i<size; i++)
                list.set(i, obj);
        } else {
            // 使用迭代器进行赋值
            ListIterator<? super T> itr = list.listIterator();
            for (int i=0; i<size; i++) {
                itr.next();
                itr.set(obj);
            }
        }
    }
```

### min
```java
    // 求最小值，最大值类似
    public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll) {
        Iterator<? extends T> i = coll.iterator();
        T candidate = i.next();
        // 依次进行比较，选出最小的元素
        while (i.hasNext()) {
            T next = i.next();
            if (next.compareTo(candidate) < 0)
                candidate = next;
        }
        return candidate;
    }
```

### rotate
```java
    // 翻转一定距离
    public static void rotate(List<?> list, int distance) {
        // 为随机访问列表或者数量小于100时
        if (list instanceof RandomAccess || list.size() < ROTATE_THRESHOLD)
            rotate1(list, distance);
        else
            rotate2(list, distance);
    }
```

```java
    private static <T> void rotate1(List<T> list, int distance) {
        int size = list.size();
        if (size == 0)
            return;
        // 将过长距离缩短
        distance = distance % size;
        // 将过短的距离增加
        if (distance < 0)
            distance += size;
        if (distance == 0)
            return;
        // 将元素进行移动distance距离
        for (int cycleStart = 0, nMoved = 0; nMoved != size; cycleStart++) {
            T displaced = list.get(cycleStart);
            int i = cycleStart;
            do {
                i += distance;
                // 将后半部分元素前移动
                if (i >= size)
                    i -= size;
                displaced = list.set(i, displaced);
                nMoved ++;
            } while (i != cycleStart);
        }
    }
```

```java
    private static void rotate2(List<?> list, int distance) {
        int size = list.size();
        if (size == 0)
            return;
        int mid =  -distance % size;
        // 防止越界
        if (mid < 0)
            mid += size;
        if (mid == 0)
            return;
        // 通过mid进行划分，分别逆置
        reverse(list.subList(0, mid));
        reverse(list.subList(mid, size));
        // 逆置所有元素
        reverse(list);
    }
```

### replaceAll
```java
    public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal) {
        boolean result = false;
        int size = list.size();
        // 当大小小于11或者为随机访问列表时
        if (size < REPLACEALL_THRESHOLD || list instanceof RandomAccess) {
            // 老的值为空时
            if (oldVal==null) {
                // 遍历修改值
                for (int i=0; i<size; i++) {
                    if (list.get(i)==null) {
                        list.set(i, newVal);
                        result = true;
                    }
                }
            } else {
                // 当不为空时，通过equals判断是否相等，然后遍历赋值
                for (int i=0; i<size; i++) {
                    if (oldVal.equals(list.get(i))) {
                        list.set(i, newVal);
                        result = true;
                    }
                }
            }
        } else {
            ListIterator<T> itr=list.listIterator();
            if (oldVal==null) {
                for (int i=0; i<size; i++) {
                    if (itr.next()==null) {
                        itr.set(newVal);
                        result = true;
                    }
                }
            } else {
                for (int i=0; i<size; i++) {
                    if (oldVal.equals(itr.next())) {
                        itr.set(newVal);
                        result = true;
                    }
                }
            }
        }
        return result;
    }
```