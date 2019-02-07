---
title: Java工程打成jar包在命令行执行运行缓慢问题解决
date: 2014-10-10 10:13:00
layout: post
---

一个普通的java工程，用cxf连接webserivice。在eclipse下运行正常，但用eclipse达成jar包，在命令行执行却异常缓慢。

通过一些尝试，发现是因为打包的方式不对。eclipse导出可运行jar包有三种方式，分别为：
- Extract required libraries into generated JAR
- Package required libraries into generated JAR
- Copy required libraries into  a sub-folder next to the generated JAR

一开始选择了第二种，把需要的jar包全部打进了生成的JAR包中，这种情况下运行非常慢。之后尝试了第一种和第三种，运行正常了。

用解压软件查看生成的jar包，发现第一种是把引用的jar包中的类提取出来了。第三种，则是把引用的jar包放在了另一个文件夹中。这两种方式运行正常。
可能和java 的jar命令执行的方式有关，还有待考证。

