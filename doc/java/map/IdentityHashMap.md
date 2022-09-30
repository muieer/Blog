## 起因
执行Spark任务，需要将变量广播，出现了内存溢出报错，最顶层的栈帧用到`IdentityHashMap`，好奇为什么用该类，它有什么不一样
![image1](/image/IdentityHashMap.png)
## IdentityHashMap VS HashMap
### 相同
1. 都继承了 Map 接口，通过数组保存数据
### 不同
1. IdentityHashMap Key 和 Value 允许为 null
2. IdentityHashMap 是根据引用地址判断 Key 是否相同，而 HashMap 是通过 equals 方法判断
3. 应用场景不同，HashMap 作为 K-V 结构来存储数据，方便查找。IdentityHashMap 只是继承了 Map 接口，
但应用场景不是这样，开发文档中写到，典型的应用场景是序列化和深拷贝
![image2](/image/IdentityHashMap1.png)
## 源码探究
### 新建
```java
public IdentityHashMap(int expectedMaxSize) {
        if (expectedMaxSize < 0)
            throw new IllegalArgumentException("expectedMaxSize is negative: "
                                               + expectedMaxSize);
        init(capacity(expectedMaxSize));
}
private static int capacity(int expectedMaxSize) {
        // assert expectedMaxSize >= 0;
        return
        (expectedMaxSize > MAXIMUM_CAPACITY / 3) ? MAXIMUM_CAPACITY :
        (expectedMaxSize <= 2 * MINIMUM_CAPACITY / 3) ? MINIMUM_CAPACITY :
        Integer.highestOneBit(expectedMaxSize + (expectedMaxSize << 1));
}
```
最终会用到上述两个方法，限定了 Map 容量最大值和最小值，容量必须为 2 的幂次方
### put
```java
// 保存数据是数组（桶）
transient Object[] table;

// 返回的是旧值，新添加返回 null
public V put(K key, V value) {
        final Object k = maskNull(key); //允许保存空值，会赋值一个不可变值

        retryAfterResize: for (;;) {
            final Object[] tab = table;
            final int len = tab.length;
            int i = hash(k, len);

            for (Object item; (item = tab[i]) != null;
                 i = nextKeyIndex(i, len)) {
                // 如果 Key 相同，替换旧值，不同，指针后移一位
                if (item == k) { 
                    @SuppressWarnings("unchecked")
                        V oldValue = (V) tab[i + 1];
                    tab[i + 1] = value;
                    return oldValue;
                }
            }

            final int s = size + 1;
            // Use optimized form of 3 * s.
            // Next capacity is len, 2 * current capacity.
        // 扩容阈值为 2/3
            if (s + (s << 1) > len && resize(len))
                continue retryAfterResize;

            modCount++;
            // key 和 value 比邻保存
            tab[i] = k;
            tab[i + 1] = value;
            size = s;
            return null;
        }
    }
```
由该方法知：IdentityHashMap 数据结构是数组，K-V 是比邻而居，一个槽只能有一个数据，HashMap 是数组加链表，同一个槽不同 K-V 可以通过
链表串起来，IdentityHashMap 扩容阈值为 2/3 不可更改
### 扩容
```java
private boolean resize(int newCapacity) {
        // assert (newCapacity & -newCapacity) == newCapacity; // power of 2
        // newCapacity 是旧表长度，所以是二倍扩容
        int newLength = newCapacity * 2;

        Object[] oldTable = table;
        int oldLength = oldTable.length;
        if (oldLength == 2 * MAXIMUM_CAPACITY) { // can't expand any further
            // 最大容量是 MAXIMUM_CAPACITY - 1，为什么？若是 MAXIMUM_CAPACITY，在 put 阶段会陷入死循环
            // 因为没有槽为空，不断移动寻找空槽，但是没有，所以死循环，这点在官方文档中说的很清楚
        if (size == MAXIMUM_CAPACITY - 1)
                throw new IllegalStateException("Capacity exhausted.");
            return false;
        }
        if (oldLength >= newLength)
            return false;

        Object[] newTable = new Object[newLength];

        // 旧表内容移到扩容后新表
        for (int j = 0; j < oldLength; j += 2) {
            Object key = oldTable[j];
            if (key != null) {
                Object value = oldTable[j+1];
                oldTable[j] = null;
                oldTable[j+1] = null;
                int i = hash(key, newLength);
                while (newTable[i] != null)
                    i = nextKeyIndex(i, newLength);
                newTable[i] = key;
                newTable[i + 1] = value;
            }
        }
        table = newTable;
        return true;
}
```
## 应用场景思考
其他 Map 如 HashMap 是通过`Object.equals`方法判断Key是否相等，重写 equals 方法就必须重写 hashCode 方法，更加关注的是 Key 内容的不同,
IdentityHashMap 直接比较对象内存地址，更关注对象本身，即便是克隆，只要是独立的， 就当作不同的 Key 看待，这样就适合序列化、深拷贝这样的场景，
即使内容一样，但对每个个体都单独看待  

Spark 对一份数据广播，定要序列化和反序列化进行网络传输，更关注数据本身，即使内容一样，在此应用 IdentifyHashMap 就有其合理性

