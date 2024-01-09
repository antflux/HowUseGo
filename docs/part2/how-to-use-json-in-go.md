# 如何在 Go 中使用 JSON

### 介绍

在现代程序中，程序之间的通信非常重要。无论然而，许多编程语言都有自己的内部存储数据的方式，其他语言无法理解。为了允许这些语言交互，需要将数据转换为它们都能理解的通用格式。其中一种格式[JSON]是通过互联网以及同一系统中的程序之间传输数据的流行方式。

许多现代编程语言都在其标准库中提供了一种将数据转换为JSON的方法，Go也是如此。通过使用Go提供的[`encoding/json`](https://pkg.go.dev/encoding/json)包，您的Go程序还可以与任何其他可以使用JSON进行通信的系统进行交互。

在本教程中，您将首先创建一个程序，该程序使用`encoding/json`包将数据从`map`编码为JSON数据，然后更新您的程序以使用`struct`类型来编码数据。之后，您将更新程序，将JSON数据解码为`map`，然后最终将JSON数据解码为`struct`类型。

## 先决条件

要遵循本教程，您需要：

- Go 1.16或更高版本已安装。要设置此设置，请按照[如何]为您的操作系统安装Go教程。
- 熟悉JSON，您可以在[JSON简介]中找到。
- 使用Go结构体标记自定义`struct`类型的字段的能力。更多信息可以在[如何在Go中使用结构标记]中找到。
- 或者，了解如何在Go中创建日期和时间值。你可以在[如何在Go中使用日期和时间中]阅读更多。

## 使用Map生成JSON

Go语言对JSON编码和解码的支持由标准库的`encoding/json`包提供。您将从该包中使用的第一个函数[编组](https://en.wikipedia.org/wiki/Marshalling_(computer_science))，有时也称为[序列化](https://en.wikipedia.org/wiki/Serialization)，是将内存中的程序数据转换为可以在其他地方传输或保存的格式的过程。`json.Marshal`函数用于将Go数据转换为JSON数据。`json.Marshal`函数接受`json.Marshal`类型作为要封送到JSON的值，因此允许任何值作为参数传入，并将返回JSON数据作为结果。在本节中，您将使用`interface{}`函数创建一个程序，以从Go `json.Marshal`值生成包含各种类型数据的JSON，然后将这些值打印到输出中。

大多数JSON都表示为对象，有`string`键和各种其他类型的值。因此，在Go中生成JSON数据最灵活的方法是使用`map`键和`string`值将数据放入`interface{}`中。`string`键可以直接转换为JSON对象键，而`interface{}`值允许该值为任何其他值，无论是`string`、`int`，甚至是另一个`map[string]interface{}`。

要开始在程序中使用`encoding/json`包，您需要有一个程序目录。在本教程中，您将使用名为`projects`的目录。

首先，创建`projects`目录并导航到它：

```bash
mkdir projects
cd projects
```


接下来，为您的项目创建目录。在这种情况下，使用目录`contexts`：

```bash
mkdir jsondata
cd jsondata
```


在`contexts`目录中，使用`nano`或您最喜欢的编辑器打开`main.go`文件：

```bash
nano main.go
```


在`main.go`文件中，您将添加一个`main`函数来运行程序。接下来，您将添加一个带有各种键和数据类型的`map[string]interface{}`值。然后，您将使用`json.Marshal`函数将`map`数据封送到JSON数据中。

在`main.go`中添加以下行：

main.go

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
	}

	jsonData, err := json.Marshal(data)
	if err != nil {
		fmt.Printf("could not marshal json: %s\n", err)
		return
	}

	fmt.Printf("json data: %s\n", jsonData)
}
```


在`data`变量中，每个值都有一个`string`作为键，但这些键的值各不相同。一个是`int`值，另一个是`bool`值，还有一个甚至是另一个`map[string]interface{}`值，里面有一个`[]int`值。

当您将`data`变量传递给`json.Marshal`时，该函数将检查您提供的所有值，并确定它们是哪种类型以及如何在JSON中表示它们。如果在翻译中有任何问题，`json.Marshal`函数将返回一个描述问题的`error`。如果转换成功，`jsonData`变量将包含`[]byte`的JSON数据。由于可以使用`[]byte`或格式字符串中的`string`[动词将](https://pkg.go.dev/fmt)`myString := string(jsonData)`值转换为`%s`值，因此可以使用`fmt.Printf`将JSON数据打印到屏幕。

保存并关闭文件。

要查看程序的输出，请使用`go run`命令并提供`main.go`文件：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json data: {"boolValue":true,"intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

在输出中，您将看到顶级JSON值是一个由花括号（`{}`）表示的对象。您在`data`中包含的所有值都存在。您还将看到`objectValue`的`map[string]interface{}`被转换为另一个JSON对象，由`{}`包围，并且还包括`arrayValue`，其中包含数组值`[1,2,3,4]`。

### JSON中的编码时间

`encoding/json`包不仅支持`string`和`int`值类型。它还可以编码更复杂的类型。它支持的更复杂的类型之一是来自[`time.Time`](https://pkg.go.dev/time)包的`time`类型。

注意：有关Go的`time`包的更多信息

要查看此操作，请再次打开`main.go`文件，并使用`time.Time`函数向数据添加`time.Date`值：

main.go

```go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
		"dateValue":   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
	}

	...
}
```


此更新将把`March 2, 2022,`时区中的日期`9:10:00 AM`和时间`UTC`分配给`dateValue`键。

保存更改后，使用与之前相同的`go run`命令再次运行程序：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json data: {"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

这一次在输出中，您

### 在JSON中编码`null`值

根据程序与之交互的系统，您可能需要在JSON数据中发送`null`值，Go的`encoding/json`包也可以为您处理。使用`map`，只需添加带有`string`值的新`nil`键即可。

要在JSON输出中添加几个`null`值，请再次打开`main.go`文件并添加以下行：

main.go

```go
...

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
                "dateValue":   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
		"nullStringValue": nil,
		"nullIntValue":    nil,
	}

	...
}
```


你添加到数据中的值有一个键，它说它是一个`string`值或一个`int`值，但实际上代码中没有任何东西使它成为这两个值。因为`map`有`interface{}`值，所以代码只知道`interface{}`值是`nil`。由于您只是使用这个`map`将Go数据转换为JSON数据，因此此时的区别并没有什么不同。

将更改保存到`main.go`后，使用`go run`运行程序：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json data: {"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"nullIntValue":null,"nullStringValue":null,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

现在，在输出中，您将看到`nullIntValue`和`nullStringValue`字段包含在JSON `null`值中。这样，您仍然可以使用`map[string]interface{}`值将Go数据转换为具有预期字段的JSON数据。

在本节中，您创建了一个可以将`map[string]interface{}`值封送到JSON数据中的程序。然后，您向数据添加了一个`time.Time`字段，并包含了一对`null`-value字段。

虽然使用`map[string]interface{}`来编组JSON数据可以非常灵活，但如果您需要在多个地方发送相同的数据，它也会变得很麻烦。如果将此数据复制到代码中的多个位置，则很容易意外地输入错误的字段名称，或将不正确的数据分配给字段。在这种情况下，使用`struct`类型来表示要转换为JSON的数据可能是有益的。

## 使用Struct生成JSON

使用像Go这样的[静态类型](https://en.wikipedia.org/wiki/Type_system#Static_type_checking)语言的好处之一是，你可以使用这些类型来让编译器检查或强制程序中的一致性。Go语言的`encoding/json`包允许您通过定义一个`struct`类型来表示JSON数据来利用这一点。您可以使用[struct标记]控制如何转换`struct`中包含的数据。在本节中，您将更新程序以使用`struct`类型而不是`map`类型来生成JSON数据。

当您使用`struct`来定义JSON数据时，必须导出您期望被翻译的字段名称（而不是`struct`类型名称本身），这意味着它们必须以大写字母开头，例如`IntValue`，否则`encoding/json`包将无法访问这些字段以将其翻译为JSON。如果你不使用struct标签来控制这些字段的命名，字段名将被直接转换为`struct`上的名称。使用默认名称可能是您希望在JSON数据中使用的名称，这取决于您希望如何形成数据。如果是这样的话，你就不需要添加任何struct标签。然而，许多JSON消费者使用诸如`intValue`或`int_value`之类的名称格式作为其字段名称，因此添加这些struct标记将允许您控制转换的发生方式。

例如，假设你有一个名为`struct`的`IntValue`字段，你将其编组为JSON：

```go
type myInt struct {
	IntValue int
}

