---
layout: post
title: Ruby 中的循环控制语句
---

**1. next**  
退出包含 next 的最内层循环的当前迭代，执行下一迭代

```ruby
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    next if j == 3
    puts "j: #{j}"
  end
end
```

输出结果：

```bash
i: 0
j: 2
j: 4
i: 1
j: 2
j: 4
```

**2. break**  
退出包含 break 的最内层循环

```ruby
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    break if j == 3
    puts "j: #{j}"
  end
end
```

输出结果：

```bash
i: 0
j: 2
i: 1
j: 2
```

**3. redo**  
重新执行包含 redo 的最内层循环的当前迭代

```ruby
for i in 0..1
  puts "i: #{i}"

  for j in 2..4
    sleep 3
    puts "j: #{j}"
    redo if j == 3
  end
end
```

输出结果：

```bash
i: 0
j: 2
j: 3
j: 3
...  # 无限输出 j: 3 
```

**4. retry (Ruby 1.9 后移除)**  
重新执行包含 retry 的最内层循环

```ruby
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
```

输出结果：

```bash
i: 0
j: 2
j: 3
j: 2
j: 3
...  # 无限交替输出 j: 2, j: 3
```

**参考：**

1. [Ruby Loops - while, for, until, break, redo and retry](http://www.tutorialspoint.com/ruby/ruby_loops.htm)
2. [Ruby Keywords](http://ruby-doc.org/docs/keywords/1.9/)