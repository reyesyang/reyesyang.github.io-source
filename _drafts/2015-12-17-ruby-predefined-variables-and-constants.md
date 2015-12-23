---
title: Ruby 预定义变量及常量
tags: ruby
---

本文翻译自： [Ruby Docs: Pre-defined variables](http://ruby-doc.org/docs/ruby-doc-bundle/Manual/man-1.4/variable.html)

Ruby 解释器预定义了一些全局变量和常量如下：

#### 全局变量

##### 异常
*   `$!` 程序抛出异常时，在 rescue 语句中通过该变量获取异常。
*   `$@` 程序抛出异常时，在 rescue 语句中通过该变量获取异常的 backtrace，字符串数组，用来表示异常发生前代码的调用信息。 of the last exception, which is the array of the string that indicates the point where methods invoked from. 格式如下：

    "filename:line"
    or
    "filename:line:in `methodname'"
    (Mnemonic: where exception occurred at.)

示例：

```ruby
begin
  1 / 0
rescue
  puts $!.class
  puts $@.class
  puts $!
  puts $@
end
```

##### 正则表达式
*   `$&` 如果最近一次正则表达式匹配成功，返回匹配结果，否则返回 nil。只读变量。
*   `` $` `` 如果最近一次正则表达式匹配成功，返回匹配结果的前缀字符串，否则返回 nil。只读变量。
*   `$'` 如果最近一次正则表达式匹配成功，返回匹配结果的后缀字符串，否则返回 nil。只读变量。
*   `$+` The last bracket matched by the last successful search pattern, or nil if the last pattern match failed. This is useful if you don't know which of a set of alternative patterns matched. (Mnemonic: be positive and forward looking.)
*   `$1, $2...` Contains the subpattern from the corresponding set of parentheses in the last successful pattern matched, not counting patterns matched in nested blocks that have been exited already, or nil if the last pattern match failed. (Mnemonic: like \digit.) These variables are all read-only.
*   `$~` The information about the last match in the current scope. Setting this variables affects the match variables like $&, $+, $1, $2.. etc. The nth subexpression can be retrieved by $~[nth]. (Mnemonic: ~ is for match.) This variable is locally scoped.

示例：

```ruby
'abcde' =~ /(b)(c)d/
p $&           # 'bcd'
p $`           # 'a'
p $'           # 'e'
p $+           # 'c'
p $1, $2       # 'b'
               # 'c'
p $~           # #<MatchData "bcd" 1:"b" 2:"c">
```

*   `$=` The flag for case insensitive, nil by default. (Mnemonic: = is for comparison.)

*   `$/` The input record separator, newline by default. Works like awk's RS variable. If it is set to nil, whole file will be read at once. (Mnemonic: / is used to delimit line boundaries when quoting poetry.)

*   `$\` The output record separator for the print and IO#write. The default is nil. (Mnemonic: It's just like /, but it's what you get "back" from Ruby.)

*   `$,` The output field separator for the print. Also, it is the default separator for Array#join. (Mnemonic: what is printed when there is a , in your print statement.)

*   `$;` The default separator for String#split.

*   `$.` The current input line number of the last file that was read.

*   `$<` The virtual concatenation file of the files given by command line arguments, or stdin (in case no argument file supplied). $<.file returns the current filename. (Mnemonic: $< is a shell input source.)

*   `$>` The default output for print, printf. $stdout by default. (Mnemonic: $> is for shell output.)

*   `$_` The last input line of string by gets or readline. It is set to nil if gets/readline meet EOF. This variable is locally scoped. (Mnemonic: partly same as Perl.)

*   `$0` 正在执行的 Ruby 代码所在的文件名。Contains the name of the file containing the Ruby script being executed. On some operating systems assigning to $0 modifies the argument area that the ps(1) program sees. This is more useful as a way of indicating the current program state than it is for hiding the program you're running. (Mnemonic: same as sh and ksh.)

*   `$*` 执行脚本时传入的命令行参数列表。Command line arguments given for the script. The options for Ruby interpreter are already removed. (Mnemonic: same as sh and ksh.)

*   `$$` 当前正在运行的 Ruby 进程的 PID。The process number of the Ruby running this script. (Mnemonic: same as shells.)

*   `$?` 最近一次执行的子进程状态。The status of the last executed child process.

*   `$:` The array contains the list of places to look for Ruby scripts and binary modules by load or require. It initially consists of the arguments to any -I command line switches, followed by the default Ruby library, probabl "/usr/local/lib/ruby", followed by ".", to represent the current directory. (Mnemonic: colon is the separators for PATH environment variable.)

*   `$"` 已经 require 的 module 数组。用来防止同一个 module 被 require 两次。（助记：防止文件被两次引用加载。）The array contains the module names loaded by require. Used for prevent require from load modules twice. (Mnemonic: prevent files to be doubly quoted(loaded).)

*   `$DEBUG` The status of the -d switch.

*   `$FILENAME`：同 `$<.filename`。

*   `$LOAD_PATH`：`$:` 的别名。

*   `$stdin`：标准输入流。 

*   `$stdout`：标准输出流。

*   `$stderr`：标准错误输出流。

*   `$VERBOSE`：verbose 标志，which is set by the -v switch to the Ruby interpreter.

##### option variables
The variables which names are in the form of $-?, where ? is the option character, are called option variables and contains the information about interpreter command line options.

*   `$-0`：`$/` 的别名。

*   `$-a`：True if option -a is set. Read-only variable.

*   `$-d`：`$DEBUG` 的别名。

*   `$-F`：`$;` 的别名。

*   `$-i`：In in-place-edit mode, this variable holds the extention, otherwise nil. Can be assigned to enable (or disable) in-place-edit mode.

*   `$-I`：`$:` 的别名。

*   `$-l`：True if option -lis set. Read-only variable.

*   `$-p`：True if option -pis set. Read-only variable.

*   `$-v`：`$VERBOSE` 的别名。

#### 常量

#### 参考

1.  [Ruby Docs: Pre-defined variables](http://ruby-doc.org/docs/ruby-doc-bundle/Manual/man-1.4/variable.html)
