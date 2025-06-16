> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/bfIScKy9qKYtrO-vi3BQTQ)

当你的数据集体 "撞衫" 怎么办？

你整理了一筐水果，结果发现苹果混进去了三次；你统计用户订单，结果重复数据像复制粘贴一样出现... 这时候就需要请出`Kotlin`家族的 "去重小能手"—`distinct`！它能像筛子一样过滤重复项，还你干净整洁的数据集~

### 初识 distinct：基础去重三连招

#### 🎯 第一式：简单粗暴去重法

```
fun main() {
    // 水果采购清单（不小心扫了三次苹果）
    val fruits = listOf("🍎", "🍌", "🍎", "🍊", "🍇", "🍌", "🍎",)
    
    // 使用魔法咒语 distinct!
    val uniqueFruits = fruits.distinct()
    
    println("原始清单：$fruits")   // [🍎, 🍌, 🍎, 🍊, 🍇, 🍌, 🍎]
    println("去重结果：$uniqueFruits") // [🍎, 🍌, 🍊, 🍇]
}

```

**效果说明**：就像超市扫码器遇到重复商品会 "滴" 一声跳过，`distinct`会保留每种元素的第一次出现，后续重复都会被无情过滤！

#### 🎯 第二式：数组特攻版

```
fun main() {
    // 员工工号数组（有两个007重复）
    val employeeIds = arrayOf(101, 007, 102, 007, 103) // 改用普通数组
    
    // 正确姿势：先转为List再施法
    val uniqueIds = employeeIds.toList().distinct()
    
    println("原始工号：${employeeIds.contentToString()}") // [101, 7, 102, 7, 103]
    println("去重结果：$uniqueIds")                     // [101, 7, 102, 103]
}

```

**注意**：数组需要先转为`List`才能使用`distinct`哦~

*   • 普通数组（Array<T> 类型）需要先通过`toList()`转换
    
*   • 基本类型数组（如`IntArray`）可直接使用`distinct()`，因为`Kotlin`偷偷给它们加了超能力✨
    

#### 🎯 第三式：对象精准去重术

```
// 社交媒体好友数据类
dataclassSocialFriend(
    val nickname: String,
    val account: String
)

fun main() {
    // 好友列表（两个相同账号的重复关注）
    val friends = listOf(
        SocialFriend("码农小明", "coder_123"),
        SocialFriend("产品经理小王", "pm_007"),
        SocialFriend("测试小美", "coder_123") // 重复账号！
    )
    
    // 按账号去重的魔法
    val realFriends = friends.distinctBy { it.account }
    
    println("真实好友：$realFriends") 
    // 只会保留第一个"coder_123"对应的对象
}

```

**小贴士**：`distinctBy`就像智能过滤器，可以指定对象的某个特征作为去重依据！

### 高阶玩法：这些冷知识你知道吗？

#### 🔥 冷知识 1：去重也要讲武德

`distinct`会严格保持元素出现的原始顺序，就像排队买奶茶时，黄牛插队会被识别出来踢出去，但其他人的顺序不变

#### 🔥 冷知识 2：超能力组合技

```
fun main() {
    // 混合类型列表（数字和字符串）
    val mixList = listOf(1, "1", 2, "2", 3, "3")
    
    // 先过滤字符串再转换数字
    val numbers = mixList.filterIsInstance<Int>()
                        .distinct()
                        .sortedDescending()
    
    println("混合处理结果：$numbers") // [3, 2, 1]
}

```

**组合技解析**：`distinct`可以和`filter`、`map`等操作链式调用，实现复杂的数据清洗流程！

### 避坑指南：这些情况要注意！

1.  1. **对象比较陷阱**：普通对象默认比较内存地址，要用`data class`或重写`equals`方法
    
2.  2. **大小写敏感**："Apple" 和 "apple" 会被认为是不同元素，需要预处理统一大小写
    
3.  3. **性能问题**：超大数据集建议使用`toSet()`转换，效率更高
    

### 实战演练：做个智能购物车

```
// 购物车商品类
dataclassCartItem(
    val sku: String,      // 商品唯一编号
    val name: String,
    var quantity: Int
)

fun main() {
    // 模拟用户重复添加商品
    val cart = listOf(
        CartItem("1001", "无线耳机", 1),
        CartItem("1002", "充电宝", 2),
        CartItem("1001", "无线耳机", 1) // 手抖重复添加
    )
    
    // 终极去重方案
    val cleanCart = cart.distinctBy { it.sku }
                       .map { it.copy(quantity = 1) } // 重置数量
    
    println("智能购物车：$cleanCart")
}

```

**运行结果**：自动合并重复商品，并重置数量为 1，有效防止重复下单！

掌握了这些`distinct`的妙用，从此告别重复数据烦恼！如果还有其他`Kotlin`黑科技想了解，欢迎留言告诉我~ 🎉

[Kotlin 数据类实战手册：让代码少写一半的秘诀](https://mp.weixin.qq.com/s?__biz=Mzg5MzIyMDczMw==&mid=2247488077&idx=1&sn=9ba58196486b76c36f7762893e532d14&scene=21#wechat_redirect)

[给代码开外挂：用 Kotlin 扩展函数化身编程乐高大师](https://mp.weixin.qq.com/s?__biz=Mzg5MzIyMDczMw==&mid=2247487999&idx=1&sn=ffd7379bad0e85b0fcc8356f5f810bad&scene=21#wechat_redirect)

[Kotlin 里两个偷懒神器：lateinit 和 by lazy 简单易懂解释](https://mp.weixin.qq.com/s?__biz=Mzg5MzIyMDczMw==&mid=2247487986&idx=1&sn=1402407cffa55f5a2f162633a9809448&scene=21#wechat_redirect)