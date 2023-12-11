# 简介

“Hello, World！”程序是计算机编程中的经典传统。这是一个简单而完整的初学者第一个程序，也是确保您的环境正确配置的良好方式。

本教程将指导您在Go中创建此程序。但是，为了使程序更有趣，您将修改传统的“Hello, World！”程序，以便询问用户的姓名。然后，您将在问候中使用该姓名。完成教程后，当您运行它时，程序将如下所示：

输出
请输入您的姓名。
Sammy
你好，Sammy！我是Go！

## 先决条件

在开始本教程之前，您需要在计算机上设置本地Go开发环境。您可以按照以下教程之一进行设置：

在macOS上安装Go并设置本地编程环境
在Ubuntu 18.04上安装Go并设置本地编程环境
在Windows 10上安装Go并设置本地编程环境

### 第1步 —— 编写基本的“Hello, World！”程序

为了编写“Hello, World！”程序，打开终端窗口，使用nano等命令行文本编辑器创建一个新文件：

```bash
nano hello.go
```

一旦文本文件在终端窗口中打开，您将输入以下程序：

**hello.go**
```go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")
}
```

让我们分解代码的不同部分。

`package`是Go的关键字，定义了该文件属于哪个代码包。每个文件夹只能有一个包，并且每个.go文件在其文件顶部必须声明相同的包名称。在此示例中，该代码属于主要包。

`import`是Go的关键字，告诉Go编译器您要在此文件中使用哪些其他包。在此处，您导入了标准库中的fmt包。fmt包提供了在开发过程中可能有用的格式化和打印函数。

`fmt.Println`是Go函数，位于fmt包中，告诉计算机将一些文本打印到屏幕上。

您通过一系列字符（如"Hello, World!"）跟在fmt.Println函数后面，用引号括起来。引号内的任何字符都被称为字符串。当程序运行时，fmt.Println函数将此字符串打印到屏幕上。

保存并退出nano，按CTRL + X，提示保存文件时按Y。

现在，您可以尝试运行程序。

### 第2步 —— 运行Go程序

编写“Hello, World！”程序后，您准备运行该程序。您将使用`go`命令，后跟刚刚创建的文件的名称。

```bash
go run hello.go
```

程序将执行并显示此输出：

输出
Hello, World!
让我们探讨实际发生了什么。

Go程序在运行之前需要进行编译。当您使用文件名（在本例中为hello.go）调用`go run`时，`go`命令将编译该应用程序，然后运行生成的二进制文件。对于使用编译型编程语言编写的程序，编译器将程序的源代码，并生成另一种较低级别的代码（如机器代码），以生成可执行程序。

Go应用程序需要一个main包和正好一个main()函数，它作为应用程序的入口点。main函数不带参数并且不返回值。相反，它告诉Go编译器应将该包编译为可执行包。

一旦编译，代码通过进入主包中的main()函数来执行。它通过调用fmt.Println函数执行fmt.Println("Hello, World!")这一行。然后将Hello, World!的字符串值传递给函数。在本例中，字符串Hello, World!也被称为参数，因为它是传递给方法的值。

在Hello, World!两侧的引号不会被打印到屏幕上，因为您使用它们告诉Go字符串从何处开始和结束。

在这一步中，您已经使用Go创建了一个有效的“Hello, World！”程序。在下一步中，您将探讨如何使程序更加交互。

### 第3步 —— 提示用户输入

每次运行程序时，它都会生成相同的输出。在此步骤中，您可以添加到程序，以提示用户输入其名称。然后，您将在输出中使用他们的名字。

不要修改现有程序，而是使用nano编辑器创建一个名为greeting.go的新程序：

```bash
nano greeting.go
```

首先，添加以下代码，提示用户输入其名称：

**greeting.go**
```go
package main

import (
	"fmt"
)

func main() {
  fmt.Println("Please enter your name.")
}
```

再次使用fmt.Println函数将一些文本打印到屏幕上。

现在，添加突出显示的行以存储用户的输入：

