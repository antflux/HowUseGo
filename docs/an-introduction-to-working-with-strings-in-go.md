# 在 Go 中使用字符串简介

字符串是一个或多个字符（字母、数字、符号）的序列，可以是常量或变量。由Unicode组成，字符串是不可变的序列，意味着它们是不可更改的。

由于文本在我们日常生活中是一种常见的数据形式，字符串数据类型是编程的一个非常重要的基本组件。

本Go教程将介绍如何创建和打印字符串，如何连接和复制字符串，以及如何将字符串存储在变量中。

## 字符串文本

在Go中，字符串存在于反引号`（有时称为反引号）或双引号"中。根据使用的引号，字符串将具有不同的特性。

使用反引号，如` ```bar``` `，将创建原始字符串字面量。在原始字符串字面量中，引号之间可以包含任何字符，但不包括反引号。以下是原始字符串字面量的示例：

```go
`Say "hello" to Go!`
```

在原始字符串字面量内，反斜杠没有特殊含义。例如，\n将显示为实际字符，即反斜杠\和字母n。与解释的字符串字面量不同，在其中\n会插入实际换行。

原始字符串字面量还可用于创建多行字符串：

```go
`Go is expressive, concise, clean, and efficient.
Its concurrency mechanisms make it easy to write programs
that get the most out of multi-core and networked machines,
while its novel type system enables flexible and modular
program construction. Go compiles quickly to machine code
yet has the convenience of garbage collection and the power
of run-time reflection. It's a fast, statically typed,
compiled language that feels like a dynamically typed,
interpreted language.`
```

解释的字符串字面量是双引号之间的字符序列，如"bar"。在引号之间，除了换行符和未转义的双引号之外，可以包含任何字符。

```go
"Say \"hello\" to Go!"
```

您几乎总是会使用解释的字符串字面量，因为它们允许其中包含转义字符。

现在您了解了Go中字符串的格式，让我们看看如何在程序中打印字符串。

## 打印字符串

您可以通过使用系统库的fmt包并调用Println()函数来打印字符串：

```go
fmt.Println("Let's print out this string.")
```

输出：

```
Let's print out this string.
```

在使用它们时，您必须导入系统包，因此打印字符串的简单程序如下所示：

```go
package main

import "fmt"

func main() {
    fmt.Println("Let's print out this string.")
}
```

## 字符串连接

连接意味着将字符串端对端连接在一起，以创建一个新字符串。您可以使用+运算符连接字符串。请注意，在处理数字时，+将是加法运算符，但在与字符串一起使用时，它是连接运算符。

让我们通过fmt.Println()语句将字符串字面量"Sammy"和"Shark"连接在一起：

```go
fmt.Println("Sammy" + "Shark")
```

输出：

```
SammyShark
```

如果您想在两个字符串之间加上空格，只需在字符串中包含空格。在此示例中，在Sammy之后的引号中添加空格：

```go
fmt.Println("Sammy " + "Shark")
```

输出：

```
Sammy Shark
```

+运算符不能在两种不同的数据类型之间使用。例如，您不能将字符串和整数连接在一起。如果尝试编写以下内容：

```go
fmt.Println("Sammy" + 27)
```

将收到以下错误：

```
cannot convert "Sammy" (type untyped string) to type int
invalid operation: "Sammy" + 27 (mismatched types string and int)
```

如果要创建字符串"Sammy27"，可以通过将数字27放在引号中（"27"）来实现，以便它不再是整数而是字符串。在处理邮政编码或电话号码时，将数字转换为字符串以进行连接可能很有用。例如，您不想在国家代码和区号之间执行加法，但您确实希望它们保持在一起。

通过连接两个或多个字符串，您正在创建一个新字符串，可以在整个程序中使用。

## 将字符串存储在变量中

变量是可以用来在程序中存储数据的符号。您可以将其视为一个空盒子，可以用一些数据或值来填充。字符串是数据，因此您可以使用它们来填充变量。将字符串声明为变量可能会使在Go程序中更轻松地处理字符串。

要将字符串存储在变量中，只需将变量分配给一个字符串。在这种情况下，将`s`声明为您的变量：

```go
s := "Sammy likes declaring strings."
```

注意：如果您熟悉其他编程语言，可能已将变量写为`sammy`。然而，Go更青睐更短的变量名称。在这种情况下，选择`s`作为变量名会被认为更符合Go编写风格的方式。

现在，您将变量`s`设置为特定字符串，可以像这样打印变量：

```go
fmt.Println(s)
```

然后，您将收到以下输出：

```
Sammy likes declaring strings.
```

通过使用变量代替字符串，您不必每次使用它时都重新输入字符串，从而更简单地处理和操作程序中的字符串。

## 结论

本教程概述了在Go编程语言中使用字符串数据类型的基础知识。创建和打印字符串，连接和复制字符串以及将字符串存储在变量中将为您提供在Go程序中使用字符串的基础知识。
