---
tags: ruby, closure
---

#### Proc

> Proc objects are blocks of code that have been bound to a set of local variables. Once bound, the code may be called in different contexts and still access those variables.

##### Proc::new

> Creates a new Proc object, bound to the current context. Proc::new may be called without a block only within a method with an attached block, in which case that block is converted to the Proc object.

```ruby
def proc_from
  Proc.new
end
proc = proc_from { "hello" }
proc.call   #=> "hello"
```

#### block

When we pass a block &param, then refer to that param without the ampersand, that is secretly a synonym for Proc.new(&param):

```ruby
def save_for_later(&b)
  @saved = Proc.new(&b) # same as: @saved = b
end

save_for_later { puts "Hello again!" }
puts "Deferred execution of a Proc works just the same with Proc.new:"
@saved.call
```

#### 参考

*   [Closure in Ruby](https://innig.net/software/ruby/closures-in-ruby)
*   [Ruby Core Doc: Proc](http://ruby-doc.org/core-2.2.4/Proc.html)
*   [Ruby Core Doc: Binding](http://ruby-doc.org/core-2.2.4/Binding.html)
*   [Back to Basics: Anonymous Functions and Closures](https://robots.thoughtbot.com/back-to-basics-anonymous-functions-and-closures)
*   [Understanding Ruby blocks, Procs and methods](http://eli.thegreenplace.net/2006/04/18/understanding-ruby-blocks-procs-and-methods/)
