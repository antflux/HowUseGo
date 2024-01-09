# 如何在 Go 中使用模板

## 介绍

您是否需要在格式良好的输出、文本报告或HTML页面中显示某些数据？你可以用Go模板来实现。任何Go程序都可以使用[`text/template`](https://pkg.go.dev/text/template)或[`html/template`](https://pkg.go.dev/html/template)包-都包含在Go标准库中-来整齐地显示数据。

这两个包都允许您编写文本模板，并将数据传递到其中，以根据您的喜好呈现格式化的文档。在模板中，您可以对数据进行循环，并使用条件逻辑来决定在文档中包含哪些项以及它们应该如何显示。本教程将向您展示如何使用这两个模板包。首先，您将使用`text/template`使用循环、条件逻辑和自定义函数将一些数据呈现为纯文本报表。然后，您将使用`html/template`将相同的数据呈现到一个没有代码注入的HTML文档中。

## 先决条件

在开始本教程之前，您只需要安装Go。阅读适合您的操作系统的教程：

- [如何在Ubuntu上安装Go 20.04]
- [如何在macOS上安装Go并设置本地编程环境]
- [如何在Windows 10上安装Go并设置本地编程环境]

您还需要Go语言的工作知识，包括[创建结构]和[使用结构方法]。

我们开始吧

## 步骤1 -重复`text/template`

假设您想要生成一个关于狗的数据的简单报告。你想像这样显示它：

```
---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

这是您将使用`text/template`包生成的报告。突出显示的项目是您的数据，其余的是模板中的静态文本。模板要么作为字符串存在于代码中，要么与代码一起存在于它们自己的文件中。它们包含与条件语句（即if/else）、流控制语句（即循环）和函数调用交错的样板静态文本，所有这些都包装在`{{. . .}}`标记中。您将向模板中传递一些数据，以呈现与上面类似的最终文档。

首先，导航到你的Go工作区（`go env GOPATH`），为这个项目创建一个新目录：

```bash
cd `go env GOPATH`
mkdir pets
cd pets
```


使用`nano`或您最喜欢的文本编辑器，打开一个名为`pets.go`的新文件并粘贴以下内容：

```bash
nano pets.go
```


pets.go

```go
package main

import (
	"os"
	"text/template"
)

func main() {
}
```


这个文件声明自己在`main`包中，并包含一个`main`函数，这意味着它可以使用`go run`运行。它导入`text/template`标准库包，允许您编写和渲染模板，以及`os`，将用于打印到终端。

## 步骤2 -创建模板数据

在编写模板之前，让我们创建一些要传递到模板中的数据。在`import`语句下面和`main()`之前，定义一个名为`Pet`的结构，它包含宠物的`Name`、`Sex`、宠物是否绝育（`Intact`）、`Age`和`Breed`字段。编辑`pets.go`并添加此结构：

pets.go

```go
. . .
type Pet struct {
	Name   string
	Sex    string
	Intact bool
	Age    string
	Breed  string
}
. . .
```


现在，在`main()`函数的主体中，创建一个`Pet`s的切片来保存关于两只狗的数据：

pets.go

```go
. . .
func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
	}
} // end main
```


此数据将传递到您的模板以呈现最终报告。当然，传递给模板的数据可以来自任何地方：数据库、第三方API等。对于本教程，最简单的方法是将一些示例数据粘贴到代码中。

现在让我们来看看如何呈现或执行，在这些软件包的术语中，一个模板。

## 步骤3 -执行模板

在这一步中，您将看到如何使用`text/template`从模板生成完成的文档，但直到第4步，您才真正编写出有用的模板。

创建一个名为`pets.tmpl`的空文本文件，其中包含一些静态文本：

pets.tmpl

```
Nothing here yet.
```

保存模板并退出编辑器。如果您使用的是`nano`，请按`CTRL+X`，然后按`Y`和`ENTER`确认更改。

虽然执行这个模板只会打印“Nothing here yet."，让我们把你的数据传进去，然后执行这个模板，只是为了证明`text/template`是有效的。在`main()`函数中，在`dogs`切片之后添加以下内容：

pets.go

```go
	. . .
	var tmplFile = “pets.tmpl”
	tmpl, err := template.New(tmplFile).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, dogs)
	if err != nil {
		panic(err)
	}
} // end main
```


在这段代码中，您使用`Template.New`创建了一个新的`Template`，然后在生成的模板上调用`ParseFiles`来解析最小模板文件。检查错误后，调用新模板的`Execute`方法，传入`os.Stdout`将完成的报告打印到终端，同时传入`dogs`切片。对于第一个参数，您可以传入任何实现[`io.Writer`](https://pkg.go.dev/io#Writer)接口的内容，这意味着您可以将报告写入文件。我们稍后会看到如何做到这一点。

完整的程序应该是这样的：

```go
package main

import (
	"os"
	"text/template"
)

type Pet struct {
	Name   string
	Sex    string
	Intact bool
	Age    string
	Breed  string
}

func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
	}
	var tmplFile = “pets.tmpl”
	tmpl, err := template.New(tmplFile).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, dogs)
	if err != nil {
		panic(err)
	}
} // end main
```


保存程序，然后使用`go run`运行它：

```bash
go run pets.go
```


```
Output
Nothing here yet.
```

程序还没有打印数据，但至少代码运行得很干净。现在让我们写一个模板。

## 步骤4 -编写模板

模板只是[UTF-8](https://en.wikipedia.org/wiki/UTF-8)纯文本，但还有更多的内容。是的，它包含一些静态文本，这些文本将在最终输出中不变地显示，但它也包含[操作](https://pkg.go.dev/text/template#hdr-Actions)，这些操作是模板引擎的指令，告诉它如何遍历传入的数据以及在输出中包含什么。操作被一对左、右双花括号（`{{ <action> }}`）包裹，它们通过光标（用点（`.`）表示）访问数据。

传入模板的数据可以是任何东西，但传入切片、数组或映射（可迭代的东西）是很常见的。让模板遍历您的`dogs`切片。

### 在切片上循环

在Go代码中，你可以在`range`循环的开始语句中使用`for`来遍历一个切片。在模板中，您可以使用`range`动作来实现相同的目的，但它具有不同的语法：没有`for`，但添加了`end`来关闭循环。

打开`pets.tmpl`并将其内容替换为以下内容：

pets.tmpl

```go
{{ range . }}
---
(Pet will appear here...)
{{ end }}
```


`range`操作在这里接受一个参数：cursor（`.`），它引用整个`dogs`切片。该循环是封闭的，`{{ end }}`在底部。在循环体中，您正在打印一些静态文本，但仍然没有任何关于狗的内容。

保存保存`pets.tmpl`并再次运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
(Pet will appear here...)

---
(Pet will appear here...)
```

