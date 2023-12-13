# Go 中的 Strings 包简介

## 简介

在Go语言中，`strings`包提供了许多可用于处理字符串数据类型的函数。这些函数使我们能够轻松地修改和操作字符串。我们可以将函数视为在我们的代码元素上执行的操作。内置函数是在Go编程语言中定义的，可以直接供我们使用。

在本教程中，我们将回顾一些可用于处理Go中字符串的不同函数。

## 转换字符串为大写和小写

`strings.ToUpper` 和 `strings.ToLower` 函数将返回一个将原始字符串中所有字母转换为大写或小写字母的新字符串。由于字符串是不可变的数据类型，返回的字符串将是一个新字符串。字符串中不是字母的任何字符都不会更改。

要将字符串 "Sammy Shark" 转换为全部大写，可以使用 `strings.ToUpper` 函数：

```go
ss := "Sammy Shark"
fmt.Println(strings.ToUpper(ss))
```

输出：

```
SAMMY SHARK
```

要转换为小写：

```go
fmt.Println(strings.ToLower(ss))
```

输出：

```
sammy shark
```

由于您正在使用`strings`包，因此首先需要将其导入到程序中。要将字符串转换为大写并将整个程序转换为小写，可以使用以下代码：

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	ss := "Sammy Shark"
	fmt.Println(strings.ToUpper(ss))
	fmt.Println(strings.ToLower(ss))
}
```

`strings.ToUpper` 和 `strings.ToLower` 函数使得通过在整个字符串中保持一致的大小写来更轻松地评估和比较字符串成为可能。例如，如果用户以全小写形式写入其名称，我们仍然可以通过将其与全大写版本进行比较来确定其名称是否在我们的数据库中。

## 字符串搜索函数

`strings`包有许多函数可帮助确定字符串是否包含特定的字符序列。

| 函数                  | 用途                         |
|-----------------------|------------------------------|
| `strings.HasPrefix`   | 从字符串开头搜索              |
| `strings.HasSuffix`   | 从字符串末尾搜索              |
| `strings.Contains`    | 在字符串的任何位置搜索        |
| `strings.Count`       | 计算字符串出现的次数          |

`strings.HasPrefix` 和 `strings.HasSuffix` 允许您检查字符串是否以特定的字符集开始或结束。

例如，要检查字符串 "Sammy Shark" 是否以 "Sammy" 开始并以 "Shark" 结束：

```go
ss := "Sammy Shark"
fmt.Println(strings.HasPrefix(ss, "Sammy"))
fmt.Println(strings.HasSuffix(ss, "Shark"))
```

输出：

```
true
true
```

您可以使用 `strings.Contains` 函数检查 "Sammy Shark" 是否包含序列 "Sh"：

```go
fmt.Println(strings.Contains(ss, "Sh"))
```

输出：

```
true
```

最后，要查看短语 "Sammy Shark" 中字母 S 出现的次数：

```go
fmt.Println(strings.Count(ss, "S"))
```

输出：

```
2
```

注意：Go中的所有字符串都是区分大小写的。这意味着"Sammy"与"sammy"不同。

使用小写字母s从"Sammy Shark"中获取计数与使用大写字母S不同：

```go
fmt.Println(strings.Count(ss, "s"))
```

输出：

```
0
```

因为S与s不同，返回的计数将为0。

字符串函数在您想要在程序中比较或搜索字符串时非常有用。

## 确定字符串长度

内置函数 `len()` 返回字符串中的字符数。当您需要强制执行最小或最大密码长度，或将较大的字符串截断为在某些限制内使用作为缩写时，此函数非常有用。

为了演示这个函数，我们将找到一个句子长的字符串的长度：

```go
import (
	"fmt"
	"strings"
)

func main() {
	openSource := "Sammy contributes to open source."
	fmt.Println(len(openSource))
}
```

输出：

```
33
```

我们将变量`openSource`设置为字符串"Sammy contributes to open source."，然后将该变量传递给 `len()` 函数，即 `len(openSource)`。最后，我们将该函数传递给 `fmt.Println()` 函数，以便我们可以在屏幕上看到程序的输出。

请注意，`len()` 函数将计算由双引号限定的任何字符的数量，包括字母、数字、空格字符和符号。

## 字符串操作函数

`strings.Join`、`strings.Split` 和 `strings.ReplaceAll` 函数是在Go中操纵字符串的一些额外方式。

`strings.Join` 函数用于将一组字符串组合成一个新的单个字符串。

要从一组字符串中创建一个以逗号分隔的字符串，我们将使用以下函数：

```go
fmt.Println(strings.Join([]string{"sharks", "crustaceans", "plankton"}, ","))
```

输出

：

```
sharks,crustaceans,plankton
```

如果我们想在我们的新字符串中在字符串值之间添加逗号和空格，我们可以简单地在逗号后重新编写我们的表达式：`strings.Join([]string{"sharks", "crustaceans", "plankton"}, ", ")`。

就像我们可以将字符串连接在一起一样，我们也可以将字符串拆分开。为此，我们可以使用 `strings.Split` 函数并在空格上拆分：

```go
balloon := "Sammy has a balloon."
s := strings.Split(balloon, " ")
fmt.Println(s)
```

输出：

```
[Sammy has a balloon]
```

输出是一个字符串切片。由于使用了 `strings.Println`，通过查看它很难确定输出是什么。要查看它确实是一个字符串切片，可以使用 `fmt.Printf` 函数并使用 `%q` 占位符将字符串引用起来：

```go
fmt.Printf("%q", s)
```

输出：

```
["Sammy" "has" "a" "balloon."]
```

除了 `strings.Split` 外，另一个有用的函数是 `strings.Fields`。区别在于 `strings.Fields` 将忽略所有空格，并且只会拆分字符串中的实际字段：

```go
data := "  username password     email  date"
fields := strings.Fields(data)
fmt.Printf("%q", fields)
```

输出：

```
["username" "password" "email" "date"]
```

`strings.ReplaceAll` 函数可以获取原始字符串并返回一个带有一些替换的更新字符串。

假设Sammy失去了他的气球。由于Sammy不再拥有这个气球，我们将在新字符串中将原始字符串balloon中的子字符串"has"更改为"had"：

```go
fmt.Println(strings.ReplaceAll(balloon, "has", "had"))
```

在括号中，首先是balloon，它存储原始字符串；第二个子字符串"has"是我们想要替换的，第三个子字符串"had"是我们想要用第二个子字符串替换的。将其合并到程序中时，我们的输出将如下所示：

输出：

```
Sammy had a balloon.
```

使用字符串函数 `strings.Join`、`strings.Split` 和 `strings.ReplaceAll` 将为您提供更多控制权以在Go中操纵字符串。

## 结论

本教程介绍了用于处理和操纵Go程序中的字符串的字符串数据类型的一些常见`strings`包函数。

您可以在 [Understanding Data Types](https://yourbasic.org/golang/understanding-data-types/) 中了解更多关于其他数据类型的信息，并在 [An Introduction to Working with Strings](https://yourbasic.org/golang/strings-explained/) 中阅读有关字符串的更多信息。