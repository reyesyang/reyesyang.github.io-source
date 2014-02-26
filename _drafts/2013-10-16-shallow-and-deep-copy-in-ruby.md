---
layout: post
title: Ruby 中的 Shallow copy 和 Deep copy
---

###什么是 Shallow copy 和 Deep copy?

* Shallow copy

  > One method of copying an object is the shallow copy. In the process of shallow copying A, B will copy all of A's field values. If the field value is a memory address it copies the memory address, and if the field value is a primitive type it copies the value of the primitive type.
  >
  > The disadvantage is if you modify the memory address that one of B's fields point to, you are also modifying what A's fields point to.

* Deep copy

  > An alternative is a deep copy. Here the data is actually copied over. The result is different from the result a shallow copy gives. The advantage is that A and B do not depend on each other but at the cost of a slower and more expensive copy.


两者最主要区别就是如何处理引用对象，Shallow copy 只复制了对象地址，而 Deep copy 则复制了被引用对象本身。

###Ruby 中的 Shallow copy

Ruby 提供的两个复制对象的方法 `Object#dup` 和 `Object#clone` 都是 Shallow copy。

###Ruby 中的 Deep copy

Ruby 默认没有提供 Deep copy，通常有三种方法解决这个问题：

方法一，使用序列化和反序列化：

{% highlight ruby %}
class Object
  def deep_clone
    Marshal::load Marshal.dump(self)
  end
end
{% endhighlight %}

不足之处是 Marshel 不能 dump 下面这些对象：

1. anonymous Class/Module.
2. objects which related to its system (ex: Dir, File::Stat, IO, File, Socket and so on)
3. an instance of MatchData, Data, Method, UnboundMethod, Proc, Thread, ThreadGroup, Continuation
4. objects which defines singleton methods

方法二：

{% highlight ruby %}
class Object
  def dclone
    case self
      when Fixnum,Bignum,Float,NilClass,FalseClass,
           TrueClass,Symbol
        klone = self
      when Hash
        klone = self.clone
        self.each{|k,v| klone[k] = v.dclone}
      when Array
        klone = self.clone
        klone.clear
        self.each{|v| klone << v.dclone}
      else
        klone = self.clone
    end
    klone.instance_variables.each {|v|
      klone.instance_variable_set(v,
        klone.instance_variable_get(v).dclone)
    }
    klone
  end
end
{% endhighlight %}

方法三：

{% highlight ruby %}
class Object
  def deep_clone
    _deep_clone({})
  end

  protected
  def _deep_clone(cloning_map)
    return cloning_map[self] if cloning_map.key? self
    cloning_obj = clone
    cloning_map[self] = cloning_obj
    cloning_obj.instance_variables.each do |var|
      val = cloning_obj.instance_variable_get(var)
      begin
        val = val._deep_clone(cloning_map)
      rescue TypeError
        next
      end
      cloning_obj.instance_variable_set(var, val)
    end
    cloning_map.delete(self)
  end
end

class Array
  protected
  def _deep_clone(cloning_map)
    return cloning_map[self] if cloning_map.key? self
    cloning_obj = super
    cloning_map[self] = cloning_obj
    cloning_obj.map! do |val|
      begin
        val = val._deep_clone(cloning_map)
      rescue TypeError
        #
      end
      val
    end
    cloning_map.delete(self)
  end
end

class Hash
  protected
  def _deep_clone(cloning_map)
    return cloning_map[self] if cloning_map.key? self
    cloning_obj = super
    cloning_map[self] = cloning_obj
    pairs = cloning_obj.to_a
    cloning_obj.clear
    pairs.each do |pair|
      pair.map! do |val|
        begin
          val = val._deep_clone(cloning_map)
        rescue TypeError
          #
        end
        val
      end
      cloning_obj[pair[0]] = pair[1]
    end
    cloning_map.delete(self)
  end
end
{% endhighlight %}

参考：

* [只想与你深发展..](http://www.iteye.com/topic/407957)
* [Object copy](http://en.wikipedia.org/wiki/Object_copy)
* [What is the difference between a deep copy and a shallow copy?](http://stackoverflow.com/questions/184710/what-is-the-difference-between-a-deep-copy-and-a-shallow-copy)
* [Deep Cloning](http://www.artima.com/forums/flat.jsp?forum=123&thread=40913)
* [深いコピーの作成(2) - pegacornの日記](http://d.hatena.ne.jp/pegacorn/20070417/1176817721)

