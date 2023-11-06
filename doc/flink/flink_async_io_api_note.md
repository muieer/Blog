# Flink 异步 IO API 学习笔记
## 一、总述
适用场景：IO 密集且耗时，CPU 利用率低的场景，如从外部数据库或存储系统中读数据，使用 Flink 异步 IO API 能提高 CPU 利用率，提升任务吞吐率。

官方文档：[Asynchronous I/O for External Data Access](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/datastream/operators/asyncio/)。

## 二、使用前提
执行 IO 操作的客户端提供了异步执行的接口，如示例的 [Spring JPA Asynchronous Query](https://docs.spring.io/spring-data/jdbc/docs/current/reference/html/#repositories.query-async)。

```java
@Async
Future<User> findByFirstname(String firstname);   
```
## 三、API 详解
### 3.1 异步操作的创建
```java
public static <IN, OUT> SingleOutputStreamOperator<OUT> unorderedWaitWithRetry(
            DataStream<IN> in,
            AsyncFunction<IN, OUT> func,
            long timeout,
            TimeUnit timeUnit,
            int capacity,
            AsyncRetryStrategy<OUT> asyncRetryStrategy) {
        return addOperator(
                in,
                func,
                timeUnit.toMillis(timeout),
                capacity,
                OutputMode.UNORDERED,
                asyncRetryStrategy);
}
```
调用该方法可以在流式处理的有向无环图中添加异步执行的计算节点。in 参数是异步计算节点依赖的上一个节点，func 参数定义了异步处理的具体逻辑，timeout 参数是流中的一条记录（record）在整个异步处理过程中的有效时间，timeUnit 参数是 timeout 参数的时间单位，capacity 参数是每个数据处理分片（流式计算的最小单位）支持的最大处理容量，asyncRetryStrategy 是重试策略。

### 3.2 AsyncFunction 
```
public interface AsyncFunction<IN, OUT> extends Function, Serializable {
     void asyncInvoke(IN input, ResultFuture<OUT> resultFuture) throws Exception;
     default void timeout(IN input, ResultFuture<OUT> resultFuture) throws Exception {
        resultFuture.completeExceptionally(
                new TimeoutException("Async function call has timed out."));
    }
}
```
AsyncFunction 接口的实现类定义了异步计算的具体处理逻辑，asyncInvoke 方法完成异步调用的计算逻辑，timeout 方法是一条记录在规定时间内取不到数据的超时处理逻辑实现。

asyncInvoke 方法的 input 参数是异步操作节点依赖的上一节点的输入元素，从该元素中携带数据中取出 Key 通过异步 IO 客户端查找数据，使用 java.util.concurrent.CompletableFuture 实现异步计算与回调，resultFuture 参数收集回调结果，因为是异步计算，整个过程不会阻塞。

asyncInvoke 方法在执行时顺序调用，而不是并行执行。方法体中异步计算，使用 resultFuture 完成异步回调，并不像同步计算那样，阻塞在某处，提高了 CPU 的利用率。

## 四、关键事项

### 4.1 异步计算的最大容量
指的是 3.1 章节中 capacity 参数，若此值设置的较小，能同时处在异步处理状态的 record 数量有限，阻塞上游节点后续流入的数据，出现反压现象；若此值设置的过大，吞吐率会提升，Flink 需要缓存的数据也同步增加，可能会出现 OOM 异常。

### 4.2 AsyncFunction.asyncInvoke(...) 方法是同步顺序执行上游节点流入数据（record）
上游节点流入到 AsyncFunction 中的数据是同步顺序执行的，asyncInvoke(...) 方法执行完上一条数据的方法调用才能执行下一条数据，所以需要使用 java.util.concurrent.CompletableFuture 这样的异步计算框架去异步执行 IO 操作，不会阻塞方法调用，能很快执行下一条数据，使用 ResultFuture 完成异步计算结果的回调。 

### 4.3 Flink 异步 IO API 提升数据处理吞吐率的原理
使用同步 IO 客户端，在结果未返回前，线程会阻塞在此处，不能处理下一条数据，CPU 利用率不高。就如 4.2 章节所描述的，使用异步计算框架和回调机制，不会阻塞方法调用，程序能很快的处理下一条数据，数据处理的吞吐率就能提高。

### 4.4 AsyncFunction 不应该处理计算密集的逻辑
Flink 异步 IO API 目标是解决 IO 请求阻塞时间长，CPU 利用率低的问题，若是计算密集的任务，使用异步 IO API，线程切换带来的资源开销和性能损耗，会导致吞吐率下降。

[//]: # (## 五、生产环境中实践)




