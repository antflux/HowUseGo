# 如何在 Go 中使用标志包

### 导言

如果不进行额外配置，命令行实用程序很少能在开箱即用。良好的默认设置固然重要，但有用的实用程序也需要接受用户的配置。在大多数平台上，命令行实用程序都接受标记来定制命令的执行。标志是在命令名称后添加的键值分隔字符串。通过使用标准库中的`flag`包，Go 可以制作接受标志的命令行实用程序。

在本教程中，你将探索使用`flag`包构建不同类型命令行实用程序的各种方法。您将使用标志来控制程序输出，引入位置参数来混合标志和其他数据，然后执行子命令。

## 使用标志改变程序行为

使用`flag`包包括三个步骤：首先，define variables用于捕获标志值的变量，然后定义 Go 应用程序将使用的标志，最后解析执行时提供给应用程序的标志。`flag`包中的大部分函数都与定义标志和将标志绑定到所定义的变量有关。解析阶段由`Parse()`函数处理。

为了说明这一点，你将创建一个程序，定义一个`Boolean`标志来改变打印到标准输出的信息。如果提供了`-color`标志，程序将以蓝色打印信息。如果没有提供任何标志，则将不带任何颜色地打印信息。

新建一个名为`boolean.go` 的文件：

```bash
vim boolean.go
```

复制

在文件中添加以下代码以创建程序：

```go title="boolean.go"
package main

import (
	"flag"
	"fmt"
)

type Color string

const (
	ColorBlack  Color = "\u001b[30m"
	ColorRed          = "\u001b[31m"
	ColorGreen        = "\u001b[32m"
	ColorYellow       = "\u001b[33m"
	ColorBlue         = "\u001b[34m"
	ColorReset        = "\u001b[0m"
)

func colorize(color Color, message string) {
	fmt.Println(string(color), message, string(ColorReset))
}

func main() {
	useColor := flag.Bool("color", false, "display colorized output")
	flag.Parse()

	if *useColor {
		colorize(ColorBlue, "Hello, DigitalOcean!")
		return
	}
	fmt.Println("Hello, DigitalOcean!")
}
```

