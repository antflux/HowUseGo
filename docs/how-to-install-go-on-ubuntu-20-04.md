# 如何在 Ubuntu 20.04 上安装 Go

### 介绍

[Go](https://golang.org/)，有时被称为“Golang”，是一种开源编程语言，由Google于2012年发布。Google的意图是创建一种可以快速学习的编程语言。

自发布以来，Go在开发人员中非常受欢迎，并用于各种应用程序，从云或服务器端应用程序到人工智能和机器人。本教程概述了如何在Ubuntu 20.04服务器上下载并安装最新版本的Go（当前版本为1.16.7），构建著名的`Hello, World!`应用程序，并将Go代码转换为可执行的二进制文件以供将来使用。

使用[DigitalOcean App Platform](https://www.digitalocean.com/products/app-platform)从GitHub部署Go应用程序。让DigitalOcean专注于扩展您的应用。

## 先决条件

本教程需要一个Ubuntu 20.04系统，该系统配置了具有`sudo`权限的非root用户和防火墙，如[Ubuntu 20.04的初始服务器设置](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)中所述。

## 步骤1 -安装Go]

在这一步中，你将在你的服务器上安装Go。

首先，通过`ssh`连接到Ubuntu服务器：

```bash
ssh sammy@your_server_ip
```

副本

接下来，在Web浏览器中导航到[官方Go下载页面](https://golang.org/dl/)。从那里，复制当前二进制版本的tarball的URL。

在撰写本文时，最新的版本是go1.16.7。要在Ubuntu服务器（或任何Linux服务器）上安装Go，请复制以`linux-amd64.tar.gz`结尾的文件的URL。

现在你已经准备好了你的链接，首先确认你在主目录中：

```bash
cd ~
```

副本

然后使用`curl`检索tarball，确保将突出显示的URL替换为您刚刚复制的URL。`-O`标志确保输出到一个文件，`L`标志指示HTTPS重定向，因为这个链接是从Go网站获取的，在文件下载之前会重定向到这里：

```bash
curl -OL https://golang.org/dl/go1.16.7.linux-amd64.tar.gz
```

副本

要验证您下载的文件的完整性，请运行`sha256sum`命令并将其作为参数传递给文件名：

```bash
sha256sum go1.16.7.linux-amd64.tar.gz
```

副本

这将返回tarball的SHA256校验和：

```
Outputgo1.16.7.linux-amd64.tar.gz
7fe7a73f55ba3e2285da36f8b085e5c0159e9564ef5f63ee0ed6b818ade8ef04  go1.16.7.linux-amd64.tar.gz
```

如果校验和与下载页面上列出的校验和匹配，则此步骤已正确完成。

接下来，使用`tar`提取tarball。此命令包含`-C`标志，它指示tar在执行任何其他操作之前更改到给定的目录。这意味着解压缩的文件将被写入到`/usr/local/`目录，这是安装Go的推荐位置......`x`标志告诉`tar`进行解压缩，`v`告诉它我们想要详细的输出（正在解压缩的文件的列表），而`f`告诉它我们将指定一个文件名：

```bash
sudo tar -C /usr/local -xvf go1.16.7.linux-amd64.tar.gz
```

副本

虽然`/usr/local/go`是安装Go的推荐位置，但有些用户可能更喜欢或需要不同的路径。

## 步骤2 -设置Go路径

在此步骤中，您将在环境中设置路径。

首先，设置Go的根值，它告诉Go在哪里查找它的文件。 您可以通过编辑`.profile`文件来执行此操作，该文件包含每次登录时系统运行的命令列表。

使用您喜欢的编辑器打开`.profile`，它存储在您用户的主目录中。在这里，我们将使用`nano`：

```bash
sudo nano ~/.profile
```

副本

然后，将以下信息添加到文件的末尾：

sudo nano～/. profile文件

```
. . .
export PATH=$PATH:/usr/local/go/bin
```

将此信息添加到配置文件后，保存并关闭该文件。如果使用了`nano`，请按`CTRL+X`、`Y`和`ENTER`。

接下来，通过运行以下命令刷新您的配置文件：

```bash
source ~/.profile
```

副本

之后，检查是否可以通过运行`go`来执行`go version`命令：

```bash
go version
```

副本

此命令将输出系统上安装的任何版本的Go的版本号：

```
Outputgo version go1.16.7 linux/amd64
```

此输出确认您现在正在服务器上运行Go。

## 步骤3 -测试安装

现在Go已经安装完毕，并且为你的服务器设置了路径，你可以尝试创建你的`Hello, World!`应用程序，以确保Go可以正常工作。

首先，为你的Go工作区创建一个新的目录，这是Go构建文件的地方：

```bash
mkdir hello
```

副本

然后进入你刚刚创建的目录：

```bash
cd hello
```

副本

当导入包时，你必须通过代码自己的模块来管理依赖项。你可以通过使用`go.mod`命令创建一个`go mod init`文件来实现这一点：

```bash
go mod init your_domain/hello
```

副本

接下来，在你喜欢的文本编辑器中创建一个`Hello, World!` Go文件：

```bash
nano hello.go
```

副本

将以下文本添加到`hello.go`文件中：

hello.go

```
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

然后，按`CTRL+X`、`Y`和`ENTER`，保存并关闭文件。

测试你的代码，检查它是否打印了`Hello, World!`问候语：

```bash
go run .
```

副本

```
OutputHello, World!
```

`go run`命令编译并运行Go包，其中包含来自您创建的新`.go`目录和导入路径的`hello`源文件列表。但是，你也可以使用`go build`来创建一个可执行文件，这样可以保存一些时间。

## 步骤4 -将Go代码转换为二进制可执行文件

`go run`命令通常用作编译和运行需要频繁更改的程序的快捷方式。如果您已经完成了代码，并且希望运行它而不需要每次都编译它，您可以使用`go build`将代码转换为可执行的二进制文件。将代码生成为可执行的二进制文件可以将应用程序合并到单个文件中，其中包含执行该二进制文件所需的所有支持代码。一旦构建了二进制可执行文件，就可以运行`go install`将程序放置在可执行文件路径上，这样就可以从系统上的任何位置运行它。然后，您的程序将在提示时成功打印`Hello, World!`，并且您不需要再次编译程序。

试试看，然后运行`go build`。请确保从存储`hello.go`文件的同一目录运行此命令：

```bash
go build
```

副本

接下来，运行`./hello`以确认代码是否正常工作：

```bash
./hello
```

副本

```
OutputHello, World!
```

这确认您已经成功地将`hello.go`代码转换为可执行的二进制文件。但是，您只能从该目录中调用此二进制文件。如果你想从服务器上的不同位置运行这个程序，你需要指定二进制文件的完整文件路径来执行它。

键入二进制文件的完整文件路径很快就会变得乏味。作为替代，您可以运行`go install`命令。这与`go build`类似，但不是将可执行文件留在当前目录中，`go install`将其放在`$GOPATH/bin`目录中，这将允许您从服务器上的任何位置运行它。

为了成功运行`go install`，您必须将使用`go build`创建的二进制文件的安装路径传递给它。要查找二进制文件的安装路径，请运行以下`go list`命令：

```bash
go list -f ‘{{.Target}}’
```

副本

`go list`生成当前工作目录中存储的所有命名Go包的列表。`f`标志将导致`go list`根据您传递给它的任何包模板以不同的格式返回输出。此命令告诉它使用`Target`模板，这将导致`go list`返回存储在此目录中的任何包的安装路径：

```
Output‘/home/sammy/go/bin/hello
```

这是您使用`go build`创建的二进制文件的安装路径。这意味着安装此二进制文件的目录是`/home/sammy/go/bin/`。

将此安装目录添加到系统的shell路径中。确保更改此命令的突出显示部分，以反映系统上二进制文件的安装目录（如果不同）：

```bash
export PATH=$PATH:/home/sammy/go/bin/
```

副本

最后，运行`go install`来编译和安装包：

```bash
go install
```

副本

尝试运行此可执行二进制文件，只需输入`hello`

```bash
hello
```

副本

```
OutputHello, World!
```

如果你收到了`Hello, World!`输出，那么你已经成功地让你的Go程序从服务器上的特定和未指定的路径都可执行了。

## 结论

通过下载和安装最新的Go包并设置其路径，您现在有一个用于Go开发的系统。您可以在我们的“Go”标签中找到并订阅有关安装和使用Go的其他文章