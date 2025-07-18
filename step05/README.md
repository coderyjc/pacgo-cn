# 步骤05：游戏结束！

在本课中，您将学习如何：

- 在switch块中使用fallthrough语句
- 使用切片操作

## 概述

我们即将完成。现在已经有玩家移动和幽灵移动功能，但幽灵还不会造成威胁。

是时候为游戏增加一些危险元素了。同时，高风险也应带来高回报，我们还将实现游戏胜利条件——清除场上所有豆子。

## 任务01：准备工作

为了实现游戏胜利条件，我们需要跟踪场上剩余的豆子数量，当数量为零时宣布胜利。

我们需要一个机制在玩家站在豆子上时将其移除，并记录分数显示给玩家。

对于游戏结束场景，我们将给玩家一条生命，当被幽灵碰到时生命值归零。然后在游戏循环中检测生命值是否为零来终止游戏。（后续可以很容易地扩展为多条生命，我们将在后续步骤中实现）

添加以下全局变量来跟踪上述内容：

```go
var score int
var numDots int
var lives = 1
```

接下来，我们需要在`loadMaze`中初始化`numDots`变量。只需在switch中添加处理`.`字符的新case：

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})
        case '.':
            numDots++
        }
    }
}
```

现在我们需要更新`printScreen`函数重新显示豆子。这是使用`fallthrough`语句的一个有趣案例：

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fallthrough
            case '.':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }
    // 其余代码省略...
}
```

最后，在`printScreen`函数末尾添加分数和生命值面板：

```go
func printScreen() {
    // 代码省略...

    // 打印分数
    simpleansi.MoveCursor(len(maze)+1, 0)
    fmt.Println("分数:", score, "\t生命:", lives)
}
```

## 任务02：游戏结束

处理游戏结束很简单。任何时候，如果玩家和幽灵处于同一位置，我们就判定玩家死亡。我们将检测这个情况的代码添加到游戏循环中。同时修改游戏退出条件，增加`lives <= 0`和`numDots == 0`：

```go
// 游戏循环
for {
    // 代码省略...

    // 处理碰撞
    for _, g := range ghosts {
        if player == *g {
            lives = 0
        }
    }

    // 检查游戏结束
    if input == "ESC" || numDots == 0 || lives <= 0 {
        break
    }

    // 重复
}
```

请注意，更详细的检查玩家位置的方法是`player.row == g.row && player.col == g.col`，但由于玩家和幽灵都是sprite类型，可以直接使用`player == *g`进行比较。我们仍然需要解引用`g`，因为不能直接比较指针和非指针类型。

## 任务03：游戏胜利

现在只缺少从游戏中移除豆子和增加分数的代码了。

我们将这段代码添加到`movePlayer`函数中：

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        // 从迷宫中移除豆子
        maze[player.row] = maze[player.row][0:player.col] + " " + maze[player.row][player.col+1:]
    }
}
```

上述代码工作原理：首先执行移动。然后检查玩家所在位置的字符。如果是豆子，就减少豆子总数(`numDots`)，增加分数并从板上移除豆子。

值得一提的是Go中的字符串是不可变的。我们不能简单地给字符串的某个位置赋值空格，这不会生效。

因此，我们在这里使用了一个技巧：通过创建由原字符串的两个切片组成的新字符串。这两个切片组合起来与原字符串完全相同，只是我们用一个空格替换了其中一个位置。

关于切片的更多信息，请参见[这里](https://blog.golang.org/go-slices-usage-and-internals)。

现在我们同时具备了游戏胜利和游戏结束条件。尝试构建一个只有几个豆子的地图来测试游戏胜利。碰到幽灵来测试游戏结束。我们正在取得进展！

（提示：step05文件夹中的maze01.txt只有3个豆子。）

[带我去步骤06！](../step06/README.md)