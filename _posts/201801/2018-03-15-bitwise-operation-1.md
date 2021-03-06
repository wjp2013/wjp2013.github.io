---
layout: post
title:  "位运算"
date:   2018-03-15 15:05:00

categories: ruby
tags: tip
author: "Victor"
---

## 基本知识

* 1 个字符占用 1 个字节(byte)，一个字节 8 位(bits)。
* 基础拉丁字母，即指常见的 ABCD 等 26 个英文字母，这些字母与英语中一些常见的符号（如数字，标点符号）称为基础拉丁字符
* ASCII 用 1 个字节表示一个字符，最高位的取值永远为 0，表示字符含义的位数为 7，可表达字符数为 2^7 = 128 个。

关于 bit(位) 和 byte(字节) 的解释可以读一下 [Working with Bits and Bytes in Ruby](/ruby/Bits-and-Bytes-in-Ruby/)

## 字符编码

### 进化历史

1. 计算机只能处理数字，如果要处理文本等，那么必须要把文本转化为二进制的数字才可以处理。
2. 最开始只有 ASCII 编码，它只支持基础拉丁字符。1 个字符占用 1 个字节。
3. 因为需要兼容它语言，中间过渡了一下 EASCII -> ISO 8859 -> GB2312 之类的编码，一个中文占用 2 个字节。
4. 后来开始使用 Unicode 编码，它包含所有国家语言，所以不会出乱码。ASCII 占用 1 个字节(byte)，Unicode 通常 2 个字，因为一些生僻字可能需要 3-4 个字节。

### UTF-8

在 Unicode 编码下采用 4 个字节表示一个字符，使用中文的时候占用 2 个字节。但使用字母的时候，其实只是用了一个字节，就是后面的一个字节。比如 `00000000 01000000`，大家会注意这里就造成了浪费，就是在原来的字节前面加 0 就可以了，这样就相当于多了一倍的储存空间，在存储和运输上不太划算。

UTF-8 编码是基于 Unicode 编码的可变长度编码方式。它把 Unicode 编码根据不同的数字大小编码成 1-6 个字节，常用的字母就是 1 个字节，汉字等通常 2-3 个字节。

```ruby
'hello world'.each_byte.to_a #=> [104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100]
"s".bytes #=> [115]
"中".bytes #=> [228, 184, 173]
```

当你在看 `bytes` 文档的时候，可能还会发现一些冷门知识，比如：代码点 codepoints，有兴趣自己去找资料学习吧。

```ruby
'hello world'.codepoints.to_a #=> [104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100]
"中".codepoints.to_a #=> [20013]
```

### 计算机通用的字符编码工作模式

刚才我们知道在计算机内存中，用的是 Unicode 编码，当要存储在硬盘或者传输时，就转换为 UTF-8 模式，当我们在编辑一个文本文件时，还没保存的时候，我们用的就是 Unicode 编码，当我们点击保存时，这时候就转换为 UTF-8 编码了，当我们读取的时候，就又变成了 Unicode 编码，就是这样转换的。

Ruby 默认是 UTF-8。

```ruby
"some string".encoding #=> #<Encoding:UTF-8>
Encoding.default_external #=> #<Encoding:UTF-8>
```

### 一些 Tips

#### UTF-8 和 GB2312 转换

中文使用者需要注意的一点是 Unicode(UTF-8) 与 GBK、GB2312 这些汉字编码规则是完全不兼容的，也就是说这两者之间不能通过任何算法来进行转换，如需转换，一般通过 GBK 查表的方式来进行。

#### UTF-8 的 BOM

BOM 的全称是 Byte Order Mark，BOM 是微软给 UTF-8 编码加上的，用于标识文件使用的是 UTF-8 编码，即在 UTF-8 编码的文件起始位置，加入三个字节 `EE BB BF`。这是微软特有的，标准并不推荐包含 BOM 的方式。采用加 BOM 的 UTF-8 编码文件，对于一些只支持标准 UTF-8 编码的环境，可能导致问题。比如，在Go语言编程中，对于包含 BOM 的代码文件，会导致编译出错。

## 什么是位运算

在计算机中所有数据都是以二进制的形式储存的，位运算其实就是直接对在内存中的二进制数据进行操作，因此处理数据的速度非常快。

位运算符作用于位，并逐位执行操作。

### Ruby 位运算符

假设如果 a = 60，且 b = 13，现在以二进制格式，它们如下所示：

```ruby
a = 0011 1100
b = 0000 1101

a&b #=> 0000 1100
a|b #=> 0011 1101
a^b #=> 0011 0001
~a  #=> 1100 0011
```

