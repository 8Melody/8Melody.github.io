---
layout: post
title: "Coroutines Flow"
subtitle: "Coroutines Flow小记"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Android
  - Coroutines
---

# Flow

Flow的出现是为了解决挂起函数只能返回一个返回值的问题，Flow支持返回一个"流"，可以理解为可以返回多个异步处理的结果。

# 创建一个Flow

创建flow的方式在`Builders.kt`中，主要有以下：

* 最基本的创建方式：
```kotlin
val mFlow = flow {
        emit(1)
        emit(2)
        emit(3)
    }
```
* asFlow
```kotlin
@FlowPreview
public fun <T> (() -> T).asFlow(): Flow<T> = flow {
    emit(invoke())
}

/**
 * Creates a _cold_ flow that produces a single value from the given functional type.
 *
 * Example of usage:
 *
 * ```
 * suspend fun remoteCall(): R = ...
 * fun remoteCallFlow(): Flow<R> = ::remoteCall.asFlow()
 * ```
 */
@FlowPreview
public fun <T> (suspend () -> T).asFlow(): Flow<T> = flow {
    emit(invoke())
}

/**
 * Creates a _cold_ flow that produces values from the given iterable.
 */
public fun <T> Iterable<T>.asFlow(): Flow<T> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a _cold_ flow that produces values from the given iterator.
 */
public fun <T> Iterator<T>.asFlow(): Flow<T> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a _cold_ flow that produces values from the given sequence.
 */
public fun <T> Sequence<T>.asFlow(): Flow<T> = flow {
    forEach { value ->
        emit(value)
    }
}
public fun <T> Array<T>.asFlow(): Flow<T> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a _cold_ flow that produces values from the array.
 * The flow being _cold_ means that the array components are read every time a terminal operator is applied
 * to the resulting flow.
 */
public fun IntArray.asFlow(): Flow<Int> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a _cold_ flow that produces values from the given array.
 * The flow being _cold_ means that the array components are read every time a terminal operator is applied
 * to the resulting flow.
 */
public fun LongArray.asFlow(): Flow<Long> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a flow that produces values from the range.
 */
public fun IntRange.asFlow(): Flow<Int> = flow {
    forEach { value ->
        emit(value)
    }
}

/**
 * Creates a flow that produces values from the range.
 */
public fun LongRange.asFlow(): Flow<Long> = flow {
    forEach { value ->
        emit(value)
    }
}
```
如：
```kotlin
 val mFlow1 = (1..3).asFlow()
 val mFlow2 = arrayOf(1, 2, 3).asFlow()
 val mFlow3 = sequenceOf(1, 2, 3).asFlow()
 //...
```
* flowOf
```kotlin

/**
 * Creates a flow that produces values from the specified `vararg`-arguments.
 *
 * Example of usage:
 *
 * ```
 * flowOf(1, 2, 3)
 * ```
 */
public fun <T> flowOf(vararg elements: T): Flow<T> = flow {
    for (element in elements) {
        emit(element)
    }
}

/**
 * Creates a flow that produces the given [value].
 */
public fun <T> flowOf(value: T): Flow<T> = flow {
    /*
     * Implementation note: this is just an "optimized" overload of flowOf(vararg)
     * which significantly reduces the footprint of widespread single-value flows.
     */
    emit(value)
}
```
如：
```kotlin
val mFlow1 = flowOf(1,2,3)
val mFlow2 = flowOf(1)
```
# 主要操作符简介：
## emit、collect
“发射”、“收集”，规律如下：
```kotlin
 val f = flow {
        (1..3).forEach {
            emit(it)
            "emit $it".log()
        }
    }
    runBlocking {
        f.collect { value ->
                "collect $value".log()
            }
    }
    //collect 1
    //emit 1
    //collect 2
    //emit 2
    //collect 3
    //emit 3
```
> 如果flow只有一个元素的时候，可以使用`single()`直接获取这个元素，但是如果flow为空，则会抛出`NoSuchElementException("Flow is empty")`，再或者flow元素不止一个的时候，会抛出`IllegalArgumentException("Flow has more than one element")`

