---
layout: post
title: "Rails 中实现 CSS Namespaces 机制"
description: ""
category: 
tags: [Rails]
---
{% include JB/setup %}

Namespaces（命名空间）的作用最主要是可以模块化 (modular)，以方便组织代码和防止代码冲突。如果某一门编程语言缺少 Namespaces，我想它很难称得上是一门健全的语言。当然 CSS 只能算是描述性语言，算不上是编程语言，所以天生缺少 Namespaces 的机制。

随着网站不断地迭代开发，CSS 的代码量激增，由于缺少 Namespaces 的机制，到了后期容易做成 CSS 之间相互冲突，变得牵一发而动全身，每次修改都得小心翼翼，如果能引入类似 Namespaces 的机制，那就舒心多了。

通过 Rails 的 `yield` 和 `content_for`，其实就可以很方便地为 CSS 引入 Namespaces。

在 layout 中：

    # app/views/layouts/application.html.erb
   ……
    <body class="<%= content_for?(:body_class) ? yield(:body_class) : '' %>">
      ……
    </body>
    ……
      

然后在 view 文件中加上这段代码，就可以将 layout 中 body 标签的 class 定义为 `example_name`：

    # app/views/users/index.html.erb
    <% content_for :body_class, 'example_name' %>
    
最后在 CSS 中将相关的样式前都加上 `body.example_name`，例如：

    body.example_name .one_class {
      ……
    }
    body.example_name .another_class {
      ……
    }
    
CSS 需要在每个相关样式前都加 body.example_name 显得有点麻烦，如果你是使用 SASS（或 SAAA）,配合它的 `Nesting` 和 `@import` 功能，那就可以更进方便地组织代码。

假设你在 layout 中引用了 application.sass（当然最终编译出来应该是 application.css），那你可以将某个模块用的样式分割到单独文件中，然后在 application.sass 中 import 就可以：

    # app/assets/stylesheets/application.sass
    @import "modules/_users"
    ……
    
单独模块的样式：

    # app/assets/stylesheets/modules/_users.sass
    body.example_name
      .one_class
        ……
      .another_class
        ……
        
上面例子中，`body.example_name` 就达到了类似 Namespaces 的作用了。
