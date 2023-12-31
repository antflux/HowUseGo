# 如何构建和安装 Go 程序

## 介绍

到目前为止，在我们[的How To Code in Go系列]中，您已经使用命令[`go run`]自动编译源代码并运行生成的可执行文件。虽然此命令对于在命令行上测试代码很有用，但分发或部署应用程序需要将代码构建到可共享的二进制可执行文件中，或者构建到包含可运行应用程序的机器字节代码的单个文件中。为此，您可以使用Go工具链来构建和安装程序。

在Go语言中，将源代码转换为二进制可执行文件的过程称为构建。一旦构建了这个可执行文件，它将不仅包含您的应用程序，还包含在目标平台上执行二进制文件所需的所有支持代码。这意味着Go二进制文件不需要系统依赖项，如Go工具在新系统上运行。将这些可执行文件放在您自己系统上的可执行文件路径中，将允许您从系统上的任何位置运行程序。这与将程序安装到系统上是一样的。

在本教程中，您将使用Go工具链来运行、构建和安装一个示例`Hello, World!`程序，使您能够有效地使用、分发和部署未来的应用程序。

## 先决条件

要遵循本文中的示例，您需要：

- 一个Go工作区，通过下面的[步骤来设置如何安装Go和设置本地编程环境]。

## 步骤1 -设置和运行Go二进制文件

首先，创建一个应用程序作为演示Go工具链的示例。为此，您将使用经典的“Hello，World！”[如何在Go中编写你的第一个程序]教程。

在`greeter`目录中创建一个名为`src`的目录：

```bash
mkdir greeter
```

接下来，移动到新创建的目录中，并在您选择的文本编辑器中创建`main.go`文件：

```bash
cd greeter
nano main.go
```

打开文件后，添加以下内容：

src/greeter/main.go

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

当运行时，这个程序将打印短语`Hello, World!`到控制台，然后程序将成功退出。

保存并退出文件。

要测试程序，请使用`go run`命令，就像您在以前的教程中所做的那样：

```bash
go run main.go
```

您将收到以下输出：

```
Output
Hello, World!
```

如前所述，`go run`命令将源文件构建为可执行的二进制文件，然后运行编译后的程序。然而，本教程旨在以一种您可以随意共享和分发的方式构建二进制文件。为此，您将在下一步中使用`go build`命令。

## 步骤2 -创建Go模块以构建Go二进制文件

Go程序和库是围绕模块的核心概念构建的。模块包含有关程序使用的库以及要使用的这些库的版本的信息。

为了告诉Go这是一个Go模块，你需要使用`go mod`命令[创建一个Go模块]：

```bash
go mod init greeter
```

这将创建文件`go.mod`，其中将包含模块的名称以及用于构建它的Go版本。

```
Output
go: creating new go.mod: module greeter
go: to add module requirements and sums:
 go mod tidy
```

Go将提示您运行`go mod tidy`，以便在将来更改此模块的要求时更新它们。现在运行它不会有任何额外的效果。

## 第3步-使用`go build`构建Go二进制文件

使用`go build`，您可以为我们的示例Go应用程序生成一个可执行的二进制文件，允许您在您想要的地方分发和部署程序。

试试这个#1。在`main.go`目录中，运行以下命令：

```bash
go build
```

如果你不给这个命令提供参数，`go build`会自动编译你当前目录下的`main.go`程序。该命令将包括目录中的所有`*.go`文件。它还将构建所有支持代码，以便能够在具有相同系统架构的任何计算机上执行二进制文件，无论该系统是否具有`.go`源文件，甚至是Go安装。

在本例中，您将`greeter`应用程序构建到一个可执行文件中，并将其添加到当前目录中。通过运行`ls`命令来检查这一点：

```bash
ls
```

如果您运行的是macOS或Linux，您将发现一个新的可执行文件，该文件以您构建程序的目录命名：

```
Output
greeter  main.go  go.mod
```

注意：在Windows上，您的可执行文件将是`greeter.exe`。

默认情况下，`go build`将为当前[平台和架构]生成一个可执行文件。例如，如果构建在`linux/386`系统上，可执行文件将与任何其他`linux/386`系统兼容，即使没有安装Go。Go支持为其他平台和架构构建，您可以在我们[的为不同操作系统和架构构建Go应用]程序文章中阅读更多信息。

现在，您已经创建了可执行文件，请运行它以确保正确构建了二进制文件。在macOS或Linux上，运行以下命令：

```bash
./greeter
```

在Windows上，运行：

```bash
greeter.exe
```

二进制文件的输出将与使用`go run`运行程序时的输出相匹配：

