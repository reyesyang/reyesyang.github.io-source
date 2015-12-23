---
title: Ruby 中三个隐含的上下文
tags: ruby
---

本文翻译自：Yugui 的文章 [Three implicit contexts in Ruby](http://yugui.jp/articles/846)

Yehuda Katz 写了一篇[关于 self 和 metaclass 的文章](http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)。文中他写道 `Person.instance_eval` 将 Person 类的 metaclass 赋值给 Person 类的 self。但显然这是错误的。

```ruby
class Person; end
Person.instance_eval { p self }  # => Person
```

正如我在[以前的一篇文章](http://yugui.jp/articles/558)中提到的（虽然很抱歉这篇文章是用日文写的），Ruby 始终含有三个隐含的上下文：self，所谓的 “klass” 和 constant definition point。而 Yehuda 是将 self 和 klass 给搞混了。

#### self

self 就是我们知道的 self，是方法调用时的默认接受者。self 始终都存在。

```ruby
p self                        # displays "main"

class Foo
  def bar(a = (p self)) end
end
foo = Foo.new
foo.bar                       # displays "#<Foo:0x471004>"

class Foo
  class Baz < (p self; self)  # displays "Foo"
  end
end
```

在顶层作用域中，self 是一个特殊的 Object 类实例，名为“main”。无论何时，你都可以通过伪变量 self 来获取 self 的值。

#### 所谓的 klass

在过去的一篇文章中，我称为 “klass” 的概念并不是一个好名称。指的是用来定义方法的默认类。现在，我想称它为 “default definee”。

Ruby 总是保存一个指向 class 的引用，类似于 `self`。但是并没有一个获取它的方法。它比 `self` 更隐晦。如果你定义一个方法时没有指定明确的接受者，换句话说，如果你使用普通的方法定义语法定义一个方法，那么 default definee 就将这个方法作为实例方法。

##### 示例

在顶层作用域中，Object is the class。所以全局函数实际上是 Object 的实例方法。

```ruby
def hoge; end
Object.instance_method(:hoge)   #=> #<UnboundMethod: Object#hoge>
```

顺便说句，“hoge”，“fuga”，“piyo” 在日文中类似于英文中的 “foo”，“bar”，“baz”。

`class` 语法同时转换 `self` 和 `default definee` 为正在定义的 class 本身。

```ruby
class T
  def hoge; end
end
T.instance_method(:hoge)    #=> #<UnboundMethod: T#hoge>
```

在普通的方法内部，self 是调用方法时的接受者，default definee 是语法外部类，这里就是 T。

```ruby
class T
  def hoge
    def fuga; end
  end
end
t = T.new
t.hoge
t.method(:fuga)           #=> #<Method: T#fuga>
T.instance_method(:fuga)  #=> #<UnboundMethod: T#fuga>
```

这里可不要把它和 `def self.fuga`，singleton 方法定义混淆了。当你在定义方法时指定了接受者的话，该方法就会添加到该接受者的 eigenclass(singleton_class) 上。

```ruby
class U
  def hoge
    def self.fuga; end
  end
end
u = U.new
u.hoge
u.method(:fuga)           #=> #<Method: #<U:0x46dbf4>.fuga>  
U.instance_method(:fuga)  # raises a NameError "undefined method `fuga' for class `U'"
```

U 类并没有 fuga 这个实例方法，因为 fuga 是 u 的单例方法（singleton method）。

无论何时，总是存在一个 default definee。当计算一个方法的默认参数时，default definee 和在方法内部一样，是这个方法的语法外部类。

```ruby
class Bar; end
$bar = Bar.new
class Baz
  def $bar.hoge(a = (def fuga; end))
    def piyo; end
  end
end
$bar.hoge

Baz.instance_method(:fuga)      #=> #<UnboundMethod: Baz#fuga>
Baz.instance_method(:piyo)      #=> #<UnboundMethod: Baz#piyo>

$bar.method(:fuga)              # raises a NameError "undefined method `fuga' for class `Bar'"
$bar.method(:piyo)              # raises a NameError "undefined method `piyo' for class `Bar'"
```

换句话说就是，class 定义会改变 default definee，但是方法定义不会。

#### eval 家族

`instance_eval` 做了什么

*   将 self 修改为 instance_eval 的接受者。
*   将 default definee 修改为 instance_eval 接受者的 eigenclass（singleton class）。
    *   如果接受者的 eigenclass 不存在，就新建一个
*   执行 block 中的代码

```ruby
o = Object.new
o.instance_eval do
  p self                            #=> #<Object:0x454f24>
  def hoge; end
end
o.method(:hoge)                   #=> #<Method: #<Object:0x454f24>.hoge>
Object.instance_method(:hoge)     # raises a NameError "undefined method `hoge' for class `Object'"
```

继续

```ruby
class T
  $o = Object.new
  $o.instance_eval do
    def hoge(a = (def fuga; end))
      def piyo; end
    end
  end
end
$o.hoge

$o.method(:fuga)           #=> #<Method: #<Object:0x42d144>.fuga>
$o.method(:piyo)           #=> #<Method: #<Object:0x42d144>.piyo>
T.instance_method(:fuga)   # raises a NameError
T.instance_method(:piyo)   # raises a NameError
```

因为 `instance_eval` 将 default definee 修改为 $o 的 eigenclass，所以 fuga 和 piyo 是 $o 的单例方法。

对了，需要说明的是

```ruby
RUBY_VERSION               #=> "1.9.1".
```

Ruby 1.8 表现地更接近于词法解析，所以你会得到相反的结果：

```ruby
$o.method(:fuga)
$o.method(:piyo)
T.instance_method(:fuga)
T.instance_method(:piyo)
```

在 Ruby 1.8 中，方法提内部的 default definee 是词法上的外部类定义。总之，在 Ruby 1.8 和 Ruby 1.9 中（此处应该有误：应该只是 Ruby 1.9），`instance_eval` 将 self 修改为接受者，将 default definee 都修改为接受者的 eigenclass。

最后，`class_eval` 将 self 和 default definee 都修改为接受者。

|                | self   | default definee     |
|----------------|--------|---------------------|
| class_eval     | 调用者 | 调用者              |
| instance_eval  | 调用者 | 调用者的 eigenclass |

在[过去的一片文章](http://yugui.jp/articles/558)中，我讨论了 `Kernel#eval` 和 `instance_eval/class_eval` 对字符串求值的情况。

#### 常量定义

当你使用实例变量时，就是 self 的实例变量。当你使用类变量时，就是 self 的类的实例变量；或者就是 self 自己的类变量，如果 self 是类的话。

但是常量并不太相同。这是 Ruby 中的另一个隐式上下文。Ruby Core 团队称之为 “cref”。

由于我现在必须给即将到来的 [RubyConf](http://www.rubyconf.org/) 准备演讲的材料。所以我只能晚点再聊 cref。
