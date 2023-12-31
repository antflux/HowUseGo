# 在 Go 中定义结构

## 介绍

围绕具体细节构建抽象是编程语言可以给予给开发人员的最好的工具。结构允许Go开发人员描述Go程序运行的世界。结构体允许我们讨论`Street`，而不是推理描述`City`、`PostalCode`或`Address`的字符串。它们是我们努力告诉未来的开发人员（包括我们自己）哪些数据对我们的Go程序很重要，以及未来的代码应该如何适当地使用这些数据的文档的自然联系。结构可以以几种不同的方式定义和使用。在本教程中，我们将看看这些技术中的每一种。

## 定义结构

结构的工作方式类似于您可能使用的纸质表单，例如，用于提交税款。纸质表单可能有文本信息字段，如您的名字和姓氏。除了文本字段之外，表单还可能有复选框来指示布尔值，如“已婚”或“单身”，或日期字段来指示出生日期。类似地，结构将不同的数据片段收集在一起，并将它们组织在不同的字段名称下。当你用一个新的结构初始化一个变量时，就好像你已经复印了一个表单，并准备好填写它。

要创建一个新的结构体，必须首先给予Go一个描述结构体包含的字段的蓝图。这个结构体定义通常以关键字`type`开头，后跟结构体的名称。在此之后，使用关键字`struct`后跟一对大括号`{}`，在此声明结构体将包含的字段。一旦定义了结构，就可以声明使用此结构定义的变量。这个例子定义了一个struct并使用它：

```go
package main

import "fmt"

type Creature struct {
 Name string
}

func main() {
 c := Creature{
  Name: "Sammy the Shark",
 }
 fmt.Println(c.Name)
}
```

当你运行这段代码时，你会看到这样的输出：

```
Output
Sammy the Shark
```

在这个例子中，我们首先定义一个`Creature`结构体，它包含一个类型为`Name`的`string`字段。在`main`的主体中，我们通过在类型名称`Creature`之后放置一对大括号，然后为该实例的字段指定值来创建`Creature`的实例。`c`中的实例将其`Name`字段设置为“Sammy the Shark”。在`fmt.Println`函数调用中，我们通过在创建实例的变量之后放置一个句点，然后是我们想要访问的字段的名称来检索实例字段的值。例如，在这种情况下，`c.Name`返回`Name`字段。

声明结构的新实例时，通常会枚举字段名称及其值，如上一个示例所示。或者，如果每个字段值都将在结构体的实例化过程中提供，则可以省略字段名称，如以下示例所示：

```
package main

import "fmt"

type Creature struct {
 Name string
 Type string
}

func main() {
 c := Creature{"Sammy", "Shark"}
 fmt.Println(c.Name, "the", c.Type)
}
```

输出与上一个示例相同：

```
Output
Sammy the Shark
```

我们在`Creature`中添加了一个额外的字段，以跟踪`Type`生物作为`string`。当在`Creature`的主体中实例化`main`时，我们选择使用较短的实例化形式，按顺序为每个字段提供值并省略它们的字段名称。在声明`Creature{"Sammy", "Shark"}`中，`Name`字段的值为`Sammy`，而`Type`字段的值为`Shark`，因为`Name`在类型声明中首先出现，然后是`Type`。

这种较短的声明形式有一些缺点，导致Go社区在大多数情况下更喜欢较长的形式。使用简短声明时，必须为结构中的每个字段提供值-不能跳过不关心的字段。这很快会导致带有许多字段的结构的简短声明变得混乱。因此，使用短格式声明的结构通常用于字段较少的结构。

到目前为止，示例中的字段名称都以大写字母开头。这比风格偏好更重要。字段名使用大写字母或小写字母会影响在其他包中运行的代码是否可以访问您的字段名。

## 结构字段导出

结构体的字段遵循与Go编程语言中其他标识符相同的导出规则。如果字段名以大写字母开头，则在定义结构体的包外部的代码可以读取和写入该字段名。如果字段以小写字母开头，则只有该结构包中的代码才能够读取和写入该字段。此示例定义了要导出的字段和不导出的字段：

```
package main

import "fmt"

type Creature struct {
 Name string
 Type string

 password string
}

func main() {
 c := Creature{
  Name: "Sammy",
  Type: "Shark",

  password: "secret",
 }
 fmt.Println(c.Name, "the", c.Type)
 fmt.Println("Password is", c.password)
}
```

这将输出：

```
Output
Sammy the Shark
Password is secret
```

我们在前面的示例中添加了一个额外的字段`secret`。`secret`是一个未导出的`string`字段，这意味着任何其他试图实例化`Creature`的软件包将无法访问或设置其`secret`字段。在同一个包中，我们可以访问这些字段，就像这个例子所做的那样。由于`main`也在`main`包中，因此它能够引用`c.password`并检索存储在那里的值。在结构中有未导出的字段是很常见的，通过导出的方法来访问它们。

## 内联结构

除了定义一个新的类型来表示一个结构，你还可以定义一个内联结构。这些动态结构定义在为结构类型发明新名称将是浪费精力的情况下很有用。例如，测试通常使用结构来定义组成特定测试用例的所有参数。当结构体只在一个地方使用时，使用像`CreatureNamePrintingTestCase`这样的新名称会很麻烦。

内联结构定义出现在变量赋值的右侧。您必须通过为您定义的每个字段提供一对带有值的附加大括号来立即提供它们的实例化。下面的示例显示了一个内联结构定义：

```
package main

import "fmt"

func main() {
 c := struct {
  Name string
  Type string
 }{
  Name: "Sammy",
  Type: "Shark",
 }
 fmt.Println(c.Name, "the", c.Type)
}
```

这个例子的输出将是：

```
Output
Sammy the Shark
```

这个例子没有定义一个新的类型来描述我们的结构，而是通过将`type`定义放在短赋值运算符`struct`之后来定义一个内联结构。我们像前面的例子一样定义结构体的字段，但是我们必须立即提供另一对大括号和每个字段将采用的值。现在使用这个结构和以前完全一样-我们可以使用点表示法引用字段名。最常见的内联结构声明是在测试过程中，因为经常定义一次性结构来包含特定测试用例的数据和期望。

## 结论

结构体是程序员为组织信息而定义的异质数据的集合。大多数程序都要处理大量的数据，如果没有结构体，就很难记住哪些`string`或`int`变量属于一起，哪些是不同的。下次当你发现自己在处理一组变量时，问问自己，也许这些变量最好用`struct`组合在一起。这些变量可能一直在沿着描述一些更高层次的概念。
