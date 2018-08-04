- [介绍](#介绍)
- [常量](#常量)
- [rangeCheck](#rangeCheck)
- [sort](#sort)
- [asList](#asList)


### 介绍
java.util中的工具类，提供数组相关的常用操作，排序、比较、填充、二分查找等功能

based on jdk8

### 常量

```

//小于等于这个值时，就使用插入排序。将被废弃
private static final int INSERTIONSORT_THRESHOLD = 7;

/*
并行排序的最小数组长度，数组长度小于这个数则不在划分数组
数组长度较小会导致排序的任务竞争内存导致效率降低
*/
 
private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;

```

### rangeCheck

```
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


```
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}

public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    DualPivotQuicksort.sort(a, fromIndex, toIndex - 1, null, 0, 0);
}

```

对Object[]、T[]的排序，主要就是legacyMergeSort()这个旧版归并排序和Timsort.sort()这个进行了很多优化的归并排序。也提供范围排序。

```
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

### toString

### equals & deepEquals


### hashCode & deepHashCode

### fill

### asList
 返回固定大小的ArrayList这个Arrays内部类，注意在使用时和java.util.ArrayList的区别。这个方法和Collection.toArray方法充当了数组和集合的桥梁
```
@SafeVarargs
@SuppressWarnings("varargs")
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```




### binarySearch


