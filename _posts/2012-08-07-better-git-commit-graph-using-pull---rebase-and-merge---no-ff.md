---
layout: post
title: 洁癖者用 Git：pull --rebase 和 merge --no-ff
---

## pull --rebase 
### 首先是吐嘈
<br>

![git pull --rebase](http://ww4.sinaimg.cn/large/a74e55b4jw1dvnhy8lfndj.jpg)

如果你正在 code review，看到上图（下文将称之为：提交线图）之后，特别是像我这样有某种洁癖的人，是否感觉特别难受？如果是的话，请看下文吧 :)

### 为什么

Git 作为分布式版本控制系统，所有修改操作都是基于本地的，在团队协作过程中，假设你和你的同伴在本地中分别有各自的新提交，而你的同伴先于你 push 了代码到远程分支上，所以你必须先执行 `git pull` 来获取同伴的提交，然后才能 push 自己的提交到远程分支。而按照 Git 的默认策略，如果远程分支和本地分支之间的提交线图有分叉的话（即不是 fast-forwarded），Git 会执行一次 merge 操作，因此产生一次没意义的提交记录，从而造成了像上图那样的混乱。

### 解决

其实在 pull 操作的时候，，使用 `git pull --rebase ` 选项即可很好地解决上述问题。

加上 `--rebase` 参数的作用是，提交线图有分叉的话，Git 会 rebase 策略来代替默认的 merge 策略。

使用 rebase 策略有什么好处呢？借用一下 `man git-merge` 中的图就可以很好地说明清楚了。

假设提交线图在执行 pull 前是这样的：

                     A---B---C  remotes/origin/master
                    /
               D---E---F---G  master
               
如果是执行 `git pull` 后，提交线图会变成这样：

                     A---B---C remotes/origin/master
                    /         \
               D---E---F---G---H master
               
结果多出了 `H` 这个没必要的提交记录。如果是执行 `git pull --rebase` 的话，提交线图就会变成这样：

                           remotes/origin/master
                               |
               D---E---A---B---C---F'---G'  master
               
`F` `G` 两个提交通过 `rebase` 方式重新拼接在 `C` 之后，多余的分叉去掉了，目的达到。

### 小结
大多数时候，使用 `git pull --rebase ` 是为了使提交线图更好看，从而方便 code review。

不过，如果你对使用 git 还不是十分熟练的话，我的建议是 `git pull --rebase ` 多练习几次之后再使用，因为 **rebase 在 git 中，算得上是『危险行为』**。

另外，还需注意的是，使用 `git pull --rebase ` 比直接 pull 容易导致冲突的产生，如果预期冲突比较多的话，建议还是直接 pull。

## merge --no-ff

上述的 `git pull --rebase` 策略目的是修整提交线图，使其形成一条直线，而即将要用到的 `git merge --no-ff <branch-name>` 策略偏偏是反行其道，刻意地弄出提交线图分叉出来。

假设你在本地准备合并两个分支，而刚好这两个分支是 fast-forwarded 的，那么直接合并后你得到一个直线的提交线图，当然这样没什么坏处，但如果你想更清晰地告诉你同伴：**这一系列的提交都是为了实现同一个目的**，那么你可以刻意地将这次提交内容弄成一次提交线图分叉。

执行 `git merge --no-ff <branch-name>` 的结果大概会是这样的：

![git merge --no-ff](http://ww1.sinaimg.cn/large/a74eed94jw1dvnhyrq8rhj.jpg)

中间的分叉线路图很清晰的显示这些提交都是为了实现 **complete adjusting user domains and tags**

### 更进一步
往往我的习惯是，在合并分支之前（假设要在本地将 feature 分支合并到 dev 分支），会先检查 feature 分支是否『部分落后』于**远程 dev 分支**：

    git checkout dev
    git pull # 更新 dev 分支
    git log feature..dev
    
如果没有输出任何提交信息的话，即表示 feature 对于 dev 分支是 up-to-date 的。如果有输出的话而马上执行了  `git merge --no-ff` 的话，提交线图会变成这样：

![git-merge](http://ww2.sinaimg.cn/large/a74e55b4jw1dvnijr276hj.jpg)

所以这时在合并前，通常我会先执行：

    git checkout feature
    git rebase dev
    
这样就可以将 feature 重新拼接到更新了的 dev 之后，然后就可以合并了，最终得到一个干净舒服的提交线图。

**再次提醒：像之前提到的，rebase 是『危险行为』，建议你足够熟悉 git 时才这么做，否则的话是得不偿失啊。**

## 总结
使用 `git pull --rebase` 和 `git merge --no-ff` 其实和直接使用 `git pull` `git merge` 得到的代码应该是一样。

使用 `git pull --rebase` 主要是为是将提交约线图平坦化，而 `git merge --no-ff` 则是刻意制造分叉。

一言以蔽之：如果你有点洁癖症状，才考虑用它们吧。
