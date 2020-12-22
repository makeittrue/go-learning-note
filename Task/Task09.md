# Task09: 包管理

![关于Go Modules，看这一篇文章就够了](https://pic4.zhimg.com/v2-b8e88a2e8e55d8f91b25b87436259ba4_1440w.jpg?source=172ae18b)

## 1.Go Modules是什么？

简单地说，Go Modules是语义化版本管理的依赖项的包管理工具

它解决了GOPATH存在的缺陷，更重要的是，它是由Go官方出品的。

Go 1.13版本之后新的包管理器Modules趋于成熟，目前越来越多的开源项目已经支持Go Modules，典型的如etcd。

### 1.1 GOPATH的缺陷

几乎所有的包管理工具在Go 1.11版本之前都绕不开GOPATH这个环境变量。GOPATH主要用来放置项目依赖包的源代码，GOPATH不区分项目，代码中任何import的路径均从GOPATH为根目录开始；但现在GOPATH已经不够用了。

### 1.2 不区分依赖项版本

当有多个项目时，不同项目对于依赖库的版本需求不一致时，无法在一个GOPATH下面放置不同版本的依赖项。典型的例子：当有多项目时候，A项目依赖C 1.0.0，B项目依赖C 2.0.0，由于没有依赖项版本的概念，C 1.0.0和C 2.0.0无法同时在GOPATH下共存，解决办法是分别为A项目和B项目设置GOPATH，将不同版本的C源代码放在两个GOPATH中，彼此独立（编译时切换），或者C 1.0.0和C 2.0.0两个版本更改包名。无论哪种解决方法，都需要人工判断更正，不具备便利性。

### 1.3 依赖项列表无法数据化

在Go Modules之前，没有任何语义化的数据可以知道当前项目的所有依赖项，需要手动找出所有依赖。对项目而言，需要将所有的依赖项全部放入源代码控制中。如果剔除某个依赖，需要在源代码中手工确认某个依赖是否剔除。

为了解决GOPATH的缺陷，Go官方和社区推出许多解决方案，比如godep、govendor、glide等，这些工具要么未彻底解决GOPATH存在的问题要么使用起来繁冗，这才催生了Go Modules的出现。

## 2. Go Modules的使用方法

### 2.1 环境变量

首先需要设置环境变量，可以使用go env命令查看当前配置。

```shell
$ go env
GO111MODULE="auto"
GOPROXY="https://proxy.golang.org,direct"
GONOPROXY=""
GOSUMDB="sum.golang.org"
GONOSUMDB=""
GOPRIVATE=""
```

如果需要更改 GO111MODULE ，可以使用go env命令

```shell
go env -w GO111MODULE=on
```

GO111MODULE

- auto：只要项目包含了 go.mod 文件的话启用 Go modules，目前在 Go1.11 至 Go1.14 中仍然是默认值。
- on：启用 Go modules，推荐设置，将会是未来版本中的默认值。
- off：禁用 Go modules，不推荐设置。

**GOPROXY**

此环境变量主要用于设计Go Module的代理

**GOSUMDB**

此环境变量用于在拉取模块的时候保证模块版本数据的一致性。

### 2.2 初始化模块

Go Modules的使用方法比较灵活，在目录下包含go.mod文件即可

首先通过如下命令创建一个新的Module

```shell
go mod init [module name]go
```

然后当前目录会生成go.mod文件，其内容为：

```shell
module ModuleName

go 1.15
```

Go Modules会自动管理包，如果需要引入依赖，只需要在go.mod下添加以下内容（以gorose为例子）

```go
module ModuleName
 
require (
	github.com/gohouse/gorose v1.0.5  //对象关系映射包
)
```

### 2.3 go get

`go get` 命令用于拉取新的依赖，以下为go get命令具体用法

| go get             | 拉取依赖，会进行指定性拉取（更新），并不会更新所依赖的其它模块。 |
| ------------------ | ------------------------------------------------------------ |
| go get -u          | 更新现有的依赖，会强制更新它所依赖的其它全部模块，不包括自身。 |
| go get -u -t ./... | 更新所有直接依赖和间接依赖的模块版本，包括单元测试中用到的。 |

其他参数

```shell
-d 只下载不安装
-f 只有在你包含了 -u 参数的时候才有效，不让 -u 去验证 import 中的每一个都已经获取了，这对于本地 fork 的包特别有用
-fix 在获取源码之后先运行 fix，然后再去做其他的事情
-t 同时也下载需要为运行测试所需要的包
-u 强制使用网络去更新包和它的依赖包
-v 显示执行的命令
```

#### 2.4 常用命令

```shell
go mod init  // 初始化go.mod
go mod tidy  // 更新依赖文件
go mod download  // 下载依赖文件
go mod vendor  // 将依赖转移至本地的vendor文件
go mod edit  // 手动修改依赖文件
go mod graph  // 查看现有的依赖结构
go mod verify  // 校验依赖
```

## 拓展参考

1. [Go包管理工具](https://zhuanlan.zhihu.com/p/92992277)
2. [关于Go Modules，看这一篇文章就够了](https://zhuanlan.zhihu.com/p/105556877)