data := &myInt{IntValue: 1234}
```


如果使用`data`函数将`json.Marshal`变量编组为JSON，则最终将得到以下值：

```json
{"IntValue":1234}
```


但是，如果JSON消费者希望字段名为`intValue`而不是`IntValue`，则需要一种方法来告诉`encoding/json`。由于`json.Marshal`不知道您希望在JSON数据中命名字段，因此您将通过向字段添加struct标记来告诉它。通过向`json`字段添加一个值为`IntValue`的`intValue` struct标签，您可以告诉`json.Marshal`在生成JSON数据时应该使用名称`intValue`：

```go
type myInt struct {
	IntValue int `json:"intValue"`
}

data := &myInt{IntValue: 1234}
```


这一次，如果您将`data`变量封送为JSON，则`json.Marshal`函数将看到`json` struct标记并知道命名字段`intValue`，因此您将获得预期结果：

```json
{"intValue":1234}
```


现在，您将更新程序以使用JSON数据的`struct`值。您将添加一个`myJSON``struct`类型来定义顶级JSON对象，并添加一个`myObject``struct`类型来为`ObjectValue`字段定义内部JSON对象。您还将向每个字段添加`json` struct标记，以告诉`json.Marshal`如何在JSON数据中命名它们。您还需要更新`data`变量赋值以使用您的`myJSON`结构体，声明它类似于任何其他Go `struct`。

打开`main.go`文件并进行以下更改：

main.go

```go
...