下表列出了 Ruby 支持的位运算符：

| 运算符	| 描述 | 运算规则 |
|---|---|---|
| `&`	| 与 | 两个位都为1时，结果才为1 |
| `|`	| 或 |	两个位都为0时，结果才为0 |
| `^`	| 异或 | 两个位相同为0，相异为1 |
| `~`	| 取反 | 0变1，1变0 |
| `<<` | 左移 | 各二进位全部左移若干位，高位丢弃，低位补0 |
| `>>` | 右移 |	各二进位全部右移若干位 |

| 运算符	| 实例 |
|---|---|
| `&`	| `(a & b)` 将得到 12，即为 0000 1100
| `|`	| `(a | b)` 将得到 61，即为 0011 1101
| `^`	| `(a ^ b)` 将得到 49，即为 0011 0001
| `~`	| `(~a )` 将得到 -61，即为 1100 0011，一个有符号二进制数的补码形式。
| `<<` | `a << 2` 将得到 240，即为 1111 0000
| `>>` | `a >> 2` 将得到 15，即为 0000 1111

注意以下几点：

* 只有 `~` 取反是单目操作符，其它 5 种都是双目操作符。
* **位操作只能用于整形数据，对float和double类型进行位操作会被编译器报错。**
* 位操作符的运算优先级比较低，因为尽量使用括号来确保运算顺序。
* `>>` 操作。对无符号数，高位补0；有符号数，各编译器处理方法不一样，有的补符号位（算术右移），有的补0（逻辑右移），见后面的代码示例。

```ruby
a, b = 15, -15
a >> 2 #=> 3
b >> 2 #=> -4
```

因为 15 的二进制是 `0000 1111`，右移二位成为 `__00 1111`，最高位由符号位填充将得到 `0000 0011` 即十进制的 3 。-15 的二进制是 `1111 0001`，右移二位成为 `__11 1100`，最高位由符号位填充将得到 `1111 1100` 即十进制的 -4。

## pack 和 unpack

上面的例子这里涉及到两个知识点：

1. 怎么把 15 变成 2进制，另外字母怎么转换成 2进制，中文怎么转换成 2进制？
2. 符号位是怎么回事？

### 不同编码之间的处理

**第一个问题很简单，以字符串为例，先转换成 ascii 码再转换成 2进制。**

```ruby
'A'.bytes.first.to_s(2) #=> "1000001"
```

闹呢，当然不可能这么傻。C 语言允许开发人员直接访问存储变量的内存，而 Ruby 不行。当我们需要在 Ruby 中访问 字节(byte) 和 位(bits) 的时候，可以使用 `pack` 和 `unpack` 方法。

一般来说，对于 `unpack` 方法你只要记住两个参数 `b*` 转换成 2进制，和 `C*` 转换成 ascii 码。

```ruby
'A'.unpack('b*') #=> ["10000010"]
"hello world".unpack('C*') #=> [104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100]
"中".unpack('C*') #=> [228, 184, 173]
```

真的足够用了，再去研究 `B*` 和 `b*` 有什么不同，又会牵扯到 MSB/LSB 的问题，'H' 转换成 16进制什么的，完全不用在意。

懂了 `unpack` 那 `pack` 也就懂了，无非是逆向操作。

```ruby
[1000001].pack('C') #=> "A"
[104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100].pack('C*') #=> "hello world"
```

### 符号位

首先要弄懂 原码，反码和补码，而我不打算摘抄一大段东西，所以可以直接看

