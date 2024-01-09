# 使用构建标签定制 Go 二进制文件

## 介绍

在Go语言中，构建标签（build tag）或构建约束（build constraint）是添加到一段代码中的标识符，用于确定在`build`进程中文件何时应该包含在包中。这允许您从相同的源代码构建不同版本的Go应用程序，并以快速和有组织的方式在它们之间切换。许多开发人员使用构建标记来改进构建跨平台兼容应用程序的工作流程，例如需要更改代码以解决不同操作系统之间差异的程序。构建标记也用于[集成测试](https://en.wikipedia.org/wiki/Integration_testing)，允许您在集成代码和带有[模拟服务或存根](https://en.wikipedia.org/wiki/Mock_object)的代码之间快速切换，并用于应用程序中不同级别的功能集。

让我们以不同客户功能集的问题为例。在编写某些应用程序时，您可能希望控制在二进制文件中包含哪些功能，例如提供免费、专业和企业级别的应用程序。随着客户在这些应用程序中增加其订阅级别，更多功能将解锁并可用。要解决这个问题，您可以维护单独的项目，并尝试通过使用`import`语句来保持它们彼此同步。虽然这种方法是可行的，但随着时间的推移，它会变得乏味和容易出错。另一种方法是使用构建标记。

在本文中，您将在Go中使用构建标记来生成不同的可执行二进制文件，这些可执行二进制文件提供示例应用程序的Free、Pro和Enterprise特性集。每个版本都有一套不同的功能，免费版本是默认的。

## 先决条件

要遵循本文中的示例，您需要：

- 一个Go工作区，通过下面的[步骤来设置如何安装Go和设置本地编程环境]。

## 构建免费版本

让我们从构建应用程序的免费版本开始，因为它将是在没有任何构建标记的情况下运行`go build`时的默认版本。稍后，我们将使用构建标记有选择地向程序中添加其他部分。

在`src`目录中，创建一个以应用程序名称命名的文件夹。本教程将使用`app`：

```bash
mkdir app
```

移动到此文件夹：

```bash
cd app
```

接下来，在您选择的文本编辑器中创建一个名为`main.go`的新文本文件：

```bash
nano main.go
```

现在，我们将定义应用程序的免费版本。在`main.go`中添加以下内容：

main.go

```go
package main

import "fmt"

var features = []string{
  "Free Feature #1",
  "Free Feature #2",
}

func main() {
  for _, f := range features {
    fmt.Println(">", f)
  }
}
```

在这个文件中，我们创建了一个程序，它声明了一个名为`features`的切片，其中包含两个字符串，表示我们的Free应用程序的特性。应用程序中的`main()`函数通过`for`切片使用`range`循环到`features`，并打印屏幕上可用的所有功能。

保存并退出文件。现在这个文件已经保存了，我们将不再需要在本文的其余部分编辑它。相反，我们将使用构建标记来更改我们将从它构建的二进制文件的特性。

构建并运行程序：

```bash
go build
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
```

该程序已经打印出我们的两个免费功能，完成了我们的应用程序的免费版本。

到目前为止，您创建了一个具有非常基本的特性集的应用程序。接下来，您将构建一种在构建时向应用程序中添加更多功能的方法。

## 使用`go build`添加专业功能

到目前为止，我们一直避免对`main.go`进行更改，模拟一个通用的生产环境，需要在其中添加代码而不更改或可能破坏主代码。由于我们不能编辑`main.go`文件，我们需要使用另一种机制来使用构建标记将更多特性注入`features`切片。

让

```bash
nano pro.go
```

编辑器打开文件后，添加以下行：

pro.go

```go
package main

func init() {
  features = append(features,
    "Pro Feature #1",
    "Pro Feature #2",
  )
}
```

在此代码中，我们使用`init()`在应用程序的`main()`函数之前运行代码，然后使用`append()`将Pro功能添加到`features`切片。保存并退出文件。

使用`go build`编译并运行应用程序：

```bash
go build
```

由于当前目录中有两个文件（`pro.go`和`main.go`），`go build`将从这两个文件创建一个二进制文件。执行此二进制文件：

```bash
./app
```

这将为您提供给予以下功能集：

```
Output
> Free Feature #1
> Free Feature #2
> Pro Feature #1
> Pro Feature #2
```

该应用程序现在包括Pro和Free功能。然而，这是不可取的：由于版本之间没有区别，免费版本现在包括了应该只在专业版中可用的功能。要解决这个问题，您可以包含更多的代码来管理应用程序的不同层，或者您可以使用构建标记来告诉Go工具链构建哪些`.go`文件，忽略哪些文件。让我们在下一步中添加构建标记。

## 添加构建标记

您现在可以使用构建标记来区分应用程序的专业版和免费版。

让我们先来看看build标签是什么样子的：

```go
// +build tag_name
```

通过将这行代码作为包的第一行，并将`tag_name`替换为构建标记的名称，您将把这个包标记为可以选择性地包含在最终二进制文件中的代码。让我们通过向`pro.go`文件添加一个构建标记来告诉`go build`命令忽略它，除非指定了该标记。在文本编辑器中打开文件：

```bash
nano pro.go
```

然后添加以下突出显示的行：

pro.go

```
// +build pro

package main

func init() {
  features = append(features,
    "Pro Feature #1",
    "Pro Feature #2",
  )
}
```

在`pro.go`文件的顶部，我们添加了`// +build pro`，后面跟着一个空白的换行符。这个结尾的换行符是必需的，否则Go会将其解释为注释。构建标记声明也必须位于`.go`文件的最顶部。任何东西，甚至是注释，都不能在构建标记之上。

`+build`声明告诉`go build`命令这不是一个注释，而是一个构建标记。第二部分是`pro`标签。通过在`pro.go`文件的顶部添加此标记，`go build`命令现在将只包括带有`pro.go`标记的`pro`文件。

再次编译并运行应用程序：

```bash
go build
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
```

由于`pro.go`文件要求存在`pro`标记，因此该文件将被忽略，应用程序将在没有它的情况下进行编译。

当运行`go build`命令时，我们可以使用`-tags`标志，通过添加标记本身作为参数，有条件地将代码包含在编译的源代码中。让我们对`pro`标签这样做：

```bash
go build -tags pro
```

这将输出以下内容：

```
Output
> Free Feature #1
> Free Feature #2
> Pro Feature #1
> Pro Feature #2
```

现在，我们只有在使用`pro` build标记构建应用程序时才能获得额外的功能。

如果只有两个版本，这很好，但是当您添加更多标记时，事情就会变得复杂。为了在下一步中添加到我们的应用程序的企业版中，我们将使用多个构建标记通过布尔逻辑连接在一起。

## 生成标记布尔逻辑

当Go包中有多个构建标签时，标签之间使用[布尔逻辑]进行交互。为了演示这一点，我们将同时使用`pro`和`enterprise`标记来添加应用程序的企业级。

为了构建Enterprise二进制文件，我们需要包含默认特性、Pro级特性和一组新的Enterprise特性。首先，打开一个编辑器并创建一个新文件`enterprise.go`，该文件将添加新的Enterprise功能：

```bash
nano enterprise.go
```

`enterprise.go`的内容看起来与`pro.go`几乎相同，但将包含新功能。在文件中添加以下行：

enterprise.go

```go
package main

func init() {
  features = append(features,
    "Enterprise Feature #1",
    "Enterprise Feature #2",
  )
}
```

保存并退出文件。

目前`enterprise.go`文件没有任何构建标记，正如您在添加`pro.go`时所了解的那样，这意味着这些功能将在执行`go.build`时添加到免费版本中。对于`pro.go`，您在文件顶部添加了`// +build pro`和一个新行，以告诉`go build`只有在使用`-tags pro`时才应该包含它。在这种情况下，您只需要一个构建标记就可以完成目标。但是，在添加新的企业功能时，您首先还必须具有专业功能。

让我们首先将对`pro` build标记的支持添加到`enterprise.go`。使用文本编辑器打开文件：

```bash
nano enterprise.go
```

接下来，在`package main`声明之前添加build标记，并确保在build标记之后包含一个换行符：

enterprise.go

```
// +build pro

package main

func init() {
  features = append(features,
    "Enterprise Feature #1",
    "Enterprise Feature #2",
  )
}
```

保存并退出文件。

编译并运行不带任何标签的应用程序：

```bash
go build
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
```

企业版功能不再出现在免费版本中。现在让我们添加`pro` build标记，并再次构建和运行应用程序：

```bash
go build -tags pro
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
> Enterprise Feature #1
> Enterprise Feature #2
> Pro Feature #1
> Pro Feature #2
```

这仍然不是我们所需要的：当我们尝试构建Pro版本时，企业功能现在会显示出来。为了解决这个问题，我们需要使用另一个构建标记。然而，与`pro`标签不同，我们现在需要确保`pro`和`enterprise`功能都可用。

Go语言的构建系统通过允许在构建标记系统中使用一些基本的布尔逻辑来解决这种情况。

让我们再次打开`enterprise.go`：

```bash
nano enterprise.go
```

在与`enterprise`标记相同的行上添加另一个构建标记`pro`：

enterprise.go

```
// +build pro enterprise

package main

func init() {
  features = append(features,
    "Enterprise Feature #1",
    "Enterprise Feature #2",
  )
}
```

保存并关闭文件。

现在让我们使用新的`enterprise` build标记编译并运行应用程序。

```bash
go build -tags enterprise
./app
```

这将给予以下内容：

```
Output
> Free Feature #1
> Free Feature #2
> Enterprise Feature #1
> Enterprise Feature #2
```

现在我们已经失去了Pro功能。这是因为当我们将多个构建标记放在`.go`文件的同一行时，`go build`会将它们解释为使用`OR`逻辑。通过添加行`// +build pro enterprise`，如果存在`enterprise.go`构建标记或`pro`构建标记，则将构建`enterprise`文件。我们需要正确地设置构建标记，以要求两者，并使用`AND`逻辑。

如果我们将两个标记放在不同的行上，而不是将它们放在同一行上，那么`go build`将使用`AND`逻辑解释这些标记。

再次打开`enterprise.go`，让我们将构建标记分隔到多行中。

enterprise.go

```
// +build pro
// +build enterprise

package main

func init() {
  features = append(features,
    "Enterprise Feature #1",
    "Enterprise Feature #2",
  )
}
```

现在使用新的`enterprise` build标记编译并运行应用程序。

```bash
go build -tags enterprise
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
```

仍然不完全在那里：因为`AND`语句需要将两个元素都视为`true`，所以我们需要同时使用`pro`和`enterprise`构建标记。

让我们再试一次：

```bash
go build -tags "enterprise pro"
./app
```

您将收到以下输出：

```
Output
> Free Feature #1
> Free Feature #2
> Enterprise Feature #1
> Enterprise Feature #2
> Pro Feature #1
> Pro Feature #2
```

现在，我们的应用程序可以从同一个源代码树中以多种方式构建，从而相应地解锁应用程序的功能。

在这个例子中，我们使用了一个新的`// +build`标记来表示`AND`逻辑，但是也有其他的方法来用构建标记来表示布尔逻辑。下表包含构建标记的其他语法格式的一些示例，沿着它们的布尔等效项：

| 建立Tag Syntax | 建立Tag Sample             | 布尔语句            |
| :------------: | -------------------------- | ------------------- |
|  空间分离元素  | `// +build !pro`           | `pro`或`enterprise` |
|  逗号分隔元素  | `// +build pro,enterprise` | `pro`和`enterprise` |
|   感叹号元素   | `// +build !pro`           | 非`pro`             |

## 结论

在本教程中，您使用了构建标记来控制哪些代码被编译到二进制文件中。首先，您声明了构建标记并将它们与`go build`一起使用，然后您使用布尔逻辑组合了多个标记。然后，您构建了一个程序，它代表了Free、Pro和Enterprise版本的不同特性集，显示了构建标记可以给予您对项目的强大控制级别。