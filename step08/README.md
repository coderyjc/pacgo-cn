# 步骤08：命令行参数配置

在本课中您将学习如何：

- 为命令行应用添加参数标志

## 概述

在上节课中，我们添加了`config.json`配置文件来处理表情符号转换。同时我们还有一个`config_noemoji.json`文件用于游戏原始字符表示。

我们还使用`maze01.txt`文件作为迷宫地图。所有这些文件名都是直接硬编码在源代码中的，但这种处理方式并不理想，我们需要改进这一点。

## 任务01：为每个文件创建参数标志

标准库中的`flag`包负责处理命令行标志。我们将使用它创建两个标志：`--config-file`和`--maze-file`。

在文件开头导入语句之后，添加以下全局变量：

```go
var (
    configFile = flag.String("config-file", "config.json", "自定义配置文件路径")
    mazeFile   = flag.String("maze-file", "maze01.txt", "自定义迷宫文件路径")
)
```

`flag`包的`String`函数接受三个参数：标志名称、默认值和描述（在使用`--help`时显示）。它返回一个字符串指针，将保存标志的值。

请注意，这个值只有在调用`flag.Parse`后才会被填充，应该在`main`函数中调用：

```go
func main() {
    flag.Parse()

    // 初始化游戏
    initialise()
    defer cleanup()

    // 函数其余部分省略...
}
```

注意我们将`flag.Parse()`作为程序的第一件事调用。我们希望在所有操作之前先解析标志，然后再将控制台切换到`cbreak`模式。

当解析标志出错时，它会调用`os.Exit`，这意味着我们的`cleanup`函数不会被调用，终端将保持无回显和cbreak模式，这会很不方便。

通过控制调用顺序，我们确保只有在标志成功解析后才初始化cbreak模式。

## 任务02：用参数标志替换硬编码文件

我们已经处理了解析部分，现在需要用参数标志替换硬编码值。

通过将硬编码值替换为标志值来实现（注意解引用操作符，因为标志是指针）。

在`main`函数中：

```go
    // 加载资源
    err := loadMaze(*mazeFile)
    if err != nil {
        log.Println("加载迷宫失败:", err)
        return
    }

    err = loadConfig(*configFile)
    if err != nil {
        log.Println("加载配置失败:", err)
        return
    }
```

现在尝试在命令行中运行：

```sh
go build
./step08 --help
```

您应该会看到类似输出：

```sh
$ ./step08 --help
Usage of ./step08:
  -config-file string
        自定义配置文件路径 (默认 "config.json")
  -maze-file string
        自定义迷宫文件路径 (默认 "maze01.txt")
```

现在先尝试使用`--config-file config_noemoji.json`运行`step08`，然后再用`--config-file config.json`运行看看区别。使用表情符号更好对吧？

您也可以尝试复制`maze01.txt`到新文件并编辑它进行实验。

也许您现在可以创建自己的主题了...试试访问[完整表情符号列表](https://unicode.org/emoji/charts/full-emoji-list.html)寻找灵感。 :)

[带我去步骤09！](../step09/README.md)