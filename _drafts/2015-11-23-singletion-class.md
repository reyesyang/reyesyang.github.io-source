---
title: Singleton Class(MetaClass) in Ruby
layout: post
---

又名 object-specific classes, anonymous classes, virtual classes, and eigenclasses 等，官方使用 singleton classes

#### 获取对象的 singleton class

Ruby 1.9.2 之前需要自己实现下

    class Object
      def singleton_class
        class << self
          self
        end
      end
    end

从 Ruby 1.9.2 开始，Object 默认已经添加了 singleton_class 方法

    Object.new.singleton_class  #=> #<Class:#<Object:0xb7ce1e24>>
    String.singleton_class      #=> #<Class:String>
    nil.singleton_class         #=> NilClass

Singleton Class 的来由

> Object do not store methods, only classes can.  
> -- \_why

Singleton Class 是什么

>  I like to think of the Ruby metaclass as "a class which an object uses to redefine itself."  
> -- \_why

Singleton Class 的特性

*   不能实例化
*   修改了继承链，但是 Object.class 时跳过它
*   如果 singleton_methods 不

##### Singleton Pattern

1.  [See Meta Class Clearly(\_why)](http://viewsourcecode.org/why/hacking/seeingMetaclassesClearly.html)
1.  [Understanding Ruby Singleton Classes(devalot)](http://www.devalot.com/articles/2008/09/ruby-singleton) 
1.  [Singleton methods and metaclasses(rubymonk)](https://rubymonk.com/learning/books/4-ruby-primer-ascent/chapters/39-ruby-s-object-model/lessons/131-singleton-methods-and-metaclasses)
