# 如何在围棋中使用日期和时间

## 介绍

软件旨在使工作更容易完成，对许多人来说，这包括与日期和时间进行交互。日期和时间值在现代软件中随处可见。例如，跟踪汽车何时需要服务并让车主知道，跟踪数据库中的更改以创建审计日志，或者只是将一个时间与另一个时间进行比较以确定过程花费了多长时间。因此，检索当前时间，操作时间值以从中提取信息，并以易于理解的格式将其显示给用户是应用程序的基本属性。

在本教程中，您将创建一个Go程序来获取计算机的当前本地时间，然后将其以更易于阅读的格式打印到屏幕上。接下来，您将解释一个字符串以提取日期和时间信息。您还将转换两个时区之间的日期和时间值，以及添加或减去时间值以确定两个时间之间的间隔。

## 先决条件

要遵循本教程，您需要：

- Go 1.16或更高版本已安装。要设置此设置，请按照[如何]为您的操作系统安装Go教程。

## 获取当前时间

在本节中，您将使用Go的`time`包获取当前时间。Go标准库中的`time`包提供了各种与日期和时间相关的函数，并且可以使用`time.Time`类型来表示特定的时间点。除了时间和日期之外，它还可以保存有关所表示的日期和时间所在时区的信息。

要开始创建一个程序来浏览`time`包，您需要为这些文件创建一个目录。这个目录可以创建在您计算机上的任何地方，但是许多开发人员倾向于为他们的项目创建一个目录。在本教程中，您将使用名为`projects`的目录。

创建`projects`目录并导航到它：

```bash
mkdir projects
cd projects
```

从`projects`目录中，运行`mkdir`以创建`datetime`目录，然后使用`cd`导航到该目录：

```bash
mkdir datetime
cd datetime
```

创建项目目录后，使用`main.go`或您喜欢的编辑器打开`nano`文件：

```bash
nano main.go
```

在`main.go`文件中，添加一个`main`函数，它将获取当前时间并将其打印出来：

projects/datetime/main.go

```go
package main

import (
 "fmt"
 "time"
)

func main() {
 currentTime := time.Now()
 fmt.Println("The time is", currentTime)
}
```

在这个程序中，来自`time.Now`包的`time`函数用于获取当前本地时间作为`time.Time`值，然后将其存储在`currentTime`变量中。一旦它被存储在变量中，`fmt.Println`函数就使用`currentTime`的默认字符串输出格式将`time.Time`打印到屏幕上。

使用`go run`和`main.go`文件运行程序：

```bash
go run main.go
```

显示当前日期和时间的输出类似于：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT m=+0.000066626
```

输出将显示您当前的日期和时间，这将与示例不同。此外，您看到的时区（此输出中的`-0500 CDT`）可能会有所不同，因为`time.Now()`返回本地时区的时间。

您可能还注意到输出中有一个`m=`值。这个值是[单调时钟](https://pkg.go.dev/time#hdr-Monotonic_Clocks)，在Go内部测量时间差时使用。单调时钟的设计是为了补偿程序运行时计算机系统时钟的日期和时间的任何潜在变化。通过使用单调时钟，将`time.Now`值与5分钟后的`time.Now`值进行比较仍然会得到正确的结果（5分钟间隔），即使计算机的系统时钟在5分钟间隔内向前或向后改变一小时。对于本教程中的代码或示例，您

现在，虽然您确实显示了当前时间，但它可能对用户没有用处。它可能不是您要查找的格式，或者它可能包含的日期或时间部分多于您想要显示的部分。

幸运的是，`time.Time`类型包含各种方法来获取所需日期或时间的特定部分。例如，如果你只想知道`currentTime`变量的年份部分，你可以使用`Year`方法，或者使用`Hour`方法获取当前的小时。

再次打开`main.go`文件，并将一些`time.Time`方法添加到输出中，看看它们会产生什么：

projects/datetime/main.go

```go
...

