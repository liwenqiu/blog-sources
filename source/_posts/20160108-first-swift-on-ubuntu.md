title: 在 Ubuntu 体验 Swift
date: 2016-01-08 17:48:26
tags: swift
---

[Swift](https://swift.org) 做为苹果官方新推出的一种编程语言，在过去的一年里面得到全世界 Apple Developer 的关注。苹果也如承诺所言，在 2015 年底将 Swift 开源，并支持用 Swift 开发 Linux 程序。

这篇文字是记录我在虚拟的 Ubuntu 系统里面体验 Hello Swift 的过程，仅作为记录和参考。

<!--more-->

## 准备虚拟环境

用 Vagrant 可以快速的创建出干净，独立的虚拟关键。首先创建一个目录来存放用到的资源。

```bash
$ mkdir ~/Machines/SwiftOnLinux && cd ~/Machines/SwiftOnLinux

# 创建一个 ubuntu 14.04 的虚拟机器
$ vagrant init ubuntu/trusty64

# 启动虚拟机
$ vagrant up

# 进入虚拟机环境内
$ vagrant ssh

# 安装必备的软件包
$ sudo apt-get update
$ sudo apt-get install clang
```

到这里一个可以安装 Swift 的 Linux 环境就已经准备好了。

## 安装 Swift

我的宿主环境是 OSX EI Capitan。Vagrant 默认会把环境目录（即：~/Machines/SwiftOnLinux) 挂载到虚拟环境的 /vagrant 目录。这样我们可以非常方便的在 OSX 里面下载好 Swift 安装包，也可以在 OSX 里面编写代码，然后到虚拟机里面编译运行。

从 Swift 官方网站 [下载](https://swift.org/builds/ubuntu1404/swift-2.2-SNAPSHOT-2016-01-06-a/swift-2.2-SNAPSHOT-2016-01-06-a-ubuntu14.04.tar.gz) 安装包到 `~/Machines/SwiftOnLinux`。然后在虚拟机环境里面直接解压开

```bash
$ tar xzf swift-2.2-SNAPSHOT-2016-01-06-a-ubuntu14.04.tar.gz

# 创建一个软连接，方便今后更新新版本
$ ln -s swift-2.2-SNAPSHOT-2016-01-06-a-ubuntu14.04 swift
```

然后在 ~/.bashrc 文件最后加入

```bash
export PATH=/vagrant/swift/usr/bin:$PATH
```

`source ~/.bashrc` 生效之后就可以验证 swift 是否安装正确

```bash
$ swift --version
Swift version 2.2-dev (LLVM 3ebdbb2c7e, Clang f66c5bb67b, Swift 54dcd16759)
Target: x86_64-unknown-linux-gnu
```

## Hello Swift

创建 Hello 项目结构

```bash
$ mkdir /vagrant/Hello && cd /vagrant/Hello

# Package.swift 描述一个 Swift Package，这里只是一个空文件
$ touch Package.swift

# Swift 代码放在 Sources 文件夹里面
$ mkdir Sources
$ touch Sources/main.swift
```

最后的项目结构如下：

```
Hello
  |-- Sources
  |   |-- main.swift
  |-- Package.swift
```

`main.swift` 文件内容：

```swift
print("Hello Swift")
```

最后，编译测试我们的 Hello Swift

```
$ pwd
/vagrant/Hello

# build 我们的 Swift Package
$ swift build
Compiling Swift Module 'Hello' (1 sources)
Linking Executable:  .build/debug/Hello

# 可执行文件已经编译链接好了，可以直接运行
./.build/debug/Hello
Hello Swift
```

Swift Package 的更多信息可以参考 [官方文档](https://swift.org/package-manager/)







