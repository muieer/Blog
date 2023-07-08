## 简介
布隆过滤器用来做查找或过滤，判断一个元素是否在其中
## 布隆过滤器和哈希表(HashSet)异同
**同**：查询时间复杂度都为常量级，布隆O(K),哈希表O(1);用途一致  
**异**：布隆存在误差的可能；布隆添加数据性能稳定，哈希表扩容会引起不稳定；布隆空间利用率高
## 布隆过滤器优缺点
**优点**：空间利用率高；性能稳定  
**缺点**：有冲突且冲突随着数据规模的增加会变大；不能删除
## 源码实现
阅读代码为 `org.apache.spark.util.sketch.BloomFilter`
### 整体架构
接口BoomFilter对方法做了抽象和约束，BoomFilterImpl是实现类，BitArray是底层数据结构，充当位图作用
### 创建
创建阶段，调用静态方法`create`，根据预期添加的个数和可以容忍的最大读取错误率，算出需要的bit数和需要的哈希次数  
创建最终调用的方法如下
```java
BloomFilterImpl(int numHashFunctions, long numBits) {
        this(new BitArray(numBits), numHashFunctions);
}
```
由上知，过滤器的大小和数据存储依赖BitArray，其实现如下
```java
final class BitArray {
    private final long[] data;
    private long bitCount;

    static int numWords(long numBits) {
        if (numBits <= 0) {
            throw new IllegalArgumentException("numBits must be positive, but got " + numBits);
        }
        long numWords = (long) Math.ceil(numBits / 64.0);
        if (numWords > Integer.MAX_VALUE) {
            throw new IllegalArgumentException("Can't allocate enough space for " + numBits + " bits");
        }
        return (int) numWords;
    }

    BitArray(long numBits) {
        this(new long[numWords(numBits)]);
    }

    private BitArray(long[] data) {
        this.data = data;
        long bitCount = 0;
        for (long word : data) {
            bitCount += Long.bitCount(word);
        }
        this.bitCount = bitCount;
    }
}
```
实现思路为
1. 变量data为位图的实现，long值8个字节共64比特，位图大小为（数组长度 * 64）
2. bitCount 用来统计值等于1的比特数量
3. numBits整除64向上取整得到data数组长度，不能超过int类型整数最大值
### 添加
支持的类型为整数类型值（Byte、Short、Integer、Long）和 String  
实现细节大同小异，Long值实现如下
```java
@Override
  public boolean putLong(long item) {
    
    int h1 = Murmur3_x86_32.hashLong(item, 0);
    int h2 = Murmur3_x86_32.hashLong(item, h1);

    long bitSize = bits.bitSize();
    boolean bitsChanged = false;
    for (int i = 1; i <= numHashFunctions; i++) {
      int combinedHash = h1 + (i * h2);
      // Flip all the bits if it's negative (guaranteed positive number)
      if (combinedHash < 0) {
        combinedHash = ~combinedHash;
      }
      bitsChanged |= bits.set(combinedHash % bitSize);
    }
    return bitsChanged;
  }
```
具体流程为：
1. 首先使用哈希函数Murmur3根据待添加值item得到h1、h2
2. 根据numHashFunctions大小，循环计算，将位图对应位置的比特值设置为1
   1. 算出combinedHash，为要置1的位置，取绝对值
   2. 设置 BitArray 对应位置比特值为 1，返回设置结果 bitsChanged。对 bitSize 取余，保证计算的数值小于位图大小
3. 返回bitsChanged，为true即添加成功  

BitArray添加细节如下
```java
boolean set(long index) {
    if (!get(index)) {
      data[(int) (index >>> 6)] |= (1L << index);
      bitCount++;
      return true;
    }
    return false;
}
```
`index >>> 6`无符号右位移6位，相当于除64，算出要修改比特的索引，`1L << index`，index大于63会溢出，
相当于对64取模，找到数组指定索引long值要修改的比特位置
### 读取
相当于添加流程中对BitArray的写改为读，但必须都读出来比特值1，否则返回false，认为该值不存在
```java
@Override
  public boolean mightContainLong(long item) {
    int h1 = Murmur3_x86_32.hashLong(item, 0);
    int h2 = Murmur3_x86_32.hashLong(item, h1);

    long bitSize = bits.bitSize();
    for (int i = 1; i <= numHashFunctions; i++) {
      int combinedHash = h1 + (i * h2);
      // Flip all the bits if it's negative (guaranteed positive number)
      if (combinedHash < 0) {
        combinedHash = ~combinedHash;
      }
      if (!bits.get(combinedHash % bitSize)) {
        return false;
      }
    }
    return true;
  }
  
  // bits.get 实现源码
    boolean get(long index) {
    return (data[(int) (index >>> 6)] & (1L << index)) != 0;
    }
```
### 误差原因
对添加和读取的实现探究后，可发现如果一个值不存在，当调用`mightContain`方法计算该值对应位图索引恰巧都为1，就
会认为该值存在，造成误读。总结就是写进去的值一定会被读到，不存在漏读情形，但可能会对不存在的值误判认为存在
## 其他
1. `Long.bitCount`用来计算Long值二进制数中1的个数，可探究源码实现思路
2. `org.apache.spark.util.sketch. Murmur3_x86_32`哈希函数与其他哈希函数异同和优缺点可探究






