- [介绍](#介绍)
- [常量](#常量)
- [rangeCheck](#rangecheck)
- [sort](#sort)
- [parallelSort](#parallelsort)
- [equals&deepEquals](#equalsdeepequals)
- [fill](#fill)
- [asList](#aslist)
- [binarySearch](#binarysearch)
- [copyOf](#copyof)


### 介绍
java.util中的工具类，提供数组相关的常用操作，排序、比较、填充、二分查找等功能

based on jdk8

### 常量

```java

//小于等于这个值时，就使用插入排序。将被废弃
private static final int INSERTIONSORT_THRESHOLD = 7;

/*
并行排序的最小数组长度，数组长度小于这个数则不在划分数组
数组长度较小会导致排序的任务竞争内存导致效率降低
*/
 
private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;

```

### rangeCheck

```java
//私有方法，检查是否越界
private static void rangeCheck(int arrayLength, int fromIndex, int toIndex) {
    if (fromIndex > toIndex) {
        throw new IllegalArgumentException(
                "fromIndex(" + fromIndex + ") > toIndex(" + toIndex + ")");
    }
    if (fromIndex < 0) {
        throw new ArrayIndexOutOfBoundsException(fromIndex);
    }
    if (toIndex > arrayLength) {
        throw new ArrayIndexOutOfBoundsException(toIndex);
    }
}

```

### sort 

对于int[]、byte[]、long[]等基本类型数组的排序，使用DualPivotQuicksort类进行排序，可选范围。

注意这个类对改动了双轴快排的策略，使用了其他的排序方法,查看其源码可以知道还使用了计数排序、插入排序、归并排序。很多会导致其他版本快排退化到O(n^2)的数据集使用这个类仍能保证O(nlogn)

[单轴快排和双轴快排](https://blog.csdn.net/Holmofy/article/details/71168530)

- 快排的思想是分治，方法是递归
- 单轴快排只有一个划分点，对点两侧的区间进行递归
- 双轴快排有两个划分点，将区间分为三段，效率会高一些


```java
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}

public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}

```

对Object[]、T[]的排序，主要就是legacyMergeSort()这个旧版归并排序和Timsort.sort()这个进行了很多优化的归并排序。也提供范围排序。

```java
public static void sort(Object[] a) {
    //为了兼容性，允许通过系统属性的方式使用旧版归并实现，不过将要废弃了
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a);
    else
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);
}
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}
```


### parallelSort
对基本类型的并行排序，以int[]为例。提供范围排序
```java
/*
这个排序算法是个将数组划分为几个子数组分别排序然后合并的并行排序-合并过程。
当子数组长度达到最小粒度，或者数组小于设定的最小粒度，
使用类似Arrays.sort()的方法(DualPivotQuickSort)来进行排序。
这个算法需要一个不大于原数组大小的额外空间，使用ForkJoin common pool
ForkJoinPool#commonPool()）来执行并行的排序任务。
*/

public static void parallelSort(int[] a) {
    int n = a.length, p, g;
    //如果数组长度小于分组的最小粒度或者只有一个执行线程，使用DualPivotQuicksort
    if (n <= MIN_ARRAY_SORT_GRAN ||
        (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
        DualPivotQuicksort.sort(a, 0, n - 1, null, 0, 0);
    else
        //g表示粒度，参数4、5、6分别为排序数组开始位置，需要排序的长度和额外空间的开始位置
        //g = n / (p << 2)不可小于最小粒度,否则使用最小粒度
        new ArraysParallelSortHelpers.FJInt.Sorter
            (null, a, new int[n], 0, n, 0,
             ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
             MIN_ARRAY_SORT_GRAN : g).invoke();
}


public static void parallelSort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    int n = toIndex - fromIndex, p, g;
    if (n <= MIN_ARRAY_SORT_GRAN ||
        (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
        DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
    else
        new ArraysParallelSortHelpers.FJInt.Sorter
            (null, a, new int[n], fromIndex, n, 0,
             ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
             MIN_ARRAY_SORT_GRAN : g).invoke();
}

```
对Object[]、T[]的排序
```java
public static <T extends Comparable<? super T>> void parallelSort(T[] a) {
    int n = a.length, p, g;
    if (n <= MIN_ARRAY_SORT_GRAN ||
        (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
        //与基本类型不同，这里使用TimSort
        TimSort.sort(a, 0, n, NaturalOrder.INSTANCE, null, 0, 0);
    else
        //不提供Comparator时，使用Arrays.NaturalOrder自然顺序的Comparator
        new ArraysParallelSortHelpers.FJObject.Sorter<T>
        (null, a,
         (T[])Array.newInstance(a.getClass().getComponentType(), n),
         0, n, 0, ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
         MIN_ARRAY_SORT_GRAN : g, NaturalOrder.INSTANCE).invoke();
}




```

### equals&deepEquals

```java
public static boolean equals(int[] a, int[] a2) {
    if (a==a2)
        return true;
    //存在null，则false
    if (a==null || a2==null)
        return false;

    int length = a.length;
    if (a2.length != length)
        return false;
    //依次比较
    for (int i=0; i<length; i++)
        if (a[i] != a2[i])
            return false;
    return true;
}

//Object[]的比较   
for (int i=0; i<length; i++) {
Object o1 = a[i];
Object o2 = a2[i];
if (!(o1==null ? o2==null : o1.equals(o2)))
    return false;
}

//深入比较，即比较多维数组。deepHashcode和deepToString也是如此。
public static boolean deepEquals(Object[] a1, Object[] a2) {
    if (a1 == a2)
        return true;
    if (a1 == null || a2==null)
        return false;
    int length = a1.length;

    //长度不同直接false
    if (a2.length != length)
        return false;

    for (int i = 0; i < length; i++) {
        Object e1 = a1[i];
        Object e2 = a2[i];

        if (e1 == e2)
            continue;
        if (e1 == null)
            return false;

        //递归比较，记录是否相等
        boolean eq = deepEquals0(e1, e2);

        if (!eq)
            return false;
    }
    return true;
}

```

### fill
用给定的值填充数组，提供范围操作
```java
public static void fill(int[] a, int val) {
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}
```
对于Object[]的填充，只是填充了同一引用，并未克隆对象
```java
public static void fill(Object[] a, Object val) {
    for (int i = 0, len = a.length; i < len; i++)
        a[i] = val;
}


```
### asList
返回固定大小的ArrayList这个Arrays内部类，注意在使用时和java.util.ArrayList的区别。这个方法和Collection.toArray方法充当了数组和集合的桥梁
```java
@SafeVarargs
@SuppressWarnings("varargs")
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

### binarySearch
二分查找，显然数组必须是有序的，否则结果不确定了。如果数组里面有多个相同的，不能保证找到哪一个
```java
public static int binarySearch(int[] a, int key) {
    //调用私有方法
    return binarySearch0(a, 0, a.length, key);
}

private static int binarySearch0(int[] a, int fromIndex, int toIndex,
                                 int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        int midVal = a[mid];

        if (midVal < key)
            low = mid + 1;
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // 找到了
    }
    return -(low + 1);  // 没找着，保证为负
}

```

### copyOf

```java
//给出数组新长度，拷贝到新数组中，使用System.arrayCopy实现，newLenght >= oldLength
public static short[] copyOf(short[] original, int newLength) {
    short[] copy = new short[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}


```
