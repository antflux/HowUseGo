# 如何为 Go 中的错误添加额外信息

## 导言

当 Go 中的函数失败时，函数将使用`错误`接口返回一个值，以便调用者处理该失败。在许多情况下，开发人员会使用[`fmt`](https://pkg.go.dev/fmt)包中的`fmt` [`.Errorf`](https://pkg.go.dev/fmt#Errorf)函数来返回这些值。不过，在 Go 1.13 之前，使用该函数的一个缺点是会丢失任何可能导致错误返回的错误信息。为了解决这个问题，开发人员要么使用软件包提供的方法将错误 "封装 "在其他错误中，要么通过在其`结构`错误类型之一上实现`Error() 字符串`方法来创建自定义错误。不过，如果有很多错误不需要调用者明确处理，创建这些`结构`类型有时会很繁琐，因此在 Go 1.13 中，该语言增加了一些功能，使处理这些情况变得更容易。

其中一项功能是使用`fmt.Errorf`函数对错误进行封装，封装后的`错误`值可在以后解除封装以访问封装后的错误。这就在 Go 标准库中构建了错误封装功能，因此不再需要使用第三方库。

此外，函数[`errors.Is`](https://pkg.go.dev/errors#Is)和[`errors.As`](https://pkg.go.dev/errors#As)可让您更轻松地确定特定错误是否被封装在给定错误的任何位置，还可让您直接访问该特定错误，而无需自己解开所有错误。

在本教程中，您将创建一个程序，使用这些函数在函数返回的错误中包含附加信息，然后创建自己的自定义错误`结构体`，支持封装和解封装功能。

## 先决条件

要学习本教程，您需要

- 已安装 Go 1.13 或更高版本。要进行设置，请根据您的操作系统查看[如何安装 Go]教程。
- (可选）阅读 "[Go 中的错误处理]"可能对本教程更深入地解释错误处理有所帮助，但本教程也将在更高层次上介绍一些相同的主题。
- (可选）本教程是对《[在 Go 中创建自定义错误]》教程的扩展，介绍了自原教程以来 Go 新增的功能。阅读之前的教程会有所帮助，但并非必须。

## 在 Go 中返回和处理错误

当程序中出现错误时，处理这些错误是一种很好的做法，这样用户就不会看到这些错误--但要处理这些错误，首先需要知道这些错误。在 Go 中，您可以使用一种特殊的`接口类型`（`错误`接口）从函数中返回错误信息，从而处理程序中的错误。只要 Go 类型定义了`Error() 字符串`方法，使用`error`接口就可以返回任何 Go 类型的`错误`值。Go 标准库提供了为这些返回值创建`错误`的功能，例如[`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf)函数。

在本节中，您将创建一个带有函数的程序，该函数使用`fmt.Errorf`返回错误，您还将添加一个错误处理程序来检查函数可能返回的错误。(如果您想了解更多有关在 Go 中处理错误的信息，请参阅教程《[在 Go 中处理错误]》）。

许多开发人员都有一个保存当前项目的目录。本教程将使用名为`projects` 的目录。

首先，创建`projects`目录并导航至该目录：

```bash
mkdir projects
cd projects
```

在`projects`目录下新建一个`errtutorial`目录，以保存新程序：

```bash
mkdir errtutorial
```

然后，使用`cd`命令导航到新目录：

```bash
cd errtutorial
```

进入`errtutorial`目录后，使用`go mod init`命令创建一个名为 `errtutorial`:

```bash
go mod init errtutorial
```

创建 Go 模块后，使用`nano` 或你喜欢的编辑器在`Ertutorial`目录中打开名为`main.go`的文件：

```bash
nano main.go
```

接下来，你将编写一个程序。程序将循环处理数字`1`至`3`，并尝试使用名为`validateValue` 的函数确定这些数字是否有效。如果确定数字无效，程序将使用`fmt.Errorf`函数生成一个从函数返回的`错误`值。`fmt.Errorf`函数允许您创建一个`错误`值，其中的错误信息就是您提供给函数的信息。它的工作原理与`fmt.Printf` 类似，但不是将信息打印到屏幕上，而是以`错误`的形式返回。

然后，在`main`函数中检查错误值是否为`nil`。如果是`nil`值，则函数成功执行，并打印`valid`信息。如果不是，则会打印收到的错误信息。

要开始程序，请在`main.go`文件中添加以下代码：

```go title="projects/errtutorial/main.go"
package main

import (
 "fmt"
)

func validateValue(number int) error {
 if number == 1 {
  return fmt.Errorf("that's odd")
 } else if number == 2 {
  return fmt.Errorf("uh oh")
 }
 return nil
}

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := validateValue(num)
  if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

程序中的`validateValue`函数接收一个数字，然后根据该数字是否为有效值返回`error`。在这个程序中，数字`1`无效，返回错误`that's odd`。数字`2`无效，返回错误`uh oh`。`validateValue`函数使用`fmt.Errorf`函数生成返回的`error`值。`fmt.Errorf`函数在返回错误时非常方便，因为它允许使用类似`fmt.Printf`或`fmt.Sprintf`的格式来格式化错误信息，而无需将该`string`传递给`errors.New`。

在`main`函数中，`for`循环将开始遍历从`1`到`3`的每个数字，并将数值存储在`num`变量中。在循环体中，调用`fmt.Printf`将打印程序当前正在验证的数字。然后，它将调用`validateValue`函数并传入当前正在验证的数字`num`，并将错误结果存储在`err`变量中。最后，如果`err`不是`nil`，则表示在验证过程中发生了错误，错误信息将使用`fmt.Println`出来。如果没有遇到错误，错误检查的`else`子句将打印`"valid!"`。

保存更改后，使用`go run`命令运行程序，参数为`errtutorial`目录中的`main.`go：

```bash
go run main.go
```

运行程序的输出结果将显示，对每个数字都进行了验证，数字`1`和数字`2`返回了相应的错误：

```
Output
validating 1... there was an error: that's odd
validating 2... there was an error: uh oh
validating 3... valid!
```

查看程序的输出结果，你会发现程序尝试验证了所有三个数字。第一次，它显示`validateValue`函数返回了`that's odd`错误，这对于值为`1` 的数字来说是意料之中`的`。最后，`3`返回的错误值为`nil`，这意味着没有错误，数字是有效的。按照`validateValue`函数的编写方式，任何值如果不是`1`或`2`，都会返回`nil`错误值。

在本节中，您使用`fmt.Errorf`创建了从函数返回的`error`值。你还添加了一个`error`处理程序，以便在函数返回错误时打印出错误信息。但有时，了解错误的含义可能会很有用，而不仅仅是发生了错误。在下一节中，你将学习针对特定情况自定义错误处理。

## 使用哨兵错误处理特定错误

当从函数中接收到`error`值时，最基本的错误处理方法是检查`error`值是否为`nil`。这将告诉您函数是否出错，但有时您可能需要针对特定错误情况自定义错误处理。例如，如果您的代码正在连接远程服务器，而返回的唯一错误信息是 "您出错了"。您可能希望知道该错误是因为服务器不可用还是您的连接凭证无效。如果你知道该错误意味着用户的凭据有误，你可能想马上让用户知道。但如果该错误意味着服务器不可用，那么在让用户知道之前，你可能需要尝试重新连接几次。确定了这些错误之间的区别，你就能编写出更强大、更方便用户使用的程序。

检查特定类型错误的一种方法是使用`error`类型上的`Error`方法获取错误信息，并将该值与您要查找的错误类型进行比较。想象一下，在你的程序中，当错误值为 `uh oh` 时，你想显示的信息不是`there was an error: uh oh`。处理这种情况的一种方法是检查`Error`方法返回的值，就像这样：

```go
if err.Error() == "uh oh" {
 // Handle 'uh oh' error.
 fmt.Println("oh no!")
}
```

在这种情况下，检查`err.Error()`的字符串值是否为`uh oh` 值（如上面的代码）是可行的。但如果程序中其他地方的`uh oh`错误`string`略有不同，则代码将不起作用。如果错误信息本身需要更新，那么用这种方法检查错误也会导致代码的大量更新，因为检查错误的每个地方都需要更新。以下面的代码为例：

```go
func giveMeError() error {
 return fmt.Errorf("uh h")
}

err := giveMeError()
if err.Error() == "uh h" {
 // "uh h" error code
}
```

在这段代码中，错误信息包含一个错字，缺少了`uh oh` 中的`o`。如果在某个时候发现并修复了这个问题，但只是在多个地方添加了错误检查之后，所有这些地方都需要将其检查更新为`err.Error() =="uh oh"`。如果漏掉了一个，这很容易，因为只是一个字符的变化，预期的自定义错误处理程序将无法运行，因为它期待的是 uh`h`而不是`uh oh`。

在这种情况下，您可能希望以不同于其他错误的方式处理某个特定错误，通常的做法是创建一个变量来保存错误值。这样，代码就可以根据该变量而不是字符串进行检查。通常，这些变量的名称以`err`或`Err`开头，表示它们是错误。如果错误只能在其定义的软件包中使用，则应使用`Err`前缀。如果错误要在其他地方使用，则应使用`Err`前缀，使其成为一个导出值，类似于函数或`struct`。

现在，假设您在之前的错字示例中使用了其中一个错误值：

```go
var errUhOh = fmt.Errorf("uh h")

func giveMeError() error {
 return errUhOh
}

err := giveMeError()
if err == errUhOh {
 // "uh oh" error code
}
```

在本例中，变量`errUhOh`被定义为 "uh oh "错误（尽管拼写错误）的错误值。函数`giveMeError`返回`errUhOh`的值，因为它想让调用者知道发生了 "uh oh "错误。然后，错误处理代码将`giveMeError`返回的`err`值与`errUhOh`进行比较，以确定是否发生了 "啊哦 "错误。即使发现并修复了错字，所有代码仍然可以工作，因为错误检查是根据`errUhOh` 的值进行的，而`errUhOh`的值是`giveMeError`返回的错误值的固定版本。

以这种方式进行检查和比较的错误值被称为*哨兵错误*。哨兵错误是一种设计为唯一值的错误，可以随时比较其特定含义。上面的`errUhOh`值将始终具有相同的含义，即发生了 "啊哦 "错误，因此程序可以依靠将错误与`errUhOh`进行比较来确定是否发生了该错误。

Go 标准库还定义了许多在开发 Go 程序时可用的哨兵错误。其中一个例子就是[`sql.ErrNoRows`](https://pkg.go.dev/database/sql#pkg-variables)错误。当数据库查询没有返回任何结果时，就会返回`sql.ErrNoRows`错误，因此该错误的处理方式与连接错误不同。由于它是一个哨兵错误，因此可以在错误检查代码中与它进行比较，以了解何时查询没有返回任何记录，而且程序可以以不同于其他错误的方式处理该错误。

一般来说，在创建哨兵错误值时，应使用[`errors`](https://pkg.go.dev/errors)包中的[`errors.New`](https://pkg.go.dev/errors#New)函数，而不是迄今为止一直使用的`fmt.Errorf`函数。不过，使用 `errors.New`代替`fmt.Errorf`并不会对错误的工作方式造成任何基础性改变，而且这两个函数在大多数情况下都可以互换使用。二者最大的区别在于`errors.New`函数只会创建一个带有静态信息的错误，而`fmt.Errorf`函数允许使用值格式化字符串，类似于`fmt.Printf`或`fmt.Sprintf`。由于哨兵错误是基本错误，其值不会改变，因此通常使用`errors.New`来创建它们。

现在，更新程序，在 "uh oh "错误中使用哨兵错误，而不是`fmt.Errorf`。

首先，打开`main.go`文件，添加新的`errUhOh`哨兵错误，并更新程序以使用它。更新`validateValue`函数以返回哨兵错误，而不是使用`fmt.Errorf`。更新`主`函数以检查`errUhOh`哨兵错误，并在遇到该错误时打印`oh no!`，而不是其他错误时显示的`there was an error:`信息。

```go title="projects/errtutorial/main.go"
package main

import (
 "errors"
 "fmt"
)

var (
 errUhOh = errors.New("uh oh")
)

func validateValue(number int) error {
 if number == 1 {
  return fmt.Errorf("that's odd")
 } else if number == 2 {
  return errUhOh
 }
 return nil
}

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := validateValue(num)
  if err == errUhOh {
   fmt.Println("oh no!")
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

现在，保存代码并使用`go run`再次运行程序：

```bash
go run main.go
```

这次的输出将显示`1`值的通用错误输出，但它使用了自定义的 `oh no!`信息,当它看到从 `validateValue` 返回的 `errUhOh` 错误时，为 `2`：

```
Output
validating 1... there was an error: that's odd
validating 2... oh no!
validating 3... valid!
```

在错误检查中使用哨兵错误可以更容易地处理特殊错误情况。例如，哨兵错误可以帮助确定读取文件失败的原因是读取到了文件的末尾（[`io.EOF`](https://pkg.go.dev/io#pkg-variables)哨兵错误的标志），还是由于其他原因。

在本节中，您创建了一个 Go 程序，该程序使用`errors.New`来表示发生特定类型错误时的哨兵错误。不过，随着程序的不断发展，您可能会希望在错误中包含更多信息，而不仅仅是错误值。该错误值没有提供错误发生位置或原因的任何上下文，在大型程序中很难追踪到错误的具体细节。为了帮助排除故障并缩短调试时间，您可以使用错误包装来包含所需的具体信息。

## 包装和拆包错误

包裹错误意味着将一个错误值放入另一个错误值中，就像包裹礼物一样。不过，与包装礼物类似，您需要拆开包装才能知道里面是什么。对错误进行包装可以在不丢失原始错误值的情况下，添加有关错误来源或发生原因的附加信息，因为错误值就在包装内。

在 Go 1.13 之前，可以对错误进行封装，因为您可以创建包含原始错误的自定义错误值。但是，您要么必须创建自己的封装器，要么必须使用已经为您完成这项工作的库。不过，在 Go 1.13 中，Go 添加了[`errors.Unwrap`](https://pkg.go.dev/errors#Unwrap)函数和`fmt.Errorf`函数的`%w`verb，作为标准库的一部分，增加了对封装和解除封装错误的支持。在本节中，您将更新程序以使用`%w`verb 来封装包含更多信息的错误，然后使用`errors.Unwrap`来检索封装的信息。

### 使用`fmt.Errorf`封装错误

封装和解封装错误时需要检查的第一个功能是对现有`fmt.Errorf`函数的补充。过去，`fmt.Errorf`用于创建带有附加信息的格式化错误信息，使用的动词包括用于字符串的`%s`和用于通用值的`%v`。Go 1.13 增加了一个带有特殊情况的新动词，即`%w`动词。当格式字符串中包含`%w`动词并提供了`error`值时，`fmt.Errorf`返回的错误信息将包含正在创建的错误信息中的`error`值。

现在，打开`main.go`文件并更新它，加入一个名为`runValidation` 的新函数。该函数将获取当前正在验证的数字，并在该数字上运行所需的任何验证。在本例中，它只需要运行`validateValue`函数。如果在验证数值时遇到错误，它将使用`fmt.Errorf`和`%w`verb 对错误进行包装，以显示发生了`run error`，然后返回新的错误。您还应更新`主`函数，使其不再直接调用`validateValue`，而是调用`runValidation`：

```go title="projects/errtutorial/main.go"
...

var (
 errUhOh = errors.New("uh oh")
)

func runValidation(number int) error {
 err := validateValue(number)
 if err != nil {
  return fmt.Errorf("run error: %w", err)
 }
 return nil
}

...

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := runValidation(num)
  if err == errUhOh {
   fmt.Println("oh no!")
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

保存更新后，使用`go run`更新后的程序：

```bash
go run main.go
```

输出结果将与此相似：

```
Output
validating 1... there was an error: run error: that's odd
validating 2... there was an error: run error: uh oh
validating 3... valid!
```

从输出结果中可以看出一些问题。首先，你会看到为值`1`打印的错误信息中包含了`run error: that's odd`。这表明该错误已被`runValidation`的`fmt.Errorf`封装，被封装的错误值`that's odd` 也包含在错误信息中。

但接下来出现了一个问题。为`errUhOh`错误添加的特殊错误处理没有运行。如果你看一下验证`2`个输入的那一行，就会发现它显示的是默认的错误信息`：there was an error: run error: uh`oh，而不是预期的`oh no!`你知道`validateValue`函数仍在返回`uh oh`错误，因为你可以在封装错误的末尾看到它，但`errUhOh`的错误检测不再起作用。出现这种情况是因为`runValidation`返回的错误不再是`errUhOh`，而是`fmt.Errorf` 创建的封装错误。当`if`语句尝试比较`err`变量和`errUhOh` 时，返回 false，因为`err`不再等于`errUhOh`，而是等于*包裹* `errUhOh` 的错误。要修复`errUhOh`错误检查，需要使用`errors.Unwrap`函数从包装器内部获取错误。

### 使用`errors.Unwrap`解包错误

除了在 Go 1.13 中添加了`%w`verb 之外，Go[`errors`](https://pkg.go.dev/errors)包中还添加了几个新函数。其中的`errors.Unwrap`函数将`error`作为参数，如果传入的错误是一个错误包装器，它将返回包装后的`error`。如果提供的`error`不是一个封装，函数将返回`nil`。

现在，再次打开`main.go`文件，使用 errors.`Unwrap` 更新`errUhOh`错误检查，以处理`errUhOh`被错误包装器包裹的情况：

```go title="projects/errtutorial/main.go"
func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := runValidation(num)
  if err == errUhOh || errors.Unwrap(err) == errUhOh {
   fmt.Println("oh no!")
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

保存编辑内容后，再次运行程序：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... there was an error: run error: that's odd
validating 2... oh no!
validating 3... valid!
```

现在，在输出中，你会看到针对`2`输入值的`oh no!`错误处理又回来了。你在`if`语句中添加的`errors.Unwrap`函数调用使它既能在 err 本身是`errUhOh`值时检测 errUhOh，也能在`err`是直接封装 `errUhOh` 的错误时检测`errUhOh`。

在本节中，您使用添加到`fmt.Errorf`中的`%w`verb 将`errorUhOh`错误封装到另一个错误中，并为其提供了附加信息。然后，您使用`errors.Unwrap`访问了封装在另一个错误中的`errorUhOh`错误。将错误以`string`值的形式包含在其他错误中对于人类阅读错误信息是没有问题的，但有时你会希望在错误包装中包含附加信息，以帮助程序处理错误，例如 HTTP 请求错误中的状态代码。在这种情况下，您可以创建一个新的自定义错误来返回。

## 定制包裹错误

由于 Go 对`错误`接口的唯一规定是包含`Error`方法，因此可以将许多 Go 类型变成自定义错误。一种方法是定义一个包含额外错误信息的`结构`类型，然后也包含一个`Error`方法。

对于验证错误来说，知道是哪个值导致了错误可能会很有用。接下来，让我们创建一个新的`ValueError`结构，其中包含一个表示导致错误的`值`的字段和一个包含实际验证错误的`Err`字段。自定义错误类型通常在类型名称末尾使用`Error`后缀，以表示该类型符合`错误`接口。

打开您的`main.go`文件，添加新的`ValueError`错误结构体，并添加`newValueError`函数来创建错误实例。您还需要为`ValueError`创建一个名为`Error`的方法，以便将该结构视为`错误`。`Error`方法应返回您希望在错误转换为字符串时显示的值。在本例中，它将使用`fmt.Sprintf`返回一个字符串，显示`value error:`和包装后的错误。此外，更新`validateValue`函数，使其不再只返回基本错误，而是使用`newValueError`函数返回自定义错误：

```go title="projects/errtutorial/main.go"
...

var (
 errUhOh = fmt.Errorf("uh oh")
)

type ValueError struct {
 Value int
 Err   error
}

func newValueError(value int, err error) *ValueError {
 return &ValueError{
  Value: value,
  Err:   err,
 }
}

func (ve *ValueError) Error() string {
 return fmt.Sprintf("value error: %s", ve.Err)
}

...

func validateValue(number int) error {
 if number == 1 {
  return newValueError(number, fmt.Errorf("that's odd"))
 } else if number == 2 {
  return newValueError(number, errUhOh)
 }
 return nil
}

...
```

保存更新后，用`go run` 再次运行程序：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... there was an error: run error: value error: that's odd
validating 2... there was an error: run error: value error: uh oh
validating 3... valid!
```

你会看到，输出结果显示错误被封装在`ValueError`中，输出结果中的 error`:`位于错误之前。但是，`uh oh`错误检测再次被破坏，因为`errUhOh`现在位于两层包装器（`ValueError`和`runValidation` 的`fmt.Errorf`包装器）内。代码只对错误使用了一次`errors.Unwrap`，因此导致第一次`errors.Unwrap(err)`只返回`*ValueError`而不是`errUhOh`。

解决这个问题的方法之一是更新`errUhOh`检查，添加额外的错误检查，即调用`errors.Unwrap()`两次来解除对两层的包裹。要添加这一功能，请打开您的 main`.go`文件，更新您的`主`函数以包含这一更改：

```go title="projects/errtutorial/main.go"
...

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := runValidation(num)
  if err == errUhOh ||
   errors.Unwrap(err) == errUhOh ||
   errors.Unwrap(errors.Unwrap(err)) == errUhOh {
   fmt.Println("oh no!")
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

现在，保存你的`main.go`文件，然后使用`go run` 再次运行你的程序：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... there was an error: run error: value error: that's odd
validating 2... there was an error: run error: value error: uh oh
validating 3... valid!
```

你会发现，呃哦，`errUhOh`特殊错误处理仍然不起作用。在验证`2`个输入的那一行，我们本以为会看到特殊错误处理`oh no!`输出仍然显示默认的`there was an error: run error:`...错误输出。出现这种情况是因为`errors.Unwrap`函数不知道如何解除`ValueError`自定义错误类型。要解除对自定义错误的封装，它需要有自己的`Unwrap`方法，该方法将内部错误作为`error`值返回。在前面使用`fmt.Errorf`和`%w`verb 创建错误时，Go 实际上是在为你创建一个已经添加了`Unwrap`方法的错误，所以你不需要自己动手。但现在您使用的是自己的自定义函数，因此需要添加自己的函数。

要最终修复`errUhOh`错误案例，请打开`main.go`，并为`ValueError`添加一个`Unwrap`方法，该方法会返回`Err`，即内部包裹的错误所存储的字段：

```go title="projects/errtutorial/main.go"
...

func (ve *ValueError) Error() string {
 return fmt.Sprintf("value error: %s", ve.Err)
}

func (ve *ValueError) Unwrap() error {
 return ve.Err
}

...
```

然后，在保存了新的`Unwrap`方法后，运行程序：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... there was an error: run error: value error: that's odd
validating 2... oh no!
validating 3... valid!
```

输出结果显示，`oh no！`错误`ErUhOh`的错误处理已恢复正常，因为`errors.Unwrap`现在也能解除`ValueError`。

在本节中，您创建了一个新的、自定义的`ValueError`错误，作为错误消息的一部分，为自己或用户提供有关验证过程的信息。您还为`ValueError`添加了错误解包支持，因此`errors.Unwrap`可用于访问已封装的错误。

不过，错误处理变得有点笨拙，而且难以维护。每当有一层新的封装时，你就不得不在错误检查中再添加一个`errors.Unwrap`来处理它。值得庆幸的是，`errors`包中的 `errors.Is`和`errors.As`函数可以让处理封装错误变得更容易。

## 处理缠绕错误

如果需要为程序中每一层潜在的错误包装都添加一个新的`errors.Unwrap`函数调用，那么程序将变得非常冗长，而且难以维护。因此，在 Go 1.13 版本中，`error`包中又增加了两个函数。这两个函数都允许你与错误进行交互，无论它们被包裹在其他错误中多深，都能让你更轻松地处理错误。通过 `errors.Is`函数，您可以检查特定的哨兵错误值是否在封装错误的任何位置。通过 `errors.As`函数，您可以在封装错误的任何位置获取对特定类型错误的引用。

### 使用`errors.Is`检查错误值

使用`errors.Is`检查特定错误可使`errUhOh`特殊错误处理更简短，因为它会处理所有嵌套的错误解包。该函数需要两个`错误`参数，第一个参数是实际收到的错误，第二个参数是要检查的错误。

要清理`errUhOh`错误处理，请打开`main` `.go`文件，更新`main`函数中的`errUhOh`检查，改用`errors.Is`代替：

```go title="projects/errtutorial/main.go"
...

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := runValidation(num)
  if errors.Is(err, errUhOh) {
   fmt.Println("oh no!")
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

然后保存代码，使用`go run` 再次运行程序：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... there was an error: run error: value error: that's odd
validating 2... oh no!
validating 3... valid!
```

输出显示了 oh no！错误信息，这意味着即使只对 errUhOh 进行了一次错误检查，仍会在错误链中找到它.`errors.Is`利用错误类型的`Unwrap`方法继续深入挖掘错误链，直到找到要找的错误值（哨兵错误）或遇到返回值为`零nil` 的 `Unwrap`方法。

既然 Go 中存在错误包装功能，那么使用`errors.Is`是检查特定错误的推荐方法。它不仅可用于自己的错误值，还可用于其他错误值，例如本教程前面提到的`sql.ErrNoRows`错误。

### 使用`errors.As`检索错误类型

Go 1.13 中为`errors`包添加的最后一个函数是`errors.As`函数。当您想获取某类错误的引用以与之进行更详细的交互时，就会用到该函数。例如，您之前添加的`ValueError`自定义错误可以访问错误的`Value`字段中正在验证的实际值，但只有先获得该错误的引用，才能访问该值。这就是`errors.As`的作用所在。您可以为 errors`.As`提供一个错误（类似于`errors.Is`）和一个错误类型变量。然后，它将遍历错误链，查看是否有任何已封装的错误与所提供的类型相匹配。如果有匹配的错误，传入的错误类型变量将被设置为`errors.As`found，函数将返回`true`。如果没有匹配的错误类型，函数将返回`false`。

现在，您可以使用`errors.As`利用`ValueError`类型在错误处理程序中显示额外的错误信息。最后一次打开 main`.go`文件，更新`main`函数，为`ValueError` 类型的错误添加一个新的错误处理案例，打印无效数字和验证`error`：

```go title="projects/errtutorial/main.go"
...

func main() {
 for num := 1; num <= 3; num++ {
  fmt.Printf("validating %d... ", num)
  err := runValidation(num)

  var valueErr *ValueError
  if errors.Is(err, errUhOh) {
   fmt.Println("oh no!")
  } else if errors.As(err, &valueErr) {
   fmt.Printf("value error (%d): %v\n", valueErr.Value, valueErr.Err)
  } else if err != nil {
   fmt.Println("there was an error:", err)
  } else {
   fmt.Println("valid!")
  }
 }
}
```

在上面的代码中，你声明了一个新的`valueErr`变量，并使用`errors.As`获取了一个指向`ValueError`的引用（如果它被包裹在`err`值中）。通过访问作为`ValueError` 的错误，您就可以访问该类型提供的任何其他字段，例如验证失败的实际值。如果验证逻辑发生在程序的更深处，而你通常无法访问这些值，从而无法向用户提示可能出错的地方，那么这将很有帮助。另一个有用的例子是，如果您在进行网络编程时遇到了[`net.DNSError`](https://pkg.go.dev/net#DNSError)。通过获取错误的引用，您就能知道该错误是由于无法连接造成的，还是由于可以连接但未找到资源造成的。知道这一点后，您就可以用不同的方法处理该错误。

要查看`errors.As`实际效果，请保存文件并使用以下命令运行程序`go run`：

```bash
go run main.go
```

输出结果将与此类似：

```
Output
validating 1... value error (1): that's odd
validating 2... oh no!
validating 3... valid!
```

这一次在输出中您将看不到默认的`there was an error: ...`消息，因为所有错误都由其他错误处理程序处理。验证`1`的输出显示`errors.As`错误检查返回了`true`，因为正在显示`value error ...`错误消息。由于`errors.As`函数返回 true，因此`valueErr`变量被设置为`ValueError`，并且可以通过访问`valueErr.Value`来打印出验证失败的值。

对于`2`值，输出还显示，即使`errUhOh`也被包装在`ValueError`包装器中，`oh no!`特殊错误处理程序仍然被执行。这是因为使用`errors.Is` for `errUhOh`的特殊错误处理程序在处理错误的`if`语句集合中排在第一位。由于这个处理程序在运行`true`之前返回`errors.As`，所以特殊的`oh no!`处理程序是被执行的。如果代码中的`errors.As`位于`errors.Is`之前，则`oh no!`错误消息将变为与`value error ...`值相同的`1`，但在这种情况下，它将打印`value error (2): uh oh`。

在本节中，您更新了程序，使用`errors.Is`函数删除了对`errors.Unwrap`的大量额外调用，并使错误处理代码更加健壮和面向未来。您还使用了`errors.As`函数来检查是否有任何被包装的错误是`ValueError`，然后如果找到了，则在值上使用字段。

## 结论

在本教程中，您使用`%w`格式动词封装了一个错误，并使用`errors.Unwrap` 解封装了一个错误。您还在自己的代码中创建了支持`errors.Unwrap`的自定义错误类型。最后，您使用自定义错误类型探索了新的辅助函数`errors.Is`和`errors.As`。

使用这些新的错误函数，可以更轻松地包含有关您创建或处理的错误的更深层次信息。此外，它还能为您的代码提供未来保障，确保即使错误变得嵌套很深，错误检查也能继续工作。

如果您想了解如何使用新错误功能的详细信息，Go 博客上有一篇关于在[Go 1.13 中处理错误](https://go.dev/blog/go1.13-errors)的文章。[`errors`](https://pkg.go.dev/errors)软件包的文档也包含更多信息。
