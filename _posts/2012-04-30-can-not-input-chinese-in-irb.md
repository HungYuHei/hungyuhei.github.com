---
layout: post
title: "irb 不能输入中文"
description: ""
category: 
tags: []
---
{% include JB/setup %}

之前将 Ruby 版本从 1.9.2 升级到 1.9.3 之后，irb 就不能输入中文了

问题出现的原因很可能是：

* 没有安装 Readline
* 安装了 Readline，但用 rvm 装 Ruby 1.9.3 时没有正确编译 Readline


不管你有没有安装 Readline，都可以先通过 rvm 下载 Readline 到 rvm 目录，然后重新编译 Ruby：

    rvm pkg install readline
    rvm reinstall 1.9.3 --with-readline-dir=$rvm_path/usr
    
其实，如果你很清楚已经安装了 Readline，并且知安装在哪里，那么可以只需直接执行：

    rvm reinstall 1.9.3 --with-readline-dir=YOUR_READLINE_PATH

**如果你是 Mac 平台，如果安装失败，尝试加上 `--with-gcc=clang`**
