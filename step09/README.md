# 步骤09：使用缓冲区优化字符串拼接

在本课中您将学习如何：

- 使用缓冲区进行字符串拼接

## 概述

本节课我们将为游戏添加多生命值支持。我们将更新碰撞检测代码，使碰到幽灵时减少生命值而非直接归零。同时记录玩家初始位置以便死亡后重生。最后，我们将在计分板用玩家表情符号显示剩余生命值，而非简单的数字。

## 任务01：创建Point类型并更新Player结构体

我们需要追踪玩家的初始位置，以便在碰撞后重置位置。我们将更新sprite结构体添加起始行列属性：

```go
type sprite struct {
    row      int
    col      int
    startRow int
    startCol int
}
```

然后在`loadMaze`函数中为玩家和幽灵填充这些属性：

```go
func loadMaze() error {
    //...省略部分代码

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col, row, col}
            case 'G':
                ghosts = append(ghosts, &sprite{row, col, row, col})
            case '.':
                numDots++
            }
        }
    }

    return nil
}
```

注意由于新增了起始位置属性，简单的坐标比较已不能用于碰撞检测，因为玩家和幽灵初始位置不会重合。我们将在下一节体现这一变化。

## 任务02：设置初始生命值并在碰撞时减少

首先设置初始生命值为3：

```go
var lives = 3
```

然后更新碰撞处理代码，每次碰撞减少1点生命值。最后检查生命值是否耗尽，未耗尽时将玩家重置到初始位置：

```go
    // 处理碰撞
    for _, g := range ghosts {
        if player.row == g.row && player.col == g.col {
            lives = lives - 1
            if lives != 0 {
                moveCursor(player.row, player.col)
                fmt.Print(cfg.Death)
                moveCursor(len(maze)+2, 0)
                time.Sleep(1000 * time.Millisecond) //重置前的戏剧性暂停
                player.row, player.col = player.startRow, player.startCol
            }
        }
    }
```

## 任务03：用表情符号显示剩余生命值

之前生命值在计分板显示为数字。现在我们将更新为用玩家表情符号显示剩余生命数。添加`getLivesAsEmoji`函数，使用缓冲区拼接正确数量的玩家表情符号：

```go
func printScreen() {
    //...省略部分代码

    moveCursor(len(maze)+1, 0)

    livesRemaining := strconv.Itoa(lives) //将生命值转为字符串
    if cfg.UseEmoji {
        livesRemaining = getLivesAsEmoji()
    }

    fmt.Println("分数:", score, "\t生命:", livesRemaining)
}

// 根据生命值拼接对应数量的玩家表情
func getLivesAsEmoji() string{
    buf := bytes.Buffer{}
    for i := lives; i > 0; i-- {
        buf.WriteString(cfg.Player)
    }
    return buf.String()
}
```

为什么使用缓冲区？Go中还有其他字符串拼接方式，最简单的就是使用`+`运算符：

```go
string1 := "pac"
string2 := "go"
pacgo := string1 + string2 //"pacgo"
```

如果用`+`运算符实现`getLivesAsEmoji`函数：

```go
func getLivesAsEmoji() string {
    emojiString := ""
    for i := lives; i > 0; i-- {
        emojiString = emojiString + cfg.Player
    }
    return emojiString
}
```

这个版本效率低于缓冲区版本，因为每次循环迭代都需要内存分配操作（Go中字符串不可变）。而缓冲区版本仅在初始化时进行一次内存分配。[这里](https://billglover.me/2019/03/13/learn-go-by-concatenating-strings/)有更详细的性能对比分析。

[带我去步骤10！](../step10/README.md)