# 理解数组和切片

### [介绍](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#introduction)

在Go语言中，数组和切片是由有序的元素序列组成的[数据结构](https://en.wikipedia.org/wiki/Data_structure)。当您想要处理许多相关值时，这些数据集合非常有用。它们使您能够将属于一起的数据保存在一起，压缩代码，并一次对多个值执行相同的方法和操作。

虽然Go语言中的数组和切片都是有序的元素序列，但两者之间有很大的区别。Go语言中的数组是一种[数据结构](https://en.wikipedia.org/wiki/Data_structure)，它由一个有序的元素序列组成，其容量在创建时定义。一旦一个数组分配了它的大小，它的大小就不能再改变了。另一方面，切片是数组的可变长度版本，为使用这些数据结构的开发人员提供了更大的灵活性。切片构成了你在其他语言中所认为的数组。

考虑到这些差异，在某些特定情况下，您可以使用其中一个。如果你是Go的新手，确定何时使用它们可能会令人困惑：尽管切片的多功能性使它们在大多数情况下成为更合适的选择，但在某些特定的情况下，数组可以优化程序的性能。

本文将详细介绍数组和切片，这将为您提供必要的信息，以便在这些数据类型之间做出适当的选择。此外，您还将回顾声明和使用数组和切片的最常用方法。本教程将首先提供数组的描述以及如何操作它们，然后解释切片以及它们的区别。

## [阵列](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#arrays)

数组是具有一定数量元素的集合数据结构。因为数组的大小是静态的，所以数据结构只需要分配一次内存，而不是可变长度的数据结构，它必须动态分配内存，以便将来可以变得更大或更小。虽然数组的固定长度可能会使它们在使用时有些僵硬，但一次性内存分配可以提高程序的速度和性能。因此，开发人员在优化程序时通常使用数组，因为数据结构永远不需要可变数量的元素。

### [定义数组](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#defining-an-array)

数组是通过在方括号`[ ]`中声明数组的大小，后跟元素的数据类型来定义的。Go语言中的数组必须所有元素都是相同的[数据类型](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go)。在数据类型之后，您可以在大括号`{ }`中声明数组元素的各个值。

下面是声明数组的一般模式：

```go
[capacity]data_type{element_values}
```

注意：重要的是要记住，每个新数组的声明都创建一个独特的类型。因此，尽管`[2]int`和`[3]int`都有整数元素，但它们的不同长度使它们的数据类型不兼容。

如果不声明数组元素的值，则默认值为零值，这意味着数组的元素将为空。对于整数，这由`0`表示，对于字符串，这由空字符串表示。

例如，下面的数组`numbers`有三个整数元素还没有值：

```go
var numbers [3]int
```

如果打印`numbers`，则会收到以下输出：

```
Output [0 0 0]
```

如果您希望在创建数组时指定元素的值，请将值放在大括号中。一个具有设置值的字符串数组看起来像这样：

```go
[4]string{"blue coral", "staghorn coral", "pillar coral", "elkhorn coral"}
```

你可以将一个数组存储在一个变量中并打印出来：

```go
coral := [4]string{"blue coral", "staghorn coral", "pillar coral", "elkhorn coral"}
fmt.Println(coral)
```

运行前面几行的程序会给你给予以下输出：

```
Output [blue coral staghorn coral pillar coral elkhorn coral]
```

请注意，在打印数组时，数组中的元素之间没有分界线，因此很难区分一个元素的结束位置和另一个元素的开始位置。因此，有时使用`fmt.Printf`函数会很有帮助，它可以在将字符串打印到屏幕之前对其进行格式化。在此命令中提供`%q`动词，以指示函数在值周围加上引号：

```go
fmt.Printf("%q\n", coral)
```

这将导致以下结果：

```
Output ["blue coral" "staghorn coral" "pillar coral" "elkhorn coral"]
```

现在每个项目都被引用。`\n`动词指示格式化程序在末尾添加一个行返回。

有了如何声明数组及其组成的一般概念，现在可以继续学习如何用索引号指定数组中的元素。

### [索引数组（和切片）](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#indexing-arrays-and-slices)

数组中的每个元素（以及切片）都可以通过索引单独调用。每个元素对应于索引号，该索引号是从索引号`int`开始并向上计数的`0`值。

我们将在下面的示例中使用数组，但您也可以使用切片，因为它们在索引方式上是相同的。

对于数组`coral`，索引分解如下所示：

| "蓝色珊瑚" | 鹿角珊瑚 | "珊瑚虫" | “埃尔克霍恩珊瑚” |
| ---------- | -------- | -------- | ---------------- |
| 0          | 1        | 2        | 3                |

第一个元素，字符串`"blue coral"`，从索引`0`开始，切片在索引`3`结束，元素`"elkhorn coral"`。

因为切片或数组中的每个元素都有一个对应的索引号，所以我们能够以与其他顺序数据类型相同的方式访问和操作它们。

现在我们可以通过引用其索引号来调用切片的离散元素：

```go
fmt.Println(coral[1])
```

```
Output staghorn coral
```

如前表所示，此切片的索引号范围为`0-3`。因此，要单独调用任何元素，我们将像这样引用索引号：

```go
coral[0] = "blue coral"
coral[1] = "staghorn coral"
coral[2] = "pillar coral"
coral[3] = "elkhorn coral"
```

如果我们调用数组`coral`，其索引号any大于`3`，则它将超出范围，因为它将无效：

```go
fmt.Println(coral[18])
```

```
Output panic: runtime error: index out of range
```

当索引数组或切片时，必须始终使用正数。不像某些语言允许你用负数向后索引，在Go中这样做会导致错误：

```go
fmt.Println(coral[-1])
```

```
Output invalid array index -1 (index must be non-negative)
```

我们可以使用`+`操作符将数组或切片中的字符串元素与其他字符串连接起来：

```go
fmt.Println("Sammy loves " + coral[0])
```

```
Output Sammy loves blue coral
```

我们能够将索引号为`0`的字符串元素与字符串`"Sammy loves "`连接起来。

使用与数组或切片中的元素相对应的索引号，我们可以离散地访问每个元素并使用这些元素。为了演示这一点，我们接下来将看看如何修改某个索引处的元素。

### [修改元件](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#modifying-elements)

我们可以使用索引来改变数组或切片中的元素，方法是将索引编号的元素设置为不同的值。这使我们能够更好地控制切片和数组中的数据，并允许我们以编程方式操作单个元素。

如果我们想将数组`1`的索引`coral`处的元素的字符串值从`"staghorn coral"`更改为`"foliose coral"`，我们可以这样做：

```go
coral[1] = "foliose coral"
```

现在当我们打印`coral`时，数组将不同：

```go
fmt.Printf("%q\n", coral)
```

```
Output ["blue coral" "foliose coral" "pillar coral" "elkhorn coral"]
```

现在您已经知道了如何操作数组或切片的各个元素，让我们来看看两个函数，它们将在处理集合数据类型时为您提供给予更大的灵活性。

### [使用`len()`计数元素](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#counting-elements-with-len)

在Go中，`len()`是一个内置函数，用于帮助您处理数组和切片。与字符串一样，您可以使用`len()`并将数组或切片作为参数传入来计算数组或切片的长度。

例如，要查找`coral`数组中有多少个元素，可以使用：用途：

```go
len(coral)
```

如果你打印出数组`coral`的长度，你会收到下面的输出：

```
Output 4
```

这给出了`4`数据类型中的数组`int`的长度，这是正确的，因为数组`coral`有四个项目：

```go
coral := [4]string{"blue coral", "foliose coral", "pillar coral", "elkhorn coral"}
```

如果你创建了一个包含更多元素的整数数组，你也可以在上面使用`len()`函数：

```go
numbers := [13]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}
fmt.Println(len(numbers))
```

这将产生以下输出：

```
Output 13
```

尽管这些示例数组的元素相对较少，但在确定非常大的数组中有多少元素时，`len()`函数特别有用。

接下来，我们将介绍如何向集合数据类型添加元素，并演示由于数组的固定长度，追加这些静态数据类型将如何导致错误。

### [使用`append()`删除元素](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#appending-elements-with-append)

`append()`是Go语言中的一个内置方法，用于向集合数据类型添加元素。但是，此方法在与数组一起使用时将不起作用。如前所述，数组与切片的主要区别在于数组的大小不能修改。这意味着，虽然可以更改数组中元素的值，但在定义数组之后，不能使其变大或变小。

让我们考虑一下你的`coral`数组：

```go
coral := [4]string{"blue coral", "foliose coral", "pillar coral", "elkhorn coral"}
```

假设你想把项目`"black coral"`添加到这个数组中。如果您尝试通过键入以下命令对数组使用`append()`函数：

```go
coral = append(coral, "black coral")
```

您将收到一个错误作为输出：

```
Output first argument to append must be slice; have [4]string
```

为了解决这个问题，让我们学习更多关于切片数据类型，如何定义切片，以及如何从数组转换为切片。

## [切片](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#slices)

切片是Go语言中的一种数据类型，它是一个可变的、有序的元素序列。由于切片的大小是可变的，因此在使用它们时有更大的灵活性;当处理将来可能需要扩展或收缩的数据集合时，使用切片将确保您的代码在尝试操作集合长度时不会出错。在大多数情况下，与数组相比，这种可变性是值得的，有时切片需要重新分配内存。当你需要存储大量的元素或重叠元素，并且你希望能够方便地修改这些元素时，你可能会希望使用切片数据类型。

### [定义切片](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#defining-a-slice)

切片是通过声明数据类型来定义的，数据类型前面是一组空的方括号（`[]`）和花括号（`{}`）之间的元素列表。您会注意到，与需要在括号之间添加`int`来声明特定长度的数组相反，切片在括号之间没有任何内容，表示其可变长度。

让我们创建一个包含字符串数据类型元素的切片：

```go
seaCreatures := []string{"shark", "cuttlefish", "squid", "mantis shrimp", "anemone"}
```

当我们打印出切片时，我们可以看到切片中的元素：

```go
fmt.Printf("%q\n", seaCreatures)
```

这将导致以下结果：

```
Output ["shark" "cuttlefish" "squid" "mantis shrimp" "anemone"]
```

如果你想创建一个特定长度的切片而不填充集合的元素，你可以使用内置的`make()`函数：

```go
oceans := make([]string, 3)
```

如果你打印这个切片，你会得到：

```
Output ["" "" ""]
```

如果你想为某个容量预先分配内存，你可以传入第三个参数到`make()`：

```go
oceans := make([]string, 3, 5)
```

这将产生长度为`3`且预分配容量为`5`个元素的归零切片。

您现在知道如何声明切片了。然而，这还不能解决我们之前使用`coral`数组时遇到的错误。要在`append()`中使用`coral`函数，您首先必须学习如何分割数组的部分。

### [将数组切片](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#slicing-arrays-into-slices)

通过使用索引号来确定开始和结束，您可以调用数组中的值的子部分。这被称为对数组进行切片，你可以通过创建一个由冒号分隔的索引号范围来实现，格式为`[first_index:second_index]`。然而，重要的是要注意，当对数组进行切片时，结果是切片，而不是数组。

假设你只想打印`coral`数组的中间项，而不打印第一个和最后一个元素。你可以通过创建一个从索引`1`开始到索引`3`之前结束的切片来实现这一点：

```go
fmt.Println(coral[1:3])
```

使用这一行运行程序将产生以下结果：

```
Output [foliose coral pillar coral]
```

创建切片时，如`[1:3]`中所示，第一个数字是切片开始的位置（包括），第二个数字是第一个数字和您想要检索的元素总数的总和：

```
array[starting_index : (starting_index + length_of_slice)]
```

在本例中，您调用了第二个元素（或索引1）作为起点，总共调用了两个元素。计算结果如下所示：

```
array[1 : (1 + 2)]
```

这就是你如何得出这个符号的：

```go
coral[1:3]
```

如果要将数组的开头或结尾设置为切片的起点或终点，可以省略`array[first_index:second_index]`语法中的一个数字。例如，如果您想打印数组`coral`的前三个项目（即`"blue coral"`、`"foliose coral"`和`"pillar coral"`），则可以通过键入以下内容来实现：

```go
fmt.Println(coral[:3])
```

这将打印：

```
Output [blue coral foliose coral pillar coral]
```

这将打印数组的开头，在索引`3`之前停止。

要包含数组末尾的所有项，您需要反转语法：

```go
fmt.Println(coral[1:])
```

这将给予以下切片：

```
Output [foliose coral pillar coral elkhorn coral]
```

本节讨论了通过划分子节来调用数组的各个部分。接下来，您将学习如何使用切片将整个数组转换为切片。

### [从数组转换为切片](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#converting-from-an-array-to-a-slice)

如果你创建了一个数组，并决定你需要它有一个可变的长度，你可以把它转换成一个切片。要将数组转换为切片，请使用您在本教程的将数组切片为切片步骤中学习的切片过程，但这一次选择整个切片时，请忽略决定端点的两个索引号：

```go
coral[:]
```

请记住，您不能将变量`coral`转换为切片本身，因为一旦在Go中定义了变量，就不能更改其类型。要解决这个问题，可以将数组的整个内容作为切片复制到一个新变量中：

```go
coralSlice := coral[:]
```

如果打印`numbers`，则会收到以下输出：

```
Output [blue coral foliose coral pillar coral elkhorn coral]
```

现在，尝试像数组部分那样添加`black coral`元素，使用`append()`和新转换的切片：

```go
coralSlice = append(coralSlice, "black coral")
fmt.Printf("%q\n", coralSlice)
```

这将输出添加了元素的切片：

```
Output ["blue coral" "foliose coral" "pillar coral" "elkhorn coral" "black coral"]
```

我们也可以在一个`append()`语句中添加多个元素：

```go
coralSlice = append(coralSlice, "antipathes", "leptopsammia")
```

```
Output ["blue coral" "foliose coral" "pillar coral" "elkhorn coral" "black coral" "antipathes" "leptopsammia"]
```

要将两个切片联合收割机在一起，可以使用`append()`，但必须使用`...`扩展语法扩展第二个参数以进行追加：

```go
moreCoral := []string{"massive coral", "soft coral"}
coralSlice = append(coralSlice, moreCoral...)
```

```
Output ["blue coral" "foliose coral" "pillar coral" "elkhorn coral" "black coral" "antipathes" "leptopsammia" "massive coral" "soft coral"]
```

现在你已经学会了如何将一个元素附加到你的切片上，我们将看看如何删除一个元素。

### [从切片中删除元素](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#removing-an-element-from-a-slice)

与其他语言不同，Go没有提供任何内置函数来从切片中删除元素。物品需要通过切片从切片中移除。

要删除一个元素，必须将该元素之前的项切掉，将该元素之后的项切掉，然后将这两个新的切片附加在一起，而不添加要删除的元素。

如果`i`是要删除的元素的索引，则此过程的格式如下所示：

```go
slice = append(slice[:i], slice[i+1:]...)
```

从`coralSlice`开始，让我们删除项目`"elkhorn coral"`。此项目位于索引位置`3`。

```go
coralSlice := []string{"blue coral", "foliose coral", "pillar coral", "elkhorn coral", "black coral", "antipathes", "leptopsammia", "massive coral", "soft coral"}

coralSlice = append(coralSlice[:3], coralSlice[4:]...)

fmt.Printf("%q\n", coralSlice)
```

```
Output ["blue coral" "foliose coral" "pillar coral" "black coral" "antipathes" "leptopsammia" "massive coral" "soft coral"]
```

现在索引位置`3`的元素，字符串`"elkhorn coral"`，不再在我们的切片`coralSlice`中。

我们也可以用同样的方法删除一个范围。假设我们不仅要删除项目`"elkhorn coral"`，还要删除项目`"black coral"`和`"antipathes"`。我们可以在表达式中使用一个范围来实现这一点：

```go
coralSlice := []string{"blue coral", "foliose coral", "pillar coral", "elkhorn coral", "black coral", "antipathes", "leptopsammia", "massive coral", "soft coral"}

coralSlice = append(coralSlice[:3], coralSlice[6:]...)

fmt.Printf("%q\n", coralSlice)
```

这段代码将从切片中取出索引`3`、`4`和`5`：

```
Output ["blue coral" "foliose coral" "pillar coral" "leptopsammia" "massive coral" "soft coral"]
```

现在您已经知道了如何在切片中添加和删除元素，让我们看看如何测量切片在任何给定时间可以容纳的数据量。

### [使用`cap()`测量切片的容量](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#measuring-the-capacity-of-a-slice-with-cap)

由于切片的长度是可变的，因此`len()`方法不是确定此数据类型大小的最佳选择。相反，您可以使用`cap()`函数来了解切片的容量。这将显示一个切片可以容纳多少元素，这取决于已经为切片分配了多少内存。

注意：因为数组的长度和容量总是相同的，所以`cap()`函数对数组无效。

`cap()`的一个常见用法是创建一个包含预设数量元素的切片，然后以编程方式填充这些元素。这避免了使用`append()`添加超出当前分配的容量的元素可能发生的潜在不必要的分配。

让我们假设我们想要制作一个数字列表，从`0`到`3`。我们可以在一个循环中使用`append()`来实现这一点，或者我们可以先预分配切片，然后使用`cap()`来循环填充值。

首先，我们可以使用`append()`：

```go
numbers := []int{}
for i := 0; i < 4; i++ {
 numbers = append(numbers, i)
}
fmt.Println(numbers)
```

```
Output [0 1 2 3]
```

在这个例子中，我们创建了一个切片，然后创建了一个`for`循环，它将重复四次。每次迭代都将循环变量`i`的当前值附加到`numbers`切片的索引中。但是，这可能会导致不必要的内存分配，从而降低程序的速度。当添加到一个空片时，每次调用append时，程序都会检查片的容量。如果添加的元素使切片超出了这个容量，程序将分配额外的内存来解决这个问题。这会在程序中产生额外的开销，并可能导致执行速度变慢。

现在让我们通过预先分配一定的长度/容量来填充切片，而不使用`append()`：

```go
numbers := make([]int, 4)
for i := 0; i < cap(numbers); i++ {
 numbers[i] = i
}

fmt.Println(numbers)
```

```
Output [0 1 2 3]
```

在这个例子中，我们使用`make()`创建一个切片，并让它预先分配`4`个元素。然后，我们在循环中使用`cap()`函数来遍历每个清零的元素，填充每个元素，直到它达到预先分配的容量。在每个循环中，我们将循环变量`i`的当前值放入`numbers`切片的索引中。

虽然`append()`和`cap()`策略在功能上是等效的，但`cap()`示例避免了使用`append()`函数所需的任何额外内存分配。

### [构造多维切片](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#constructing-multidimensional-slices)

您还可以将由其他切片组成的切片定义为元素，每个带括号的列表都包含在父切片的大括号内。像这样的切片集合称为多维切片。这些可以被认为是描述多维坐标;例如，每个六个元素长的五个切片的集合可以表示水平长度为五，垂直高度为六的二维网格。

让我们看看下面的多维切片：

```go
seaNames := [][]string{{"shark", "octopus", "squid", "mantis shrimp"}, {"Sammy", "Jesse", "Drew", "Jamie"}}
```

要访问这个切片中的元素，我们必须使用多个索引，每个索引对应于结构的每个维度：

```go
fmt.Println(seaNames[1][0])
fmt.Println(seaNames[0][0])
```

在前面的代码中，我们首先标识索引`0`处的切片的索引`1`处的元素，然后我们指示索引`0`处的切片的索引`0`处的元素。这将产生以下结果：

```
Output Sammy
shark
```

以下是其余单个元素的索引值：

```go
seaNames[0][0] = "shark"
seaNames[0][1] = "octopus"
seaNames[0][2] = "squid"
seaNames[0][3] = "mantis shrimp"

seaNames[1][0] = "Sammy"
seaNames[1][1] = "Jesse"
seaNames[1][2] = "Drew"
seaNames[1][3] = "Jamie"
```

在使用多维切片时，重要的是要记住，为了访问相关嵌套切片中的特定元素，您需要引用多个索引号。

## [结论](https://www.digitalocean.com/community/tutorials/understanding-arrays-and-slices-in-go#conclusion)

在本教程中，您学习了在Go中使用数组和切片的基础。您通过多个练习演示了数组的长度是固定的，而切片的长度是可变的，并发现了这种差异如何影响这些数据结构的情景使用。
