# 介绍

Go是一门由对编程语言的不满而诞生的编程语言。在谷歌，开发人员厌倦了在选择新项目的语言时不得不做出权衡。一些语言执行效率高，但编译时间长，而另一些易于编写，但在生产环境中运行效率低下。因此，谷歌发明了Go，并设计了这门语言，使其具备一切：快速编译、快速执行、易于编写和易于部署。

虽然Go是一种多才多艺的语言，可用于许多项目，从Web应用程序到命令行工具，但它特别适用于分布式系统和微服务架构，因此被誉为云计算的语言。它通过强大的工具集，帮助现代程序员更有效地工作，通过将格式作为语言规范的一部分，消除了关于格式的争论，以及通过将每个程序及其所有依赖项编译成单个二进制文件，使部署变得轻松。Go易于学习，具有非常小的关键字集，这使它成为初学者和资深开发人员的理想选择。

在这个入门教程中，您将在本地macOS机器上安装Go，并运行您的第一个程序以证明安装成功。

## 先决条件

您需要一台连接到互联网的具有管理员访问权限的macOS计算机。

## 步骤1 —— 打开终端

macOS终端是一个您可以使用的应用程序，用于访问命令行界面。您可以通过进入Finder，导航到应用程序文件夹，然后进入实用程序文件夹找到它。从这里，双击打开终端。

现在您已经打开了终端，您可以下载并安装Xcode，这是一个包含您安装Go所需的开发工具的软件包。

## 步骤2 —— 安装Xcode

Xcode是一个集成开发环境（IDE），包含macOS的软件开发工具。您可以通过在终端中键入以下内容检查是否已安装Xcode：

```bash hl_lines="1"
xcode-select -p
```

如果输出如下，表示Xcode已安装：

```bash
/Library/Developer/CommandLineTools
```

如果收到错误，请从App Store安装Xcode并接受默认选项。

安装Xcode后，请返回到终端窗口。接下来，您需要安装Xcode的单独的命令行工具应用程序，您可以通过键入以下内容来完成：

```bash
xcode-select --install
```

此时，Xcode及其命令行工具应用程序已完全安装，您准备好安装软件包管理器Homebrew。

## 步骤3 —— 安装和设置Homebrew

虽然macOS终端与Linux终端和其他Unix系统的终端非常相似，但它没有像Linux发行版那样附带官方的命令行软件包管理器。软件包管理器可帮助您安装、升级和配置软件，并在终端或脚本内进行交互式卸载。有一些开源（非官方）的macOS软件包管理器，Homebrew是其中最受欢迎的之一。它提供了一种快速灵活的方式在macOS上安装和更新Go。

要安装Homebrew，请在终端中运行：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

这个命令从GitHub下载一个脚本并安装Homebrew。如果需要输入密码，请注意您的按键不会显示在终端窗口中，但它们将被记录。输入密码后，只需按一下回车键。否则，在确认安装时每次提示时按y键。

安装完成后，您将Homebrew的目录放在PATH环境变量的顶部，以便通过Homebrew安装的任何东西将优先于macOS上默认安装的同名程序（如果有的话）。由于macOS不附带Go，将Homebrew添加到PATH的顶部在这种情况下并不是绝对必要的，但为了适应其他情况，许多开发人员更喜欢将Homebrew添加到他们的PATH的顶部。

要执行此操作，请使用命令行文本编辑器nano创建或打开文件~/.zprofile：

```bash
nano ~/.zprofile
```

注意：如果您运行的macOS版本早于10.15 Catalina，您的终端可能使用Bash shell（/bin/bash）而不是Z shell（/bin/zsh）。在这种情况下，您需要创建或打开文件~/.bash_profile而不是~/.zprofile。要检查您使用的shell，请运行echo $SHELL。

在文件中添加以下行：

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

通过输入CTRL+x退出nano，然后在提示保存文件时按y，然后按ENTER。

现在激活这些更改：

```bash
source ~/.zprofile
```

通过键入以下内容，您可以确保Homebrew已成功安装：

```bash
brew doctor
```

如果此时不需要更新，则输出将为：

```bash title="install brew"
Your system is ready to brew.
```

否则，您可能会收到警告，要求运行其他命令，例如brew update，以确保您的Homebrew安装是最新的。

Homebrew准备就绪后，您可以安装Go。:man_raising_hand:

## 步骤4 —— 安装Go

您可以使用brew search命令搜索所有可用的Homebrew软件包。出于本教程的目的。
