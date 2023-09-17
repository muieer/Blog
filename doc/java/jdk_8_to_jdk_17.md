# 从 JDK 8 升级到 JDK 17
## 一、重要的新特性
### 1.1 jshell
官方文档：https://docs.oracle.com/en/java/javase/17/docs/specs/man/jshell.html
### 1.2 var 关键字
见下
### 1.3 文字块
官方文档：https://docs.oracle.com/en/java/javase/17/text-blocks/  
```java
// 第一个 """ 是开始分隔符，单独成行
var text = """
        ....
            生命就是这样的
            你总要做些什么 \
        或者感受些什么
            这两种过程都值得尊敬 不能怠慢
        ....
        """;
// 「\」是换行转义符，有些类似与 Shell 脚本中的换行转义符
// 最后一个 """ 是结束分隔符，它之前的空格都会被省略，解决源码缩进问题
System.out.print(text);
```
本质：就是字符串，在编译期处理，JVM 不知道这是源自普通字符串还是文字块  
使用场景：多行文本、频繁转义、需要换行  
优点：更好写、更好读  
使用要点：
## 二、文档
官方文档：https://docs.oracle.com/en/java/javase/17/  
迁移手册：https://docs.oracle.com/en/java/javase/17/migrate