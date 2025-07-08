<div style="background-color: #efefef; padding: 10px; margin: 5px 0;border-radius: 5px;">
说明：本仓库fork自<a href="https://github.com/danicat/pacgo" target="_blank">pacgo</a>，目的是在学习Go语言的同时制作一个中文版，在尽量不改变原仓库内容的情况下，对原版进行了本土化翻译，以方便后来的学习者。
</div>

# Pac Go

使用Go语言开发吃豆人游戏

<img src="./screenshot.jpg" height="400px" width="400px" />

## 介绍

欢迎来到Pac Go！本项目是一个通过编写游戏来进行[Go语言](https://golang.org)学习的教程。

### 为什么要通过编写游戏入门Go语言？

虽然市面上已有大量优质教程，但本项目的核心理念是通过趣味游戏的方式学习Go语言。当然，日常工作中我们需要处理API和CRUD操作，但在学习新语言时，为何不尝试开发游戏呢？

Go语言以重拾编程乐趣著称，而开发游戏正是最富趣味的学习方式之一。感兴趣吗？太好了，让我们继续吧！

我们将开发一个终端版的吃豆人(Pac-Man)克隆游戏。在游戏开发过程中，您将接触到各种有趣的编程需求，这些需求完美涵盖了语言的多个特性，包括输入输出、函数、指针、并发以及基础数学运算。

同时您还将深入了解终端及其神奇的转义序列。

### 相关的技术演讲

如果您想在动手前观看教程演讲（约25分钟），可参考以下链接：

> 译注：这些演讲都是英文的

1. [Google Cloud Next UK '19](https://cloud.withgoogle.com/next/uk/sessions?session=DZ224), London, UK (November, 21st 2019)
2. [London Gophers](https://youtu.be/SM8LTMnB4x0), London, UK
3. [GoWayFest 3.0](https://youtu.be/0qvW4kIlS8I), Minsk, Belarus
4. [GothamGo 2019](https://youtu.be/GH0DlCKTppE), New York City, NY, USA

### 贡献

本项目采用MIT开源协议，这意味着您可以自由使用它，只需保留适当的署名即可 :)

如需贡献，请提交issue或pull request。

如需灵感，您可以浏览开放issue或查看TODO清单。

除非特别说明，TODO清单上的所有项目都应作为教程的新步骤进行规划。

> 译注：原仓库并没有提供todo清单上的教程，因此在本仓库中把`todo.md`删去了，读者可自行到原仓库查看todo清单。
> 另外，原仓库中的`stepxx`也没有教程，因此本仓库中也删去了。

### License

查看 [LICENSE](LICENSE).

### 联系作者

原作者邮箱：[daniela.petruzalek@gmail.com](mailto:daniela.petruzalek@gmail.com). Twitter [@danicat83](https://twitter.com/danicat83).

中文翻译仓库作者邮箱：[coderyjc@163.com](mailto:coderyjc@163.com)

## 相较于原仓库的改动

- 原仓库使用的是1.17，我使用的是1.23.3，经验证所有代码都能正常运行。
- 更改模块名为`module github.com/coderyan/pacgo-cn`
- 在不影响学习的情况下，删除以下文件/目录：`TODO.md`,`pacgo.slide`等，上述文件都可以在[原仓库](https://github.com/danicat/pacgo)找到
- 将`readme`文档的Getting Started部分独立出来，移动到了`./step00`中
