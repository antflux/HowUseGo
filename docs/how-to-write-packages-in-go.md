# 在 Go 中如何写软件包

软件包由位于同一目录下的 Go 文件组成，开头有相同的软件包声明。你可以从软件包中加入额外的功能，使你的程序更加复杂。有些软件包可以通过 Go 标准库获得，因此会在安装 Go 时一并安装。其他软件包可以通过 Go 的`go get`命令安装。您也可以通过使用必要的软件包语句，在要共享代码的同一目录下创建 Go 文件，从而构建自己的 Go 软件包。

本教程将指导您编写 Go 软件包，以便在其他编程文件中使用。

## [先决条件]

- 按照 "[如何安装和设置 Go 的本地编程环境]"系列教程中的教程设置 Go 编程环境。按照本地编程环境教程中的步骤 5 创建 Go 工作区。要遵循本文中的示例和命名规则，请阅读第一节 "编写和导入软件包"。
- 要加深对 GOPATH 的了解，请阅读我们的文章[了解 GOPATH]。

## [编写和导入软件包]

编写软件包就像编写其他 Go 文件一样。软件包可以包含函数、[类型]和[变量]的定义，这些定义可以在其他 Go 程序中使用。

在创建新软件包之前，我们需要进入 Go 工作区。这通常是在我们的`gopath` 下。在本教程的示例中，我们将把软件包称为`greet`。为此，我们在项目空间下的`gopath`中创建了一个名为`greet`的目录。如果我们的组织是`gopherguides`，我们想在组织下创建`greet`包，同时使用 Github 作为代码库，那么我们的目录会是这样的：

```
└── $GOPATH
    └── src
        └── github.com
            └── gopherguides
```

`问候`目录位于`gopherguides`目录内：

```
└── $GOPATH
    └── src
        └── github.com
            └── gopherguides
                └── greet
```

最后，我们可以添加目录中的第一个文件。通常的做法是，软件包中的`主要`文件或`入口`文件以目录名命名。在这种情况下，我们将在`greet`目录中创建一个名为`greet` `.go`的文件：

```
└── $GOPATH
    └── src
        └── github.com
            └── gopherguides
                └── greet
                    └── greet.go
```

文件创建完成后，我们就可以开始编写要在项目中重复使用或共享的代码了。在本例中，我们将创建一个名为`Hello`的函数，用于打印`Hello World`。

用文本编辑器打开`greet.go`文件，添加以下代码：

```go title="greet.go"
package greet

import "fmt"

func Hello() {
 fmt.Println("Hello, World!")
}
```

让我们来分解第一个文件。每个文件的第一行都需要输入所使用`软件包`的名称。由于您所在的是`greet`软件包，所以要使用`package`关键字，后面跟上软件包的名称：

```go
package greet
```

这会告诉编译器将文件中的所有内容都视为`greet`包的一部分。

接下来，通过`import`语句声明需要使用的其他软件包。本文件中只使用了一个软件包--`fmt`软件包：

```go
import "fmt"
```

最后，创建函数`Hello`。它将使用`fmt`软件包打印出`Hello, World!`

```go
func Hello() {
 fmt.Println("Hello, World!")
}
```

现在你已经编写了`greet`软件包，可以在创建的任何其他软件包中使用它。让我们创建一个新软件包，在其中使用你的`greet`软件包。

你将创建一个名为`example` 的软件包，这意味着你需要一个名为`example` 的目录。在你的`gopherguides`组织中创建这个软件包，目录结构如下：

```
└── $GOPATH
    └── src
        └── github.com
            └── gopherguides
                    └── example
```

有了新软件包的目录，就可以创建入口点文件了。因为这将是一个可执行程序，所以最好将入口点文件命名为`main.go`：

```
└── $GOPATH
    └── src
        └── github.com
            └── gopherguides
                └── example
                    └── main.go
```

在文本编辑器中打开`main.go`，添加以下代码以调用`greet`软件包：

main.go

```go title="main.go"
package main

import "github.com/gopherguides/greet"

func main() {
 greet.Hello()
}
```

由于您导入的是软件包，因此在调用函数时需要使用点符号引用软件包名称。*点号表示法*是在使用的软件包名称和要使用的软件包资源之间加一个`.`号。例如，在`greet`软件包中，`Hello`函数是一个资源。如果要调用该资源，可以使用`greet.Hello()` 的点号表示。

现在，你可以打开终端，在命令行上运行程序：

```bash
go run main.go
```

执行此操作后，您将收到以下输出结果：

```
Output
Hello, World!
```

为了了解如何在软件包中使用变量，让我们在`greet.go`文件中添加一个变量定义：

```go title="greet.go"
package greet

import "fmt"

var Shark = "Sammy"

func Hello() {
 fmt.Println("Hello, World!")
}
```

接下来，打开`main.go`文件，添加以下突出显示的一行，在`fmt.Println()`函数中调用`greet.go`中的变量：

```go title="main.go"
package main

import (
 "fmt"

 "github.com/gopherguides/greet"
)

func main() {
 greet.Hello()

 fmt.Println(greet.Shark)
}
```

再次运行程序后

