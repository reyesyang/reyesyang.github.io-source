---
title: "[翻译] 词法作用域和 Ruby 中的类变量"
tags: [Ruby, Scope, Lexical Scope]
---

本文翻译自 Starr Horne 发表在 Honeybadger 上的文章：[Lexical scoping and Ruby class variables](http://blog.honeybadger.io/lexical-scoping-and-ruby-class-variables/)

Ruby 的类变量时常让人感到迷惑。即使是有经验的 Ruby 程序员也觉得它们很难理解。最明显的一个例子就是继承：

```ruby
class Fruit
  @@kind = nil

  def self.kind
    @@kind
  end
end

class Apple < Fruit
  @@kind = "apple"
end

Apple.kind
# => "apple"
Fruit.kind
# => "apple"
```

在子类中修改 `@@kind` 的值的话，父类中的也被修改了。这实在太令人混乱了。但 Ruby 就是设计成这样工作的，这是很久以前模仿 Smalltalk 所做出的设计决策。

#### 更糟糕的

类变量还有一些其他古怪表现的例子，看起来并不像是当初设计语言时的架构决策而故意这么实现的怪癖。今天我就说其中我觉得很有意思的一个。

我们将看到两段代码，感觉上他们应该产生同样的结果，然而并不是。

在第一个例子中，我们设置了一个类变量并且创建了一个方法返回它。这没有任何花哨的东西。一切都运行正常。

```ruby
class Foo
  @@val = 1234

  class << self
    def val
      @@val
    end
  end
end

Foo.val
# => 1234
```

也许你并不知道，其实 `class << self` 不是必须放在类定义的内部的。在下面的例子中，我们把它移到外面。我们添加了一个类方法，但是它并不能访问类变量。

```ruby
class Bar
  @@val = 1234
end

class << Bar
  def val
    @@val
  end
end

Bar.val

# warning: class variable access from toplevel
# NameError: uninitialized class variable @@val in Object
```

当我们尝试通过方法访问类变量的时候，我们却得到警告和异常。发生了什么呢？

#### 来到词法作用域

我已经 99% 的确信导致这古怪结果的原因是词法作用域，它也是 Ruby 中让人迷惑的一点。

以防万一你对这个概念还不太熟悉，词法作用域指的是代码中的事物按照他们出现在代码中的位置分类工作，而不是按照他们应该属于的抽象对象模型来分类工作的。看下面的例子就很好理解了：

```ruby
class B
  # x and y share the same lexical scope
  x = 1
  y = 1
end

class B
  # z has a different lexical scope from x and y, even though it's in the same class
  z = 3
end
```

#### 类是按照词法来决定的

所以在类变量中词法作用域又是如何工作的呢？

其实，Ruby 为了获取类变量，它不得不先确定从哪个类中搜索。Ruby 就是使用词法作用域去查找这个类的。

如果我们再仔细看前面可以正确运行的示例，我们就能发现代码访问的类变量是本来就存在于类定义中的。

```ruby
class Foo
  class << self
    def val
      # I'm lexically scoped to Foo!
      @@val
    end
  end
end
```

在不能运行的示例中，代码访问的类变量并不在类的词法作用域中。

```ruby
class << Foo
  def val
    # Foo is nowhere in sight.
    @@val
  end
end
```

但是如果它的词法范围不是 `Foo` 类的话，那是什么呢？Ruby 打印的警告信息给了我们一些线索：`warning: class variable access from toplevel`。

那个不能运行的示例证明了类变量的词法范围是顶层作用域。这会导致很奇怪的行为。

例如，如果我们尝试在词法范围是 main 的上下文中设置一个类变量，这个类变量就会设置到 `Object`。

```ruby
class Bar
end

class << Bar
  def var=(n)
    # This code is lexically scoped to the top level object.
    # That means, that '@@val' will be set on `Object`, not `Bar`
    @@val = i
  end
end

Bar.val = 100

# Whaa?
Object.class_variables
# => [:@@foo]
```

#### 更多示例

有很多方法在类的词法范围之外引用类变量。它们都可能给你带来麻烦。这里几个例子供你参考：

```ruby
class Foo
  @@foo = :foo
end

# This won't work
Foo.class_eval { puts @@foo }

# neither will this
Foo.send :define_method, :x do
  puts @@foo
end

# ..and you were't thinking about using modules, were you?
module Printable
  def foo
    puts @@foo
  end
end

class Foo
  @@foo = :foo
  include Printable
end

Foo.new.foo
```

现在你明白为什么大家都说不要使用 Ruby 中的类变量了吧。:)
