[toc]

### 一、简介

Go（又称 Golang）是 Google 的 Robert Griesemer，Rob Pike 及 Ken Thompson 开发的一种静态强类型、编译型语言。Go 语言语法与 C 相近，但功能上有：内存安全，GC（垃圾回收），结构形态及 CSP-style 并发计算。

==菜鸟教程：==https://www.runoob.com/go/go-environment.html

### 二、安装

1. 安装地址：https://golang.google.cn/dl/

https://golang.google.cn/dl/go1.16.3.src.tar.gz

2. 解压后配置环境变量

   ```shell
   #go
   export PATH=$PATH:/home/environment/go/bin
   ```

3. 新建 test.go 文件

   ```go
   package main
   import "fmt"
   func main() {
      fmt.Println("Hello, World!")
   }
   ```

4. 启动

   ```shell
   go run test.go
   ```

   
