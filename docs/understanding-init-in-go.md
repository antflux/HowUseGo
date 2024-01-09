# 了解 Go 中的 init

## [导言]

在 Go 中，预定义的`init()`函数会设置一段代码，在软件包的其他部分之前运行。这段代码将在软件包[导入]后立即执行，当您需要应用程序在特定状态下初始化时，例如当您有特定的配置或资源集需要启动应用程序时，可以使用这段代码。在*导入副作用*时也会用到它，这是一种通过导入特定软件包来设置程序状态的技术。这通常用于将一个软件包与另一个软件包进行`注册`，以确保程序在执行任务时使用正确的代码。

虽然`init()`是一个有用的工具，但有时也会使代码难以阅读，因为难以找到的`init()`实例会极大地影响代码的运行顺序。因此，对于刚接触 Go 的开发人员来说，了解该函数的方方面面非常重要，这样他们在编写代码时就能确保以清晰易读的方式使用`init()`。

在本教程中，您将了解`init()`如何用于设置和初始化特定软件包变量、一次性计算以及注册软件包以便与其他软件包一起使用。

## [先决条件]

对于本文中的一些示例，您需要

- 根据《[如何安装 Go 和设置本地编程环境]》建立的 Go 工作区。本教程将使用以下文件结构：

```bash
.
├── bin 
│ 
└── src
    └── github.com
        └── gopherguides
```

## [声明`init()`]

只要声明`init()`函数，Go 就会先于软件包中的其他内容加载并运行该函数。为了演示这一点，本节将介绍如何定义`init()`函数，并展示其对软件包运行的影响。

下面我们先以不使用`init()`函数的代码为例：

```go title="main.go"
package main

import "fmt"

var weekday string

func main() {
	fmt.Printf("Today is %s", weekday)
}
```

在这个程序中，我们声明了一个名为`weekday` 的[全局变量]。默认情况下，`weekday`的值为空字符串。

让我们运行这段代码：

```bash
go run main.go
```

由于`weekday`的值为空白，因此当我们运行程序时，会得到以下输出结果：


```
Output  Today is
```

我们可以通过引入`init()`函数来填补空白变量，该函数将`weekday`的值初始化为当前日期。在`main.go` 中添加以下突出显示的行：

```go title="main.go"
package main

import (
 "fmt"
 "time"
)

var weekday string

func init() {
 weekday = time.Now().Weekday().String()
}

func main() {
 fmt.Printf("Today is %s", weekday)
}
```

在这段代码中，我们导入并使用了时间包来获取当前星期`（Now().Weekday().String()）`，然后使用`init()` 将星期初始化为该值。

现在，当我们运行程序时，它会打印出当前的工作日：

```
输出今天是星期一
```

虽然这说明了`init()`如何工作，但`init()`更典型的用例是在导入软件包时使用它。当你需要在软件包中完成特定的设置任务后才能使用该软件包时，init() 就能派上用场。为了演示这一点，让我们创建一个程序，该程序需要特定的初始化才能使软件包正常工作。

## 在导入时初始化软件包

首先，我们将编写一些代码，从[片段]中随机选择一个生物并打印出来。不过，我们不会在初始程序中使用`init()`。这样可以更好地展示我们遇到的问题，以及`init()`如何解决我们的问题。

在`src/github.com/gopherguides/`目录中，使用以下命令创建名为`creature 的`文件夹：

```bash
mkdir creature
```

在`creature`文件夹中创建一个名为`creature.go` 的文件：

```bash
nano creature/creature.go
```

在该文件中，添加以下内容：

```go title="creature.go"
package creature

import (
	"math/rand"
)

var creatures = []string{"shark", "jellyfish", "squid", "octopus", "dolphin"}

func Random() string {
	i := rand.Intn(len(creatures))
	return creatures[i]
}
```

该文件定义了一个名为`creatures`的变量，其中有一组被初始化为值的海洋生物。文件还[导出]了一个`Random`函数，该函数将从`creatures`变量中返回一个随机值。

保存并退出该文件。

接下来，让我们创建一个`cmd`包，用来编写`main()`函数和调用`creature`包。

在创建`creature`文件夹的同一文件层，使用以下命令创建`cmd`文件夹：

```bash
mkdir cmd
```

在 `cmd` 文件夹中, 创建一个名为 `main.go`的文件:

```bash
nano cmd/main.go
```

在文件中添加如下内容:

