# 介绍

本文将引导您深入了解GOPATH是什么，它的工作原理以及如何设置。这是设置Go开发环境的关键步骤，同时也有助于理解Go如何查找、安装和构建源文件。在本文中，我们在提到文件夹结构的概念时将使用GOPATH，并使用`$GOPATH`来指代Go用于查找该文件夹结构的环境变量。

Go Workspace是Go管理我们的源文件、编译后的二进制文件以及用于加快后续编译的缓存对象的方式。通常建议只使用一个Go Workspace，尽管也可以有多个。GOPATH充当工作区的根文件夹。

## 设置`$GOPATH`环境变量

`$GOPATH`环境变量列出了Go查找Go Workspaces的位置。

默认情况下，Go假定我们的GOPATH位置在`$HOME/go`，其中`$HOME`是计算机上我们用户帐户的根目录。我们可以通过设置`$GOPATH`环境变量来更改这个位置。有关进一步学习，请参阅[Linux环境变量的读取和设置教程](#)。

有关设置`$GOPATH`变量的更多信息，请参阅[Go文档](#)。

此外，本系列教程将介绍如何[安装Go和设置Go开发环境](#)。

### `$GOPATH`不是`$GOROOT`

`$GOROOT`是Go代码、编译器和工具的存放位置 —— 这不是我们的源代码。`$GOROOT`通常类似于`/usr/local/go`。我们的`$GOPATH`通常类似于`$HOME/go`。

虽然我们不再需要专门设置`$GOROOT`变量，但在较旧的资料中仍然会提及。

现在，让我们讨论Go Workspace的结构。

## Go Workspace的解剖

在Go Workspace或GOPATH中，有三个目录：`bin`、`pkg`和`src`。每个目录对于Go工具链都有特殊的含义。

```
.
├── bin
├── pkg
└── src
  └── github.com/foo/bar
    └── bar.go
```

让我们看看这些目录。

- `$GOPATH/bin`目录是Go放置`go install`编译的二进制文件的地方。我们的操作系统使用`$PATH`环境变量查找可以在没有完整路径的情况下执行的二进制应用程序。建议将此目录添加到全局`$PATH`变量中。

  例如，如果我们不将`$GOPATH/bin`添加到`$PATH`以从该目录执行程序，我们需要运行：

  ```bash
  $GOPATH/bin/myapp
  ```

  当将`$GOPATH/bin`添加到`$PATH`时，我们可以使用相同的方式进行调用：

  ```bash
  myapp
  ```

- `$GOPATH/pkg`目录是Go存储预编译对象文件以加速后续程序编译的地方。通常，大多数开发人员不需要访问此目录。如果遇到编译问题，可以安全地删除此目录，Go将重新构建它。

- `src`目录是我们所有`.go`文件或源代码必须位于的位置。这不应与Go工具使用的源代码混淆，后者位于`$GOROOT`。随着我们编写Go应用程序、包和库，我们将这些文件放在`$GOPATH/src/path/to/code`下。

## 什么是包？

Go代码组织成包。一个包代表磁盘上单个目录中的所有文件。一个目录只能包含来自同一个包的特定文件。包与所有用户编写的Go源文件一起存储在`$GOPATH/src`目录下。我们可以通过导入不同的包来理解包的解析。

如果我们的代码位于`$GOPATH/src/blue/red`，那么它的包名应该是`red`。

red包的导入语句将是：

```go
import "blue/red"
```

存储在源代码存储库中的包，如GitHub和BitBucket，其导入路径包含存储库的完整位置。

例如，我们将使用以下导入路径导入[https://github.com/gobuffalo/buffalo](https://github.com/gobuffalo/buffalo)的源代码：

``` golang
import "github.com/gobuffalo/buffalo"
```

因此，此源代码将位于以下磁盘位置：

```bash
$GOPATH/src/github.com/gobuffalo/buffalo
```

## 结论

在本文中，我们讨论了GOPATH作为Go期望我们的源代码所在的一组文件夹，以及这些文件夹是什么以及它们包含了什么。我们讨论了如何通过设置`$GOPATH`环境变量将该位置从默认值`$HOME/go`更改为用户选择的位置。最后，我们讨论了Go如何在该文件夹结构中搜索包。

在Go 1.11中引入的Go模块旨在取代Go Workspaces和GOPATH。虽然建议开始使用模块，但一些环境（如企业环境）可能尚未准备好使用模块。

GOPATH是设置Go的更棘手的方面之一，但一旦设置好，我们通常就可以忘记它。