* [原码，反码和补码的关系？](https://www.zhihu.com/question/23172611/answer/27248266)
* [原码，反码和补码](https://zhuanlan.zhihu.com/p/22718975)

关于第 2 个问题你目前只需要了解到：

* 符号位是第一个字节 8 位(bits)的第 1 位，1为负，0为正。
* ruby 的 `>>` 操作是算术右移。低位溢出，符号位不变，**并用符号位数补溢出的高位。**

## 实际应用

缘由出来了，今天遇到了这个函数。问这个函数有什么用。

```ruby
# File activesupport/lib/active_support/security_utils.rb, line 11
def secure_compare(a, b)
  return false unless a.bytesize == b.bytesize
  l = a.unpack "C#{a.bytesize}"
  res = 0
  b.each_byte { |byte| res |= byte ^ l.shift }
  res == 0
end
```

看完了上面的文章，这函数应该能看懂了吧。当然，除了 `a.unpack "C#{a.bytesize}"` 这句有点迷糊，刚才明明说好只要记住 `C*` 和 `b*` 两个参数足够了啊。

没办法去看文档吧，然后，我懵了！

```ruby
"aaa".unpack('h2H2c') #=> ["16", "61", 97]
"whole".unpack('xax2aX2aX1aX2a') #=> ["h", "e", "l", "l", "o"]
```

这什么鬼参数，好歹找到了一篇中文解释 [pack模板字符串](http://www.kuqin.com/rubycndocument/man/pack_template_string.html)。

翻译过来，这一句就跟 `a.unpack 'C*'` 没区别。

其它没什么好解释的了，按位 异或 比较传入的两个参数是否相等。

我想了半天也没想到这么比较有什么好处，又不想靠 gg 搜答案，晚上跟老伙伴讨论了一波，他也没什么头绪，没办法只好 gg 了。

论坛上有相关帖子，这函数主要是搞定 *计时攻击* 的，具体可看 [计时攻击原理以及 Rails 对应的防范](https://ruby-china.org/topics/21380)。

## 后记

延展开去还有关于 IO 打开文件时候的外部编码和内部编码的问题，可以直接看相关阅读中 *Ruby 对多语言的支持* 一文，虽然文章有点老，还是针对 Ruby 1.9 版本阐述的问题，但是不影响你理解。

再扯其它的就离主题太远了，初衷只是因为今天做笔试题时候发现自己对这块知识基本忘光了，赶快奶自己一波。

**如果你知道自己在某一领域上有所欠缺，就应该立刻开始学习相关知识。**

## 关于字符集和编码的补充

* 字符(Character)是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等。
* 字符集(Character set)是多个字符的集合，字符集种类较多，每个字符集包含的字符个数不同，常见字符集名称：ASCII字符集、GB2312字符集、BIG5字符集、 GB18030字符集、Unicode字符集等。
* 字符编码，是编码字符集和实际存储数值之间的转换关系。

「字符集」和「编码」等几个层次的概念被彻底分离且模块化的这样一个模型，其实是 Unicode 时代才得到广泛认同的。而对于 ASCII、GB2312、Big5 之类的遗留（legacy）方案，其字符集及其编码的关系基本是锁定的，所以常常用「字符编码」（character encoding）、「代码页」（code page）等概念来统称它们那样从字符到编码字节流的整体方案。比如 ASCII 本身就既是字符集又是编码方案，而 GB2312 用的都是 EUC-CN 编码方案。

### 常见的编码字符集（简称字符集）

* Unicode：也叫统一字符集，它包含了几乎世界上所有的已经发现且需要使用的字符（如中文、日文、英文、德文等）。
* ASCII：早期的计算机系统只能处理英文，所以ASCII也就成为了计算机的缺省字符集，包含了英文所需要的所有字符。
* GB2312：中文字符集，包含ASCII字符集。ASCII部分用单字节表示，剩余部分用双字节表示。
* GBK：GB2312的扩展，但完整包含了GB2312的所有内容。
* GB18030：GBK字符集的超集，常叫大汉字字符集，也叫CJK（Chinese，Japanese，Korea）字符集，包含了中、日、韩三国语言中的所有字符。

### 实例案例

#### ASCII

* ASCII 既是编码字符集，又是字符编码
* ASCII 直接将字符在编码字符集中的序号作为字符在计算机中存储从数值。

例如：在ASCII 中A在表中排第65位，序号是65，而编码后A的数值是 `0100 0001`，也即十进制
的 65 的二进制转换结果。

#### Unicode

* 编码字符集：Unicode
* 字符编码：UTF-8

UTF-8 的物理存储和 Unicode 序号的转换关系，可以看[字符编码与字符集的区别](http://blog.csdn.net/qq_29028175/article/details/52959551)。

## 相关阅读

* [位运算简介及实用技巧（一）：基础篇](http://www.matrix67.com/blog/archives/263)
* [Ruby 运算符](http://www.runoob.com/ruby/ruby-operator.html)
* [Issue-3 字符串和编码，了解bytes str unicode的区别](http://blog.csdn.net/jinguasu/article/details/74939678)
* [Ruby：字符集和编码学习总结](https://www.cnblogs.com/happyframework/p/3275367.html)
* [位操作](https://zh.wikipedia.org/wiki/%E4%BD%8D%E6%93%8D%E4%BD%9C)
* [位运算简介及基本技巧](https://blog.yangx.site/2016/07/06/bit-operation-skills/)
* [Ruby pack unpack](http://blog.bigbinary.com/2011/07/20/ruby-pack-unpack.html)
* [Ruby 对多语言的支持](https://www.cnblogs.com/rockchip/archive/2013/07/16/3192501.html)
* [What the Pack](https://idiosyncratic-ruby.com/4-what-the-pack.html)
