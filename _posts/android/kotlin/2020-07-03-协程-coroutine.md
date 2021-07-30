---
layout:     post
title:      " kotlin-协程"
subtitle:   "coroutine"
date:       2020-07-03
author:     "zhouhaoh"
catalog: true
tags:
- Kotlin
---

## 协程

> 挂起(suspend)   --> 恢复(resume)、用同步的代码到达异步的效果。

### 协程分类
1. 按调用栈

   |                           有栈协程                           |                           无栈协程                           |
   | :----------------------------------------------------------: | :----------------------------------------------------------: |
   | 每个协会分配单独的调用栈，类似线程的调用栈，可以再任意函数嵌套中挂起 | 不会分配单独的调用栈，挂起点状态通过闭包或对象保存，只能在当前函数中挂起 |

2. 按调用关系

   |                   对称协程                   |                    非对称协程                    |
   | :------------------------------------------: | :----------------------------------------------: |
   | 调度权可以转接给任意协程、协程之前是对等关系 | 调度权只能转移给调用自己的协程，协程存在父子关系 |

### 基本要素

#### 挂起函数

```kotlin
suspend fun xxx(){
    
}
```

挂起函数只能在其他挂起函数或者协程中调用，通过*Continuation*实现， 这种转换叫：CPS 转换(Continuation-Passing-Style Transformation)。

```kotlin
public interface Continuation<in T> {
    /**
     * The context of the coroutine that corresponds to this continuation.
     */
    public val context: CoroutineContext

    /**
     * Resumes the execution of the corresponding coroutine passing a successful or failed [result] as the
     * return value of the last suspension point.
     */
    public fun resumeWith(result: Result<T>)
}

```

类型: **suspend()->Unit**,通过编译器传入**Continuation**实现回调的效果：

```kotlin
fun foo(continuation: Continuation<Unit>): Any{}

fun bar(a: Int, continuation: Continuation<String>): Any{
    return "Hello"
}

```

#### kotlin标准库

1. 协程上下文 CoroutineContext：协程执行过程中需要携带数据，类似List 。

   ```kotlin
   public interface CoroutineContext {
       /**
        * Returns the element with the given [key] from this context or `null`.
        */
       public operator fun <E : Element> get(key: Key<E>): E?
   
       /**
        * Accumulates entries of this context starting with [initial] value and applying [operation] 
        * from left to right to current accumulator value and each element of this context.
        */
       public fun <R> fold(initial: R, operation: (R, Element) -> R): R
       //添加元素
       public operator fun plus(context: CoroutineContext): CoroutineContext 
   }
   ```

2. 拦截器 是一类协程上下文元素.类似Retrofit拦截器。主要作用切换线程。

   

3. 挂起函数

####  协程框架

1. Job

2. 调度器

   调度器是上下文的实现。

3. 作用域