源码：
```kotlin
public suspend fun <T> Flow<T>.single(): T {
    var result: Any? = NULL
    collect { value ->
        require(result === NULL) { "Flow has more than one element" }
        result = value
    }

    if (result === NULL) throw NoSuchElementException("Flow is empty")
    return result as T
}
```
类似的还有`singleOrNull`、`first`、`firstOrNull`。

如：
```kotlin
val f = flowOf(1, 2, 3)
    runBlocking {
        f.first().toString().log()
    }
    //1
    
 val f = flowOf(1, 2, 3)
    runBlocking {
        f.first {
            it > 2
        }.toString().log()
    }
    //3
```
## transform（转换）
源码：
```kotlin
public inline fun <T, R> Flow<T>.transform(
    @BuilderInference crossinline transform: suspend FlowCollector<R>.(value: T) -> Unit
): Flow<R> = flow { // Note: safe flow is used here, because collector is exposed to transform on each operation
    collect { value ->
        // kludge, without it Unit will be returned and TCE won't kick in, KT-28938
        return@collect transform(value)
    }
}
```
例：
```kotlin
 val f = flow {
        (1..3).forEach {
            "emit $it".log()
            emit(it)
        }
    }
    runBlocking {
        f.transform { originValue->
            emit("transform-$originValue")
            emit("ok")
        }.collect { value ->
            "collect $value".log()
        }
    }
    //emit 1
    //collect transform-1
    //collect ok
    //emit 2
    //collect transform-2
    //collect ok
    //emit 3
    //collect transform-3
    //collect ok
```
## filter（过滤）
内部实现是`transform`，根据`filter`的表达式过滤，并生成一个新的`flow`。
源码：
```kotlin
public inline fun <T> Flow<T>.filter(crossinline predicate: suspend (T) -> Boolean): Flow<T> = transform { value ->
    if (predicate(value)) return@transform emit(value)
}
```
例：
```kotlin
val f = flow {
        (1..3).forEach {
            "emit $it".log()
            emit(it)
        }
    }
    runBlocking {
        f.filter { 
            it > 1
        }.collect { value ->
            "collect $value".log()
        }
    }
    //emit 1
    //collect 2
    //emit 2
    //collect 3
    //emit 3
```
还有与之对应的`filterNot`、`filterIsInstance`、`filterNotNull`。
## map（转换）
实现内部是`transform`，与`transform`不同的是`map`的参数不需要返回一个`FlowCollector`，只是干预`emit`发出来的值。
 
源码：
```kotlin
public inline fun <T, R> Flow<T>.map(crossinline transform: suspend (value: T) -> R): Flow<R> = transform { value ->
   return@transform emit(transform(value))
}
```
例：
```kotlin
val f = flow {
        (1..3).forEach {
            "emit $it".log()
            emit(it)
        }
    }
    runBlocking {
        f.map { originValue->
          "transform-$originValue"
        }.collect { value ->
            "collect $value".log()
        }
    }
    //emit 1
    //collect transform-1
    //emit 2
    //collect transform-2
    //emit 3
    //collect transform-3
```
此外还有`mapNotNull`。
## take
从开始位置到需要`take`数量的为止，返回一个新的`flow`当collect到`take`数量之后，原始的`flow`会被取消。

源码：
```kotlin
public fun <T> Flow<T>.take(count: Int): Flow<T> {
    require(count > 0) { "Requested element count $count should be positive" }
    return flow {
        var consumed = 0
        try {
            collect { value ->
                // Note: this for take is not written via collectWhile on purpose.
                // It checks condition first and then makes a tail-call to either emit or emitAbort.
                // This way normal execution does not require a state machine, only a termination (emitAbort).
                // See "TakeBenchmark" for comparision of different approaches.
                if (++consumed < count) {
                    return@collect emit(value)
                } else {
                    return@collect emitAbort(value)
                }
            }
        } catch (e: AbortFlowException) {
            e.checkOwnership(owner = this)
        }
    }
}

private suspend fun <T> FlowCollector<T>.emitAbort(value: T) {
    emit(value)
    throw AbortFlowException(this)
}
```
例：
```kotlin
 val f = (1..3).asFlow()
    runBlocking {
        f.take(1)
            .collect { value ->
                "collect $value".log()
            }
    }
    //collect 1
```
此外还有`takeWhile`。
## drop 
顾名思义，从`drop`的的位置开始，返回一个新的`flow`。

