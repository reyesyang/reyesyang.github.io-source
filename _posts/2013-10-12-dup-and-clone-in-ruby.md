---
layout: post
title: Ruby中的dup和clone
---

前段时间项目中有个地方需要在ActiveRecord对象被删除后继续使用对象的一些属性进行些操作，很自然想到了在删除前把对象复制一份，然后就接触到了dup和clone方法，看了文档后感觉二者还是比较容易搞混淆，结合一些搜索到的资料便做个记录。

**dup的官方文档：**

> [dup](http://ruby-doc.org/core-2.0.0/Object.html#method-i-dup)
>
> Produces a shallow copy of obj—the instance variables of obj are copied, but not the objects they reference. dup copies the tainted state of obj. See also the discussion under Object#clone. In general, clone and dup may have different semantics in descendant classes. While clone is used to duplicate an object, including its internal state, dup typically uses the class of the descendant object to create the new instance.
>
> This method may have class-specific behavior. If so, that behavior will be documented under the #initialize_copy method of the class.

大意是dup会进行浅拷贝--只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身。而且dup会拷贝对象的tainted状态。而特定类的dup方法可能会有额外的一些行为，具体参考该类对于initialize_copy方法的解释。

**clone的官方文档：**

> [clone](http://ruby-doc.org/core-2.0.0/Object.html#method-i-clone)
>
> Produces a shallow copy of obj—the instance variables of obj are copied, but not the objects they reference. Copies the frozen and tainted state of obj. See also the discussion under Object#dup.
>
> This method may have class-specific behavior. If so, that behavior will be documented under the #initialize_copy method of the class.

大意是clone会进行浅拷贝--只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身。而且clone会拷贝对象的tainted和frozen状态。而特定类的clone方法可能会有额外的一些行为，具体参考该类对于initialize_copy方法的解释。

**对两者的简单总结**

相同点：

* 浅拷贝(shallow copy)：只拷贝对象包含的实例变量，而不是实例变量所引用的对象本身
{% highlight ruby %}
class ProgramLanguage
  attr_accessor :name

  def initialize(name)
    @name = name
  end
end

ruby = ProgramLanguage.new "ruby"
ruby.name.object_id == ruby.dup.name.object_id  #=> true
ruby.name.object_id == ruby.clone.name.object_id  #=> true
{% endhighlight %}

* 拷贝对象的tainted状态
{% highlight ruby %}
ruby.taint
ruby.tainted?  #=> true
ruby.dup.tainted?  #=> true
ruby.clone.tainted?  #=> true
{% endhighlight %}

不同点：

* clone会拷贝对象的frozen状态
{% highlight ruby %}
ruby.freeze
ruby.frozen?  #=> true
ruby.dup.frozen?  #=> false
ruby.clone.frozen?  #=> true
{% endhighlight %}

* clone会拷贝对象的单例方法
{% highlight ruby %}
ruby = ProgramLanguauge.new "ruby"

def ruby.creator
  "Matz"
end

ruby.creator  #=> Matz
ruby.dup.creator  #=> undefined method `creator' for #<ProgramLanguage @name="ruby">
ruby.clone.creator  #=> Matz
{% endhighlight %}

Matz在对于clone和dup的描述如下：

> 'clone' copies everything; internal state, singleton methods, etc. 
> 'dup' copies object contents only (plus taintness status). 


Rails 中 dup 的变化

参考：

1. [只想与你深发展..](http://www.iteye.com/topic/407957)
2. [initialize_clone, initialize_dup and initialize_copy in Ruby](http://www.jonathanleighton.com/articles/2011/initialize_clone-initialize_dup-and-initialize_copy-in-ruby/)
3. [Ruby-talk about dup vs clone from Matz](http://computer-programming-forum.com/39-ruby/20826f090dd40ca3.htm)
