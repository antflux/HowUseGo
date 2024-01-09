# 为不同操作系统和架构构建 Go 应用程序

在软件开发中，考虑您想要编译二进制文件的[操作系统](https://en.wikipedia.org/wiki/Operating_system)和底层处理器[架构](https://en.wikipedia.org/wiki/Microarchitecture)是很重要的。由于在不同的操作系统/架构平台上运行二进制文件通常很慢或不可能，因此通常的做法是为许多不同的平台构建最终的二进制文件，以最大限度地增加程序的受众。但是，当您用于开发的平台与您希望将程序部署到的平台不同时，这可能会很困难。例如，在过去，在Windows上开发一个程序并将其部署到Linux或macOS机器上，将涉及为您想要二进制文件的每个环境设置构建机器。您还需要保持工具的同步，以及其他可能增加成本并使协作测试和分发更加困难的考虑因素。

Go通过直接在`go build`工具以及Go工具链的其余部分中构建对多平台的支持来解决这个问题。通过使用[环境变量]和[构建标记]，您可以控制最终二进制文件是为哪个操作系统和体系结构构建的，此外还可以整合一个工作流，可以快速切换包含平台相关代码而无需更改代码库。

在本教程中，您将组合一个示例应用程序，该应用程序将[字符串]连接到文件路径中，创建并选择性地包含平台相关的片段，并在您自己的系统上为多个操作系统和系统架构构建二进制文件，向您展示如何使用Go编程语言的强大功能。

## 先决条件

要遵循本文中的示例，您需要：

- 一个Go工作区，通过下面的[步骤来设置如何安装Go和设置本地编程环境]。

## `GOOS`和`GOARCH`的可能平台

在展示如何控制构建过程为不同的平台构建二进制文件之前，让我们首先检查Go能够为哪些平台构建，以及Go如何使用环境变量`GOOS`和`GOARCH`引用这些平台。

Go工具有一个命令可以打印Go可以构建的平台列表，这个列表可以随着每个新的Go版本而改变，所以这里讨论的组合在另一个版本的Go上可能不一样。在撰写本教程时，当前的Go版本是[`1.13`](https://golang.org/doc/go1.13)。

要查找此可能平台的列表，请运行以下命令：

```bash
go tool dist list
```

您将收到类似以下内容的输出：

```
Output

aix/ppc64        freebsd/amd64   linux/mipsle   openbsd/386
android/386      freebsd/arm     linux/ppc64    openbsd/amd64
android/amd64    illumos/amd64   linux/ppc64le  openbsd/arm
android/arm      js/wasm         linux/s390x    openbsd/arm64
android/arm64    linux/386       nacl/386       plan9/386
darwin/386       linux/amd64     nacl/amd64p32  plan9/amd64
darwin/amd64     linux/arm       nacl/arm       plan9/arm
darwin/arm       linux/arm64     netbsd/386     solaris/amd64
darwin/arm64     linux/mips      netbsd/amd64   windows/386
dragonfly/amd64  linux/mips64    netbsd/arm     windows/amd64
freebsd/386      linux/mips64le  netbsd/arm64   windows/arm
```

此输出是一组由`/`分隔的键值对。组合的第一部分，在`/`之前，是操作系统。在Go中，这些操作系统是环境变量`GOOS`的可能值，发音为“goose”，代表Go Operating System。第二部分，在#4之后，是架构。和前面一样，这些都是环境变量的可能值：`/`。它的发音是“gore-ch”，代表Go Architecture。

让我们以`linux/386`为例，分解其中一个组合，以了解它的含义和工作原理。键值对以`GOOS`开始，在本例中是`linux`，指的是[Linux OS]。这里的`GOARCH`是`386`，代表[Intel 80386微处理器](https://en.wikipedia.org/wiki/Intel_80386)。

有许多平台可以使用`go build`命令，但大多数情况下，您最终会使用`linux`、`windows`或`darwin`作为`GOOS`的值。这些涵盖了三大操作系统平台：[Linux](https://en.wikipedia.org/wiki/Linux)，[Windows](https://en.wikipedia.org/wiki/Microsoft_Windows)和[macOS](https://en.wikipedia.org/wiki/MacOS)，后者基于[达尔文操作系统](https://en.wikipedia.org/wiki/Darwin_(operating_system))，因此被称为`darwin`。然而，Go也可以覆盖不太主流的平台，比如`nacl`，它代表[了Google](https://developer.chrome.com/native-client)

当你运行像`go build`这样的命令时，Go使用当前平台的`GOOS`和`GOARCH`来确定如何构建二进制文件。要了解您的平台是什么组合，您可以使用`go env`命令并将`GOOS`和`GOARCH`作为参数传递：

```bash
go env GOOS GOARCH
```

在测试此示例时，我们在具有[AMD 64架构](https://en.wikipedia.org/wiki/X86-64)的机器上的macOS上运行此命令，因此我们将收到以下输出：

```
Output

darwin
amd64
```

在这里，命令的输出告诉我们，我们的系统有`GOOS=darwin`和`GOARCH=amd64`。

现在你知道了Go中的`GOOS`和`GOARCH`是什么，以及它们可能的值。接下来，您将编写一个程序，作为如何使用这些环境变量和构建标记为其他平台构建二进制文件的示例。

## 用`filepath.Join()`写一个依赖于平台的程序

在开始为其他平台构建二进制文件之前，让我们构建一个示例程序。一个很好的例子是Go标准库中[`Join`](https://godoc.org/path/filepath)包中的`path/filepath`函数。此函数接受多个字符串，并返回一个用正确的文件路径分隔符连接在一起的字符串。

这是一个很好的示例程序，因为程序的操作取决于它运行在哪个操作系统上。在Windows上，路径分隔符是反斜杠，`\`，而基于Unix的系统使用正斜杠，`/`。

让我们从构建一个使用`filepath.Join()`的应用程序开始，然后，您将编写自己的`Join()`函数实现，该函数将代码定制为特定于平台的二进制文件。

首先，在您的`src`目录中创建一个文件夹，并使用您的应用名称：

```bash
mkdir app
```

进入该目录：

```bash
cd app
```

接下来，在您选择的文本编辑器中创建一个名为`main.go`的新文件。在本教程中，我们将使用Nano：

```bash
nano main.go
```

打开文件后，添加以下代码：

src/app/main.go

```go
package main

import (
  "fmt"
  "path/filepath"
)

func main() {
  s := filepath.Join("a", "b", "c")
  fmt.Println(s)
}
```

此文件中的`main()`函数使用`filepath.Join()`将三[个字符串]与正确的、依赖于平台的路径分隔符连接在一起。

保存并退出文件，然后运行程序：

```bash
go run main.go
```

运行此程序时，您将收到不同的输出，具体取决于您使用的平台。在Windows上，您将看到由`\`分隔的字符串：

```
Output

a\b\c
```

在macOS和Linux等Unix系统上，您将收到以下内容：

```
Output

a/b/c
```

这表明，由于这些操作系统上使用的文件系统协议不同，程序将不得不为不同的平台构建不同的代码。但是由于它已经根据操作系统使用了不同的文件分隔符，我们知道`filepath.Join()`已经考虑了平台的差异。这是因为Go工具链会自动检测您的机器

让我们考虑一下`filepath.Join()`函数从哪里获得分隔符。运行以下命令，检查Go语言标准库中的相关代码段：

```bash
less /usr/local/go/src/os/path_unix.go
```

这将显示`path_unix.go`的内容。查找文件的以下部分：

/usr/local/go/os/path_unix.go

```go
. . .
// +build aix darwin dragonfly freebsd js,wasm linux nacl netbsd openbsd solaris

package os

const (
  PathSeparator     = '/' // OS-specific path separator
  PathListSeparator = ':' // OS-specific path list separator
)
. . .
```

本节定义了Go支持的所有类Unix系统的`PathSeparator`。请注意顶部的所有构建标记，它们是与Unix相关的可能的Unix `GOOS`平台之一。当`GOOS`匹配这些术语时，您的程序将产生Unix风格的文件路径分隔符。

按`q`返回命令行。

接下来，打开定义`filepath.Join()`在Windows上使用时的行为的文件：

```bash
less /usr/local/go/src/os/path_windows.go
```

您将看到以下内容：

/usr/local/go/src/os/path_windows.go

```go
. . .
package os

const (
        PathSeparator     = '\\' // OS-specific path separator
        PathListSeparator = ';'  // OS-specific path list separator
)
. . .
```

虽然这里`PathSeparator`的值是`\\`，但代码将呈现Windows文件路径所需的单个反斜杠（`\`），因为第一个反斜杠只需要作为转义字符。

请注意，与Unix文件不同，顶部没有构建标记。这是因为`GOOS`和`GOARCH`也可以通过添加下划线（`go build`）和环境变量值作为文件名的后缀传递给`_`，我们将在[使用GOOS和GOOS文件名后缀]一节中详细介绍。在这里，`_windows`的`path_windows.go`部分使文件的行为就像它在文件顶部有构建标记`// +build windows`一样。正因为如此，当你的程序在Windows上运行时，它将使用来自`PathSeparator`代码片段的常量`PathListSeparator`和`path_windows.go`。

要返回到命令行，请按`less`退出`q`。

在这一步中，您构建了一个程序，展示了Go如何将`GOOS`和`GOARCH`自动转换为构建标记。考虑到这一点，您现在可以更新程序并编写自己的`filepath.Join()`实现，使用构建标记手动设置Windows和Unix平台的正确`PathSeparator`。

## 实现特定于平台的功能

现在你已经知道了Go的标准库是如何实现特定于平台的代码的，你可以在你自己的`app`程序中使用构建标签来实现这一点。要做到这一点，你需要编写自己的`filepath.Join()`实现。

打开您的`main.go`文件：

```bash
nano main.go
```

使用您自己的函数`main.go`，将`Join()`的内容替换为以下内容：

src/app/main.go

```go
package main

import (
  "fmt"
  "strings"
)

func Join(parts ...string) string {
  return strings.Join(parts, PathSeparator)
}

func main() {
  s := Join("a", "b", "c")
  fmt.Println(s)
}
```

`Join`函数接受若干个`parts`，并使用[`strings.Join()`包]中的[`strings`](https://godoc.org/strings.Join)方法将它们连接在一起，然后使用`parts`将`PathSeparator`连接在一起。

您还没有定义`PathSeparator`，所以现在在另一个文件中进行定义。保存并退出`main.go`，打开您最喜欢的编辑器，然后创建一个名为`path.go`的新文件：

```
nano path.go
```

定义`PathSeparator`并将其设置为Unix文件路径分隔符`/`：

src/app/path.go

```go
package main

const PathSeparator = "/"
```

编译并运行应用程序：

```bash
go build
./app
```

您将收到以下输出：

```
Output

a/b/c
```

这将成功运行以获得Unix风格的文件路径。但这还不是我们想要的：输出总是`a/b/c`，不管它在什么平台上运行。要添加创建Windows风格文件路径的功能，您需要添加`PathSeparator`的Windows版本，并告诉`go build`命令使用哪个版本。在下一节中，您将使用[构建标记]来完成此任务。

## 使用`GOOS`或`GOARCH`构建标记

要考虑Windows平台，您现在将为`path.go`创建一个备用文件，并使用构建标记来确保代码片段仅在`GOOS`和`GOARCH`是适当的平台时运行。

但首先，在`path.go`中添加一个构建标签，告诉它构建除Windows之外的所有内容。打开文件：

```bash
nano path.go
```

将以下突出显示的构建标记添加到文件中：

src/app/path.go

```
// +build !windows

package main

const PathSeparator = "/"
```

Go build标记允许反转，这意味着您可以指示Go为除Windows之外的任何平台构建此文件。要反转构建标记，请在标记前放置`!`。

保存并退出文件。

现在，如果你在Windows上运行这个程序，你会得到以下错误：

```
Output

./main.go:9:29: undefined: PathSeparator
```

在这种情况下，Go语言将无法包含`path.go`来定义变量`PathSeparator`。

现在你已经确保了当`path.go`是Windows时`GOOS`不会运行，添加一个新文件，`windows.go`：

```bash
nano windows.go
```

在`windows.go`中，定义Windows `PathSeparator`，以及一个构建标签，让`go build`命令知道它是Windows实现：

src/app/windows.go

```go
// +build windows

package main

const PathSeparator = "\\"
```

保存文件并退出文本编辑器。应用程序现在可以为Windows编译一种方式，为所有其他平台编译另一种方式。

虽然二进制文件现在可以为它们的平台正确地构建，但为了为您无法访问的平台进行编译，您必须进行进一步的更改。要做到这一点，您将在下一步中更改本地`GOOS`和`GOARCH`环境变量。

## 使用本地`GOOS`和`GOARCH`环境变量

之前，您运行了`go env GOOS GOARCH`命令以了解您正在使用的操作系统和架构。当您运行`go env`命令时，它会查找两个环境变量`GOOS`和`GOARCH`;如果找到，将使用它们的值，但如果找不到，则Go将使用当前平台的信息设置它们。这意味着您可以更改`GOOS`或`GOARCH`，使其不默认为您的本地操作系统和体系结构。

`go build`命令的行为方式与`go env`命令类似。您可以使用`GOOS`设置`GOARCH`或`go build`环境变量以构建不同的平台。

如果您使用的不是Windows系统，请在运行`windows`命令时通过将`app`环境变量设置为`GOOS`来构建`windows`的`go build`二进制文件：

```bash
GOOS=windows go build
```

现在列出当前目录中的文件：

```bash
ls
```

列出目录的输出显示项目目录中现在有一个`app.exe` Windows可执行文件：

```
Output

app  app.exe  main.go  path.go  windows.go
```

使用`file`命令，您可以获得有关此文件的更多信息，确认其构建：

```bash
file app.exe
```

您将收到：

```
Output

app.exe: PE32+ executable (console) x86-64 (stripped to external PDB), for MS Windows
```

您还可以在构建时设置一个或两个环境变量。运行以下命令：

```bash
GOOS=linux GOARCH=ppc64 go build
```

您的`app`可执行文件现在将被用于不同架构的文件替换。在此二进制文件上运行`file`命令：

```bash
file app
```

您将收到如下输出：

```
app: ELF 64-bit MSB executable, 64-bit PowerPC or cisco 7500, version 1 (SYSV), statically linked, not stripped
```

通过设置本地的`GOOS`和`GOARCH`环境变量，你现在可以为任何Go的兼容平台构建二进制文件，而无需复杂的配置或设置。接下来，您将使用文件名约定来保持文件的整齐组织，并自动为特定的平台构建，而无需构建标记。

## 使用`GOOS`和`GOARCH`Film后缀

如前所述，Go标准库大量使用构建标记，通过将不同的平台实现分隔到不同的文件中来简化代码。当您打开`os/path_unix.go`文件时，会看到一个build标记，其中列出了所有被视为类Unix平台的可能组合。然而，`os/path_windows.go`文件没有包含构建标记，因为文件名的后缀足以告诉Go文件是针对哪个平台的。

让我们看看这个特性的语法。命名`.go`文件时，您可以按此顺序将`GOOS`和`GOARCH`作为后缀添加到文件名中，并用下划线（`_`）分隔值。如果你有一个名为`filename.go`的Go文件，你可以通过将文件名改为`filename_GOOS_GOARCH.go`来指定操作系统和架构。例如，如果您希望在64位[ARM架构的](https://en.wikipedia.org/wiki/ARM_architecture)Windows上编译它，则可以将文件名设置为`filename_windows_arm64.go`。这种命名约定有助于保持代码的整洁。

更新您的程序以使用文件名后缀而不是构建标记。首先，重命名`path.go`和`windows.go`文件，以使用`os`包中使用的约定：

```bash
mv path.go path_unix.go
mv windows.go path_windows.go
```

更改两个文件名后，您可以删除添加到`path_windows.go`的构建标记：

```bash
nano path_windows.go
```

删除`// +build windows`，使您的文件看起来像这样：

path_windows.go

```go
package main

const PathSeparator = "\\"
```

保存并退出文件。

因为`unix`不是有效的`GOOS`，所以`_unix.go`后缀对Go编译器没有意义。但是，它确实传达了文件的预期目的。与`os/path_unix.go`文件一样，您的`path_unix.go`文件仍然需要使用构建标记，因此保持该文件不变。

通过使用文件名约定，您从源代码中删除了不可复制的构建标记，并使文件系统更干净、更清晰。

## 结论

为多个平台生成不需要依赖的二进制文件的能力是Go工具链的一个强大功能。在本教程中，您通过添加构建标记和文件名后缀来使用此功能，以将某些代码段标记为仅针对某些体系结构进行编译。您创建了自己的依赖于平台的程序，然后操纵`GOOS`和`GOARCH`环境变量来为当前平台以外的平台生成二进制文件。这是一项很有价值的技能，因为拥有一个持续集成过程是一种常见的做法，该过程自动运行这些环境变量，以构建适用于所有平台的二进制文件。
