---
title: "[翻译] 理解 Ruby 中的改良和词法作用域"
tags: ruby
---

本文翻译自 Starr Horne 发表在 Honeybadger 上的文章：[Understanding Ruby Refinements and Lexical Scope](http://blog.honeybadger.io/understanding-ruby-refinements-and-lexical-scope/)

如果你之前还没有使用过改良，这可能对你是个惊喜。你可能已经听说过 Ruby 引入改良（Refinements）是为了替换猴子补丁（Monkey Patch）。所以你可能会期待这你能够实现一个类似 ActiveSupport 中的 `hours` 方法：

```ruby
module TimeExtension
  refine Fixnum do
    def hours
      self * 60
    end
  end
end

class MyFramework
  using TimeExtension
end

class MyApp < MyFramework
  def index
    1.hours
  end
end

MyApp.new.index  # undefined method `hours` for 1:Fixnum (NoMethodError)
```

如果你运行上面的代码，就会发现它没法工作。所以发生了什么？


像其他伟大的想法一样，改良的原始概念不得不经过一些优化才能使它在残酷的现实中工作。

我们看到上面代码令人吃惊的表现就是由于这些优化之一导致的。尤其那条规则说改良是词法范围的。

#### 什么是词法范围？

当我们说什么事情是词法的，意味着不得不和文本打交道 —— 屏幕中的代码而不是代码的含义。

如果有两行代码是词法范围的，简而言之就是他们出现在同一个代码块中 —— 而不用在乎代码块会如何计算。

比起上面说这么多，看例子更简单一些：

```ruby
class B
  # x and y share the same lexical scope
  x = 1
  y = 1
end

class B
  # z has a different lexical scooe from x and y, even though it's in the same class.
  z = 3
end
```

#### 改良是词法范围的

当我们通过 `using` 关键字使用改良时，改良仅在词法作用域中可见。

下面的例子能表达出我的意思：

```ruby
module TimeExtension
  refine Fixnum do
    def hours
      self * 60
    end
  end
end

class MyApp
  using TimeExtension
  def index
    1.hours
  end
end

class MyApp
  def show
    2.hours
  end
end

MyApp.new.show  # undefined method `hours` for 1:Fixnum (NoMethodError)
```

即使 `index` 和 `show` 方法属于同一个类，但是只有 `index` 方法可以访问改良，因为它只通过 `using` 语句来共享词法作用域。

这看起来可能有些奇怪，因为几乎所有的东西在 Ruby 中都是动态范围的。当你给类添加了一个方法时，即使你推出了类定义，这个方法还保存在那里。但是改良并不是这样工作的。

这是若干个首次不太明显的影响。

#### 你不能动态调用改良后的方法

因为 `send` 方法没有和包含 `using` 语句的代码定义在同一个代码块中，所以看不到这个改良后的方法。

所以下面这些并不工作：

```ruby
class MyApp
  using TimeExtension
  def index
    1.send(:hours)
  end
end
```

#### 你不能查询改良后的方法是否存在

`respond_to?` 方法也没有在同一个代码块中。所以因为同样的原因 `send` 方法也不工作。

```ruby
class MyApp
  using TimeExtension
  def index
    1.respond_to?(:hours)
  end
end
```

#### 总结

我希望通过这篇文章能扫除一些我发现其他人在使用改良时遇到的混乱。它们无疑是 Ruby 中的一个潜在的很有用的功能，但是如果你从没有使用它们，它们可能会让你感到有些棘手。