func main() {
 currentTime := time.Now()
 fmt.Println("The time is", currentTime)
 
 fmt.Println("The year is", currentTime.Year())
 fmt.Println("The month is", currentTime.Month())
 fmt.Println("The day is", currentTime.Day())
 fmt.Println("The hour is", currentTime.Hour())
 fmt.Println("The minute is", currentTime.Hour())
 fmt.Println("The second is", currentTime.Second())
}
```

接下来，使用`go run`再次运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output

The time is 2021-08-15 14:30:45.0000001 -0500 CDT m=+0.000066626
The year is 2021
The month is August
The day is 15
The hour is 14
The minute is 14
The second is 45
```

与前面的输出一样，您当前的日期和时间将与示例不同，但格式应该相似。这一次在输出中，您将看到完整的日期和时间像以前一样打印出来，但您也有一个年、月、月中的一天、小时、分钟和秒的列表。请注意，月份不是打印为数字（就像在完整日期中一样），而是显示为英文字符串`August`。这是因为`Month`方法将月份作为`time.Month`类型返回，而不仅仅是一个数字，并且该类型被设计为在打印为`string`时打印出完整的英文名称。

现在，再次更新程序中的`main.go`文件，并将各种函数输出替换为对`fmt.Printf`的单个函数调用，这样您就可以以更接近您可能想要显示给用户的格式打印当前日期和时间：

projects/datetime/main.go

```go
...

func main() {
 currentTime := time.Now()
 fmt.Println("The time is", currentTime)
 
 fmt.Printf("%d-%d-%d %d:%d:%d\n",
  currentTime.Year(),
  currentTime.Month(),
  currentTime.Day(),
  currentTime.Hour(),
  currentTime.Hour(),
  currentTime.Second())
}
```

将更新保存到`main.go`文件后，请像以前一样使用`go run`命令运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT m=+0.000066626
2021-8-15 14:14:45
```

这一次，您的输出可能更接近您想要的，但仍然有一些事情可以调整输出。月份现在再次以数字格式显示，因为`fmt.Printf`格式使用`%d`来告诉`time.Month`类型它应该使用数字而不是`string`，但它只显示为个位数。如果您想显示两位数，您可以更改`fmt.Printf`格式，但是如果您还想显示12小时时间而不是上面输出中显示的24小时时间，该怎么办？使用`fmt.Printf`方法，你必须自己做数学计算。使用`fmt.Printf`打印日期和时间是可能的，但正如你所看到的，它最终会变得很麻烦。这样做，您可能会为每个想要显示的部分设置大量的行，或者您需要自己进行一些计算来确定要显示的内容。

在本节中，您创建了一个新程序来使用`time.Now`获取当前时间。一旦有了当前时间，就可以使用各种函数，比如在`Year`类型上使用`Hour`和`time.Time`来打印当前时间的信息。它确实开始成为一个自定义格式显示它的大量工作，虽然。为了使这种常见的工作更容易，许多编程语言，包括Go，提供了一种特殊的方式来格式化日期和时间，类似于如何使用`fmt.Printf`来格式化字符串。

## 打印和打印特定日期

除了`Year`、`Hour`和其他与数据相关的方法之外，`time.Time`类型还提供了一个名为`Format`的方法。`Format`方法允许您提供一个布局`string`，类似于您如何提供`fmt.Printf`或`fmt.Sprintf`格式，它将告诉`Format`方法您希望如何打印日期和时间。在本节中，您将复制在上一节中添加的时间输出，但使用`Format`方法以更简洁的方式进行复制。

但是，在使用`Format`方法之前，如果每次运行程序时都不更改日期和时间，那么将更容易看到`Format`如何影响日期和时间的输出。到目前为止，您使用`time.Now`获取当前时间，因此每次运行它都会显示不同的数字。Go `time`包提供了另一个有用的函数，`time.Date`函数，它允许你指定一个特定的日期和时间来表示`time.Time`。

要开始在程序中使用`time.Date`而不是`time.Now`，请再次打开`main.go`文件并更新它，将`time.Now`替换为`time.Date`：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)
}
```

