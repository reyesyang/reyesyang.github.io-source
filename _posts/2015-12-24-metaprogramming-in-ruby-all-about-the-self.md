---
title: "[翻译] Ruby 元编程：都是 Self 的事"
tags: [Ruby, Meta Programming]
---

本文翻译自 Yehuda Katz 的博客文章 [Metaprogramming in Ruby: It's All About the Self](http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)

在我写完上篇[关于 Rails 插件的惯用语法的文章](http://yehudakatz.com/2009/11/12/better-ruby-idioms/)后，我意识到 Ruby 元编程，就核心而言，其实非常简单。

归根结底是因为所有的 Ruby 代码都是可执行代码 —— 不需要额外的编译或者运行时阶段。在 Ruby 中，每一行代码都是针对一个特定的 `self` 执行的。考虑下面五段代码：

```ruby
class Person
  def self.species
    "Homo Sapien"
  end
end

class Person
  class << self
    def species
      "Homo Sapien"
    end
  end
end

class << Person
  def species
    "Homo Sapien"
  end
end

Person.instance_eval do
  def species
    "Homo Sapien"
  end
end

def Person.species
  "Homo Sapien"
end
```

上面全部五段代码都定义了一个返回 “Homo Sapien” 字符串的 `Person.species` 方法。再看看另外一段代码：

```ruby
class Person
  def name
    "Matz"
  end
end

Person.class_eval do
  def name
    "Matz"
  end
end
```

这段代码都是在 Person 类上定义了一个名为 `name` 的方法。所以 `Person.new.name` 将会返回 “Matz”。对于熟悉 Ruby 的人来讲，这并不是什么新鲜事情。在学习元编程的时候，这些代码都是分开独立展示的：另一种用于确定方法归属的机制。实际上，以上所有代码之所以如此工作，都可以归咎于统一的一个原因。

首先，理解 Ruby 的 metaclass 如何工作是非常重要的。当开始学习 Ruby 的时候，你就接触了类这个概念，Ruby 中的每个对象都归属于一个类：

```ruby
class Person
end

Person.class  #=> Class

class Class
  def loud_name
    "#{name.upcase}!"
  end
end

Person.loud_name  #=> "PERSON!"
```

`Person` 是 `Class` 类的一个实例，所以任何添加给 `Class` 类的方法在 `Person` 中都有效。但是有一点他们没有告诉你，其实，Ruby 中的每一个对象还拥有自己的 metaclass，也是一个可以拥有方法的 `Class`，但是只关联于对象自己。

```ruby
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end
```

当我们给 `matz` 的 metaclass 添加了方法 `speak` 的时候发生了什么呢？其实 `matz` 对象先继承自它的 metaclass，然后是 Object。这之所以多少有些不太好理解的原因是 metaclass 在 Ruby 中不可见：

```ruby
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end

matz.class  #=> Object
```

实际上 `matz` 的类应该是那个不可见的 metaclass。我们可以通过下面的方法进入 metaclass：

```ruby
metaclass = class << matz; self; end
metaclass.instance_methods.grep(/speak/)  #=> ["speak"]
```

这时如果是在其他相同话题的文章中，你可能已经在努力想要记住所有这些细节了；看起来有很多规则。还有，`class << matz` 到底是什么？

事实上，这些奇怪的规则都可以归纳为一个：掌控代码中的 `self`。让我们回头看看前面的代码：

```ruby
class Person
  def name
    "Matz"
  end

  self.name  #=> "Person"
end
```

这里，我们给 `Person` 类添加了一个 `name` 方法。当我们说 `class Person` 时，`self` 直到 `Person` 类代码块的结尾都是其自身。

```ruby
Person.class_eval do
  def name
    "Matz"
  end

  self.name  #=>  "Person"
end
```

在这，我们做了一件和上面完全相同的事：给 `Person` 类的实例添加了 `name` 方法，`class_eval` 将 `self` 设置为 `Person`，直到代码块的结尾。类处理起来感觉非常的直观，其实 metaclass 也同样很直观：

```ruby
def Person.species
  "Homo Sapien"
end

Person.name  #=> "Person"
```
在前面 `matz` 的例子中，我们在 `Person` 的 metaclass 上定义了 `species` 方法。我们没去操作 `self`，但是你可以看到使用 `def` 方法并且关联到一个对象上的话，就会使方法定义到该对象的 metaclass 上。

```ruby
class Person
  def self.species
    "Homo Sapien"
  end

  self.name  #=> "Person"
end
```

这里，我们打开 `Person` 类，在代码块中将 `self` 设置为 `Person`，正如上例所示。所以，当我们将方法定义到对象 `self` 上时，我们其实就是将方法定义到了 `Person` 类的 metaclass 上。还有，你可以看到，在 Person 类里面的 `self.name` 和外面的 `Person.name` 其实是一样的。

```ruby
class << Person
  def species
    "Homo Sapien"
  end

  self.name  #=> ""
end
```

Ruby 提供了一个可以直接进入对象 metaclass 的语法。通过 `class << Person`，我们就将代码块中的 `self` 设置为了 `Person` 类的 metaclass。所以，`species` 方法就添加到了 `Person` 类的 metaclass 上，而不是 `Person` 类本身。

```ruby
class Person
  class << self
    def species
      "Homo Sapien"
    end

    self.name  #=> ""
  end
end
```

这里，我们将前面的几个技巧组合使用。首先，我们打开 `Person` 类，让 `self` 等价与 `Person` 类。其次，我们使用 `class << self` 让 `self` 等价于 `Person` 类的 metaclass。当我们定义 `species` 方法是，它就定义到了 `Person` 的 metaclass 上。

```ruby
Person.instance_eval do
  def species
    "Homo Sapien"
  end

  self.name  #=> "Person"
end
```

在最后这个 `instance_eval` 的示例中做了些有意思的事情。他将 `self` 分为用于执行代码的 `self` 和用于定义方法的 `self` 两部分。当使用 `instance_eval` 时，新方法定义在 metaclass，但是 `self` 还是对象本身。

上面的几个例子中，殊途同归自然地产生于 Ruby 的语义。通过上面的解释，应该明白了 `def Person.species`，`class << Person; def species` 和 `class Person; class << self; def species` 并不是特地设计了这三种方法来做同一件事情，只是因为 Ruby 灵活地考虑到你程序中特定环境下的 `self` 而产生的。

另一方面，`class_eval` 稍有不同。因为它使用了代码块，不同于作为关键字，他可以捕捉到周围的局部变量。这可提供了强大的 DSL 能力，此外可以控制代码块中的 `self`。但是除此之外，它和在这里的其它结构是完全相同的。

最后，`instance_eval` 将 `self` 分为两部分，同时也可以让你访问到定义之外的局部变量。

在下面的表格中，创建一个新作用域指的是代码块中的语句不能访问代码块之外的局部变量。

| mechanism | method resolution | method definition | new scope? |
|-----------|-------------------|-------------------|------------|
| class Person | Person | same | yes |
| class << Person | Person's metaclass | same | yes |
| Person.class_eval | Person | same | no |
| Person.instance_eval | Person | Person's metaclass | no |

需要注意的是 `class_eval` 仅仅在 Module（Class 继承自 Module）中有效，而且它是 `module_eval` 的别名。另外，Ruby 从 1.8.7 版本开始添加了 `instance_exec` 方法，和 `instance_eval` 类似，但是它允许你传入块参数。

**更新**：感谢 Ruby 核心团队的 Yugui [纠正原文中的错误](http://yugui.jp/articles/846)，当时我忽略了在 `instance_eval` 中 `self` 是被分成两部分来对待的。
