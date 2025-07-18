# 步骤10：能量豆与幽灵变身！

在本课中您将学习如何：

- 使用定时器
- 何时及如何使用互斥锁

## 概述

本节课我们将为游戏添加能量豆功能。首先更新配置支持新特性，添加能量豆绘制逻辑。然后处理吃豆人吞食能量豆后与幽灵的碰撞机制。最后解决连续吞食能量豆时的状态管理问题。

## 任务01：绘制能量豆

首先更新配置文件支持能量豆特性。在`config_noemoji.json`和`config.json`中添加`ghost_blue`(字符串)和`pill_duration_secs`(整数)配置项：

```go
type config struct {
    ...
    GhostBlue        string        `json:"ghost_blue"`
    PillDurationSecs time.Duration `json:"pill_duration_secs"`
}
```

## 任务02：实现吞食能量豆

在`movePlayer`函数中添加处理能量豆('X')的逻辑：

```go
case 'X':
    score += 10
    removeDot(player.row, player.col)
    go processPill()
```

定义幽灵状态类型和两种状态常量：

```go
type GhostStatus string

const (
    GhostStatusNormal GhostStatus = "Normal"
    GhostStatusBlue   GhostStatus = "Blue"
)
```

更新幽灵结构体，包含位置信息和状态：

```go
type ghost struct {
    position sprite
    status   GhostStatus
}
```

在`loadMaze`中初始化幽灵状态为普通状态：

```go
ghosts = append(ghosts, &ghost{sprite{row, col, row, col}, GhostStatusNormal})
```

更新`printScreen`支持显示两种幽灵状态：

```go
for _, g := range ghosts {
    moveCursor(g.position.row, g.position.col)
    if g.status == GhostStatusNormal {
        fmt.Printf(cfg.Ghost)
    } else if g.status == GhostStatusBlue {
        fmt.Printf(cfg.GhostBlue)
    }
}
```

实现能量豆处理函数，使用定时器控制状态持续时间：

```go
var pillTimer *time.Timer

func processPill() {
    for _, g := range ghosts {
        g.status = GhostStatusBlue
    }
    pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
    <-pillTimer.C
    for _, g := range ghosts {
        g.status = GhostStatusNormal
    }
}
```

## 任务03：处理连续吞食能量豆

改进`processPill`函数，处理能量豆效果叠加情况：

```go
var pillTimer *time.Timer

func processPill() {
    updateGhosts(ghosts, GhostStatusBlue)
    if pillTimer != nil {
        pillTimer.Stop()
    }
    pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
    <-pillTimer.C
    pillTimer.Stop()
    updateGhosts(ghosts, GhostStatusNormal)
}
```

## 任务04：避免竞态条件

引入互斥锁解决并发问题：

```go
var pillTimer *time.Timer
var pillMx sync.Mutex

func processPill() {
    pillMx.Lock()
    updateGhosts(ghosts, GhostStatusBlue)
    if pillTimer != nil {
        pillTimer.Stop()
    }
    pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
    pillMx.Unlock()
    <-pillTimer.C
    pillMx.Lock()
    pillTimer.Stop()
    updateGhosts(ghosts, GhostStatusNormal)
    pillMx.Unlock()
}
```

使用读写锁保护幽灵状态：

```go
var ghostsStatusMx sync.RWMutex

func updateGhosts(ghosts []*ghost, ghostStatus GhostStatus) {
    ghostsStatusMx.Lock()
    defer ghostsStatusMx.Unlock()
    for _, g := range ghosts {
        g.status = ghostStatus
    }
}
```

读取幽灵状态时使用读锁：

```go
ghostsStatusMx.RLock()
// 读取状态操作
ghostsStatusMx.RUnlock()
```

现在您拥有了一个更具挑战性的吃豆人游戏！祝您游戏/编码愉快！

## 教程完结

[返回项目首页](../README.md)