静态文本打印两次，因为切片中有两条狗。现在让我们用一些更有用的静态文本沿着狗数据来替换它。

### 阿格拉机场

当在这个模板中传递`.`到`range`时，点指的是整个切片，但是在`range`循环的每次迭代中，点指的是切片中的当前项。这可以让您访问导出的领域，每个宠物只使用裸点。您不需要引用切片索引。

一个字段非常简单，只需将其括在花括号中，并在其前面加上点即可。打开`pets.tmpl`并将其内容替换为：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }}

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

现在`pets.go`将为两只狗中的每一只打印五个字段中的四个，包括字段的一些标签。(We我马上就到第五场了。）

保存并再次运行程序：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

看起来不错现在让我们看看如何使用条件逻辑来显示第五个字段。

### 使用条件句

我们没有通过在模板中添加`Intact`来包含`{{ .Intact }}`字段的原因是这对读者来说不是很友好。想象一下，如果你的兽医账单在关于你的狗的摘要中说`Intact: false`。虽然将此字段存储为布尔值而不是字符串可能更有效，并且`Intact`是此字段的一个很好的中性名称，但我们可以通过使用if-else操作在最终报告中以不同的方式显示它。

再次打开`pets.tmpl`并添加如下所示的突出显示部分：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}fixed{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

模板现在检查`Intact`字段是否为真，如果是，则打印`(intact)`，否则打印`(fixed)`。但我们可以做得更好。让我们进一步编辑模板，为固定的狗打印性别特定的术语-`spayed`或`neutered`-而不是`fixed`。在原始`if`中添加嵌套的`else`：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if (eq .Sex "Female") }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

保存模板并运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

我们有两只狗，但有三种可能的情况显示`Intact`。让我们在`pets.go`中的切片中再添加一条狗，以涵盖所有三种情况。编辑`pets.go`并将第三只狗附加到切片：

pets.go

```go
. . .
func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
		{
			Name:	"Bruce Wayne",
			Sex:	"Male",
			Intact:	false,
			Age:	"3 years, 8 months",
			Breed:	"Chihuahua",
		},
	}
. . .
```


现在保存并运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

太好了，看起来和预期的一样。

现在让我们讨论模板函数，比如你刚刚使用的`eq`函数。

### 使用模板函数

沿着`eq`，还有其他函数用于比较字段值并返回布尔值：`gt`（），`ne`（！=），`le`（=）等。您可以通过以下两种方式之一来调用这些函数和[任何模板函数](https://pkg.go.dev/text/template#hdr-Functions)：

1. 写出函数名，后跟一个或多个参数，每个参数之间有一个空格。这就是你如何使用上面的`eq`：`eq .Sex “Female”`。
2. 先写一个参数，然后写一个管道（`|`），然后写函数名，再写任何其他参数。这类似于[Unix命令行上的命令管道]，并且在命令行上，您可以将[管道](https://pkg.go.dev/text/template#hdr-Pipelines)中的许多函数调用链接在一起，将一个调用的输出作为下一个调用的输入传递，依此类推。

因此，虽然模板中的`eq`比较被写为`eq .Sex “Female”`，但它也可以被写为`.Sex | eq “Female”`。这两个表达式是等价的。

让我们使用`len`函数在报告的顶部显示狗的数量。打开`pets.tmpl`并将以下内容添加到顶部：

pets.tmpl

```go
Number of dogs: {{ . | len -}}

{{ range . }}
. . .
```


你也可以写#1。

请注意右双花括号旁边的破折号（`-`）。这可以防止在操作后打印换行符（`\n`）。您也可以在双花括号（`{{-`）后面附加一个破折号，以在操作之前取消换行符。

保存模板并运行`pets.go`：

```bash
go run pets.go
```


```
Output
Number of dogs: 3
---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd & Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd & Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

由于`{{ . | len -}}`中的破折号，`Number of dogs`标签和第一只狗之间没有空行。

您可能已经注意到，`text/template`文档中的内置函数列表非常小。好消息是，你可以让任何Go函数在你的模板中可用，只要它返回一个值，或者如果第二个是`error`类型，则返回两个值。

### 在模板中使用Go函数

假设您想编写一个模板，它采用一个狗切片，只打印最后一个。您可以使用`slice`内置函数在模板中获取切片的子集，其工作方式类似于Go中的`mySlice[x:y]`。你可以写`{{ slice . 2 }}`来获取一个三项切片的最后一项，尽管`slice`函数返回另一个切片，而不是一个项。也就是说，`{{ slice . 2 }}`相当于`slice[2:]`，而不是`slice[2]`。(The函数也可以使用多个索引，例如`{{ slice . 0 2 }}`来获取切片`slice[0:2]`，但这里不会使用它。

但是如何在模板中引用切片的最后一个索引呢？`len`函数是可用的，但切片中的最后一项的索引为`len - 1`，不幸的是，您无法在模板中进行算术运算。

这就是自定义函数非常有用的地方。让我们写一个函数，减少一个整数，并使该函数可用于我们的模板。

但在此之前，让我们做一个新的模板。打开一个新文件`lastPet.tmpl`并粘贴以下内容：

lastPet.tmpl

```go
{{- range (len . | dec | slice . ) }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if ("Female" | eq .Sex) }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end -}}
```


第一行中对`dec`函数的调用引用了您将要定义并传递给模板的自定义函数。在`main()`中的`pets.go`函数内，在`dogs`切片下方和调用`tmpl.Execute()`之前进行以下更改（突出显示）：

pets.go

```go
	. . .
	funcMap := template.FuncMap{
		"dec": func(i int) int { return i - 1 },
	}
	var tmplFile = “lastPet.tmpl”
	tmpl, err := template.New(tmplFile).Funcs(funcMap).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	. . .
```


首先，你声明了一个`FuncMap`，这是一个函数映射：键是模板可用的函数名，值是函数本身。这里的一个函数，`dec`，是一个内联提供[的匿名函数，](https://en.wikipedia.org/wiki/Anonymous_function)因为它很短。它接受一个整数，从中减去1，然后返回结果。

然后，您将更改模板文件名。最后，在对`Template.Funcs`的调用之前插入对`ParseFile`的调用，并将刚刚定义的`funcMap`传递给它。`Funcs`方法必须在`ParseFiles`之前调用。

在运行你的代码之前，让我们了解一下你的模板中的`range`动作发生了什么：

template_last_pet.go

```go
{{- range (len . | dec | slice . ) }}
```


您将获取`dogs`切片的长度，将其传递给自定义的`dec`函数以减去1，然后将其作为第二个参数传递给前面讨论的`slice`函数。因此，对于一个三狗切片，`range`操作相当于`{{- range (slice . 2) }}`。

保存`pets.go`并运行它：

```
go run pets.go
Output

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

