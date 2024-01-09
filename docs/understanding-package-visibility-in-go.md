# 了解 Go 中的包可见性

### [介绍](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go#introduction)

当[在Go中创建一个包](https://www.digitalocean.com/community/tutorials/how-to-write-packages-in-go)时，最终目标通常是让其他开发人员可以访问该包，无论是在高阶包还是整个程序中。通过[导入包](https://www.digitalocean.com/community/tutorials/importing-packages-in-go)，您的代码片段可以作为其他更复杂工具的构建块。但是，只有某些软件包可用于导入。这是由包的可见性决定的。

在此上下文中，可见性意味着可以从中引用包或其他构造的文件空间。例如，如果我们在函数中定义一个变量，该变量的可见性（作用域）仅在定义它的函数中。类似地，如果在包中定义变量，则可以使其仅对该包可见，或者允许其在包外部可见。

在编写符合人体工程学的代码时，仔细控制包的可见性非常重要，特别是在考虑将来可能要对包进行的更改时。如果你需要修复一个bug，提高性能，或者改变功能，你会希望以一种不会破坏任何使用你的包的人的代码的方式进行修改。最小化破坏性更改的一种方法是只允许访问包中正确使用所需的部分。通过限制访问，您可以在包内部进行更改，而不会影响其他开发人员使用您的包的方式。

在本文中，您将学习如何控制包的可见性，以及如何保护只应在包中使用的代码部分。为此，我们将创建一个基本的日志记录器来记录和调试消息，使用具有不同项目可见度的包。

## [先决条件](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go#prerequisites)

要遵循本文中的示例，您需要：

- 一个Go工作区，通过下面的[步骤来设置如何安装Go和设置本地编程环境](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-go)。本教程将使用以下文件结构：

```
.
├── bin 
│ 
└── src
    └── github.com
        └── gopherguides
```

## [出口和未出口的物品](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go#exported-and-unexported-items)

与其他编程语言（如Java和[Python](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-python-3)）使用访问修饰符（如`public`、`private`或`protected`）来指定作用域不同，Go语言通过如何声明来确定一个项是否为`exported`和`unexported`。在这种情况下，导出一个项目会使它在当前包之外。如果它没有被导出，那么它只能在它被定义的包中可见和可用。

这种外部可见性通过将声明项的第一个字母大写来控制。所有声明，如`Types`、`Variables`、`Constants`、`Functions`等，以大写字母开头的文件在当前包外可见。

让我们来看看下面的代码，注意大小写：



```go title="greet.go"
package greet

import "fmt"

var Greeting string

func Hello(name string) string {
	return fmt.Sprintf(Greeting, name)
}
```

这段代码声明它在`greet`包中。然后它声明了两个符号，一个名为`Greeting`的变量和一个名为`Hello`的函数。因为它们都以大写字母开头，所以它们都是`exported`，并且可用于任何外部程序。如前所述，创建一个限制访问的包将允许更好的API设计，并使内部更新包更容易，而不会破坏依赖于您的包的任何人的代码。

## [定义包可见性](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go#defining-package-visibility)

为了给予更近距离的观察，让我们创建一个`logging`包，记住我们想要在我们的包之外显示什么，我们不想显示什么。这个日志记录包将负责将我们的任何程序消息记录到控制台。它还将查看我们正在记录的级别。级别描述日志的类型，并且将是三种状态之一：`info`、`warning`或`error`。

首先，在您的`src`目录中，让我们创建一个名为`logging`的目录来放置日志文件：

```bash
mkdir logging
```


接下来进入该目录：

```bash
cd logging
```


然后，使用类似nano的编辑器，创建一个名为`logging.go`的文件：

```bash
nano logging.go
```


将下面的代码放在我们刚刚创建的`logging.go`文件中：



```go title="logging/logging.go"
package logging

import (
	"fmt"
	"time"
)

var debug bool

func Debug(b bool) {
	debug = b
}

func Log(statement string) {
	if !debug {
		return
	}

	fmt.Printf("%s %s\n", time.Now().Format(time.RFC3339), statement)
}
```


这段代码的第一行声明了一个名为`logging`的包。在这个包中，有两个`exported`函数：`Debug`和`Log`。这些函数可以被导入`logging`包的任何其他包调用。还有一个私有变量，叫做`debug`。此变量只能从`logging`包中访问。值得注意的是，虽然函数`Debug`和变量`debug`都有相同的拼写，但函数是大写的，而变量不是。这使得它们具有不同的作用域。

保存并退出文件。

要在我们代码的其他区域使用这个包，我们可以[将其`import`到一个新包中](https://www.digitalocean.com/community/tutorials/importing-packages-in-go)。我们将创建这个新的包，但是我们首先需要一个新的目录来存储这些源文件。

让我们离开`logging`目录，创建一个名为`cmd`的新目录，然后移动到新目录中：

```bash
cd ..
mkdir cmd
cd cmd
```


在我们刚刚创建的`main.go`目录中创建一个名为`cmd`的文件：

```bash
nano main.go
```


现在我们可以添加以下代码：



```go title="cmd/main.go"
package main

import "github.com/gopherguides/logging"

func main() {
	logging.Debug(true)

	logging.Log("This is a debug statement...")
}
```


我们现在已经写好了整个程序。然而，在我们运行这个程序之前，我们还需要为我们的代码创建几个配置文件以使其正常工作。Go使用[Go Modules](https://blog.golang.org/using-go-modules)来配置包依赖以导入资源。Go模块是放在包目录中的配置文件，它告诉编译器从哪里导入包。虽然了解模块超出了本文的范围，但我们可以编写几行配置来使这个示例在本地工作。

在`go.mod`目录中打开以下`cmd`文件：

```bash
nano go.mod
```


然后将以下内容放入文件中：

go.mod

```
module github.com/gopherguides/cmd

replace github.com/gopherguides/logging => ../logging
```

这个文件的第一行告诉编译器`cmd`包的文件路径是`github.com/gopherguides/cmd`。第二行告诉编译器可以在本地磁盘上的`github.com/gopherguides/logging`目录中找到包`../logging`。

我们还需要为我们的`go.mod`包创建一个`logging`文件。让我们回到`logging`目录并创建一个`go.mod`文件：

```bash
cd ../logging
nano go.mod
```


将以下内容添加到文件中：

go.mod

```
module github.com/gopherguides/logging
```

这告诉编译器我们创建的`logging`包实际上是`github.com/gopherguides/logging`包。这使得我们可以使用我们之前写的下面一行导入我们的`main`包中的包：


```go title="cmd/main.go"
package main

import "github.com/gopherguides/logging"

func main() {
	logging.Debug(true)

	logging.Log("This is a debug statement...")
}
```

您现在应该拥有以下目录结构和文件布局：

```
├── cmd
│   ├── go.mod
│   └── main.go
└── logging
    ├── go.mod
    └── logging.go
```

现在我们已经完成了所有的配置，我们可以使用以下命令从`main`包运行`cmd`程序：

```bash
cd ../cmd
go run main.go
```


您将得到类似于以下内容的输出：

```
Output
2019-08-28T11:36:09-05:00 This is a debug statement...
```

程序将以RFC 3339格式打印出当前时间，然后是我们发送给记录器的任何语句。[RFC 3339](https://tools.ietf.org/html/rfc3339)是一种时间格式，旨在表示Internet上的时间，通常用于日志文件。

因为`Debug`和`Log`函数是从logging包导出的，所以我们可以在`main`包中使用它们。但是，`debug`包中的`logging`变量不会导出。尝试引用未导出的声明将导致编译时错误。

将以下突出显示的行添加到`main.go`：


```go title="cmd/main.go"
package main

import "github.com/gopherguides/logging"

func main() {
	logging.Debug(true)

	logging.Log("This is a debug statement...")

	fmt.Println(logging.debug)
}
```

保存并运行文件。您将收到类似以下内容的错误：

```
Output. . .
./main.go:10:14: cannot refer to unexported name logging.debug
```

现在我们已经了解了包中`exported`和`unexported`项的行为，接下来我们将看看如何从`fields`导出`methods`和`structs`。

## 结构内的可见性

虽然我们在上一节中构建的日志记录器中的可见性方案可能适用于简单的程序，但它在多个包中共享太多的状态而无法使用。这是因为导出的变量可以被多个包访问，这些包可能会将变量修改为矛盾的状态。允许以这种方式更改包的状态会使您很难预测程序的行为。例如，在当前的设计中，一个包可以将`Debug`变量设置为`true`，而另一个包可以在同一实例中将其设置为`false`。这会产生一个问题，因为导入`logging`包的两个包都会受到影响。

我们可以通过创建一个结构体，然后将方法挂在它上面来隔离日志记录器。这将允许我们创建一个日志记录器的`instance`，以便在每个使用它的包中独立使用。

将`logging`包更改为以下内容，以重构代码并隔离记录器：


```go title="logging/logging.go"
package logging

import (
	"fmt"
	"time"
)

type Logger struct {
	timeFormat string
	debug      bool
}

func New(timeFormat string, debug bool) *Logger {
	return &Logger{
		timeFormat: timeFormat,
		debug:      debug,
	}
}

func (l *Logger) Log(s string) {
	if !l.debug {
		return
	}
	fmt.Printf("%s %s\n", time.Now().Format(l.timeFormat), s)
}
```


在这段代码中，我们创建了一个`Logger`结构体。这个结构体将存放我们未导出的状态，包括要打印的时间格式和`debug`变量设置`true`或`false`。`New`函数设置初始状态以创建记录器，例如时间格式和调试状态。然后，它将我们在内部给它的值存储到未导出的变量`timeFormat`和`debug`中。我们还在`Log`类型上创建了一个名为`Logger`的方法，它接受我们想要打印出来的语句。在`Log`方法中，有一个对它的局部方法变量`l`的引用，以访问它的内部字段，如`l.timeFormat`和`l.debug`。

这种方法将允许我们在许多不同的包中创建`Logger`，并独立于其他包如何使用它来使用它。

要在另一个包中使用它，让我们将`cmd/main.go`修改为如下所示：



```go title="cmd/main.go"
package main

import (
	"time"

	"github.com/gopherguides/logging"
)

func main() {
	logger := logging.New(time.RFC3339, true)

	logger.Log("This is a debug statement...")
}
```


运行此程序将给予以下输出：

```
Output  2019-08-28T11:56:49-05:00 This is a debug statement...
```

在这段代码中，我们通过调用导出的函数`New`创建了一个logger实例。我们将对该实例的引用存储在`logger`变量中。我们现在可以调用`logging.Log`来打印语句。

如果我们试图从`Logger`引用一个未导出的字段，比如`timeFormat`字段，我们将收到一个编译时错误。尝试添加以下突出显示的行并运行`cmd/main.go`：



```go title="cmd/main.go"
package main

import (
	"time"

	"github.com/gopherguides/logging"
)

func main() {
	logger := logging.New(time.RFC3339, true)

	logger.Log("This is a debug statement...")

	fmt.Println(logger.timeFormat)
}
```

这将导致给予以下错误：

```
Output. . .
cmd/main.go:14:20: logger.timeFormat undefined (cannot refer to unexported field or method timeFormat)
```

编译器识别出`logger.timeFormat`没有被导出，因此不能从`logging`包中检索。

## [方法中的可见性](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go#visibility-within-methods)

与结构字段相同，方法也可以导出或取消导出。

为了说明这一点，让我们将分级日志记录添加到我们的日志记录器。分级日志记录是一种对日志进行分类的方法，这样您就可以在日志中搜索特定类型的事件。我们将放入记录器的级别是：

- `info`级别，表示通知用户操作的信息类型事件，如`Program started`或`Email sent`。这些帮助我们调试和跟踪程序的某些部分，以查看是否发生了预期的行为。
- `warning`水平。 这些类型的事件标识何时发生了非错误的意外事件，如`Email failed to send, retrying`。 他们帮助我们看到我们的计划的一部分，没有像我们期望的那样顺利。
- `error`级别，这意味着程序遇到了问题，如`File not found`。这往往会导致程序运行失败。

您可能还希望打开和关闭某些级别的日志记录，特别是如果程序没有按预期执行，并且您希望调试程序。我们将通过更改程序来添加此功能，以便当`debug`设置为`true`时，它将打印所有级别的消息。否则，如果它是`false`，它将只打印错误消息。

通过对`logging/logging.go`进行以下更改来添加分级日志记录：



```go title="logging/logging.go"
package logging

import (
	"fmt"
	"strings"
	"time"
)

type Logger struct {
	timeFormat string
	debug      bool
}

func New(timeFormat string, debug bool) *Logger {
	return &Logger{
		timeFormat: timeFormat,
		debug:      debug,
	}
}

func (l *Logger) Log(level string, s string) {
	level = strings.ToLower(level)
	switch level {
	case "info", "warning":
		if l.debug {
			l.write(level, s)
		}
	default:
		l.write(level, s)
	}
}

func (l *Logger) write(level string, s string) {
	fmt.Printf("[%s] %s %s\n", level, time.Now().Format(l.timeFormat), s)
}
```


在这个例子中，我们为`Log`方法引入了一个新参数。我们现在可以传入日志消息的`level`。`Log`方法确定它是什么级别的消息。如果它是一个`info`或`warning`消息，并且`debug`字段是`true`，则它写入该消息。否则，它将忽略该消息。如果它是任何其他级别，如`error`，它将写出消息。

用于确定是否打印出消息的大部分逻辑存在于`Log`方法中。我们还引入了一个名为`write`的未导出方法。`write`方法是实际输出日志消息的方法。

我们现在可以在另一个包中使用这种分级日志记录，方法是将`cmd/main.go`修改为如下所示：



```go title="cmd/main.go"
package main

import (
	"time"

	"github.com/gopherguides/logging"
)

func main() {
	logger := logging.New(time.RFC3339, true)

	logger.Log("info", "starting up service")
	logger.Log("warning", "no tasks found")
	logger.Log("error", "exiting: no work performed")

}
```


运行此命令将为您提供给予：

```
Output[info] 2019-09-23T20:53:38Z starting up service
[warning] 2019-09-23T20:53:38Z no tasks found
[error] 2019-09-23T20:53:38Z exiting: no work performed
```

在本例中，`cmd/main.go`成功使用了导出的`Log`方法。

我们现在可以通过将`level`切换到`debug`来传递每个消息的`false`：



```go title="main.go"
package main

import (
	"time"

	"github.com/gopherguides/logging"
)

func main() {
	logger := logging.New(time.RFC3339, false)

	logger.Log("info", "starting up service")
	logger.Log("warning", "no tasks found")
	logger.Log("error", "exiting: no work performed")

}
```


现在我们将看到只有`error`级别的消息打印：

```
Output[error] 2019-08-28T13:58:52-05:00 exiting: no work performed
```

如果我们尝试从`write`包外部调用`logging`方法，我们将收到一个编译时错误：



```go title="main.go"
package main

import (
	"time"

	"github.com/gopherguides/logging"
)

func main() {
	logger := logging.New(time.RFC3339, true)

	logger.Log("info", "starting up service")
	logger.Log("warning", "no tasks found")
	logger.Log("error", "exiting: no work performed")

	logger.write("error", "log this message...")
}
Outputcmd/main.go:16:8: logger.write undefined (cannot refer to unexported field or method logging.(*Logger).write)
```

当编译器看到你试图引用另一个以小写字母开头的包时，它知道它没有被导出，因此抛出一个编译器错误。

本教程中的记录器说明了我们如何编写代码，只公开我们希望其他包使用的部分。因为我们控制了包的哪些部分在包外可见，所以我们现在能够在不影响依赖于我们包的任何代码的情况下进行未来的更改。 例如，如果我们只想在`info`为false时关闭`debug`级别的消息，您可以在不影响API任何其他部分的情况下进行此更改。我们还可以安全地对日志消息进行更改，以包含更多信息，例如程序运行的目录。

## [结论]

本文展示了如何在包之间共享代码，同时保护包的实现细节。这允许您导出一个简单的API，该API很少会因为向后兼容性而更改，但允许根据需要在包中进行私下更改，以使其在未来更好地工作。这被认为是创建包及其相应API时的最佳实践。