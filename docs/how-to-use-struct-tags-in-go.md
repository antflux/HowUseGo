# 如何在 Go 中使用结构标记

## 介绍

结构（structures）用于将多条信息收集在一个单元中。这些[信息集合]用于描述更高级别的概念，例如由`Address`、`Street`、`City`和`State`组成的`PostalCode`。当你从数据库或API等系统中读取这些信息时，你可以使用struct标签来控制如何将这些信息分配给结构体的字段。Struct标签是附加到结构体字段的一小块元数据，为其他使用该结构体的Go代码提供指令。

## Struct Tag看起来像什么？

Go结构体标记是出现在Go结构体声明中的类型之后的注释。每个标记都由与某个相应值相关联的短字符串组成。

一个struct标签看起来像这样，标签偏移量带有反勾号```字符：

```go
type User struct {
	Name string `example:"name"`
}
```


其他Go代码可以检查这些结构，并提取分配给它请求的特定键的值。如果没有其他代码检查结构标记，则结构标记对代码的操作没有任何影响。

试试这个例子，看看结构体标签是什么样子的，如果没有来自另一个包的代码，它们将没有任何效果。

```go
package main

import "fmt"

type User struct {
	Name string `example:"name"`
}

func (u *User) String() string {
	return fmt.Sprintf("Hi! My name is %s", u.Name)
}

func main() {
	u := &User{
		Name: "Sammy",
	}

	fmt.Println(u)
}
```


这将输出：

```
Output
Hi! My name is Sammy
```

这个例子定义了一个带有`User`字段的`Name`类型。`Name`字段被赋予了一个struct标签`example:"name"`。我们将会话中的这个特定标记称为“example struct标记”，因为它使用单词“example”作为其键。`example` struct标记的`"name"`字段的值为`Name`。在`User`类型上，我们还定义了`String()`接口所需的`fmt.Stringer`方法。当我们将类型传递给`fmt.Println`时，它将自动调用，并给我们一个机会来生成一个格式良好的结构版本。

在`main`的主体中，我们创建了`User`类型的一个新实例，并将其传递给`fmt.Println`。即使结构体有一个结构体标记，我们看到它对这段Go代码的操作没有影响。如果不存在struct标记，它的行为将完全相同。

要使用struct标签来完成一些事情，必须编写其他Go代码来在运行时检查结构。标准库中的包使用结构体标记作为其操作的一部分。其中最受欢迎的是`encoding/json`包。

## JSON编码

JavaScript Object Notation（JSON）是一种文本格式，用于对在不同字符串键下组织的数据集合进行编码。它通常用于不同程序之间的数据通信，因为格式足够简单，可以用许多不同的语言对其进行解码。下面是一个JSON的例子：

```
{
  "language": "Go",
  "mascot": "Gopher"
}
```

这个JSON对象包含两个键，`language`和`mascot`。这些键后面是相关的值。这里，`language`键的值为`Go`，`mascot`键的值为`Gopher`。

标准库中的JSON编码器使用struct标记作为注释，向编码器指示您希望如何在JSON输出中命名字段。这些JSON编码和解码机制可以在`encoding/json`[包](https://pkg.go.dev/encoding/json)中找到。

试试这个例子，看看JSON是如何在没有struct标签的情况下编码的：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name          string
	Password      string
	PreferredFish []string
	CreatedAt     time.Time
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```


这将打印以下输出：

```
Output
{
  "Name": "Sammy the Shark",
  "Password": "fisharegreat",
  "CreatedAt": "2019-09-23T15:50:01.203059-04:00"
}
```

我们定义了一个描述用户的结构体，该结构体的字段包括用户名、密码和创建用户的时间。在`main`函数中，我们通过为除`PreferredFish`（Sammy likes all fish）之外的所有字段提供值来创建此用户的实例。然后我们将`User`的实例传递给`json.MarshalIndent`函数。这样我们就可以更容易地看到JSON输出，而无需使用外部格式化工具。这个调用可以替换为`json.Marshal(u)`来打印JSON，而不需要任何额外的空格。`json.MarshalIndent`的两个附加参数控制输出的前缀（我们在空字符串中省略了它）和用于缩进的字符，这里是两个空格字符。从`json.MarshalIndent`产生的任何错误都被记录下来，程序使用`os.Exit(1)`终止。最后，我们将从`[]byte`返回的`json.MarshalIndent`转换为`string`，并将结果字符串传递给`fmt.Println`，以便在终端上打印。

结构的字段与命名的字段完全相同。不过，这并不是您所期望的典型JSON样式，它使用骆驼大小写作为字段名称。在下一个示例中，您将更改字段的名称，使其遵循骆驼大小写样式。正如你在运行这个例子时所看到的，这是行不通的，因为所需的字段名与Go语言关于导出字段名的规则冲突。

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	name          string
	password      string
	preferredFish []string
	createdAt     time.Time
}