`time.Date`函数的参数包括年、月、月中的一天、小时、分钟和你想要的日期和时间的秒。最后两个参数中的第一个表示纳秒，最后一个参数是要为其创建时间的时区。使用时区本身将在本教程后面介绍。

保存更新后，使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
```

您看到的输出现在应该更接近上面的输出，因为您在运行程序时使用的是特定的本地日期和时间，而不是当前日期和时间。（根据您的计算机设置的时区，时区可能会显示不同。输出格式看起来仍然与您之前看到的类似，因为它仍然使用默认的`time.Time`格式。现在，您已经有了一个标准的日期和时间，您可以使用它来开始调整使用`Format`方法显示时间时的格式。

### 使用`Format`方法自定义日期字符串

许多其他编程语言都包含类似的方式来格式化要显示的日期和时间，但是Go构建这些格式的布局的方式可能与您在其他语言中使用过的方式略有不同。在其他语言中，日期格式使用类似于Go中`Printf`的工作方式，其中`%`字符后跟一个字母，表示要插入的日期或时间部分。例如，一个4位数的年份可以用`%Y`表示。然而，在Go中，日期或时间的这些部分由表示特定日期的字符表示。要在Go date格式中包含4位数的年份，实际上需要在字符串本身中包含`2006`。这种布局的好处是，您在代码中看到的内容实际上代表了您将在输出中看到的内容。当您能够看到输出的表示时，它可以更容易地仔细检查您的格式是否与您正在寻找的内容相匹配，并且还可以使其他开发人员更容易理解程序的输出，而无需首先运行程序。

Go在字符串格式中用于日期和时间布局的特定日期是`01/02 03:04:05PM '06 -0700`。如果查看日期和时间的每个组成部分，您将看到它们每增加一个部分。月份首先出现在`01`，然后是月份的第一天出现在`02`，然后是小时出现在`03`，分钟出现在`04`，第二个出现在`05`，年份出现在`06`（或`2006`），最后是时区出现在`07`。记住此顺序将使将来创建日期和时间格式更容易。格式化选项的例子也可以在Go语言的`time`包文档中找到

现在，使用这个新的`Format`方法来复制和清理上一节中打印的日期格式。它需要几行代码和函数调用来显示，使用`Format`方法应该可以更容易和更清晰地复制它。

打开`main.go`文件，添加一个新的`fmt.Println`调用，并使用`theTime`方法将日期格式化为`Format`：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)

 fmt.Println(theTime.Format("2006-1-2 15:4:5"))
}
```

如果您查看用于格式化的布局，您将看到它使用与上面相同的时间来指定日期和时间的格式（2006年1月2日）。需要注意的一点是，小时使用`15`而不是像前面的例子那样使用`03`。这表明您希望以24小时格式而不是12小时格式显示小时。

要查看这种新格式的输出，请保存您的程序并使用`go run`运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-8-15 14:30:45
```

您现在看到的输出将与上一节结尾处的输出类似，但要简单得多。您只需要包含一行代码和一个布局字符串。`Format`函数为您完成其余的工作。

根据您显示的日期或时间，使用可变长度格式（如您在直接打印数字时使用的格式）可能很难为您自己、您的用户或其他试图读取值的代码读取。使用`1`作为月份格式将导致三月显示为`3`，而十月将使用两个字符并显示为`10`。现在，打开`main.go`，并添加一个额外的行到您的程序与另一个，更结构化的布局。在此布局中，在组件上包含`0`前缀，并将小时更新为使用12小时格式：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)

 fmt.Println(theTime.Format("2006-1-2 15:4:5"))
 fmt.Println(theTime.Format("2006-01-02 03:04:05 pm"))
}
```

保存代码后，使用`go run`再次运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-8-15 14:30:45
2021-08-15 02:30:45 pm
```

您将看到，通过向布局字符串中的数字添加`0`前缀，新输出中表示月份的`8`将变为`08`以匹配布局。小时，现在是12小时格式，也有自己的`0`前缀。不过，最终在输出中看到的内容反映了在代码中看到的格式，因此，如果需要，可以更容易地调整格式。

