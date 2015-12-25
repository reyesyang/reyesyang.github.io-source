---
title: "[翻译] 理解 Ruby 中的作用域"
tags: [Ruby, Scope]
---

本文翻译自 Darko Gjorgjievski 发表在 sitepoint 上的文章 [Understanding Scope in Ruby](http://www.sitepoint.com/understanding-scope-in-ruby/)

作用域（Scope）是非常必要去理解的概念，而且不仅仅是在 Ruby 中，在其他编程语言中也一样。回想起我以前刚开始使用 Ruby 的日子，编程中遇到的大半错误都是由于我没有充分理解这个概念。比如为定义变量，变量的错误赋值等等。一旦你很好的理解了作用与，就不会再遇到这些让人头痛的问题啦！希望通过这篇文章能帮助你避免很多我翻过的错误。

#### 作用域究竟是什么？

当人们提起作用域的时候，通常会想到两个词语：**变量** 和 **可见性**。这很直观：作用域就是在哪些地方可以访问哪些东西。就是在具体的某一时刻，什么东西（变量，常量，方法）是你可以使用的。如果你理解了作用域，你就可以指出 Ruby 程序中任何一行代码的上下文中，哪些变量是可以使用的，更重要的是，哪些不能使用。

那么首先让我们考虑下限制变量可见性的原因是什么？为什么不让所有变量在任何时候都能使用？那样的话编程不就更方便一些，不是吗？的确没有这么简单……

程序员们有很多事情存在争议（使用函数式还是面向对象的方式，使用不同的变量命名模板等等），但是作用域并不在其中。尤其当程序员随着编程阅历的增加，他们更加支持作用域的存在，为什么呢？

因为随着越来越多的编程阅历的增加，你越有机会体会到让所有的变量任何时候都能访问到是多么可怕的事情。[全局状态(Global State)](http://programmers.stackexchange.com/questions/148108/why-is-global-state-so-evil)让我们的程序运行非常难以预测。当所有人都能修改它时，我们很难追踪到始作俑者。谁修改了这个变量？谁读取了它？想想如果我们在面对成千上万行代码的时候都会面临这样的问题！

而且还会引入命名问题。假设你有一庞大的程序，以免命名冲突，你不得不给每一个变量一个唯一的名字。想想，这就需要你跟踪成千上万的变量名。

作用域类似于计算机科学的另一个原则：[最小可访问原则（The Principle of Least Access）](http://etutorials.org/Server+Administration/securing+windows+server+2003/Chapter+2.+Basics+of+Computer+Security/2.3+POLA+The+Principle+of+Least+Access/)。想象一下如果银行中的每个员工（出纳，会计）都能（读取和修改）每个客户，财务等等的记录。突然，有人修改了客户的存款余额。是出纳还是会计做的？还是有可能他们都做过？你明白我要表达的意思了吧。

#### Ruby 变量作用域：快速参考

你可能已经了解了，Ruby 在不同的作用域中定义了变量。我知道很多教程都已经简要的描述了它们（变量的类型），但是它们却恰恰没有提到它们的作用域是什么。这里我将详细说明下：

*   类变量（@@a_variable）：在类和所有子类的定义中有效，其余地方均无效。
*   实例变量（@a_variable）：仅在某个特定的对象中有效，贯穿一个类实例中的所有方法。但是在类定义中直接使用无效。
*   全局变量（$a_variable）：在 Ruby 脚本的任何地方都有效。
*   局部变量（a_variable）：取决于作用域。这是我们最常打交道的，也是最容易出错的，因为它依赖于很多因素。

通过[一个插图](http://natashatherobot.com/ruby-variable-scope/)来直观的展示下这四种不同作用域的可见性。你将对他们的范围和有效性有一个大概的了解：

![](http://77fmoa.com1.z0.glb.clouddn.com/blog/variable-scope-ruby.jpg)

下面的篇章中我将主要讨论局部变量。我发现，就个人的经验和与别人的聊天来看，大多数的问题来自于对局部变量的了解不足以及不明白它们如何工作的。

#### 作用域中什么情况下是局部变量？

在 Ruby 中你不需要声明实例变量。你可以试试在一个方法中输入`@anything_goes_here`，那么你将得到一个 nil。现在，让我们删掉变量名开始的`@`字符（将该变量转变成一个局部变量），重新运行代码，你将得到`NameError(undefined local variable or method)`。

Ruby 解释器一旦遇到你给一个局部变量赋值，它就会将这个局部变量放入作用域中。它不关心这段代码有没有执行，一旦解释器看到赋值给一个局部变量，就放入作用域中：

```ruby
if false  # the code blow will not run
  a = 'hello' # the interpreter saw this, so the local var. is in scope from now on.
end
p a # nil, since that code didn't execute, thus the variable wasn't initialized
```

尝试将`a = 'hello'`移除后再运行这段代码，看看会发生什么。

#### 局部变量命名冲突

假如你有如下代码：

```ruby
def something
  'hello'
end

p something
==> hello
something = 'Ruby'
p something
==> Ruby  # 'hello' is not printed
```

在 Ruby 中，方法调用可以没有显示的接收者或括号，就像局部变量一样。所以，你可能会遇到像上面一样的潜在命名冲突。

如果同一个作用域中有一个初始化过的局部变量和方法重名，那么局部变量将会“遮蔽”方法并取得优先访问权。但这并不意味着方法没有了并且再也不能调用了。你可以通过在方法名后添加括号（如 `something()`）或者在方法名前添加  self 作为显示的接收者（如`self.something`）来调用该方法。让我们看看：

```ruby
def  some_var; 'I am a method'; end
public :some_var
some_var = 'I am a variable'
p some_var  # I am a variable
p some_var()  # I am a method
p self.some_var  # I am a method
```

#### 一个有用的测试来检查变量是否在作用域之外

如果你想看看一个局部变量是否在作用域中，首先，将你的鼠标指针指向它。现在，在代码中按下鼠标左键向前拖拽到知道遇到下面任意一个停止：

1.  到达作用域的起始处（def/class/module/do-end 块）
1.  到达局部变量赋值语句

如果你先到达 1) 再到达 2)，你的代码很有可能就会遇到`NameError`。如果你先到达 2) 再到达 1)，那么恭喜你！

#### 局部变量 vs. 实例变量

实例变量和一个特定的对象关联。只要你在对象内部，就可以访问它们。局部变量和实例变量不同，是和特定的作用域关联。只要你在这个这个作用域中，就可以访问它们。实例变量会在新的对象中被替换掉。局部变量会在新的作用域中被替换掉。那和你怎么知道作用域变了？两个词：作用域门（scope gates）。

#### 作用域门（Scope gate）：理解作用域的关键

发生下面的事情时，你觉得发生了什么：

1.  定义类（`class SomeClass`）
1.  定义模块（`module SomeModule`）
1.  定义方法（`def some_method`）

每次你做上面三种事情的任何一个时，你都进入了一个新的作用域中。就好像 Ruby 为你打开了一扇通往不同上下文和完全不同变量的门。

每个类、模块、方法定义都是一个作用域门，因为会创建一个新的作用域。老的作用域将不再生效，其中的所有变量将被新的替换。

很困惑是吧，不过别担心。让我们通过下面的示例来帮助你更好的掌握这个概念：

```ruby
v0 = 0
class SomeClass
  v1 = 1
  p local_variables

  def some_method
    v2 = 2
    p local_variables
  end
end

some_class = SomeClass.new
some_class.some_method
```

你将会看到 [:v1] 和 [:v2] 显示在终端中。当程序运行到 `def some_method` 时，v1 变量发生了什么？类作用域被替换为实例方法作用域。将新的变量混入（在这里，只有一个）。

那么 v0 呢？它在任何地方都没有显示！是的，当程序进入 `class SomeClass` 时，顶级作用域（v0 定义在顶级作用域中）也被替换掉了。这里说的“替换”，只是临时的，并非永久替换。在这段程序的末尾添加 `p local_variables` 你将会看到 v0 还在。

#### 打破作用域门！

我们前面提到类、模块、方法定义中，会限制变量的可见性。如果你在类中有一个局部变量，当在该类中定义一个新方法时，如我们所见，类中的局部变量将不再有效。那么如果你还想在方法定义中访问这些变量的话怎么办？我们如何“打破”这个作用域门？

说来其实也很简单：使用方法调用替换掉作用域门。具体说就是：

*   类定义使用 `Class.new` 替换
*   模块定义使用 `Module.new` 替换
*   方法定义使用 `define_method` 替换

让我们看看上面的代码示例（变量名等其他都保持一样），使用方法调用替换作用域门：

```ruby
v0 = 0
SomeClass = Class.new do
  v1 = 1
  p local_variables

  define_method(:some_method) do
    v2 = 2
    p local_variables
  end
end

some_class = SomeClass.new
some_class.some_method
```

运行代码，你会看到打印出了：[:v1, :v0, :some_class] 和 [:v2, :v1, :v0, :some_class]。我们成功的打破了作用域门，使得外面的作用域依旧有效。这段代码之所以有效得益于代码块（block）的强大功能，我们会在下面详细解释。

#### 代码块是作用域门吗？

你可能回认为代码块也是作用域门。毕竟，他们引入了一个新的作用域，而且你不能在外部访问定义在其中的变量，如下例：

```ruby
sample_list = [1,2,3]
hi = '123'
sample_list.each do |item|
  puts hi
  hello = 'hello'
end

p hello
```

正如我们所见，定义在特定代码块内部的变量是该代码块的局部变量，其他地方不能访问。

如果代码块是作用域门，那么 `puts hi` 应该产生一个错误，因为变量 `hi` 在另一个作用域中。但这里不同，你可以看到上面的代码可以运行。

你不但可以读取外部的变量，而且可以修改他们！试着将 `hi = '456'` 添加到 `do/end` 内部，它的值就会被修改。

如果你不想代码块修改外部的变量该怎么办呢？代码块局部变量可以帮上忙。定义一个代码块局部变量，在代码块参数（下面的代码块仅有一个参数，`i`）后面添加分号，然后列出他们：

```ruby
hi = 'hi'
hello = 'hello'
3.times do |i; hi, hello|
  p i
  hi = 'hi again'
  hello = 'hello again'
end
p hi  # "hi"
p hello  # "hello"
```

如果我们删除 `; hi, hello` 语句，你将会得到 “hi again” 和 “hello again” 作为两个变量的新值。

记住在代码块的 `do` 和 `end` 之间，我们引入了一个新的作用域：

```ruby
[1,2,3].select do |item|  # do is here, new scope is being introduced
  # some code
end
```
使用 `each`、`map`、`detect` 或者其他方法替换 `select`。事实就是当使用代码块时（当你看到 `do/end`）意味着一个新的作用域被引入了。

#### 代码块和作用域的一些怪异行为

猜猜下面的 Ruby 代码会打印出什么：

```ruby
2.times do
  i ||= 1
  print "#{i}"
  i += 1
  print "#{i}"
end

你期待的是 `1 2 2 2` 吗？答案是：`1 2 1 2`。`times` 在每次迭代时，使用的都是一个新的重置了内部局部变量的代码块定义。本例中有两次迭代，所以第二次迭代开始时，`i` 已经被再次重置为 1 了。

下面的 Ruby 代码将会打印什么（最后一行）：

```ruby
def foo
  x = 1
  lambda { x }
end

x = 2

p foo.call
```

答案是 1。因为代码块和代码块对象（procs, lambda）能访问定义他们时的作用域而不是调用他们时的。因为在 Ruby 中他们被视为闭包（Closure），所以不得不具有这样的性质。闭包是一段含有以下特征的代码：

*   和对象一样被传递（可以随后被调用）
*   能够记住定义该闭包的作用域中的变量

这在很多情况下都能拍上用场，比如定义一个无线的数字生成器：

```ruby
def increase_by(i)
  start = 0
  lambda { start += 1 }
end

increase = increase_by(3)
start = 453534534
p increase.call
p increase.call
```

你也可以使用 lambda 延迟变量的定义：

```ruby
i = 0
a_lambda = lambda do
  i = 3
end

p i  # 0
a_lambda.call
p i  # 3
```

你认为最后一行会打印出什么：

```ruby
a = 1
ld = lambda { a }
a = 2
p ld.call
```

如果你的答案是 1，那就错了。应该打印出 2。但是等等，不是说 lambda/proc 访问定义它们时的作用域吗？这是正确的，如果你这么想，`a = 2` 也在定义的作用域中。直到第一次调用 lambda 时，才能确定变量在定义中的值，正如你在上面例子中看到的一样。还没意识到实际上这可能会引入难以追踪的 bug。

#### 两个方法如何分享同一个变量？

一旦我们如何打破作用域门，我们就可以使用它做一些令人惊奇的事情。我从 [Metaprogramming Ruby](http://www.amazon.com/Metaprogramming-Ruby-Program-Like-Facets/dp/1941222129/ref) 这本书中学到的这个概念，这本书帮助我理解了作用域背后的工作原理。总之，让我们看代码：

```ruby
def let_us_define_methods
  shared_variable = 0

  Kernel.send(:define_method, :increase_var) do
    shared_variable += 1
  end

  Kernel.send(:define_method, :decrease_var) do
    shared_variable -= 1
  end
end

let_us_define_methods
p increase_var  # 1
p increase_var  # 2
p decrease_var  # 1
```

简洁漂亮，对不对啊？

#### 顶级作用域

在 Ruby 或者其他编程语言中顶级作用域意味着什么呢？你如何确认自己在顶级作用域中？在顶级作用域中，意味着要不你还没有调用任何方法，要么就是所有的方法已经调用完毕返回。

在 Ruby 中，一切皆对象。即使你在顶级作用域中，你仍然身处在一个对象中（名为 `main`，是 Object 类的对象）。试着自己运行下面的代码看看：


```ruby
p self  # main
p self.class  # Object
```

#### 我在哪里？

通常来说，调试的时候，如果我们知道当前 `self` 的值的话，[很多令你头痛的问题](http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)就解决了。当前 `self` 的值会影响实例变量和没有显式接收者的方法。如果对于你确定已经定义（因为你能在代码中看到它）的方法，在使用的时候却遇到了 `undefined method/instance variable` 的错误时，那就可能是在追踪 `self` 时遇到了问题。

#### 小测试：哪些对我有效？

通过下面的小测试来确认下你已经明白了本篇中的基本原则。假设你是一个 Ruby 调试器，并且正在运行下面的代码：

```ruby
class SomeClass
  b = 'hello'
  @@m = 'hi'
  def initialize
    @some_var = 1
    c = 'hi'
  end

  def some_method
    sleep 1000
    a = 'hello'
  end
end

some_object = SomeClass.new
some_object.some_method
```

试着在代码 `sleep 1000` 时停下。你能看到什么？在当前这个点上，哪些变量对你是有效？在你继续运行之前试着猜猜答案。你的答案不但要含有哪些变量，而且要知道他们为什么有效的原因。

正如我们前面提到的，局部变量是和作用域绑定的。`some_method` 的定义是作用域门，替换前面所有的作用域并开始一个新的。在新作用域中，`a` 变量是唯一有效的局部变量。

实例变量，像我们前面提到过的，是和 `self` 绑定的，这里，`some_object` 是当前的实例，`@some_var` 在关于它的所有方法中都是有效的，包括 `some_method`。类变量是类似的，`@mm` 在这个作用域中也是有效的。局部变量 `b` 和 `c` 访问不到，因为作用域门的存在。如果你想他们在任何地方都有效，参考打破作用域门的章节。

希望这篇文章对你有所帮助。如果你有任何问题，在下面留言个我！