源码：
```kotlin
public fun <T> Flow<T>.drop(count: Int): Flow<T> {
    require(count >= 0) { "Drop count should be non-negative, but had $count" }
    return flow {
        var skipped = 0
        collect { value ->
            if (skipped >= count) emit(value) else ++skipped
        }
    }
}
```
例：
```kotlin
 val f = flow {
        (1..3).forEach {
            "emit $it".log()
            emit(it)
        }
    }
    runBlocking {
        f.drop(2).collect { value ->
            "collect $value".log()
        }
    }
    //emit 1
    //emit 2
    //emit 3
    //collect 3
```
此外还有`dropWhile`。
## reduce
根据操作符累计值。

源码：
```kotlin
public suspend fun <S, T : S> Flow<T>.reduce(operation: suspend (accumulator: S, value: T) -> S): S {
    var accumulator: Any? = NULL

    collect { value ->
        accumulator = if (accumulator !== NULL) {
            @Suppress("UNCHECKED_CAST")
            operation(accumulator as S, value)
        } else {
            value
        }
    }

    if (accumulator === NULL) throw NoSuchElementException("Empty flow can't be reduced")
    @Suppress("UNCHECKED_CAST")
    return accumulator as S
}
```
例：
```kotlin
runBlocking {
        val result = flowOf(1, 2, 3)
            .reduce { accumulator, value ->
                "accumulator $accumulator，value $value".log()
                accumulator + value
            }
        "result = $result".log()
    }
    //accumulator 1，value 2
    //accumulator 3，value 3
    //result = 6
```
```kotlin
 runBlocking {
        val result = flowOf(1, 2, 3)
            .reduce { accumulator, value ->
                accumulator - value 
            }
        "result = $result".log()
    }
    //result = -4
```
类似的还有`fold`，`fold`会提供一个初始值参与计算。

源码：
```kotlin
public suspend inline fun <T, R> Flow<T>.fold(
    initial: R,
    crossinline operation: suspend (acc: R, value: T) -> R
): R {
    var accumulator = initial
    collect { value ->
        accumulator = operation(accumulator, value)
    }
    return accumulator
}
```
例：
```kotlin
runBlocking {
        val result = flowOf(1, 2, 3)
            .fold(10) { accumulator, value ->
                "accumulator $accumulator，value $value".log()
                accumulator + value
            }
        "result = $result".log()
    }
    //accumulator 10，value 1
    //accumulator 11，value 2
    //accumulator 13，value 3
    //result = 16
```
## buffer
先运行完emit的代码，再进行collect。

不使用`buffer`的情况：

```kotlin
val f = flowOf("A", "B", "C")
    runBlocking {
        f.onEach { "each 1$it".log() }
            .collect { value -> "collect 2$value".log() }
    }
    //each 1A
    //collect 2A
    //each 1B
    //collect 2B
    //each 1C
    //collect 2C
```
使用buffer操作的结果：

```kotlin
  val f = flowOf("A", "B", "C")
    runBlocking {
        f.onEach { "each 1$it".log() }
            .buffer()
            .collect { value -> "collect 2$value".log() }
    }
    //each 1A
    //each 1B
    //each 1C
    //collect 2A
    //collect 2B
    //collect 2C
```

## conflate
将`flow`任务混合，并将`collect`运行在一个单独的协程中，这样`emit`不会因为`collect`执行慢而挂起等待，`collect`会一直拿到最新的`emit`值。
```kotlin
val f = flow {
        for (i in 1..3) {
            delay(100)
            emit(i)
            "emit $i".log()
        }
    }
    runBlocking {
        val time = measureTimeMillis {
            f.conflate().collect {
                delay(300)
                "collect $it".log()
            }
        }
        "time is $time".log()
    }
    //emit 1
    //emit 2
    //emit 3
    //collect 1
    //collect 3
    //time is 792
```
## collectLatest
```kotlin
  val f = flow {
        for (i in 1..3) {
            delay(100)
            emit(i)
            "emit $i".log()
        }
    }
    runBlocking {
        val time = measureTimeMillis {
            f.collectLatest {
                "before delay collect $it".log()
                delay(300)
                "collect $it".log()
            }
        }
        "time is $time".log()
    }
    //before delay collect 1
    //emit 1
    //before delay collect 2
    //emit 2
    //before delay collect 3
    //emit 3
    //collect 3
    //time is 745
    
```
## zip
根据`transform`表达式“压缩”另一个`flow`，并返回一个新的`flow`，“压缩”过程中`flow`会以最先完成的`flow`为结束依据，剩下的没有完成的`flow`会被取消。