type myJSON struct {
	IntValue        int       `json:"intValue"`
	BoolValue       bool      `json:"boolValue"`
	StringValue     string    `json:"stringValue"`
	DateValue       time.Time `json:"dateValue"`
	ObjectValue     *myObject `json:"objectValue"`
	NullStringValue *string   `json:"nullStringValue"`
	NullIntValue    *int      `json:"nullIntValue"`
}

type myObject struct {
	ArrayValue []int `json:"arrayValue"`
}

func main() {
	otherInt := 4321
	data := &myJSON{
		IntValue:    1234,
		BoolValue:   true,
		StringValue: "hello!",
		DateValue:   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		ObjectValue: &myObject{
			ArrayValue: []int{1, 2, 3, 4},
		},
		NullStringValue: nil,
		NullIntValue:    &otherInt,
	}

	...
}
```


其中许多更改与前面的`IntValue`字段名称示例类似，但其中一些更改值得特别指出。其中之一，`ObjectValue`字段，使用引用类型`*myObject`来告诉JSON编组器期望引用`myObject`值或`nil`值。这就是如何定义一个JSON对象，它是多层自定义对象的深度。如果你的JSON数据需要它，你也可以在`struct`类型中引用另一个`myObject`类型，依此类推。使用这种模式，你可以使用Go `struct`类型来描述非常复杂的JSON对象。

在上面的代码中要查看的另一对字段是`NullStringValue`和`NullIntValue`。与`StringValue`和`IntValue`不同，这些值的类型是引用类型`*string`和`*int`。默认情况下，`string`和`int`类型的值不能为`nil`，因为它们的“空”值为`""`和`0`。因此，如果你想表示一个字段，它可以是一种类型或`nil`，你需要使它成为一个引用。例如，假设您有一个用户调查问卷，您希望能够表示用户是否选择不回答问题（`null`值），或者用户没有回答问题（`""`值）。

这段代码还更新了`NullIntValue`字段，为它分配了一个值`4321`，以显示如何为引用类型（如`*int`）分配值。在Go语言中，你只能使用变量创建对[基本类型的](https://en.wikipedia.org/wiki/Primitive_data_type)引用，比如`int`和`string`。因此，为了给`NullIntValue`字段赋值，首先要给另一个变量`otherInt`赋值，然后使用`&otherInt`获取对它的引用（而不是直接使用`&4321`）。

保存更新后，使用`go run`运行程序：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json data: {"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullStringValue":null,"nullIntValue":4321}
```

