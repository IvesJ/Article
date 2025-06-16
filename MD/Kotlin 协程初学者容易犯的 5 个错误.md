> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1kdY2NMiWSWAKAhcVxVDiQ)

引言
==

对于安卓开发者来说，Kotlin 协程已成为日常工作流程中不可或缺的一部分。它们让异步编程和并发处理变得轻松自如，甚至让人觉得过于轻松了。但这种简便性可能具有欺骗性，许多开发者不知不觉地陷入了一些看似是最佳实践，实则微妙且危险的陷阱。

1. View 中直接调用挂起函数
=================

在 View 中直接调用挂起函数可能会导致严重的生命周期问题和主线程阻塞问题，这在 MVP（Model-View-Presenter）架构中尤为常见。因为开发者常经常在 ViewModel 中定义挂起函数，在 View 中调用它们获取数据并更新 UI 状态。例如：

```
class MyViewModel(private val repository: MyRepository) : ViewModel() {

   suspend fun fetchUserData() {
        val user = repository.getUser() ❌ 
    }
}


```

这可能会带来以下问题：

*   **阻塞主线程**：如果挂起函数执行耗时任务却没有切换到后台线程（例如没有使用 Dispatchers.IO），它就会阻塞主线程，导致卡顿或 ANR（应用无响应）错误。
    
*   **生命周期问题**：ViewModel 会跨越 UI 配置变化保存状态。但如果协程没有正确作用域（比如使用 viewModelScope），即使 UI 已销毁，协程可能仍在继续执行，从而导致内存泄漏或资源浪费。
    
*   **结果丢失**：如果协程绑定的 ViewModel 被清除（例如，当 Activity 或 Fragment 结束时），那么协程会被立刻取消，其结果永远无法到达 UI。
    

上面代码正确的写法如下

```
class MyViewModel(private val repository: MyRepository) : ViewModel() {

    private val _user = MutableLiveData<User>()
    val user: LiveData<User> get() = _user

    fun fetchUserData() {
        viewModelScope.launch {
            try {
                val result = repository.getUser() // ✅ Safe coroutine call
                _user.postValue(result)  // ✅ Updates UI on the Main thread
            } catch (e: Exception) {
                // Handle error properly
            }
        }
    }
}


```

`fetchUserData` 函数使用 `viewModelScope.launch` 在 ViewModel 的作用域内安全地执行协程。结果通过 LiveData 传递给 UI，即使 ViewModel 重建， LiveData 的状态也有法子继续保持

2. 错误使用 GlobalScope
===================

当使用协程时，错误使用 GlobalScope 会导致内存泄漏、后台任务难以管理以及崩溃等严重问题。

```
GloablScope.launch{
  delay(400)
  Log.d("Global scope", "Task Successful")
}


```

GlobalScope 存在以下问题

*   **无法取消**：一旦启动，GlobalScope 中的协程就会独立于应用的生命周期运行。如果用户离开应用，协程可能仍在后台运行，从而潜在地浪费资源。
    
*   **忽视生命周期感知**：与 viewModelScope 不同，GlobalScope 不会在 ViewModel 被清除时取消协程。这意味着在已销毁的 UI 上尝试更新数据时，可能会导致崩溃和意外行为。
    
*   **缺少自定义调度器或异常处理**：GlobalScope.launch 始终默认使用 Dispatchers.Default，这可能并非网络请求（应在 Dispatchers.IO 上运行）的最佳选择。此外，GlobalScope 中未处理的异常可能会使整个应用崩溃，因为没有结构化的并发机制来妥善处理这些异常。
    

为确保正确的生命周期管理，应始终使用 `viewModelScope.launch`，而不是 `GlobalScope.launch`。

```
viewmodelscope.launch{
  delay(400)
  Log.d("viewmodel", "Task Successful")
}


```

3. 没有并行执行
=========

Kotlin 协程的另一个常见陷阱就是在可以并行执行网络调用时却按顺序执行。这会导致显著的长时间执行，造成不必要的延迟，进而影响用户体验。

```
suspend fun mistakeGetCarNames(ids: List<Int>): List<String> {
    val names = mutableListOf<String>()
    for (id in ids) {
        names.add(getCarNameById(id)) ❌ // Wrong: Calls run one after another, wasting time!
    }
    return names
}


```

上面的代码需要一个接一个的执行 `getCarNameById`，造成以下问题

