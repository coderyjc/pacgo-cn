# 第一步：输入与输出

在本课程中，你将学习：

- 从文件读取数据
- 打印到标准输出
- 处理多返回值
- 错误处理
- 创建切片并添加元素
- 使用range遍历切片
- 延迟函数调用(defer)
- 错误日志记录

## 概述

基础知识已经掌握，现在正式开始开发游戏！

首先我们要读取迷宫数据。我们有一个名为`maze01.txt`的文件，它本质上是迷宫的ASCII表示（你可以用文本编辑器打开查看）。文件中的符号含义如下：

```
- # 表示墙壁
- . 表示豆子
- P 表示玩家
- G 表示幽灵(敌人)
- X 表示能量药丸
```

我们的第一个任务是将这个ASCII迷宫加载到字符串切片中，然后打印到屏幕上。看起来很简单对吧？确实如此！

## 任务01：加载迷宫

让我们从读取`maze01.txt`文件开始。

我们将使用`os`包的`Open`函数打开文件，使用缓冲IO包(`bufio`)的 scanner 对象将其读取到内存（存入名为`maze`的全局变量）。最后需要通过调用`os.Close`释放文件句柄。

完整代码如下：

```go
var maze []string

func loadMaze(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }

    return nil
}
```

现在让我们分解这段代码看看发生了什么。

注意需要导入`bufio`和`os`包：

```go
import "bufio"
import "os"
```

或者，如果已经导入了`fmt`，可以写成列表形式：

```go
import (
    "bufio"
    "fmt"
    "os"
)
```

`os.Open()`函数返回两个值：文件对象和错误。在Go中函数返回多个值是常见模式，特别是用于返回错误。

```go
f, err := os.Open(file)
```

`:=`是赋值操作符，其特殊之处在于能根据右侧值自动推断变量类型。

记住Go是强类型语言，但这个特性让我们在类型可推断时无需显式声明。

上面例子中，Go会自动推断`f`和`err`的类型。

当函数返回错误时，立即检查错误是常见模式：

```go
    f, err := os.Open(file)
    if err != nil {
        // 处理错误
        log.Print("...")
        return
    }
```

注意：将“正常路径”保持左对齐，“错误路径”向右缩进（即尽早终止函数）。

> 译注：
> 正常路径（happy path）指函数的主要逻辑，即一切正常时的执行流程。应尽量保持靠左（不嵌套或少嵌套），使读者能快速理解核心逻辑。
> 错误路径（bad path）指错误处理、边界检查等非正常流程。应尽早返回（靠右），减少嵌套深度。

Go中的`nil`表示变量未赋值。
 
`if`语句在条件为真时执行代码块。它可以像`for`语句一样包含初始化子句，以及条件为假时执行的`else`子句。注意变量的作用域仅限于if语句体内。例如：

```go
// 可选的初始化子句
if foo := rand.Intn(2); foo == 0 {
    fmt.Print(foo) // foo在此有效
} else {
    fmt.Print(foo) // 这里也有效
}
// 但这里不能使用foo！
```

`loadMaze`代码另一个有趣之处是`defer`关键字的使用。它表示将函数调用推迟到当前函数结束时执行。这对清理工作非常有用，这里我们用它来关闭刚打开的文件：

```go
func loadMaze(file) error {
    f, err := os.Open(file)
    // 省略错误处理
    defer f.Close() // 将f.Close()放入调用栈

    // 其余代码

    return nil
    // f.Close会被隐式调用
}
```

接下来的代码逐行读取文件并追加到maze切片：

```go
    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }
```

scanner是读取文件方便的方式。`scanner.Scan()`在文件还有内容可读时返回true，`scanner.Text()`返回下一行输入。

内置函数`append`负责向`maze`切片添加新元素。

## 任务02：打印到屏幕

将迷宫文件加载到内存后，我们需要将其打印到屏幕。

一种方法是遍历`maze`切片的每个元素并打印。这可以方便地使用`range`操作符完成：

```go
func printScreen() {
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

注意我们使用`:=`赋值操作符初始化两个值：下划线(_)和`line`变量。下划线只是编译器预期变量名的占位符。使用下划线表示我们忽略该值。

对于`range`操作符，第一个返回值是元素的索引（从零开始），第二个返回值是元素值。

如果我们不写下划线字符来忽略第一个值，range操作符将只返回索引（而不返回值）。例如：

```go
for idx := range maze {
    fmt.Println(idx)
}
```

由于本例中我们只关心内容而非索引，可以安全地用下划线忽略索引。

## 任务03：更新游戏循环

现在我们有了`loadMaze`和`printScreen`函数，应该更新`main`函数来初始化迷宫并在游戏循环中打印它。代码如下：

```go
func main() {
    // 初始化游戏

    // 加载资源
    err := loadMaze("maze01.txt")
    if err != nil {
        log.Println("加载迷宫失败:", err)
        return
    }

    // 游戏循环
    for {
        // 更新屏幕
        printScreen()

        // 处理输入

        // 处理移动

        // 处理碰撞

        // 检查游戏结束

        // 临时：中断无限循环
        break

        // 重复
    }
}
```

一如既往地，我们保持正常路径左对齐，所以如果`loadMaze`函数失败，我们使用`log.Println`记录并`return`终止程序执行。由于使用了新包`log`，请确保它已添加到导入部分：

```go
import (
    "bufio"
    "fmt"
    "log"
    "os"
)
```

一些IDE（如`vscode`）可以配置为自动完成此操作。

注意：也可以使用`log.Fatalln`达到同样效果，但需要确保在退出`main`函数前执行所有延迟调用，而`log.Fatal`系列函数通过内部调用`os.Exit(1)`跳过延迟调用。目前main函数中还没有延迟调用，但我们将在下一章添加。

现在我们已经完成了游戏循环的修改，可以使用`go run`运行程序或用`go build`编译后作为独立程序运行。

```sh
go run main.go
```

你应该能看到迷宫打印到终端。

[前往第二步！](../step02/README.md)