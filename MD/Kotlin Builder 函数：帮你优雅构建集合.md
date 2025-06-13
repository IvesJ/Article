> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247499064&idx=1&sn=e279059584483492f7708d52b8324d6b&chksm=c16f5f057288d9b27d6904a466d4f9be5877fcc074951a5b062b481c1e33c4b16e21c95c5572&mpshare=1&srcid=0613HXn16qEwDbphA4Kvh5No&sharer_shareinfo=b2be074f89acd7387cc78ff0ec189811&sharer_shareinfo_first=d8c6a9e43784cf5df1d0e555a941346a&from=singlemessage&scene=1&subscene=317&clicktime=1749779080&enterid=1749779080&sessionid=0&ascene=1&fasttmpl_type=0&fasttmpl_fullversion=7773940-zh_CN-zip&fasttmpl_flag=0&realreporttime=1749779080609&devicetype=android-31&version=28003c39&nettype=WIFI&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQdlmFrhe7EnxoSLJ%2FTZcL%2BhLjAQIE97dBBAEAAAAAANKlCjlkUo4AAAAOpnltbLcz9gKNyK89dVj0CtvpWEsRpEb3qTWiaVFA3j1anINC%2Fk%2FqeA2a%2FgmAcZz%2BJjtJWJZdNmnoDrhY9cnXTkkKsMQ8JfVcsx88bbSc%2Bc8SN%2FvO2U69ShDQalpcidJKwdwTK2LcpgiUqRosf4Um3ToElcFSqeehq82H5aSOqKkEUlw9k8hsRGj2amzoMayBZw6uW6PJdGQ%2BW2av3qUxIqCm2eDezPE4KfzyE1wj2FJ8sQi5gxDwEo2%2BewO2kLwI%2FY5PhK0hlXj7jbFX&pass_ticket=IwbkK5%2Bbzli%2BjfwT0T3xJF%2BKwtL1HXO%2FoCs6s03uq4xESqnIQ32lMxNQ77J3O5vk&wx_header=3)

前言
==

很多开发者不知道 Kotlin 为 List，Map，Set 等集合类的构造提供了若干便捷函数，让我们来看看吧

创建 List
=======

以下是创建并填充 MutableList 的常见做法：

```
val newList = mutableListOf<Int>()
newList.add(1)
if (conditionFullfilled) {
    newList.add(2)
}

// OR

val newList = mutableListOf<Int>().apply { 
    add(1)

    if (conditionFullfilled) {
        add(2)
    }
}


```

其实我们可以用 `buildList { }` 函数简化操作，它在底层创建一个 MutableList，允许我们对其调用函数，最后返回一个 ImmutableList。像这样：

```
val newList = buildList { 
    add(1)

    if (conditionFullfilled) {
        add(2)
    }
}


```

看看 `buildList()` 函数的实现，它的工作方式和我们最初的代码类似。它调用 `buildListInternal()` 函数来构造新的 MutableList 。且因为它是内联函数，所以由于 lambda 带来的额外开销也被消除了，因为代码会在编译时在调用位置展开，这对于关注性能的程序很重要。

```
@SinceKotlin("1.6")
@WasExperimental(ExperimentalStdlibApi::class)
@kotlin.internal.InlineOnly
@Suppress("LEAKED_IN_PLACE_LAMBDA", "WRONG_INVOCATION_KIND")
public inline fun <E> buildList(@BuilderInference builderAction: MutableList<E>.() -> Unit): List<E> {
    contract { callsInPlace(builderAction, InvocationKind.EXACTLY_ONCE) }
    return buildListInternal(builderAction)
}


```

`buildList()` 可用于任何类型，包括 Int、Float、Double、Long 等基本类型。不过，如果想在存储和检索元素时避免自动装箱，我们可以使用专门的函数：

*   `buildIntList { }`：构造 MutableIntList 并返回 IntList
    
*   `buildLongList { }`：构造 MutableLongList 并返回 LongList
    
*   `buildFloatList { }`：构造 MutableFloatList 并返回 FloatList
    
*   `buildDoubleList { }`：构造 MutableDoubleList 并返回 DoubleList
    

创建字符串
=====

构造需要基于某些条件进行拼接的字符串时，典型做法是创建 `StringBuilder` 对象，调用 `append` 函数，再转成字符串。

