---
authors:
  - mingyi_zhang
date:
  created: 2023-07-23
  updated: 2023-10-22
tags:
  - Java
  - Kotlin
---

# Integrating Kotlin's Suspend Functions with Java's CompletableFuture

Kotlin's coroutine-based asynchronous model offers suspend functions that make asynchronous programming straightforward.
However, when integrating with Java, which does not natively understand these suspend functions, one might face
compatibility issues. Thankfully, we can bridge this gap by converting suspend functions into
Java's `CompletableFuture`, which provides a non-blocking way to handle asynchronous operations.

## Converting Suspend Functions to CompletableFuture

To facilitate this conversion, we can use the following utility function, `suspendToFuture`, which takes in
an `Executor` and a suspend function, then returns a `CompletableFuture`:

``` kotlin
fun <T> suspendToFuture(
  executor: Executor,
  block: suspend () -> T
): CompletableFuture<T> {
  val future = CompletableFuture<T>()
  val dispatcher = executor.asCoroutineDispatcher()
  CoroutineScope(dispatcher + SupervisorJob()).launch {
    try {
      future.complete(block())
    } catch (e: Exception) {
      future.completeExceptionally(e)
    }
  }
  return future
}
```

## Usage Example

Let's take a suspend function as an example:

```kotlin
suspend fun add(x: Int, y: Int): Int {
  delay(1000)
  return x + y  
}
```

To use this function from Java, we first wrap it into a non-blocking function that returns a `CompletableFuture`:

```kotlin
fun addAsync(
  x: Int,
  y: Int,
  executor: Executor
): CompletableFuture<Int> = suspendToFuture(executor) {
  add(x, y)
}
```

Subsequently, from Java, we can then execute this function asynchronously and handle the result:

```java
final var executor = Executors.newSingleThreadExecutor();
final var future = addAsync(1, 2, executor);
future.thenAccept(System.out::println);
```

In the above Java code, the `addAsync` function is called, and its result (after a delay) will be printed to the
console. This approach allows seamless interoperability between Kotlin's coroutine model and Java's asynchronous
paradigm.
