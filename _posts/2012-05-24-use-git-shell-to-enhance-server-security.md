---
layout: post
title: 使用 git-shell 增强 Git 服务器安全
---

在开发一个项目时，往往需要多人共用一个 Git 库，但同时又希望限制其它人只能使用 Git 库而不能对服务器进行其它操作。

在此情景下，你可以用 Git 自带的 git-shell 工具限制 Git 库所属用户的活动范围。只要把它设为此用户登入的 shell，那么该用户就无法使用普通的 bash 或者 csh 什么的 shell 程序。

假设服务器上使用 `git` 用户来管理 git 库

    which git-shell # 一般会输出 /usr/bin/git-shell
    sudo usermod -s /usr/bin/git-shell git

现在 `git` 用户依然可以用 SSH 来读取和推送 Git 库，但不能用 SSH 登录服务器，如果尝试登录服务器，会出现下面的提示：

    ssh git@server_address
    fatal: What do you think I am? A shell?
    Connection to gitserver closed.