很多时候，格式化的日期是要由其他程序解释的，每次想要使用它们时重新创建这些格式可能会成为一种负担。在某些情况下，您可以使用预定义的格式。

### 使用预定义格式

有许多常用的日期格式，例如日志消息的时间戳，每次想要使用它们时重新创建它们可能会很麻烦。对于其中的一些情况，`time`包包括您可以使用的预定义格式。

[RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)中定义的格式是一种可用且经常使用的格式。[RFC](https://en.wikipedia.org/wiki/Request_for_Comments)是一个文档，用于定义互联网上的标准如何工作，然后其他RFC可以相互建立。例如，有一个RFC定义了HTTP的工作方式（[RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616)），还有其他建立在此基础上的RFC进一步定义了HTTP。因此，在RFC 3339的情况下，RFC定义了用于Internet上的时间戳的标准格式。这种格式在互联网上是众所周知的，并且得到了很好的支持，所以在其他地方看到它的机会很高。

`time`包中的每个预定义时间格式都由一个以它们所代表的格式命名的`const string`表示。RFC 3339格式有两种可用的格式，一种称为`time.RFC3339`，另一种称为`time.RFC3339Nano`。两种格式之间的区别在于`time.RFC3339Nano`版本在格式中包含纳秒。

现在，再次打开`main.go`文件并更新程序以使用`time.RFC3339Nano`格式输出：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)
 
 fmt.Println(theTime.Format(time.RFC3339Nano))
}
```

由于预定义格式是具有所需格式的`string`值，因此您只需将通常使用的格式替换为其中之一。

要查看输出，请再次使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-08-15T14:30:45.0000001-05:00
```

如果您需要将时间值保存为`string`，则可以使用RFC 3339格式。它可以被许多其他编程语言和应用程序读取，并且与灵活的`string`格式中的日期和时间一样紧凑。

在本节中，您更新了程序以使用`Format`方法打印日期和时间。使用这种灵活的布局还允许您的代码看起来与最终输出相似。最后，您使用了一个预定义的布局字符串，以一种支持良好的格式打印出日期和时间。在下一节中，您将更新程序，将相同的`string`值转换回您可以使用的`time.Time`值。

## 解析字符串中的日期和时间

在开发应用程序时，您经常会遇到表示为`string`值的日期，您需要以某种方式解释这些日期。有时候，你需要知道值的日期部分，有时候你可能需要知道时间部分，而有时候你可能需要整个值。除了使用`Format`方法从`string`值创建`time.Time`值之外，Go `time`包还提供了一个`time.Parse`函数来将`string`值转换为`time.Time`值。`time.Parse`函数的工作方式类似于`Format`方法，因为它将预期的日期和时间布局以及`string`值作为参数。

现在，在程序中打开`main.go`文件并更新它，以使用`time.Parse`函数将`timeString`解析为`time.Time`变量：

projects/datetime/main.go

```go
...

func main() {
 timeString := "2021-08-15 02:30:45"
 theTime, err := time.Parse("2006-01-02 03:04:05", timeString)
 if err != nil {
  fmt.Println("Could not parse time:", err)
 }
 fmt.Println("The time is", theTime)
 
 fmt.Println(theTime.Format(time.RFC3339Nano))
}
```

与`Format`方法不同，`time.Parse`方法还返回一个潜在的`error`值，以防传入的`string`值与作为第一个参数提供的布局不匹配。如果你看一下使用的布局，你会发现给`time.Parse`的布局使用了与`1`方法相同的`2`表示月份，`Format`表示月份的日期等。

