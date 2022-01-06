---
title: "Day 6 RSpec 超基礎語法！"
date: 2021-09-21T17:23:05+08:00
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

正式進入基本語法！
<!--more-->

{{< toc >}}
昨天的文章介紹了 TDD 的流程和精神等等。

今天我們要正式進入 RSpec 的世界，所以我們從最一開始容易見到的幾種語法開始做解釋和教學！

# describe method 

假設我們今天要開一間漢堡店，那我們就要來想像漢堡店的測試！

首先我們在 Spec 檔案裡面加入一個 `burger_spec.rb` 的檔案！

這樣我們就可以開始來寫啦～

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
 Rspec.describe 'Burger' do
   ...
 end
<!-- endtab -->
{{</ tabbed-codeblock>}}

這個 `describe` 呢，主要在解釋我們要測試的主要目標是什麼？( 這邊是測試漢堡這個類別 )，而這邊也有一個 Ruby 小小的慣例，就是可以省略 ( ) 小括號，所以原本應該是這樣...

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
RSpec.describe('Burger') do
  ...
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

所以從這邊我們可以了解到，`describe` 應該是 RSpec 的類別方法唷！

接著 `describe` 也同時要接收一個 Ruby 的 block 用 `do..end` 的方式接在後面。

寫到這邊，我們已經完成了一個 `example group`，直翻的話叫做：例子的集合？？？

也就是我們提供了一個 `example` 的地基，在這裡面可以放入許多的小 `example`，在 RSpec 的世界中，每一個我們所期待的事件都是一個 `example`，而 `describe` 在做的事情就組織著這些小東西！

# it method

上面的例子中，我們建立了一個 `example group` 而在這個介紹的語法裡，我們要建立單一個測試，也就是一個小的 `example`。

回到上面的例子，我們繼續寫下去，我們想了一下，希望漢堡要有種類，就用坊間常見的漢堡來做例子就可以了！

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
RSpec.describe 'Burger' do
  it 'has a type' do
    ...
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

可以看到我們寫了一個 `it` 這也是一種方法，只能存活在 `example group` 裡面，而後面會接一段字串來描述這個測試的目的是什麼？然後再接一個 block。

這個 `it` 方法最大的價值就在於，他能夠讓一個測試變得易讀，然後清楚的描述這個行為是什麼。

而不是在描述的地方寫到你是如何做到的，不需要寫我是用 Array 來存放，或用 Hash 做 map，諸如此類的東西，不需要被特別的提出來！只需要專注在這個測試的行為是什麼？什麼樣的行為？

像我們剛剛寫的就很明確，一個漢堡他應該要有種類，非常的易讀！

這主要是 `it` 這個方法的目的，描述一個測試的行為，不是結果喔！結果會有另一個方法來執行，但這邊的目的在於 **描述物件的行為**

# expect & eq methods

接著繼續來開發我們的漢堡店，還記得開發的需求是說 漢堡需要有不同的種類，就著我們就來想像一下，我們需要做出漢堡的實體，然後給予他種類，並通過測試。

上述也算是一個 TDD 的迴圈喔！還記得嗎？現在我們在根據需求並使其通過測試！

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
RSpec.describe 'Burger' do
  it 'has a typy' do
    burger = Burger.new('Big mac')
    ...
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們在這邊建立了一個 `burger` 的實體，便給予他種類的名稱`Big mac`，接著我們需要測試它，所以我們會用到一個叫做 `expect` 的方法。

`expect` 這個方法接收的參數是我們要測試的對象，也就是我們剛剛做出來的 `burger`。

那要記得我們這次測試的目的是，希望他有一個種類，所以我們在 `expect` 的後方寫的是 `burger.type` 並希望得到我們要的答案！

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
RSpec.describe 'Burger' do
  it 'has a typy' do
    burger = Burger.new('Big mac')
    expect(burger.type).... 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著我們繼續的寫下去：

{{< tabbed-codeblock "測試碼" >}}
<!-- tab ruby -->
RSpec.describe 'Burger' do
  it 'has a typy' do
    burger = Burger.new('Big mac')
    expect(burger.type).to eq('Big mac')
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們專注在第四行上面，如果用我破爛的英文來翻譯的話，會是
> 期待漢堡的種類等於 `Big mac`

這樣應該是還蠻容易閱讀的，雖然不是文法很精確的英文，但我覺得看得很舒服！

我們重新的整理一下我們寫了一些什麼，以及他們有什麼用途。

首先是 `expect`，他是一個 RSpec 提供的方法，可以接受物件、物件的屬性、方法、甚至是基本的 Ruby 運算式，而這個 `expect` 方法會針對你傳入的參數進行運算，並且回傳一個物件。

```ruby=
#<RSpec::Expectations::ExpectationTarget:0x00007fb81f240278 @target="Big mac">
```

若是把 `expect(burger.type)` 印出來就會長的像上面一樣，是一個物件的概念，以及具備 `@target` 這個實體變數 ( @target 會是我們主要接受測試的項目喔！ )

而這個物件會擁有 `to` 這個實體方法，這個 `to` 方法呢，需要接受一個叫做 `matcher` 的物件，那這個物件是怎麼產生的？

就在於後面我們寫入的 `eq('Big mac')` 這個方法也會產生一個物件

```ruby=
#<RSpec::Matchers::BuiltIn::Eq:0x00007fdb92afa3f0 @expected="Big mac">
```

具有一個 `@expected` 的實體變數！

而 `to` 這個方法呢，簡單的來說就是比對這兩個物件的實體變數，回傳成功或是失敗，當然實際上更複雜一點，有針對許多的情況來做 raise error 但基本邏輯就是這樣進行的，所以若是你沒有傳入 `matcher` 這個例子就沒有辦法完成！

OK，所以到這邊我們已經寫完了一個最基本的測試了！接著我們對著終端機輸入`rspec spec/burger_spec.rb` 會看到～

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109211728630.png)

竟然沒有通過... 到底發生什麼事情了？我們明天會針對 RSpec 的錯誤來進行解說，看懂錯誤訊息也是工程師很重要的一環喔！

# 小結

今天我們學會了整個 RSpec 的主架構，從利用 `describe` 框出一個區塊讓我們寫小 `example` 再到使用 `it` 來敘述我們測試的行為，以及利用 `expect`＆`eq` 來進行比對測試！

這樣可以算是一個小小的測試囉！運用這幾個方法，相信很簡單的測試也難不倒你了，但可惜的是我們最後遇到了問題，但沒關係！我們明天來解決它。