---
title: "Day 12 Hooks 們以及作用域的差別！"
date: 2021-09-26T14:08:26+08:00
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

來釐清作用域吧！
<!--more-->
{{< toc >}}

# After Hooks

之前有介紹過 `before hooks` 的使用方法，今天也可以介紹一下 `after hooks` 的使用方法，我們先用範例來看看！

{{< tabbed-codeblock "After hooks" >}}
<!-- tab ruby -->
RSpec.describe "before and after hooks" do
  before(:example) do
    p "before hooks"
  end
  
  after(:example) do
    p "after hooks" 
  end
  
  it "test for before & after" do
    p "inner example"
    expect(1 + 1).to eq(2) 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

下面這是 Output 的結果，可以看到執行順序的不同！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109261410586.png)

那一定會很好奇，什麼時候要使用 `after hooks`，有時候我們為了效能考慮，某些工程師會不想要每次都產生一個新的物件。

所以會在 `after hooks` 中把物件 Reset 資料或是清空資料庫等等的動作，都會在這時候使用！

可以用一個簡單的範例來看看～

{{< tabbed-codeblock "After hooks Example" >}}
<!-- tab ruby -->
RSpec.describe "before and after hooks" do
  before(:example) do
  end
  
  after(:example) do
    p @array
    p @array.clear
  end
  
  it "test for before & after" do
    @array = []
    p "Inner #{@array}"
    @array << '1'
    expect(1 + 1).to eq(2) 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

我們在測試中產生一個 `@array` 然後在 `after hooks` 中清空它，下面是 Out put 的樣子！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109261411921.png)

可以看到 `@array` 在結束後被清除了，這對於某些情況是有幫助的，但使用的程度並沒有到極高的情形，還是可以把它記起來，說不定哪天會派上用場喔！

但我自己還是比較喜歡在每個例子前產生物件，最後還是取決於你們團隊的決定和方向～


# Before & After Hooks 的選項

剛剛介紹完 `after hooks` 的運作模式，我要介紹一下傳入的參數所造成的差別！

我們一般都是預設在 `(:each)` 的情形，也就是你每一個 `example` 都跑一次的意思！

這邊介紹一個 `(:context)` 的選項，可以讓你在 `describe` & `context` 的前後執行 hooks，我們來看看例子吧！

{{< tabbed-codeblock "指定作用域" >}}
<!-- tab ruby -->
RSpec.describe "before and after hooks" do
  before(:context) do
    p "before context"
  end

  before(:example) do
    p "before example"
  end
  
  after(:example) do
    p "after example"
  end
  
  after(:context) do
    p "after context" 
  end

  it "test for before & after" do
    expect(1 + 1).to eq(2) 
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

接著看 OutPut 會非常明顯和清楚！

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109261412572.png)

`(:context)` 會在整個測試最開始的時候執行一次，然後會在整個測試結束的時候執行！

這個的用處在於，某些設定你只需要一次，就可以在每個測試裡面拿到，而不需要每一個 example 都產生一次，這樣就會有效能上很大的差別。

所以寫測試的時候，稍微考慮一下是不是都會被使用到而且又不會被改變，如果答案是確定的！那就可以使用 `(:context)` 的選項來幫助你喔！

# 箝套的邏輯和順序

熟悉的兩個 hooks 的運作模式之後，我們要來把 hooks 加入箝套裡面測試一下自己是不是真的理解了？

所以我們先上程式碼( 可以自己先想一下答案！ )

{{< tabbed-codeblock "測驗題目" >}}
<!-- tab ruby -->
RSpec.describe "before and after hooks" do
  before(:context) do
    p "OUTER Before context"
  end

  before(:example) do
    p "OUTER Before example"
  end

  it "will pass" do
    expect(1 + 1).to eq(2)
  end

  context "when it can pass" do
    before(:context) do
      p "INNER Before context"
    end

    before(:example) do
      p "INNER Before example"
    end

    it "will pass too" do
      expect(2 + 2).to eq(4)
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

有辦法一步步的推敲出正確答案的話，代表已經完全理解了 hooks 的運作順序了。

我們要去思考測試碼是由上至下的讀取，所以會先經過什麼？遇到什麼又會觸發什麼？

只要能一步一步地釐清其實就不會感覺非常複雜～

公佈答案：

![](https://cdn.jsdelivr.net/gh/robeeerto/Pics/img/202109261413786.png)

我猜最容易沒想到的地方就是在 context 裡面的 example 也吃到了外面的 before hooks 吧！

最外層的物件，在裡面也是拿得到的，這對於一開始寫 RSpec 的人還蠻有幫助的。

因為在一邊整理程式碼的時候，會出現這個物件會不會拿不到，要不要再寫一個 `let` 等等的疑問～

那至於為什麼要在不同的地方寫 hooks 呢？

是因為某些 Context 可能有屬於他的物件需要放在裡面，若是所有的東西都放在最外層，很有可能不小心就命到相同的名字，或是看起來很雜亂！

雖說寫在最外面肯定是不會有什麼問題，但能夠精準的物件放在他的作用域，也是一個工程師厲不厲害的指標喔！

# 利用作用域來覆寫物件

剛剛提到了內層可以取得外層的物件，但如果我想要在 context 裡面測試同一個物件，不同的行為怎麼做呢？

我們先用個範例來看看：

{{< tabbed-codeblock "測試不同行為" >}}
<!-- tab ruby -->
class Burger
  attr_accessor :type
  def initialize(type = "small")
    @type = type
  end
end

RSpec.describe Burger do
  let(:burger) { Burger.new("Big") }
  
  it "type will be Big" do
    expect(burger.type).to eq("Big")
  end
  
  context "when we didn't give argument" do
    let(:burger) { Burger.new } # 利用覆寫來通過測試
    it "typ will be small" do
      expect(burger.type).to eq("small")
    end
  end
end
<!-- endtab -->
{{</ tabbed-codeblock>}}

當我們想要給一個 context 的情境來測試沒有傳入參數的預設值時，要如何可以通過測試？

我們可以利用覆寫 burger 這個物件的內容，來通過測試。

當測試碼跑到第 18 行時，他會從自己的作用域開始找尋一個叫做 burger 的物件，若是沒有就會往上去找！

所以不覆寫的話，會找到第 9 行的 burger 物件！就會沒辦法通過測試。

所以當我們理解了作用域，就可以隨心所欲的撰寫測試，根據不同的情境加入不同的條件！

而且使用 `let` 也不擔心效能的問題，還記得我們在介紹 `let` 的時候提過的 lazy-loading 嗎？ 超讚的！

# 小結

今天介紹了 RSpec 的作用域，以及覆寫帶來的幫助，希望可以幫到像我一樣的新手。

畢竟測試也不是超基本的知識，還是需要自己去讀文件和練習 KATA 才會進步！

所以希望自己的想法可以有所幫助！

明天會介紹 `subject` 這個物件，也會介紹縮短程式碼的小方法！( Ruby 生態圈就是越簡潔越好～ )