*   每个请求相互独立：如果获取函数 A 不依赖于获取函数 B，顺序运行它们是一种浪费，会不必要地增加总执行时间 。
    
*   网络调用耗时：依次运行多个 1 秒钟的调用会线性增加时间（5 次调用 = 5 秒钟）。
    
*   用户体验受影响：应用等待时间过长，会导致用户感到沮丧。
    

正确的写法应该是这样

```
suspend fun getCarNames(ids: List<Int>): List<String> {
    return coroutineScope { 
        ids.map { id -> 
            async { getCarNameById(id) } // ✅ Runs all requests in parallel!
        }.awaitAll() // ✅ Waits for all coroutines to complete and returns results
    }
}


```

我们可以使用 coroutineScope 中的 async 函数并发获取所有数据。

*   并发运行所有网络调用，而不是逐个运行。
    
*   async 允许在相同时间内启动所有调用。例如，如果每个调用耗时 1 秒，有 5 个调用，总时间约为 1 秒，而不是 5 秒，这让应用响应更快。
    

4. 没有正确捕获异常
===========

捕获取消异常后没有正确处理，从而导致意外行为，甚至在协程应停止时仍继续运行。这会导致内存泄漏和资源浪费。如下：

```
suspend fun mistakeRiskyTask() {
    try {
        delay(3_000L) // Simulating long-running operation
        val error = 10 / 0  // ❌ This will throw an ArithmeticException
    } catch (e: Exception) {
        println("iyke: code failed") ❌ // Catching all exceptions, including CancellationException!
    }
}


```

当协程因生命周期事件（如 Activity 销毁）而被取消时：

*   如果捕获此异常但不重新抛出，父协程将不知道它已被取消。
    
*   可能会导致内存泄漏、UI 卡顿以及不必要的后台处理。
    

```
suspend fun riskyTask() {
    try {
        delay(3_000L) // Simulating long-running operation
        val error = 10 / 0  // ❌ This will throw an ArithmeticException
    } catch (e: Exception) {
        if (e is CancellationException) throw e // ✅ Rethrow CancellationException!
        println("Emmanuel iyke: code not working") // ✅ Handle other exceptions properly
    }
}


```

修改方法如上：

*   始终在 catch 块中重新抛出 CancellationException，以便父协程知晓取消情况。
    
*   分别处理其他异常（如 IOException、ArithmeticException ）。
    

5. 执行长时间任务但为检查协程取消状态
====================

在执行长时间任务时未检查协程是否仍处于活动状态。

```
suspend fun mistakeDoSomething() {
    val job = CoroutineScope(Dispatchers.Default).launch {
        var random = Random.nextInt(100_000)
        while (random != 50_000) {  // ❌ No check for cancellation
            println("Random: $random")
            random = Random.nextInt(100_000)
        }
    }
    println("Random: our job cancelled")
    delay(500L)
    job.cancel() // ❌ This won't immediately stop the loop
}


```

问题如下：

*   如果协程已被取消，但循环仍在运行，就会导致不必要的处理，浪费处理能力。
    
*   协程在不再需要时仍驻留在内存中，可能导致内存泄漏。
    
*   如果协程正在处理 UI 更新，这可能会导致 UI 卡顿或应用崩溃，产生意外行为。
    

```
suspend fun doSomethingWithIsActive() {
    val job = CoroutineScope(Dispatchers.Default).launch {
        var random = Random.nextInt(100_000)
        while (isActive && random != 50_000) { // ✅ Manually check if coroutine is active
            println("Random: $random")
            random = Random.nextInt(100_000)
        }
    }
    println("Random: our job cancelled")
    delay(500L)
    job.cancel() // ✅ Coroutine will now stop properly
}


```

争取的处理方式是使用协程的内置属性 `isActive`。 在循环内检查它，如果协程被取消，立即抛出 `CancellationException` 停止循环。

-- END --

推荐阅读

*   [Kotlin 协程切勿滥用 Dispatchers.IO](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247498832&idx=1&sn=78d9e7c6f67b2a3994d86c593ae0824a&scene=21#wechat_redirect)
    
*   [Compose 经验分享：开发要点 & 常见错误 & 面试题](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247494728&idx=1&sn=c905a45f78b77480548e057a96f8891c&scene=21#wechat_redirect)
    
*   [Kotlin 协程的并发安全与最佳实践](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247497717&idx=1&sn=dbbd8d2441422ae60d19813f13b75ba4&scene=21#wechat_redirect)