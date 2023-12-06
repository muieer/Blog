# Java 模块化系统

## 一、简介
官方文档：[Modular JDK](https://openjdk.org/projects/jigsaw/)

**模块化的好处：** 
1. 使开发者更容易构建和维护代码库和应用程序  
2. 提升 JDK 的可维护性  
3. 优化程序性能  
4. 减小 JDK 体积，可以在小型设备或云部署中使用

## 二、错误解决
**背景：** JDK 17，Spark 3.4.1，执行 `org.apache.spark.sql.SparkSession$Builder.getOrCreate()` 方法。

**错误调用栈：**  
Exception in thread "main" java.lang.IllegalAccessError: class org.apache.spark.storage.StorageUtils$ <wbr>
(in unnamed module @0x16eb3ea3) cannot access class sun.nio.ch.DirectBuffer (in module java.base) because module java.base does not export sun.nio.ch to unnamed module @0x16eb3ea3<br>
&nbsp;&nbsp;&nbsp;&nbsp;at org.apache.spark.storage.StorageUtils$.<clinit>(StorageUtils.scala:213)  
&nbsp;&nbsp;&nbsp;&nbsp;at org.apache.spark.storage.BlockManagerMasterEndpoint.<init>(BlockManagerMasterEndpoint.scala:114)
	
**添加虚拟机配置选项解决：**
1. --add-exports java.base/sun.nio.ch=ALL-UNNAMED  
2. --add-opens java.base/sun.nio.ch=ALL-UNNAMED

**解决原理：** 自 JDK 17 开始，禁止通过反射方式访问不被导出的内部类，<wbr>
具体见官方文档 [Strong Encapsulation in the JDK](https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-7BB28E4D-99B3-4078-BDC4-FC24180CE82B)。

