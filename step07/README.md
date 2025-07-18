# 步骤07：终于加入emoji！

在本课中您将学习如何：

- 加载json文件
- 打印emoji！！！

## 概述

我们已经成功在终端中创建了一个像样的游戏。但我承诺的emoji在哪里呢？终于，是时候加入它们了！

在这一步中，我们将创建一个名为`config.json`的文件。这个文件将存储游戏中使用的每个符号的映射关系。在2D游戏中，我们通常将移动的物体称为"精灵"。

由于现今大多数终端都支持Unicode，我们可以直接使用emoji作为精灵，而无需借助任何图形库。

提供的`config.json`文件内容如下：

```json
{
    "player": "😋",
    "ghost": "👻",
    "wall": "🌵",
    "dot": "🧀",
    "pill": "🍹",
    "death": "💀",
    "space": "  ",
    "use_emoji": true
}
```

这是默认的映射关系，但您可以自由尝试各种emoji组合。我们有无限的可能性！

配置文件中的一个重要方面是`use_emoji`配置项。我们使用这个标志来告知游戏是否使用emoji。这是必要的，因为emoji通常会在屏幕上占用多个字符（大多数占用2个字符）。

使用这个标志，我们可以有替代的代码路径来进行调整以补偿这种差异。否则迷宫看起来会变形。

## 任务01：加载json

Go标准库支持加载json。

首先我们需要定义一个结构体来保存json数据。反引号(\`)之间的文本称为`结构体标签`。它被json解码器用来确定结构体的哪个字段对应json文件中的哪个字段。

```go
// Config保存emoji配置
type Config struct {
    Player   string `json:"player"`
    Ghost    string `json:"ghost"`
    Wall     string `json:"wall"`
    Dot      string `json:"dot"`
    Pill     string `json:"pill"`
    Death    string `json:"death"`
    Space    string `json:"space"`
    UseEmoji bool   `json:"use_emoji"`
}

var cfg Config
```

注意我们为`Config`结构体使用了公开成员。这是json解码器正常工作所必需的。

下面的代码解析json并将其存储在全局变量`cfg`中：

```go
func loadConfig(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    decoder := json.NewDecoder(f)
    err = decoder.Decode(&cfg)
    if err != nil {
        return err
    }

    return nil
}
```

现在在main函数的初始化部分，在`loadMaze`之后添加`loadConfig`调用：

```go
err = loadConfig("config.json")
if err != nil {
    log.Println("加载配置失败:", err)
    return
}
```

## 任务02：调整水平位移

我们需要创建一个自定义的`moveCursor`函数，在启用emoji标志时校正水平位移：

```go
func moveCursor(row, col int) {
    if cfg.UseEmoji {
        simpleansi.MoveCursor(row, col*2)
    } else {
        simpleansi.MoveCursor(row, col)
    }
}
```

确保将所有`simpleansi.MoveCursor`调用替换为新的`moveCursor`函数（新函数内部的调用除外）。

将`col`值乘以2可以确保我们将每个字符定位在正确的位置。这也会产生使迷宫看起来更大的副作用。

## 任务03：用配置替换硬编码字符

最后一步是在`printScreen`函数中用配置项替换硬编码的字符。我们还将使用`simpleansi.WithBlueBackground`函数改变墙壁的颜色，使其更接近原版游戏。

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Print(simpleansi.WithBlueBackground(cfg.Wall))
            case '.':
                fmt.Print(cfg.Dot)
            default:
                fmt.Print(cfg.Space)
            }
        }
        fmt.Println()
    }

    moveCursor(player.row, player.col)
    fmt.Print(cfg.Player)

    for _, g := range ghosts {
        moveCursor(g.row, g.col)
        fmt.Print(cfg.Ghost)
    }

    moveCursor(len(maze)+1, 0)
    fmt.Println("分数:", score, "\t生命:", lives)
}
```

## 任务04：游戏结束和能量豆

作为额外奖励，让我们在游戏结束条件中添加游戏结束精灵。

```go
// 检查游戏结束
if numDots == 0 || lives == 0 {
    if lives == 0 {
        moveCursor(player.row, player.col)
        fmt.Print(cfg.Death)
        moveCursor(len(maze)+2, 0)
    }
    break
}
```

同时，让我们添加代码将能量豆视为得分更高的豆子，作为实际能量机制的占位符。我们现在这样做是为了获得完整游戏的感觉，但稍后我们会实现真正的能量机制。

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)

    removeDot := func(row, col int) {
        maze[row] = maze[row][0:col] + " " + maze[row][col+1:]
    }

    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        removeDot(player.row, player.col)
    case 'X':
        score += 10
        removeDot(player.row, player.col)
    }
}
```

上面代码中一个有趣的地方是，我们定义了一个内联函数来处理豆子和X的移除。我们也可以重复代码，但这样使代码更可读和可维护。

我们有了emoji！是不是很棒？ :)

[带我去步骤08！](../step08/README.md)