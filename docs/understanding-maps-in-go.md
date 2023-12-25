# 理解 maps

大多数现代编程语言都有字典或哈希类型的概念。这些类型通常用于与映射到值的键成对存储数据。

在Go中，map数据类型是大多数程序员认为的字典类型。它将键映射到值，使键值对成为Go中存储数据的有用方式。映射是通过使用关键字`map`，后跟方括号`[ ]`中的键数据类型，再后跟值数据类型来构造的。然后将键值对放在两边的花括号内{ }：

```
map[key]value{}
```

在Go语言中，你通常使用map来保存相关数据，比如ID中包含的信息。一个包含数据的map看起来像这样：

```go
map[string]string{"name": "Sammy", "animal": "shark", "color": "blue", "location": "ocean"}
```

除了花括号之外，整个映射中还有连接键值对的冒号。冒号左边的单词是键。在Go语言中，键可以是任何类似的类型，比如`strings`、`ints`等等。

示例映射中的键是：

- `"name"`
- `"animal"`
- `"color"`
- `"location"`

冒号右边的单词是值。值可以是任何数据类型。示例映射中的值为：

- `"Sammy"`
- `"shark"`
- `"blue"`
- `"ocean"`

与其他数据类型一样，您可以将map存储在变量中，并将其打印出来：

```go
sammy := map[string]string{"name": "Sammy", "animal": "shark", "color": "blue", "location": "ocean"}
fmt.Println(sammy)
```

这将是你的输出：

```
Output  map[animal:shark color:blue location:ocean name:Sammy]
```

键-值对的顺序可能已经改变。在Go语言中，map数据类型是无序的。无论顺序如何，键值对都将保持不变，使您能够基于其关系含义访问数据。

## [地图项目](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#accessing-map-items)

您可以通过引用相关键来调用映射的值。由于映射提供了存储数据的键值对，它们在Go程序中可能是重要而有用的项目。

如果你想隔离Sammy的用户名，你可以通过调用`sammy["name"]`;保存你的map和相关键的变量来实现。让我们把它打印出来：

```go
fmt.Println(sammy["name"])
```

并接收值作为输出：

```
Output  Sammy
```

地图的行为就像数据库;不像切片那样调用整数来获取特定的索引值，而是将一个值赋给一个键，然后调用该键来获取其相关值。

通过调用键`"name"`，您将获得该键的值，即`"Sammy"`。

类似地，您可以使用相同的格式调用`sammy`映射中的其余值：

```go
fmt.Println(sammy["animal"])
// returns shark

fmt.Println(sammy["color"])
// returns blue

fmt.Println(sammy["location"])
// returns ocean
```

通过使用映射数据类型中的键-值对，可以引用键来检索值。

## [键和值](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#keys-and-values)

与某些编程语言不同，Go没有任何方便的函数来列出map的键或值。这方面的一个例子是Python的字典的`.keys()`方法。但是，它允许使用`range`操作符进行迭代：

```go
for key, value := range sammy {
 fmt.Printf("%q is the key for the value %q\n", key, value)
}
```

在Go中遍历map时，它会返回两个值。第一个值将是键，第二个值将是值。Go将创建这些具有正确数据类型的变量。在本例中，映射键是`string`，因此`key`也将是一个字符串。`value`也是一个字符串：

```
Output  "animal" is the key for the value "shark"
"color" is the key for the value "blue"
"location" is the key for the value "ocean"
"name" is the key for the value "Sammy"
```

要获得仅包含键的列表，可以再次使用范围操作符。你可以只声明一个变量来访问键：

```go
keys := []string{}

for key := range sammy {
 keys = append(keys, key)
}
fmt.Printf("%q", keys)
```

程序首先声明一个切片来存储你的键。

输出将仅显示您的map的键：

```
Output  ["color" "location" "name" "animal"]
```

