# 步骤03：添加移动功能

在本课中，您将学习如何：

- 创建结构体
- 使用switch语句
- 处理方向键输入
- 使用命名返回值

## 概述

我们已经有了迷宫，可以优雅地退出游戏...但还没有什么令人兴奋的事情发生，对吧？让我们为游戏增添一些趣味，添加移动功能！

在这一步中，我们将添加玩家角色并启用方向键控制移动。

## 任务01：追踪玩家位置

第一步是创建一个变量来保存玩家数据。由于我们需要追踪二维坐标（行和列），我们将定义一个结构体来保存这些信息：

```go
type sprite struct {
    row int
    col int
}

var player sprite
```

为了简单起见，我们将player定义为全局变量。

接下来，我们需要在加载迷宫时捕获玩家位置，在`loadMaze`函数中：

```go
// 遍历迷宫的每个字符，当找到'P'时创建新玩家
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        }
    }
}
```

注意这次我们使用了完整的`range`操作符形式，因为我们需要知道找到玩家的具体行和列。

以下是完整的`loadMaze`函数供参考：

```go
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

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col}
            }
        }
    }

    return nil
}
```

---

### 可选：关于可见性的说明

为了教程的简洁性，我们在这里保持简单。由于所有内容都在单个文件中，我们没有考虑变量的可见性（即它们是公共的还是私有的）。

不过，Go在定义可见性方面有一个有趣的机制。它没有public关键字，而是将名称以大写字母开头的每个符号视为公共的。另一方面，如果名称以小写字符开头，则它是私有符号。

这就是为什么到目前为止我们使用的所有库函数名称都以大写字母开头。这也是为什么如果您定义任何变量、函数或类型时使用首字母大写，您的IDE可能会抱怨缺少注释。在Go的惯例中，公共符号应该始终有注释，因为这些注释稍后会被提取成为包文档。

在这个特定情况下，我们对所有变量、类型和函数都使用小写符号，因为从`main`包导出符号没有意义。

---

## 任务02：处理方向键输入

接下来，我们需要修改`readInput`来处理方向键：

```go
if cnt == 1 && buffer[0] == 0x1b {
    return "ESC", nil
} else if cnt >= 3 {
    if buffer[0] == 0x1b && buffer[1] == '[' {
        switch buffer[2] {
        case 'A':
            return "UP", nil
        case 'B':
            return "DOWN", nil
        case 'C':
            return "RIGHT", nil
        case 'D':
            return "LEFT", nil
        }
    }
}
```

方向键的转义序列是3字节长，以`ESC+[`开头，然后是A到D之间的字母。

现在我们需要一个函数来处理移动：

```go
func makeMove(oldRow, oldCol int, dir string) (newRow, newCol int) {
    newRow, newCol = oldRow, oldCol

    switch dir {
    case "UP":
        newRow = newRow - 1
        if newRow < 0 {
            newRow = len(maze) - 1
        }
    case "DOWN":
        newRow = newRow + 1
        if newRow == len(maze) {
            newRow = 0
        }
    case "RIGHT":
        newCol = newCol + 1
        if newCol == len(maze[0]) {
            newCol = 0
        }
    case "LEFT":
        newCol = newCol - 1
        if newCol < 0 {
            newCol = len(maze[0]) - 1
        }
    }

    if maze[newRow][newCol] == '#' {
        newRow = oldRow
        newCol = oldCol
    }

    return
}
```

注意：如果您熟悉其他语言中的switch语句，请注意在Go中每个`case`条件后都有一个隐式的`break`。所以我们不需要在每个块后显式地break。如果我们想要继续执行下一个`case`块，可以使用`fallthrough`关键字。

上面的函数利用了`命名返回值`来返回移动后的新位置（`newRow`和`newCol`）。基本上，函数先"尝试"移动，如果新位置碰到墙(`#`)，则取消移动。

它还处理了当角色移动到迷宫范围外时出现在对面的特性。

移动功能的最后一部分是定义一个移动玩家的函数：

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
}
```

## 任务03：更新迷宫显示

我们已经有了所有的移动逻辑，但需要让屏幕反映这些变化。我们将重构`printScreen`函数，只打印我们想要打印的内容，而不是整个地图。

这将给我们更多的控制权，使我们能够使用`moveCursor`函数在任意位置打印玩家。请看下面的代码：

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }

    simpleansi.MoveCursor(player.row, player.col)
    fmt.Print("P")

    // 将光标移动到迷宫绘制区域外
    simpleansi.MoveCursor(len(maze)+1, 0)
}
```

目前，我们忽略了除墙和玩家之外的所有内容。

## 任务04：动画效果！

最后，我们需要在游戏循环中调用`movePlayer`：

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

    // 处理碰撞

    // 检查游戏结束
    if input == "ESC" {
        break
    }

    // 重复
}
```

我们准备好出发了！

[带我去步骤04！](../step04/README.md)