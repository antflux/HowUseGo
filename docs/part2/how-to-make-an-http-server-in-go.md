# 如何用 Go 创建 HTTP 服务器

### 介绍

许多开发人员至少花费一些时间来创建服务器，以便在互联网上分发内容。[超文本传输协议](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)（HTTP）为这些内容提供服务，无论是请求一个猫图像还是请求加载您阅读的教程。Go标准库提供了内置的支持，用于创建HTTP服务器来提供Web内容或向这些服务器发出HTTP请求。

在本教程中，您将使用Go的标准库创建一个HTTP服务器，然后扩展服务器以从请求的查询字符串、主体和表单数据中读取数据。您还将更新程序，以便使用自己的HTTP头和状态码来响应请求。

## 先决条件

要遵循本教程，您需要：

- Go 1.16或更高版本已安装。要设置此设置，请按照[如何]为您的操作系统安装Go教程。
- 能力使用 [卷曲](https://curl.se/) 进行Web请求。要了解curl，请查看 [如何使用cURL下载文件].
- 熟悉在Go中使用JSON，可以在[如何在Go中使用JSON]教程中找到。
- 使用Go的`context`包的经验
- 运行goroutines和阅读通道的经验，可以从教程中获得，[如何在Go中并发运行多个函数]。
- 熟悉[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)请求是如何组成和发送的（推荐）。

## 设置项目

在Go中，大部分HTTP功能由标准库中的[`net/http`](https://pkg.go.dev/net/http)包提供，而其余的网络通信由[`net`](https://pkg.go.dev/net)包提供。`net/http`包不仅包括发出HTTP请求的能力，还提供了一个HTTP服务器，您可以使用它来处理这些请求。

在本节中，您将创建一个程序，该程序使用[`http.ListenAndServe`](https://pkg.go.dev/net/http#ListenAndServe)函数启动一个HTTP服务器，该服务器响应请求路径`/`和`/hello`。然后，您将扩展该程序以在同一程序中运行多个HTTP服务器。

但是，在编写任何代码之前，您需要创建程序的目录。许多开发人员将他们的项目保存在一个目录中，以保持它们的组织性。在本教程中，您将使用名为`projects`的目录。

首先，创建`projects`目录并导航到它：

```bash
mkdir projects
cd projects
```

接下来，为您的项目创建目录并导航到该目录。在这种情况下，使用目录`httpserver`：

```bash
mkdir httpserver
cd httpserver
```

现在已经创建了程序的目录，并且位于`httpserver`目录中，可以开始实现HTTP服务器了。

## 倾听请求并提供响应

Go HTTP服务器包括两个主要组件：侦听来自HTTP客户端的请求的服务器和一个或多个响应这些请求的请求处理程序。在本节中，您将首先使用函数`http.HandleFunc`来告诉服务器调用哪个函数来处理对服务器的请求。然后，您将使用`http.ListenAndServe`函数启动服务器，并告诉它侦听新的HTTP请求，然后使用您设置的处理程序函数为它们提供服务。

现在，在您创建的`httpserver`目录中，使用`nano`或您最喜欢的编辑器打开`main.go`文件：

```bash
nano main.go
```

在`main.go`文件中，您将创建两个函数，`getRoot`和`getHello`，作为您的处理函数。然后，您将创建一个`main`函数，并使用它来设置与`http.HandleFunc`函数的请求处理程序，方法是将`/`路径传递给`getRoot`处理程序函数，将`/hello`路径传递给`getHello`处理程序函数。设置好处理程序后，调用`http.ListenAndServe`函数来启动服务器并监听请求。

将以下代码添加到文件中以启动程序并设置处理程序：

main.go

```go
package main

import (
 "errors"
 "fmt"
 "io"
 "net/http"
 "os"
)

func getRoot(w http.ResponseWriter, r *http.Request) {
 fmt.Printf("got / request\n")
 io.WriteString(w, "This is my website!\n")
}
func getHello(w http.ResponseWriter, r *http.Request) {
 fmt.Printf("got /hello request\n")
 io.WriteString(w, "Hello, HTTP!\n")
}
```

在第一段代码中，您为Go程序设置了`package`，为程序设置了`import`所需的包，并创建了两个函数：`getRoot`函数和`getHello`函数。这两个函数具有相同的函数签名，它们接受相同的参数：一个`http.ResponseWriter`值和一个`*http.Request`值。此函数签名用于HTTP处理程序函数，定义为[`http.HandlerFunc`](https://pkg.go.dev/net/http#HandlerFunc)。当向服务器发出一个请求时，它会设置这两个值，并提供有关所发出请求的信息，然后使用这些值调用处理程序函数。

在`http.HandlerFunc`中，[`http.ResponseWriter`](https://pkg.go.dev/net/http#ResponseWriter)值（在处理程序中命名为`w`）用于控制将响应信息写回发出请求的客户端，例如响应的主体或状态代码。然后，使用[`*http.Request`](https://pkg.go.dev/net/http#Request)值（在处理程序中命名为`r`）获取有关进入服务器的请求的信息，例如在`POST`请求的情况下发送的主体或有关发出请求的客户端的信息。

现在，在两个HTTP处理程序中，当请求进入处理程序函数时，使用`fmt.Printf`打印，然后使用`http.ResponseWriter`向响应体发送一些文本。`http.ResponseWriter`是一个[`io.Writer`](https://pkg.go.dev/io#Writer)，这意味着你可以使用任何能够写入该接口的东西来写入响应体。在本例中，您

现在，通过启动`main`函数继续创建程序：

main.go

```go
...
func main() {
 http.HandleFunc("/", getRoot)
 http.HandleFunc("/hello", getHello)

 err := http.ListenAndServe(":3333", nil)
...
```

在`main`函数中，有两个对`http.HandleFunc`函数的调用。对函数的每次调用都为默认服务器多路复用器中的特定请求路径设置一个处理程序函数。服务器多路复用器是一个[`http.Handler`](https://pkg.go.dev/net/http#Handler)，它能够查看请求路径并调用与该路径相关的给定处理函数。因此，在您的程序中，您告诉默认服务器多路复用器在有人请求`getRoot`路径时调用`/`函数，在有人请求`getHello`路径时调用`/hello`函数。

一旦设置好处理程序，您就可以调用`http.ListenAndServe`函数，该函数告诉全局HTTP服务器使用可选的`http.Handler`来侦听特定端口上的传入请求。在你的程序中，你告诉服务器监听`":3333"`。如果不在冒号前指定IP地址，服务器将侦听与您的计算机关联的每个IP地址，并且将侦听端口`3333`。一个[网络端口](https://en.wikipedia.org/wiki/Port_(computer_networking))，比如这里的`3333`，是一台计算机可以让多个程序同时相互通信的一种方式。每个程序都使用自己的端口，因此当客户端连接到特定端口时，计算机知道将其发送到哪个程序。如果您只想允许连接到`localhost`，即IP地址`127.0.0.1`的主机名，您可以改为使用`127.0.0.1:3333`。

您的`http.ListenAndServe`函数还为`nil`参数传递了一个`http.Handler`值。这告诉`ListenAndServe`函数，您希望使用默认的服务器多路复用器，而不是您设置的多路复用器。

`ListenAndServe`是一个阻塞调用，这意味着你的程序不会继续运行，直到`ListenAndServe`完成运行。但是，在您的程序完成运行或HTTP服务器被告知关闭之前，`ListenAndServe`不会完成运行。即使`ListenAndServe`是阻塞的，并且您的程序不包含关闭服务器的方法，但包含错误处理仍然很重要，因为调用`ListenAndServe`可能会失败。因此，在`ListenAndServe`函数中为`main`添加错误处理，如下所示：

main.go

```go
...

func main() {
 ...
 err := http.ListenAndServe(":3333", nil)
  if errors.Is(err, http.ErrServerClosed) {
  fmt.Printf("server closed\n")
 } else if err != nil {
  fmt.Printf("error starting server: %s\n", err)
  os.Exit(1)
 <^>}
}
```

您要检查的第一个错误这通常是一个预期错误，因为您将自己关闭服务器，但它也可以用于在输出中显示服务器停止的原因。在第二个错误检查中，您将检查任何其他错误。如果发生这种情况，它会将错误打印到屏幕上，然后使用`http.ErrServerClosed`函数以错误代码`1`退出程序。

运行程序时可能会看到的一个错误是`address already in use`错误。当`ListenAndServe`无法侦听您提供的地址或端口时，会返回此错误，因为另一个程序已经在使用它。有时，如果端口是常用的，并且计算机上的另一个程序正在使用它，则会发生这种情况，但如果您多次运行自己程序的多个副本，也会发生这种情况。如果您在使用本教程时看到此错误，请确保在再次运行程序之前已从上一步停止程序。

注意事项：如果您看到`address already in use`错误，并且您没有运行程序的另一个副本，这可能意味着其他程序正在使用它。如果发生这种情况，无论您在本教程中看到`3333`，请将其更改为1024以上65535以下的另一个数字，例如`3334`，然后重试。如果您仍然看到错误，您可能需要继续尝试查找未使用的端口。一旦你找到了一个可以工作的端口，就可以将它用于本教程中的所有命令。

现在你的代码已经准备好了，保存你的`main.go`文件，然后使用`go run`运行你的程序。不像你写的其他Go程序，这个程序不会自动退出。运行程序后，继续执行下一个命令：

```bash
go run main.go
```

由于您的程序仍在终端中运行，因此您需要打开第二个终端来与服务器进行交互。当您看到命令或输出与下面的命令颜色相同时，这意味着在第二个终端中运行它。

在第二个终端中，使用[curl](https://curl.se/)程序向HTTP服务器发出HTTP请求。`curl`是一个通常默认安装在许多系统上的实用程序，可以向各种类型的服务器发出请求。在本教程中，您将使用它来发出HTTP请求。您的服务器正在侦听您计算机的端口`3333`上的连接，因此您需要在同一端口上向`localhost`发出请求：

```bash
curl http://localhost:3333
```

输出如下所示：

```
Output
This is my website!
```

在输出中，您将看到来自`This is my website!`函数的`getRoot`响应，因为您访问了HTTP服务器上的`/`路径。

现在，在同一个终端中，向同一个主机和端口发出请求，但在`/hello`命令的末尾添加`curl`路径：

```bash
curl http://localhost:3333/hello
```

您的输出将如下所示：

```
Output
Hello, HTTP!
```

这一次，您将看到来自`Hello, HTTP!`函数的`getHello`响应。

如果返回到运行HTTP服务器函数的终端，则现在有两行来自服务器的输出。一个用于`/`请求，另一个用于`/hello`请求：

```
Output
got / request
got /hello request
```

由于服务器将继续运行，直到程序完成运行，您需要自己停止它。要执行此操作，请按`CONTROL+C`向您的程序发送中断信号以停止它。

在本节中，您创建了一个HTTP服务器程序，但它使用默认的服务器多路复用器和默认的HTTP服务器。使用默认值或全局值可能会导致难以复制的错误，因为程序的多个部分可能会在不同的时间更新它们。如果这导致了错误的状态，那么就很难追踪到bug，因为它可能只存在于某些函数以特定顺序被调用的情况下。因此，为了避免这个问题，您将更新您的服务器，以使用您在下一节中自己创建的服务器多路复用器。

## 多路复用请求处理程序

当您在上一节中启动HTTP服务器时，您为`ListenAndServe`函数传递了`nil`值作为`http.Handler`参数，因为您使用的是默认的服务器多路复用器。因为`http.Handler`是一个[接口]，所以可以创建自己的`struct`来实现该接口。但有时，您只需要一个基本的`http.Handler`，它为特定的请求路径调用单个函数，例如默认的服务器多路复用器。在本节中，您将更新您的程序以使用[`http.ServeMux`](https://pkg.go.dev/net/http#ServeMux)（一个服务器多路复用器）和`http.Handler`包提供的`net/http`实现，您可以在这些情况下使用它们。

`http.ServeMux``struct`可以配置为默认的服务器多路复用器，所以你不需要对你的程序做太多的更新就可以开始使用你自己的而不是全局默认的多路复用器。要更新程序以使用`http.ServeMux`，请再次打开`main.go`文件并更新程序以使用您自己的`http.ServeMux`：

main.go

```go
...

func main() {
 mux := http.NewServeMux()
 mux.HandleFunc("/", getRoot)
 mux.HandleFunc("/hello", getHello)

 err := http.ListenAndServe(":3333", mux)
 
 ...
}
```

在此更新中，您使用`http.ServeMux`构造函数创建了一个新的`http.NewServeMux`，并将其分配给`mux`变量。在此之后，您只需要更新`http.HandleFunc`调用以使用`mux`变量，而不是调用`http`包。最后，您更新了对`http.ListenAndServe`的调用，以提供您创建的`http.Handler`（`mux`）而不是`nil`值。

现在你可以使用`go run`再次运行你的程序：

```bash
go run main.go
```

您的程序将像上次一样继续运行，因此您需要在另一个终端中运行命令来与服务器进行交互。首先，使用`curl`再次请求`/`路径：

```bash
curl http://localhost:3333
```

输出如下所示：

```
Output
This is my website!
```

你会看到这个输出和以前一样。

接下来，对`/hello`路径运行与之前相同的命令：

```bash
curl http://localhost:3333/hello
```

输出如下所示：

```
Output
Hello, HTTP!
```

此路径的输出也与之前相同。

最后，如果返回原始终端，您将看到`/`和`/hello`请求的输出：

```
Output
got / request
got /hello request
```

您对程序所做的更新在功能上是相同的，但这次您使用自己的`http.Handler`而不是默认的。

最后，再次按`CONTROL+C`退出服务器程序。

## 同时运行多个服务器

除了使用您自己的`http.Handler`，Go `net/http`包还允许您使用默认服务器之外的HTTP服务器。有时您可能希望自定义服务器的运行方式，或者您可能希望在同一个程序中同时运行多个HTTP服务器。例如，你可能有一个公共网站和一个私人管理网站，你想从同一个程序运行。因为你只能有一个默认的HTTP服务器，所以你不能用默认的HTTP服务器来做这件事。在本节中，您将更新您的程序，以便在以下情况下使用`http.Server`包提供的两[个`net/http`](https://pkg.go.dev/net/http#Server)值-当您希望对服务器进行更多控制或同时需要多个服务器时。

在`main.go`文件中，您将使用`http.Server`设置多个HTTP服务器。您这将允许您在`context.Context`变量中设置请求来自哪个服务器，因此您可以在处理程序函数的输出中打印服务器。

再次打开您的`main.go`文件并更新它，如下所示：

main.go

```go
package main

import (
 // Note: Also remove the 'os' import.
 "context"
 "errors"
 "fmt"
 "io"
 "net"
 "net/http"
)

const keyServerAddr = "serverAddr"

func getRoot(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()

 fmt.Printf("%s: got / request\n", ctx.Value(keyServerAddr))
 io.WriteString(w, "This is my website!\n")
}
func getHello(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()

 fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))
 io.WriteString(w, "Hello, HTTP!\n")
}
```

在上面的代码更新中，您更新了`import`语句以包含更新所需的包。然后，创建了一个名为`const`的`string``keyServerAddr`值，作为`http.Request`上下文中HTTP服务器地址值的键。最后，您更新了`getRoot`和`getHello`函数以访问`http.Request`的`context.Context`值。一旦有了这个值，就可以在`fmt.Printf`输出中包含HTTP服务器的地址，这样就可以看到两个服务器中的哪一个处理了HTTP请求。

现在，通过添加两个`main`值中的第一个来开始更新`http.Server`函数：

main.go

```go
...
func main() {
 ...
 mux.HandleFunc("/hello", getHello)

 ctx, cancelCtx := context.WithCancel(context.Background())
 serverOne := &http.Server{
  Addr:    ":3333",
  Handler: mux,
  BaseContext: func(l net.Listener) context.Context {
   ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
   return ctx
  },
 }
```

在更新后的代码中，您要做的第一件事是使用可用的函数`context.Context`创建一个新的`cancelCtx`值，以取消上下文。然后，你定义你的`serverOne``http.Server`值。这个值与您已经使用的HTTP服务器非常相似，但是您没有将地址和处理程序传递给`http.ListenAndServe`函数，而是将它们设置为`Addr`的`Handler`和`http.Server`值。

另一个变化是添加了`BaseContext`功能。`BaseContext`是一种改变处理函数在调用`context.Context`的`Context`方法时接收到的`*http.Request`部分的方法。在您的程序中，您正在使用键`l.Addr().String()`将服务器正在侦听的地址（`serverAddr`）添加到上下文中，然后将其打印到处理程序函数的输出中。

接下来，定义第二台服务器`serverTwo`：

main.go

```go
...

func main() {
 ...
 serverOne := &http.Server {
  ...
 }

 serverTwo := &http.Server{
  Addr:    ":4444",
  Handler: mux,
  BaseContext: func(l net.Listener) context.Context {
   ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
   return ctx
  },
 }
```

此服务器的定义方式与第一个服务器相同，只是将`:3333`字段设置为`Addr`，而不是`:4444`。这样，一台服务器将侦听端口`3333`上的连接，第二台服务器将侦听端口`4444`上的连接。

现在，更新你的程序，在一个goroutine中启动第一个服务器`serverOne`：

main.go

```go
...

func main() {
 ...
 serverTwo := &http.Server {
  ...
 }

 go func() {
  err := serverOne.ListenAndServe()
  if errors.Is(err, http.ErrServerClosed) {
   fmt.Printf("server one closed\n")
  } else if err != nil {
   fmt.Printf("error listening for server one: %s\n", err)
  }
  cancelCtx()
 }()
```

在goroutine内部，你用`ListenAndServe`启动服务器，和之前一样，但是这次你不需要像用`http.ListenAndServe`那样为函数提供参数，因为`http.Server`的值已经配置好了。然后，执行与以前相同的错误处理。在函数结束时，调用`cancelCtx`取消提供给HTTP处理程序和两个服务器`BaseContext`函数的上下文。这样，如果服务器由于某种原因结束，上下文也会结束。

最后，更新你的程序，在goroutine中启动第二个服务器：

main.go

```go
...

func main() {
 ...
 go func() {
  ...
 }()
 go func() {
  err := serverTwo.ListenAndServe()
  if errors.Is(err, http.ErrServerClosed) {
   fmt.Printf("server two closed\n")
  } else if err != nil {
   fmt.Printf("error listening for server two: %s\n", err)
  }
  cancelCtx()
 }()

 <-ctx.Done()
}
```

这个goroutine在功能上和第一个一样，它只是从`serverTwo`开始，而不是`serverOne`。此更新还包括`main`函数的结尾，您可以在从`ctx.Done`函数返回之前从`main`通道读取。这确保了你的程序会一直运行，直到任何一个服务器goroutines结束并且`cancelCtx`被调用。一旦上下文结束，程序将退出。

完成后，保存并关闭文件。

使用`go run`命令运行服务器：

```bash
go run main.go
```

你的程序将继续运行，所以在你的第二个终端运行`curl`命令，从监听`/`的服务器请求`/hello`和`3333`路径，与之前的请求相同：

```bash
curl http://localhost:3333
curl http://localhost:3333/hello
```

输出如下所示：

```
Output
This is my website!
Hello, HTTP!
```

在输出中，您将看到与之前看到的相同的响应。

现在，再次运行相同的命令，但这次使用端口`4444`，对应于程序中的`serverTwo`：

```bash
curl http://localhost:4444
curl http://localhost:4444/hello
```

输出如下所示：

```
Output
This is my website!
Hello, HTTP!
```

您将看到这些请求的输出与端口`3333`服务的端口`serverOne`上的请求的输出相同。

最后，回顾一下运行服务器的原始终端：

```
Output
[::]:3333: got / request
[::]:3333: got /hello request
[::]:4444: got / request
[::]:4444: got /hello request
```

输出与您之前看到的类似，但这次它显示了响应请求的服务器。前两个请求显示它们来自侦听端口`3333`（`serverOne`）的服务器，后两个请求来自侦听端口`4444`（`serverTwo`）的服务器。这些是从`BaseContext`的`serverAddr`值检索的值。

您的输出也可能与上面的输出略有不同，具体取决于您的计算机是否设置为使用[IPv6](https://en.wikipedia.org/wiki/IPv6)。如果是，您将看到与上面相同的输出。如果没有，您将看到`0.0.0.0`而不是`[::]`。这样做的原因是，如果配置了，您的计算机将通过IPv6与自身通信，而`[::]`是`0.0.0.0`的IPv6表示法。

完成后，再次使用`CONTROL+C`停止服务器。

在本节中，您使用`http.HandleFunc`和`http.ListenAndServe`创建了一个新的HTTP服务器程序来运行和配置默认服务器。然后，您更新了它，将`http.ServeMux`用于`http.Handler`，而不是默认的服务器多路复用器。最后，您更新了程序，使用`http.Server`在同一程序中运行多个HTTP服务器。

虽然您现在有一个HTTP服务器在运行，但它的交互性不是很好。您可以添加它响应的新路径，但用户没有真正的方式与它进行交互。HTTP协议包括用户可以通过路径与HTTP服务器进行交互的多种方式。在下一节中，您将更新程序以支持第一种类型：查询字符串值。

## 检查请求的查询字符串

用户能够影响他们从HTTP服务器返回的HTTP响应的方法之一是使用[查询字符串](https://en.wikipedia.org/wiki/Query_string)。查询字符串是添加到[URL](https://en.wikipedia.org/wiki/URL)末尾的一组值。它以`?`字符开始，使用`&`作为字符添加其他值。查询字符串值通常用于筛选或自定义HTTP服务器作为响应发送的结果。例如，一个服务器可以使用`results`值来允许用户指定类似于`results=10`的内容，表示他们希望在结果列表中看到10个项目。

在本节中，您将更新`getRoot`处理程序函数，以使用其`*http.Request`值来访问查询字符串值并将其打印到输出。

首先，打开`main.go`文件并更新`getRoot`函数以使用`r.URL.Query`方法访问查询字符串。然后，更新`main`方法以删除`serverTwo`及其所有相关代码，因为您不再需要它：

main.go

```go
...

func getRoot(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()

 hasFirst := r.URL.Query().Has("first")
 first := r.URL.Query().Get("first")
 hasSecond := r.URL.Query().Has("second")
 second := r.URL.Query().Get("second")

 fmt.Printf("%s: got / request. first(%t)=%s, second(%t)=%s\n",
  ctx.Value(keyServerAddr),
  hasFirst, first,
  hasSecond, second)
 io.WriteString(w, "This is my website!\n")
}
...
```

在`getRoot`函数中，您使用`r.URL`的`getRoot`的`*http.Request`字段来访问有关所请求URL的属性。然后使用`Query`字段的`r.URL`方法访问请求的查询字符串值。访问查询字符串值后，可以使用两种方法与数据进行交互。`Has`方法返回一个`bool`值，该值指定查询字符串是否具有一个带有所提供键的值，例如`first`。然后，`Get`方法返回一个`string`，其中包含所提供的键的值。

从理论上讲，您可以始终使用`Get`方法来检索查询字符串值，因为它将始终返回给定键的实际值或空字符串（如果键不存在）。对于许多用途来说，这已经足够好了-但在某些情况下，您可能想知道用户提供空值与根本不提供值之间的区别。根据您的用例，您可能想知道用户是否提供了无值的`filter`，或者他们是否根本没有提供`filter`。如果他们提供了一个没有任何内容的`filter`值，你可能想把它当作“不要给我看任何东西”，而不提供一个`filter`值就意味着“给我看所有东西”。使用`Has`和`Get`可以让你区分这两种情况。

在`getRoot`函数中，您还更新了输出，以显示`Has`和`Get`查询字符串值的`first`和`second`值。

现在，更新您的`main`函数，以便再次使用一台服务器：

main.go

```go
...

func main() {
 ...
 mux.HandleFunc("/hello", getHello)
 
 ctx := context.Background()
 server := &http.Server{
  Addr:    ":3333",
  Handler: mux,
  BaseContext: func(l net.Listener) context.Context {
   ctx = context.WithValue(ctx, keyServerAddr, l.Addr().String())
   return ctx
  },
 }

 err := server.ListenAndServe()
 if errors.Is(err, http.ErrServerClosed) {
  fmt.Printf("server closed\n")
 } else if err != nil {
  fmt.Printf("error listening for server: %s\n", err)
 }
}
```

在`main`函数中，您删除了对`serverTwo`的引用，并将运行`server`（以前的`serverOne`）从goroutine中移到了`main`函数中，类似于您之前运行`http.ListenAndServe`的方式。您也可以将其更改回`http.ListenAndServe`，而不是使用`http.Server`值，因为您只有一个服务器再次运行，但是使用`http.Server`，如果您希望将来对服务器进行任何其他自定义，则需要更新的内容较少。

现在，保存更改后，使用`go run`再次运行程序：

```bash
go run main.go
```

您的服务器将重新开始运行，因此换回第二个终端运行带有查询字符串的`curl`命令。在此命令中，您需要用单引号（`'`）将URL括起来，否则终端的shell可能会将查询字符串中的`&`符号解释为许多shell都包含的“在后台运行此命令”功能。在URL中，对于`first=1`，包括值`first`，对于`second=`，包括值`second`：

```bash
curl 'http://localhost:3333?first=1&second='
```

输出如下所示：

```
Output
This is my website!
```

您将看到`curl`命令的输出与以前的请求相比没有变化。

但是，如果切换回服务器程序的输出，您将看到新的输出包括查询字符串值：

```
Output
[::]:3333: got / request. first(true)=1, second(true)=
```

`first`查询字符串值的输出显示了`Has`方法返回了`true`，因为`first`有一个值，并且`Get`返回了`1`的值。`second`的输出显示`Has`返回了`true`，因为包含了`second`，但是`Get`方法除了空字符串之外没有返回任何东西。您还可以尝试通过添加和删除`first`和`second`或设置不同的值来创建不同的请求，以查看它如何更改这些函数的输出。

完成后，按`CONTROL+C`停止服务器。

在本节中，您更新了程序，再次只使用一个`http.Server`，但您还添加了对从`first`处理程序函数的查询字符串中阅读`second`和`getRoot`值的支持。

不过，使用查询字符串并不是用户向HTTP服务器提供输入的唯一方式。将数据发送到服务器的另一种常见方法是在请求主体中包含数据。在下一节中，您将更新您的程序，以便从`*http.Request`数据中读取请求的主体。

## 阅读请求正文

当创建基于HTTP的API（如[REST API）](https://en.wikipedia.org/wiki/Representational_state_transfer)时，用户可能需要发送的数据超过URL长度限制中可以包含的数据，或者您的页面可能需要接收与如何解释数据无关的数据，如过滤器或结果限制。在这些情况下，通常将数据包含在请求的主体中，并使用`POST`或`PUT` HTTP请求发送数据。

在Go `http.HandlerFunc`中，`*http.Request`值用于访问有关传入请求的信息，它还包括一种使用`Body`字段访问请求主体的方法。在本节中，您将更新您的`getRoot`处理程序函数以读取请求的主体。

要更新你的`getRoot`方法，打开你的`main.go`文件并更新它，使用`ioutil.ReadAll`读取`r.Body`请求字段：

main.go

```go
package main

import (
 ...
 "io/ioutil"
 ...
)

...

func getRoot(w http.ResponseWriter, r *http.Request) {
 ...
 second := r.URL.Query().Get("second")

 body, err := ioutil.ReadAll(r.Body)
 if err != nil {
  fmt.Printf("could not read body: %s\n", err)
 }

 fmt.Printf("%s: got / request. first(%t)=%s, second(%t)=%s, body:\n%s\n",
  ctx.Value(keyServerAddr),
  hasFirst, first,
  hasSecond, second,
  body)
 io.WriteString(w, "This is my website!\n")
}

...
```

在此更新中，您使用`ioutil.ReadAll`函数读取`r.Body`的`*http.Request`属性以访问请求的主体。`ioutil.ReadAll`函数是一个实用函数，它将从`io.Reader`读取数据，直到遇到错误或数据结束。由于`r.Body`是`io.Reader`，因此您可以使用它来读取正文。一旦您读取了主体，您还更新了`fmt.Printf`以将其打印到输出。

保存更新后，使用`go run`命令运行服务器：

```bash
go run main.go
```

由于服务器将继续运行，直到您停止它，转到您的另一个终端，使用带有`POST`选项的`curl`和使用`-X POST`选项的主体发出`-d`请求。您也可以使用之前的`first`和`second`查询字符串值：

```bash
curl -X POST -d 'This is the body' 'http://localhost:3333?first=1&second='
```

您的输出将如下所示：

```
Output
This is my website!
```

处理程序函数的输出是相同的，但您将看到服务器日志已再次更新：

```
Output
[::]:3333: got / request. first(true)=1, second(true)=, body:
This is the body
```

在服务器日志中，您将看到之前的查询字符串值，但现在您还将看到`This is the body`命令发送的`curl`数据。

现在，按`CONTROL+C`停止服务器。

在本节中，您更新了程序，以便将请求的主体读入输出的变量中。通过将这种阅读主体的方式与其他功能结合起来，例如[`encoding/json`](https://pkg.go.dev/encoding/json)将JSON主体解组为Go数据，您将能够创建用户可以以他们熟悉的方式与其他API进行交互的API。

然而，并非所有从用户发送的数据都是以API的形式。许多网站都有要求用户填写的表单，因此在下一节中，您将更新程序，以便在已有的请求正文和查询字符串之外还读取表单数据。

## 检索表单数据

长期以来，使用表单发送数据是用户将数据发送到HTTP服务器并与网站交互的标准方式。表单现在不像过去那么流行了，但它们仍然有很多用途，可以作为用户向网站提交数据的一种方式。`*http.Request`中的`http.HandlerFunc`值还提供了访问此数据的方法，类似于它提供对查询字符串和请求主体的访问的方式。在本节中，您将更新您的`getHello`程序，以便从表单接收用户的姓名，并使用用户的姓名进行响应。

打开`main.go`并更新`getHello`函数，以使用`PostFormValue`的`*http.Request`方法：

main.go

```go
...

func getHello(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()

 fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))

 myName := r.PostFormValue("myName")
 if myName == "" {
  myName = "HTTP"
 }
 io.WriteString(w, fmt.Sprintf("Hello, %s!\n", myName))
}

...
```

现在在您的`getHello`函数中，您正在阅读提交给处理程序函数的表单值，并查找名为`myName`的值。如果未找到值或找到的值为空字符串，则将`myName`变量设置为默认值`HTTP`，以便页面不会显示空名称。然后，您更新了用户的输出，以显示他们发送的名称，或者如果他们没有发送名称，则显示`HTTP`。

要使用这些更新运行服务器，请保存更改并使用`go run`运行它：

```bash
go run main.go
```

现在，在您的第二个终端中，使用`curl`和`-X POST`选项到`/hello` URL，但这次不是使用`-d`来提供数据主体，而是使用`-F 'myName=Sammy'`选项来提供带有值为`myName`的`Sammy`字段的表单数据：

```bash
curl -X POST -F 'myName=Sammy' 'http://localhost:3333/hello'
```

输出如下所示：

```
Output
Hello, Sammy!
```

在上面的输出中，您将看到预期的`Hello, Sammy!`问候语，因为您发送的表单与`curl`一起发送的`myName`是`Sammy`。

您在`r.PostFormValue`函数中使用的用于检索`getHello`表单值的`myName`方法是一个特殊的方法，它只在请求体中包含从表单发送的值。但是，也可以使用一个`r.FormValue`方法，它同时包含表单主体和查询字符串中的任何值。因此，如果你使用`r.FormValue("myName")`，你也可以删除`-F`选项，并在查询字符串中包含`myName=Sammy`，以查看返回的`Sammy`。但是，如果您这样做而不更改`r.FormValue`，则会看到名称的默认`HTTP`响应。注意从何处检索这些值可以避免名称中的潜在冲突或难以跟踪的错误。更严格地使用`r.PostFormValue`是有用的，除非你想灵活地把它放在查询字符串中。

如果你回头看看你的服务器日志，你会看到`/hello`请求的日志记录与以前的请求类似：

```
Output
[::]:3333: got /hello request
```

要停止服务器，请按`CONTROL+C`。

在本节中，您更新了`getHello`处理程序函数，以便从发送到页面的表单数据中读取名称，然后将该名称返回给用户。

在程序中的这一点上，在处理请求时可能会出现一些错误，用户不会得到通知。在下一节中，您将更新处理程序函数以返回HTTP状态码和头。

## 使用标题和状态码进行响应

HTTP协议使用一些用户通常看不到的功能来发送数据，以帮助浏览器或服务器进行通信。其中一个特性称为[状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)，服务器使用它来给予HTTP客户端一个更好的概念，即服务器是否认为请求成功，或者服务器端或客户端发送的请求是否出错。

HTTP服务器和客户端通信的另一种方式是使用[头字段](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)。头字段是一个键和值，客户端或服务器将发送给对方，让他们了解自己。HTTP协议预定义了许多头，例如`Accept`，客户端使用它来告诉服务器它可以接受和理解的数据类型。也可以通过在它们前面加上`x-`，然后加上名称的其余部分来定义自己的名称。

在本节中，您将更新程序，使`myName`的`getHello`表单字段成为必填字段。如果没有为`myName`字段发送值，您的服务器将向客户端返回一个“Bad Request”状态代码，并添加一个`x-missing-field`头，让客户端知道缺少哪个字段。

要将此功能添加到您的程序中，请最后一次打开您的`main.go`文件，并将验证检查添加到`getHello`处理程序函数：

main.go

```go
...

func getHello(w http.ResponseWriter, r *http.Request) {
 ctx := r.Context()

 fmt.Printf("%s: got /hello request\n", ctx.Value(keyServerAddr))

 myName := r.PostFormValue("myName")
 if myName == "" {
  w.Header().Set("x-missing-field", "myName")
  w.WriteHeader(http.StatusBadRequest)
  return
 }
 io.WriteString(w, fmt.Sprintf("Hello, %s!\n", myName))
}

...
```

在此更新中，当`myName`为空字符串时，您不会将默认名称设置为`HTTP`，而是向客户端发送错误消息。首先，使用`w.Header().Set`方法在响应HTTP头中设置一个值为`x-missing-field`的`myName`头。然后，使用`w.WriteHeader`方法将任何响应头以及“Bad Request”状态代码写入客户端。最后，它将从处理程序函数中返回。你要确保你这样做，这样你就不会在错误信息之外意外地向客户端写入一个`Hello, !`响应。

同样重要的是，要确保您以正确的顺序设置标题和发送状态代码。在HTTP请求或响应中，所有的头都必须在主体发送到客户端之前发送，因此任何更新`w.Header()`的请求都必须在调用`w.WriteHeader`之前完成。一旦调用了`w.WriteHeader`，页面的状态将与所有的头一起发送，之后只能写入主体。

保存更新后，您可以使用`go run`命令再次运行程序：

```bash
go run main.go
```

现在，使用您的第二个终端向`curl -X POST` URL发出另一个`/hello`请求，但不包括发送表单数据的`-F`。您还需要包含`-v`选项来告诉`curl`显示详细输出，以便您可以看到请求的所有头部和输出：

```bash
curl -v -X POST 'http://localhost:3333/hello'
```

这一次，在输出中，由于详细的输出，您将在处理请求时看到更多的信息：

```
Output
*   Trying ::1:3333...
* Connected to localhost (::1) port 3333 (#0)
> POST /hello HTTP/1.1
> Host: localhost:3333
> User-Agent: curl/7.77.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 400 Bad Request
< X-Missing-Field: myName
< Date: Wed, 02 Mar 2022 03:51:54 GMT
< Content-Length: 0
< 
* Connection #0 to host localhost left intact
```

输出中的前几行显示`curl`正在尝试连接到`localhost`端口`3333`。

然后，以`>`开头的行显示了`curl`向服务器发出的请求。它说`curl`正在使用HTTP 1.1协议向`POST` URL发出`/hello`请求，以及其他一些头。然后，它发送一个空的body，如空的`>`行所示。

一旦`curl`发送请求，您就可以看到它从带有`<`前缀的服务器接收到的响应。第一行表示服务器以`Bad Request`响应，也称为400状态代码。然后，您可以看到您设置的`X-Missing-Field`头包含一个值`myName`。在发送了一些额外的头部之后，请求完成，而不发送任何主体，这可以通过长度为`Content-Length`的`0`（或主体）来看出。

如果您再次查看服务器输出，您将在输出中看到服务器处理的`/hello`请求：

```
Output
[::]:3333: got /hello request
```

最后一次按`CONTROL+C`停止服务器。

在本节中，您更新了HTTP服务器，以向`/hello`表单输入添加验证。如果名称没有作为表单的一部分发送，则使用`w.Header().Set`设置要发送回客户端的头。一旦设置了header，您就可以使用`w.WriteHeader`将header写入客户端，以及向客户端指示这是一个错误请求的状态代码。

## 结论

在本教程中，您使用Go标准库中的`net/http`包创建了一个新的Go HTTP服务器。然后，您更新了程序以使用特定的服务器多路复用器和多个`http.Server`实例。您还更新了服务器，以便通过查询字符串值、请求正文和表单数据读取用户输入。最后，您更新了服务器，以便使用自定义HTTP头和“Bad Request”状态代码将表单验证信息返回给客户机。

Go HTTP生态系统的一个好处是，许多框架都被设计成可以整齐地集成到Go的`net/http`包中，而不是重新发明大量已经存在的代码。[github.com/go-chi/chi](https://github.com/go-chi/chi)项目就是一个很好的例子。内置在Go中的服务器多路复用器是开始使用HTTP服务器的好方法，但它缺乏大型Web服务器可能需要的许多高级功能。像`chi`这样的项目能够在Go标准库中实现`http.Handler`接口，以适应标准`http.Server`，而无需重写代码的服务器部分。这使他们能够专注于创建[中间件](https://en.wikipedia.org/wiki/Middleware)和其他工具，以增强可用的功能，而不是处理基本的功能。

除了像`chi`这样的项目之外，Go[`net/http`](https://pkg.go.dev/net/http)包还包括许多本教程中没有涉及的功能。要探索更多关于使用cookie或提供HTTPS流量的信息，`net/http`包是一个很好的起点。
