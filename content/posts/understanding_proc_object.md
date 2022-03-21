---
title: "了解 Proc 物件"
date: 2022-03-21T23:05:32+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203140228710.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203212257621.jpg' 
coverMeta: out
coverSize: partial
categories:
- Ruby
tags:
- Syntax
summary: '討論 Proc 物件'
keywords:
- Ruby
- block
- DSL
- method
- Proc
---

{{< toc >}}

如果你是一個 Ruby 的工程師，你幾乎每天都在使用 Proc 物件，雖然經常在沒有察覺的情況下呼叫它。

Blocks，是一個在 Ruby 裡面無所不在的東西；lambdas，被使用在 Rails 的 scopes，包括 Proc 物件也是！

在這篇文章，將會更細微地研究 Proc 物件，我們先從使用 Proc 物件呼叫 `hello world` 開始看看怎麼使用吧。

後面我們會從 Ruby 的原始文件裡面看看 Proc 物件的解釋，最後我們會知道 Proc 物件和 blocks 以及 lambdas 之間的關聯。

# Proc 物件的基本使用

在我們討論 Proc 物件之前，讓我們操作一下 Proc 物件，看看他是怎麼樣子的呈現方式。

Ruby 官方手冊提供關於 Proc 物件的 `hello world` 範例：

{{< tabbed-codeblock "proc example" >}}
<!-- tab ruby -->
square = Proc.new { |x| x**2 }
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們可以打開 `irb` 並且嘗試看看結果：

{{< tabbed-codeblock "proc irb console" >}}
<!-- tab ruby -->
> square = Proc.new { |x| x**2 }
 => #<Proc:0x00000001333a8660 (irb):1> 
> square.call(3)
 => 9 
> square.call(4)
 => 16 
> square.call(5)
 => 25
<!-- endtab -->
{{</ tabbed-codeblock>}}

看完之後會有一種大概理解 Proc 物件的使用方式，有點像是方法一樣，定義一個行為，並且可以在想要使用的地方重複使用。

所以我們現在對於 Proc 物件有了比較鬆散的理解，讓我們繼續深入吧！

# 深入理解 Proc 物件

官方文件對於 Proc 物件的定義：

```
Proc 物件
- 是一個封裝的 block 程式碼
- 可以存放在區域變數
- 傳進方法或是另一個 proc
- 並且可以被 call 方法呼叫
```

上面這組解釋聽起來有點饒口，當我遇到這樣的比較複雜的定義時，我喜歡把問題切割成更小的問題，可以幫助我更快理解，所以根據上面的條列，我可以一步一步地研究。

# Proc 物件是一個封裝的 block 程式碼

對一段程式碼的封裝意味著什麼呢？封裝某樣東西，我會把他比喻成放在一個膠囊裡面。膠囊裡面的東西和外界是完全隔離的，也可以說被封裝的東西是被 `打包` 了！

所以當文件說 Proc 物件是 `一個封裝的 block 程式碼` 時，通常意味著這段程式碼被打包並且和外界徹底的隔離了。

# Proc 物件可以被存放在區域變數

像上面的範例中也有使用到：

{{< tabbed-codeblock "proc example" >}}
<!-- tab ruby -->
square = Proc.new { |x| x**2 }
<!-- endtab -->
{{</ tabbed-codeblock>}}

這段程式碼先建立了一個 Proc 物件，並且把他儲存在 `square` 的區域變數裡面，所以這部分算是比較好理解的。

# Proc 物件可以被傳進方法中或是另一個 Proc 物件

這邊的定義其實可以拆成兩個來討論，我們先來試試看傳入方法吧

我們先定義了一個可以接受 Proc 物件的方法 ( 我們有在裡面用 `call` 方法 )

接著定義了兩個 Proc 物件，並且分別傳入 `pass_to_method` 這個方法！

{{< tabbed-codeblock "pass to method" >}}
<!-- tab ruby -->
def pass_to_method(number, operation)
  operation.call(number)
end

square = Proc.new { |x| x**2 }
double = Proc.new { |x| x * 2 }

puts pass_to_method(5, square)
puts pass_to_method(5, double)
<!-- endtab -->
{{</ tabbed-codeblock>}}

如果我們執行上面的程式碼，會得到下面的結果：