func main() {
	u := &User{
		name:      "Sammy the Shark",
		password:  "fisharegreat",
		createdAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```


这将显示以下输出：

```
Output
{}
```

在这个版本中，我们已经将字段的名称更改为骆驼大小写。现在`Name`是`name`，`Password`是`password`，最后`CreatedAt`是`createdAt`。在`main`的主体中，我们已经更改了结构的实例化以使用这些新名称。然后我们像以前一样将结构体传递给`json.MarshalIndent`函数。这次的输出是一个空的JSON对象`{}`。

Camel大小写字段要求第一个字符必须小写。虽然JSON不关心你如何命名字段，但Go关心，因为它指示字段在包外的可见性。由于`encoding/json`包是一个独立于我们使用的`main`包的包，所以我们必须将第一个字符替换为字符串，以使其对`encoding/json`可见。我们似乎陷入了僵局。我们需要某种方式来向JSON编码器传达我们希望该字段被命名的内容。

### 使用结构标记控制编码

您可以修改前面的示例，通过使用struct标记注释每个字段，使导出的字段使用大小写混合的字段名正确编码。`encoding/json`识别的struct标记有一个键`json`和一个控制输出的值。通过将字段名称的骆驼式大小写版本作为`json`键的值，编码器将使用该名称。此示例修复了前两次尝试：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name          string    `json:"name"`
	Password      string    `json:"password"`
	PreferredFish []string  `json:"preferredFish"`
	CreatedAt     time.Time `json:"createdAt"`
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```


这将输出：

```
Output
{
  "name": "Sammy the Shark",
  "password": "fisharegreat",
  "preferredFish": null,
  "createdAt": "2019-09-23T18:16:17.57739-04:00"
}
```

我们通过将结构体名称的首字母大写，将结构体字段改回对其他包可见。然而，这一次我们以`json:"name"`的形式添加了struct标记，其中`"name"`是我们希望`json.MarshalIndent`在将结构体打印为JSON时使用的名称。

我们现在已经成功地正确格式化了JSON。但是，请注意，即使我们没有设置某些值，也会打印这些值的字段。如果你愿意，JSON编码器也可以消除这些字段。

### 删除空JSON字段

在JSON中，通常会禁止输出未设置的字段。由于Go语言中的所有类型都有一个“零值”，即它们被设置为的某个默认值，所以`encoding/json`包需要额外的信息来判断当它假设这个零值时，某些字段应该被视为未设置。在任何`json` struct标签的value部分中，您可以使用`,omitempty`后缀字段的所需名称，以告知JSON编码器在字段设置为零值时抑制此字段的输出。下面的示例修复了前面的示例，使其不再输出空字段：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name          string    `json:"name"`
	Password      string    `json:"password"`
	PreferredFish []string  `json:"preferredFish,omitempty"`
	CreatedAt     time.Time `json:"createdAt"`
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```


此示例将输出：

```
Output
{
  "name": "Sammy the Shark",
  "password": "fisharegreat",
  "createdAt": "2019-09-23T18:21:53.863846-04:00"
}
```

我们修改了前面的示例，使`PreferredFish`字段现在具有struct标记`json:"preferredFish,omitempty"`。`,omitempty`增强的存在导致JSON编码器跳过该字段，因为我们决定不设置它。在我们前面的例子的输出中，它的值是`null`。

这个输出看起来好多了，但是我们仍然在打印用户的密码。`encoding/json`包为我们提供了另一种完全忽略私有字段的方法。

### 忽略私有字段

某些字段必须从结构导出，以便其他包可以正确地与该类型交互。然而，这些字段的性质可能是敏感的，因此在这些情况下，我们希望JSON编码器完全忽略该字段-即使它被设置。这是使用特殊值`-`作为`json:`结构标记的值参数来完成的。

此示例修复了暴露用户密码的问题。

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"time"
)

type User struct {
	Name      string    `json:"name"`
	Password  string    `json:"-"`
	CreatedAt time.Time `json:"createdAt"`
}

func main() {
	u := &User{
		Name:      "Sammy the Shark",
		Password:  "fisharegreat",
		CreatedAt: time.Now(),
	}

	out, err := json.MarshalIndent(u, "", "  ")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	fmt.Println(string(out))
}
```


运行此示例时，您将看到以下输出：

```
Output
{
  "name": "Sammy the Shark",
  "createdAt": "2019-09-23T16:08:21.124481-04:00"
}
```

与之前的例子相比，我们在这个例子中唯一改变的是密码字段现在使用特殊的`"-"`值作为其`json:`结构标记。在此示例的输出中，`password`字段不再存在。

`encoding/json`包的这些功能- `,omitempty`、`"-"`和[其他选项](https://pkg.go.dev/encoding/json#Marshal)-不是标准。包决定如何处理结构体标记的值取决于它的实现。因为`encoding/json`包是标准库的一部分，其他包也以同样的方式实现了这些功能。但是，阅读任何使用struct标记的第三方软件包的文档以了解什么是支持的，什么是不支持的是很重要的。

## 结论

结构标记提供了一种强大的方法来增强使用结构的代码的功能。许多标准库和第三方包都提供了通过使用struct标记来自定义其操作的方法。在代码中有效地使用它们既提供了这种自定义行为，又简洁地说明了未来的开发人员如何使用这些字段。