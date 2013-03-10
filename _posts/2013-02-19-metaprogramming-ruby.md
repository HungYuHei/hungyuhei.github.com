---
layout: post
title: 《Ruby 元编程》摘录
---

除去 Rails 元编程那章，已经《Ruby 元编程》读了两遍多了，伴隨着的也是自己由初级水平向中级水平的提升。
刚看此书时，自己基本上只处于刚好知道一点 Ruby 元编程。然后第看过第一遍后，接着便是在公司的实际项目中将所学的进行实践，接着看第二遍便是对之前所学知识、所写代码的进行一检阅。

下面是看书过程中的一些摘录、笔记。

## 方法查找

    String.ancestors # => [String, Comparable, Object, Kernel, BasicObject]

「向右一步，再向上」：

<img src="http://ww4.sinaimg.cn/large/a74ecc4cjw1e1y3eiio3hj.jpg" alt="方法查找" width="610" />

## 动态方法

    send           # => 动态派发
    defind_method  # => 动态定义方法
    method_missing # => 幽灵方法

<br>

### 方法冲突时

当一个幽灵方法和一个真实方法发生命名冲突时，后者会胜出。  
为了安全起见，你应该在代理类中删除绝大多数继承来的方法（可以使用 `Module#undef_method` 和 `Module#remove_method` ）。这就是所谓的白板 ( Blank State ) 类，它所拥有的方法比 Object 类还要少。

### 性能的忧虑

<br>

* 在早期版本时通常不会是什么大问题。
* 用性能分析器测试代码。
* 折中方式：第一次调用幽灵方法时，为它创建动态方法。

<br>

## Block:

当代码运行时，它需要一个执行环境：局部变量、实例变量、self ...... 这些东西是绑定在对象上的名字，可以把它们简称为绑定 ( binding )。

当定义一个块时，它会获取当时环境中的绑定。而且将这个块传给一个方法时，它会带着这些绑定一起进入该方法。

### 作用域门 ( Scope Gate )

每个 Ruby 作用域包含一组绑定，并且不同的作用域之间被称为作用域门。  
程序会在这三个地方关闭前一个作用域，同时打开一个新的作用域：

* 类定义
* 模块定义
* 方法

![作用域门](http://ww3.sinaimg.cn/large/a74ecc4cjw1e1y3o1nqsaj.jpg)

### 扁平化作用域 ( flattening the scope )

让绑定穿越作用域门：

    Class.new() { }      # 代替 class
    Module.new() { }     # 代替 module
    Module#defind_method # 代替 def

<br>

### & 操作符

Ruby 中绝大多数东西都是对象，但是 block 不是。  
`&` 操作符的真正含义：在 Proc 和 block 之间转换。  
block 使用 `yield` 语句来调用，Proc 使用 `call` 语句来调用。

### Proc 与 lambda 的对比

* return 语句
* 参数检查

在 lambda 中，`return` 仅仅表示从这个 lambda 中返回  
在 Proc 中，`return` 则表示从定义 Proc 的作用域中返回

在 lambda 中，参数数量要对应  
在 Proc 中，会忽略多余的参数，或者对未指定的参数赋值 nil

建议：**将 lambda 作为第一选择。**

### 重访方法

注意：方法的对象必须属于同一个类

    class MyClass
      def initialize(value) 
         @x = value
      end
      
      def my_method
         @x
      end
    end
 
    obj = MyClass.new(1)
    m = obj.method :my_methodm.call  # => 1
    unbound = m.unbindanother_obj = MyClass.new(2)
    m = unbound.bind(another_obj)
    m.call  # => 2

<br>

## 类定义

当使用 `class` 关键字时，并非是在指定对象未来的行为方式，相反，实际上是在运行代码。  
类只是增强的模块。

### 当前类

Ruby 解释器总是追踪当前类（或模块）的引用。在类的定义中，当前类就是 self ——正在定义的类。  
`instance_eval` 方法仅仅会修改 self，而 `class_eval` 方法会同时修改 self 和当前类。  

### 类实例变量

类的实例变量不同于类的对象的实例变量

    class MyClass
      @my_var = 1

      def self.read() @my_var; end
      def write() @my_var = 2; end
      def read() @my_var; end
    end


    obj = MyClass.new
    obj.write
    obj.read       # => 2
    MyClass.read   # => 1


绝大多数 Ruby 主义者都避免使用类变量，而尽量使用类实例变量。

### Eigenclass

    obj = Object.new
    eigenclass = class << obj
      self
    end

    eigenclass.class # => Class

![Eigenclass](http://ww4.sinaimg.cn/large/a74eed94jw1e1y3yy1suij.jpg)


* 只有一个对象——要么是普通对象，要么是模块。
* 只有一种模块——可以是普通模块、类、eigenclass 或代理类。
* 只有一个方法，它存在于一种模块中——通常是类中。
* 每个对象（包括类）都有自己的「真正的类」——要么是普通类，要么是 eigenclass
* 除了 BasicObject 类无超类外，每个类有且只有一个超类。这意味着从任何类只有一条向上直到 BasicObject 的祖先链。
* 一个对象的 eigenclass 的超类是这个对象的类，一个类的 eigenclass 的超类是这个类的 eigenclass
* 当调用一个方法，Ruby 先向「右」迈一步进入接收者真正的类，然后向「上」进入祖先链。这就是 Ruby 查找方法的全部内容。

<br>

### 对象扩展和类扩展

    module MyModule
      def my_method() 'hello'; end
    end

    obj = Object.new
    obj.extend MyModule
    obj.my_method     # => 'hello'

    class MyClass
      extend MyModule
    end
    MyClass.my_method # => 'hello'

`Object#extend` 只是在接收者 eigenclass 中包含模块的快捷方式，你完全可以自己手动实现。

### 别名

    alias :new :old           # Ruby 关键字，而非方法，所以两个方法之间没有逗号
    alias_method :new, :old   # Ruby 方法

可以通过以下三个步骤写一个环绕别名

1. 给已有的方法定义一个别名
2. 重定义这个方法
3. 在新的方法中调用老的方法

