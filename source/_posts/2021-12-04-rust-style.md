---
title: Kotlin实现Rust风格的Result
date: 2021-12-04 20:30
tags:
- Kotlin
---

# 背景
前段时间看到rust的错误处理方式，觉得十分优雅，于是就想能不能用Kotlin模仿一个版本。
<!--more-->
# 实现
先看原版
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```
然后看看我用Kotlin实现的版本
```kotlin
fun main() {
    val fileUtil = FileUtil()
    val result = fileUtil.openFile("error")
    val content = result match {
        OK { str ->
            str
        }
        Error { error ->
            throw error
        }
    }
    println(content)
}
```
在语法结构上看起来已经十分接近了，可惜的是最后返回的content是可用类型，在后续使用的时候必须带上`!!`或者`?:`操作符，
下面我们来看看如何通过Kotlin的语法实现这样的错误处理。

首先Kotlin没有`match`关键字，但是Kotlin有infix函数，可以在语法上形成rust这样的视觉效果。OK和Err分支操作的实现可以通过`sealed class`基类和子类完成，在返回`KResult`的函数中根据具体逻辑返回OK或者Err。
下面是具体代码
```kotlin
sealed class KResult<T, E : Throwable> {
    fun isOk(): Boolean = this is OK
    fun isError(): Boolean = this is Error

    fun <T, E : Throwable> KResult<T, E>.OK(block: (T) -> T): T? {
        return block((this as OK).data)
    }

    fun <T, E : Throwable> KResult<T, E>.Error(block: (E) -> Unit): T? {
        this as Error
        block(this.error)
        return data
    }
}

class OK<T, E : Throwable>(val data: T): KResult<T, E>() {}

class Error<T, E : Throwable>(val data: T?, val error: E): KResult<T, E>() {}

infix fun<T, E: Throwable> KResult<T, E>.match(block: KResult<T, E>.() -> T): T? {
    if (this.isOk()) {
        return (this as OK).data
    }
    throw (this as Error).error
}

```
函数中的使用如下
```kotlin
fun openFile(fileName: String): KResult<String?, Throwable> {
    if (fileName == "error") return Error(null, IOException("io exception"))
    return OK("content")
}
```