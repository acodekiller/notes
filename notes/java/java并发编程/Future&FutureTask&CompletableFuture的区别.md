## 1. Future —— 最基础的结果占位符

> **本质**：一个接口，表示异步任务的结果，但只能通过**阻塞**或**轮询**获取。

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;  // 阻塞直到完成
    V get(long timeout, TimeUnit unit) throws ...;            // 限时阻塞
}
```

**优点**：比直接操作Thread更优雅，可以获取返回值。

**缺点**：

- `get()`是阻塞的，无法注册回调
- 无法手动完成
- 多个Future无法链式组合
- 没有异常处理机制（只能try-catch）

```java
// 典型用法
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Hello";
});
String result = future.get(); // 阻塞1秒
```

------



## 2. FutureTask —— 可执行的Future

> **本质**：Future的一个**具体实现类**，同时实现了`Runnable`和`Future`，所以既可以提交给线程池，也可以自己用Thread运行。

```java
public class FutureTask<V> implements RunnableFuture<V> {
    // RunnableFuture extends Runnable, Future
}
```

**关键特性**：

- 既可以作为`Runnable`被线程执行
- 又可以作为`Future`获取结果
- 内部封装了任务状态转换（NEW -> COMPLETING -> NORMAL/EXCEPTIONAL/CANCELLED）

```java
// 用法1：交给线程池
FutureTask<String> task = new FutureTask<>(() -> "Hello");
executor.submit(task);
String result = task.get();

// 用法2：直接用线程运行（不推荐，无法复用线程）
new Thread(task).start();
```

**相比Future的进步**：有了具体实现，可以自定义任务逻辑。

**缺点**：仍然无法解决阻塞、编排、回调等问题。



------

## 3. CompletableFuture —— 响应式编程的基石

> **本质**：Java 8引入，实现了`Future`和`CompletionStage`接口，支持**函数式、非阻塞、链式异步编程**。

### 核心能力对比

| 特性             | Future   | FutureTask | CompletableFuture                               |
| :--------------- | :------- | :--------- | :---------------------------------------------- |
| 手动完成/异常    | ❌        | ❌          | ✅ `complete()`/`completeExceptionally()`        |
| 非阻塞回调       | ❌        | ❌          | ✅ `thenApply()`/`thenAccept()`/`whenComplete()` |
| 组合多个异步任务 | ❌        | ❌          | ✅ `thenCombine()`/`allOf()`/`anyOf()`           |
| 异常处理链       | ❌        | ❌          | ✅ `exceptionally()`/`handle()`                  |
| 超时控制         | 有限支持 | 有限支持   | ✅ `orTimeout()`/`completeOnTimeout()`           |
| 流式API          | ❌        | ❌          | ✅ 方法链                                        |

### 典型示例

```java
// 非阻塞链式调用
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(String::toUpperCase)           // 转换
    .thenApply(s -> s + " World")
    .thenAccept(System.out::println)          // 消费（无返回值）
    .exceptionally(ex -> {
        System.err.println("出错: " + ex);
        return null;
    });

// 组合两个独立任务
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");
future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2)
       .thenAccept(System.out::println);  // 输出 "Hello World"

// 等待多个任务全部完成
CompletableFuture.allOf(future1, future2).join();
```



### 手动完成（这是Future完全没有的能力）

```java
CompletableFuture<String> future = new CompletableFuture<>();
// 在另一个线程或回调中手动设置结果
new Thread(() -> {
    // ... 做一些计算 ...
    future.complete("手动设置的结果");
}).start();
String result = future.get(); // 拿到 "手动设置的结果"
```