同样，键没有排序。如果你想对它们进行排序，你可以使用[`sort.Strings`](https://golang.org/pkg/sort)包中的`sort`函数：

```go
sort.Strings(keys)
```

使用此函数，您将收到以下输出：

```
Output  ["animal" "color" "location" "name"]
```

您可以使用相同的模式仅检索映射中的值。在下一个示例中，您预分配切片以避免分配，从而使程序更高效：

```go
sammy := map[string]string{"name": "Sammy", "animal": "shark", "color": "blue", "location": "ocean"}

items := make([]string, len(sammy))

var i int

for _, v := range sammy {
 items[i] = v
 i++
}
fmt.Printf("%q", items)
```

首先声明一个切片来存储键;因为知道需要多少项，所以可以通过将切片定义为完全相同的大小来避免潜在的内存分配。然后声明索引变量。因为你不想要这个键，所以在开始循环时，你使用`_`操作符来忽略键的值。输出结果如下：

```
Output  ["ocean" "Sammy" "shark" "blue"]
```

要确定映射中的项目数量，您可以使用内置的`len`函数：

```go
sammy := map[string]string{"name": "Sammy", "animal": "shark", "color": "blue", "location": "ocean"}
fmt.Println(len(sammy))
```

输出显示地图中的项目数：

```
Output  4
```

尽管Go语言没有提供方便的函数来获取键和值，但它只需要几行代码就可以在需要时检索键和值。

## [检查是否存在](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#checking-existence)

当请求的键丢失时，Go语言中的Maps将返回map值类型的零值。因此，您需要另一种方法来区分存储的零和丢失的键。

让我们在map中查找一个你知道不存在的值，然后看看返回的值：

```go
counts := map[string]int{}
fmt.Println(counts["sammy"])
```

您将看到以下输出：

```
Output  0
```

即使键`sammy`不在map中，Go仍然返回值`0`。这是因为value数据类型是`int`，并且因为Go对所有变量都有一个零值，所以它返回了`0`的零值。

在许多情况下，这是不可取的，并会导致程序中的错误。当在map中查找值时，Go可以返回第二个可选值。第二个值是`bool`，如果找到了键，则为`true`，如果没有找到键，则为`false`。在Go中，这被称为`ok`习惯用法。尽管你可以随意命名捕获第二个参数的变量，但在Go语言中，你总是将其命名为`ok`：

```go
count, ok := counts["sammy"]
```

如果键`sammy`存在于`counts`映射中，则`ok`将是`true`。否则`ok`将为false。

你可以使用`ok`变量来决定在程序中做什么：

```go
if ok {
 fmt.Printf("Sammy has a count of %d\n", count)
} else {
 fmt.Println("Sammy was not found")
}
```

这将产生以下输出：

```
Output  Sammy was not found
```

在Go语言中，你可以将联合收割机变量声明和条件检查与if/else块结合起来。这允许您使用单个语句进行此检查：

```go
if count, ok := counts["sammy"]; ok {
 fmt.Printf("Sammy has a count of %d\n", count)
} else {
 fmt.Println("Sammy was not found")
}
```

当在Go中从map中检索值时，检查它的存在也是一个很好的做法，以避免程序中的错误。

## [修改贴图](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#modifying-maps)

映射是一种可变的数据结构，因此您可以修改它们。让我们看看在本节中添加和删除地图项。

### [添加和更改地图项目](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#adding-and-changing-map-items)

不使用方法或函数，您可以向映射添加键值对。您可以使用maps变量名，后跟方括号中的键值`[ ]`，并使用equal `=`运算符设置新值：

```go
map[key] = value
```

在实践中，你可以通过向一个名为`usernames`的映射添加一个键值对来看到这一点：

```go
usernames := map[string]string{"Sammy": "sammy-shark", "Jamie": "mantisshrimp54"}

usernames["Drew"] = "squidly"
fmt.Println(usernames)
```

输出将在映射中显示新的`Drew:squidly`键值对：

```
Output  map[Drew:squidly Jamie:mantisshrimp54 Sammy:sammy-shark]
```

因为map是无序返回的，所以这一对可以出现在map输出的任何地方。如果您稍后在程序文件中使用`usernames`映射，它将包含额外的键值对。

您还可以使用此语法修改分配给键的值。在这种情况下，您引用了一个现有的键，并向它传递了一个不同的值。

考虑一个名为`followers`的地图，它跟踪给定网络上用户的追随者。用户`"drew"`今天的关注者增加了，所以你需要更新传递给`"drew"`键的整数值。您将使用`Println()`函数来检查映射是否已修改：

```go
followers := map[string]int{"drew": 305, "mary": 428, "cindy": 918}
followers["drew"] = 342
fmt.Println(followers)
```

您的输出将显示`drew`的更新值：

```
Output  map[cindy:918 drew:342 mary:428]
```

您可以看到关注者的数量从整数值`305`跳到了`342`。

您可以使用此方法通过用户输入向映射添加键-值对。让我们编写一个名为`usernames.go`的快速程序，它在命令行上运行，并允许用户输入以添加更多名称和关联的用户名：

usernames.go

```go
package main

import (
 "fmt"
 "strings"
)

func main() {
 usernames := map[string]string{"Sammy": "sammy-shark", "Jamie": "mantisshrimp54"}

 for {
  fmt.Println("Enter a name:")

  var name string
  _, err := fmt.Scanln(&name)

  if err != nil {
   panic(err)
  }

  name = strings.TrimSpace(name)

  if u, ok := usernames[name]; ok {
   fmt.Printf("%q is the username of %q\n", u, name)
   continue
  }

  fmt.Printf("I don't have %v's username, what is it?\n", name)

  var username string
  _, err = fmt.Scanln(&username)

  if err != nil {
   panic(err)
  }

  username = strings.TrimSpace(username)

  usernames[name] = username

  fmt.Println("Data updated.")
 }
}
```

在`usernames.go`中，首先定义原始贴图。然后设置一个循环来遍历这些名称。您请求用户输入一个名称并声明一个变量来存储它。接下来，检查是否有错误;如果有，程序将退出并出现异常。因为`Scanln`捕获了整个输入，包括回车，所以需要从输入中删除所有空格;可以使用`strings.TrimSpace`函数来完成。

`if`块检查名称是否存在于映射中并打印反馈。如果名称存在，则继续返回到循环的顶部。如果名称不在地图中，它会向用户提供反馈，然后要求为关联名称提供新的用户名。程序再次检查是否有错误。如果没有错误，它会删除回车符，将username值赋给name键，然后打印数据已更新的反馈。

让我们在命令行上运行程序：

```bash
go run usernames.go
```

您将看到以下输出：

```
Output  Enter a name:
Sammy
"sammy-shark" is the username of "Sammy"
Enter a name:
Jesse
I don't have Jesse's username, what is it?
JOctopus
Data updated.
Enter a name:
```

当你完成测试，按`CTRL + C`退出程序。

这显示了如何以交互方式修改贴图。对于这个特殊的程序，只要你用`CTRL + C`退出程序，你就会丢失所有的数据，除非你实现一种处理阅读和写入文件的方法。

总之，您可以使用`map[key] = value`语法向映射添加项或修改值。

### [删除地图项目](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#deleting-map-items)

就像您可以在映射数据类型中添加键值对和更改值一样，您也可以删除映射中的项。

要从映射中删除键值对，可以使用内置函数`delete()`。第一个参数是要从中删除的映射。第二个参数是您要删除的键：

```go
delete(map, key)
```

让我们定义一个权限映射：

```go
permissions := map[int]string{1: "read", 2: "write", 4: "delete", 8: "create", 16:"modify"}
```

您不再需要`modify`权限，因此将从地图中删除它。然后，您将打印出地图以确认它已被删除：

```go
permissions := map[int]string{1: "read", 2: "write", 4: "delete", 8: "create", 16: "modify"}
delete(permissions, 16)
fmt.Println(permissions)
```

输出将确认删除：

```
Output  map[1:read 2:write 4:delete 8:create]
```

行`delete(permissions, 16)`从`16:"modify"`映射中删除键值对`permissions`。

如果你想清除一个map的所有值，你可以通过将它设置为一个相同类型的空map来实现。这将创建一个新的空映射来使用，而旧的映射将被垃圾收集器从内存中清除。

让我们移除`permissions` map中的所有项目：

```go
permissions = map[int]string{}
fmt.Println(permissions)
```

输出显示，现在你有一个没有键值对的空map：

```
Output  map[]
```

因为映射是可变的数据类型，所以它们可以被添加、修改，以及删除和清除项。

## [结论](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go#conclusion)

本教程探索了Go中的map数据结构。映射由键-值对组成，并提供了一种不依赖索引存储数据的方法。这允许我们根据值的含义和与其他数据类型的关系来检索值。