您将看到此输出与使用`map[string]interface{}`值时相同，但这次`nullIntValue`的值为`4321`，因为这是`otherInt`的值。

最初，设置`struct`值可能需要一些额外的时间，但是一旦定义了它们，就可以在代码中反复使用它们，无论在哪里使用，结果都是相同的。您也可以在一个地方更新它们，而不是试图找到可能使用`map`的每个地方。

Go的JSON marshaller还允许您根据值是否为空来控制是否应将字段包含在JSON输出中。有时候，你可能有一个大的JSON对象或可选字段，你不想总是包括在内，所以省略这些字段可能是有用的。通过`omitempty`结构体标记中的`json`选项来控制字段是否在为空时被省略。

现在，更新您的程序，使`NullStringValue`字段`omitempty`并添加一个名为`EmptyString`的新字段，并使用相同的选项：

main.go

```go
...

type myJSON struct {
	...
	
	NullStringValue *string   `json:"nullStringValue,omitempty"`
	NullIntValue    *int      `json:"nullIntValue"`
	EmptyString     string    `json:"emptyString,omitempty"`
}

...
```


现在，当`myJSON`被编组时，如果它们的值为空，则`EmptyString`和`NullStringValue`字段都将从输出中排除。

保存更改后，使用`go run`运行程序：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json data: {"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullIntValue":4321}
```

这一次在输出中，您将看到`nullStringValue`字段不再出现。由于它的值为`nil`，因此被认为是空的，所以`omitempty`选项将其从输出中排除。您还将看到新的`emptyString`字段也不包含在内。即使`emptyString`值不是`nil`，字符串的默认值`""`也被认为是空的，因此也被排除在外。

在本节中，您更新了您的程序，以使用`struct`类型来生成带有`json.Marshal`而不是`map`类型的JSON数据。您还更新了程序，以便从JSON输出中省略空字段。

但是，为了让你的程序很好地适应JSON生态系统，你需要做的不仅仅是生成JSON数据。您还需要能够读取响应您的请求发送的JSON数据，或其他系统向您发送请求。`encoding/json`包还提供了一种将JSON数据解码为各种Go类型的方法。在下一节中，您将更新程序，将JSON字符串解码为Go `map`类型。

## 使用Map解析JSON

类似于本教程的第一部分，您使用`map[string]interface{}`作为灵活的方式来生成JSON数据，您也可以使用它作为灵活的方式来读取JSON数据。`json.Unmarshal`函数，本质上与`json.Marshal`函数相反，将获取JSON数据并将其转换回Go数据。您为`json.Unmarshal`提供JSON数据以及Go变量，以将解组的数据放入其中，如果无法完成，则返回`error`值，如果成功，则返回`nil`错误值。在本节中，您将更新您的程序，以使用`json.Unmarshal`函数将JSON数据从预定义的`string`值读取到`map`变量中。您还将更新程序以将Go数据打印到输出。

现在，更新您的程序以使用`json.Unmarshal`将JSON数据解组到`map[string]interface{}`。首先，将原始的`data`变量替换为包含JSON字符串的`jsonData`变量。然后将一个新的`data`变量声明为`map[string]interface{}`变量，以接收JSON数据。最后，您将使用`json.Unmarshal`和这些变量来访问JSON数据。

打开你的`main.go`文件，用下面的代码替换你的`main`函数中的代码行：

main.go

```go
...