```go title="cmd/main.go"
package main

import (
 "fmt"

 "github.com/gopherguides/creature"
)

func main() {
 fmt.Println(creature.Random())
 fmt.Println(creature.Random())
 fmt.Println(creature.Random())
 fmt.Println(creature.Random())
}
```

在这里，我们导入了`creature`软件包，然后在`main()`函数中使用`creature` `.Random()`函数获取一个随机生物，并打印出来四次。

保存并退出`main.go`。

现在我们已经写好了整个程序。不过，在运行这个程序之前，我们还需要创建几个配置文件，以便代码正常运行。Go 使用[Go 模块](https://blog.golang.org/using-go-modules)来配置导入资源的软件包依赖关系。这些模块是放置在软件包目录中的配置文件，它们告诉编译器从哪里导入软件包。虽然学习模块超出了本文的讨论范围，但我们只需编写几行配置文件，就能让本示例在本地正常运行。

在`cmd`目录中创建一个名为`go.mod` 的文件：

```bash
nano cmd/go.mod
```

打开文件后，输入以下内容：


```bash title="cmd/go.mod"
module github.com/gopherguides/cmd
 replace github.com/gopherguides/creature => ../creature
```

该文件的第一行告诉编译器，我们创建的`cmd`包实际上是`github.com/gopherguides/cmd`。第二行告诉编译器，`github.com` `/` `gopherguides/creature`可以在本地磁盘的`./creature`目录中找到。

保存并关闭文件。接下来，在`生物`目录下创建一个`go.mod`文件：

```bash
nano creature/go.mod
```

在文件中添加以下代码行：

creature/go.mod

```
  module github.com/gopherguides/creature
```

这会告诉编译器，我们创建的`creature`实际上是`github.com/gopherguides/creature`包。如果不这样做，`cmd`软件包就不知道从哪里导入这个软件包。

保存并退出文件。

现在您应该有以下目录结构和文件布局：

```bash
├── cmd
│   ├── go.mod
│   └── main.go
└── creature
    ├── go.mod
    └── creature.go
```

完成所有配置后，我们就可以使用以下命令运行`主程序`了：

```bash
go run cmd/main.go
```

这将提供

```
Output
jellyfish
squid
squid
dolphin
```

当我们运行这个程序时，我们收到了四个值并打印出来。如果多次运行这个程序，我们会发现**总是**得到相同的输出，而不是预期的随机结果。 这是因为`rand`软件包创建的伪随机数会在单个初始状态下持续生成相同的输出结果。为了获得更随机的数字，我们可以给软件包添加*种子*，或者设置一个不断变化的源，这样每次运行程序时的初始状态都会不同。在 Go 中，通常使用当前时间作为`rand`软件包的种子。

Since we want the `creature` package to handle the random functionality, open up this file:

```bash
nano creature/creature.go
```

在`creature.go`文件中添加以下突出显示的行：

```go title="creature/creature.go"
package creature

import (
 "math/rand"
 "time"
)

var creatures = []string{"shark", "jellyfish", "squid", "octopus", "dolphin"}

func Random() string {
 rand.Seed(time.Now().UnixNano())
 i := rand.Intn(len(creatures))
 return creatures[i]
}
```

在这段代码中，我们导入了`时间`软件包，并使用`Seed()`为当前时间播种。保存并退出文件。

现在，当我们运行程序时，将得到一个随机结果：

```bash
go run cmd/main.go
```

```
Output  jellyfish
octopus
shark
jellyfish
```

如果不断重复运行该程序，就会不断得到随机结果。不过，这还不是我们代码的理想实现，因为每次调用`creature.Random` `() 时`，都会再次调用`rand` `.Seed(time.Now().UnixNano()) 对 rand`包进行重新播种。如果内部时钟没有改变，重新播种将增加以相同初始值播种的几率，这将导致随机模式可能重复，或者让程序等待时钟改变而增加 CPU 处理时间。

要解决这个问题，我们可以使用`init()`函数。让我们更新`creature.go`文件：

```bash
nano creature/creature.go
```

添加以下代码行：

```go title="creature/creature.go"
package creature

import (
	"math/rand"
	"time"
)

var creatures = []string{"shark", "jellyfish", "squid", "octopus", "dolphin"}

func init() {
	rand.Seed(time.Now().UnixNano())
}

func Random() string {
	i := rand.Intn(len(creatures))
	return creatures[i]
}
```

添加`init()`函数会告诉编译器，在导入`creature`包时，它应该运行一次`init()`函数，为随机数生成提供一个种子。这样可以确保我们不会在必要的情况下运行更多代码。现在，如果我们运行程序，我们将继续得到随机结果：

```bash
go run cmd/main.go
```

```
Output
dolphin
squid
dolphin
octopus
```

在本节中，我们已经了解了使用`init()`如何确保在软件包使用之前执行适当的计算或初始化。接下来，我们将了解如何在软件包中使用多个`init()`语句。

## [`init()`的多个实例]

与只能声明一次的`main()`函数不同，`init`() 函数可以在整个软件包中声明多次。但是，多个`init()`函数会让人难以确定哪个函数优先于其他函数。在本节中，我们将展示如何保持对多个`init()`语句的控制。

在大多数情况下，`init()`函数会按照您遇到的顺序执行。让我们以下面的代码为例：

```go title="main.go"
package main

import "fmt"

func init() {
 fmt.Println("First init")
}

func init() {
 fmt.Println("Second init")
}

func init() {
 fmt.Println("Third init")
}

func init() {
 fmt.Println("Fourth init")
}

func main() {}
```

如果我们使用以下命令运行程序

```bash
go run main.go
```

我们将收到以下输出：

```
Output  First init
Second init
Third init
Fourth init
```
请注意，每个`init()`都是按照编译器遇到它的顺序运行的。不过，要确定`init()`函数的调用顺序可能并不总是那么容易。

让我们来看看一个更复杂的软件包结构，其中有多个文件，每个文件都声明了自己的`init()`函数。为了说明这一点，我们将创建一个程序，共享一个名为`message`的变量并将其打印出来。

删除前一节中的`creature`和`cmd`目录及其内容，代之以以下目录和文件结构：

```
├── cmd
│   ├── a.go
│   ├── b.go
│   └── main.go
└── message
    └── message.go
```

现在让我们添加每个文件的内容。在`a.go` 中添加以下几行：

```go title="cmd/a.go"
package main

import (
	"fmt"

	"github.com/gopherguides/message"
)

func init() {
	fmt.Println("a ->", message.Message)
}
```

该文件包含一个`init()`函数，用于从`消息`包中打印出`message` `.Message`的值。

接下来，在`b.go` 中添加以下内容：

```go title="cmd/b.go"
package main

import (
	"fmt"

	"github.com/gopherguides/message"
)

func init() {
	message.Message = "Hello"
	fmt.Println("b ->", message.Message)
}
```

在`b.go` 中，我们只有一个`init()`函数，它将`message.Message`的值设置为`Hello`并打印出来。

接下来，创建`main.go`，如下所示：

```go title="cmd/main.go"
package main

func main() {}
```

这个文件什么也不做，只是为程序的运行提供一个入口。

最后，像下面这样创建`message.go`文件：

```go title="message/message.go"
package message

var Message string
```

我们的`信息`包声明了导出的`Message`变量。

要运行程序，请在`cmd`目录中执行以下命令：

```bash
go run *.go
```

由于`cmd`文件夹中有多个组成`主`软件包的 Go 文件，因此我们需要告诉编译器`cmd`文件夹中的所有`.go`文件都应被编译。使用`*.go`命令会告诉编译器加载`cmd`文件夹中所有以`.go` 结尾的文件。如果我们发出`go run main.`go 命令，程序将无法编译，因为它看不到`a.go`和`b.go`文件中的代码。

输出结果如下

```
Output  
a ->
b -> Hello
```

根据 Go 语言的[包初始化](https://golang.org/ref/spec#Package_initialization)规范，当在一个包中遇到多个文件时，会按字母顺序进行处理。因此，我们第一次从`a.go` 中打印出`message.Message`时，其值是空白的。直到运行了`b.`go 中的`init()`函数，该值才被初始化。

如果我们把`a.go`的文件名改为`c.` `go`，就会得到不同的结果：

```go
Output  
b -> Hello
a -> Hello
```

现在，编译器首先遇到`b.go`，因此当遇到`c.`go 中的`init()`函数时，`message.Message`的值已经被初始化为`Hello`。

这种行为可能会给代码带来问题。在软件开发过程中，更改文件名是很常见的，而由于`init()`的处理方式，更改文件名可能会改变`init()`的处理顺序。这可能会改变程序的输出结果，从而造成不必要的后果。为确保初始化行为的可重现性，我们鼓励构建系统按照词法文件名顺序向编译器提交属于同一软件包的多个文件。 确保按顺序加载所有`init()`函数的一种方法是在一个文件中声明所有函数。 这样，即使更改了文件名，也不会改变顺序。

除了确保`init()`函数的顺序不变外，还应该尽量避免使用*全局变量*（即可以从程序包中任何地方访问的变量）来管理程序包中的状态。在前面的程序中，`message.Message`变量对整个程序包都可用，并保持着程序的状态。由于这种访问权限，`init()`语句可以更改变量并破坏程序的可预测性。为了避免这种情况，请尽量在受控空间中使用变量，在允许程序运行的情况下，尽量减少变量的访问权限。

我们已经看到，在一个程序包中可以有多个`init()`声明。但是，这样做可能会产生不想要的效果，使程序难以阅读或预测。避免使用多个`init()`语句或将它们全部放在一个文件中，可以确保在文件移动或名称更改时，程序的行为不会发生变化。

接下来，我们将研究`init()`如何用于导入副作用。

## [使用`init()`实现副作用]

在 Go 中，有时导入一个包并不是为了它的内容，而是为了导入包后产生的副作用。这通常意味着在导入的代码中，有一条`init()`语句会在其他代码之前执行，从而允许开发者操纵程序的启动状态。这种技术被称为*导入副作用*。

导入副作用的一个常见用例是在代码中*注册*功能，让软件包知道程序需要使用哪部分代码。例如，在[`图像`软件包](https://golang.org/pkg/image/)中，`image.Decode`函数在执行之前需要知道要解码的图像格式`（jpg`、`png`、`gif` 等）。为此，您可以首先导入一个具有`init()`语句副作用的特定程序。

比方说，您正试图在一个`.png`文件上使用`image` `.` `Decode`，代码片段如下：

```go title="解码片段示例"
. . .
func decode(reader io.Reader) image.Rectangle {
	m, _, err := image.Decode(reader)
	if err != nil {
		log.Fatal(err)
	}
	return m.Bounds()
}
. . .
```

使用此代码的程序仍然可以编译，但每次我们尝试解码`png`图像时，都会出现错误。

要解决这个问题，我们首先需要为`image.Decode` 注册一种图像格式。幸运的是，`image/png`软件包包含以下`init()`语句：

```go title="image/png/reader.go"
func init() {
	image.RegisterFormat("png", pngHeader, Decode, DecodeConfig)
}
```

因此，如果我们在解码片段中导入`image/png`，那么`image/png`中的`image.RegisterFormat()`函数将在我们的代码之前运行：

解码片段示例

```
. . .
import _ "image/png"
. . .

func decode(reader io.Reader) image.Rectangle {
	m, _, err := image.Decode(reader)
	if err != nil {
		log.Fatal(err)
	}
	return m.Bounds()
}
```

这将设置状态并注册我们需要的`png`版本的`image.Decode()`。此注册将作为导入`image/png` 的副作用发生。

您可能已经注意到`"image/png "`前的[空白标识符](https://golang.org/ref/spec#Blank_identifier)`(_`)。之所以需要这样做，是因为 Go 不允许导入程序中不使用的包。通过加入空白标识符，导入本身的值将被丢弃，因此只有导入的副作用才会出现。这意味着，即使我们从未在代码中调用过`image/png`包，我们仍然可以导入它以获得副作用。

了解何时需要导入软件包以获得其副作用是很重要的。如果没有适当的注册，程序很可能会编译成功，但运行时却无法正常工作。标准库中的软件包会在其文档中声明需要这种类型的导入。如果你编写的软件包需要导入副作用，你也应该确保你使用的`init()`语句是有文档说明的，这样导入你的软件包的用户才能正确使用它。

## [结论]

在本教程中，我们了解到`init()`函数在加载软件包中的其他代码之前加载，它可以执行软件包的特定任务，如初始化所需的状态。我们还了解到，编译器执行多条`init`() 语句的顺序取决于编译器加载源文件的顺序。如果您想了解有关`init()` 的更多信息，请查看[Golang](https://golang.org/doc/effective_go.html#init) 官方文档，或阅读[Go 社区中有关该函数的讨论](https://github.com/golang/go/issues/25885)。

有关函数的更多信息，请参阅我们的《[如何在 Go 中定义和调用函数]》一文，或浏览[整个《如何在 Go 中编码》系列]。
