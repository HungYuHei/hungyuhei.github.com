---
layout: post
title: Ubuntu 10.04 安装 JDK7
---

实现全文搜索时用到了 [Sunspot](http://sunspot.github.com/)，因此在服务器上需要搭建 Java 的运行环境

万恶的 Oracle 将 SUN 买下之后，也退出了 "Operating System Distributor License for Java" (JDL)，因此不
能直接通过 Ubuntu 的源来安装 Oracle Java (JVM/JDK) ，只能安装 OpenJDK

但对于 OpenJDK 始终不放心，幸好可以通过添加 PPA 库来安装 Oracle JDK7 ：

    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java7-installer
    sudo apt-get remove oracle-java7-installer

安装完毕后，在命令行运行：

    java -version

输出了对应的 JDK 版本则表示安装成功