func main() {
	jsonData := `
		{
			"intValue":1234,
			"boolValue":true,
			"stringValue":"hello!",
			"dateValue":"2022-03-02T09:10:00Z",
			"objectValue":{
				"arrayValue":[1,2,3,4]
			},
			"nullStringValue":null,
			"nullIntValue":null
		}
	`

	var data map[string]interface{}
	err := json.Unmarshal([]byte(jsonData), &data)
	if err != nil {
		fmt.Printf("could not unmarshal json: %s\n", err)
		return
	}

	fmt.Printf("json map: %v\n", data)
}
```


在此更新中，使用[原始字符串文字](https://go.dev/ref/spec#String_literals)设置`jsonData`变量，以允许声明跨越多行，以便于阅读。在将`data`声明为`map[string]interface{}`之后，将`jsonData`和`data`传递给`json.Unmarshal`以将JSON数据解组到`data`变量中。

`jsonData`变量作为`json.Unmarshal`传递给`[]byte`，因为函数需要`[]byte`类型，而`jsonData`最初定义为`string`类型。这是因为Go中的`string`可以转换为`[]byte`，反之亦然。`data`变量被作为引用传递，因为为了让`json.Unmarshal`将数据放入变量，它需要引用变量在内存中的存储位置。

最后，一旦JSON数据被解组到`data`变量中，就可以使用`fmt.Printf`将其打印到屏幕上。

要运行更新后的程序，请保存更改并使用`go run`运行程序：

```bash
go run main.go
```


输出将类似于以下内容：

```
Output
json map: map[boolValue:true dateValue:2022-03-02T09:10:00Z intValue:1234 nullIntValue:<nil> nullStringValue:<nil> objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]
```

这一次，您的输出显示了JSON翻译的Go端。您有一个`map`值，其中包含JSON数据中的各种字段。您将看到，即使是JSON数据中的`null`字段也会显示在映射中。

现在，因为你的Go数据在`map[string]interface{}`中，所以需要做一些工作来使用这些数据。您需要使用所需的`map`键值从`string`中获取值，然后您需要确保收到的值是您所期望的值，因为它将作为`interface{}`值返回给您。

为此，请打开`main.go`文件并更新程序，以使用以下代码读取`dateValue`字段：

main.go

```go
...

func main() {
	...
	
	fmt.Printf("json map: %v\n", data)

	rawDateValue, ok := data["dateValue"]
	if !ok {
		fmt.Printf("dateValue does not exist\n")
		return
	}
	dateValue, ok := rawDateValue.(string)
	if !ok {
		fmt.Printf("dateValue is not a string\n")
		return
	}
	fmt.Printf("date value: %s\n", dateValue)
}
```


在此更新中，您使用`data["dateValue"]`将`rawDateValue`作为`interface{}`类型获取，并使用`ok`变量确保`dateValue`字段位于`map`中。

然后，您使用[类型断言](https://go.dev/tour/methods/15)来断言`rawDateValue`的类型实际上是`string`值，并将其分配给变量`dateValue`。之后，再次使用`ok`变量来确保断言成功。

最后，使用`fmt.Printf`打印`dateValue`。

要再次运行更新后的MySQL，请保存更改并使用`go run`运行它：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json map: map[boolValue:true dateValue:2022-03-02T09:10:00Z intValue:1234 nullIntValue:<nil> nullStringValue:<nil> objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]
date value: 2022-03-02T09:10:00Z
```