**greeting.go**
```go
package main

import (
	"fmt"
)

func main() {
  fmt.Println("Please enter your name.")
  var name string
}
```

`var name string`行将使用`var`关键字创建一个新变量。您将变量命名为name，它的类型是string。

然后，添加突出显示的行以捕获用户的输入：

**greeting.go**
```go
package main

import (
	"fmt"
)

func main() {
  fmt.Println("Please enter your name.")
  var name string
  fmt.Scanln(&name)
}
```

`fmt.Scanln`方法告诉计算机等待键盘输入，直到遇到换行符（\n）为止。这会暂停程序，允许用户输入他们想要的任何文本。当用户按下键盘上的ENTER键时，程序将继续。然后将所有按键，包括ENTER按键，捕获并转换为一个字符字符串。

您希

望在程序的输出中使用这些字符，因此通过将它们写入名为name的字符串变量来保存这些字符。Go将该字符串存储在计算机的内存中，直到程序运行结束。

最后，添加以下突出显示的行以打印输出：

**greeting.go**
```go
package main

import (
	"fmt"
)

func main() {
  fmt.Println("Please enter your name.")
  var name string
  fmt.Scanln(&name)
  fmt.Printf("Hi, %s! I'm Go!", name)
}
```

这次，不再使用fmt.Println方法，而是使用fmt.Printf。fmt.Printf函数接受一个字符串，并使用特殊的打印动词（%s），将name的值注入到字符串中。您这样做是因为Go不支持字符串插值，它允许您将分配给变量的值放入字符串中。

保存并退出nano，按CTRL + X，提示保存文件时按Y。

现在运行程序。您将被要求输入姓名，因此输入并按ENTER。输出可能不是您期望的：

输出
Please enter your name.
Sammy
Hi, Sammy
! I'm Go!
与Hi, Sammy! I'm Go!不同，姓名后面有一个换行符。

程序捕获了我们按下的所有按键，包括我们按下ENTER键以告诉程序继续。在字符串中，按下ENTER键会创建一个特殊字符，该字符会创建一个新行。程序的输出正是您告诉它要做的事情；它显示您输入的文本，包括该新行。这不是您期望的输出，但您可以使用其他函数进行修复。

在编辑器中打开greeting.go文件：

```bash
nano greeting.go
```

在程序中找到这行：

**greeting.go**
```go
...
fmt.Scanln(&name)
...
```

在其后添加以下行：

**greeting.go**
```go
name = strings.TrimSpace(name)
```

这使用Go的标准库`strings`包中的`TrimSpace`函数，对使用`fmt.Scanln`捕获的字符串执行操作。`strings.TrimSpace`函数会从字符串的开头和结尾删除任何空格字符，包括换行符。在这种情况下，它会删除在按下ENTER键时创建的字符串末尾的换行符。

要使用`strings`包，您需要在程序顶部导入它。

在程序中找到这些行：

**greeting.go**
```go
import (
	"fmt"
)
```

添加以下行以导入`strings`包：

**greeting.go**
```go
import (
	"fmt"
	"strings"
)
```

您的程序现在将包含以下内容：

**greeting.go**
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println("Please enter your name.")
	var name string
	fmt.Scanln(&name)
	name = strings.TrimSpace(name)
	fmt.Printf("Hi, %s! I'm Go!", name)
}
```

保存并退出nano。按CTRL + X，然后在提示时按Y保存文件。

再次运行程序：

```bash
go run greeting.go
```

这次，在输入姓名并按ENTER后，您将获得预期的输出：

输出
Please enter your name.
Sammy
Hi, Sammy! I'm Go!
您现在有一个Go程序，可以接受用户输入并将其打印回屏幕上。

## 结论

在本教程中，您编写了一个“Hello, World！”程序，该程序接受用户的输入，处理结果并显示输出。现在您有一个基本的程序可以使用，尝试进一步扩展您的程序。例如，询问用户最喜欢的颜色，并让程序说它最喜欢的颜色是红色。您甚至可以尝试使用相同的技术创建一个简单的Mad-Lib程序。