```kotlin
  val flow = flowOf(1, 2, 3).onEach { delay(10) }
    val flow2 = flowOf("A", "B", "C", "D").onEach { delay(15) }
    runBlocking {
        flow.zip(flow2) { i, s -> i.toString() + s }
            .collect { it.log() }
    }
    //1A
    //2B
    //3C
```
## combine 
与`zip`不同的是`combine`会将两个`flow`都执行完，并且会根据最新`emit`的值组合。
```kotlin
val flow = flowOf(1, 2).onEach { delay(10) }
    val flow2 = flowOf("A", "B", "C").onEach { delay(15) }
    runBlocking {
        flow.combine(flow2) { i, s -> i.toString() + s }
            .collect { it.log() }
    }
    //1A
    //2A
    //2B
    //2C
```
## oneach
在返回新`flow`之前，先执行`onEach`表达式。
```kotlin
public fun <T> Flow<T>.onEach(action: suspend (T) -> Unit): Flow<T> = transform { value ->
    action(value)
    return@transform emit(value)
}
```
如：
```kotlin
 val intFlow = flowOf(1, 2, 3)
    runBlocking {
        intFlow.onEach { value ->
            "onEach $value".log()
        }.collect()
    }
    //onEach 1
    //onEach 2
    //onEach 3
```
其中`collect()`会触发`flow`的`collect`方法，但是会忽略`emit`的内容，其实就是`collect{}`的缩略写法。
`collect()`一般会跟`onEach`、`onCompletion`、`catch`操作符一起出现。
## flowOn
正常情况下`flow`的`emit`跟`collect`会在同一个协程中执行，知道基本的协程切换的话，可能会想通过`withContext`来将`flow`的`emit`和
`collect`放在不同协程中执行，但是这是错误的，实现`flow`的协程切换要通过`flowOn`。

错误示范：
```kotlin
val intFlow = flow {
        withContext(Dispatchers.IO) {
            emit(1)
        }
    }
    runBlocking {
        intFlow.collect { value ->
            "collect $value".log()
        }
    }
```
以上代码会抛出异常
> java.lang.IllegalStateException: Flow invariant is violated:
> 
正确使用方式：

