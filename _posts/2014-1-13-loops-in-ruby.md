---
layout: post
title: Ruby 中的循环语句
---

由于 Ruby 提供了很多便捷的方法进行对象的循环遍历，如：Array#each, Integer#times, Integer#downto, Integer#upto 等，所以日常的开发中很少会需要写循环语句。便整理了相关内容，方便日后查看参考。Ruby 中的循环语句有 for..in, while, until 和 loop  四种，具体使用方法如下：

**1. for .. in**  
遍历集合中的所有成员

```ruby
for i in 0..3
  puts i
end
```

**2. while**  
当条件成立时，执行循环体

```ruby
i = 0
while i <= 3 do  # do 可以省略
  puts i
  i += 1
end
```

while 可以后置，循环体至少执行一次

```ruby
i = 0
begin
  puts i
  i += 1
end while i <= 3
```

**3. until**  
当条件不成立时，执行循环体

```ruby
i = 0
until i > 3 do  # do 可以省略
  puts i
  i += 1
end
```

和 while 一样，until 也可以后置，循环体至少执行一次

```ruby
i = 0
begin
  puts i
  i += 1
end until i > 3
```

**4. loop**  
无限循环

```ruby
loop do
  puts 0
end
```

**参考：**

1. [Ruby学习札记(7)-Ruby中具有循环控制的方法和语句大归纳](http://blog.csdn.net/daydreamingboy/article/details/6725328)
2. [Ruby While and Until Loops](http://www.techotopia.com/index.php/Ruby_While_and_Until_Loops)
3. [Looping with for and the Ruby Looping Methods](http://www.techotopia.com/index.php/Looping_with_for_and_the_Ruby_Looping_Methods)
4. [Ruby Loops - while, for, until, break, redo and retry](http://www.tutorialspoint.com/ruby/ruby_loops.htm)