```bash
go run main.go
```

您将收到以下输出结果：

```
Output
Hello, World!
Sammy
```

最后，我们还要在`greet.go`文件中定义一个类型。您将创建带有`名称`和`颜色`字段的`Octopus`类型，以及一个在调用时打印出字段的函数：

```go title="greet.go"
package greet

import "fmt"

var Shark = "Sammy"

type Octopus struct {
 Name  string
 Color string
}

func (o Octopus) String() string {
 return fmt.Sprintf("The octopus's name is %q and is the color %s.", o.Name, o.Color)
}

func Hello() {
 fmt.Println("Hello, World!")
}
```

打开`main.go`，在文件末尾创建该类型的实例：

```go title="main.go"
package main

import (
 "fmt"

 "github.com/gopherguides/greet"
)

func main() {
 greet.Hello()

 fmt.Println(greet.Shark)

 oct := greet.Octopus{
  Name:  "Jesse",
  Color: "orange",
 }

 fmt.Println(oct.String())
}
```

使用`oct := greet.` `Octopus`创建 Octopus 类型实例后，就可以在`main.go`文件的命名空间中访问该类型的函数和字段。这样，您就可以在最后一行写入`oct.String()`而无需调用`greet`。例如，您还可以调用`oct.Color`等类型字段，而无需引用`greet`软件包的名称。

`八达通`类型的`String`方法使用`fmt.Sprintf`函数创建一个句子，并将结果（一个字符串）`返回`给调用者（在本例中，就是你的主程序）。

运行程序后，您将收到以下输出：

```bash
go run main.go
```

```
Output
Hello, World!
Sammy
The octopus's name is "Jesse" and is the color orange.
```

通过在`Octopus` 上创建`String`方法，您现在有了一种可重复使用的方法来打印自定义类型的信息。如果将来要更改该方法的行为，只需编辑这一个方法即可。

## [导出代码](https://www.digitalocean.com/community/tutorials/how-to-write-packages-in-go#exported-code)

你可能已经注意到，你调用的`greet.go`文件中的所有声明都是大写的。Go 不像其他语言那样有`public`、`private` 或`protected`修饰符的概念。外部可见性由大写字母控制。以大写字母开头的类型、变量、函数等在当前包之外是公开可见的。在软件包外可见的符号被认为是`导出的`。

如果在`八达通`中添加名为`reset` 的新方法，可以在`greet`软件包中调用，但不能在`greet`软件包之外的`main.go`文件中调用：

```go title="greet.go"
package greet

import "fmt"

var Shark = "Sammy"

type Octopus struct {
 Name  string
 Color string
}

func (o Octopus) String() string {
 return fmt.Sprintf("The octopus's name is %q and is the color %s.", o.Name, o.Color)
}

func (o *Octopus) reset() {
 o.Name = ""
 o.Color = ""
}

func Hello() {
 fmt.Println("Hello, World!")
}
```

如果您尝试从`main.go`文件中调用`重置`：

main.go

```go title="main.go"
package main

import (
 "fmt"

 "github.com/gopherguides/greet"
)

func main() {
 greet.Hello()

 fmt.Println(greet.Shark)

 oct := greet.Octopus{
  Name:  "Jesse",
  Color: "orange",
 }

 fmt.Println(oct.String())

 oct.reset()
}
```

您将收到以下编译错误：

```
Output
oct.reset undefined (cannot refer to unexported field or method greet.Octopus.reset)
```

要从`八达通` `导出` `重置`功能，请将`重置`中的`R`大写：

greet.go

```go title="greet.go"
package greet

import "fmt"

var Shark = "Sammy"

type Octopus struct {
 Name  string
 Color string
}

func (o Octopus) String() string {
 return fmt.Sprintf("The octopus's name is %q and is the color %s.", o.Name, o.Color)
}

func (o *Octopus) Reset() {
 o.Name = ""
 o.Color = ""
}

func Hello() {
 fmt.Println("Hello, World!")
}
```

因此，您可以从其他软件包中调用`重置`，而不会出现错误：

```go title="main.go"
package main

import (
 "fmt"

 "github.com/gopherguides/greet"
)

func main() {
 greet.Hello()

 fmt.Println(greet.Shark)

 oct := greet.Octopus{
  Name:  "Jesse",
  Color: "orange",
 }

 fmt.Println(oct.String())

 oct.Reset()

 fmt.Println(oct.String())
}
```

现在运行程序

```bash
go run main.go
```

您将收到以下输出结果：

```
Output
Hello, World!
Sammy
The octopus's name is "Jesse" and is the color orange
The octopus's name is "" and is the color .
```

通过调用`重置`，您清除了`名称`和`颜色`字段中的所有信息。调用`String`方法时，由于`Name`和`Color`字段现在是空的，因此通常不会打印出任何信息。

## [结论]

编写 Go 软件包与编写其他任何 Go 文件一样，但将其放在另一个目录中可以隔离代码，以便在其他地方重复使用。本教程介绍了如何在软件包中编写定义，演示了如何在其他 Go 编程文件中使用这些定义，并解释了在何处保存软件包以便访问的选项。
