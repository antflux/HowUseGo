# 使用 ldflags 为 Go 应用程序设置版本信息

### 介绍

在将应用程序部署到生产环境中时，使用版本信息和其他元数据构建二进制文件将通过添加标识信息来帮助跟踪随时间推移的构建，从而改进监视、日志记录和调试过程。此版本信息通常可以包括高度动态的数据，例如构建时间、构建二进制文件的计算机或用户、构建二进制文件时所依据的[版本控制系统（Version Control System，简称VCS）](https://www.atlassian.com/git/tutorials/what-is-version-control)提交ID等等。由于这些值不断变化，因此将这些数据直接编码到源代码中并在每次新构建之前修改它是繁琐的，并且容易出错：源文件可以四处移动，[变量/常量]可能会在整个开发过程中切换文件，从而中断构建过程。

在Go中解决这个问题的一种方法是使用`-ldflags`和`go build`命令在构建时将动态信息插入到二进制文件中，而不需要修改源代码。在这个标志中，`ld`代表[链接器](https://en.wikipedia.org/wiki/Linker_(computing))，这个程序将编译后的源代码的不同部分链接到最终的二进制文件中。`ldflags`代表链接器标志。之所以这样称呼它，是因为它将一个标志传递给底层的Go工具链链接器[`cmd/link`](https://golang.org/cmd/link)，允许您在构建时从命令行更改导入包的值。

在本教程中，您将使用`-ldflags`在构建时更改变量的值，并使用一个将版本信息打印到屏幕上的示例应用程序将您自己的动态信息引入到二进制文件中。

## 先决条件

要遵循本文中的示例，您需要：

- 一个Go工作区，通过下面的[步骤来设置如何安装Go和设置本地编程环境]。

## 构建示例应用程序

在使用`ldflags`引入动态数据之前，首先需要一个应用程序来插入信息。在这一步中，您将创建这个应用程序，它在这个阶段只打印静态版本信息。现在让我们创建这个应用程序。

在`src`目录中，创建一个以应用程序命名的目录。本教程将使用应用程序名称`app`：

```bash
mkdir app
```

将您的工作目录更改为此文件夹：

```bash
cd app
```

接下来，使用您选择的文本编辑器，创建程序的入口点`main.go`：

```bash
nano main.go
```

现在，通过添加以下内容，使您的应用程序打印出版本信息：

app/main.go

```go
package main

import (
 "fmt"
)

var Version = "development"

func main() {
 fmt.Println("Version:\t", Version)
}
```

在`main()`函数中，声明了`Version`变量，然后输出[字符串]`Version:`，后跟制表符`\t`，然后输出声明的变量。

此时，变量`Version`被定义为`development`，这将是此应用的默认版本。稍后，您将根据[语义版本格式](https://semver.org/)将此值更改为正式版本号。

保存并退出文件。完成后，构建并运行应用程序以确认它打印正确的版本：

```bash
go build
./app
```

您将看到以下输出：

```bash
Output
Version:  development
```

您现在有了一个打印默认版本信息的应用程序，但是您还没有办法在构建时传递当前版本信息。在下一步中，您将使用`-ldflags`和`go build`来解决此问题。

## 使用`ldflags`和`go build`

如前所述，`ldflags`代表链接器标志，用于将标志传递给Go工具链中的底层链接器。这是根据以下语法工作的：

```bash
go build -ldflags="-flag"
```

在这个例子中，我们将`flag`传递给作为`go tool link`的一部分运行的底层`go build`命令。这个命令在传递给`ldflags`的内容周围使用双引号，以避免其中出现中断字符，或者命令行可能会将其解释为我们不想要的字符。从这里，您可以传入[许多不同的`link`标志](https://golang.org/cmd/link/)。在本教程中，我们将使用`-X`标志在链接时将信息写入变量，然后是变量的[包]路径及其新值：

```bash
go build -ldflags="-X 'package_path.variable_name=new_value'"
```

在引号内，现在有`-X`选项和一个[键值对]，表示要更改的变量及其新值。`.`字符分隔包路径和变量名，单引号用于避免键值对中的中断字符。

要替换示例应用程序中的`Version`变量，请使用最后一个命令块中的语法传入新值并构建新的二进制文件：

```bash
go build -ldflags="-X 'main.Version=v1.0.0'"
```

在这个命令中，`main`是`Version`变量的包路径，因为这个变量在`main.go`文件中。`Version`是你要写入的变量，`v1.0.0`是新值。

为了使用`ldflags`，您想要更改的值必须存在，并且是类型为`string`的包级变量。此变量可以导出或取消导出。该值不能是`const`或由函数调用的结果设置其值。幸运的是，`Version`符合所有这些要求：它已经在`main.go`文件中声明为变量，并且当前值（`development`）和期望值（`v1.0.0`）都是字符串。

构建新的`app`二进制文件后，运行应用程序：

```bash
./app
```

您将收到以下输出：

```bash
Output
Version:  v1.0.0
```

使用`-ldflags`，您已经成功地将`Version`变量从`development`更改为`v1.0.0`。

现在，您已经在构建时修改了一个简单应用程序内部的`string`变量。使用`ldflags`，您可以仅使用命令行将版本详细信息、许可信息等嵌入到可供分发的二进制文件中。

在本例中，您更改的变量位于`main`程序中，从而降低了确定路径名的难度。但有时候，这些变量的路径更难找到。在下一步中，您将向子包中的变量写入值，以演示确定更复杂的包路径的最佳方法。

## 目标子包变量

在上一节中，您操作了`Version`变量，它位于应用程序的顶层包中。但情况并非总是如此。通常将这些变量放在另一个包中更实用，因为`main`不是一个可导入的包。为了在您的示例应用程序中模拟这一点，您将创建一个新的子包`app/build`，它将存储有关构建二进制文件的时间和发出构建命令的用户的名称的信息。

要添加一个新的子包，首先将一个名为`build`的新目录添加到您的项目中：

```bash
mkdir -p build
```

然后创建一个名为`build.go`的新文件来保存新变量：

```bash
nano build/build.go
```

在文本编辑器中，为`Time`和`User`添加新变量：

app/build/build.go

```go
package build

var Time string

var User string
```

`Time`变量将保存二进制文件构建时间的字符串表示。`User`变量将保存构建二进制文件的用户的名称。由于这两个变量总是有值，所以你不需要像对`Version`那样用默认值初始化这些变量。

保存并退出文件。

接下来，打开`main.go`将这些变量添加到应用程序中：

```bash
nano main.go
```

在`main.go`中，添加以下突出显示的行：

main.go

```
package main

import (
 "app/build"
 "fmt"
)

var Version = "development"

func main() {
 fmt.Println("Version:\t", Version)
 fmt.Println("build.Time:\t", build.Time)
 fmt.Println("build.User:\t", build.User)
}
```

在这几行中，您首先导入了`app/build`包，然后以与打印`build.Time`相同的方式打印了`build.User`和`Version`。

保存文件，然后退出文本编辑器。

接下来，要使用`ldflags`来定位这些变量，可以使用导入路径`app/build`，后面跟着`.User`或`.Time`，因为您已经知道导入路径。然而，为了模拟一个更复杂的情况，即变量的路径不明显，让我们使用Go工具链中的`nm`命令。

`go tool nm`命令将输出给定的可执行文件、目标文件或归档文件中涉及的符号。在这种情况下，符号引用代码中的对象，例如定义或导入的变量或函数。通过使用`nm`生成符号表并使用`grep`搜索变量，您可以快速找到有关其路径的信息。

注意：如果软件包名称包含任何非[ASCII](https://en.wikipedia.org/wiki/ASCII)字符，或者是`nm`或`"`字符，则`%`命令将无法帮助您找到变量的路径，因为这是工具本身的限制。

要使用此命令，首先构建`app`的二进制文件：

```bash
go build
```

现在已经构建了`app`，将`nm`工具指向它并搜索输出：

```bash
go tool nm ./app | grep app
```

运行时，`nm`工具将输出大量数据。因此，前面的命令使用`|`将输出传递给`grep`命令，然后由该命令搜索标题中具有顶级`app`的术语。

您将收到类似于以下内容的输出：

```
Output
  55d2c0 D app/build.Time
  55d2d0 D app/build.User
  4069a0 T runtime.appendIntStr
  462580 T strconv.appendEscapedRune
. . .
```

在本例中，结果集的前两行包含您要查找的两个变量的路径：`app/build.Time`和`app/build.User`。

现在您知道了路径，再次构建应用程序，这次在构建时更改`Version`、`User`和`Time`。为此，将多个`-X`标志传递给`-ldflags`：

```bash
go build -v -ldflags="-X 'main.Version=v1.0.0' -X 'app/build.User=$(id -u -n)' -X 'app/build.Time=$(date)'"
```

在这里，您传入了`id -u -n` Bash命令来列出当前用户，传入了`date`命令来列出当前日期。

构建可执行文件后，运行程序：

```bash
./app
```

在Unix系统上运行此命令时，将生成类似于以下内容的输出：

```
Output
Version:  v1.0.0
build.Time:  Fri Oct  4 19:49:19 UTC 2019
build.User:  sammy
```

现在，您有了一个包含版本控制和构建信息的二进制文件，可以在解决问题时为生产提供重要帮助。

## 结论

本教程展示了在正确应用时，`ldflags`如何成为在构建时将有价值的信息注入二进制文件的强大工具。通过这种方式，您可以控制特性标志、环境信息、版本信息等，而无需对源代码进行更改。通过将`ldflags`添加到您当前的构建工作流中，您可以最大限度地发挥Go自包含的二进制分发格式的优势。
