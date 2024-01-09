# 如何在 Go 中发出 HTTP 请求

## 介绍

当一个程序需要与另一个程序通信时，许多开发人员会使用[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)。Go语言的优势之一是其标准库的广度，HTTP也不例外。Go[`net/http`](https://pkg.go.dev/net/http)包不仅支持[创建HTTP服务器]，还可以作为客户端发出HTTP请求。

在本教程中，您将创建一个向HTTP服务器发出几种类型的HTTP请求的程序。首先，您将使用默认的Go HTTP客户端发出`GET`请求。然后，你将增强你的程序，用一个body发出一个`POST`请求。最后，您将自定义您的`POST`请求以包含HTTP标头，并添加一个超时，如果您的请求花费太长时间，则会触发该超时。

## 先决条件

要遵循本教程，您需要：

- Go 1.16或更高版本已安装。要设置此设置，请按照[如何]为您的操作系统安装Go教程。
- 在Go中创建HTTP服务器的经验，可以在教程中找到，[如何在Go中创建HTTP服务器]。
- 熟悉goroutine和阅读通道。有关更多信息，请参阅教程，[如何在Go中并发运行多个函数]。
- 建议理解[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)请求是如何组成和发送的。

## 创建GET请求

Go[`net/http`](https://pkg.go.dev/net/http)包有几种不同的方式将其用作客户端。您可以使用一个通用的、全局的HTTP客户端，并使用诸如[`http.Get`](https://pkg.go.dev/net/http#Get)之类的函数来快速发出一个只有URL和正文的HTTP `GET`请求，或者您可以创建一个[`http.Request`](https://pkg.go.dev/net/http#Request)来开始定制单个请求的某些方面。在本节中，您将使用`http.Get`创建一个初始程序来发出HTTP请求，然后将其更新为使用默认HTTP客户端的`http.Request`。

### 使用`http.Get`进行请求

在程序的第一次迭代中，您将使用`http.Get`函数向程序中运行的HTTP服务器发出请求。`http.Get`函数很有用，因为你不需要在程序中进行任何额外的设置来发出请求。如果您需要快速提出一个请求，`http.Get`可能是最好的选择。

要开始创建程序，您需要一个目录来保存程序的目录。在本教程中，您将使用名为`projects`的目录。

首先，创建`projects`目录并导航到它：

```bash
mkdir projects
cd projects
```

接下来，创建项目目录并导航到该目录。在本例中，使用目录`httpclient`：

```bash
mkdir httpclient
cd httpclient
```

在`httpclient`目录中，使用`nano`或您最喜欢的编辑器打开`main.go`文件：

```bash
nano main.go
```

在`main.go`文件中，开始添加以下行：

main.go

```go
package main

import (
 "errors"
 "fmt"
 "net/http"
 "os"
 "time"
)

const serverPort = 3333
```

您可以添加`package`名称`main`，以便将程序编译为可以运行的程序，然后在此程序中使用的各种包中包含`import`语句。之后，创建一个名为`const`的`serverPort`，值为`3333`，将其用作HTTP服务器监听的端口和HTTP客户端连接的端口。

接下来，在`main`文件中创建一个`main.go`函数，并设置一个goroutine来启动一个HTTP服务器：

main.go

```go
...
func main() {
 go func() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   fmt.Printf("server: %s /\n", r.Method)
  })
  server := http.Server{
   Addr:    fmt.Sprintf(":%d", serverPort),
   Handler: mux,
  }
  if err := server.ListenAndServe(); err != nil {
   if !errors.Is(err, http.ErrServerClosed) {
    fmt.Printf("error running http server: %s\n", err)
   }
  }
 }()

 time.Sleep(100 * time.Millisecond)
```

您的HTTP服务器被设置为使用`fmt.Printf`打印有关传入请求的信息，只要根`/`路径被请求。它也被设置为监听`serverPort`。最后，一旦你启动服务器goroutine，你的程序会在短时间内使用`time.Sleep`。这个休眠时间允许HTTP服务器有足够的时间启动并开始响应您接下来要发出的请求。

现在，同样在`main`函数中，使用`fmt.Sprintf`设置请求URL，将`http://localhost`主机名与服务器正在监听的`serverPort`值组合在一起。然后，使用`http.Get`向该URL发出请求，如下所示：

main.go

```go
...
 requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
 res, err := http.Get(requestURL)
 if err != nil {
  fmt.Printf("error making http request: %s\n", err)
  os.Exit(1)
 }

 fmt.Printf("client: got response!\n")
 fmt.Printf("client: status code: %d\n", res.StatusCode)
}
```

当`http.Get`函数被调用时，Go将使用默认的HTTP客户端向提供的URL发出HTTP请求，如果请求失败，则返回[`http.Response`](https://pkg.go.dev/net/http#Response)或`error`值。如果请求失败，它将打印错误，然后使用[`os.Exit`](https://pkg.go.dev/os#Exit)退出程序，错误代码为`1`。如果请求成功，你的程序将打印出它得到了一个响应和它收到的HTTP状态码。

完成后，保存并关闭文件。

要运行您的程序，请使用`go run`命令并向其提供`main.go`文件：

```bash
go run main.go
```

您将看到以下输出：

```
Output
server: GET /
client: got response!
client: status code: 200
```

在输出的第一行，服务器打印出它从客户端收到了一个针对`GET`路径的`/`请求。然后，接下来的两行表示客户端从服务器收到了一个响应，并且响应的状态代码是`200`。

`http.Get`函数对于快速HTTP请求很有用，就像您在本节中所做的那样。但是，`http.Request`提供了更广泛的选项来定制您的请求。

### 使用`http.Get`进行请求

与`http.Get`相反，`http.Request`函数为您提供了对请求的更大控制，而不仅仅是HTTP方法和请求的URL。您还不会使用其他功能，但是通过使用`http.Request`，您将能够在本教程的后面添加这些自定义。

在您的代码中，第一个更新是更改HTTP服务器处理程序，以使用`fmt.Fprintf`返回假JSON数据响应。如果这是一个完整的HTTP服务器，那么这些数据将使用Go的`encoding/json`包生成如果你此外，您还需要包含`io/ioutil`作为导入，以便稍后在此更新中使用。

现在，再次打开`main.go`文件并更新程序以开始使用`http.Request`，如下所示：

main.go

```go
package main

import (
 ...
 "io/ioutil"
 ...
)

...

func main() {
 ...
 mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  fmt.Printf("server: %s /\n", r.Method)
  fmt.Fprintf(w, `{"message": "hello!"}`)
 })
 ...
```

现在，更新你的HTTP请求代码，不要使用`http.Get`向服务器发出请求，而是使用`http.NewRequest`和`http.DefaultClient`的`Do`方法：

main.go

```go
...
 requestURL := fmt.Sprintf("http://localhost:%d", serverPort)
 req, err := http.NewRequest(http.MethodGet, requestURL, nil)
 if err != nil {
  fmt.Printf("client: could not create request: %s\n", err)
  os.Exit(1)
 }

 res, err := http.DefaultClient.Do(req)
 if err != nil {
  fmt.Printf("client: error making http request: %s\n", err)
  os.Exit(1)
 }

 fmt.Printf("client: got response!\n")
 fmt.Printf("client: status code: %d\n", res.StatusCode)

 resBody, err := ioutil.ReadAll(res.Body)
 if err != nil {
  fmt.Printf("client: could not read response body: %s\n", err)
  os.Exit(1)
 }
 fmt.Printf("client: response body: %s\n", resBody)
}
```

在此更新中，您使用`http.NewRequest`函数生成`http.Request`值，或者在无法创建值时处理错误。但是，与`http.Get`函数不同，`http.NewRequest`函数不会立即向服务器发送HTTP请求。由于它不会立即发送请求，因此您可以在发送请求之前对其进行任何更改。

一旦创建并配置了`http.Request`，就可以使用`Do`的`http.DefaultClient`方法将请求发送到服务器。`http.DefaultClient`值是Go的默认HTTP客户端，与您在`http.Get`中使用的相同。不过，这一次，您直接使用它来告诉它发送您的`http.Request`。HTTP客户端的`Do`方法返回的值与您从`http.Get`函数接收的值相同，因此您可以以相同的方式处理响应。

打印请求结果后`ioutil.ReadAll`是一个[`Body`](https://pkg.go.dev/io#ReadCloser)值，[是`Body`](https://pkg.go.dev/io#Reader)和[`io.ReadCloser`](https://pkg.go.dev/io#Closer)的组合，这意味着你可以使用任何可以从`io.Reader`值读取的东西来读取body`io.Closer`函数很有用，因为它将从`io.Reader`读取数据，直到它到达数据的末尾或遇到`ioutil.ReadAll`。然后，它将返回数据作为一个`io.Reader`值，您可以使用`error`打印，或者它遇到的`[]byte`值。

要运行更新后的程序，请保存更改并使用`go run`命令：

```bash
go run main.go
```

这一次，您的输出应该与之前非常相似，但增加了一个：

```
Output
server: GET /
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```

在第一行中，您可以看到服务器仍然在接收对`GET`路径的`/`请求。客户端也从服务器接收到`200`响应，但它也在阅读并打印服务器响应的`Body`。在一个更复杂的程序中，您可以将从服务器接收的`{"message": "hello!"}`值作为主体，并使用[`encoding/json`](https://pkg.go.dev/encoding/json)包将其作为JSON处理。

在本节中，您创建了一个带有HTTP服务器的程序，您以各种方式向该HTTP服务器发出HTTP请求。首先，您使用`http.Get`函数仅使用服务器的URL向服务器发出`GET`请求。然后，您更新了程序，使用`http.NewRequest`创建了一个`http.Request`值。一旦创建好了，就可以使用Go默认HTTP客户端的`Do`方法`http.DefaultClient`来发出请求，并将`http.Response``Body`打印到输出中。

HTTP协议使用的不仅仅是`GET`请求来在程序之间进行通信。当你想从其他程序接收信息时，`GET`请求很有用，但是当你想从你的程序向服务器发送信息时，可以使用另一种HTTP方法，`POST`方法。

## 发送POST请求

在[REST API](https://en.wikipedia.org/wiki/Representational_state_transfer)中，`GET`请求仅用于从服务器检索信息，因此要使程序完全参与REST API，程序还需要支持发送`POST`请求。`POST`请求几乎是`GET`请求的逆，其中客户端在请求的主体中向服务器发送数据。

在本节中，您将更新您的程序，以将您的请求作为`POST`请求而不是`GET`请求发送。您的`POST`请求将包含一个请求主体，您将更新服务器以打印出有关您从客户端发出的请求的更多信息。

要开始进行这些更新，请打开您的`main.go`文件并添加一些您将在`import`语句中使用的新包：

main.go

```go
...

import (
 "bytes"
 "errors"
 "fmt"
 "io/ioutil"
 "net/http"
 "os"
 "strings"
 "time"
)

...
```

然后，更新你的服务器处理函数，以打印出关于请求的各种信息，比如查询字符串值、头值和请求体：

main.go

```go
...
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   fmt.Printf("server: %s /\n", r.Method)
   fmt.Printf("server: query id: %s\n", r.URL.Query().Get("id"))
   fmt.Printf("server: content-type: %s\n", r.Header.Get("content-type"))
   fmt.Printf("server: headers:\n")
   for headerName, headerValue := range r.Header {
    fmt.Printf("\t%s = %s\n", headerName, strings.Join(headerValue, ", "))
   }

   reqBody, err := ioutil.ReadAll(r.Body)
   if err != nil {
    fmt.Printf("server: could not read request body: %s\n", err)
   }
   fmt.Printf("server: request body: %s\n", reqBody)

   fmt.Fprintf(w, `{"message": "hello!"}`)
  })
...
```

在服务器的HTTP请求处理程序的此更新中，您添加了一些更有用的`fmt.Printf`语句，以查看有关传入请求的信息。使用`r.URL.Query().Get`获取名为`id`的查询字符串值，使用`r.Header.Get`获取名为`content-type`的头的值。您还可以使用一个带有`for`的`r.Header`循环来打印服务器接收到的每个HTTP头的名称和值。如果您的客户端或服务器没有按照您期望的方式运行，则此信息对于故障排除问题非常有用。最后，您还使用了`ioutil.ReadAll`函数来读取`r.Body`中的HTTP请求体。

更新服务器处理函数后，更新`main`函数的请求代码，使其发送带有请求体的`POST`请求：

main.go

```go
...
 time.Sleep(100 * time.Millisecond)
 
 jsonBody := []byte(`{"client_message": "hello, server!"}`)
 bodyReader := bytes.NewReader(jsonBody)

 requestURL := fmt.Sprintf("http://localhost:%d?id=1234", serverPort)
 req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
...
```

在对`main`函数请求的更新中，您定义的新值之一是`jsonBody`值。在这个例子中，值被表示为`[]byte`而不是标准的`string`，因为如果你使用`encoding/json`包来编码JSON数据，它会给你一个给予`[]byte`而不是`string`。

下一个值是`bodyReader`，它是一个[`bytes.Reader`](https://pkg.go.dev/bytes#Reader)，它包装了`jsonBody`数据。一个`http.Request`体要求值是一个`io.Reader`，而`jsonBody`的`[]byte`值不实现`io.Reader`，所以你不能把它本身作为一个请求体。`bytes.Reader`值的存在是为了提供`io.Reader`接口，因此您可以使用`jsonBody`值作为请求体。

`requestURL`值也被更新为包括`id=1234`查询字符串值，主要是为了显示查询字符串值如何也可以与其他标准URL组件沿着包括在请求URL中。

最后，将`http.NewRequest`函数调用更新为使用带有`POST`的`http.MethodPost`方法，并通过将最后一个参数从`nil`主体更新为`bodyReader`（JSON数据`io.Reader`）来包含请求主体。

保存更改后，您可以使用`go run`运行程序：

```bash
go run main.go
```

输出将比以前更长，因为您更新了服务器以显示其他信息：

```
Output
server: POST /
server: query id: 1234
server: content-type: 
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```

来自服务器的第一行显示您的请求现在作为`POST`请求到达`/`路径。第二行显示您添加到请求URL的`1234`查询字符串值的`id`值。第三行显示客户端发送的`Content-Type`头的值，在此请求中恰好为空。

第四行可能与上面的输出略有不同。在Go语言中，当你使用`map`覆盖它们时，`range`值的顺序是不保证的，所以来自`r.Headers`的头可能会以不同的顺序打印出来。根据您使用的Go版本，您可能还会看到与上面版本不同的`User-Agent`版本。

最后，输出中的最后一个变化是服务器显示了它从客户端接收到的请求主体。然后，服务器可以使用`encoding/json`包来解析客户端发送的JSON数据并制定响应。

在本节中，您更新了程序以发送HTTP `POST`请求而不是`GET`请求。您还更新了程序，以发送一个请求主体，其中`[]byte`数据由`bytes.Reader`读取。最后，您更新了服务器处理程序函数，以打印出有关HTTP客户机发出的请求的更多信息。

通常在HTTP请求中，客户端或服务器将在主体中告诉对方它发送的内容类型。但是，正如您在上一个输出中所看到的，您的HTTP请求没有包含`Content-Type`头来告诉服务器如何解释主体的数据。在下一节中，您将进行一些更新以自定义HTTP请求，包括设置`Content-Type`头以让服务器知道您发送的数据类型。

## 自定义HTTP请求

随着时间的推移，HTTP请求和响应已被用于在客户端和服务器之间发送更多种类的数据。在某种程度上，HTTP客户端可以假设他们从HTTP服务器接收的数据是HTML，并且很有可能是正确的。现在，它可以是HTML、JSON、音乐、视频或任何其他数据类型。为了提供有关通过HTTP发送的数据的更多信息，该协议包括HTTP头，其中一个重要的头是`Content-Type`头。这个头告诉服务器（或客户端，取决于数据的方向）如何解释它接收的数据。

在本节中，您将更新程序以在HTTP请求上设置`Content-Type`头，以便服务器知道它正在接收JSON数据。您还将更新您的程序以使用Go默认的`http.DefaultClient`之外的HTTP客户端，以便您可以自定义如何发送请求。

要进行这些更新，请再次打开您的`main.go`文件并更新您的`main`函数，如下所示：

main.go

```go
...

  req, err := http.NewRequest(http.MethodPost, requestURL, bodyReader)
  if err != nil {
   fmt.Printf("client: could not create request: %s\n", err)
   os.Exit(1)
  }
  req.Header.Set("Content-Type", "application/json")

  client := http.Client{
  Timeout: 30 * time.Second,
  }

  res, err := client.Do(req)
  if err != nil {
   fmt.Printf("client: error making http request: %s\n", err)
   os.Exit(1)
  }

...
```

在此更新中，您使用`http.Request`访问`req.Header`头，然后将请求上的`Content-Type`头的值设置为`application/json`。`application/json`媒体类型在[媒体类型](https://www.iana.org/assignments/media-types/media-types.xhtml)列表中定义为JSON的媒体类型。这样，当服务器接收到您的请求时，它知道将主体解释为JSON而不是[XML](https://en.wikipedia.org/wiki/XML)。

下一个更新是在`http.Client`变量中创建您自己的`client`实例。在这个客户机中，您将`Timeout`值设置为30秒。这一点很重要，因为它表明任何与客户端发出的请求都将在30秒后给予并停止尝试接收响应。Go的默认值`http.DefaultClient`没有指定超时，所以如果您使用该客户端发出请求，它将等待，直到它收到响应，被服务器断开连接，或者您的程序结束。如果您有许多这样的请求等待响应，您可能会使用计算机上的大量资源。设置一个`Timeout`值可以限制请求在您定义的时间之前等待的时间。

最后，您更新了请求，以使用`Do`变量的`client`方法。您不需要在这里进行任何其他更改，因为您一直在对`Do`值调用`http.Client`。Go的默认HTTP客户端`http.DefaultClient`只是默认创建的`http.Client`。所以，当你调用`http.Get`时，函数会为你调用`Do`方法，而当你更新请求使用`http.DefaultClient`时，你会直接使用`http.Client`。现在唯一的区别是您创建了这次使用的`http.Client`值。

现在，保存您的文件并使用`go run`运行程序：

```bash
go run main.go
```

您的输出应该与上一个输出非常相似，但包含有关内容类型的更多信息：

```
Output
server: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
        Content-Type = application/json
server: request body: {"client_message": "hello, server!"}
client: got response!
client: status code: 200
client: response body: {"message": "hello!"}
```

您将看到服务器为`content-type`发送了一个值，客户端发送了一个`Content-Type`头。这就是如何使用相同的HTTP请求路径同时为JSON和XML API提供服务。通过指定请求的内容类型，服务器和客户端可以以不同的方式解释数据。

不过，此示例不会触发您配置的客户端超时。要查看当请求时间过长并触发超时时会发生什么，请打开`main.go`文件并向HTTP服务器处理程序函数添加`time.Sleep`函数调用。 然后，使`time.Sleep`持续的时间长于您指定的超时。在本例中，您将设置为35秒：

main.go

```go
...

func main() {
 go func() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
   ... 
   fmt.Fprintf(w, `{"message": "hello!"}`)
   time.Sleep(35 * time.Second)
  })
  ...
 }()
 ...
}
```

现在，保存您的更改并使用`go run`运行程序：

```bash
go run main.go
```

当您这次运行它时，它将需要比以前更长的时间才能退出，因为它直到HTTP请求完成后才退出。由于您添加了`time.Sleep(35 * time.Second)`，HTTP请求将在达到30秒超时之前完成：

```
Output
server: POST /
server: query id: 1234
server: content-type: application/json
server: headers:
        Content-Type = application/json
        Accept-Encoding = gzip
        User-Agent = Go-http-client/1.1
        Content-Length = 36
server: request body: {"client_message": "hello, server!"}
client: error making http request: Post "http://localhost:3333?id=1234": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
exit status 1
```

在这个程序的输出中，您可以看到服务器收到了请求并进行了处理，但是当它到达HTTP处理程序函数的末尾时，即您的`time.Sleep`函数调用所在的地方，它开始休眠35秒。同时，HTTP请求的超时正在倒计时，并在HTTP请求完成之前达到30秒的限制。这会导致`client.Do`方法调用失败，并出现`context deadline exceeded`错误，因为请求的30秒期限已过。然后，您的程序退出，并使用`1`返回失败状态代码`os.Exit(1)`。

在本节中，您更新了程序，通过向其添加`Content-Type`头来自定义HTTP请求。您还更新了程序，以创建一个具有30秒超时的新`http.Client`，然后使用该客户端发出HTTP请求。您还通过向HTTP请求处理程序添加`time.Sleep`来测试30秒超时。最后，您还看到了为什么如果您想避免许多请求可能永远处于空闲状态，那么使用您自己的带有超时设置的`http.Client`值很重要。

## 结论

在本教程中，您创建了一个带有HTTP服务器的新程序，并使用Go的`net/http`包向该服务器发出HTTP请求。首先，您使用`http.Get`函数向Go默认HTTP客户端的服务器发出`GET`请求。然后，您使用`http.NewRequest`和`http.DefaultClient`的`Do`方法来发出`GET`请求。接下来，您更新了您的请求，使其成为带有使用`POST`的主体的`bytes.NewReader`请求。最后，您在`Set`的`http.Request`字段上使用了`Header`方法来设置请求的`Content-Type`头，并通过创建自己的HTTP客户端而不是使用Go的默认客户端来设置请求持续时间的30秒超时。

[`net/http`](https://pkg.go.dev/net/http)包包含的不仅仅是您在本教程中使用的功能。它还包括一个[`http.Post`](https://pkg.go.dev/net/http#Post)函数，可以用来发出`POST`请求，类似于`http.Get`函数。该软件包还支持保存和检索[cookie](https://pkg.go.dev/net/http#CookieJar)等功能。