保存更新后，您可以使用`go run`运行更新后的程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 02:30:45 +0000 UTC
2021-08-15T02:30:45Z
```

在这个输出中有几件事需要注意。一个是从`timeString`解析的时区使用默认时区，这是一个`+0`偏移，称为[协调世界时](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)（UTC）。由于时间值和布局都不包括时区，因此`time.Parse`函数不知道将其与哪个时区相关联。但是，如果您在某些时候需要它，`time`包确实包含一个[`time.ParseInLocation`](https://pkg.go.dev/time#ParseInLocation)函数，因此您可以提供要使用的位置。另一个需要注意的部分是RFC 3339的输出。输出使用`time.RFC3339Nano`布局，但输出不包括任何纳秒。这是因为`time.Parse`函数没有解析任何纳秒，所以该值被设置为默认值`0`。当纳秒为`0`时，`time.RFC3339Nano`格式将不会在输出中包含纳秒。

在解析`time.Parse`值时，`time`方法也可以使用`string`包中提供的任何预定义时间布局。要在实践中看到这一点，请打开您的`main.go`文件并更新`timeString`值以匹配之前来自`time.RFC3339Nano`的输出，并更新`time.Parse`参数以匹配：

projects/datetime/main.go

```go
...

func main() {
 timeString := "2021-08-15T14:30:45.0000001-05:00"
 theTime, err := time.Parse(time.RFC3339Nano, timeString)
 if err != nil {
  fmt.Println("Could not parse time:", err)
 }
 fmt.Println("The time is", theTime)
 
 fmt.Println(theTime.Format(time.RFC3339Nano))
}
```

一旦你更新了代码，你可以保存你的程序，并使用`go run`再次运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-08-15T14:30:45.0000001-05:00
```

这一次，来自`Format`方法的输出显示`time.Parse`能够解析来自`timeString`的时区和纳秒。

在本节中，您使用了`time.Parse`函数来解析任意格式的日期和时间字符串值以及预定义的布局。不过，到目前为止，您还没有与您看到的各种时区进行交互。Go为`time.Time`类型提供的另一组特性是在不同时区之间转换的能力，正如您将在下一节中看到的那样。

## 使用时区

当开发一个应用程序时，用户来自世界各地，甚至只有几个时区，一个常见的做法是使用[协调世界时](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)（UTC）存储日期和时间，然后在需要时转换为用户的本地时间。这允许数据以一致的格式存储，并使它们之间的计算更容易，因为只有在向用户显示日期和时间时才需要转换。

在本教程的前几节中，您创建了一个主要基于您自己的本地时区的时间工作的程序。要将`time.Time`值保存为UTC，首先需要将它们转换为UTC。您将使用`UTC`方法来完成此操作，该方法将返回您调用它的时间的副本，但以UTC为单位。

注意：此部分将时间转换为您计算机的本地时区和UTC。如果您使用的计算机将本地时区设置为与UTC匹配的时区，您会注意到UTC之间的转换不会显示时间差异。

现在，打开您的`main.go`文件以更新您的程序，以便在`UTC`上使用`theTime`方法来返回UTC版本的时间：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)
 fmt.Println(theTime.Format(time.RFC3339Nano))
 
 utcTime := theTime.UTC()
 fmt.Println("The UTC time is", utcTime)
 fmt.Println(utcTime.Format(time.RFC3339Nano))
}
```

这一次，你的程序在你的本地时区创建了一个`theTime`作为`time.Time`值，以两种不同的格式打印出来，然后使用`UTC`方法将该时间从你的本地时间转换为UTC时间。

要查看程序的输出，您可以使用`go run`运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-08-15T14:30:45.0000001-05:00
The UTC time is 2021-08-15 19:30:45.0000001 +0000 UTC
2021-08-15T19:30:45.0000001Z
```

您的输出将根据您的当地时区而有所不同，但在上面的输出中，您会看到第一次打印出来的时间是`CDT`（北美中部夏令时），距离UTC有`-5`小时。一旦调用了`UTC`方法并打印了UTC时间，您可以看到时间中的小时从`14`变为`19`，因为将时间转换为UTC增加了5个小时。

