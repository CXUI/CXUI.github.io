---
layout: post
title:  "Ruby 基础教程第4版读书笔记 1"
date:   2014-11-04 18:30:00
categories: ruby
tags: learningnote
author: "Victor"
---

## Ruby 初体验

* ```print("Hello, \nRuby!")``` 程序会对双引号里 ```\``` 后的字符进行转义，而单引号中的则不会。
* ```puts "Hello, ", "Ruby!"``` 当参数为两个字符串的时候，各字符串末尾都会加上换行符。
* 使用 p 方法，打印出的数值结果和字符串结果会以不同的形式输出。另外其中的转移符不会转义。
* Ruby 2 开始默认编码使用 UTF-8 编码方式，如果输出中文有乱码的话，可以用 -E 指定输出结果的编码方式。

```ruby
ruby -E UTF-8 hello.rb
irb -E UTF-8
```

* irb 命令后的选项 --simple-prompt 会简化 irb 的输出结果。
* 利用 **Math** 模块进行的数学运算的返回结果是科学计数法 9.26535896604902e-05
* ```--begin``` 和 ```=end``` 中间的部分也是注释。
* ```while``` 语句可以省略后面的 ```do```
* 正则表达式可以简单的实现：
  * 将字符串和模式 pattern 相匹配
  * 使用模式分割字符串
* 利用运算符 ```=~``` 来匹配模式和字符串。若匹配成功则返回匹配部分的位置，否则返回 nil。
* 使用 ARGV 这个数组来获取从命令行传递过来的数据。从这里得到的数据都是字符串。
* 一下子读取全部数据很消耗内容，改成 ```File.each_line``` 就很好了。
* ```Regexp.new(str)``` 表示把字符串转换成正则表达式对象。

```ruby
pattern = Regexp.new(ARGV[0])
filenmae = ARGV[1]

file = File.open(filename)
file.each_line do |line|
  if pattern =~ line
    puts line
  end
end
file.close
```

* 引用 pp 库之后的 ```pp``` 方法可以打印出更好的对象。
* 库与脚本放在同一文件夹时，需要用 ```./``` 来表示文件存放在当前目录。

## Ruby 的基础

### 并行赋值

```ruby
a = 1; b = 2; c = 3
a, b, c = 1, 2, 3
```

* 左右两边数量不相等，也不报错。左边数量较多时，会自动将 nil 赋值给未分配值的变量。

```ruby
a, b, c, d = 1, 2
p [a, b, c] #=> [1, 2, nil]
```

* 变量部分较少时，会自动忽略该值，不会分配多余的值。

```ruby
a, b, c = 1, 2, 3, 4
p [a, b, c] #=> [1, 2, 3]
```

* 变量前加上 * 表示会将未分配的值封装为数组赋值给该变量。

```ruby
a, b, *c = 1, 2, 3, 4, 5
p [a, b, c] #=> [1, 2, [3, 4, 5]]
a, *b, c = 1, 2, 3, 4, 5
p [a, b, c] #=> [1, [2, 3, 4], 5]
```

* 置换变量的值

```ruby
a, b = b, a
```

* 用数组赋值，左边有多个变量时，会自动获取数组的元素进行多重赋值。

```ruby
ary = [1, 2]
a, b = ary
p a #=> 1
p b #=> 2

a, = ary
p a #=> 1
```

* 获取嵌套数组的元素

```ruby
ary = [1, [2, 3], 4]
a, b, c = ary
p a #=> 1
p b #=> [2, 3]
p c #=> 4

a, (b1, b2), c = ary
p a #=> 1
p b1 #=> 2
p b2 #=> 3
p c #=> 4
```
