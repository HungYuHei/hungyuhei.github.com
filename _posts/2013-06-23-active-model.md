---
layout: post
title: Rails 源码学习笔记 2：漫游 ActiveModel
---

模块化大概是计算机编程中一个永恒的追求吧。Rails 3 相对于 Rails 2 其中的一大改进就是更完善的模块化，
因此这也带给了 Rails 良好的扩展性。

Rails 默认使用 ActiveRecord 作为 ORM (PostgreSQL)

```
使用 ActiveRecord 的例子
```

如果要使用其它 ORM，像 MongoDB，那么用 Mongoid
如何保持兼容 Rails 原有的 Controller 和 View 呢，答案是 ActiveModel

> Active Model was created to hold the behavior shared between Active Record and
> Active Resource in modules that can be cherry-picked at will.
> It’s also responsible for defining the API required by Rails controllers and views,
> so any other ORM can use Active Model to ensure Rails behaves exactly as it would with Active Record.
>
> Crafting Rails Applications -- José Valim

[ActiveModel 源码](https://github.com/rails/rails/blob/master/activemodel/lib/active_model.rb) 

### AttributeMethods 模块

Rails 通过 ActiveSupport 模块提供了很方便 `present?` 方法来做非空判断，
不过对 ActiveModel 的 attribute 其实还有更方便的方式来做非空判断：

    user.name.present? # => true
    user.name?         # => true

而且更有趣的是 ActiveModel 下的所有 attribute 都可以使用这种方式来判断，
那么 Rails 是怎样实现的，答案是 ActiveModel::AttributeMethods 模块。

那么下面我们来实现一个类似的方法吧：

    class SimpleUser
      include ActiveModel::AttributeMethods
    
      attribute_method_suffix '_blank?'
      define_attribute_methods ['name']

      attr_accessor :name

      private
        def attribute_blank?(attr)
          attr.blank?
        end
    end

    simple_user = SimpleUser.new
    simple_user.name_blank?   # => true
    simple_user.name.present? # => false

    simple_user.name = 'Tom'
    simple_user.name_blank?   # => false
    simple_user.name.present? # => true


### ActiveModel::Conversion 模块

Rails 的 Controller 和 View helper 在调用 Model 的任何方法前，会先调用 `to_model`，然后使用该方法会返回结果来调用方法。
这样做是为了允许其它一些不想依赖于 ActiveModel 的 ORM 通过这个方法来返回代理对象（proxy object）来实现兼容。
对于我们来说，只需要简单地返回 `self` 即可：

    def to_model
      self
    end


在 Rails 的 Controller 或者 View 中，我们经常会有下列用法：

    user_path(@user)
    div_for(@user)

实际在调用上面的第一行代码时，Rails 会调用 `@post.to_param` 并根据返回的结果来生成 URL。  
而对于第二行代码，Rails 会调用 `@user.to_key` 并根据返回的结果来生成 div 元素。

`to_key` 方法应当返回能够唯一识别 Model 的 keys 数组，一般是 `[id]`  
`to_param` 方法应当返回 Model 的唯一标识，一般是 `id`

不过由于我们的例子中不会储存到数据库中，也就不存在唯一的 id，
所以这两个方法都只需直接返回 `nil` 即可。

这些默认行为其实已经封装在 `ActiveModel::Conversion` 模块中，
所以我们只需简单地 include 这个模块即可（接上面 SimpleUser 要例子）：

    class SimpleUser
      include ActiveModel::AttributeMethods
      include ActiveModel::Conversion
      ...
    end