也可以用同样的方法在`Local`上使用`time.Time`方法将UTC时间转换回本地时间。再次打开你的`main.go`文件并更新它，在`Local`上添加一个对`utcTime`方法的调用，将其转换回你的本地时区：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.Local)
 fmt.Println("The time is", theTime)
 fmt.Println(theTime.Format(time.RFC3339Nano))
 
 utcTime := theTime.UTC()
 fmt.Println("The UTC time is", utcTime)
 fmt.Println(utcTime.Format(time.RFC3339Nano))

 localTime := utcTime.Local()
 fmt.Println("The Local time is", localTime)
 fmt.Println(localTime.Format(time.RFC3339Nano))
}
```

保存文件后，使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-08-15T14:30:45.0000001-05:00
The UTC time is 2021-08-15 19:30:45.0000001 +0000 UTC
2021-08-15T19:30:45.0000001Z
The Local time is 2021-08-15 14:30:45.0000001 -0500 CDT
2021-08-15T14:30:45.0000001-05:00
```

您将看到UTC时间已转换回您的本地时区。在上面的输出中，UTC转换回CDT意味着从UTC中减去了5个小时，将小时从`19`更改为`14`。

在本节中，您更新了程序，使用`UTC`方法在本地时区和标准UTC时区之间转换日期和时间，然后再使用`Local`方法将其转换回来。

一旦你有了你的日期和时间值，Go `time`包提供了一些额外的特性，这些特性在你的应用程序中很有用。这些特征之一是确定给定的时间是在另一个之前还是之后。

## 比较两次

比较两个日期有时可能会很困难，因为在比较它们时需要考虑所有的变量。例如，需要考虑时区，或者每个月有不同的天数。`time`包提供了一些方法来简化此操作。

`time`包提供了两种方法来简化这些比较：`Before`和`After`方法，在`time.Time`类型上可用。这两个方法都接受一个时间值，并返回`true`或`false`，这取决于它们被调用的时间是在提供的时间之前还是之后。

要查看这些函数的示例，请打开`main.go`文件并更新它以包含两个不同的日期，然后使用`Before`和`After`比较这些日期以查看输出：

projects/datetime/main.go

```go
...

func main() {
 firstTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.UTC)
 fmt.Println("The first time is", firstTime)

 secondTime := time.Date(2021, 12, 25, 16, 40, 55, 200, time.UTC)
 fmt.Println("The second time is", secondTime)

 fmt.Println("First time before second?", firstTime.Before(secondTime))
 fmt.Println("First time after second?", firstTime.After(secondTime))

 fmt.Println("Second time before first?", secondTime.Before(firstTime))
 fmt.Println("Second time after first?", secondTime.After(firstTime))
}
```

更新文件后，保存它并使用`go run`运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The first time is 2021-08-15 14:30:45.0000001 +0000 UTC
The second time is 2021-12-25 16:40:55.0000002 +0000 UTC
First time before second? true
First time after second? false
Second time before first? false
Second time after first? true
```

由于代码使用的是UTC时区的显式日期，因此您的输出应该与上面的输出匹配。您将看到，当在`Before`上使用`firstTime`方法并提供它`secondTime`进行比较时，它会说`true`在`2021-08-15`之前的是`2021-12-25`。当使用`After`的`firstTime`方法并提供`secondTime`时，它说`false`在`2021-08-15`之后是`2021-12-25`。更改调用`secondTime`上的方法的顺序会显示相反的结果，正如预期的那样。

使用`time`包比较两个日期和时间的另一种方法是`Sub`方法。`Sub`方法将从一个日期中减去另一个日期，并使用新类型`time.Duration`返回一个值。与表示绝对时间点的`time.Time`值不同，`time.Duration`值表示时间差。例如，“in one hour”是一个持续时间，因为它的含义与当前时间不同，但“at noon”表示一个特定的绝对时间。Go语言在很多地方都使用了`time.Duration`类型，比如当你想定义一个函数在返回一个错误之前应该等待多长时间，或者在这里，你需要知道一个时间和另一个时间之间的时间。

现在，更新您的`main.go`文件，对`Sub`和`firstTime`值使用`secondTime`方法，并打印出结果：

projects/datetime/main.go

```go
...