没有使用`flowOn`：
```kotlin
 val intFlow = flow {
        "emit thread ${Thread.currentThread().name}".log()
        emit(1)
    }
    runBlocking {
        intFlow.collect { value ->
            "collect thread ${Thread.currentThread().name}".log()
            "collect $value".log()
        }
    }
    //emit thread main @coroutine#1
    //collect thread main @coroutine#1
    //collect 1
```
使用了`flowOn`：
```kotlin
val intFlow = flow {
        "emit thread ${Thread.currentThread().name}".log()
        emit(1)
    }.flowOn(Dispatchers.IO)
    runBlocking {
        intFlow.collect { value ->
            "collect thread ${Thread.currentThread().name}".log()
            "collect $value".log()
        }
    }
    //emit thread DefaultDispatcher-worker-1 @coroutine#2
    //collect thread main @coroutine#1
    //collect 1
```
完整的解释：
```kotlin
withContext(Dispatchers.Main) {
    val singleValue = intFlow // will be executed on IO if context wasn't specified before
        .map { ... } // Will be executed in IO
        .flowOn(Dispatchers.IO)
        .filter { ... } // Will be executed in Default
        .flowOn(Dispatchers.Default)
        .single() // Will be executed in the Main
}
```
如果一个`flow`操作调用了多次的`flowOn`，则以第一次的`flowOn`为准，如：
```kotlin
flow.map { ... } // Will be executed in IO
    .flowOn(Dispatchers.IO) // This one takes precedence
    .flowOn(Dispatchers.Default)
```
**注意`SharedFlow`的子类本身没有执行的`context`，所以`flowOn`对其无效。**
# Exceptions
## try catch
```kotlin
  val f = flow {
        for (i in 1..3) {
            "emit $i".log()
            emit(i)
        }
    }

    runBlocking {
        try {
            f.collect { value ->
                "collect $value".log()
                check(value <= 1) {
                    "crashed on $value"
                }
            }
        } catch (e: Throwable) {
            "异常 $e".log()
        }
    }
    //emit 1
    //collect 1
    //emit 2
    //collect 2
    //异常 java.lang.IllegalStateException: crashed on 2
```
```kotlin
 val f = flow {
        for (i in 1..3) {
            "emit $i".log()
            emit(i)
        }
    }.map { value ->
        check(value <= 1) {
            "crashed on $value"
        }
        "mapValue $value"
    }

    runBlocking {
        try {
            f.collect { value ->
                "collect $value".log()
            }
        } catch (e: Throwable) {
            "异常 $e".log()
        }
    }
    //emit 1
    //collect mapValue 1
    //emit 2
    //异常 java.lang.IllegalStateException: crashed on 2
```
## catch
```kotlin
val f = flow {
        for (i in 1..3) {
            "emit $i".log()
            emit(i)
        }
    }.map { value ->
        check(value <= 1) {
            "crashed on $value"
        }
        "mapValue $value"
    }
    runBlocking {
        f.catch { e->
            emit("不小心捕获了个异常 $e")
        }.collect { s->
            "collect $s".log()
        }
    }
    //emit 1
    //collect mapValue 1
    //emit 2
    //collect 不小心捕获了个异常 java.lang.IllegalStateException: crashed on 2
```
collect发生异常无法捕获到
```kotlin
 val f = flow {
        for (i in 1..3) {
            "emit $i".log()
            emit(i)
        }
    }
    runBlocking {
        f.catch { e ->
            "不小心捕获了个异常 $e".log()
        }.collect { i ->
            "collect $i".log()
            check(i <= 1) {
                "crashed on $i"
            }
        }
    }
    //emit 1
    //collect 1
    //emit 2
    //collect 2
    //Exception in thread "main" java.lang.IllegalStateException: crashed on 2
```
兼并上述两种方法
```kotlin
  val f = flow {
        for (i in 1..3) {
            "emit $i".log()
            emit(i)
        }
    }
    runBlocking {
        f.onEach { value ->
            check(value <= 1) {
                "crashed on $value"
            }
            "collect $value".log()
        }.catch { e ->
            "不小心捕获了个异常 $e".log()
        }.collect()
    }
    //emit 1
    //collect 1
    //emit 2
    //不小心捕获了个异常 java.lang.IllegalStateException: crashed on 2
```
# completion
## finally
```kotlin
  val f = flowOf(1, 2, 3)
  runBlocking {
    try {
        f.collect { value->
            "collect $value".log()
        }
    } finally {
        "complete".log()
       }
    }
    //collect 1
    //collect 2
    //collect 3
    //complete
```
## onCompletion
**onCompletion接收一个 Throwable 的nullable参数，可以用这个判断Flow是正常结束还是发生了异常。**
```kotlin
 val f = flowOf(1, 2, 3)
 runBlocking {
    f.onCompletion { "complete".log() }
        .collect { value -> "collect $value".log() }
    }
    //collect 1
    //collect 2
    //collect 3
    //complete
```

```kotlin
   val f = flow {
        emit(1)
        throw RuntimeException()
    }
    runBlocking {
        f.onCompletion { cause ->
            if (cause != null) {
                "Flow completed exceptionally".log()
            }
        }
            .catch { cause ->
                "Caught exception $cause".log()
            }
            .collect { value -> "collect $value".log() }
    }
    //collect 1
    //Flow completed exceptionally
    //Caught exception java.lang.RuntimeException
```
**不同于`catch`，`onCompletion `接收所有异常，如果接收得参数为null则代表Flow正常结束。**
```kotlin
 val f = flowOf(1, 2, 3)
    runBlocking {
        f.onCompletion { cause -> println("Flow completed with $cause") }
            .collect { value ->
                check(value <= 1) {
                    "crashed on $value"
                }
                "collect $value".log()
            }
    }
    //collect 1
    //Flow completed with java.lang.IllegalStateException: crashed on 2
    //Exception in thread "main" java.lang.IllegalStateException: crashed on 2
```
