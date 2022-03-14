---
title: "Ruby 的 block ep.1"
date: 2022-03-14T02:15:06+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203140228710.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202203140229027.jpg'
coverMeta: out
coverSize: partial
categories:
- "Ruby"
tags:
- Syntax
summary: 聊聊 Ruby 的 block
keywords:
- Ruby
- block
- DSL
- method
---

{{< toc >}}

# Ruby 的 block 

在 Ruby 這個程式語言中，blocks 是一個很基本的概念，許多 Ruby 的方法都有使用到 blocks。

Block 也是許多 domain-specific languages (DSLs) 不可或缺的一部分，像是 RSpec, Factory Bot 或是 Rails 本身。

下面會討論一下關於什麼是 block，並且看看在 Ruby 裡四種使用到 block 的原生方法 (times, each, map & tap)，為了可以更了解什麼情況下使用哪個 block 是更適合的。

然後，我們會用 block 來定義一些客製的方法！

# 什麼是 block

實際上所有的程式語言都有他獨特的方式來拿到參數給 function ( 在 Ruby 我們稱作 Method ) 做使用，我們把資料傳進 function 裡面，然後他對這些資料做一些處理～

但 block 把這個想法提升到了一個不同的等級，block 是傳遞行為，而不再是傳遞資料給一個 function，看看下方的圖解，你會更理解這句話代表什麼意思！

# 使用 block 的原生 Ruby 方法

以下是四種原生就使用 block 的 Ruby 方法，每一個方法都會附上形容，並且加入使用的例子！

還記得上面有提到 block 是一種方式去傳遞行為而不是傳遞資料的嗎？

下面我都會用 `行為` 來形容被方法使用的動作！

# Times 方法

形容：根據我給予的次數，就重複幾次 `行為`

例子：我給予 3 次的次數，重覆印出 `'Hello'` 3 次 ( 這邊的 `行為` 就是印出 `'Hello'` )

{{< tabbed-codeblock Times 方法 >}}
<!-- tab ruby -->
3.times do
  p 'hello'
end

Output:
"hello"
"hello"
"hello"
<!-- endtab -->
{{</ tabbed-codeblock>}}

# Each 方法

形容：拿一個陣列，根據陣列中每一個的元素來執行 `行為`

例子：依序印出陣列裡面的每一個元素 ( 這邊的 `行為` 就是印出當前陣列中的元素 )

{{< tabbed-codeblock Each 方法 >}}
<!-- tab ruby -->
[1, 2, 3].each do |n|
  p n
end

Output:
1
2
3
<!-- endtab -->
{{</ tabbed-codeblock>}}

# Map 方法

形容：拿一個陣列，同時對陣列中的元素執行 `行為`，並且將 `行為` 結束的值回傳到一個新的陣列，接著回傳新的陣列

例子：讓陣列中的元素對自己做相乘 ( 這邊的 `行為` 就是元素自己相乘 )

{{< tabbed-codeblock Map 方法 >}}
<!-- tab ruby -->
squares = [1, 2, 3].map do |n|
  n * n
end

p squares.join(",")

Output:
"1,4,9"
<!-- endtab -->
{{</ tabbed-codeblock>}}


# Tap 方法

形容：對某個物件執行 `行為`，並且把執行後的物件回傳

例子：初始化一個文件，寫一些文字在裡面，之後回傳整個文件 ( 這邊的 `行為` 就是寫一些文字在裡面 )

{{< tabbed-codeblock Tap 方法 >}}
<!-- tab ruby -->
require 'tempfile'

file = Tempfile.new.tap do |f|
  f.write('hello world')
  f.rewind
end

p file.read

Output:
"hello world"
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著可以看看我們如何把 block 加入到我們自己實作的方法！

# 加入 block 的客製方法

這裡有一個方法，可以輸出一個 HTML tag 和一段 `行為`，他會執行我們給予的 `行為`，並且在 `行為` 的前後加上 HTML tag。

{{< tabbed-codeblock 客製方法 >}}
<!-- tab ruby -->
inside_tag('p') do
  p 'Hello'
  p 'How are you?'
end
<!-- endtab -->
{{</ tabbed-codeblock>}}


這個方法會產生的結果是這樣：

{{< tabbed-codeblock Output >}}
<!-- tab ruby -->
<p>
Hello
How are you?
</p>
<!-- endtab -->
{{</ tabbed-codeblock>}}

這邊的 `行為` 就是印出 "Hello" & "How are you?"

接著可以看看是如何定義這個方法的：

{{< tabbed-codeblock 定義方法 >}}
<!-- tab ruby -->
def inside_tag(tag, &block)
  p "<#{tag}>"  # 印出開始 tag
  block.call    # 呼叫我們傳入的 block
  p "</#{tag}>" # 印出結束 tag
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

在下面的例子裡，我們傳入了一個 `Tag` 類別的實體回到 `inside_tag` 的 block 裡面，並允許 block 裡面可以呼叫 `tag.content` 而不是只是 `p`。

這甚至還可以讓我們的內容做到縮排的效果。

{{< tabbed-codeblock 完整定義 >}}
<!-- tab ruby -->
class Tag
  def content(value)
    p "  #{value}"
  end
end

def inside_tag(tag, &block)
  p "<#{tag}>"
  block.call(Tag.new)
  p "</#{tag}>"
end

inside_tag('p') do |tag|
  tag.content 'Hello'
  tag.content 'How are you?'
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

上面的 Code 會有下面的結果：

{{< tabbed-codeblock Output >}}
<!-- tab ruby -->
<p>
  Hello
  How are you?
</p>
<!-- endtab -->
{{</ tabbed-codeblock>}}

傳送物件到 block 是常見的 DSL 技術被應用在 RSpec、Factory Bot 以及 Rails 本身。

# 更多關於 block 的細節

其實還有許多關於 block 的細節，也有看到一些關於 block 有趣的問題，之後也想繼續花時間去研究並且寫下文章做紀錄！

1. 呼叫 block 的不同方式，以及為什麼要用它？
2. proc、block & lambda 之間的差別？
3. 什麼是 Proc 物件以及它和 block 的關聯？
4. 什麼是封包？
5. 常常看到的 `&block` 是什麼意思？
6. `[1,2,3].map(&:to_s)` 是怎麼運作的？

# 結語

Ruby 的 block 並不只是傳遞資料的工具，而是可以拿來傳遞行為，這在 Ruby 的世界是非常流行的，可以看到很多開源的專案或是有名的工具，都不斷地在使用，若是可以在自己的專案中加入，也不乏是提升自己能力的一種方式吧！





