# 步骤06：实现实时游戏机制

在本课中，您将学习如何：

- 使用goroutine
- 使用匿名函数（lambda）
- 使用channel
- 使用select语句异步读取channel
- 使用time包

## 概述

目前我们的游戏已经有了基本框架：明确的目标、胜利和失败条件，以及正常的玩家输入控制。

但存在一个主要问题：敌人只在玩家移动时才移动。这看起来不太像真正的游戏，让我们来解决这个问题。

这个问题是因为读取输入是一个阻塞操作。我们需要以某种方式使其异步...如果我们有Go的异步功能就好了...等等！我们确实有！

这就是神奇的channel和goroutine登场的时候了！

Goroutine类似于线程，但更加轻量级。在底层，Go运行时会创建线程来管理goroutine，但一个线程可以管理多个goroutine，所以它们的关系不是简单的1:1。

但最棒的部分是：Go语言设计使得创建goroutine非常简单——只需在函数调用前添加`go`关键字，函数就会在单独的goroutine中异步运行。

看看下面的代码：

```go
func main() {
    go fmt.Println("hello")
    go fmt.Println("world")
}
```

这段代码有三个goroutine：第一个运行`main`函数，第二个打印"hello"，第三个打印"world"。

关于goroutine的一个重要特性是：由于操作是异步的，可以确定上面的程序不会产生任何输出。因为`main`函数很可能在两个goroutine执行前就结束了（因为启动goroutine有一些开销）。

我们可以给main函数添加延迟：

```go
func main() {
    go fmt.Println("hello")
    go fmt.Println("world")
    time.Sleep(100 * time.Millisecond)
}
```

这样可以确保goroutine执行，因为我们预期它们能在100ms内完成，但程序的输出顺序仍然不可预测，因为我们无法控制goroutine的执行顺序。

除了goroutine，我们还有channel。channel允许我们通过传递或接收值与goroutine通信。

创建channel使用内置的`make`函数：

```go
ch := make(chan int)
```

每个channel都有类型，还可以有缓冲区大小。如果未指定大小，默认为1。

对channel的读写可能是阻塞操作，除非channel为空。

使用箭头操作符向channel写入：

```go
// 向ch写入内容
ch <- something
```

如果ch为空，操作不会阻塞；如果已满，则会阻塞直到channel另一端被消费。

同样，从channel读取也使用箭头操作符：

```go
// 在另一个goroutine中
foo := <-ch
```

设计异步处理时，必须注意避免两个goroutine互相依赖导致阻塞或产生不一致结果。更多关于死锁和竞态条件的信息，请参考[StackOverflow上的这个回答](https://stackoverflow.com/a/3130212/4893628)。

## 任务01：重构输入代码

现在了解了goroutine和channel的基础知识，让我们看看实际应用。首先，从游戏循环中移除输入处理代码，在循环开始前插入以下代码：

```go
func main() {
    // 初始化代码省略...

    // 异步处理输入
    input := make(chan string)
    go func(ch chan<- string) {
        for {
            input, err := readInput()
            if err != nil {
                log.Println("读取输入错误:", err)
                ch <- "ESC"
            }
            ch <- input
        }
    }(input)

    // 游戏循环
    for {
        // 循环代码...
    }
}
```

这段代码创建一个名为`input`的channel，并将其作为参数传递给用`go`语句调用的匿名函数。这是Go中异步处理的常见模式。

匿名函数创建一个无限循环，等待输入并将其写入channel`ch`（函数参数）。如果出错，就返回"ESC"代码，因为我们知道这会终止程序。

在游戏循环中，我们将替换玩家移动处理的代码：

```go
// 处理移动
select {
case inp := <-input:
    if inp == "ESC" {
        lives = 0
    }
    movePlayer(inp)
default:
}
```

可以把select语句想象成switch语句，但是用于channel。这个select语句具有非阻塞特性，因为它有default子句。这意味着如果`input` channel有内容可读就会被读取，否则执行`default`case（这里是一个空块）。

最后，由于我们将"ESC"逻辑移到了上面的代码块中，需要从游戏结束条件中移除它（因为`lives <= 0`已经包含了这个条件）。

我们还将引入200ms的延迟。因为现在不再等待输入，没有延迟游戏会运行得太快。相关代码如下：

```go
    // 更新屏幕
    printScreen()

    // 检查游戏结束
    if numDots == 0 || lives <= 0 {
        break
    }

    // 重复
    time.Sleep(200 * time.Millisecond)
```

现在尝试运行游戏。是不是更有趣了？ :)

[带我去步骤07！](../step07/README.md)