```
val sb = StringBuilder()
sb.append("Some")
if (conditionFullfilled) {
    sb.append(" string")
}
val newString = sb.toString()

// OR

val newString = StringBuilder().apply { 
    append("Some")

    if (conditionFullfilled) {
        append(" string")
    }
}.toString()


```

我们可以用 `buildString { }` 函数简化，它会为我们创建 `StringBuilder` 对象并返回最终字符串。在代码块内，我们可以调用常规的 append 函数。

```
val newString = buildString {
    append("Some")

    if (conditionFullfilled) {
        append(" string")
    }
}


```

看看 `buildString()` 函数的实现，它的工作原理和我们最初的代码一样。由于是内联函数，额外的 lambda 开销也被消除了。

```
@kotlin.internal.InlineOnly
public inline fun buildString(builderAction: StringBuilder.() -> Unit): String {
    contract { callsInPlace(builderAction, InvocationKind.EXACTLY_ONCE) }
    return StringBuilder().apply(builderAction).toString()
}


```

创建 Set
======

和创建列表、字符串类似，Kotlin 有辅助函数创建集合。它构造一个可变集合，允许我们对其调用函数，之后返回保留插入顺序的不可变集合版本。

```
val newSet = buildSet {
    add(1)

    if (conditionFullfilled) {
        addAll(someOtherSet)
    }
}


```

`buildSet()` 函数可用于任何类型。不过，也有针对特定类型集合的其他版本函数：

*   `buildIntSet { }`：构造 MutableIntSet 并返回 IntSet
    
*   `buildFloatSet { }`：构造 MutableFloatSet 并返回 FloatSet
    
*   `buildLongSet { }`：构造 MutableLongSet 并返回 LongSet
    

这三种类型限定的集合底层使用扁平哈希表，不保留插入顺序。

创建 Map
======

我们可以用 `buildMap { }` 函数创建新映射。它在底层构造新的可变映射，允许我们对其调用函数，最后返回不可变映射。

```
val newMap = buildMap { 
    put("key", "value")
    
    if (conditionFullfilled) {
        putAll(someOtherMap)
    }
}


```

如果键或值是 Int、Float、Double 或 Long 等基本类型，我们可以使用优化的函数变体：

*   `buildIntIntMap { }`：构造 MutableIntIntMap 并返回 IntIntMap，键和值都是 Int 基本类型
    
*   `buildLongLongMap { }`：构造 MutableLongLongMap 并返回 LongLongMap，键和值都是 Long 基本类型
    
*   `buildFloatFloatMap { }`：构造 MutableFloatFloatMap 并返回 FloatFloatMap，键和值都是 Float 基本类型
    
*   `buildObjectObjectMap { }`：构造 MutableObjectObjectMap 并返回 ObjectObjectMap，键和值是引用类型
    

针对键值类型的不同组合，还有更多变体函数，比如 `buildIntFloatMap { }`、`buildIntLongMap { }`、`buildObjectFloatMap { }` 等等。

其他构建器
=====

上述函数都属于 Kotlin 标准库，在大多数项目中都能使用。其他依赖库中也有类似的构建器函数，比如 Compowe 中的 `buildSpannedString { }` 和 `buildAnnotatedString { }`，Kotlinx Serialization 库中的 `buildJsonObject` 等等。

总结
==

Kotlin 标准库为构建 List、Map、Set、String 等常见类型，提供了若干便捷函数。它们给出了构建这些类型的标准化方式，避免了常规的样板代码。

-- END --

推荐阅读

*   [Kotlin 协程初学者容易犯的 5 个错误](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247498963&idx=1&sn=6bd070cb0722a5559273469c07349cf8&scene=21#wechat_redirect)
    
*   [性能优化：ArrayList#removelAll 竟如此耗时?](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247496607&idx=1&sn=12f24723bf6473c60b955ee52b2713ba&scene=21#wechat_redirect)
    
*   [Kotlin 协程切勿滥用 Dispatchers.IO](https://mp.weixin.qq.com/s?__biz=Mzg5MzYxNTI5Mg==&mid=2247498832&idx=1&sn=78d9e7c6f67b2a3994d86c593ae0824a&scene=21#wechat_redirect)