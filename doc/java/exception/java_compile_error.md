# Java 编译错误
## Maven 打包基于 Jdk11 编译 Jdk8 版本代码，运行时出错
现象：java.lang.NoSuchMethodError: java.nio.ByteBuffer.flip()Ljava/nio/ByteBuffer;  
问题排查：查看源代码，发现 Jdk8 java.nio.ByteBuffer 类无 flip() 方法，其父类有；(stack overflow 回答解释说这是 Jdk11 编译引起的错误)
[https://stackoverflow.com/questions/61267495/exception-in-thread-main-java-lang-nosuchmethoderror-java-nio-bytebuffer-flip]
，并且 Jdk11 Java.nio.ByteBuffer 类有 flip() 方法  
结论：编译 Jdk8 版本代码最好还是用 Jdk8，用 Jdk11 会引入潜在 bug 