您可以看到`date value`行显示了从`dateValue`提取并转换为`map`值的`string`字段。

在本节中，您更新了程序，使用带有`json.Unmarshal`变量的`map[string]interface{}`函数将JSON数据解组为Go数据。然后，您更新了程序，以便从Go数据中提取`dateValue`的值并将其打印到屏幕上。

然而，这个更新确实显示了在Go中使用`map[string]interface{}`来解组JSON的缺点之一。由于Go不知道每个字段是哪种类型的数据（它唯一知道的是它是`interface{}`），所以它能做的最好的解组数据是做出最佳猜测。这意味着像`time.Time`字段的`dateValue`这样的复杂值不能为您解组，只能作为`string`访问。如果您尝试以`map`这种方式访问任何数字值，则会发生类似的问题。由于`json.Unmarshal`不知道这个数字应该是`int`、`float`、`int64`等等，所以它能做出的最好的猜测是将它放入最灵活的数字类型中，即`float64`。

虽然使用`map`来解码JSON数据可以很灵活，但在解释数据时，它也为您留下了更多的工作。类似于`json.Marshal`函数可以使用`struct`值来生成JSON数据，`json.Unmarshal`函数可以使用`struct`值来读取JSON数据。这可以通过使用`map`字段上的类型定义来确定JSON数据应该被解释为哪些类型，从而帮助消除使用`struct`的类型断言复杂性。在下一节中，您将更新程序以使用`struct`类型来消除这些复杂性。

## 使用结构解析JSON

当您在阅读JSON数据时，您很可能已经知道您正在接收的数据的结构;否则，将很难解释。您可以使用这种结构知识来给予Go一些提示，让它了解您的数据是什么样的以及您期望的数据类型。

在上一节中，您定义了`myJSON`和`myObject` `struct`值，并添加了`json` struct标记，以让Go知道如何在生成JSON时命名字段。现在，您可以使用这些相同的`struct`值来解码您一直使用的JSON字符串，如果您正在编组和解组相同的JSON数据，这将有助于减少程序中的重复代码。使用`struct`解组JSON数据的另一个好处是，你可以告诉Go每个字段所期望的数据类型。最后，你还可以使用Go的编译器来检查你是否在字段上使用了正确的名称，而不是在你使用的`string`值中可能遗漏了`map`值中的一个错字。

现在，打开`main.go`文件，更新`data`变量声明，使用对`myJSON` `struct`的引用，并添加几行`fmt.Printf`来显示`myJSON`上各个字段的数据：

main.go

```go
...

func main() {
	...
	
	var data *myJSON
	err := json.Unmarshal([]byte(jsonData), &data)
	if err != nil {
		fmt.Printf("could not unmarshal json: %s\n", err)
		return
	}

	fmt.Printf("json struct: %#v\n", data)
	fmt.Printf("dateValue: %#v\n", data.DateValue)
	fmt.Printf("objectValue: %#v\n", data.ObjectValue)
}
```


由于您之前定义了`struct`类型，因此您只需要更新`data`字段的类型以支持对`struct`的解组。其余的更新显示了`struct`本身的一些数据。

现在，保存您的更新并使用`go run`运行程序：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json struct: &main.myJSON{IntValue:1234, BoolValue:true, StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC), ObjectValue:(*main.myObject)(0x1400011c180), NullStringValue:(*string)(nil), NullIntValue:(*int)(nil), EmptyString:""}
dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)
objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}
```

在这次的输出中有几件事需要注意。您将在`json struct`行和`dateValue`行中看到，JSON数据中的日期值现在已转换为`time.Time`值（当使用`time.Date`作为格式动词时，将显示`%#v`格式）。由于Go语言能够在`time.Time`的`myJSON`字段上看到`DateValue`类型，因此它也能够为您解析`string`值。