本例使用[ANSI Escape 序列](https://en.wikipedia.org/wiki/ANSI_escape_code)指示终端显示彩色输出。这些是专门的字符序列，因此为它们定义一种新类型是合理的。在本例中，我们将该类型称为`Color`，并将其定义为`string`。然后，我们定义了一个颜色调色板，供后面的`const`代码块使用。在`const`代码块后定义的`colorize`函数接受其中一个`Color`常量和一个`string`变量，该`string`变量表示要着色的信息。然后，它首先打印所要求颜色的转义序列，然后打印信息，最后打印特殊的颜色重置序列，要求终端重置颜色，从而指示终端改变颜色。

在`main` 中，我们使用`flag.Bool`函数定义了一个名为`color` 的布尔标志。该函数的第二个参数`false` 将在未提供该标志时为其设置默认值。与您的预期相反，将该参数设置为`true`并不会反转行为，即提供一个标志会导致该标志变为 false。因此，对于布尔标志，该参数的值几乎总是`false`。

最后一个参数是一个文档字符串，可以作为使用信息打印出来。该函数返回的值是一个指向`bool` 变量的指针。下一行中的`flag.Parse`函数使用该指针根据用户传入的标志设置`bool`变量。然后，我们可以通过取消引用指针来检查该`bool`指针的值。有关指针变量的更多信息，请参阅指针[教程]。利用这个布尔值，我们可以在设置了`-color`标志时调用`colorize`，在没有设置该标志时调用`fmt.Println`变量。

保存文件并运行程序，无需任何标记：

```bash
go run boolean.go
```


您将看到以下输出：

```
Output
Hello, DigitalOcean!
```

现在使用`-color`标志再次运行该程序：

```bash
go run boolean.go-color
```

输出将是相同的文本，但这次是蓝色。

标志并不是传递给命令的唯一值。您还可以发送文件名或其他数据。

## 使用位置参数

一般来说，命令会包含一些参数，这些参数是命令关注的主题。例如，`head`命令用于打印文件的第一行，经常被调用为 head`example.txt`。文件`example.txt`是调用`head`命令时的位置参数。

`Parse()`函数将继续解析遇到的标志，直到检测到一个非标志参数。`flag`包通过`Args()`和`Arg()`函数提供这些参数。

为了说明这一点，你将重新实现一个简化的`head`命令，它可以显示给定文件的前几行：

新建一个名为`head.go`的文件，并添加以下代码：

```go title="head.go"
package main

import (
	"bufio"
	"flag"
	"fmt"
	"io"
	"os"
)

func main() {
	var count int
	flag.IntVar(&count, "n", 5, "number of lines to read from the file")
	flag.Parse()

	var in io.Reader
	if filename := flag.Arg(0); filename != "" {
		f, err := os.Open(filename)
		if err != nil {
			fmt.Println("error opening file: err:", err)
			os.Exit(1)
		}
		defer f.Close()

		in = f
	} else {
		in = os.Stdin
	}

	buf := bufio.NewScanner(in)

	for i := 0; i < count; i++ {
		if !buf.Scan() {
			break
		}
		fmt.Println(buf.Text())
	}

	if err := buf.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "error reading: err:", err)
	}
}
```


首先，我们定义了一个`count`变量，用来保存程序应从文件中读取的行数。然后，我们使用`flag.IntVar` 定义`-n`标志，这与原始`head`程序的行为如出一辙。与没有`Var`后缀的`flag`函数不同，该函数允许我们传递自己的变量[指针]。除此以外，`flag.IntVar`的其他参数也与`flag.`Int 类似：标志名称、默认值和说明。与前面的示例一样，我们随后调用`flag.Parse()`来处理用户的输入。

下一部分是读取文件。我们首先定义了一个`io.Reader`变量，该变量将被设置为用户请求的文件或传递给程序的标准输入。在`if`语句中，我们使用`flag.Arg`函数访问所有标志后的第一个位置参数。如果用户提供了文件名，就会将其设置为文件名。否则，它将是空字符串（`""`）。如果存在文件名，我们将使用`os.Open`函数打开该文件，并将之前定义的`io.Reader`设置为该文件。否则，我们将使用`os.Stdin`从标准输入读取数据。

最后一节使用`bufio.NewScanner`创建的`*bufio.Scanner`从`io.Reader`变量`中`读取行数。我们使用`for`循环遍历到`count`值，如果使用`buf.Scan` 扫描行产生了`false`，表明行数少于用户请求的行数，则调用`break`。

使用`head.go`作为文件参数，运行该程序并显示刚才编写的文件内容：

```bash
go run head.go -- head.go
```

分隔符`--`是`flag`软件包识别的特殊标志，表示后面不再有标志参数。运行该命令时，会收到以下输出：

```bash
Output
package main

import (
        "bufio"
        "flag"
```

使用定义的`-n`标志来调整输出量：

```bash
go run head.go-n 1head.go
```


这只会输出软件包语句：

```
Output
package main
```

最后，当程序检测到没有提供位置参数时，就会像`head` 一样从标准输入端读取输入。试试运行这条命令：

```bash
echo "fish\nlobsters\nsharks\nminnows" | go run head.go -n 3
```

您将看到输出结果：

```
Output
fish
lobsters
sharks
```

到目前为止，您所看到的`flag`函数的行为仅限于检查整个命令调用。你并不总是想要这种行为，尤其是在编写支持子命令的命令行工具时。

## 使用 FlagSet 执行子命令

现代命令行应用程序通常使用 "sub-commands"，将一系列工具捆绑在一条命令下。最著名的使用这种模式的工具就是`git`。在查看`git init` 这样的命令时，`git`是命令，而`init`是`git` 的子命令。子命令的一个显著特点是，每个子命令都可以有自己的标志集。

Go 应用程序可以使用`flag.(*FlagSet)`类型支持带有自己标志集的子命令。 为了说明这一点，请创建一个程序，使用两个具有不同标志的子命令来实现一条命令。

新建一个名为`subcommand.go`的文件，并在文件中添加以下内容：

```go title="subcommand.go"
package main

import (
	"errors"
	"flag"
	"fmt"
	"os"
)

func NewGreetCommand() *GreetCommand {
	gc := &GreetCommand{
		fs: flag.NewFlagSet("greet", flag.ContinueOnError),
	}

	gc.fs.StringVar(&gc.name, "name", "World", "name of the person to be greeted")

	return gc
}

type GreetCommand struct {
	fs *flag.FlagSet

	name string
}

func (g *GreetCommand) Name() string {
	return g.fs.Name()
}

func (g *GreetCommand) Init(args []string) error {
	return g.fs.Parse(args)
}

func (g *GreetCommand) Run() error {
	fmt.Println("Hello", g.name, "!")
	return nil
}

type Runner interface {
	Init([]string) error
	Run() error
	Name() string
}

func root(args []string) error {
	if len(args) < 1 {
		return errors.New("You must pass a sub-command")
	}

	cmds := []Runner{
		NewGreetCommand(),
	}

	subcommand := os.Args[1]

	for _, cmd := range cmds {
		if cmd.Name() == subcommand {
			cmd.Init(os.Args[2:])
			return cmd.Run()
		}
	}

	return fmt.Errorf("Unknown subcommand: %s", subcommand)
}

func main() {
	if err := root(os.Args[1:]); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

复制

该程序分为几个部分：`main`函数、`root`函数和执行子命令的各个函数。`main`函数处理命令返回的错误。如果任何函数返回[错误]，`if`语句将捕获错误并打印错误信息，程序将以状态代码`1` 退出，向操作系统的其他部分表明发生了错误。在`main` 中，我们将程序调用的所有参数传递给`root`。我们先将`os.Args`分割，删除第一个参数，即程序名称（在前面的示例中为`./subcommand`）。

`main`函数定义了`[]Runner`，所有子命令都将在此定义。`Runner`是子命令的[接口]，允许`root`使用`Name()`获取子命令的名称，并与`子`命令变量的内容进行比较。在遍历`cmds`变量后找到正确的子命令后，我们将使用其余参数初始化子命令，并调用该命令的`Run()`方法。

我们只定义了一个子命令，尽管这个框架允许我们创建其他子命令。`GreetCommand`使用`NewGreetCommand`实例化，我们使用`flag`.NewFlagSet 创建一个新的`*flag.` `FlagSet`。flag`.NewFlagSet`接收两个参数：一个是标志集的名称，另一个是报告解析错误的策略。我们可以使用 flag.`(*FlagSet).Name`方法访问`*flag`.`FlagSet`的名称。我们在`（*GreetCommand）.Name()`方法中使用它，这样子命令的名称就与我们给`*flag.`FlagSet的名称一致了。`NewGreetCommand`也以与之前示例类似的方式定义了`-name`标志，但它将此作为`*GreetCommand` 的`*flag.FlagSet`字段的一个方法来调用，即`gc.fs`。当`root`调用`*GreetCommand` 的`Init()`方法时，我们会将提供的参数传递给`*flag.FlagSet`字段的`Parse`方法。

如果先编译这个程序，然后再运行它，会更容易看到子命令。编译程序

```bash
go build subcommand.go
```

现在不带参数运行程序：

```bash
./subcommand
```


您将看到以下输出：

```bash
Output
You must pass a sub-command
```

现在使用`greet`子命令运行程序：

```bash
./subcommand greet
```

输出结果如下

```
Output
Hello World !
```

现在使用`-name`标志和`greet`来指定名称：

```bash
./subcommand greet -name Sammy
```

您将看到程序的输出结果：

```
Output
Hello Sammy !
```

这个示例说明了如何在 Go 中构建大型命令行应用程序的一些原则。`FlagSets`的设计目的是让开发人员能够更好地控制标志解析逻辑处理标志的位置和方式。

## 结论

Flags能让用户控制程序的执行方式，从而使应用程序在更多情况下更有用。为用户提供有用的默认设置固然重要，但也应让他们有机会覆盖不适合他们情况的设置。你已经看到，`flag`包提供了灵活的选择，可以向用户展示配置选项。你可以选择几个简单的标志，也可以建立一套可扩展的子命令。无论哪种情况，使用`flag`包都能帮助你按照历史悠久的灵活、可编写脚本的命令行工具的风格创建实用程序。