func main() {
 firstTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.UTC)
 fmt.Println("The first time is", firstTime)

 secondTime := time.Date(2021, 12, 25, 16, 40, 55, 200, time.UTC)
 fmt.Println("The second time is", secondTime)

 fmt.Println("Duration between first and second time is", firstTime.Sub(secondTime))
 fmt.Println("Duration between second and first time is", secondTime.Sub(firstTime))
```

保存文件，然后使用`go run`运行它：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The first time is 2021-08-15 14:30:45.0000001 +0000 UTC
The second time is 2021-12-25 16:40:55.0000002 +0000 UTC
Duration between first and second time is -3170h10m10.0000001s
Duration between second and first time is 3170h10m10.0000001s
```

上面的输出表示两个日期之间有3，170小时，10分钟，10秒和100纳秒，关于输出有一些注意事项。第一个是第一和第二时间之间的持续时间是负值。这意味着第二次是在第一次之后，并且类似于如果你从`5`中减去`0`并得到`-5`。第二件要注意的事情是，持续时间的最大度量单位是一个小时，所以它不会将其分解为几天或几个月。由于一个月中的天数并不一致，并且在夏令时切换期间，“天”可能具有不同的含义，因此小时是最准确的测量值，不会波动。

在本节中，您更新了程序，以便使用三种不同的方法比较两次。首先，您使用了`Before`和`After`方法来确定一个时间是在另一个时间之前还是之后，然后使用`Sub`来查看两个时间之间的间隔。不过，获取两次之间的持续时间并不是`time`包使用`time.Duration`的唯一方式。您还可以使用它来添加或删除`time.Time`值中的时间，正如您将在下一节中看到的那样。

## 添加或减去时间

在编写应用程序时，使用日期和时间的一个常见操作是根据另一个时间确定过去或未来的时间。它可以用于功能，例如确定订阅下次续订的时间，或者自检查某个值以来是否已经过了一定的时间。无论哪种方式，Go `time`包都提供了一种处理它的方法。但是，要根据您已经知道的日期查找另一个日期，您需要能够定义自己的`time.Duration`值。

创建一个`time.Duration`值类似于在纸上写一个持续时间，只是用乘法表示时间单位。例如，要创建一个表示小时的`time.Duration`，可以使用`time.Hour`值乘以希望`time.Duration`表示的小时数来定义它：

```go
oneHour := 1 * time.Hour
twoHours := 2 * time.Hour
tenHours := 10 * time.Hour
```

声明更小的时间单位以类似的方式处理，除了使用`time.Minute`、`time.Second`等：

```go
tenMinutes := 10 * time.Minute
fiveSeconds := 5 * time.Second
```

一个持续时间也可以与另一个持续时间相加，以获得持续时间之和。要查看实际效果，请打开`main.go`文件并更新它，以声明`toAdd` duration变量并为其添加不同的duration：

projects/datetime/main.go

```go
...

func main() {
 toAdd := 1 * time.Hour
 fmt.Println("1:", toAdd)

 toAdd += 1 * time.Minute
 fmt.Println("2:", toAdd)

 toAdd += 1 * time.Second
 fmt.Println("3:", toAdd)
}
```

完成更新后，保存文件并使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
1: 1h0m0s
2: 1h1m0s
3: 1h1m1s
```

当您查看输出时，您会看到第一个打印的持续时间是一个小时，对应于代码中的`1 * time.Hour`。接下来，您将`1 * time.Minute`添加到`toAdd`值，该值显示为一小时一分钟值。最后，您将`1 * time.Second`添加到`toAdd`，这导致`time.Duration`值为1小时、1分钟和1秒。

也可以在一个语句中组合联合收割机将持续时间相加，或从另一个语句中减去持续时间：

```go
oneHourOneMinute := 1 * time.Hour + 1 * time.Minute
tenMinutes := 1 * time.Hour - 50 * time.Minute
```

接下来，打开你的`main.go`文件，更新你的程序，使用这样的组合来从`toAdd`中减去一分一秒：

projects/datetime/main.go

```go
...