```
Output
Hello, World!
```

现在，您已经创建了一个可执行的二进制文件，其中不仅包含您的程序，还包含运行该二进制文件所需的所有系统代码。您现在可以将此程序分发到新系统或部署到服务器，因为该文件将始终运行相同的程序。

在下一节中，本教程将解释二进制文件是如何命名的，以及如何更改它，以便您可以更好地控制程序的构建过程。

## 步骤4 -更改二进制名称

现在您已经知道如何生成可执行文件，下一步是确定Go如何为二进制文件选择名称，并为您的项目定制此名称。

运行`go build`时，默认情况下Go自动决定生成的可执行文件的名称。它通过使用您之前创建的模块来实现这一点。当运行`go mod init greeter`命令时，它创建了名为“greeter”的模块，这就是为什么生成的二进制文件被命名为`greeter`。

让我们仔细看看模块方法。如果你的项目中有一个带有`go.mod`声明的`module`文件，如下所示：

go.mod

```
module github.com/sammy/shark
```

那么生成的可执行文件的默认名称将是`shark`。

在需要特定命名约定的更复杂的程序中，这些默认值并不总是命名二进制文件的最佳选择。在这些情况下，最好使用`-o`标志自定义输出。

为了测试这一点，将您在上一节中创建的可执行文件的名称更改为`hello`，并将其放置在名为`bin`的子文件夹中。您不必创建此文件夹; Go将在构建过程中自行创建。

运行以下带有`go build`标志的`-o`命令：

```bash
go build -o bin/hello
```

`-o`标志使Go将命令的输出与您选择的任何参数相匹配。在本例中，结果是一个名为`hello`的子文件夹中的名为`bin`的新可执行文件。

要测试新的可执行文件，请切换到新目录并运行二进制文件：

```bash
cd bin
./hello
```

您将收到以下输出：

```
Output
Hello, World!
```

现在，您可以自定义可执行文件的名称以满足项目的需求，完成我们关于如何在Go中构建二进制文件的调查。但是使用`go build`，您仍然只能从当前目录运行二进制文件。为了使用新构建的可执行文件从任何地方在您的系统，您可以安装他们使用`go install`。

## 步骤5 -使用`go install`安装Go程序

到目前为止，在本文中，我们已经讨论了如何从`.go`源文件生成可执行的二进制文件。这些可执行文件有助于分发、部署和测试，但它们还不能从其源目录外部执行。如果您想在shell脚本或其他工作流中积极使用程序，这将是一个问题。为了使程序更容易使用，您可以将它们安装到您的系统中，并从任何地方访问它们。

为了理解这意味着什么，您将使用`go install`命令来安装示例应用程序。

`go install`命令的行为几乎与`go build`相同，但不是将可执行文件留在当前目录或由`-o`标志指定的目录中，而是将可执行文件放入`$GOPATH/bin`目录。

要查找`$GOPATH`目录的位置，请运行以下命令：

```bash
go env GOPATH
```

您收到的输出会有所不同，但默认值是`go`目录中的`$HOME`目录：

```
Output
$HOME/go
```

由于`go install`会将生成的可执行文件放在`$GOPATH`的子目录`bin`中，因此必须将该目录添加到`$PATH`环境变量中。这在必备文章[How To Install Go and Set Up a Local Programming Environment的]创建Go工作区步骤中有所介绍。

设置好`$GOPATH/bin`目录后，移回`greeter`目录：

```bash
cd ..
```

现在运行install命令：

```bash
go install
```

这将构建您的二进制文件并将其放置在`$GOPATH/bin`中。要对此进行测试，请运行以下命令：

```bash
ls $GOPATH/bin
```

这将列出`$GOPATH/bin`的内容：

```
Output
greeter
```

注意：`go install`命令不支持`-o`标志，因此它将使用前面描述的默认名称来命名可执行文件。

安装二进制文件后，测试程序是否会从其源目录外部运行。返回到您的主目录：

```bash
cd $HOME
```

使用以下命令运行程序：

```bash
greeter
```

这将产生以下结果：

```
Output
Hello, World!
```

现在，您可以将自己编写的程序安装到系统中，以便随时随地使用它们。

## 结论

在本教程中，您演示了Go工具链如何轻松地从源代码构建可执行二进制文件。这些二进制文件可以被分发到其他系统上运行，甚至是那些没有Go工具和环境的系统。您还使用`go install`自动构建并安装我们的程序作为系统`$PATH`中的可执行文件。使用`go build`和`go install`，您现在可以随意共享和使用您的应用程序。
