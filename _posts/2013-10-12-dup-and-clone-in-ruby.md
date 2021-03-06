---
title: Ruby 中的 dup 和 clone
tags: [Ruby]
---

<del>前段时间项目中有个地方需要在 ActiveRecord 对象被删除后继续使用对象的一些属性进行些操作，很自然想到了在删除前把对象复制一份</del>(删除说明：AR 对象调用 destroy 方法删除对象时，也只是删除了数据库中的记录，实际上该对象还是存在的，不过已经变为了 frozen 的状态)，然后就接触到了 dup 和 clone 方法，看了文档后感觉二者还是比较容易搞混淆，结合一些搜索到的资料便做个记录。

#### dup [官方文档](http://ruby-doc.org/core-2.0.0/Object.html#method-i-dup)

> Produces a shallow copy of obj—the instance variables of obj are copied, but not the objects they reference. dup copies the tainted state of obj. See also the discussion under Object#clone. In general, clone and dup may have different semantics in descendant classes. While clone is used to duplicate an object, including its internal state, dup typically uses the class of the descendant object to create the new instance.
>
> This method may have class-specific behavior. If so, that behavior will be documented under the #initialize_copy method of the class.

大意是 dup 会进行浅拷贝--只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身。而且 dup 会拷贝对象的 tainted 状态。而特定类的 dup 方法可能会有额外的一些行为，具体参考该类对于 initialize_copy 方法的解释。

#### clone [官方文档](http://ruby-doc.org/core-2.0.0/Object.html#method-i-clone)

> Produces a shallow copy of obj—the instance variables of obj are copied, but not the objects they reference. Copies the frozen and tainted state of obj. See also the discussion under Object#dup.
>
> This method may have class-specific behavior. If so, that behavior will be documented under the #initialize_copy method of the class.

大意是 clone 会进行浅拷贝--只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身。而且 clone 会拷贝对象的 tainted 和 frozen 状态。而特定类的 clone 方法可能会有额外的一些行为，具体参考该类对于 initialize_copy 方法的解释。

#### 对两者的简单总结

相同点：

1.  浅拷贝(shallow copy)：只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身

    ```ruby
    class ProgramLanguage
      attr_accessor :name

      def initialize(name)
        @name = name
      end
    end

    ruby = ProgramLanguage.new "ruby"
    ruby.name.object_id == ruby.dup.name.object_id  #=> true
    ruby.name.object_id == ruby.clone.name.object_id  #=> true
    ```

2.  拷贝对象的 tainted 状态

    ```ruby
    ruby.taint
    ruby.tainted?  #=> true
    ruby.dup.tainted?  #=> true
    ruby.clone.tainted?  #=> true
    ```

不同点：

1.  clone 会拷贝对象的 frozen 状态

    ```ruby
    ruby.freeze
    ruby.frozen?  #=> true
    ruby.dup.frozen?  #=> false
    ruby.clone.frozen?  #=> true
    ```

2.  clone 会拷贝对象的单例方法

    ```ruby
    ruby = ProgramLanguauge.new "ruby"

    def ruby.creator
      "Matz"
    end

    ruby.creator  #=> Matz
    ruby.dup.creator  #=> undefined method `creator' for #<ProgramLanguage @name="ruby">
    ruby.clone.creator  #=> Matz
    ```

#### Matz 对 clone 和 dup 的描述如下：

> 'clone' copies everything; internal state, singleton methods, etc.
> 'dup' copies object contents only (plus taintness status).


<!-- Rails 中 dup 的变化 -->

#### 参考

*   [只想与你深发展..](http://www.iteye.com/topic/407957)
*   [initialize_clone, initialize_dup and initialize_copy in Ruby](http://www.jonathanleighton.com/articles/2011/initialize_clone-initialize_dup-and-initialize_copy-in-ruby/)
*   [Ruby-talk about dup vs clone from Matz](http://computer-programming-forum.com/39-ruby/20826f090dd40ca3.htm)