看起来不错如果你想展示最后两只狗而不仅仅是最后一只呢？编辑`lastPet.tmpl`并将另一个对`dec`的调用添加到管道中：

lastPet.tmpl

```go
{{- range (len . | dec | dec | slice . ) }}
. . .
```


保存文件并再次运行`pets.go`：

```go
go run pets.go
```


```
Output

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

你可以想象如何改进`dec`函数，让它接受一个参数并更改它的名称，这样你就可以调用`minus 2`而不是`dec | dec`。

现在假设你想以不同的方式显示像Zephyr这样的混血狗，用&符号代替斜杠。你不需要自己写函数，因为`strings`包有一个，你可以从任何包中借用一个函数来在你的模板中使用。编辑`pets.go`以导入`strings`包并将其函数之一添加到`funcMap`：

pets.go

```go
package main

import (
	"os"
	"strings"
	"text/template"
)
. . .
func main() {
	. . .
	funcMap := template.FuncMap{
		"dec":     func(i int) int { return i - 1 },
		"replace": strings.ReplaceAll,
	}
	. . .
} // end main
```


您现在导入了`strings`包，并将其`ReplaceAll`函数添加到您的`funcMap`中，命名为`replace`。现在编辑`lastPet.tmpl`来使用这个函数：

lastPet.tmpl

```go
{{- range (len . | dec | dec | slice . ) }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if ("Female" | eq .Sex) }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ replace .Breed “/” “ & ” }}
{{ end -}}
```


保存文件并再次运行它：

```bash
go run pets.go
```


```
Output

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd & Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

和风的品种现在包含一个&而不是斜线。

您可以在`pets.go`中操作该字符串而不是您的模板，但数据的表示是模板的工作，而不是代码。

事实上，一些狗的数据已经包含了一些表示，也许它不应该。`Breed`字段将多个品种压缩到一个字符串中，并用斜杠将它们标点。这种单字符串模式可能会让数据输入者在数据库中引入不同的格式：`Labrador/Poodle`、`Labrador & Poodle`、`Labrador, Poodle`、`Labrador-Poodle mix`等。最好将`Breed`存储为字符串片段（`[]string`）而不是字符串，以避免这种格式歧义，并使按品种搜索更灵活，更容易呈现。然后你可以使用模板中的`strings.Join`函数来打印所有的品种，再加上`.Breed`字段的额外注释（`(purebred)`或`(mixed breed)`）。

自己试试

尝试修改您的代码和模板来实现这些更改。完成后，单击下面的解决方案以检查您的工作。

