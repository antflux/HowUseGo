# 如何使用 Go 模块

## 导言

在 1.13 版中，Go 的作者添加了一种管理 Go 项目所依赖库的新方法，称为[Go 模块](https://golang.org/ref/mod)。Go 模块的添加是为了满足日益增长的需求，使开发人员更容易维护其依赖库的各种版本，并使开发人员在计算机上组织项目的方式更具灵活性。Go 模块通常由一个项目或库组成，包含一组 Go 包，然后一起发布。Go 模块解决了原始系统[`GOPATH`](https://golang.org/doc/gopath_code) 的许多问题，允许用户将项目代码放在自己选择的目录中，并为每个模块指定依赖的版本。

在本教程中，您将创建自己的 Go 公共模块，并为新模块添加软件包。此外，你还将把别人的公共模块添加到自己的项目中，并把该模块的特定版本添加到自己的项目中。

## 先决条件

要学习本教程，您需要

- 已安装 Go 1.16 或更高版本。要进行此设置，请按照操作系统的 "[如何安装 Go]"教程进行操作。
- 熟悉用 Go 编写软件包。要了解更多信息，请参阅[如何用 Go 编写]软件包教程。

## 创建新模块

乍一看，Go 模块与[Go 软件包]很相似。模块有许多 Go 代码文件来实现软件包的功能，但它的根目录中还有两个额外的重要文件：go`.mod`文件和`go.sum`文件。这两个文件包含`go`工具用来跟踪模块配置的信息，通常由工具维护，因此你不需要维护。

首先要确定模块所在的目录。随着 Go 模块的引入，Go 项目可以位于文件系统的任何位置，而不仅仅是 Go 定义的特定目录。你可能已经有了一个项目目录，但在本教程中，你将创建一个名为`projects`的目录，新模块将被命名为`mymodule`。你可以通过集成开发环境或命令行创建`projects`目录。

如果使用命令行，首先创建`projects`目录并导航到该目录：

```bash
mkdir projects
cd projects
```

接下来，创建模块目录。通常情况下，模块的顶层目录名称与模块名称相同，这样更容易跟踪。在你的`projects`目录下，运行以下命令创建`mymodule`目录：

```bash
mkdir mymodule
```

创建模块目录后，目录结构将如下所示：

```
└── projects
    └── mymodule
```

下一步是在`mymodule`目录下创建`go.mod`文件，以定义 Go 模块本身。为此，你需要使用`go`工具的`mod init`命令，并提供模块的名称，在本例中就是`mymodule`。现在从`mymodule`目录运行`go mod init`命令创建模块，并提供模块名称`mymodule`：

```bash
go mod init mymodule
```

创建模块时，该命令将返回以下输出：

```bash
Output
go: creating new go.mod: module mymodule
```

创建模块后，您的目录结构将如下所示：

```
└── projects
    └── mymodule
        └── go.mod
```

创建了模块后，让我们来看看`go.mod`文件，看看`go mod init`命令都做了些什么。

### 了解`go.mod`文件]

使用`go`工具运行命令时，`go.mod`文件是整个过程中非常重要的一部分。该文件包含模块名称和模块依赖的其他模块的版本。它还可以包含其他指令，如[`replace`](https://golang.org/ref/mod#go-mod-file-replace)，这对同时开发多个模块很有帮助。

在`mymodule`目录中，用`nano` 或你喜欢的文本编辑器打开`go.mod`文件：

```bash
vim go.mod
```

内容看起来与此类似，但并不多：

```go title="projects/mymodule/go.mod"
module mymodule

go 1.16
```

第一行是`module`指令，它告诉 Go 模块的名称，这样当 Go 在软件包中查找`导入`路径时，就不会在其他地方查找`mymodule`。在 `mymodule`的值来自你传递给`go mod init` 的参数：

```go
module mymodule
```

此时，文件中唯一的其他行，即`go`指令，会告诉 Go 模块使用的语言版本。在本例中，由于模块是使用 Go 1.16 创建的，所以`go`指令写的是`1.16`：

```go
go 1.16
```

随着更多信息被添加到模块中，该文件也会随之扩展，但最好现在就查看一下，看看随着依赖关系的进一步添加，该文件会发生怎样的变化。

现在，您已经使用 `go mod init`创建了一个 Go 模块，并查看了初始`go.mod`文件的内容，但您的模块还没有做任何事情。是时候进一步完善模块并添加一些代码了。

### 在模块中添加 Go 代码]

为了确保模块创建正确，并添加代码以便运行第一个 Go 模块，你需要在`mymodule`目录下创建一个`main.go`文件。 在 Go 程序中，main`.`go 文件通常用来表示程序的起点。文件名并不像里面的`main`函数那么重要，但两者匹配起来更容易找到。在本教程中，`main`函数运行时将打印出`Hello, Modules!`

要创建该文件，请使用`vim` 或您最喜欢的文本编辑器打开`main.go`文件：

```bash
vim main.go
```

在`main.go`文件中，添加以下代码以定义`主`软件包，导入`fmt`软件包，然后在`main`函数中打印出`Hello, Modules!`

```go title="projects/mymodule/main.go"
package main

import "fmt"

func main() {
 fmt.Println("Hello, Modules!")
}
```

在 Go 中，每个目录都被视为自己的包，每个文件都有自己的`package`声明行。在你刚刚创建的`main.go`文件中，`package`被命名为`main`。通常情况下，你可以以任何方式命名软件包，但`main`软件包在 Go 中是特殊的。当 Go 看到一个软件包被命名为`main`时，它就知道该软件包应被视为二进制文件，并应被编译为可执行文件，而不是一个设计用于其他程序的库。

在定义`package`后，`import`声明说要导入[`fmt`](https://pkg.go.dev/fmt)软件包，这样就可以使用其`Println`函数将`"Hello, Modules!`" 消息打印到屏幕上。

最后，定义了`main`函数。`main`函数是 Go 的另一个特例，与`main`包有关。当 Go 在名为 main 的包内看到一个名为`main` 的函数时，它就知道`main`函数是它应该运行的第一个函数。这就是所谓的程序入口点。

创建了`main.go`文件后，模块的目录结构将与下面相似：

```
└── projects
    └── mymodule
        └── go.mod
        └── main.go
```

如果你熟悉 Go 和[`GOPATH`](https://www.digitalocean.com/community/tutorials/understanding-the-gopath)，运行模块中的代码与在`GOPATH` 目录中运行代码类似。(如果你不熟悉`GOPATH`，也不用担心，因为使用模块可以替代它的用法）。

在 Go 中运行可执行程序有两种常见方法：用`go build`生成二进制文件或用`go run` 运行文件。在本教程中，你将使用`go`run 直接运行模块，而不是编译二进制文件，后者必须单独运行。

使用`go run 运行`您创建的`main.go`文件：

```bash
go run main.go
```

运行该命令将打印代码中定义的 "`Hello, Modules!`"文本：

```
Output
Hello, Modules!
```

在本节中，你为你的模块添加了一个`main` `.go`文件，其中包含一个打印`Hello, Modules!`此时，你的程序还没有从 Go 模块中受益--它可以是计算机上任何地方的一个文件，用`go` run 运行。Go 模块的第一个真正好处是可以在任何目录下为项目添加依赖项，而不仅仅是`GOPATH`目录结构。您还可以为模块添加软件包。在下一节中，你将通过在模块中创建额外的包来扩展你的模块。

## 为模块添加软件包

与标准 Go 包类似，模块可以包含任意数量的包和子包，也可以不包含任何包。在本例中，你将在`mymodule`目录下创建一个名为`mypackage`的包。

在`mymodule`目录下运行`mkdir`命令，以`mypackage`为参数，创建新软件包：

```bash
mkdir mypackage
```

复制

这将创建新目录`mypackage`，作为`mymodule`目录的子软件包：

```
└── projects
    └── mymodule
        └── mypackage
        └── main.go
        └── go.mod
```

使用`cd`命令将目录更改为新的`mypackage`目录，然后使用`nano` 或你喜欢的文本编辑器创建一个`mypackage.go`文件。该文件可以有任何名称，但使用与软件包相同的名称会更容易找到软件包的主文件：

```bash
cd mypackage
vim mypackage.go
```

在`mypackage.go`文件中，添加一个名为`PrintHello`的函数，打印`Hello, Modules`的信息`！`调用该函数时，将打印 "你好，模块们`！`"信息：

```go title="projects/mymodule/mypackage/mypackage.go"
package mypackage

import "fmt"

func PrintHello() {
 fmt.Println("Hello, Modules! This is mypackage speaking!")
}
```

复制

由于您希望`PrintHello`函数可以从其他程序包中使用，因此函数名中的大写字母`P`非常重要。大写字母意味着该函数已被导出，可被任何外部程序使用。有关 Go 中软件包可见性的更多信息，请参阅《[了解 Go 中的软件包可见性》。

现在你已经创建了带有导出函数的`mypackage`包，需要从`mymodule`包中`import`才能使用它。这与导入其他包（如之前的`fmt`包）的方法类似，只不过这次你要在导入路径的开头加上你的模块名称。打开`mymodule`目录中的`main.go`文件，添加下面突出显示的几行，调用`PrintHello`：

```go title="projects/mymodule/main.go"
package main

import (
 "fmt"

 "mymodule/mypackage"
)

func main() {
 fmt.Println("Hello, Modules!")

 mypackage.PrintHello()
}
```

仔细看一下`import`语句，你会发现新的导入以`mymodule` 开头，这与你在`go.mod`文件中设置的模块名相同。之后是路径分隔符和要导入的软件包，本例中是`mypackage`：

```go
"mymodule/mypackage"
```

今后，如果在`mypackage` 中添加软件包，也会以类似的方式将它们添加到导入路径的末尾。例如，如果在`mypackage` 中添加了名为`extrapackage`的软件包，那么该软件包的导入路径就是`mymodule/mypackage/extrapackage`。

和之前一样，在`mymodule`目录中使用`go run`和`main.go`运行更新后的模块：

```bash
go run main.go
```

再次运行模块时，您将看到之前的`Hello, Modules!`消息，以及新的`mypackage`的`PrintHello`函数打印出的新消息：

```
Output
Hello, Modules!
Hello, Modules! This is mypackage speaking!
```

通过创建一个名为`mypackage`并带有`PrintHello`函数的目录，您已经为初始模块添加了一个新包。不过，随着模块功能的扩展，开始在自己的模块中使用他人的模块可能会很有用。在下一节中，你将添加一个远程模块作为你的模块的依赖关系。

## 将远程模块添加为依赖项

Go 模块是从版本控制仓库（通常是 Git 仓库）发布的。当你想添加一个新模块作为自己的依赖时，你可以使用版本库的路径来引用你想使用的模块。当 Go 看到这些模块的导入路径时，它可以根据该版本库路径推断出在哪里可以远程找到它。

在本例中，您将在模块中添加对[`github.com/spf13/cobra`](https://github.com/spf13/cobra)库的依赖。Cobra 是创建控制台应用程序的常用库，但我们不会在本教程中讨论它。

与创建`mymodule`模块时类似，你将再次使用`go`工具。不过，这次你要从`mymodule`目录运行`go get`命令。运行`go get`并提供你要添加的模块。在本例中，你将得到`github.com/spf13/cobra`：

```bash
go getgithub.com/spf13/cobra
```

运行此命令后，`go`工具将从您指定的路径查找 Cobra 代码库，并通过查看代码库的分支和标记来确定哪个版本的 Cobra 是最新的。然后，它会下载该版本，并将模块名称和版本添加到`go.mod`文件中，以记录所选择的版本，供将来参考。

现在，打开`mymodule`目录中的`go.mod`文件，看看添加新依赖时`go`工具是如何更新 go`.mod`文件的。下面的示例可能会根据当前发布的 Cobra 版本或您使用的 Go 工具版本而发生变化，但总体变化结构应该是相似的：

```go title="projects/mymodule/go.mod"
module mymodule

go 1.16

require (
 github.com/inconshreveable/mousetrap v1.0.0 // indirect
 github.com/spf13/cobra v1.2.1 // indirect
 github.com/spf13/pflag v1.0.5 // indirect
)
```

新增了使用`require`指令的部分。该指令告诉 Go 你需要哪个模块，例如`github.com/spf13/cobra`，以及你添加的模块的版本。有时`require`指令还会包含`// 间接`注释。该注释表示，在添加`require`指令时，模块的源文件中没有直接引用该模块。文件中还添加了一些额外`的 require`行。这些行是 Cobra 依赖的其他模块，Go 工具认为这些模块也应该被引用。

您可能还注意到，在运行`go run`命令后，`mymodule`目录中创建了一个新文件 go`.sum`。这是 Go 模块的另一个重要文件，其中包含 Go 用来记录依赖项的特定哈希值和版本的信息。这可以确保依赖关系的一致性，即使它们安装在不同的机器上。

下载依赖项后，您需要用一些最基本的 Cobra 代码更新您的`main.go`文件，以使用新的依赖项。使用下面的 Cobra 代码更新`mymodule`目录中的`main.go`文件，以使用新的依赖关系：

```go title="projects/mymodule/main.go"
package main

import (
 "fmt"
    
 "github.com/spf13/cobra"

 "mymodule/mypackage"
)

func main() {
 cmd := &cobra.Command{
  Run: func(cmd *cobra.Command, args []string) {
   fmt.Println("Hello, Modules!")

   mypackage.PrintHello()
  },
 }

 fmt.Println("Calling cmd.Execute()!")
 cmd.Execute()
}
```

这段代码创建了一个`cobra.Command`结构，其中的`Run`函数包含现有的 "Hello "语句，调用`cmd.Execute()` 即可执行这些语句。现在，运行更新后的代码：

```bash
go run main.go
```

你将看到以下输出，与之前的输出类似。不过，正如`Calling cmd.Execute()!`一行所示，这次使用的是新的依赖关系：

```
Output
Calling cmd.Execute()!
Hello, Modules!
Hello, Modules! This is mypackage speaking!
```

使用`go get`添加远程依赖项的最新版本（例如`github.com/sp13/cobra`），可以更方便地将最新的错误修复更新到依赖项中。不过，有时你可能更希望使用模块的特定版本、版本库标签或版本库分支。在下一节中，你将使用`go get`来引用这些版本。

## 使用模块的特定版本

由于 Go 模块是从版本控制仓库发布的，因此它们可以使用版本控制功能，如标签、分支甚至提交。您可以在模块路径末尾使用`@`符号以及您希望使用的版本，在依赖关系中引用这些功能。早些时候，当你安装最新版本的 Cobra 时，就已经利用了这一功能，但你并不需要将其明确添加到命令中。`go`工具知道，如果没有使用`@` 提供特定的版本，就应该使用特殊的`latest` 版本。实际上，`最新`版本并不在版本库中，比如`my-tag`或`my-branch`。它是`go`工具内置的一个辅助工具，因此你无需自己搜索最新版本。

例如，您最初添加依赖关系时，也可以使用以下命令获得相同的结果：

```bash
go get github.com/spf13/cobra@latest
```

现在，假设有一个您正在使用的模块正在开发中。在这个例子中，将其称为`your_domain/sammy/awesome`。这个`awesome`模块正在添加一项新功能，而这项工作正在一个名为 `new-feature`.要将该分支添加为自己模块的依赖关系`，`需要提供模块路径、`@`符号和分支名称：

```bash
go get your_domain/sammy/awesome@new-feature
```

运行此命令后，`go`会连接到`your_domain/sammy/awesome`代码库，下载当前最新提交的`new-feature`分支，并将该信息添加到`go.mod`文件中。

分支并不是使用`@`选项的唯一方式。该语法可用于标签，甚至是版本库的特定提交。例如，有时你正在使用的库的最新版本可能有一个损坏的提交。在这种情况下，引用破损提交之前的提交会很有用。

以模块的 Cobra 依赖关系为例，假设您需要引用提交的 `07445ea`的提交 07445`ea`，因为它有一些你需要的改动，而且由于某些原因你不能使用其他版本。在这种情况下，你可以在`@`符号后提供提交哈希值，就像提供分支或标记一样。在你的`mymodule`目录下运行`go get`命令，输入模块和版本，下载新版本：

```bash
go get github.com/spf13/cobra@07445ea
```

如果再次打开模块的`go.mod`文件，就会发现`go get`已经更新了`github.com/spf13/cobra`的`require`行，以引用您指定的提交：

```go title="projects/mymodule/go.mod"
module mymodule

go 1.16

require (
 github.com/inconshreveable/mousetrap v1.0.0 // indirect
 github.com/spf13/cobra v1.1.2-0.20210209210842-07445ea179fc // indirect
 github.com/spf13/pflag v1.0.5 // indirect
)
```

与标签或分支不同，提交是一个特定的时间点，因此 Go 在`require`指令中包含了额外的信息，以确保将来使用的是正确的版本。如果仔细查看版本，你会发现它确实包含了你提供的提交哈希值：`v1.1.2-0.20210209210842-07445ea179fc`。

Go 模块也使用这一功能来支持发布模块的不同版本。当 Go 模块发布新版本时，版本库中就会添加一个以版本号为标签的新标签。如果你想使用某个特定版本，可以查看版本库中的标签列表，找到你要找的版本。如果你已经知道了版本，可能就不需要在标签中搜索了，因为版本标签的命名是一致的。

以 Cobra 为例，假设您想使用 Cobra 1.1.1 版本。您可以查看 Cobra 代码库，发现其中有一个名为`v1.1.1` 的标签。要使用这个标记版本，可以在`go get`命令中使用`@`符号，就像使用非版本标记或分支一样。现在，更新模块以使用 Cobra 1.1.1，在运行`go get`命令时使用 `v1.1.1`作为版本：

```bash
go get github.com/spf13/cobra@v1.1.1
```

现在打开模块的`go.mod`文件，就会看到`go get`已经更新了`github.com/spf13/cobra`的`require`行，以引用您提供的版本：

```go title="projects/mymodule/go.mod"
module mymodule

go 1.16

require (
 github.com/inconshreveable/mousetrap v1.0.0 // indirect
 github.com/spf13/cobra v1.1.1 // indirect
 github.com/spf13/pflag v1.0.5 // indirect
)
```

最后，如果您正在使用某个特定版本的库，例如`07445ea`版本或更早的`v1.1.1`，但您决定开始使用最新版本，可以使用特殊的 `latest`版本。要将模块更新为最新版本的Cobra，请使用模块路径和 `latest`版本：

```bash
go get github.com/spf13/cobra@latest
```

完成此命令后，`go.mod`文件将更新为您引用特定版本 Cobra 之前的样子。根据您的 Go 版本和当前 Cobra 的最新版本，输出结果可能会略有不同，但您仍然可以看到`require`部分中的`github.com/spf13/cobra`行已更新为最新版本：

```go
module mymodule

go 1.16

require (
 github.com/inconshreveable/mousetrap v1.0.0 // indirect
 github.com/spf13/cobra v1.2.1 // indirect
 github.com/spf13/pflag v1.0.5 // indirect
)
```

`go get`命令是一个强大的工具，你可以用它来管理`go.mod`文件中的依赖关系，而无需手动编辑。正如本节所述，使用带有模块名的`@`字符可以为模块使用特定版本，从发布版本到特定版本库提交。它甚至可以用来返回到依赖项的`latest`版本。结合使用这些选项，可以确保程序未来的稳定性。

## 结论

在本教程中，您创建了一个带有子软件包的 Go 模块，并在模块中使用了该软件包。你还将另一个模块作为依赖关系添加到你的模块中，并探索了如何以各种方式引用模块版本。

有关 Go 模块的更多信息，Go 项目有[一系列博文](https://blog.golang.org/using-go-modules)介绍 Go 工具如何与模块交互并理解模块。Go 项目还在《[Go 模块参考](https://golang.org/ref/mod)》（Go Modules Reference）中提供了非常详细的 Go 模块技术参考。
