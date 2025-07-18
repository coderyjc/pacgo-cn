# 步骤04：添加幽灵敌人

在本课中，您将学习如何：

- 创建映射表（字典）
- 生成随机数
- 使用指针

## 概述

既然我们已经可以让玩家移动，现在是时候处理敌人（幽灵）了。

我们将使用与玩家相同的移动机制`makeMove`函数，但不是从键盘读取输入，而是使用一个简单算法：生成0到3之间的随机数，并为每个值分配一个方向。

如果幽灵碰到墙也没关系，它会在下一次迭代中再次尝试。

## 任务01：创建幽灵

就像我们为玩家数据创建结构体一样，我们将为幽灵创建类似的结构。唯一的区别是我们不会在内存中保存玩家全局变量，而是保存一个指向精灵的指针切片。这样可以非常高效地更新每个幽灵的位置。

首先声明：

```go
var ghosts []*sprite
```

注意`*`符号表示`[]*sprite`是一个指向sprite对象的指针切片。

接下来是加载。在`loadMaze`函数中，为处理地图上的`G`符号添加一个新的case：

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})
        }
    }
}
```

请注意与符号(`&`)操作符。这意味着我们不是向切片添加sprite对象，而是添加指向它的指针。

Go是一种垃圾回收语言，这意味着它可以自动释放不再使用的内存。因此，我们可以比在C++等语言中更安全地使用指针。我们也不能对指针进行数学运算。实际上，Go中的指针几乎就像引用一样工作。

现在，既然我们在`loadMaze`函数中处理`G`，我们也需要在`printScreen`中打印它们。在打印玩家之后添加以下代码块：

```go
for _, g := range ghosts {
    simpleansi.MoveCursor(g.row, g.col)
    fmt.Print("G")
}
```

## 任务02：非常"智能"的AI

我们之前提到过将使用随机数生成器来控制幽灵。这听起来比实际复杂得多。请看代码：

```go
func drawDirection() string {
    dir := rand.Intn(4)
    move := map[int]string{
        0: "UP",
        1: "DOWN",
        2: "RIGHT",
        3: "LEFT",
    }
    return move[dir]
}
```

`math/rand`包中的`rand.Intn`函数生成一个区间`[0, n)`内的随机数，其中`n`是传递给函数的参数。（注意区间是左闭右开的，所以不包括`n`）。

我们使用了一个技巧，通过`map`将整数值映射到实际移动方向。map是一种将一个值映射到另一个值的数据结构。在上面的例子中，map`move`将整数映射到字符串。

## 任务03：添加移动功能！

最后，我们需要一个函数来处理幽灵移动。`moveGhosts`函数如下：

```go
func moveGhosts() {
    for _, g := range ghosts {
        dir := drawDirection()
        g.row, g.col = makeMove(g.row, g.col, dir)
    }
}
```

现在更新游戏循环以调用`moveGhosts`：

```go
// 游戏循环
for {
    // 更新屏幕
    printScreen()

    // 处理输入
    input, err := readInput()
    if err != nil {
        log.Println("error reading input:", err)
        break
    }

    // 处理移动
    movePlayer(input)
    moveGhosts()

    // 处理碰撞

    // 检查游戏结束
    if input == "ESC" {
        break
    }

    // 重复
}
```

完成了！现在我们有了会移动的幽灵！多么可怕 -_-'''

[带我去步骤05！](../step05/README.md)