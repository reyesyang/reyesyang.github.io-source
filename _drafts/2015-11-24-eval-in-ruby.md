---
title: Eval in Ruby
layout: post
---

#### Kernel#eval(string [, binding [, filename [,lineno]]]) → obj

> Evaluates the Ruby expression(s) in string. If binding is given, which must be a Binding object, the evaluation is performed in its context. If the optional filename and lineno parameters are present, they will be used when reporting syntax errors.


#### Binding#eval(string [, filename [,lineno]]) → obj

> Evaluates the Ruby expression(s) in string, in the binding’s context. If the optional filename and lineno parameters are present, they will be used when reporting syntax errors.

#### BasicObject#instance_eval(string [, filename [, lineno]] ) → obj

#### BasicObject#instance_eval {|obj| block } → obj

> Evaluates a string containing Ruby source code, or the given block, within the context of the receiver (obj). In order to set the context, the variable self is set to obj while the code is executing, giving the code access to obj’s instance variables and private methods.
>
> When instance_eval is given a block, obj is also passed in as the block’s only argument.
>
> When instance_eval is given a String, the optional second and third parameters supply a filename and starting line number that are used when reporting compilation errors.

#### Module#module_eval {|| block } → obj

#### Module#module_eval(string [, filename [, lineno]]) → obj

> Evaluates the string or block in the context of mod, except that when a block is given, constant/class variable lookup is not affected. This can be used to add methods to a class. module_eval returns the result of evaluating its argument. The optional filename and lineno parameters set the text for error messages.

Difference between pass string and proc as argument:

    CONST = '::CONST'

    class A
      CONST = 'A::CONST'
    end

    A.class_eval { puts CONST } # => ::CONST
    A.class_eval "puts CONST" # => A::CONST

Module#class_eval is alias for Module#module_eval 

    # http://ruby-doc.org/core-1.9.3/UnboundMethod.html#method-i-3D-3D
    Module.instance_method(:module_eval) == Module.instance_method(:class_eval)

1.  [Three implicit contexts in Ruby(Yuki Yugui Sonoda)](http://yugui.jp/articles/846)
1.  [Ruby Internals](http://www.slideshare.net/burkelibbey/ruby-internals)
1.  [Ruby constants have weird behavior in class_eval](https://www.pgrs.net/2007/09/12/ruby-constants-have-weird-behavior-in-class_eval/)