<details style="box-sizing: border-box; margin: 1em 0px 0px; display: block; background: rgb(249, 250, 254); border-radius: 16px; padding: 1em;"><summary style="box-sizing: border-box; margin: 0px; display: list-item; cursor: pointer; list-style: none; padding: 0px 1em 0px 0px; position: relative;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;">溶液</font></summary><div class="code-label" title="pets.go" style="box-sizing: border-box; margin: 1em 0px 0px; background-color: rgb(214, 220, 234); border-radius: 16px 16px 0px 0px; color: rgb(36, 51, 90); display: block; font-size: 16px; padding: 12px; position: relative; text-align: center; z-index: 2;"><font __transmart="" style="box-sizing: border-box; margin: 0px;"></font></div><div class="code-toolbar" style="box-sizing: border-box; margin: 0px 0px 1em; position: relative;"><pre class="language-go" style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: rgb(17, 25, 46); border-radius: 0px 0px 16px 16px; color: rgb(247, 248, 251); line-height: 22px; padding: 24px; box-shadow: rgba(17, 25, 46, 0.1) 0px 0px 0px 2px inset; display: block; overflow: auto; white-space: normal; overflow-wrap: normal;"><code style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: transparent; border-radius: 0px; color: inherit; line-height: 22px; padding: 0px; white-space: pre; text-shadow: none;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token function" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(255, 175, 140);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token number" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(11, 225, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token comment" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></code></pre><div class="toolbar" style="box-sizing: border-box; margin: 0px; position: absolute; right: 24px; top: 24px;"><div class="toolbar-item" style="box-sizing: border-box; margin: 0px; display: inline-block;"><button type="button" style="box-sizing: border-box; margin: 0px; font-family: inherit; font-size: 0.9em; line-height: normal; overflow: visible; text-transform: none; appearance: button; font-style: inherit; font-variant: inherit; font-weight: inherit; font-stretch: inherit; font-optical-sizing: inherit; font-kerning: inherit; font-feature-settings: inherit; font-variation-settings: inherit; background: rgb(105, 111, 176); border: 0px; cursor: pointer; user-select: none; border-radius: 10px; color: rgb(255, 255, 255); padding: 6px 8px; transition: color 0.25s ease 0s, background 0.25s ease 0s;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;"></font></button></div></div></div><div class="code-label" title="lastPet.tmpl" style="box-sizing: border-box; margin: 1em 0px 0px; background-color: rgb(214, 220, 234); border-radius: 16px 16px 0px 0px; color: rgb(36, 51, 90); display: block; font-size: 16px; padding: 12px; position: relative; text-align: center; z-index: 2;"><font __transmart="" style="box-sizing: border-box; margin: 0px;"></font></div><div class="code-toolbar" style="box-sizing: border-box; margin: 0px 0px 1em; position: relative;"><pre class="language-go" style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: rgb(17, 25, 46); border-radius: 0px 0px 16px 16px; color: rgb(247, 248, 251); line-height: 22px; padding: 24px; box-shadow: rgba(17, 25, 46, 0.1) 0px 0px 0px 2px inset; display: block; overflow: auto; white-space: normal; overflow-wrap: normal;"><code style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: transparent; border-radius: 0px; color: inherit; line-height: 22px; padding: 0px; white-space: pre; text-shadow: none;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token number" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(11, 225, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></code></pre><div class="toolbar" style="box-sizing: border-box; margin: 0px; position: absolute; right: 24px; top: 24px;"><div class="toolbar-item" style="box-sizing: border-box; margin: 0px; display: inline-block;"><button type="button" style="box-sizing: border-box; margin: 0px; font-family: inherit; font-size: 0.9em; line-height: normal; overflow: visible; text-transform: none; appearance: button; font-style: inherit; font-variant: inherit; font-weight: inherit; font-stretch: inherit; font-optical-sizing: inherit; font-kerning: inherit; font-feature-settings: inherit; font-variation-settings: inherit; background: rgb(105, 111, 176); border: 0px; cursor: pointer; user-select: none; border-radius: 10px; color: rgb(255, 255, 255); padding: 6px 8px; transition: color 0.25s ease 0s, background 0.25s ease 0s;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;"></font></button></div></div></div></details>

最后，让我们将相同的狗数据呈现到HTML文档中，看看为什么当模板的输出是HTML时，应该始终使用`html/template`包。

## 第5步-编写HTML模板

命令行工具可能使用`text/template`来整齐地打印输出，而其他一些批处理程序可能使用它来从某些数据创建结构良好的文件。但是Go模板通常也用于为Web应用程序呈现HTML页面。例如，流行的开源静态站点生成器[Hugo](https://gohugo.io/)使用`text/template`和`html/template`作为其模板的基础。

HTML有一些纯文本没有的特殊挑战。它使用尖括号来包装元素（`<td>`），使用&符号来标记实体（` `），使用引号来包装标记属性（`<a href=“https://www.digitalocean.com/”>`）。如果模板插入的任何数据包含这些字符，使用`text/template`包可能会导致HTML格式错误，或者更糟的是，代码注入。

`html/template`包不是这样的。这个包转义了这些有问题的字符，插入了它们的安全HTML等价物（实体）。数据中的&号变为`&`，左尖括号变为`<`，依此类推。

让我们使用前面相同的狗数据来创建一个HTML文档，但首先我们将继续使用`text/template`来显示它的危险。

打开`pets.go`，并将以下突出显示的文本添加到Repubbe的`Name`字段中：

pets.go

```go
        . . .
	dogs := []Pet{
		{
			Name:   "<script>alert(\"Gotcha!\");</script>Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pit Bull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
		{
			Name:   "Bruce Wayne",
			Sex:    "Male",
			Intact: false,
			Age:    "3 years, 8 months",
			Breed:  "Chihuahua",
		},
	}
        . . .
```


现在在名为`petsHtml.tmpl`的新文件中创建一个HTML模板：

petsHtml.tmpl

```go
<p><strong>Pets:</strong> {{ . | len }}</p>
{{ range . }}
<hr />
<dl>
	<dt>Name</dt>
	<dd>{{ .Name }}</dd>
	<dt>Sex</dt>
	<dd>{{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if (eq .Sex "Female") }}spayed{{ else }}neutered{{ end }}{{ end }})</dd>
	<dt>Age</dt>
	<dd>{{ .Age }}</dd>
	<dt>Breed</dt>
	<dd>{{ replace .Breed “/” “ & ” }}</dd>
</dl>
{{ end }}
```


保存HTML模板。在运行`pets.go`之前，我们需要编辑`tmpFile` var，但我们也要编辑程序，将模板输出到文件而不是终端。打开`pets.go`并在`main()`函数中添加突出显示的代码：

pets.go

```go
	. . .
	funcMap := template.FuncMap{
		"dec":     func(i int) int { return i - 1 },
		"replace": strings.ReplaceAll,
	}
	var tmplFile = "petsHtml.tmpl"
	tmpl, err := template.New(tmplFile).Funcs(funcMap).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	var f *os.File
	f, err = os.Create("pets.html")
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(f, dogs)
	if err != nil {
		panic(err)
	}
	err = f.Close()
	if err != nil {
		panic(err)
	}
} // end main
```


你打开一个新的名为`File`的`pets.html`，将它（而不是`os.Stdout`）传递给`tmpl.Execute`，然后在完成时关闭文件。

现在运行`go run pets.go`生成HTML文件。然后，在浏览器中打开此本地网页：

![gotcha.png](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/gotcha.png)

浏览器已运行注入的脚本。这就是为什么你不应该使用`text/template`包来生成HTML，特别是当你不能完全信任你的模板数据的来源时。

除了转义数据中的HTML字符外，`html/template`包的工作方式与`text/template`一样，并且具有相同的基本名称（`template`），这意味着要使模板注入安全，您所要做的就是将`text/template`导入替换为`html/template`。编辑`pets.go`并立即执行：

pets.go

```go
package main

import (
	"os"
	"strings"
	"html/template"
)
. . .
```


保存该文件并最后运行一次，重复`pets.html`。然后在浏览器中刷新HTML文件：

![gotcha-sanitized.png](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/gotcha-sanitized.png)

`html/template`包已经将注入的脚本呈现为网页中的文本。在你的文本编辑器中打开`pets.html`（或者在浏览器中查看页面源代码）[介绍]

您是否需要在格式良好的输出、文本报告或HTML页面中显示某些数据？你可以用Go模板来实现。任何Go程序都可以使用[`text/template`](https://pkg.go.dev/text/template)或[`html/template`](https://pkg.go.dev/html/template)包-都包含在Go标准库中-来整齐地显示数据。

这两个包都允许您编写文本模板，并将数据传递到其中，以根据您的喜好呈现格式化的文档。在模板中，您可以对数据进行循环，并使用条件逻辑来决定在文档中包含哪些项以及它们应该如何显示。本教程将向您展示如何使用这两个模板包。首先，您将使用`text/template`使用循环、条件逻辑和自定义函数将一些数据呈现为纯文本报表。然后，您将使用`html/template`将相同的数据呈现到一个没有代码注入的HTML文档中。

## 先决条件

在开始本教程之前，您只需要安装Go。阅读适合您的操作系统的教程：

- [如何在Ubuntu上安装Go 20.04]
- [如何在macOS上安装Go并设置本地编程环境]
- [如何在Windows 10上安装Go并设置本地编程环境]

您还需要Go语言的工作知识，包括[创建结构]和[使用结构方法]。

我们开始吧

## 步骤1 -重复`text/template`

假设您想要生成一个关于狗的数据的简单报告。你想像这样显示它：

```
---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

这是您将使用`text/template`包生成的报告。突出显示的项目是您的数据，其余的是模板中的静态文本。模板要么作为字符串存在于代码中，要么与代码一起存在于它们自己的文件中。它们包含与条件语句（即if/else）、流控制语句（即循环）和函数调用交错的样板静态文本，所有这些都包装在`{{. . .}}`标记中。您将向模板中传递一些数据，以呈现与上面类似的最终文档。

首先，导航到你的Go工作区（`go env GOPATH`），为这个项目创建一个新目录：

```bash
cd `go env GOPATH`
mkdir pets
cd pets
```


使用`nano`或您最喜欢的文本编辑器，打开一个名为`pets.go`的新文件并粘贴以下内容：

```bash
nano pets.go
```


pets.go

```go
package main

import (
	"os"
	"text/template"
)

func main() {
}
```


这个文件声明自己在`main`包中，并包含一个`main`函数，这意味着它可以使用`go run`运行。它导入`text/template`标准库包，允许您编写和渲染模板，以及`os`，将用于打印到终端。

## 步骤2 -创建模板数据

在编写模板之前，让我们创建一些要传递到模板中的数据。在`import`语句下面和`main()`之前，定义一个名为`Pet`的结构，它包含宠物的`Name`、`Sex`、宠物是否绝育（`Intact`）、`Age`和`Breed`字段。编辑`pets.go`并添加此结构：

pets.go

```go
. . .
type Pet struct {
	Name   string
	Sex    string
	Intact bool
	Age    string
	Breed  string
}
. . .
```


现在，在`main()`函数的主体中，创建一个`Pet`s的切片来保存关于两只狗的数据：

pets.go

```go
. . .
func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
	}
} // end main
```


此数据将传递到您的模板以呈现最终报告。当然，传递给模板的数据可以来自任何地方：数据库、第三方API等。对于本教程，最简单的方法是将一些示例数据粘贴到代码中。

现在让我们来看看如何呈现或执行，在这些软件包的术语中，一个模板。

## 步骤3 -执行模板

在这一步中，您将看到如何使用`text/template`从模板生成完成的文档，但直到第4步，您才真正编写出有用的模板。

创建一个名为`pets.tmpl`的空文本文件，其中包含一些静态文本：

pets.tmpl

```
Nothing here yet.
```

保存模板并退出编辑器。如果您使用的是`nano`，请按`CTRL+X`，然后按`Y`和`ENTER`确认更改。

虽然执行这个模板只会打印“Nothing here yet."，让我们把你的数据传进去，然后执行这个模板，只是为了证明`text/template`是有效的。在`main()`函数中，在`dogs`切片之后添加以下内容：

pets.go

```go
	. . .
	var tmplFile = “pets.tmpl”
	tmpl, err := template.New(tmplFile).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, dogs)
	if err != nil {
		panic(err)
	}
} // end main
```


在这段代码中，您使用`Template.New`创建了一个新的`Template`，然后在生成的模板上调用`ParseFiles`来解析最小模板文件。检查错误后，调用新模板的`Execute`方法，传入`os.Stdout`将完成的报告打印到终端，同时传入`dogs`切片。对于第一个参数，您可以传入任何实现[`io.Writer`](https://pkg.go.dev/io#Writer)接口的内容，这意味着您可以将报告写入文件。我们稍后会看到如何做到这一点。

完整的程序应该是这样的：

```go
package main

