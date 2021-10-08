---
title: DSL 思考
date: "2018-07-17 23:55:28"
categories: "编程"
tags: ["DSL"]
desc: DSL，即Domain Specific Language，是为在某些特定领域内解决特定问题而设计的专用语言，其基本思想是“求专不求全”。
toc: true
---



### 一、什么是DSL

DSL，即Domain Specific Language，是为在某些特定领域内解决特定问题而设计的专用语言，其基本思想是“求专不求全”。DSL主要目的是消除代码复杂度和间接性，并且应该注重专业领域。此外，也需要合理恰当的语法形式来实现DSL。

<!-- more -->

DSL 的类型：

- 非计算

  HTML、CSS  

- 计算能力有限

  SQL、 CPP templates 、正则表达式

- 异构

  GLSL

- 特殊应用

  Matlab  

### 二、 Embedded DSL

Embedded domain-specific language（eDSL）是嵌入式DSL，其优势是利用宿主语言实现。嵌入式DSL可以看做是库，API *can usefully be thought of as a language*。

**Embedding DSL** 利用宿主语言的现有api实现，不再需要编写解析器。

实现 eDSL 的方式有如下几种：

- 字符串

  是最简单的实现方式，因为不需要编写解析器。最好的例子是在JQuery中检索CSS

  ```javascript
  const myTable = $("#foo div.tabular-data table);
  ```

  CSS选择器是只是指定一组HTML document 元素的DSL。

  同样，还有正则表达式。

  ```javascript
  const matches = input.match(".*/.+\\.png$");
  ```

- 宏定义、准引用(Quasiquotation)

  宏定义（Macros）能够在编译时期执行代码。宏常常与准引用一起使用，比如在Lisp和 Haskell 中可以自定义语法。

  在Yesod 网络框架中，Haskell 有 [Shakespearean Templates](http://www.yesodweb.com/book/shakespearean-templates)  的例子，允许HTML/CSS/JS代码在Haskell 代码中插值。

  ```haskell
  data Person = Person
      { name :: String
      , age  :: Int
      }

  main :: IO ()
  main = putStrLn $ renderHtml [shamlet|
  <p>Hello, my name is #{name person} and I am #{show $ age person}.
  <p>
      Let's do some funny stuff with my name: #
      <b>#{sort $ map toLower (name person)}
  <p>Oh, and in 5 years I'll be #{show ((+) 5 (age person))} years old.
  |]
    where
      person = Person "Michael" 26
  ```

- 组合Combinators

  Combinators 是利用小函数或者对象进行构建，因为没有自定义语法，所以很像 API。

  例如，Ruby的Rake构建系统对`.md`文件运行`pandoc`生成`.html`文件。

  ```ruby
  task :default => :html
  task :html => %W[ch1.html ch2.html ch3.html]

  rule ".html" => ".md" do |t|
    sh "pandoc -o #{http://t.name} #{t.source}"
  end
  ```

- Monads

  在Haskell 中可以利用 Monad 来实现 eDSL。

  ```haskell
  result = do a <- [1..10]
              b <- [1..10]
              guard (a /= b)
              guard (a + b == 7)
              return (a, b)
  ```

eDSL 因为能够用于处理专用领域中的问题，所以用处极大。但是在DSL设计和使用中，应该注重在实现上使用恰当的语法。同时，对于DSL解决的问题可能是“动态逻辑加载”，可以使用现有语言动态调用解析器来完成。

库（library）和 eDSL 很相似，有时候最简单的库就能够解决问题。

> **Embedded DSLs are useful because they let us apply everything we know about programming languages to specific domains.** 

### 参考

1. [What is an embedded domain-specific language?](https://www.quora.com/What-is-an-embedded-domain-specific-language)

2. [王垠——DSL](http://www.yinwang.org/blog-cn/2017/05/25/dsl)

3. [Embedded Domain Specific Language](http://c2.com/cgi/fullSearch?search=EmbeddedDomainSpecificLanguage) 

4. [领域专用语言迷思](http://www.infoq.com/cn/articles/dsl-discussion)

5. [DSL在实际工作中的应用](http://jyiigpgf.github.io/dsl/2015/02/09/DSL%E5%9C%A8%E5%AE%9E%E9%99%85%E5%B7%A5%E4%BD%9C%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8.html)
