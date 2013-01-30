---
layout: post
title: "Rails 源码学习笔记 1：ActiveSupport"
description: "Rails 源码学习笔记 1：ActiveSupport"
category: 
tags: [Rails, ActiveSupport]
---

Rails 默认将运行环境分为 production, test, development 三种，而在开发 Rails 的过程中，有些时候需要判断当前的运行环境，从而作出不同的操作。

可以这样来判断：

    Rails.env.production?

很 Ruby-style，很 Magic 吧。

好奇心趋势我去一窥源码实现，执行 `Rails.env.class`，于是顺利找到[源码实现](https://github.com/rails/rails/blob/v3.2.11/activesupport/lib/active_support/string_inquirer.rb)。

原来是定义了一个 `StringInquirer` 类来进行包装，然后通过 `method_missing` 来判断。

代码很简单，主要逻辑就是：如果方法名最后是一个问号，便将去除问号后的方法名作为字符串与 Rails.env 对比。

如果要在非 Rails 环境中使用这种方式，只要加载 ActiveSupport 就可以了：

    require 'active_support'
    
    str = ActiveSupport::StringInquirer.new('foo')
    str.foo? # => true
    str.bar? # => false

当然，不考虑太多的话，自己做一个 Monkey patch 玩玩也可以

    class String
      def method_missing(method_name, *arguments)
        if method_name[-1] == '?'
          self == method_name[0..-2]
        else
          super
        end
      end
    end
    
    str = 'foo'
    str.foo? # => true
    str.bar? # => false

{% include JB/setup %}
