---
layout: post
title: 使用 logrotate 来管理日志文件
---

logrotate 是 Linux 下的一个日志管理工具。通过 logrotate 可以很轻松地管理（拆分、压缩等）系统的所有日志文件。
通过配置，我们可以按日志大小或者固定周期来管理日志文件。

安装 logrotate

    sudo apt-get install logrotate

打开配置文件

    sudo vim /etc/logrotate.conf

在文件最后增加以下配置

    /PATH/TO/YOUR/LOG/*.log {
        daily          # 按日，也可以 weekly 按周，monthly 按月
        dateext        # 增加日期作为后缀
        missingok      # 如果文件不存在，忽略错误信息
        rotate 30      # 保留 30 份
        compress       # 压缩
        delaycompress  # 延迟压缩，也就是本次拆分的文件在下次执行时再压缩
        notifempty     # 忽略空白文件
        copytruncate   # 拆分文件后，清空原有文件，而不是创建一个新文件
    }

马上执行 logrotate

    sudo /usr/sbin/logrotate -f /etc/logrotate.conf

查看你的 log 文件所在目录，应该可以得到类似这个的结果

    -rw-r--r-- 1 deploy deploy  0K 2012-06-27 23:39 production.log
    -rw-r--r-- 1 deploy deploy  27M 2012-06-27 23:15 production.log-20120627
