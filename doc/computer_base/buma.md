## 简介
二进制的一种表达形式，对于整数，0和正数高位为0，负数为1
## 位移运算
左位移：低位补0，如`0001`左位移一位为`0010`，相当于✖️2
右位移：对于正数高位补0，负数高位补1，如`1001`右位移一位为`1100`
## 计算负数真实值
对于负数`1100`计算方式为按位取反再加一，得到该值绝对值为`0100`，所以该数为-4
## 计算负数补码
对其绝对值按位取反再加一
## Java 进制转换
```java
//      显示负数补码值
        System.out.println(Integer.toBinaryString(-1));

        // 二进制转十进制
        System.out.println(Integer.parseInt("1111", 2));

        // 八进制转十进制
        System.out.println(Integer.parseInt("011", 8));
```
## 补码好处
加减逻辑都可以复用加法器实现