{{< tabbed-codeblock "output" >}}
<!-- tab ruby -->
25
10
<!-- endtab -->
{{</ tabbed-codeblock>}}

所以上面整段就是把 Proc 物件傳入方法的範例，比較像是之前講 blocks 時提到的，這是一種傳遞行為的方式，而不是一般的傳遞數據！

換句話說，你也可以傳遞一個封裝的程式碼，然後在需要的時候 `call` 它來使用。

接著，如果我們想把 Proc 物件傳入另一個 Proc 物件時，其實和上面的寫法很像：

{{< tabbed-codeblock "pass to proc" >}}
<!-- tab ruby -->
pass_to_proc = Proc.new do |number, operation|
  operation.call(number)
end

square = Proc.new { |x| x**2 }
double = Proc.new { |x| x * 2 }

puts pass_to_proc.call(5, square)
puts pass_to_proc.call(5, double)
<!-- endtab -->
{{</ tabbed-codeblock>}}

和上面的唯一區別就在於 `被定義的結果` 一個是 Proc 物件，一個是方法，但就結果來說是完全一樣的！

# Proc 物件可以被呼叫

Proc 物件的最後一個定義，可以被 `call`，這也是一個顯而易見的例子，但我們還是做點範例吧

{{< tabbed-codeblock "proc can be called" >}}
<!-- tab ruby -->
square = Proc.new { |x| x**2 }
square.call(5)
<!-- endtab -->
{{</ tabbed-codeblock>}}

其實還有其他可以呼叫 Proc 物件的方式，但對於理解 Proc 並沒有太大的幫助，所以先不討論。

# 閉包 Closures

要能夠理解 Proc 物件，閉包的概念也是需要釐清的，閉包是一個廣泛的概念，並不是只有 Ruby 才有。

但因為閉包的解釋實在太過複雜，之後再另外寫一篇來研究一下！目前也還不知道怎麼好好地講

但長話短說的版本就是，閉包是一個儲存了函式以及變數的 `紀錄` 

# Proc 物件以及 blocks

所有的 block 在 Ruby 裡面都是 Proc 物件，下面有一個客製的方法接收一個 block 做為參數：

{{< tabbed-codeblock "custom method accept block been argument" >}}
<!-- tab ruby -->
def custom_method(&block)
  puts block.class
end

custom_method { "hello" }
<!-- endtab -->
{{</ tabbed-codeblock>}}

執行後的 Output：

{{< tabbed-codeblock "output" >}}
<!-- tab ruby -->
Proc
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以看到我們傳入的 block 變成了 Proc 物件！

接著這段程式碼的結果和上面的一模一樣，但傳入的方式略有不同：

`my_proc` 前面的 `&` 把它轉成了一個 block，這邊的 `&` 符號也是一個有趣的題目，找時間也來寫一下！

{{< tabbed-codeblock "custom method accept proc been argument" >}}
<!-- tab ruby -->
def custom_method(&block)
  puts block.class
end

my_proc = Proc.new { "hello" }
custom_method &my_proc
<!-- endtab -->
{{</ tabbed-codeblock>}}

# Proc 物件和 lambda

lambdas 也是一個 Proc 物件，簡單的方式就能夠證明一下：

{{< tabbed-codeblock "lambda" >}}
<!-- tab ruby -->
> my_lambda = lambda { |x| x**2 }
 => #<Proc:0x00000001241e82a8 (irb):1 (lambda)> 
> my_lambda.class
 => Proc
<!-- endtab -->
{{</ tabbed-codeblock>}}

而這兩者最大的差別在於，在 `lambda` 中加入 `return` 意味著退出這個 `lambda` 的執行區塊；但在 Proc 物件中 `return` 意味著結束該方法的執行程序 ( 通常我們都是在方法內呼叫 Proc 物件，但在 Proc 內 `return` 會連外層的方法一起結束 )

其實還有一些小細節，但可以另外寫文章討論，今天主要放在 Proc 物件身上！

# 總結

1. Proc 物件是一個被封裝的 block 程式碼，可以被儲存在區域變數中，可以被傳入方法或是 Proc，可以被 call 方法呼叫
2. 閉包是一個儲存了函式以及變數的 `紀錄` 
3. lambda 也是 Proc 物件，但在行為上還是有些微的不同










