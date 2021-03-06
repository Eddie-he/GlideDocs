# 入门指南

在你安装了Glide之后，这是一个快速入门指南。

## 检测项目依赖

Glide可以检测项目的依赖关系，并为你创建一个初始的 `glide.yaml`文件。此检测可以从 Godep、GPM、Gom和GB中导入配置。进入项目的顶级目录并执行以下命名即可：

    $ glide init

完成之后将会生成一个`glide.yaml`文件。你可以打开此文件，甚至编辑它以添加诸如版本之类的信息。

运行`glide init`还会询问您是否要使用向导来发现有关依赖关系版本的信息，并使用版本或范围。每个决定都是交互的。

## 更新依赖

要获取依赖项并将其设置为`glide.yaml`文件中指定的任何版本，请使用以下命令：

    $ glide up

`up`是`update`的简写。这将获取`glide.yaml`文件中指定的任何依赖项，依次遍历依赖关系树，以确保依赖包的依赖都被获取，并将其设置为适当的版本。遍历依赖关系树时将确定版本，导入 Godep、GPM、Gom 和 GB 中配置的依赖。

获取的依赖包全部放在项目根目录下的`vendor/`文件夹中。如果你使用Go 1.6+或Go 1.5，启用了Go 1.5 的`vendor`功能，那么`go`工具链将在搜索`GOPATH`或`GOROOT`之前使用这里的依赖包。

Glide 之后会创建一个`glide.lock`文件。这个文件包含了特定`commit id`的整个依赖关系树。这个文件，我们稍后会看到，可以用来重新创建所使用的确定的依赖关系树和版本。

如果要从依赖关系中删除嵌套的`vendor/`目录，请使用`--strip-vendor`或`-v`参数。

### 依赖扁平化

Glide 获取的所有依赖都放置在项目顶级目录下的`vendor/`中。Go 支持每个包都有一个`vendor/`目录，但 Glide 仅使用顶级文件夹有两个原因：

1. 每个导入都会被编译成二进制文件。如果同一个依赖包出现在三个`vendor/`目录中，那么它会被编译三次。这样会迅速导致二进制文件体积变大。
2. 一个`vendor/`目录中的依赖包创建的实例与其他位置的依赖包不兼容，即便它们是相同的版本。Go 将它们视作不同的包，因为它们位于不同的位置。这对 数据库驱动、日志记录器 和 其他很多事情都是个问题。如果你 [尝试将一个个位置的包创建的实例传递给另一个位置的包将发生错误](https://github.com/mattfarina/golang-broken-vendor)

如果一个依赖包有它自己的 `vendor/`目录，Glide 默认情况下不会去删除它。`go`工具链的做法是如果这些嵌套的依赖包存在则会使用它。要删除它们，请在`up`或`install`命令上使用`--strip-vendor` 或者`-v`参数。

## 安装依赖

如果你想安装一个项目所需要的依赖包，请使用`install`命令：

    $ glide install

这个命令做了以下两者之一：

* 如果一个glide.lock文件存在，它将检索（`vendor/`目录中缺失的）依赖包，并将其设置为`glide.lock`文件中的确切版本。获取依赖包设置版本是并发的，因此操作相当快。
* 如果没有`glide.lock`,那么`update`将会被执行。

如果你不管理项目的依赖包的版本，但是需要安装依赖包，则应使用`install`命令。

## 添加更多依赖

Glide可以帮助您使用`get`命令为`glide.yaml`文件添加更多的依赖项。

    $ glide get github.com/Masterminds/semver

`get`命令类似于`go get`,将依赖包下载到`vendor/`目录中，并将它们添加到`glide.yaml`文件中。此命令可以获取一个或者多个依赖包。

`get`命令也可以使用版本。

    $ glide get github.com/Masterminds/semver#~1.2.0

`#`用作依赖包和要使用的版本之间的分隔符。版本可以是一个语义版本、版本范围、`branch`、`tag`、或者 `commit id`。

如果没有指定版本或范围，并且依赖包使用语义版本，那么Glide将询问你是否要使用它们。