import (
	"os"
	"text/template"
)

type Pet struct {
	Name   string
	Sex    string
	Intact bool
	Age    string
	Breed  string
}

func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
	}
	var tmplFile = “pets.tmpl”
	tmpl, err := template.New(tmplFile).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, dogs)
	if err != nil {
		panic(err)
	}
} // end main
```


保存程序，然后使用`go run`运行它：

```bash
go run pets.go
```


```
Output
Nothing here yet.
```

程序还没有打印数据，但至少代码运行得很干净。现在让我们写一个模板。

## 步骤4 -编写模板

模板只是[UTF-8](https://en.wikipedia.org/wiki/UTF-8)纯文本，但还有更多的内容。是的，它包含一些静态文本，这些文本将在最终输出中不变地显示，但它也包含[操作](https://pkg.go.dev/text/template#hdr-Actions)，这些操作是模板引擎的指令，告诉它如何遍历传入的数据以及在输出中包含什么。操作被一对左、右双花括号（`{{ <action> }}`）包裹，它们通过光标（用点（`.`）表示）访问数据。

传入模板的数据可以是任何东西，但传入切片、数组或映射（可迭代的东西）是很常见的。让模板遍历您的`dogs`切片。

### 在切片上循环

在Go代码中，你可以在`range`循环的开始语句中使用`for`来遍历一个切片。在模板中，您可以使用`range`动作来实现相同的目的，但它具有不同的语法：没有`for`，但添加了`end`来关闭循环。

打开`pets.tmpl`并将其内容替换为以下内容：

pets.tmpl

```go
{{ range . }}
---
(Pet will appear here...)
{{ end }}
```


`range`操作在这里接受一个参数：cursor（`.`），它引用整个`dogs`切片。该循环是封闭的，`{{ end }}`在底部。在循环体中，您正在打印一些静态文本，但仍然没有任何关于狗的内容。

保存保存`pets.tmpl`并再次运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
(Pet will appear here...)

---
(Pet will appear here...)
```

静态文本打印两次，因为切片中有两条狗。现在让我们用一些更有用的静态文本沿着狗数据来替换它。

### 阿格拉机场

当在这个模板中传递`.`到`range`时，点指的是整个切片，但是在`range`循环的每次迭代中，点指的是切片中的当前项。这可以让您访问导出的领域，每个宠物只使用裸点。您不需要引用切片索引。

一个字段非常简单，只需将其括在花括号中，并在其前面加上点即可。打开`pets.tmpl`并将其内容替换为：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }}

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