func main() {
 
 ...
 
 toAdd += 1 * time.Second
 fmt.Println("3:", toAdd)
 
 toAdd -= 1*time.Minute + 1*time.Second
 fmt.Println("4:", toAdd)
}
```

保存代码后，您可以使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
1: 1h0m0s
2: 1h1m0s
3: 1h1m1s
4: 1h0m0s
```

添加到输出中的新的第四行显示了用于减去`1 * time.Minute`和`1 * time.Second`之和的新代码，该代码工作正常，再次得到原始的一小时值。

使用这些持续时间与`Add`类型的`time.Time`方法配对，允许您采用已有的时间值，并确定该时间之前或之后的其他任意点的时间。要查看运行的示例，请打开`main.go`文件并更新它，将`toAdd`值设置为24小时或`24 * time.Hour`。然后，对`Add`值使用`time.Time`方法来查看该点之后24小时的时间：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.UTC)
 fmt.Println("The time is", theTime)

 toAdd := 24 * time.Hour
 fmt.Println("Adding", toAdd)

 newTime := theTime.Add(toAdd)
 fmt.Println("The new time is", newTime)
}
```

保存文件后，使用`go run`运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 +0000 UTC
Adding 24h0m0s
The new time is 2021-08-16 14:30:45.0000001 +0000 UTC
```

查看输出，您将看到将24小时添加到`2021-08-15`会导致新日期`2021-08-16`。

要减去时间，您还可以使用`Add`方法，这有点违反直觉。由于`Sub`方法用于获取两个日期之间的时间差，因此可以使用带负值的`time.Time`从`Add`值中减去时间。

同样，要在你的程序中看到这一点，打开你的`main.go`文件并更新它，将24小时的`toAdd`值更改为负值：

projects/datetime/main.go

```go
...

func main() {
 theTime := time.Date(2021, 8, 15, 14, 30, 45, 100, time.UTC)
 fmt.Println("The time is", theTime)

 toAdd := -24 * time.Hour
 fmt.Println("Adding", toAdd)

 newTime := theTime.Add(toAdd)
 fmt.Println("The new time is", newTime)
}
```

保存文件后，使用`go run`再次运行程序：

```bash
go run main.go
```

输出将类似于以下内容：

```
Output
The time is 2021-08-15 14:30:45.0000001 +0000 UTC
Adding -24h0m0s
The new time is 2021-08-14 14:30:45.0000001 +0000 UTC
```

这次在输出中，您将看到新的日期比原始时间早24小时，而不是在原始时间上增加24小时。

在本节中，您使用了`time.Hour`、`time.Minute`和`time.Second`来创建不同程度的`time.Duration`值。您还将`time.Duration`值与`Add`方法一起使用，以在原始值之前和之后获得新的`time.Time`值。通过结合使用`time.Now`、`Add`方法以及`Before`或`After`方法，您将可以为应用程序访问强大的日期和时间功能。

## 结论

在本教程中，您使用`time.Now`检索计算机上当前本地时间的`time.Time`值，然后使用`Year`、`Month`、`Hour`和其他方法访问有关该时间的特定信息。然后，您使用`Format`方法使用自定义格式和预定义格式打印时间。接下来，您使用`time.Parse`函数来解释其中包含时间的`string`值并从中提取时间值。一旦您获得了时间值，您就使用`UTC`和`Local`方法在UTC和本地时区之间转换时间。之后，您使用`Sub`方法获取两个不同时间之间的持续时间，然后使用`Add`方法查找相对于不同时间值的时间。

使用本教程中描述的各种函数和方法将在您的应用程序中有很长的路要走，但是如果您感兴趣，Go[`time`](https://pkg.go.dev/time)包还包括许多其他功能。

本教程也是[DigitalOcean][如何在Go中编码]系列的一部分。本系列涵盖了许多Go主题，从首次安装Go到如何使用语言本身。
