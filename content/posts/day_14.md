---
title: "Day 14 分享你的 Example！"
date: 2021-09-28T17:15:43+08:00
metaAlignment: center
thumbnailImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png'
coverImage: 'https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211620030.png' 
coverMeta: out
coverSize: partial
categories:
- "2021 鐵人賽"
tags:
- RSpec
---

分享共用的 Example！
<!--more-->
{{< toc >}}
# 前言

昨天結尾有稍微提到，若是有相同的 example 該怎麼辦？

RSpec 在節省程式碼這塊可以說是不遺餘力，甚至連常常使用到 example 都可以收納在一個檔案中再引入來使用！

# Shared_examples

我們先來舉一個極端一點的例子，假設你在不同的測試檔案中，都剛好要測試 `subject` 的長度好了！

{{< tabbed-codeblock "舉例">}}
<!-- tab ruby -->
RSpec.describe Hash do
  subject {{a: 1, b: 2, c: 3}}
end

RSpec.describe Array do
  subject {[1, 2, 3]}
end

RSpec.describe String do
  subject {"abc"}
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以想一下上面若是三個不同的測試檔案，想必我們就會正常的寫：

{{< tabbed-codeblock "舉例">}}
<!-- tab ruby -->
RSpec.describe Hash do
  subject {{a: 1, b: 2, c: 3}}
  it "return length of subject" do
    expect(subject.length).to be(3)
  end
end

RSpec.describe Array do
  subject {[1, 2, 3]}
  .....
end

RSpec.describe String do
  subject {"abc"}
  .....
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

就一樣的東西寫三次，感覺也沒什麼，但若是量提高了，這時候我們能做些什麼？

就可以使用 RSpec 提供的 `shared_examples` 來抽離這些重複的 examples。

使用範例應該就能快速理解！

{{< tabbed-codeblock "抽離重複性質的 Code">}}
<!-- tab ruby -->
RSpec.shared_examples "Object with three elements" do
  it "return length of subject" do
    expect(subject.length).to eq(3)
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這樣我們就建立好了一個可以被分享的 example，至於要如何使用呢？

我們只要引入這個 example 最上層的敘述就可以了！所以改寫一下我們原本的範例：

{{< tabbed-codeblock "引入使用">}}
<!-- tab ruby -->
RSpec.describe Hash do
  subject {{a: 1, b: 2, c: 3}}
  include_exmaples "Object with three elements"
end

RSpec.describe Array do
  subject {[1, 2, 3]}
  include_exmaples "Object with three elements"
end

RSpec.describe String do
  subject {"abc"}
  include_exmaples "Object with three elements"
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

以上的測試會全數通過喔！ `include_exmaples` 這個方法可以把我們放在別的地方的 exmaple 引入做使用，效果就像你直接寫在裡面一樣！

# Shared_context

接下來這個也是一樣可以搜集程式碼的好方法，但我覺得更好用的一點。

畢竟 example 這種東西比較客製化，但如果是 `shared_context` 就可以放比較多設定的部分，物件等等...

直接用程式碼應該會更好懂一點：

{{< tabbed-codeblock "shared_context">}}
<!-- tab ruby -->
RSpec.shared_context "common_burger" do
  before do
    @big_mac = Burger.new("Beef", "Cheddar")
  end
  
  def shrimp_burger
    @shrimp_burger = Burger.new("Shrimp", "Mozarella")
  end
  
  let(:pork_burger) { Burger.new("Pork", "Pecorino")}
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們製作了一個拿來分享的 `context` 裡面有著一堆不同種類的漢堡，他可能需要在不同的測試檔做使用！

這時候如果我們發現我們在設定物件的時候不停的重複，我們就可以把重複的物件蒐集到 `shared_context` 裡面，然後在 `include` 進來也一樣！

就像範例中一樣：

{{< tabbed-codeblock "實際使用">}}
<!-- tab ruby -->
RSpec.describe Burger do
  include_context "common_burger"
  
  it "return shrimp" do
   expect(shrimp_burger.meat).to eq("Shrimp") 
  end
  ....
end

RSpec.describe Store do
  include_context "common_burger"

  it "have big mac" do
    expect(@big_mac.meat).to eq("Beef")
  end
  ....
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們不用在上方堆疊一大堆的 `let` 或是 `before hooks` 這類的東西，我們可以在很多的測試檔中 `include` 進來使用！

有沒有超方便的，但如果你要放在不同的檔案裡面，記得要 `require` 來做使用喔！

不然 Ruby 本身會不知道你在 `include` 什麼樣的東西！

```ruby=
require "../你的檔名" # 詳細還是要看檔案放置位置，來決定前面的 .. 數量或是不需要 ..
```

# 小結

今天介紹了兩個整理程式碼的方法，`shared_examples` 實際使用的時候比較少一點；但 `shared_context` 我自己覺得蠻好用的。

因為專案越大，有交集的邏輯會越來越多，所以有些東西都可以抽出來，使得當頁面的負擔較小，看起來更舒服易讀！




