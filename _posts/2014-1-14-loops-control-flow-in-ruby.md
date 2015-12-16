---
layout: post
title: Ruby 中的循环控制语句
---

Ruby 中常用到的循环控制语句包括：next, break, redo, retry 四种，下面就详细看看使用情景：

#### next
退出包含 next 的最内层循环的当前迭代，执行下一迭代

{% highlight ruby linenos %}
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    next if j == 3
    puts "j: #{j}"
  end
end
{% endhighlight %}

输出结果：

{% highlight shell %}
i: 0
j: 2
j: 4
i: 1
j: 2
j: 4
{% endhighlight %}

#### break
退出包含 break 的最内层循环

{% highlight ruby linenos %}
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    break if j == 3
    puts "j: #{j}"
  end
end
{% endhighlight %}

输出结果：

{% highlight shell %}
i: 0
j: 2
i: 1
j: 2
{% endhighlight %}

#### redo
重新执行包含 redo 的最内层循环的当前迭代

{% highlight ruby linenos %}
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    sleep 3
    puts "j: #{j}"
    redo if j == 3
  end
end
{% endhighlight %}

输出结果：

{% highlight shell %}
i: 0
j: 2
j: 3
j: 3
...  # 无限输出 j: # retry (Ruby 1.9 后移除)**
{% endhighlight %}

#### retry (Ruby 1.9 后移除)
重新执行包含 retry 的最内层循环

{% highlight ruby linenos %}
# 请使用 Ruby1.9 之前的版本运行
# 否则会抛异常：SyntaxError: (eval):8: Invalid retry
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    sleep 3
    puts "j: #{j}"
    retry if j == 3
  end
end
{% endhighlight %}

输出结果：

{% highlight shell %}
i: 0
j: 2
j: 3
j: 2
j: 3
...  # 无限交替输出 j: 2, j: 3
{% endhighlight %}

**参考：**

1. [Ruby Loops - while, for, until, break, redo and retry](http://www.tutorialspoint.com/ruby/ruby_loops.htm)
2. [Ruby Keywords](http://ruby-doc.org/docs/keywords/1.9/)