另一件需要注意的事情是，`EmptyString`出现在`json struct`行，即使它没有包含在原始JSON数据中。如果一个字段包含在用于JSON解组的`struct`中，而没有包含在正在解组的JSON数据中，则该字段将被设置为其类型的默认值并被忽略。通过这种方式，您可以安全地定义JSON数据可能具有的所有可能字段，而不必担心在过程的任何一端都不存在字段时会出现错误。`NullStringValue`和`NullIntValue`也被设置为默认值`nil`，因为JSON数据表示它们的值是`null`，但如果这些字段已从JSON数据中排除，它们也将被设置为`nil`。

类似于当JSON数据中缺少`EmptyString`字段时，`struct`上的`json.Unmarshal`字段被`emptyString`忽略，相反的情况也是如此。如果一个字段包含在JSON数据中，但在Go `struct`上没有对应的字段，则该JSON字段将被忽略，并继续解析下一个JSON字段。这样，如果您正在阅读的JSON数据非常大，而您的程序只关心其中的一小部分字段，则可以选择创建一个仅包含您关心的字段的`struct`。JSON数据中包含的任何没有在`struct`上定义的字段都会被忽略，Go的JSON解析器将继续处理下一个字段。

要查看实际效果，请最后一次打开`main.go`文件，并更新`jsonData`以包含`myJSON`中未包含的字段：

main.go

```go
...

func main() {
	jsonData := `
		{
			"intValue":1234,
			"boolValue":true,
			"stringValue":"hello!",
			"dateValue":"2022-03-02T09:10:00Z",
			"objectValue":{
				"arrayValue":[1,2,3,4]
			},
			"nullStringValue":null,
			"nullIntValue":null,
			"extraValue":4321
		}
	`

	...
}
```


添加JSON数据后，保存文件并使用`go run`运行它：

```bash
go run main.go
```


您的输出将类似于以下内容：

```
Output
json struct: &main.myJSON{IntValue:1234, BoolValue:true, StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC), ObjectValue:(*main.myObject)(0x14000126180), NullStringValue:(*string)(nil), NullIntValue:(*int)(nil), EmptyString:""}
dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)
objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}
```

您应该不会看到此输出与前一个输出之间有任何差异，因为Go将忽略JSON数据中的`extraValue`字段并继续。

在本节中，您更新了程序，以使用之前定义的`struct`类型来解组JSON数据。你已经看到Go语言如何解析一个`time.Time`值，并忽略定义在`EmptyString`类型上而不是JSON数据中的`struct`字段。您还向JSON数据添加了一个额外的字段，以确保Go可以安全地继续解析数据，即使您只在JSON数据中定义了字段的子集。

## 结论

在本教程中，您创建了一个新程序来使用Go标准库中的`encoding/json`包。首先，您使用带有`json.Marshal`类型的`map[string]interface{}`函数以灵活的方式创建JSON数据。然后，您更新了您的程序，使用`struct`类型和`json`结构标记，以使用`json.Marshal`以一致和可靠的方式生成JSON数据。之后，您使用带有`json.Unmarshal`类型的`map[string]interface{}`函数将JSON字符串解码为Go数据。最后，您使用了之前用`struct`函数定义的`json.Unmarshal`类型，让Go根据这些`struct`字段为您进行解析和类型转换。

使用`encoding/json`包，您将能够与互联网上可用的许多API进行交互，以创建您自己的与流行网站的集成。你还可以将自己程序中的Go数据转换成一种可以保存的格式，然后在以后加载，从程序停止的地方继续。

除了您在本教程中使用的函数之外，[`encoding/json`](https://pkg.go.dev/encoding/json)包还包括其他有用的函数和类型，可用于与JSON进行交互。例如，可以使用[`json.MarshalIndent`](https://pkg.go.dev/encoding/json#MarshalIndent)函数来[漂亮地打印](https://en.wikipedia.org/wiki/Prettyprint)JSON数据以进行故障排除。