现在`pets.go`将为两只狗中的每一只打印五个字段中的四个，包括字段的一些标签。(We我马上就到第五场了。）

保存并再次运行程序：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

看起来不错现在让我们看看如何使用条件逻辑来显示第五个字段。

### 使用条件句

我们没有通过在模板中添加`Intact`来包含`{{ .Intact }}`字段的原因是这对读者来说不是很友好。想象一下，如果你的兽医账单在关于你的狗的摘要中说`Intact: false`。虽然将此字段存储为布尔值而不是字符串可能更有效，并且`Intact`是此字段的一个很好的中性名称，但我们可以通过使用if-else操作在最终报告中以不同的方式显示它。

再次打开`pets.tmpl`并添加如下所示的突出显示部分：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}fixed{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

模板现在检查`Intact`字段是否为真，如果是，则打印`(intact)`，否则打印`(fixed)`。但我们可以做得更好。让我们进一步编辑模板，为固定的狗打印性别特定的术语-`spayed`或`neutered`-而不是`fixed`。在原始`if`中添加嵌套的`else`：

pets.tmpl

```
{{ range . }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if (eq .Sex "Female") }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end }}
```

保存模板并运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie
```

我们有两只狗，但有三种可能的情况显示`Intact`。让我们在`pets.go`中的切片中再添加一条狗，以涵盖所有三种情况。编辑`pets.go`并将第三只狗附加到切片：

pets.go

```go
. . .
func main() {
	dogs := []Pet{
		{
			Name:   "Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pitbull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
		{
			Name:	"Bruce Wayne",
			Sex:	"Male",
			Intact:	false,
			Age:	"3 years, 8 months",
			Breed:	"Chihuahua",
		},
	}
. . .
```


现在保存并运行`pets.go`：

```bash
go run pets.go
```


```
Output

---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd/Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

太好了，看起来和预期的一样。

现在让我们讨论模板函数，比如你刚刚使用的`eq`函数。

### 使用模板函数

沿着`eq`，还有其他函数用于比较字段值并返回布尔值：`gt`（），`ne`（！=），`le`（=）等。您可以通过以下两种方式之一来调用这些函数和[任何模板函数](https://pkg.go.dev/text/template#hdr-Functions)：

1. 写出函数名，后跟一个或多个参数，每个参数之间有一个空格。这就是你如何使用上面的`eq`：`eq .Sex “Female”`。
2. 先写一个参数，然后写一个管道（`|`），然后写函数名，再写任何其他参数。这类似于[Unix命令行上的命令管道]，并且在命令行上，您可以将[管道](https://pkg.go.dev/text/template#hdr-Pipelines)中的许多函数调用链接在一起，将一个调用的输出作为下一个调用的输入传递，依此类推。

因此，虽然模板中的`eq`比较被写为`eq .Sex “Female”`，但它也可以被写为`.Sex | eq “Female”`。这两个表达式是等价的。

让我们使用`len`函数在报告的顶部显示狗的数量。打开`pets.tmpl`并将以下内容添加到顶部：

pets.tmpl

```go
Number of dogs: {{ . | len -}}

{{ range . }}
. . .
```


你也可以写#1。

请注意右双花括号旁边的破折号（`-`）。这可以防止在操作后打印换行符（`\n`）。您也可以在双花括号（`{{-`）后面附加一个破折号，以在操作之前取消换行符。

保存模板并运行`pets.go`：

```bash
go run pets.go
```


```
Output
Number of dogs: 3
---
Name:  Jujube

Sex:   Female (spayed)

Age:   10 months

Breed: German Shepherd & Pitbull

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd & Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

由于`{{ . | len -}}`中的破折号，`Number of dogs`标签和第一只狗之间没有空行。

您可能已经注意到，`text/template`文档中的内置函数列表非常小。好消息是，你可以让任何Go函数在你的模板中可用，只要它返回一个值，或者如果第二个是`error`类型，则返回两个值。

### 在模板中使用Go函数

假设您想编写一个模板，它采用一个狗切片，只打印最后一个。您可以使用`slice`内置函数在模板中获取切片的子集，其工作方式类似于Go中的`mySlice[x:y]`。你可以写`{{ slice . 2 }}`来获取一个三项切片的最后一项，尽管`slice`函数返回另一个切片，而不是一个项。也就是说，`{{ slice . 2 }}`相当于`slice[2:]`，而不是`slice[2]`。(The函数也可以使用多个索引，例如`{{ slice . 0 2 }}`来获取切片`slice[0:2]`，但这里不会使用它。

但是如何在模板中引用切片的最后一个索引呢？`len`函数是可用的，但切片中的最后一项的索引为`len - 1`，不幸的是，您无法在模板中进行算术运算。

这就是自定义函数非常有用的地方。让我们写一个函数，减少一个整数，并使该函数可用于我们的模板。

但在此之前，让我们做一个新的模板。打开一个新文件`lastPet.tmpl`并粘贴以下内容：

lastPet.tmpl

```go
{{- range (len . | dec | slice . ) }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if ("Female" | eq .Sex) }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ .Breed }}
{{ end -}}
```


第一行中对`dec`函数的调用引用了您将要定义并传递给模板的自定义函数。在`main()`中的`pets.go`函数内，在`dogs`切片下方和调用`tmpl.Execute()`之前进行以下更改（突出显示）：

pets.go

```go
	. . .
	funcMap := template.FuncMap{
		"dec": func(i int) int { return i - 1 },
	}
	var tmplFile = “lastPet.tmpl”
	tmpl, err := template.New(tmplFile).Funcs(funcMap).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	. . .
```


首先，你声明了一个`FuncMap`，这是一个函数映射：键是模板可用的函数名，值是函数本身。这里的一个函数，`dec`，是一个内联提供[的匿名函数，](https://en.wikipedia.org/wiki/Anonymous_function)因为它很短。它接受一个整数，从中减去1，然后返回结果。

然后，您将更改模板文件名。最后，在对`Template.Funcs`的调用之前插入对`ParseFile`的调用，并将刚刚定义的`funcMap`传递给它。`Funcs`方法必须在`ParseFiles`之前调用。

在运行你的代码之前，让我们了解一下你的模板中的`range`动作发生了什么：

template_last_pet.go

```go
{{- range (len . | dec | slice . ) }}
```


您将获取`dogs`切片的长度，将其传递给自定义的`dec`函数以减去1，然后将其作为第二个参数传递给前面讨论的`slice`函数。因此，对于一个三狗切片，`range`操作相当于`{{- range (slice . 2) }}`。

保存`pets.go`并运行它：

```
go run pets.go
Output

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

看起来不错如果你想展示最后两只狗而不仅仅是最后一只呢？编辑`lastPet.tmpl`并将另一个对`dec`的调用添加到管道中：

lastPet.tmpl

```go
{{- range (len . | dec | dec | slice . ) }}
. . .
```


保存文件并再次运行`pets.go`：

```go
go run pets.go
```


```
Output

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd/Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

你可以想象如何改进`dec`函数，让它接受一个参数并更改它的名称，这样你就可以调用`minus 2`而不是`dec | dec`。

现在假设你想以不同的方式显示像Zephyr这样的混血狗，用&符号代替斜杠。你不需要自己写函数，因为`strings`包有一个，你可以从任何包中借用一个函数来在你的模板中使用。编辑`pets.go`以导入`strings`包并将其函数之一添加到`funcMap`：

pets.go

```go
package main

import (
	"os"
	"strings"
	"text/template"
)
. . .
func main() {
	. . .
	funcMap := template.FuncMap{
		"dec":     func(i int) int { return i - 1 },
		"replace": strings.ReplaceAll,
	}
	. . .
} // end main
```


您现在导入了`strings`包，并将其`ReplaceAll`函数添加到您的`funcMap`中，命名为`replace`。现在编辑`lastPet.tmpl`来使用这个函数：

lastPet.tmpl

```go
{{- range (len . | dec | dec | slice . ) }}
---
Name:  {{ .Name }}

Sex:   {{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if ("Female" | eq .Sex) }}spayed{{ else }}neutered{{ end }}{{ end }})

Age:   {{ .Age }}

Breed: {{ replace .Breed “/” “ & ” }}
{{ end -}}
```


保存文件并再次运行它：

```bash
go run pets.go
```


```
Output

---
Name:  Zephyr

Sex:   Male (intact)

Age:   13 years, 3 months

Breed: German Shepherd & Border Collie

---
Name:  Bruce Wayne

Sex:   Male (neutered)

Age:   3 years, 8 months

Breed: Chihuahua
```

和风的品种现在包含一个&而不是斜线。

您可以在`pets.go`中操作该字符串而不是您的模板，但数据的表示是模板的工作，而不是代码。

事实上，一些狗的数据已经包含了一些表示，也许它不应该。`Breed`字段将多个品种压缩到一个字符串中，并用斜杠将它们标点。这种单字符串模式可能会让数据输入者在数据库中引入不同的格式：`Labrador/Poodle`、`Labrador & Poodle`、`Labrador, Poodle`、`Labrador-Poodle mix`等。最好将`Breed`存储为字符串片段（`[]string`）而不是字符串，以避免这种格式歧义，并使按品种搜索更灵活，更容易呈现。然后你可以使用模板中的`strings.Join`函数来打印所有的品种，再加上`.Breed`字段的额外注释（`(purebred)`或`(mixed breed)`）。

自己试试

尝试修改您的代码和模板来实现这些更改。完成后，单击下面的解决方案以检查您的工作。

<details style="box-sizing: border-box; margin: 1em 0px 0px; display: block; background: rgb(249, 250, 254); border-radius: 16px; padding: 1em;"><summary style="box-sizing: border-box; margin: 0px; display: list-item; cursor: pointer; list-style: none; padding: 0px 1em 0px 0px; position: relative;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;">溶液</font></summary><div class="code-label" title="pets.go" style="box-sizing: border-box; margin: 1em 0px 0px; background-color: rgb(214, 220, 234); border-radius: 16px 16px 0px 0px; color: rgb(36, 51, 90); display: block; font-size: 16px; padding: 12px; position: relative; text-align: center; z-index: 2;"><font __transmart="" style="box-sizing: border-box; margin: 0px;"></font></div><div class="code-toolbar" style="box-sizing: border-box; margin: 0px 0px 1em; position: relative;"><pre class="language-go" style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: rgb(17, 25, 46); border-radius: 0px 0px 16px 16px; color: rgb(247, 248, 251); line-height: 22px; padding: 24px; box-shadow: rgba(17, 25, 46, 0.1) 0px 0px 0px 2px inset; display: block; overflow: auto; white-space: normal; overflow-wrap: normal;"><code style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: transparent; border-radius: 0px; color: inherit; line-height: 22px; padding: 0px; white-space: pre; text-shadow: none;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token function" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(255, 175, 140);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token number" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(11, 225, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token comment" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></code></pre><div class="toolbar" style="box-sizing: border-box; margin: 0px; position: absolute; right: 24px; top: 24px;"><div class="toolbar-item" style="box-sizing: border-box; margin: 0px; display: inline-block;"><button type="button" style="box-sizing: border-box; margin: 0px; font-family: inherit; font-size: 0.9em; line-height: normal; overflow: visible; text-transform: none; appearance: button; font-style: inherit; font-variant: inherit; font-weight: inherit; font-stretch: inherit; font-optical-sizing: inherit; font-kerning: inherit; font-feature-settings: inherit; font-variation-settings: inherit; background: rgb(105, 111, 176); border: 0px; cursor: pointer; user-select: none; border-radius: 10px; color: rgb(255, 255, 255); padding: 6px 8px; transition: color 0.25s ease 0s, background 0.25s ease 0s;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;"></font></button></div></div></div><div class="code-label" title="lastPet.tmpl" style="box-sizing: border-box; margin: 1em 0px 0px; background-color: rgb(214, 220, 234); border-radius: 16px 16px 0px 0px; color: rgb(36, 51, 90); display: block; font-size: 16px; padding: 12px; position: relative; text-align: center; z-index: 2;"><font __transmart="" style="box-sizing: border-box; margin: 0px;"></font></div><div class="code-toolbar" style="box-sizing: border-box; margin: 0px 0px 1em; position: relative;"><pre class="language-go" style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: rgb(17, 25, 46); border-radius: 0px 0px 16px 16px; color: rgb(247, 248, 251); line-height: 22px; padding: 24px; box-shadow: rgba(17, 25, 46, 0.1) 0px 0px 0px 2px inset; display: block; overflow: auto; white-space: normal; overflow-wrap: normal;"><code style="box-sizing: border-box; margin: 0px; font-family: monospace, monospace; font-size: 14px; background: transparent; border-radius: 0px; color: inherit; line-height: 22px; padding: 0px; white-space: pre; text-shadow: none;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><mark style="box-sizing: border-box; margin: 0px; background: rgb(41, 51, 77); border-radius: 2px; color: inherit; display: inline; line-height: calc(1.4em + 1px); padding: 2px 6px;"><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token string" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(177, 228, 144);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token builtin" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token number" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(11, 225, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token keyword" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(159, 221, 255);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></mark><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token operator" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span><span class="token punctuation" style="box-sizing: border-box; margin: 0px; text-shadow: none; background: none; border-radius: 0px; display: inline; font-weight: 400; padding: 0px; color: rgb(247, 248, 251);"></span></code></pre><div class="toolbar" style="box-sizing: border-box; margin: 0px; position: absolute; right: 24px; top: 24px;"><div class="toolbar-item" style="box-sizing: border-box; margin: 0px; display: inline-block;"><button type="button" style="box-sizing: border-box; margin: 0px; font-family: inherit; font-size: 0.9em; line-height: normal; overflow: visible; text-transform: none; appearance: button; font-style: inherit; font-variant: inherit; font-weight: inherit; font-stretch: inherit; font-optical-sizing: inherit; font-kerning: inherit; font-feature-settings: inherit; font-variation-settings: inherit; background: rgb(105, 111, 176); border: 0px; cursor: pointer; user-select: none; border-radius: 10px; color: rgb(255, 255, 255); padding: 6px 8px; transition: color 0.25s ease 0s, background 0.25s ease 0s;"><font __transmart="TARGET" class="transmart-tgt-font-container" style="box-sizing: border-box; margin: 0px 0px 0px 3px;"></font></button></div></div></div></details>

最后，让我们将相同的狗数据呈现到HTML文档中，看看为什么当模板的输出是HTML时，应该始终使用`html/template`包。

## 第5步-编写HTML模板

命令行工具可能使用`text/template`来整齐地打印输出，而其他一些批处理程序可能使用它来从某些数据创建结构良好的文件。但是Go模板通常也用于为Web应用程序呈现HTML页面。例如，流行的开源静态站点生成器[Hugo](https://gohugo.io/)使用`text/template`和`html/template`作为其模板的基础。

HTML有一些纯文本没有的特殊挑战。它使用尖括号来包装元素（`<td>`），使用&符号来标记实体（` `），使用引号来包装标记属性（`<a href=“https://www.digitalocean.com/”>`）。如果模板插入的任何数据包含这些字符，使用`text/template`包可能会导致HTML格式错误，或者更糟的是，代码注入。

`html/template`包不是这样的。这个包转义了这些有问题的字符，插入了它们的安全HTML等价物（实体）。数据中的&号变为`&`，左尖括号变为`<`，依此类推。

让我们使用前面相同的狗数据来创建一个HTML文档，但首先我们将继续使用`text/template`来显示它的危险。

打开`pets.go`，并将以下突出显示的文本添加到Repubbe的`Name`字段中：

pets.go

```go
        . . .
	dogs := []Pet{
		{
			Name:   "<script>alert(\"Gotcha!\");</script>Jujube",
			Sex:    "Female",
			Intact: false,
			Age:    "10 months",
			Breed:  "German Shepherd/Pit Bull",
		},
		{
			Name:   "Zephyr",
			Sex:    "Male",
			Intact: true,
			Age:    "13 years, 3 months",
			Breed:  "German Shepherd/Border Collie",
		},
		{
			Name:   "Bruce Wayne",
			Sex:    "Male",
			Intact: false,
			Age:    "3 years, 8 months",
			Breed:  "Chihuahua",
		},
	}
        . . .
```


现在在名为`petsHtml.tmpl`的新文件中创建一个HTML模板：

petsHtml.tmpl

```go
<p><strong>Pets:</strong> {{ . | len }}</p>
{{ range . }}
<hr />
<dl>
	<dt>Name</dt>
	<dd>{{ .Name }}</dd>
	<dt>Sex</dt>
	<dd>{{ .Sex }} ({{ if .Intact }}intact{{ else }}{{ if (eq .Sex "Female") }}spayed{{ else }}neutered{{ end }}{{ end }})</dd>
	<dt>Age</dt>
	<dd>{{ .Age }}</dd>
	<dt>Breed</dt>
	<dd>{{ replace .Breed “/” “ & ” }}</dd>
</dl>
{{ end }}
```


保存HTML模板。在运行`pets.go`之前，我们需要编辑`tmpFile` var，但我们也要编辑程序，将模板输出到文件而不是终端。打开`pets.go`并在`main()`函数中添加突出显示的代码：

pets.go

```go
	. . .
	funcMap := template.FuncMap{
		"dec":     func(i int) int { return i - 1 },
		"replace": strings.ReplaceAll,
	}
	var tmplFile = "petsHtml.tmpl"
	tmpl, err := template.New(tmplFile).Funcs(funcMap).ParseFiles(tmplFile)
	if err != nil {
		panic(err)
	}
	var f *os.File
	f, err = os.Create("pets.html")
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(f, dogs)
	if err != nil {
		panic(err)
	}
	err = f.Close()
	if err != nil {
		panic(err)
	}
} // end main
```


你打开一个新的名为`File`的`pets.html`，将它（而不是`os.Stdout`）传递给`tmpl.Execute`，然后在完成时关闭文件。

现在运行`go run pets.go`生成HTML文件。然后，在浏览器中打开此本地网页：

![gotcha.png](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/gotcha.png)

浏览器已运行注入的脚本。这就是为什么你不应该使用`text/template`包来生成HTML，特别是当你不能完全信任你的模板数据的来源时。

除了转义数据中的HTML字符外，`html/template`包的工作方式与`text/template`一样，并且具有相同的基本名称（`template`），这意味着要使模板注入安全，您所要做的就是将`text/template`导入替换为`html/template`。编辑`pets.go`并立即执行：

pets.go

```go
package main

import (
	"os"
	"strings"
	"html/template"
)
. . .
```


保存该文件并最后运行一次，重复`pets.html`。然后在浏览器中刷新HTML文件：

![gotcha-sanitized.png](https://ob-alioss.oss-cn-beijing.aliyuncs.com/markdown/gotcha-sanitized.png)

`html/template`包已经将注入的脚本呈现为网页中的文本。在你的文本编辑器中打开`pets.html`（或者在浏览器中查看页面源代码），看看第一条狗的名字：

pets.html

```markup
. . .
<dl>
        <dt>Name</dt>
        <dd>&lt;script&gt;alert(&#34;Gotcha!&#34;);&lt;/script&gt;Jujube</dd>
        <dt>Sex</dt>
        <dd>Female (spayed)</dd>
        <dt>Age</dt>
        <dd>10 months</dd>
        <dt>Breed</dt>
        <dd>German Shepherd &amp; Pit Bull</dd>
</dl>
. . .
```


html软件包取代了alibbe名字中的尖括号和引号字符，也取代了品种中的&符号。

## 结论

Go模板是一个方便的工具，可以将任何文本包装在任何数据周围。您可以在命令行工具中使用它们来格式化输出，在Web应用程序中使用它们来呈现HTML等。在本教程中，您使用Go内置的模板包从相同的数据打印格式良好的文本和HTML页面。要了解有关如何使用这些包的更多信息，请查看[`text/template`](https://pkg.go.dev/text/template)和[`html/template`](https://pkg.go.dev/html/template)的文档。，看看第一条狗的名字：

pets.html

```markup
. . .
<dl>
        <dt>Name</dt>
        <dd>&lt;script&gt;alert(&#34;Gotcha!&#34;);&lt;/script&gt;Jujube</dd>
        <dt>Sex</dt>
        <dd>Female (spayed)</dd>
        <dt>Age</dt>
        <dd>10 months</dd>
        <dt>Breed</dt>
        <dd>German Shepherd &amp; Pit Bull</dd>
</dl>
. . .
```


html软件包取代了alibbe名字中的尖括号和引号字符，也取代了品种中的&符号。

## 结论

Go模板是一个方便的工具，可以将任何文本包装在任何数据周围。您可以在命令行工具中使用它们来格式化输出，在Web应用程序中使用它们来呈现HTML等。在本教程中，您使用Go内置的模板包从相同的数据打印格式良好的文本和HTML页面。要了解有关如何使用这些包的更多信息，请查看[`text/template`](https://pkg.go.dev/text/template)和[`html/template`](https://pkg.go.dev/html/template)的文档。